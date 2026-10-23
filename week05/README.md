1. 실습 목표Docker 실행 환경을 점검하고, 가상화 컨테이너 개념을 이해한다.   이미지와 컨테이너의 관계를 파악하고 기본 명령어(run, ps, logs, exec, stop, rm 등)를 익힌다.   Nginx 웹 서버를 컨테이너로 띄워보고, 포트 포워딩 및 격리 구조를 확인한다.   GitHub Issue와 PR(Pull Request)을 연계하여 협업 및 실습 관리 프로세스를 경험한다.   2. 주요 개념 정리(1) 가상머신(VM) vs 도커 컨테이너하이퍼바이저 방식 (VM): 하드웨어를 가상화하여 각각의 Guest OS를 통째로 올리기 때문에 무겁고 용량이 큼 (GB 단위).   도커 컨테이너 방식: 호스트 OS의 커널을 공유하면서 프로세스 수준으로 실행 환경을 격리하므로 가볍고 빠르게 실행됨 (MB 단위).   (2) 이미지(Image)와 컨테이너(Container)이미지: 앱과 실행 환경을 함께 묶어놓은 읽기 전용 템플릿 (붕어빵 틀에 비유할 수 있음).   컨테이너: 이미지를 실행해 인스턴스화한 상태로, 쓰기 계층이 존재하여 내부 파일을 수정할 수 있지만 컨테이너를 삭제하면 변경 사항은 사라짐 (붕어빵).   3. 실습 내용 및 명령어 정리① 기본 환경 확인 및 테스트 실행도커 버전 확인 및 hello-world 이미지를 통한 정상 작동 테스트:   Bashdocker --version
docker run hello-world
동작 원리: 로컬에 hello-world 이미지가 없으면 Docker Hub 레지스트리에서 자동으로 이미지를 다운로드(pull)한 뒤 컨테이너를 실행한다.   ② Nginx 웹 서버 컨테이너 실행하기백그라운드(-d)로 실행하며, 호스트의 8080번 포트와 컨테이너의 80번 포트를 연결(-p)하고 컨테이너 이름을 web으로 지정한다:   Bashdocker run -d -p 8080:80 --name web nginx
확인: 브라우저나 curl을 통해 http://localhost:8080에 접속하면 "Welcome to nginx!" 기본 화면이 출력되는 것을 확인.   ③ 주요 도커 관리 명령어컨테이너 목록 확인:Bashdocker ps      # 실행 중인 컨테이너
docker ps -a   # 정지된 컨테이너 포함 전체 확인
로그 확인:Bashdocker logs -f web   # 실시간으로 로그 모니터링 (종료: Ctrl + C)
정지 및 재시작:Bashdocker stop web2     # 컨테이너 정지
docker start web2    # 컨테이너 재시작
삭제:Bashdocker rm -f web2    # 실행 중인 컨테이너 강제 삭제 (-f)
④ 컨테이너 내부 진입 및 파일 수정 (exec)컨테이너 내부 셸에 접속하여 파일 시스템을 확인하고 수정해 보았다:   Bashdocker exec -it web bash
# 컨테이너 내부 진입 후 기본 인덱스 페이지 내용 변경 테스트
echo "<h1>Hi</h1>" > /usr/share/nginx/html/index.html
exit
참고: 컨테이너를 삭제하고 동일한 이미지로 다시 실행하면, 쓰기 계층에 저장되었던 수정 내용(Hi)은 초기화되고 기본 Nginx 화면으로 돌아간다.   4. 혼자서 해보기 (응용 실습)다중 컨테이너 실행:
서로 다른 포트(8091, 8092, 8093)를 할당하여 세 개의 Nginx 컨테이너를 각각 띄운다.   Bashdocker run -d -p 8091:80 --name nginx1 nginx
docker run -d -p 8092:80 --name nginx2 nginx
docker run -d -p 8093:80 --name nginx3 nginx
내부 파일 커스텀: 각 컨테이너 내부(exec)로 들어가 index.html을 각각 다르게 수정하여 접속 시 구분되도록 설정.   정리: 실습이 끝난 후 사용한 컨테이너들을 깔끔하게 삭제하여 자원을리소스 정리.   Bashdocker rm -f nginx1 nginx2 nginx3 web
5. GitHub Issue 및 PR 연계 관리Issue 생성: GitHub 저장소에서 5주차 Docker 실행 환경 확인 및 nginx 실행 제목으로 이슈(#6 등)를 등록하고 완료 조건을 체크리스트로 작성.   작업 브랜치 파기 및 기록:Bashgit switch -c docs/issue-6-docker-start
# week05/README.md 파일에 실습 결과 기록 및 커밋
Commit & Push:Bashgit add week05/
git commit -m "5주차: Docker 실행 환경 및 nginx 확인 (#6)"
git push origin docs/issue-6-docker-start
Pull Request (PR) 및 Merge:PR 본문에 Close #6을 작성하여 머지(Merge)와 동시에 이슈가 자동으로 닫히도록 연동 완료.   
