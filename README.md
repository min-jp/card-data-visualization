# 🚀 VirtualBox에서 ElasticSearch와 Kibana 설치 가이드

## 📝 개요

이 문서는 **VirtualBox**에서 두 개의 가상머신에 각각 **ElasticSearch**와 **Kibana**를 설치하는 과정을 다룹니다.

**한 가상머신**에 ElasticSearch와 Kibana를 모두 설치할 경우 **리소스 문제**가 발생할 수 있어, 이를 해결하기 위해 **두 개의 가상머신에 분리 설치**하는 방법을 설명합니다.

설치 과정은 다음과 같은 주요 단계로 구성됩니다.

1. VirtualBox에서 **가상머신 생성** 및 **네트워크 설정**
2. **ElasticSearch**와 **Kibana**의 **설치** 및 **설정**
3. **포트포워딩**을 통한 접속 설정

## 🖥️ VirtualBox 여러 가상머신 실행

VirtualBox에서 두 개의 가상머신을 생성하고 우분투를 설치한 뒤 기본 설정을 진행합니다.

### ⚙️ 기본 설정

각 가상머신에서 다음 명령어로 초기 설정을 완료합니다.

```bash
sudo apt update  # 패키지 목록 업데이트
sudo apt upgrade  # 패키지 업데이트
sudo apt install openssh-server  # open-ssh 설치
sudo timedatectl set-timezone <타임존>  # 타임존 설정
```

### 💻 각 가상머신 정보

**🖥️ 가상머신 1**

- **OS**: Ubuntu 24.04.1 LTS (amd64)
- **역할**: ElasticSearch 설치

**🖥️ 가상머신 2**

- **OS**: Ubuntu 24.04.1 LTS (amd64)
- **역할**: Kibana 설치

## 🌐 네트워크 설정

두 가상머신 간 통신을 위해 **NAT 네트워크**를 설정합니다. NAT 네트워크를 사용하면 가상머신이 **동일 네트워크를 공유**할 수 있습니다.

### 🛠️ NAT 네트워크 생성

