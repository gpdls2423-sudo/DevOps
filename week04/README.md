4week 자동화와 협업

목표 
- 절대경로와 상대경로 구분, 안전하게 경로 찾기
- 'grep', 'find', '|'(파이프)를 이용하여 필요한 파일과 내용을 찾을 수 있다
- 변수,인자,조건,반복을 이용하여 반복 작업을 자동화하는 셸 스크립트를 작성할 수 있다.
- 프로그램과 프로세스를 구분하고 PID를 이용하여 프로세스를 확인하고 종료할 수 있다.
- 변경 내용을 확인하고 "브랜치 → PR → Conflict 해결 → Merge"의 협업 흐름을 수행할 수 있다.

순서
1. 자동화 기초
2. 검색 및 셸 자동화
3. 프로세스와 백업 
4. git/github 협업 
5. conflict, merge 

자동화 기초
디렉터리는 서버의 지도 
- home/ : 사용자 작업 공간 
- etc/  : 시스템과 프로그램의 설정
- var/log/ : 시스템과 서비스의 로그가 주로 저장되는 위치
- tmp/ :임시파일
- usr/bin/ : 주요 실행 파일과 명령어

내가 만든 파일은? : 사용자 홈 디렉터리
시스템 전역 설정 찾기 -> /etc
왜 실패했나 ? -> /var/log 또는 애플리케이션 로그

ls -ld /home /etc /var/log /tmp /usr/bin
- -l: 상세보기
- -d: 디렉터리 자체 정보

절대경로와 상대경로 
- 절대경로 :/home/neary/devops/week03/qwen-web/start.sh
- 상대경로 : ./week03/qwen-web/start.sh (단, 현재 위치가 ~/devops 일 때)
(또는) week03/qwen-web/start.sh
- 현재 위치 . 
- 상위 위치 ..
- 홈 ~
- 루트 /

자동화 기초 - 파일찾기
find . -type f -name "*.sh" 
- 쉘 스크립트 파일 찾기

grep -R -n "exit 1" week03 2>/dev/null
- exit 1을 포함하는 파일 찾기
- -R: 하위 디렉터리까지 모두 검색
- -n : 줄 번호 표시

 grep -R "exit 1" week03 2>/dev/null | wc -l 
- exit 1을 포함하는 파일 개수 보기
- wc -l : (word count, lines) 입력된 텍스트의 줄 수를 계산

| (파이프)
- 파일을 전달하는 것이 아닌 앞 명령의 표준 출력을 뒤 명령의 표준 입력으로 연결 
grep 이름의 역사적 유래:  g/re/p (global / regular expression / print) 
- 텍스트에서 원하는 패턴이 포함된 줄을 검색하는 명령 

텍스트 파일 만들고 확인하기
- head -2 diary.txt :앞 2줄 출력 
- tail -1 diary.txt :뒷 1줄 출력
- wc -l diary.txt   :전체 줄 수 출력

greb과 find 
- grep "ERROR" server.log : ERROR가 든 줄만
- grep -i "error" server.log  : -i: 대소문자 무시
- grep -c "ERROR" server.log : -c: 개수만
- grep -n "ERROR" server.log : -n: 줄 번호와 함께
- grep -v "INFO" server.log : -v: INFO가 없는 줄만 (반전)

- find . -name "*.txt" : txt 파일 찾기
- find . -type d : 디렉터리만 찾기
- find / -name "start.sh" 2>/dev/null : 시스템 전체에서 찾기 (오래 걸릴 수 있음)
- find / -name "start.sh"  : 2>/dev/null 빼고 실행해보기


파이프와 리다이렉션
 
로그 내용 필터해서 확인 
cat server.log | grep ERROR 
- cat의 결과를 grep에게 전달하기
- grep ERROR server.log 동일한 결과
cat server.log | grep ERROR | wc -l 
-  위의 결과를 wc -l로 확인
ls -l /etc | grep conf | head -5 
- 파이프 2번 사용한 예제
저장 
-  grep ERROR server.log > errors.txt
- cat errors.txt

변수와 명령 치환 
cat > var.sh << 'EOF'
> #!/bin/bash
> USERNAME="홍길동"
> TODAY=$(date +%Y%m%d)
> echo "이름: $USERNAME"
> echo "파일명: backup_${TODAY}.tar.gz"
> EOF
chmod u+x var.sh && ./var.sh

