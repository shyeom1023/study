# AWS MQTT 쿠버네티스 설치 과정

CPMS와 유사하게 Bastion 서버를 통한 쿠버네티스 제어를 하여, chatGPT 한테 큰 틀에 대해서 물어본 뒤 구성을 진행하였습니다.



![image-20241114131542180](assets/image-20241114131542180.png)

이와같은 구조를 생각하고 있고 이것에 대한 gpt의 답변은 아래와 같았다.



## chatGPT 답변1



AWS에서 쿠버네티스를 구성하여 여러 서버를 배포하려는 경우, 네임스페이스와 구조를 설정하는 방법은 애플리케이션의 독립성과 관리의 용이성을 고려하여 결정하는 것이 중요합니다. 아래와 같이 네임스페이스와 구조를 구성하는 방법을 제안드립니다.

### 1. 네임스페이스 구성

네임스페이스는 리소스의 논리적 분리와 관리 효율성을 위해 사용하는 것이 좋습니다. 다음과 같이 구성할 수 있습니다:

- **네임스페이스 1: mqtt-broker**
  - MQTT 서버는 다른 애플리케이션과 독립적일 수 있어, 별도의 네임스페이스에서 운영하면 좋습니다. 이 네임스페이스에 배포하면 브로커 서버 관련 리소스를 명확하게 분리하고 관리할 수 있습니다.
- **네임스페이스 2: app-server**
  - 애플리케이션 서버는 데이터베이스와의 연동이나 다른 마이크로서비스와의 통신이 빈번하다면, 관련 네임스페이스와 잘 연결될 수 있도록 별도의 네임스페이스로 구성합니다.
- **네임스페이스 3: monitoring**
  - InfluxDB와 Grafana를 함께 배포하여 모니터링 환경을 구성하는 네임스페이스입니다. 이 네임스페이스에 InfluxDB와 Grafana를 배포하면, 데이터 스토리지와 시각화 도구를 한 곳에 관리할 수 있어 편리합니다.

### 2. 클러스터 구조 및 네트워킹

- **클러스터 내 네트워킹**: 각 네임스페이스 간 네트워크 접근은 필요에 따라 제한하거나, 네임스페이스 내에서만 접근 가능하게 설정할 수 있습니다.
- **Ingress Controller**: 외부 접근이 필요한 서버에 대해 Ingress Controller를 사용하여 경로 기반 라우팅이나 인증을 추가할 수 있습니다.
- **Persistent Volumes**: InfluxDB는 데이터가 손실되지 않도록 스토리지가 중요하므로 EBS를 Persistent Volume으로 사용하여 안정성을 높일 수 있습니다.

### 3. 권장 구성 예시

- 네임스페이스: `mqtt-broker`, `app-server`, `monitoring`
- Ingress 설정: 외부에 노출할 애플리케이션 서버와 Grafana를 Ingress로 설정
- 각 네임스페이스에 필요한 역할 기반 접근 제어(RBAC) 및 보안 설정 적용

이렇게 구성하면 리소스를 논리적으로 분리하고 관리하기 수월하며, 네임스페이스 별로 접근 권한을 설정하여 보안성을 높일 수 있습니다.



## chatGPT 답변2

AWS에서 쿠버네티스를 관리하기 위해 **Bastion 서버**를 설정하는 것은 보안과 접근 관리 측면에서 매우 유용합니다. Bastion 서버를 통해 AWS 외부에서 쿠버네티스 클러스터에 안전하게 접근하고 관리할 수 있습니다. Bastion 서버를 구축하고 활용하는 방법을 안내드리겠습니다.

### Bastion 서버 설정 및 접근 절차

1. **Bastion 서버 생성**

   - **EC2 인스턴스**를 생성하고 Bastion 서버 역할을 부여합니다. 이 인스턴스는 보통 퍼블릭 서브넷에 위치시켜 외부에서 SSH 접속이 가능하도록 설정합니다.
   - 인스턴스 유형은 트래픽이 많지 않다면 `t3.micro`와 같은 작은 유형을 사용해도 충분합니다.

2. **보안 그룹 설정**

   - Bastion 서버의 보안 그룹을 통해 **포트 22**(SSH)를 허용하고, 접근 가능한 IP 대역을 제한하여 보안을 강화합니다. (예: 특정 IP 주소만 허용)
   - Bastion 서버가 EKS 클러스터와 통신할 수 있도록 **포트 443**을 포함한 필요한 포트를 허용합니다.

3. **IAM 역할 부여**

   - Bastion 서버에 쿠버네티스 클러스터 접근 권한이 있는 IAM 역할을 부여합니다.
   - 이 IAM 역할에는 클러스터를 제어할 수 있는 적절한 권한이 포함되어 있어야 하며, `eks:DescribeCluster`, `eks:GetToken` 등의 권한을 포함해야 합니다.

4. **kubectl 및 aws-cli 설치**

   - Bastion 서버에 접속한 후 `kubectl`과 `aws-cli`를 설치합니다.
   - `aws-cli`를 통해 AWS 클러스터에 접근할 수 있도록 `aws configure`를 통해 AWS 계정 정보를 설정하거나, 부여된 IAM 역할로 접근합니다.

