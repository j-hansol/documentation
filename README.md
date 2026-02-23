# 부과대 DX-Spark 온프레미스 서버 설정

## 시스템 정보
| 항목 | 내용 |
|------|------|
| 운영체제 | Rocy Linux 10 |
| 데이터베이스 | PostgreSQL 16 |

## 1. 네트워크 설정
### 1.1. 네트워크 어답터 이름 확인
```
nmcli device
```
아래와 같이 ```ethernet``` 타입 확인
```
DEVICE  TYPE      STATE                   CONNECTION
ens3    ethernet  connected               cloud-init ens3
lo      loopback  connected (externally)  lo
```

### 1.2. IP 주소 및 서브넷 마스크 설정
```
# nmctl con mod <어답터 이름> ipv4.addresses [서버넷 마스크]
nmcli con mod eth0 ipv4.addresses [192.168.1.100/24]
```

### 1.3. 게이트웨이 주소 설정
```
# nmctl con mod <어답터 이름> ipv4.addresses [게이트웨이 IP 주소]
nmcli con mod eth0 ipv4.gateway [192.168.1.1]
```

### 1.4. 기본 네임서버 설정
```
# nmcli con mod <어답터 이름> ipv4.dns "네임서버 IP 컴마로 분리하여 기록"
nmcli con mod eth0 ipv4.dns "8.8.8.8,8.8.4.4"
```

### 1.5. 수동 설정 모드로 변경
```
nmcli con mod eth0 ipv4.method manual
```

### 1.6. 설정 적용
```
nmcli con up eth0
```

### 1.7. 설정 확인
```
ip addr show eth0
# or
nmcli device show eth0
```

## 2. PostgreSQL 설치 절차
### 레포지토리 설치 및 모듈 비활성화
```
dnf install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-9-x86_64/pgdg-redhat-repo-latest.noarch.rpm
dnf -qy module disable postgresql
```

### 2.1. PostgreSQL 서버 설치
```
dnf install -y postgresql16-server
```

### 2.2. 데이터베이스 폴더 초기화
서비스 실행 전에 반드시 필요
```
/usr/pgsql-16/bin/postgresql-16-setup initdb
```

### 2.3. 서비스 실행 및 부팅 시 자동 실행
```
systemctl start postgresql-16
systemctl enable postgresql-16
```

### 2.4. 확인 및 접속
```
sudo -i -u postgres psql
```

### 2.5. 외부에서 접속 가능하도록 수신 설정
```/var/lib/pgsql/16/data/postgresql.conf``` 파일 편집
```
vi /var/lib/pgsql/16/data/postgresql.conf
```
주석을 풀어 아래 내용으로 수정
```
listen_addresses = '*'
```

### 2.6. 방화벽 개방
방화벽이 설치된 경우 아래 명령으로 개방 필요
```
firewall-cmd --permanent --add-service=postgresql
firewall-cmd --reload
```

### 2.7. pgvector 설치
```
dnf install -y pgvector_16
```

### 2.8. 서비스 재실행
```
systemctl restart postgresql-16
```

## 3. 데이터베이스 생성
아래의 작업으로 관리자 계정으로 데이터베이스에 접속
```
sudo -i -u postgres psql
```

### 3.1. 사용자 생성
```
CREATE USER dxspark WITH PASSWORD 'BistDXSpark@2026';
```

### 3.2. dxspark이 오너인 데이터베이스 생성
```
CREATE DATABASE dxspark OWNER dxspark;
```

### 3.3. 권한 설정
```
GRANT ALL PRIVILEGES ON DATABASE dxspark TO dxspark;
```

### 3.4. 데이터베이스 선택(이동)
```
\c dxspark
```

### 3.5. 스키마 권한 추가
```
GRANT ALL ON SCHEMA public TO dxspark;
```

### 3.6. Vector 칼럼 확장 활성화
데이터베이스 선택 후 아래 SQL 실행 필요
```
CREATE EXTENSION IF NOT EXISTS vector;
```