# AWS Serverless Lotto

EC2 기반으로 운영되던 로또 조회 서비스를 더 낮은 비용과 적은 운영 부담으로 재구성하기 위해 만든 서버리스 프로젝트.

## Overview
이 프로젝트는 로또 데이터를 주기적으로 수집해 S3에 저장하고, CloudFront를 통해 정적 웹에서 조회할 수 있도록 구성한 서비스다.

기존처럼 항상 켜져 있는 서버를 운영하는 대신, EventBridge, Lambda, S3, CloudFront를 활용해 주기성 작업에 맞는 서버리스 구조로 전환했다.  
현재는 `https://svc.nagom.io` 서브도메인으로 분리 배포해 실제 서비스 형태로 운영하고 있다.

## Why
기존 EC2 기반 로또 서비스는 월 비용이 발생하고, 가끔 다운되며, 항상 켜져 있는 서버를 관리해야 하는 부담이 있었다.

이 프로젝트는 이런 문제를 바탕으로 다음 목표를 두고 설계했다.

- 상시 실행 서버 제거
- 운영 단순화
- 장애 포인트 축소
- 비용 최소화
- 정적 웹 기반의 가벼운 서비스 구성

## Architecture
- **EventBridge**: 정해진 시각에 로또 수집 작업 실행
- **Lambda**: 로또 데이터 수집 및 JSON 생성
- **S3 (data bucket)**: 회차별 데이터와 `latest.json` 저장
- **S3 (web bucket)**: 정적 웹 파일 저장
- **CloudFront**: 정적 웹 배포 및 `/data/*` 경로 라우팅
- **OAC**: S3를 직접 공개하지 않고 CloudFront만 접근 허용
- **WAF**: 기본 보호, rate limit, 국가 제한 적용
- **CloudWatch / SNS**: 모니터링 및 알림 구성

## Tech Stack
- Amazon EventBridge
- AWS Lambda
- Amazon S3
- Amazon CloudFront
- AWS WAF
- Amazon CloudWatch
- Amazon SNS
- AWS Certificate Manager (ACM)
- HTML
- CSS
- JavaScript

## Key Features
- 최신 회차 데이터 조회
- 회차별 JSON 이력 저장
- 정적 웹에서 latest / round 데이터 조회
- 서버리스 기반 주기 실행
- CloudFront 캐시 정책 분리 적용
- 커스텀 도메인 + HTTPS 적용

## Current Status
- EventBridge → Lambda → S3 데이터 적재 완료
- web/data 버킷 분리 완료
- CloudFront 단일 배포에서 web / data 경로 분기 완료
- OAC 적용으로 S3 비공개 접근 구성 완료
- 커스텀 도메인 및 ACM 인증서 적용 완료
- `svc.nagom.io` 서브도메인 분리 배포 완료
- 정적 웹에서 latest / round 데이터 조회 가능
- WAF rate limit 및 국가 제한 적용 완료
- CloudWatch Alarm 및 SNS 알림 연동 완료
- GCP 기반 멀티클라우드 확장 작업 진행 중

## Security
- S3 직접 공개 차단
- CloudFront OAC 기반 접근
- AWS WAF 기본 보호 적용
- rate-limit 규칙 적용
- 한국 외 요청 차단 규칙 적용

## Cost Optimization
- 상시 실행 EC2 없이 이벤트 기반으로만 실행
- S3 정적 파일 저장 구조 사용
- CloudFront 캐시 활용으로 원본 요청 감소
- 필요 시에만 Lambda 실행되는 구조로 운영

## Data Structure
- latest: `data/lotto/latest.json`
- history: `data/lotto/round/{round}.json`

## Domain
- Live Service: `https://svc.nagom.io`

## Project Direction
이 프로젝트는 단순한 로또 조회 페이지가 아니라,  
주기성 데이터 수집 작업을 서버리스 아키텍처로 어떻게 단순하고 저렴하게 운영할 수 있는지 보여주기 위한 포트폴리오 프로젝트다.

또한 AWS 기반 서버리스 구조를 중심으로 구현한 뒤, 이후 GCP를 포함한 멀티클라우드 방향으로도 확장 가능성을 검토하고 있다.

## Next Steps
- 정적 웹 UI 개선
- README 아키텍처 다이어그램 추가
- Terraform으로 인프라 코드화
- CloudWatch / SNS 운영 기준 정교화
- GCP 기반 분석 / 멀티클라우드 연계 구조 구체화

## Links
- Live Service: [svc.nagom.io](https://svc.nagom.io)
- Portfolio: (추가 예정)
- Repository: [GitHub Repository](https://github.com/skrud9418/aws-serverless-lotto)
