# eb-tomcat-snakes(http://bit.ly/eb-handson)
본 Tomcat 애플리케이션은 AWS Elastic Beanstalk 환경에서 J2EE 애플리케이션이 RDS를 사용하는 방법을 보여줍니다. 이 프로젝트에는 서블릿, JSP, 심플 태그(simple tag) 지원, 태그 파일(tag file), JDBC, SQL, Log4J, 부트스트랩(Bootstrap), Jackson 과 Elastic Beanstalk 환경 파일들이 포함되어 있습니다.

- [Elastic Beanstalk 학습 리스소](https://docs.aws.amazon.com/ko_kr/elasticbeanstalk/latest/dg/RelatedResources.html)
- 설명 영상: [AWS re:Invent 2017: Manage Your Applications with AWS Elastic Beanstalk (DEV305)](https://www.youtube.com/watch?v=NhsELnv28NU)  
- [자습서](https://docs.aws.amazon.com/ko_kr/elasticbeanstalk/latest/dg/tutorials.html)
- [Github 샘플코드](https://github.com/awsdocs/elastic-beanstalk-samples)

## 방법
Java 8 SDK 를 설치합니다. Java 컴파일러는 빌드 스크립트를 실행하기 위해 필요합니다.

* 리눅스에서 설치 참고
- RedHat 계열(Amazon Linux, CentOS 등) 사용시
```
sudo yum install -y java-1.8.0-openjdk-devel.x86_64
sudo /usr/sbin/alternatives --config java

# 기존 jdk 1.7 이 설치되어 있는 경우 삭제
sudo yum remove java-1.7.0-openjdk

# gradle 사용시 설치
sudo yum install gradle
```

- Ubuntu 계열 사용시
```
sudo apt-get install openjdk-8-jdk -y

# gradle 사용시 설치
sudo apt-get install gradle -y
```

* 윈도우 사용시 설치 참고
1. cmd 창 관리자 권한으로 실행
2. https://chocolatey.org/docs/installation 접속
3. 아래 실행 (Install with cmd.exe 로 검색)
	@"%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command "iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"
4. https://chocolatey.org/packages/awscli 접속
5. awscli 설치. 다음을 실행(업데이트는 나중에)

	choco install awscli
6. cmd 창을 관리자 권한으로 새로 실행 후 aws 라고 치면 명령어가 먹는걸 확인할 수 있음.
   (그래야 설치한 것이 AWS 그대로 먹힘)
7. 파이썬 설치 (CMD 창에서 실행)

	choco install python3
8. eb-CLI 설치

	pip install awsebcli --upgrade --user
9. 한번 더 CMD 새로 실행
10. eb 실행해서 정상 설치되었는지 확인

	eb
   
이외에 윈도우 환경에서 git-bash 설치는 https://blog.hanumoka.net/2018/05/16/git-20180516-git-install-on-windows/ 를 참고하도록 합시다.

### 프로젝트 다운로드, 빌드, 배포

프로젝트를 clone 합니다(SSH):

	~$ git clone git@github.com:awskrug/eb-tomcat-snakes.git

혹은 HTTPS 로:

	~$ git clone https://github.com/awskrug/eb-tomcat-snakes.git

빌드 스크립트(``build.sh``)를 실행하여 웹 앱과 WAR 파일을 생성합니다(OS X 나 리눅스에서):

	~$ cd eb-tomcat-snakes
	~/eb-tomcat-snakes$ ./build.sh

윈도우 사용자는 Git Bash 에서:

	~/eb-tomcat-snakes$ ./build-windows.sh

혹은 gradle 을 사용할 경우에는:

	~/eb-tomcat-snakes$ gradle build

**중요**
항상 build.sh 를 프로젝트 루트 디렉토리에서 실행하세요.

### 로컬에서 실행?

본 웹 애플리케이션을 로컬 환경에서 실행해 보려면, Tomcat 8 혹은 8.5 와 Postgresql 9.4 를 설치해야 합니다.

빌드 스크립트는 프로젝트를 컴파일하고 필요한 파일을 웹 아카이브(.WAR 파일)로 묶고, 로컬 환경 테스트를 위해 WAR 파일을 ``/Library/Tomcat`` 에 복사합니다. Tomcat을 다른 위치에 설치했다면, ``build.sh`` 에서 경로를 바꾸세요:

	if [ -d "/path/to/Tomcat/webapps" ]; then
	  cp ROOT.war /path/to/Tomcat/webapps

브라우저에서 [localhost:8080](http://localhost:8080/)를 열면, 로컬에서 실행되는 그 애플리케이션을 볼 수 있습니다.

AWS 관리 콘솔 이나 EB CLI 를 이용해서 컴파일한 WAR 파일을 실행할 수 있습니다. EB CLI 를 이용한 방법은 아래로 스크롤하여 보십시요.

##### AWS 관리 콘솔에서 배포하기

1. [Elastic Beanstalk Management Console](https://console.aws.amazon.com/elasticbeanstalk/home) 열기
2. *Create New Application* 선택
3. *Application Name* 에 **tomcat-snakes** 입력하고 *Create* 선택
4. 애플리케이션이 생성되면, *Create One now* 클릭
5. *Web server environment* 를 선택하고 *Select* 클릭
6. *Environment name" 에 *tomcat-snakes* 를 입력
7. *Domain* 에 고유한 주소(예를 들어 핸드폰번호) 입력하고 *Check availability* 클릭
   *is available* 이 나오지 않으면, 다른 고유한 값으로 다시 입력한다.
8. *is available* 이 보이면, *Platform* 에 *Preconfigured platform* 선택하고, *Choose a platform* 에서 *Tomcat* 을 선택
9. *Application code* 에서 *Upload your code* 선택하고 *Upload* 클릭
10. 프로젝트 최상위 폴더에서 *ROOT.war* 를 업로드하고 *Configure more options* 클릭
11. *Database* 항목에서 *Modify* 클릭
12. *Database settings* 에서 아래 RDS 설정을 적용하고 *Save* 클릭(다른 항목은 기본값으로 남겨둡니다)
    - *Engine*을 *postgres*
	- *Engine version* 에 *9.4.19*
	- *Instance class* 에 *db.t2.micro*
	- *Username* 에 *사용자명*
	- *Password* 에 *암호* ( 8자 이상)
	- *Retension* 에 *Delete*
13. *Create environment* 클릭

이 과정은 약 15분 정도 소요됩니다. 초기 환경 설정 중에 시간을 절약하려면 데이터베이스없이 환경을 생성한 다음, Configuration 페이지에서 추가할 수 있습니다. RDS DB 인스턴스를 시작하는데 약 10분 정도 걸립니다.

##### EB CLI 로 배포하기

EB CLI는 Python 2.7 이나 3.4(권장)과 패키지 관리자인 ``pip`` 이 필요합니다. EB CLI 설치에 관한 자세한 설명은 AWS Elastic Beanstalk 개발자 가이드에서 [Install the EB CLI](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3-install.html)를 참고하십시요.

EB CLI 설치하기:

	~$ pip install awsebcli

프로젝트 저장소 초기화하기:

혹시 awscli 가 설치되어 있다면, 아래 명령어로 표시되는 리전을 기억하고,

    ~/eb-tomcat-snakes$ aws configure get region
    ap-northeast-2

위에 나온 ``ap-northeast-2`` 를 아래처럼 ``eb init --region `` 추가 파라미터로 입력합니다.

    ~/eb-tomcat-snakes$ eb init --region ap-northeast-2

처음 수행한다면, 다음과 같이 진행합니다.

    ~/eb-tomcat-snakes$ eb init

``.elasticbeanstalk/config.yml``에 아래 내용 추가:

	deploy:
	  artifact: ROOT.war

RDS 데이터베이스를 포함한 환경 생성(진행 과정을 보여줍니다):

	~/eb-tomcat-snakes$ eb create tomcat-snakes -d -p tomcat-8.5-java-8 \
	--sample --single --timeout 20 -i t2.micro \
	-db.engine postgres -db.i db.t2.micro -db.user *사용자명* -db.pass *암호*

위 과정 진행 중에 timedout 발생 했다면, [Elastic Beanstalk Management Console](https://console.aws.amazon.com/elasticbeanstalk/home)에서 **Rebuild Environment**로 'tomcat-snakes' 환경을 재생성하시면 됩니다(좀 더 빨리 생성됩니다).

프로젝트 WAR 파일을 새 환경에 배포:

	~/eb-tomcat-snakes$ eb deploy --staged

**문제 해결**: 위에 명령어로 배포하는데 ``ERROR: InvalidParameterValueError - Environment named tomcat-snakes is in an invalid state for this operation. Must be Ready.``이러한 오류 메시지를 보셨다면, tomcat-snakes 의 EC2 인스턴스가 Running 상태가 아니어서 발생합니다. 너무 빨리 진행하신 것으로 조금 더 기다리시면 좋겠습니다. 환경 생성 과정에서 충분히 기다렸음에도(20분 이상) 오류 메시지가 발생하는 경우에는 timedout 문제입니다. [Elastic Beanstalk Management Console](https://console.aws.amazon.com/elasticbeanstalk/home)에서 **Rebuild Environment**로 'tomcat-snakes' 환경을 재생성하시면 됩니다(좀 더 빨리 생성됩니다).

브라우저에서 생성된 환경 열어 보기(맥 환경이 아니라면 권장하지 않습니다):

	~/eb-tomcat-snakes$ eb open

올려진 웹 사이트를 확인하시려면, 다음 명령어로 생성한 URL을 확인합니다.

	~/eb-tomcat-snakes$ eb status | grep CNAME
	CNAME: tomcat-snakes.kbjqhiyz98.ap-northeast-2.elasticbeanstalk.com

에서 위에 경우에는, ``tomcat-snakes.kbjqhiyz98.ap-northeast-2.elasticbeanstalk.com`` 입니다. 이 주소를 브라우저에서 열어 보면, 배포된 애플리케이션을 확인할 수 있습니다.

## 사이트 기능

이 앱은 심플 태그, 태그 파일과 Amazon RDS(Amazon Relational Database Service)의 외부 데이터베이스에서 호스팅되는 SQL 데이터베이스를 이용하는 간단한 J2EE 사이트입니다.

첫 페이지는 자바스크립트를 이용한 매우 기본적인 소개입니다. 모든 페이지는 헤더용 심플 태그와 태그 파일을 이용하고, 모바일 친화적인 화면을 위해 부트스트랩 CSS를 이용하였습니다.

**Browse Movies** 페이지는 심플 태그로 생성된 데이터베이스의 영화 목록을 보여줍니다.

**Add a Movie** 페이지는 사용자가 데이터베이스에 영화를 추가하는 입력 양식입니다. 영화 제목, IMDB 나 tt0118615와 같은 IMDB 영화 ID, 영화에서 뱀이 나오는지 여부를 나타내는 불린 값을 입력받습니다. movies model의 정규식으로 입력 값을 검증합니다.

**Search** 페이지는 전체 일치방식으로만 영화 이름을 검색할 수 있습니다.

## 데이터베이스 사용

이 애플리케이션은 Elastic Beanstalk 환경의 일부인 RDB DB 인스턴스나 Elastic Beanstalk 외부의 독립적인 RDS DB 인스턴스에 접속할 수 있습니다. 외부 DB 인스턴스에 연결하려면 접속 변수(RDS_HOST 등)를 Environment Properties에서 설정하거나, Amazon S3에 JSON 파일로 접속 정보(connecton string 등)을 저장하십시요.

후자의 방법은 ``src/.ebextensions/inactive``에 있는 설정 파일을 수정하여 S3 에서 접속정보를 다운로드할 수 있습니다. 환경설정 파일에 S3 버킷과 개체 주소를 가르키도록 수정하고 ``src/.ebextensions`` 폴더에 옮겨 놓으면, Elastic Beastalk 은 배포 중에 애플리케이션을 실행하는 EC2 인스턴스로 S3에서 접속정보를 다운로드합니다. 애플리케이션이 데이터베이스에 접속할 때에 환경변수 읽어오는 것보다 보다 우선적으로 이 값을 접속할 데이터 베이스를 지정하는 접속 문자열(connection string)로 사용합니다.

애플리케이션은 movies 테이블을 찾고, 없으면 새로 생성하고 소스에 포함된 JSON 파일에서 읽은 몇 개의 항목을 테이블에 넣어 둡니다.

### DB 인스턴스 관리

RDS DB 인스턴스를 관리하려면 먼저 SSH로 사용자 환경의 인스턴스에 접속합니다. 환경을 생성할 때에 SSH 키를 선택하지 않았다면, EB CLI 를 이용해서 지정할 수 있습니다. ``eb config`` 실행하여 ``settings`` 아래 ``EC2KeyName`` 필드에 키 이름을 입력합니다. SSH 키가 없다면, ``eb init -i``를 실행해서 프롬프트가 나오면 SSH 키를 생성하도록 합니다.

만약 movies 테이블을 지우거나, 테이블 초기화 코드의 변경사항을 시험하려면 사용자 환경 인스턴스에서 DB 인스턴스에 연결해서 관리자 명령을 실행합니다.

``eb ssh`` 실행해서 인스턴스에 접속합니다:

	~\eb-tomcat-snakes$ eb ssh

PostgresSQL 클라이언트 설치:

	[ec2-user@ip-555-55-55-555 ~]$ sudo yum install postgresql94

RDS DB 인스턴스 접속:

	[ec2-user@ip-555-55-55-555 ~]$ psql --dbname=ebdb --host=*DB_INSTANCE_HOSTNAME* --username=*DB_USERNAME*

``movies`` 테이블 보기:

	ebdb=> SELECT * FROM movies;

``movies`` 테이블 삭제(주의: ``movies`` 테이블을 삭제한다)

	ebdb=> DROP TABLE Movies;

psql 종료:

	ebdb=> \q

SSH 종료:

	[ec2-user@ip-555-55-55-555 ~]$ exit

## Log4j

이 애플리케이션은 Log4j 를 이용하여 ``snakes.log``라는 이름의 로그 파일을 생성한다. 이 프로젝트는 자세한 로그 필요시에, ``snakes.log``를 포함하도록 Elastic Beanstalk 를 설정하는 환경 설정 파일이 ``src/.ebextensions``에 포함되어 있습니다.

## 프로젝트 파일 내용

이 프로젝트는 다음과 같이 구성되어 있습니다(일부 파일은 제외):

	├── LICENSE             - License
	├── README.md           - 이 파일
	├── build.sh            - 빌드 스크립트
	└── src
	    ├── .ebextensions   - Elastic Beanstalk 환경 설정 파일
	    ├── 404.jsp         - 404 오류 JSP
	    ├── add.jsp         - 영화 추가 JSP
	    ├── default.jsp     - 홈 페이지 JSP
	    ├── movies.jsp      - 영화 목록 JSP
	    ├── search.jsp      - 영화 검색 JSP
	    ├── WEB-INF
	    │   ├── lib         - 라이브러리(.JAR 파일. JSP, Servlet API, Jasper, Log4J, PostgreSQL)
	    │   ├── tags        - 헤더 태그 파일
	    │   │   └── header.tag
	    │   ├── tlds        - ListMovies 심플 태그를 위한 테그 라이브러리 디스크립터
	    │   │   └── movies.tld
	    │   ├── log4j2.xml  - Log4J 환경설정 파일
	    │   └── web.xml     - 배포 설명 파일
	    ├── com
	    │   └── snakes
	    │       ├── model   - 모델 클래스
	    │       │   ├── Media.java
	    │       │   └── Movie.java
	    │       └── web     - 서블릿, 단순 태그 클래스들
	    │           ├── AddMovie.java
	    │           ├── ListMovies.java
	    │           └── SearchMovies.java
	    ├── css             - 스타일 시트
	    │   ├── movies.html - 스타일 변화를 테스트 하기 위한 HTML 페이지
	    │   └── snakes.css  - 사용자 스타일
	    ├── images          - 헤더와 프론트를 위한 무료 사용 이미지
	    └── js              - 부트스트랩과 시차 효과를 위한 자바 스크립트
  
### build.sh

빌드 스크립트는 자바 컴파일러(```javac```)로 각 클래스를 컴파일하고 컴파일된 클래스와 다른 파일들을 ```ROOT.war```라는 이름으로 웹 아카이브로 묶습니다. ```ROOT``` 는 이 앱이 tomcat 에서 서비스하는 사이트의 루트 경로에서 실행된다는 의미입니다.

***참고*** Elastic Beanstalk 환경에 WAR 파일을 배포하면, 배포 중에 풀어져서 파일명에 상관없이 최상위 로투 경로로서 실행됩니다. 여러 WAR 파일을 ZIP 파일로 묶어 배포하는 경우에 Elastic Beanstalk은 다른 경로의 앱들을 실행합니다.

WAR 파일에는 해당 앱의 실행에 필요한 파일들만 포함됩니다. 컴파일하지 않은 java 클래스나 ``.ebextensions/inactive`` 에 있는 설정 파일은 제외됩니다.

윈도우 버전 스크립트(``build-windows.sh``)도 포함되어 있다. classpath 인자들이 콜론(:)이 아니라 세미콜론(;)으로 구분되어 있다.

### 고급 기능

HTTPS와 로드발란서를 연동하는 다양한 방법에 대해서는 [HTTPS.md](src/.ebextensions/inactive/HTTPS.md)를 참고하시기 바랍니다.

### 종료

생성한 Beanstalk 환경과 애플리케이션을 삭제합니다. AWS 계정을 추가하였다면 IAM 에서 계정을 삭제합니다.

환경 삭제:

    ~/eb-tomcat-snakes$ eb terminate

애플리케이션 삭제:
1. [Elastic Beanstalk Management Console](https://console.aws.amazon.com/elasticbeanstalk/home?application/overview?applicationName=eb-tomcat-snakes)을 엽니다.
2. **eb-tomcat-snakes** 에 있는 **Actions** 에서 **Delete application** 선택

AWS 계정을 추가하였다면 [IAM](https://console.aws.amazon.com/iam/home?region=ap-northeast-2)에서 추가한 계정을 삭제합니다.
