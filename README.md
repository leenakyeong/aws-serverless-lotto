# AWS Serverless Lotto Architecture

주기적으로 실행되는 로또 데이터 수집 서비스를, 상시 실행 EC2 대신 **Event-Driven Serverless Architecture**로 재구성한 프로젝트다.  
불필요한 고정비를 줄이고, 운영 포인트를 단순화하는 데 초점을 두었다.

---

## 1. Project Background

기존 구조는 주 1회 수준의 작업을 위해 서버를 계속 켜 두어야 했고, 작은 서비스임에도 운영 확인 포인트가 계속 발생했다.  
이 프로젝트에서는 이런 비효율을 줄이기 위해, 필요한 시점에만 실행되는 서버리스 구조로 전환했다.

### 목표
- **Fixed Cost Reduction**: 상시 실행 서버를 제거하고 사용량 기반 구조로 전환
- **Operational Overhead Reduction**: 관리형 서비스를 중심으로 운영 포인트 축소
- **Secure Delivery**: S3 직접 공개를 막고 CloudFront 기반으로만 배포
- **Stable Static Serving**: 정적 웹과 데이터 제공 경로를 분리해 단순하고 예측 가능한 구조 구성

---

## 2. Architecture Overview

### Event Pipeline
- **EventBridge**: 정해진 시간에 수집 작업 트리거
- **Lambda**: 로또 API 호출 및 데이터 가공
- **S3 (data bucket)**: 회차별 이력 데이터와 최신 데이터 저장

### Web Delivery
- **S3 (web bucket)**: 정적 웹 리소스 저장
- **CloudFront**: 웹 페이지 및 JSON 데이터 배포
- 기본 경로는 web bucket, `/data/*` 경로는 data bucket으로 분리 구성

### Security
- **OAC (Origin Access Control)**: CloudFront를 통해서만 S3 접근 허용
- **AWS WAF**: Rate Limit 및 국가 제한 적용
- **ACM / Route 53**: HTTPS 및 커스텀 도메인 연결

### Monitoring
- **CloudWatch**: Lambda 실행 로그 및 알람 기반 감시
- **SNS**: 이상 상황 알림 전송

---

## 3. Tech Stack

| Category | Tech / AWS Service |
| :--- | :--- |
| **Compute** | AWS Lambda |
| **Storage** | Amazon S3 |
| **Eventing** | Amazon EventBridge |
| **Network & Delivery** | Amazon CloudFront, Route 53, ACM |
| **Security** | AWS WAF, OAC, IAM |
| **Monitoring** | Amazon CloudWatch, SNS |
| **Frontend** | HTML, CSS, Vanilla JavaScript |

---

## 4. Key Engineering Decisions

### 4-1. Always-On Server 대신 Event-Driven 구조 선택
이 서비스는 실시간 상시 처리보다 주기적 실행에 가까운 성격이므로, EC2를 계속 유지하는 방식보다 EventBridge + Lambda 조합이 더 적합하다고 판단했다.

### 4-2. Web Bucket / Data Bucket 분리
정적 웹 파일과 회차 데이터의 성격이 다르기 때문에 버킷을 분리했다.  
이렇게 하면 배포 경로, 캐시 정책, 접근 제어를 목적에 맞게 나눌 수 있다.

### 4-3. latest.json과 round 이력 데이터의 캐시 성격 분리
최신 회차 데이터는 즉시 반영이 중요하고, 회차별 이력 데이터는 변경 가능성이 낮다.  
그래서 `latest.json`은 캐시를 최소화하고, `round/*.json`은 캐시를 유지하는 방향으로 구성했다.

### 4-4. S3 직접 공개 대신 CloudFront + OAC 적용
S3를 직접 공개하지 않고 CloudFront만 Origin에 접근하도록 구성해, 정적 콘텐츠 배포 경로를 단순화하면서 공개 범위를 통제했다.

### 4-5. 최소 비용 범위 내 보안 적용
과도한 보안 비용을 추가하기보다, 프로젝트 성격에 맞는 최소 수준의 보호를 적용했다.  
현재는 CloudFront 앞단에 WAF를 두고, Rate Limit과 국가 제한 규칙을 운영 중이다.

---

## 5. Current Status

## 완료
- [x] EventBridge → Lambda → S3 데이터 수집 파이프라인 구성
- [x] 회차별 이력 데이터 및 최신 데이터 저장 구조 구성
- [x] web/data 버킷 분리
- [x] CloudFront 단일 배포에서 경로 기반 라우팅 구성
- [x] OAC를 통한 S3 비공개 접근 구성
- [x] 정적 웹에서 최신 회차 조회 및 회차별 조회 기능 구현
- [x] AWS WAF Rate Limit / 국가 제한 적용
- [x] `svc.nagom.io` 서브도메인 기반 서비스 연결

## 진행 예정
- [ ] **Infrastructure as Code**: Terraform 기반 전체 리소스 코드화
- [ ] **Observability Hardening**: CloudWatch 알람 및 운영 지표 정리
- [ ] **Backfill Automation**: 누락 회차 자동 적재 및 초기 적재 보완
- [ ] **Architecture Expansion**: 멀티클라우드/GCP 기반 분석 확장 검토

## 다음 실습 트랙 (EC2 운영형 확장)
- [ ] **EC2 운영형 실습 환경 구성**: Single AZ + On-Demand + Ubuntu + `t4g.small` 기준 검토
- [ ] **인스턴스 선정 근거 정리**: T 계열 선택 이유, Graviton2(ARM) 기반 가성비, `x86_64` / `arm64` 호환성 검토
- [ ] **CLI 기본기 학습**: 파일/권한/프로세스/포트/로그/서비스 관리 명령 익히기
- [ ] **리눅스 표준 디렉토리 구조 학습**: `/etc`, `/var/log`, `/opt`, `/srv` 기준으로 설정/로그/서비스 경로 구분
- [ ] **운영 스택 구성**: EC2 + Docker + Nginx Reverse Proxy + systemd 자동기동
- [ ] **systemd 운영 실습**: `/etc/systemd/system` 유닛 파일 직접 관리, Restart 정책 적용
- [ ] **Nginx 운영형 설정 반영**: `proxy_set_header`, `client_max_body_size` 등 실제 운영 설정 포함
- [ ] **운영 증빙 정리**: 장애 재시작 테스트, 로그 확인, 포트 확인 등 운영 기록 남기기
- [ ] **IaC 확장**: 위 운영형 실습 환경을 Terraform으로 코드화
- [ ] **멀티클라우드 데이터 연계 검토**: AWS S3 로또 데이터를 GCP GCS/BigQuery로 연계
- [ ] **GCP 분석 확장**: BigQuery ML 기반 빈도/분포/패턴 분석 실습

---

## 6. Why This Project Matters

이 프로젝트는 단순히 로또 데이터를 보여주는 웹 페이지를 만드는 것이 아니라,  
**작은 주기성 서비스를 어떤 구조로 운영하면 비용과 운영 부담을 줄일 수 있는지**를 직접 설계하고 구현해 본 작업이다.

특히 아래와 같은 판단을 실제로 다뤘다.

- 상시 서버가 필요한가, 아니면 이벤트 기반 실행이 더 적합한가
- 최신 데이터와 이력 데이터는 같은 캐시 정책을 써도 되는가
- S3를 직접 공개할 것인가, CloudFront를 앞단에 둘 것인가
- 작은 서비스에서 어느 수준까지 보안과 비용을 함께 고려할 것인가

---

## 7. Repository / Demo

- **Demo**: `https://svc.nagom.io`
- **Repository**: `https://github.com/nakyeong/aws-serverless-lotto`