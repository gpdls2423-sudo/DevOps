#3주차복습 

목표 
- 반복적으로 사용하는 명령어를 셸 스크립트로 작성하고 실행한다
- 셸 스크립트의 권한,shebang,경로 처리 방식을 이해한다
- 환경변수와 조건문을 할용하여 실행환경을 설정하고 오류 상황에 대응한다
- Qwen과 Ollema 기반 로컬 ai서비스를 실행하며 자동화의 필요성과 활용 방법을 이해한다

Ollama설치 

Qwen 다운

수업 순서 
1. 셸 스크립트와 자동화의 필요성
2. 파이썬 웹 프로그램을 이용한 로컬 ai서비스 실행 
- qwen 모델 다운로드 및 Ollama 서버 실행 
3. 셸 스크립트 작성 및 실행
- 환경변수 설정
- 조건문을 이용한 실행 환경 점검

자동화의 필요성
Owen w/ Ollama 로컬 대화방 만들기 

Owen 
- 알리바바 클라우드(Alibaba Cloud)에서 개발한 대형 언어 모델

- 다양한 모델 라인업 
 - 텍스트 생성,추론,이미지,비디오,오디오 통합 처리 등

Ollama 
- 로컬 환경에서 대규모 언어 모델(LLM)을 쉽게 설치 및 실행할 수 있게 해주는 오픈소스 플랫폼 

Ollama 생성 API 
- 기본적으로 스트리밍
- stream: false
 - 응답을 한 번에 보여주기 
- think: false 
 - 생각 모드 비활성화

구조
브라우저(로컬호스트, 질문전송/답변표시) -> python 웹 앱(질문을 API로 전달)
-> Ollama서버 (모델실행) -> qwen3:0.6b

웹 브라우저에서 열기 feat.파이썬

로컬 -> ubuntu 터미널 파일 복사 방법
터미널 이용하기 
cp "mnt/c/Users/PC/Downloads/chat.py" .

파이썬 파일 실행
- python3 chat.py

다른 모델 사용 방법 
- ollama pull qwen3:1.7b
- ollama list 

b란?
- billion(10억)
- 모델이 학습한 파라미터 개수(뇌의 크기비유)
- GPT-3: 175B(GPT-4 이후 주요 계열은 공개하지 않는 흐름)

파라미터가 클 수록?
- 더 많은 언어 패턴, 문법 규칙 등 저장할 여유가 생긴다.
- 비슷한 계열와 학습 조건에서는 더 높은 성능을 보인다
	- 단, 실제성능은 모델구조,학습데이터, 학습 방법 등에 따라 달라진다.
- 곱하고 더하는 연산량도 커져 -> 느린 응답, 더많은 저장공간 및 메모리가 필요하다.

모델 수정을 위해 nano 에디터 사용 - nano chat.py 
!python3 - python3 가장 최근 명령 재실행 


셸 스크립트
- nano start.sh : start.sh 파일 생성

#!/bin/bash 
- #!: 셔뱅(shebang)/어떤 인터프리터로 실행할지 os에 알려주는 부분으로 파일의 첫 줄에 작성
- #!: 이 파일을 실행할 때, 바로 뒤에 적힌 프로그램으로 이 파일을 해석하라 
- #!/bin/bash : 이 파일은 bash 문법이니, bash로 실행하라
- 주의- #하나만 있으면 bash에서 주석으로 처리한다. 

#!(셔뱅)을 파이썬으로 이해하기
파일이름 : run_py.sh
- #!/usr/bin/env python3 또는 #!/user/bin/python3
- print('Hello, devops')

권한 설정 및 실행 
- chmod +x run_py.sh
- ./run_py.sh
Hello, devops!

셸스크립트 코드 분석
cd "$(dirname "$0")" || exit 1
- 해당 스크립트가 있는 디렉터리로 이동하고, 이동 실패시 스크립트 종료 
- $0: 현재 실행한 스크립트의 이름 또는 경로
- "$0": 스크립트 경로에 공백이 있을 수 있기 때문에 따옴표 사용 
- dirname : 파일 경로에서 파일이름 빼고 디렉터리 부분만 출력하는 명령어

- $(...) : 명령어 치환 , 괄호 안의 명령어를 실행하고 그 명령의 표준 출력을 이 자리에 [>
- ex) echo"$(pwd)"의 결과는 pwd와 같다.

