# Git 협업 실습: Fetch, Merge 및 충돌 해결

이 실습은 하나의 GitHub 저장소를 두 개의 독립된 로컬 저장소에서 사용하면서  
작업자 A와 작업자 B의 협업 과정을 재현합니다.

핵심 흐름은 다음과 같습니다.

```text
fetch → merge → 충돌 해결 → commit → push
```

## 전체 협업 흐름

```mermaid
sequenceDiagram
    autonumber

    actor A as 작업자 A
    participant LA as A 로컬 저장소
    participant GH as GitHub 원격 저장소
    participant LB as B 로컬 저장소
    actor B as 작업자 B

    Note over A,B: 1. 실습 환경 구성

    A->>LA: 기존 로컬 저장소 사용
    B->>GH: 같은 GitHub 저장소 Clone
    GH-->>LB: 저장소 복제

    A->>LA: user.name = Worker A 설정
    B->>LB: user.name = Worker B 설정

    Note over A,B: 2. 충돌 없는 협업

    A->>LA: worker-a.md 생성
    A->>LA: git add / commit
    LA->>GH: git push
    GH-->>GH: origin/main 갱신

    B->>LB: git fetch origin
    GH-->>LB: origin/main 정보 가져오기

    Note right of LB: fetch는 원격 커밋 정보를 가져오지만<br/>작업 파일은 바로 변경하지 않음

    B->>LB: git merge origin/main
    LB-->>B: worker-a.md 반영

    B->>LB: worker-b.md 생성
    B->>LB: git add / commit
    LB->>GH: git push
    GH-->>GH: origin/main 갱신

    A->>LA: git fetch origin
    GH-->>LA: B의 커밋 정보 전달

    A->>LA: git merge origin/main
    LA-->>A: worker-b.md 반영

    Note over LA,LB: 두 로컬 저장소가 최신 상태로 동기화됨

    Note over A,B: 3. 같은 README.md를 수정하여 충돌 재현

    A->>LA: README.md 같은 문장 수정
    A->>LA: git add / commit
    LA->>GH: git push
    GH-->>GH: A의 변경 origin/main 반영

    B->>LB: 같은 README.md 문장 수정
    B->>LB: git add / commit

    B->>GH: git push
    GH-->>B: rejected - fetch first

    Note over GH,LB: origin/main과 B의 main이 서로 diverged

    Note over A,B: 4. Fetch 및 Merge

    B->>GH: git fetch origin
    GH-->>LB: A의 최신 커밋 가져오기

    B->>LB: git merge origin/main
    LB-->>B: README.md Merge Conflict 발생

    Note right of LB: both modified: README.md<br/>main|MERGING

    Note over B,LB: 5. 충돌 해결

    B->>LB: README.md 충돌 내용 확인

    Note right of LB: HEAD = 작업자 B의 변경<br/>origin/main = 작업자 A의 변경

    B->>LB: 두 작업자의 내용을 검토
    B->>LB: 최종 내용으로 수정
    B->>LB: 충돌 표시 제거

    B->>LB: git add README.md
    LB-->>B: 충돌 해결 완료

    B->>LB: git commit
    LB-->>B: Merge Commit 생성

    Note over B,GH: 6. Merge 결과 Push

    B->>GH: git push
    GH-->>GH: Merge Commit origin/main 반영

    Note over A,GH: 7. 작업자 A 최종 동기화

    A->>GH: git fetch origin
    GH-->>LA: Merge Commit 가져오기

    A->>LA: git merge origin/main
    LA-->>A: 최종 변경 내용 반영

    Note over A,B: 작업자 A · 작업자 B · GitHub 최종 동기화 완료
```

## 충돌 발생 및 해결 핵심 흐름

```mermaid
sequenceDiagram
    actor A as 작업자 A
    participant GH as GitHub
    actor B as 작업자 B

    A->>A: README.md 수정
    A->>A: git add / commit
    A->>GH: git push

    B->>B: 같은 README.md 문장 수정
    B->>B: git add / commit

    B->>GH: git push
    GH-->>B: rejected (fetch first)

    B->>GH: git fetch origin
    GH-->>B: A의 최신 커밋 전달

    B->>B: git merge origin/main
    B-->>B: README.md 충돌 발생

    B->>B: 충돌 내용 수정
    B->>B: git add README.md
    B->>B: git commit

    B->>GH: git push

    A->>GH: git fetch origin
    GH-->>A: Merge Commit 전달

    A->>A: git merge origin/main

    Note over A,B: A · B · GitHub 최신 상태 동기화
```

## 충돌 해결 순서

```bash
git fetch origin
git merge origin/main
```

충돌이 발생하면 `README.md`의 충돌 표시를 확인합니다.

```text
<<<<<<< HEAD
작업자 B의 내용
=======
작업자 A의 내용
>>>>>>> origin/main
```

최종 내용을 결정한 뒤 충돌 표시를 모두 삭제합니다.

```bash
git add README.md
git commit -m "B: A의 변경과 README 충돌 해결"
git push
```

다른 작업자는 최종 Merge 결과를 다시 가져옵니다.

```bash
git fetch origin
git merge origin/main
```

## 핵심 정리

각 로컬 저장소는 같은 GitHub 저장소에 연결되어 있어도 서로 독립적입니다.

```text
작업자 A 로컬 main
작업자 B 로컬 main
GitHub origin/main
```

따라서 다른 작업자가 Push한 내용이 내 로컬 저장소에 자동으로 반영되지는 않습니다.

일반적인 협업 순서는 다음과 같습니다.

```text
git fetch origin
        ↓
원격 변경 확인
        ↓
git merge origin/main
        ↓
파일 수정
        ↓
git add
        ↓
git commit
        ↓
git push
```