1. VirtualBox 메뉴에서 **도구 ➔ 네트워크**로 이동합니다.
    
    ![image](https://github.com/user-attachments/assets/7912bde2-ce97-41e6-8f1c-62ffc06b736f)
    
2. **NAT 네트워크 ➔ 만들기**를 클릭하여 새 네트워크를 생성합니다.
    
    ![image 1](https://github.com/user-attachments/assets/b5c914f6-45e6-4def-b385-c988318a4ba2)


### 🔗 각 가상머신의 네트워크 설정 변경

1. 각 가상머신의 설정 메뉴에서 **네트워크 ➔ 네트워크 어댑터**로 이동합니다.
2. **다음에 연결됨**을 "NAT 네트워크"로 변경합니다.
3. 생성한 NAT 네트워크를 선택합니다.
    
    ![image 2](https://github.com/user-attachments/assets/0e2d948c-77eb-4dd8-9c56-95382b32d029)
    

### 🔧 포트포워딩 설정

가상머신에 외부에서 접근하기 위해 포트포워딩을 설정합니다. 설정 방법은 다음과 같습니다.

1. **NAT 네트워크 설정 ➔ 포트포워딩**으로 이동합니다.
2. 필요한 포트를 추가합니다.
    
    ![image 3](https://github.com/user-attachments/assets/0e656940-8291-4797-af82-e774bc933e63)
    

- **포트 포워딩 목록**
    
    
    | **이름** | **프로토콜** | **호스트 IP** | **호스트 포트** | **게스트 IP** | **게스트 포트** |
    | --- | --- | --- | --- | --- | --- |
    | **server1-elasticsearch** | TCP | 127.0.0.1 | 9200 | \<server1 IP\> | 9200 |
    | **server2-kibana** | TCP | 127.0.0.1 | 5601 | \<server2 IP\> | 5601 |
    | **server1-ssh** | TCP | 127.0.0.1 | 2222 | \<server1 IP\> | 22 |
    | **server2-ssh** | TCP | 127.0.0.1 | 2223 | \<server2 IP\> | 22 |

## 📦 ElasticSearch & Kibana 설치

### 📂 ElasticSearch 설치 (가상머신 1)

- **🗑️ 기존 버전 삭제 (필요 시)**
    
    기존에 설치된 ElasticSearch를 삭제하려면 다음 명령어를 실행합니다.
    
    **기존버전 삭제**
    
    ```bash
    sudo apt purge elasticsearch  # 설정파일 포함하여 elasticsearch 삭제
    sudo apt autoremove  # 불필요한 패키지 삭제
    sudo rm -rf /var/lib/elasticsearch/  # elasticsearch 남은 디렉토리 삭제
    ```
    
    **service 재로딩**
    
    ```bash
    systemctl daemon-reload  # service 재로딩
    systemctl reset-failed
    ```


### ⬇️ ElasticSearch 7.11 설치

1. **설치 파일 다운로드**
    
    ```bash
    wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.11.1-amd64.deb
    wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.11.1-amd64.deb.sha512
    shasum -a 512 -c elasticsearch-7.11.1-amd64.deb.sha512  # 파일 무결성 확인
    ```
    
2. **설치**
    
    ```bash
    sudo dpkg -i elasticsearch-7.11.1-amd64.deb
    ```
    
3. **서비스 시작**
    
    ```bash
    sudo systemctl daemon-reload
    sudo systemctl enable elasticsearch.service
    sudo systemctl start elasticsearch.service
    ```
    

### ⚙️ 설정 변경

1. **config파일 수정**
    
    `/etc/elasticsearch/elasticsearch.yml` 파일을 수정합니다.
    
    ```yaml
    discovery.type: single-node
    network.host: 0.0.0.0
    ```
    
    - `discovery.type: single-node` - 단일 노드 환경 설정
    - `network.host: 0.0.0.0` - 모든 IP에서 접속 허용
2. **설정 적용**
    
    ```bash
    sudo systemctl restart elasticsearch.service
    ```
    

### 📂 Kibana 설치 (가상머신 2)

### ⬇️ Kibana 7.11 설치

1. **설치 파일 다운로드**
    
    ```bash
    wget https://artifacts.elastic.co/downloads/kibana/kibana-7.11.1-amd64.deb
    shasum -a 512 kibana-7.11.1-amd64.deb  # 파일 무결성 확인
    ```
    
2. **설치**
    
    ```bash
    sudo dpkg -i kibana-7.11.1-amd64.deb
    ```
    
3. **서비스 시작**
    
    ```bash
    sudo systemctl daemon-reload
    sudo systemctl enable kibana.service
    sudo systemctl start kibana.service
    ```
    

### ⚙️ 설정 변경

1. **config파일 수정**
    
    `/etc/kibana/kibana.yml` 파일을 수정합니다.
    
    ```yaml
    server.host: "0.0.0.0"
    elasticsearch.hosts: ["http://<server1 IP>:9200"]
    ```
    
    - `server.host: 0.0.0.0` - 모든 IP에서 접속 허용
    - `elasticsearch.hosts: ["http://<서버ip>:9200"]` - ElasticSearch의 IP 주소 (1번 가상머신 IP) 설정
2. **설정 적용**
    
    ```bash
    sudo systemctl restart kibana.service
    ```
    

## 🔍 ElasticSearch & Kibana 접속 확인

### 🌐 ElasticSearch 확인

![image 4](https://github.com/user-attachments/assets/dc886097-c328-4164-aadb-a397d9077f7f)

브라우저에서 [http://localhost:9200/](http://localhost:9200/)에 접속하여 ElasticSearch가 정상적으로 실행되는지 확인합니다.

### 🌐 Kibana 확인

![image 5](https://github.com/user-attachments/assets/8e7c7ffd-0938-4289-9c60-fe1c36607f18)

브라우저에서 [http://localhost:5601/](http://localhost:5601/)에 접속하여 Kibana가 정상적으로 실행되는지 확인합니다.

## 📚 참고 문서

- [ElasticSearch 7.11 설치 가이드](https://www.elastic.co/guide/en/elasticsearch/reference/7.11/deb.html)
- [Kibana 7.11 설치 가이드](https://www.elastic.co/guide/en/kibana/7.11/deb.html)