- exit -> 리눅스 명령어는 성공 또는 실패 값을 남긴다
- 명령종료상태 성공:0
- 명령종료상태 실패: 0 이외 
- cd를 성공하면 exit status = 0 -> true
- 존재하지 않는 디렉터리면 exit status = 1 -> false

- || : 논리 or, 앞 명령이 실패하면 뒤 명령을 실행한다.
 * 숫자0은 거짓이지만, 명령 종료 상태 0은 성공을 뜻한다. (숫자 그대로의 불리언 값으로 대응시키면 안된다.)
- || exit 1 : 앞 명령이 실패할 경우 종료를 하되, '실패'를 의미하는 상태코드 1을 남긴다. 


./command.sh qwen3:0.6b 8000
- command.sh : $0 -> 스크립트
- qwen3:0.6b : $1 -> 첫 번째 인자
- 8000 : $2 -> 두 번째 인자 

참고
- $# : 전달받은인자(argument) 개수
- $@ : 전달받은 모든 인자
- $? : 직전 실행한 명령의 종료 상태

exec python3 chat.py 
- 현재 bash를 python3으로 대체(pid 유지)
- 종료 시, 출력 안된다.
- exec 빼면 출력됨.

cat > start_with_export.sh << 'E0F'
- 입력하는 여러 줄의 내용을 start_with_export.sh에 저장하되 E0F줄이 나오면 입력을 끝낸다. 
- cat : 입력받은 내용 그대로 출력
- > : 표준 출력을 파일로 보냄
- start_with_export.sh : 저장한 파일
- << : heredoc(히어독, Here Document) 시작
- 'EOF': 입력 종료 표시(End of File) <- 자유롭게 설정이 가능하다.
- 'EOF' : EOF문자열은 finish나 end 등 다른 문자열로 변경 가능하다. 

셸 스크립트 수정해 포트번호 8080으로 바꿔 실행하기
- $ cat > start_with_export_2.sh << 'EOF'
- > #!/bin/bash
- > cd "$(dirname "$0")" || exit 1
- > export WEB_PORT="8080"
- > exec ./start.sh
- > EOF
- $ chmod u+x start_with_export_2.sh
- $ ./start_with_export_2.sh

프로그램 설치 여부 알려주는 부분 만들기 
- cd "$(dirname "$0")" || exit 1
- if ! command -v python3 >/dev/null 2>&1; then
- 	echo "Python3를 먼저 설치하세요." >&2
- 	exit 1
- fi
- exec python3 chat.py # 이 줄 바로 위에 검사 코드를 추가

조건문 
- if 명령어; then 
- 	실행할 내용
- fi

if ! command -v python3 >/dev/null 2>&1; then 
- command -v : 명령어를 실행할 수 있으면 , 어느 경로에 있는지 확인
- 없는 명령어라면, 아무것도 출력하지 않고 실패 상태를 반환

if ! command -v python >/dev/null 2>&1; then
- 만약 python3 명령어를 찾을 수 있다면
if command -v python3 ....; then
- 만약 python3 명령어를 찾을 수 없다면 

if ! command -v python3 >/dev/null 2>&1; then
- > : 표준 출력(stdout)을 다른 곳으로 보내는 리다이렉션
- /dev/null : 리눅스의 특수한 장치로 일종의 쓰레기통

>/dev/null : 출력을 버림 , 우리가 알고싶은 것은 "있는지 없는지"이기 때문에 출력을 버림

if ! command -v python3 >/dev/null 2>&1; then 
2>&1
- 2번 표준 오류를 1번 표준 출력이 가는 곳(여기서는 /dev/null)으로 보내기
- 순서에 따라 다른 해석이 됨 (왼쪽→오른쪽) 
참고
- 0: 표준입력 stdin
- 1: 표준출력 stdout
- 2: 표준오류 stderr

exit 1 
- 오류 상태로 종료 
- 오류 메세지로 보내는 것과 오류 상태로 종료하는것은 다르다
- 즉, 자동실패처리되지 않는다.

정리
Qwen 
- 답변을 만드는 모델
Ollama 
- 모델을 실행하고 관리하는 로컬 서버 및 도구
- 답변을 만드는 모델을 실행
셸 스크립트
- 여러 명령을 파일에 작성하여 같은 절차로 실행하는 셸 스크립트 작성
- 실행권한, shebang을 확인하고 작업 경로 설정
- 환경 변수로 프로그램의 실행 설정에 전달
- 명령의 종료 상태로 성공과 실패 판단 가가능
- 표준 출력, 표준 오류 , 실패 종료 등 구분 

