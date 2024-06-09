# dcss-server

이 스크립트는 던전 크롤 스톤 수프의 서버를 한번에 간단히 배포 및 관리할 수 있게 하기 위해 제작되었습니다. 최신 리눅스 환경에서 가능한 많은 포크 버전(DCSS CA, HellCrawl, GnollCrawl, BloatCrawl2, GoonCrawl, X-Crawl, StoatSoup, KimchiCrawl, BcadrenCrawl)과 정식 릴리즈 버전(0.11 ~ 0.31)을 포함합니다.

### First Run Guide:

#### Prerequisites

* Docker (with Docker Compose)

#### Fast deploy
참고사항: release.sh 스크립트의 다운로드 기능을 사용하기 위해서는 해당 환경에 `jq`와 `curl`이 설치되어 있어야 합니다. (데비안 환경에서 `apt install jq curl -y`를 이용하여 설치할 수 있습니다.) 
```bash
git clone https://github.com/refracta/dcss-server
cd dcss-server/server

# 사전 빌드된 게임 바이너리와 설정 다운로드
./release.sh download -p data -n game-data
# 랜덤 포트와 함께 실행
docker compose up -d && docker compose logs -f
# 지정된 포트에서 실행
docker compose -f docker-compose.yml -f docker-compose.ports.yml up -d && docker compose logs -f
```

#### Deploy with Build
```bash
git clone https://github.com/refracta/dcss-server
cd dcss-server/server

# 이 명령은 선택적으로 사용할 수 있습니다, ccache 파일을 다운로드하여 컴파일 속도를 가속할 수 있습니다. 
# 미적용시 GitHub Action Runner의 ubuntu-24.04 이미지 기준, 전체 빌드에 6시간 이상이 소요되며, 적용시 45분 정도로 가속됩니다.
./release.sh download -p data/ccache -n ccache

# USE_DWEM: https://github.com/refracta/dcss-webtiles-extension-module 사용을 적용합니다.
# USE_REVERSE_PROXY: X-Forwarded-For 아이피를 로그에 기록하기 위한 패치를 적용합니다.
# COMMAND: "build-all"=전체 버전을 모두 빌드합니다, "build-trunk"= 트렁크 버전만 빌드합니다.
USE_DWEM=true USE_REVERSE_PROXY=true COMMAND=build-all docker compose up -d && docker compose logs -f

# Docker Hub에 저장된 이미지를 다운로드하지 않고 빌드가 필요한 경우 다음 명령어를 사용할 수 있습니다.
COMMAND=build-all docker compose -f docker-compose.yml -f docker-compose.build.yml up -d && docker compose logs -f
```

#### Server Data
모든 서버 데이터는 `server/{versionsdb,crawl-master,dgldir,games}`에 저장됩니다.

### Repository Management
* 이 레포지토리는 crawl.nemelex.cards 운영을 위해서 사용됩니다.
* 새로운 포크 또는 버전 추가가 필요하다면, Pull-Request를 통해서 요청할 수 있습니다.

### Upstream projects
* https://github.com/crawl/dgamelaunch-config
* 던전 크롤 스톤 수프 서버 운영에 필요한 스크립트입니다. `utils/testing-container`에는 크롤의 CI/CD 검증 작업을 위해 제작된 컨테이너 환경 구성이 있습니다.

* https://github.com/Rytisgit/dgamelaunch-dcss-forks-server
* 이 프로젝트는 dgamelaunch-config을 기반으로 하여, 여러 포크를 컨테이너로 구성된 환경에서 한번에 쉽게 구성할 수 있도록 만들어졌습니다. 본 프로젝트는 이 프로젝트를 기반으로 시작되었습니다.

### Thanks to

DCSS IRC `#crawl-dev` 채널의 많은 개발자 분의 도움으로, 본 프로젝트의 구현 목표가 성공적으로 달성될 수 있었습니다. 
특히 서버 세팅과 관련해서 많은 도움을 주신 gammafunk, [DCSSReplay](https://github.com/rytisgit/dcssreplay)의 개발자이자 dgamelaunch-dcss-forks-server의 메인테이너로서
서버 설정 진행 과정을 모니터링하며 아낌없는 조언과 이슈 해결을 도와준 Rytisgit에게 감사를 전하고 싶습니다.