5. **클러스터 접근 설정**

   - AWS CLI 명령어로 클러스터 인증을 받습니다.

     ```
     bash
     
     
     코드 복사
     aws eks update-kubeconfig --name <EKS_Cluster_Name> --region <Region>
     ```

   - 위 명령어를 통해 `kubeconfig` 파일이 업데이트되면, Bastion 서버에서 `kubectl` 명령어를 사용하여 쿠버네티스 클러스터를 제어할 수 있게 됩니다.

6. **로컬에서 Bastion 서버를 통한 클러스터 접근 (SSH 터널링)**

   - 로컬에서 직접 접근하는 대신, Bastion 서버를 거쳐 클러스터에 접근하고자 할 때 SSH 터널링을 활용할 수 있습니다.

     ```
     bash
     
     
     코드 복사
     ssh -i <Your_Key.pem> -L 6443:<EKS_API_Server_Endpoint>:443 ec2-user@<Bastion_Server_IP>
     ```

   - 터널링이 설정되면 `kubectl` 명령어로 로컬에서도 클러스터를 관리할 수 있습니다.

### Bastion 서버의 주요 활용

- **쿠버네티스 클러스터 관리**: Bastion 서버를 통해서만 접근이 가능하도록 설정하면 보안을 강화할 수 있습니다.
- **CI/CD 연결**: Bastion 서버를 통해 특정 CI/CD 파이프라인의 클러스터 접근을 제한하거나, 스크립트를 통해 클러스터 제어가 가능합니다.
- **추가 보안 옵션**: Bastion 서버를 AWS SSM (Session Manager)으로 연결하면 SSH 키 없이도 안전하게 접속할 수 있습니다.

이와 같이 설정하면 AWS EKS 클러스터에 외부 접근을 Bastion 서버를 통해 제어할 수 있어 보안이 강화됩니다.



## 내가 내린 결론

일단 가장 먼저 해야할건 bastion 서버를 구축하는 것이다.



bastion 서버를 통해서 쿠버네티스를 제어하고 보안도 강화 하려는 것이 목적이다.



### Bastion 서버의 역할

Bastion 서버는 네트워크 보안을 강화하기 위해 사용되는 **중계 서버** 또는 **게이트웨이 서버**입니다. 외부 네트워크에서 내부 시스템에 직접 접근하지 않고, Bastion 서버를 거쳐서만 내부 네트워크에 접근하도록 함으로써 **보안을 강화**할 수 있습니다.

#### Bastion 서버의 주요 역할과 특징

1. **보안 강화**
   - Bastion 서버를 통해서만 내부 네트워크에 접근하도록 제한함으로써, 외부 네트워크와 내부 네트워크 간의 경계에서 보안 계층 역할을 합니다.
   - 외부에서는 직접적으로 데이터베이스나 애플리케이션 서버에 접근할 수 없고, Bastion 서버를 경유해야 하기 때문에 보안이 강화됩니다.
2. **접근 제어**
   - Bastion 서버는 보통 **SSH**(Linux) 또는 **RDP**(Windows)로 접근 가능하게 설정됩니다.
   - 보안 그룹이나 방화벽 설정을 통해 특정 IP 주소나 포트에 대해서만 접근을 허용하여, 허가된 사용자만 접근할 수 있게 합니다.
3. **감사 및 모니터링 용이**
   - Bastion 서버는 보통 접근 이력을 관리하여 어떤 사용자가 내부 서버에 접속했는지 기록할 수 있습니다.
   - 접근을 Bastion 서버를 통해 제어하면 감사 로그를 통해 내부 시스템에 대한 접속 및 액션을 모니터링할 수 있습니다.
4. **AWS에서의 Bastion 서버 활용**
   - AWS에서는 Bastion 서버를 보통 **퍼블릭 서브넷**에 배치하고, 이를 통해서만 **프라이빗 서브넷**에 위치한 리소스에 접근하도록 구성합니다.
   - AWS Systems Manager의 Session Manager를 활용해 SSH 키 없이 Bastion 서버에 접근하도록 설정하면 보안을 더욱 강화할 수 있습니다.

#### Bastion 서버의 활용 예시

예를 들어, AWS 환경에서 프라이빗 서브넷에 위치한 데이터베이스가 있다면, 외부에서 데이터베이스에 직접 접근할 수 없습니다. 이 경우, Bastion 서버에 SSH 접속 후, 이 서버에서 데이터베이스에 접근하게 하면 **직접적인 외부 접근을 차단**하면서도 **인증된 접근**만 가능하게 설정할 수 있습니다.

요약하면, Bastion 서버는 외부 네트워크와 내부 리소스 사이의 보안 경계 역할을 하며, **접근 제어 및 모니터링**이 필요한 환경에서 매우 유용하게 사용되는 서버입니다.



# AWS에서 Bastion 구축 하는 법

