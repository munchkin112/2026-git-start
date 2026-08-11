코드 실행 시퀀스 다이어그램
1. 실행 흐름
첨부된 실습 파일은 Python 코드 실행 과정이 아니라 Git 협업 과정을 설명하는 실습이다. 따라서 여기서는 사용자 → 프로그램 → 클래스 → 객체 구조가 아니라, 실제 실습에 등장하는 작업자 A 로컬 저장소 / 작업자 B 로컬 저장소 / GitHub 원격 저장소를 중심으로 표현했다.
특히 실습의 핵심인 Clone → 수정 → Commit → Push → Fetch → Merge → 충돌 → 해결 → Merge Commit → Push → 최종 동기화 순서를 실제 실행 순서에 맞춰 구성했다. 4-2_2단계_작업자_AB의_Fetch_Merge_및_충돌_해결.pdfPDF
sequenceDiagram
    participant A as 작업자 A 로컬
    participant B as 작업자 B 로컬
    participant GH as GitHub 원격 저장소

    Note over A,GH: 1. 작업자 A와 B의 로컬 저장소 준비

    A->>GH: 기존 저장소 사용
    B->>GH: git clone
    GH-->>B: 저장소 복제 및 origin 연결

    A->>A: git status
    B->>B: git status
    A->>A: Worker A 작성자 설정
    B->>B: Worker B 작성자 설정

    Note over A,GH: 2. 충돌 없는 협업 - 작업자 A

    A->>A: worker-a.md 생성
    A->>A: git add worker-a.md
    A->>A: git commit
    A->>GH: git push
    GH-->>A: A의 커밋 반영

    Note over B,GH: 3. 작업자 B가 A의 변경사항 가져오기

    B->>GH: git fetch origin
    GH-->>B: 원격 커밋 정보 전달
    B->>B: main..origin/main 확인
    B->>B: git merge origin/main
    B->>B: worker-a.md 반영

    B->>B: worker-b.md 생성
    B->>B: git add worker-b.md
    B->>B: git commit
    B->>GH: git push
    GH-->>B: B의 커밋 반영

    Note over A,GH: 4. 작업자 A가 B의 변경사항 가져오기

    A->>GH: git fetch origin
    GH-->>A: B의 원격 커밋 정보 전달
    A->>A: main..origin/main 확인
    A->>A: git merge origin/main
    A->>A: worker-b.md 반영

    Note over A,B: 여기까지는 서로 다른 파일을 수정했기 때문에 충돌 없이 병합

    Note over A,GH: 5. 충돌 실습을 위한 상태 동기화

    A->>GH: git fetch origin
    GH-->>A: 최신 원격 정보 전달
    A->>A: git merge origin/main

    B->>GH: git fetch origin
    GH-->>B: 최신 원격 정보 전달
    B->>B: git merge origin/main

    Note over A,B: A와 B가 동일한 README.md 상태를 가짐

    Note over A,GH: 6. 공통 충돌 문장 준비

    A->>A: README.md에 공통 문장 추가
    A->>A: git add README.md
    A->>A: git commit
    A->>GH: git push
    GH-->>A: 공통 문장 반영

    B->>GH: git fetch origin
    GH-->>B: 공통 문장 커밋 정보 전달
    B->>B: git merge origin/main
    Note over A,B: 이제 두 작업자의 README.md가 동일한 상태

    Note over A,GH: 7. 작업자 A가 README.md 수정 후 먼저 Push

    A->>A: README.md 수정
    A->>A: git add README.md
    A->>A: git commit
    A->>GH: git push
    GH-->>A: A의 README 수정 반영

    Note over B: B는 아직 A의 변경사항을 가져오지 않음

    Note over B: 8. 작업자 B도 같은 문장을 다르게 수정

    B->>B: README.md 수정
    B->>B: git add README.md
    B->>B: git commit

    B->>GH: git push
    GH-->>B: Push 거절
    Note over B,GH: 원격에 B가 모르는 A의 커밋이 존재함

    Note over B,GH: 9. Push 거절 원인 확인

    B->>GH: git fetch origin
    GH-->>B: A의 커밋 정보 전달

    B->>B: git log --oneline --graph --all
    B->>B: main..origin/main 확인
    B->>B: origin/main..main 확인

    Note over B,GH: 로컬 main과 origin/main이 서로 다른 커밋을 가짐
    Note over B,GH: 즉, 두 브랜치가 diverged 상태

    Note over B,GH: 10. 작업자 B가 Merge 실행

    B->>B: git merge origin/main
    B->>B: README.md 비교
    B->>B: 같은 부분을 서로 다르게 수정한 것을 발견
    B-->>B: Merge Conflict 발생

    Note over B: README.md에 충돌 표시 생성
    B->>B: <<<<<<< HEAD
    B->>B: B의 로컬 내용
    B->>B: =======
    B->>B: A의 원격 내용
    B->>B: >>>>>>> origin/main

    Note over B: 11. 충돌 내용 검토 및 최종 내용 결정

    B->>B: A와 B의 변경 내용 비교
    B->>B: 최종 문장으로 수정
    B->>B: 충돌 표시 삭제

    B->>B: git add README.md
    B->>B: git status
    Note over B: 파일 충돌은 해결되었지만 Merge는 아직 진행 중

    Note over B: 12. Merge Commit 생성

    B->>B: git commit
    B-->>B: MERGING 상태 종료

    B->>B: git log --oneline --graph --all
    Note over B: A 커밋 + B 커밋 + Merge Commit 구조 확인

    Note over B,GH: 13. 충돌 해결 결과를 GitHub에 Push

    B->>GH: git push
    GH-->>B: Merge Commit 반영

    Note over A,GH: 14. 작업자 A가 최종 결과 동기화

    A->>GH: git fetch origin
    GH-->>A: B의 Merge Commit 정보 전달

    A->>A: main..origin/main 확인
    A->>A: git merge origin/main
    A->>A: README.md 최종 내용 확인

    Note over A,B,GH: 최종 상태
    Note over A,B,GH: 작업자 A / 작업자 B / GitHub가 동일한 최신 결과를 가짐

