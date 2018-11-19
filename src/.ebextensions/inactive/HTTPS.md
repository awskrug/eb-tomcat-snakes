# HTTPS 설정 이용하기
HTTPS를 이용하도록 설정하는 여러 방법이 있습니다. 단순한 방법은 사용 환경에서 로드 발란서를 HTTPS 접속을 끊고 HTTP로 벡엔드 인스턴스와 통신하도록 설정하는 것입니다. 이 방법을 시작하고, 인증서가 작동하는지 확인해 봅시다.

만약 [AWS Certificate Manager (ACM)](https://console.aws.amazon.com/acm)를 사용할 수 있으면, 이를 사용해서 무료로 가지고 있는 도메인용으로 매니지드 인증서를 생성할 수 있습니다. 인증서를 구입했거나 셀프 사인드 인증서라면, [IAM 에 인증서를 업로드](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/configuring-https-ssl-upload.html) 할 수 있습니다.

HTTPS를 종료 시키는 다른 방법은 고개 인증서와 개인키가 필요합니다. 안전한 S3 버킷에 개인키를 저장하고 [인스턴스 프로파일](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/concepts-roles.html))에서 그 버킷과 개체를 읽을 수 있는 권한이 있도록 합니다. 이를 진행하는 가장 쉬운 방법은 Elastic Beanstalk 저장소 버킷에 키를 넣고 [개발자 가이드](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/concepts-roles.html#concepts-roles-instance)의 샘플 인스턴스 프로파일을 사용하는 것입니다.

각각의 방법을 활성화하는 다른 조합으로 사용하도록 7개의 환경 파일([`src/.ebextensions/inactive`](https://github.com/awslabs/eb-tomcat-snakes/blob/master/src/.ebextensions/inactive/))을 제공합니다.

- `https-instance.config`
- `https-instance-single.config`
- `https-ssl.conf`
- `https-redirect.conf`
- `https-lbpassthrough.config`
- `https-lbreencrypt.config`
- `https-lbreencrypt-backendauth.config`
- `https-lbterminate.config`
- `https-lbterminate-listener.config`

각 환경 설정 파일은 만들거나 커스터마이징하기 위한 정보를 포함한 코멘트를 포함합니다. 환경 파일에 대한 추가 정보는 개발자 가이드[구성 파일을 사용하는 고급 환경 사용자 지정](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/ebextensions.html)항목을 찾아보십시요.

## HTTPS를 로드 발란서에서 끝내고, HTTP 로 통신하기(client-to-AWS 암호화)
이 방법은 [AWS Certificate Manager (ACM)](https://console.aws.amazon.com/acm)에서 생성한 매니지드 인증서나 IAM 에 업로드 된 인증서가 필요합니다.

### client-to-AWS HTTPS 활성화
1. `https-lbterminate.config`, `https-lbterminate-listener.config` 를 `src/.ebextensions` 에 복사하고, `http-healthcheckurl.config` 를 `src/.ebextensions/inactive` 에 이동시킵니다.
2. `https-lbterminate.config`에서 인증서 ARN를 수정합니다.
        - namespace:  aws:elb:loadbalancer
          option_name:  SSLCertificateId
          value:  arn:aws:acm:us-east-1:#############:certificate/############
3. `https-lbterminate.config`를 기본 혹은 사용자의 VPC ID 값을 갖도록 수정합니다.
        loadbalancersg:
          Type: AWS::EC2::SecurityGroup
          Properties:
            GroupDescription: load balancer security group
            VpcId: vpc-########
4. 빌드하고 배포합니다.

## 백엔드에서 HTTPS를 사용하여로드 밸런서에서 HTTPS 종료 (종단간 암호화)
이 방법은 매지드 인증서(로드 발란서용)와 백엔드 인스턴스용으로 서명하기 위한 서명된 공개 인증서와 개인키도 필요합니다. 이 방법은 좀 더 보안적이지만, 추가 설정을 해야 합니다. 프론트엔드를 위한 매지지드 인증서와 백엔드를 위한 셀프 사인 인증서를 사용할 수 있습니다.

### HTTPS 의 종단간 암호화 활성화
1. `https-lbterminate.config`, `https-lbreencrypt.config`, `https-instance.config`를 `src/.ebextensions` 복사하고, `http-healthcheckurl.config` 를 `src/.ebextensions/inactive`에 이동합니다. `src/.ebextensions/httpd/conf.d` 디렉토리를 생성하고, `https-ssl.conf` 파일을 `src/.ebextensions/httpd/conf.d` 에 복사합니다.
2. `https-lbterminate.config`에서 인증서의 ARN을 수정합니다.
	
        - namespace:  aws:elb:loadbalancer
          option_name:  SSLCertificateId
          value:  arn:aws:acm:us-east-1:#############:certificate/############
3. `https-lbterminate.config`에 기본 혹은 사용자의 VPC ID 값을 갖도록 수정합니다.

        loadbalancersg:
          Type: AWS::EC2::SecurityGroup
          Properties:
            GroupDescription: load balancer security group
            VpcId: vpc-########

4. `https-instance.config`의 버킷 이름을 갖도록 수정합니다. 

        AWS::CloudFormation::Authentication:
          S3Auth:
            type: "s3"
            buckets: ["elasticbeanstalk-#########-#############"]
5. `https-instance.config`의 개인키 주소를 수정합니다.

        /etc/pki/tls/certs/server.key:
          mode: "000400"
          owner: root
          group: root
          authentication: "S3Auth"
          source: https://s3-#########.amazonaws.com/elasticbeanstalk-#########-#############/server.key
6. `https-instance.config`에 공개 인증서 내용으로 수정합니다.:

        /etc/pki/tls/certs/server.crt:
          mode: "000400"
          owner: root
          group: root
          content: |
            -----BEGIN CERTIFICATE-----
            ################################################################
            ################################################################
7. HTTP 를 HTTPS 로 리다렉션할 경우에는, `https-redirect.conf`를 `src/.ebextensions/httpd/conf.d`에 복사합니다. 그리고 `https-redirect.conf`에서 실제 서버이름으로 HTTPS URL 리다렉션을 설정합니다. 

        ServerName www.###############.com
        Redirect permanent / https://www.################.com
8. 빌드하고 배포합니다.

### 벡엔드 인증
추가적으로, 강제로 특정한 인증서를 이용해서 벡엔드 EC2 인스턴스와 로드발란서를 인증할 수 있도록 설정할 수 있습니다. 

이 기능을 사용하려면 `https-lbreencrypt-backendauth.config`를 이용해야 합니다. 이 파일은 두 가지 정책을 정의합니다. 첫 번째 정책은 공개 인증서를 지정합니다:
```
  aws:elb:policies:backendkey:
    PublicKey: |
      -----BEGIN CERTIFICATE-----
      ################################################################
      ################################################################
```

해시 표시('#' 문자)를 인스턴스의 인증서 내용으로 변경합니다. 두번째 정책은 인스턴스의 포트 443으로 접속할 때에, 밸런서의 인증서만을 신뢰하도록 지시합니다:
```
    aws:elb:policies:backendencryption:
      PublicKeyPolicyNames: backendkey
      InstancePorts: 443
```
## 인스턴스에서 끝내기(단일 인스턴스 환경)
단일 인스턴스 환경에서는 인증서와 개인키가 필요하다. 이 방법은 인터넷에 직접 연뎔되고, ACM 에서 공짜 인증서를 사용할 수 없고, 환경을 스케일링하거나 롤링 업데이트를 할 수 없다. 이 방법은 테스트와 개발을 위해서 사용하자.

### 단일 인스턴스 HTTPS 활성화
1. `https-instance.config`, `https-instance-single.config` 파일을 `src/.ebextensions`에 복사하고, `http-healthcheckurl.config` 파일을  `src/.ebextensions/inactive` 으로 옮겨 놓자. `src/.ebextensions/httpd/conf.d` 디렉터리를 생성하고, `https-ssl.conf` 파일을 `src/.ebextensions/httpd/conf.d` 에 복사해 넣자.
2. `https-instance.config`에서 버킷명을 수정한다:

        AWS::CloudFormation::Authentication:
          S3Auth:
            type: "s3"
            buckets: ["elasticbeanstalk-#########-#############"]
3. `https-instance.config`에서 개인키 URL 을 수정한다:

        /etc/pki/tls/certs/server.key:
          mode: "000400"
          owner: root
          group: root
          authentication: "S3Auth"
          source: https://s3-#########.amazonaws.com/elasticbeanstalk-#########-#############/server.key
4. `https-instance.config` 에서 인증서 내용을 채워 넣는다.

        /etc/pki/tls/certs/server.crt:
          mode: "000400"
          owner: root
          group: root
          content: |
            -----BEGIN CERTIFICATE-----
            ################################################################
            ################################################################
5. HTTP 를 HTTPS 로 리다렉션할 경우, `https-redirect.conf`를 `src/.ebextensions/httpd/conf.d`에 복사하고, `https-redirect.conf` 에서 HTTPS URL 을 실제 서버를 가르키도록 수정한다.

        ServerName www.###############.com
        Redirect permanent / https://www.################.com
6. 빌드하고 배포합니다.

## 인스턴스에서 종료하기 (로드 발란서 통과)
이 방법은 인스턴스에서 종료하지만, HTTPS를 로드발란서에서 종료하지 않고, 암호화된 TCP 패킷 그대로 전달을 받는다. 이 방법은 로드발란서에서 request를 알 수 없기에, 라우팅이나 응답 측정값을 최적화할 수 없다.

### 로드발란서에서 HTTPS를 통과시키도록 활성화 하기
1. `https-instance.config`, `https-lbpassthrough.config`를 `src/.ebextensions` 옮기고, `http-healthcheckurl.config` 도 `src/.ebextensions/inactive` 옮긴다. `src/.ebextensions/httpd/conf.d` 디렉터리를 생성하고, `https-ssl.conf`파일을 `src/.ebextensions/httpd/conf.d`에 복사한다.
2. `https-instance.config`에 버킷 이름을 수정한다:

        AWS::CloudFormation::Authentication:
          S3Auth:
            type: "s3"
            buckets: ["elasticbeanstalk-#########-#############"]
3. `https-instance.config`에 개인키 주소를 넣는다:

        /etc/pki/tls/certs/server.key:
          mode: "000400"
          owner: root
          group: root
          authentication: "S3Auth"
          source: https://s3-#########.amazonaws.com/elasticbeanstalk-#########-#############/server.key
4. `https-instance.config`에 인증서 내용을 채워넣는다:

        /etc/pki/tls/certs/server.crt:
          mode: "000400"
          owner: root
          group: root
          content: |
            -----BEGIN CERTIFICATE-----
            ################################################################
            ################################################################
5. `https-lbpassthrough.config`에 VPC ID 를 입력한다:

        loadbalancersg:
          Type: AWS::EC2::SecurityGroup
          Properties:
            GroupDescription: load balancer security group
            VpcId: vpc-########
6. HTTP 를 HTTPS 로 리다렉션할 경우, `https-redirect.conf`를 `src/.ebextensions/httpd/conf.d`에 복사하고, `https-redirect.conf` 에서 HTTPS URL 을 실제 서버를 가르키도록 수정한다.

        ServerName www.###############.com
        Redirect permanent / https://www.################.com
7. 빌드하고 배포합니다.