https://youtu.be/uGhCjaExw0M?si=L7S418mQCznhZnNE

해당 자료를 토대로 구축을 시작 하였습니다.





### 1. EKS Cluster

- 클러스터 하드웨어 아키텍처

  - Default VPC : Bastion 서버

    - Ubuntu Linux OS, AWS CLI, eksctl, kubectl
    - Docker Container build 및 컨테이너 push

  - EKS Cluster

    <img src="assets/image-20241114135543893.png" alt="image-20241114135543893" style="zoom:150%;" />

### 2. EKS 클러스터 관리 서버(Bastion) 생성 및 환경 구성



#### Defailt VPC에 kubectl, eksctl 실행을 위한 Bastion 생성

- 키페어 생성
  - 이름: demo-key
  - 프라이빗 키 파일 형식 : .pem <== default 값 확인해서 .ppk 만들어지지 않도록 주의!
- 보안 그룹 생성
  - 이름 : demo-bastion-sg
  - VPC : default VPC
  - 인바운드 규칙 : 내IP에서 ssh로의 접근 허용
- 인스턴스 생성
  - 이름 : demo-bastion
  - AMI : Ubuntu Server 24.04 LTS
  - 인스턴스 유형 : 
    - t2.medium => 무료가 아니여서 아래 내용 적용
    - t2.micro
  - 키페어 : demo-key
  - 네트워크
    -  VPC: Default VPC
    - 서브넷 : ap-northeast=2a
    - 보안그룹 : 기존 보안 그룹 선택 => demo-bastion-sg
  - 스토리지 : 8G, gp3
  - 인스턴스 생성 후 ubuntu 유저로 로그인

------

#### Bastion 서버를 클러스터 관리 서버로 구성

- AWS 프로 파일 등록

  - AWS CLI 연결을 위한 Access 키 생성

    - IAM => [보안 자격 증명] - [액세스키 만들기] - Access Key, Secret Key 확인 및 저장

    - Bastion Host(ubuntu Linux)에 AWS CLI 관리툴인 AWS 설치

    - 참고: https://docs.aws.amazon.com/ko_kr/cli/latest/userguide/getting-started-install.html

      - ```bash
        $ sudo apt install unzip
        
        curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
        unzip awscliv2.zip
        sudo ./aws/install
        
        $ aws --version
        aws-cli/2.21.0 Python/3.12.6 Linux/6.8.0-1016-aws exe/x86_64.ubuntu.24
        ```

      - ```bash
        # config 설정
        
        $ aws configure
        AWS Access Key ID [None]: # IAM key
        AWS Secret Access Key [None]: # IAM Secret Key
        Default region name [None]: ap-northeast-2
        Default output format [None]: json
        
        # config 정보 확인
        $ aws sts get-caller-identity
        {
            "UserId": "699475938633",
            "Account": "699475938633",
            "Arn": "arn:aws:iam::699475938633:root"
        }
        ```

        

- kubectl 설치

  - **Kubectl 설치 명령으로 EKS 권한을 할당**

    - 설치 매뉴열 : https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/install-kubectl.html

    - Kubernetes `1.31`

      ```bash
      curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.31.0/2024-09-12/bin/linux/amd64/kubectl
      ```

    - 바이너리에 실행 권한을 적용합니다.

      ```bash
      chmod +x ./kubectl
      ```

    - 바이너리를 `PATH`의 폴더에 복사합니다. `kubectl` 버전이 이미 설치된 경우 `$HOME/bin/kubectl`을 생성하고 `$HOME/bin`이 `$PATH`로 시작하도록 해야 합니다.

      ```bash
      mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$HOME/bin:$PATH
      ```

    - (선택 사항) 셸 초기화 파일에 `$HOME/bin` 경로를 추가하면 셸을 열 때 구성됩니다.

      ###### 참고

      이 단계에는 Bash 셸을 사용한다고 가정합니다. 다른 셸을 사용하는 경우, 특정 셸 초기화 파일을 사용하도록 명령을 변경하십시오.

      ```bash
      echo 'export PATH=$HOME/bin:$PATH' >> ~/.bashrc
      ```

    - `kubectl`을 설치한 이후 버전을 확인할 수 있습니다.

      ```bash
      $ kubectl version --client
      Client Version: v1.31.0-eks-a737599
      Kustomize Version: v5.4.2
      ```

- eksctl 명령어 설치 후 사용

  - 설치 메뉴얼 : https://eksctl.io/installation/

  - 설치 내용

    - 최신 릴리스를 다운로드하려면 다음을 실행하세요.

      ```bash
      # for ARM systems, set ARCH to: `arm64`, `armv6` or `armv7`
      ARCH=amd64
      PLATFORM=$(uname -s)_$ARCH
      
      curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"
      
      # (Optional) Verify checksum
      curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_checksums.txt" | grep $PLATFORM | sha256sum --check
      
      tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz
      
      sudo mv /tmp/eksctl /usr/local/bin
      
      $ eksctl version
      0.194.0
      ```

      













