2. 실행 흐름 요약
작업자 A와 B는 같은 GitHub 저장소를 각각 가지고 있지만, 로컬 저장소는 서로 독립적이다. 한쪽에서 수정해도 다른 쪽이 자동으로 바뀌지 않는다. 4-2_2단계_작업자_AB의_Fetch_Merge_및_충돌_해결.pdfPDF

작업자 A가 push하면 GitHub에 변경사항이 올라가고, 작업자 B는 fetch로 원격에 새로운 커밋이 있다는 정보를 가져온 뒤 merge로 자신의 로컬에 반영한다.

서로 다른 파일을 수정하면 Git이 자동으로 합칠 가능성이 높지만, A와 B가 같은 README.md의 같은 문장을 서로 다르게 수정하면 충돌이 발생한다.

B의 push가 거절된 이유는 GitHub에 이미 A의 커밋이 있는데 B의 로컬에는 그 커밋이 없기 때문이다. B는 fetch → merge를 수행하고, 이 과정에서 충돌을 직접 해결한다.

B가 충돌 내용을 수정한 뒤 git add → git commit으로 Merge Commit을 만들고 push하면, 마지막으로 A가 fetch → merge하여 A·B·GitHub의 상태를 동일하게 만든다.

3. 핵심 개념
개념	쉽게 말하면
로컬 main	내 컴퓨터에 있는 작업 기록
origin/main	GitHub의 main이 어디까지 진행됐는지 내 로컬 Git이 기억하고 있는 위치
git fetch	GitHub에 뭐가 새로 올라왔는지 정보만 가져오는 것
git merge	가져온 변경사항을 내 로컬 작업에 합치는 것
Push 거절	GitHub에 내가 모르는 더 최신 작업이 있어서 바로 올리면 안 되는 상황
diverged	A와 B의 작업이 서로 다른 방향으로 갈라진 상태
Merge Conflict	Git이 A와 B 중 어느 내용을 선택해야 할지 자동으로 결정하지 못한 상태
<<<<<<< HEAD	현재 작업자 B의 로컬 내용
=======	두 내용을 나누는 경계
>>>>>>> origin/main	GitHub에서 가져온 A의 내용
Merge Commit	A와 B의 서로 다른 작업을 합쳤다는 것을 기록하는 커밋


이 실습에서 가장 중요한 한 줄
fetch는 "다른 사람이 뭘 했는지 가져오기", merge는 "그 작업을 내 작업과 합치기", 충돌이 나면 사람이 최종 내용을 결정한 뒤 commit하고 push하는 과정이다.