셸 스크립트에서 인자 입력 여부 확인 방법
cat > greet.sh << 'EOF'
> #!/bin/bash
> if [ $# -eq 0 ]; then         # 주의: [ , ] 주변에 공백 필요
> echo "사용법: $0 [이름]"
> exit 1                           # 비정상 종료
> fi
> echo "안녕하세요, $1님! (인자 개수: $#)"   # ${1}도 사용 가능
> EOF
chmod u+x greet.sh
./greet.sh # 사용법 출력
echo $? # → 1
./greet.sh 홍길동 # 안녕하세요, 홍길동님! (인자 개수: 1) 출력
echo $? # → 0

조건문과 파일검사
cat > check.sh << 'EOF'
> #!/bin/bash
> FILE="$1"
> if [ -z "$FILE" ]; then
> echo "파일명을 입력하세요"; exit 1
> fi
> if [ -f "$FILE" ]; then
> echo "파일"
> elif [ -d "$FILE" ]; then
> echo "디렉터리"
> else
> echo "없음"; exit 1
> fi
> EOF
chmod u+x check.sh
./check.sh var.sh # 파일
./check.sh /etc # 디렉터리
./check.sh 없는파일 # 없음


현재 디렉터리의 .sh 파일 이름을 출력하는 예제
-  for f in *.sh; do echo "스크립트: $f"; done
- var.sh
- greet.sh
- check.sh


프로세스와 백업 - 패키지 관리와 실행 환경 
패키지 관리 
$ sudo apt update
$ sudo apt install -y tree jq htop
$ tree ~/devops -L 2 
설치까지 통째로 포장하기

프로그램(program): 디스크에 저장된 실행코드
프로세스(process): 실행 중인 프로그램 

 ps aux | head -5 # 프로세스 정보 앞 5줄
$ ps aux | grep bash # bash를 포함하는 프로세스
$ htop # 실행 중인 프로세스, CPU, 메모리 사용량을 실시간으로 확인하는 도구 (종료: q)
# 직접 만들고 죽여보기
$ sleep 300 & # & = 백그라운드 (화면에 <PID> 표시)
$ jobs # 현재 셸의 실행 작업
$ ps aux | grep sleep # PID 확인
$ kill <PID> # 정상 종료 요청 (SIGTERM)
$ ps aux | grep sleep # 사라진 것 확인
$ sleep 300 &
$ kill -9 <PID> # 강제 종료 (SIGKILL) — 최후의 수단


ps aux: 시스템에서 실행 중인 거의 모든 프로세스를 상세하게 보여주기
aux: a(모든 사용자) u(사용자 중심의 상세) x(터미널에 연결되지 않은 프로세스 포함, 백그라운드 등))

tar,diff 이해하기
tar : 여러 파일을 하나의 아카이브로 묶는 도구
- -tzf: 내용확인
- -xzf: 압축 해제
- -czf: 압축 생성
- -C : change directory(작업할 디렉터리 지정)

diff -rq
-  다르면, A and B differ 등으로 표시됨 (r: recursive, q: brief 모드 - 차이가 있는지만 간단히 표시)
- echo $? 시, 1 출력 (0: 같음, 1: 다름, 2: 오류)
-  diff -ru A B → 다른 내용을 -와 +로 보여줌 (u: unified format, 변경 사항을 표현하는 표준 텍스트 형식)

.gitignor 
- 지정한 파일을 새로 git 추적 대상에 포함하지 않도록 한다.
(지정된 파일은 git add 대상에서 제외되어 일반적으로 커밋 및 푸시되지 않음)
- 이미 git이 추적 중이라면, 추가해도 자동으로 제외되지않는다. 
 git rm --cached <파일명> 또는 git rm -r --cached <디렉터리명>로 해결


깃협업
같은 파일을 여러 명이 고칠 때 코드 충돌이 나면?
깃/깃허브으로 협업하는 이유
- 실패해도 안전한 공간이 있음 -> 브랜치
- 합치기 전에 누군가 봐줘야 함 -> pull Request(코드 리뷰)
- 컴퓨터가 자동으로 검사 해줌 -> CL

[참고] PR 생성 업데이트 시, github actions가 실행되도록 설정할 수 있다
브랜치
평행세계 
- main : 기준이 되는 안정적인 브랜치
- feature/backup-log : 내 작업실(터져도 main은 안전)
실습
- git branch : 현재 브랜치 확인
- git switch -c feature/backup-log : 브랜치생성

팀에서 사용하는 브랜치 이름 예시
- feature/기능명 (또는 feat/기능명)
- fix/버그명
- docs/문서명 

변경 확인 후 커밋 그리고 푸시 

합치기(스쿼시 머지 사용)
- gh pr merge --squash --delete-branch : main에 합치면서 삭제
- git switch main : main 브랜치로 이동
- git pull --ff-only : 바뀐 내용 pull 하기
- git log --oneline -3 : 최근 3개 커밋 보기

충돌 경험 및 해결 
- --merge : 기존 커밋 유지 + merge commit 생성
- --squash : 여러 커밋을 하나로 합쳐 반영 
- --rebase : feature 브랜치와 각 커밋을 새 커밋으로 만들어 main 뒤에 일렬로 적용


자동화
• 디렉터리와 경로를 이해하고, find, grep, 파이프로 필요한 정보 찾기
• 변수, 인자, 조건, 반복을 이용하여 셸 스크립트 작성
• 프로세스와 PID를 확인하고, 백업 및 복원 작업 자동화
협업
• .gitignore로 관리하지 않을 결과물 제외
• 브랜치 → 커밋/푸시 → PR → Merge
• Conflict 발생 → 로컬에서 해결 → Commit/Push → PR Merge

