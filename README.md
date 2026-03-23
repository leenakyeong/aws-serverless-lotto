# aws-serverless-lotto
Serverless lotto service built with EventBridge, Lambda, S3 and CloudFront
# AWS Serverless Lotto

EC2 기반으로 운영되던 로또 조회 서비스를 더 낮은 비용과 적은 운영 부담으로 재구성하기 위해 만든 서버리스 프로젝트.

## Overview
이 프로젝트는 EventBridge, Lambda, S3, CloudFront를 이용해 로또 데이터를 주기적으로 수집하고, 정적 웹에서 조회할 수 있도록 구성한 서비스다.

## Why
기존 EC2 기반 서비스는 월 비용이 발생하고, 항상 켜져 있는 서버를 관리해야 했다.
이 프로젝트는 주기성 작업에 맞는 서버리스 구조로 전환하여 비용과 운영 부담을 줄이는 것을 목표로 했다.

## Stack
- Amazon EventBridge
- AWS Lambda
- Amazon S3
- Amazon CloudFront
- AWS WAF
- Amazon CloudWatch
- Amazon SNS
- HTML / CSS / JavaScript

## Current Status
- EventBridge → Lambda → S3 데이터 적재 완료
- web/data 버킷 분리 완료
- CloudFront 배포 완료
- OAC 적용 완료
- 정적 웹에서 latest / round 데이터 조회 가능
- WAF rate limit 및 국가 제한 적용
- CloudWatch Alarm 및 SNS 알림 연동 완료

## Current Status
- EventBridge → Lambda → S3 데이터 적재 완료
- web/data 버킷 분리 완료
- CloudFront 배포 완료
- OAC 적용 완료
- 정적 웹에서 latest / round 데이터 조회 가능
- WAF rate limit 및 국가 제한 적용

## Next Steps
- 커스텀 도메인 연결
- 정적 웹 UI 개선
- Terraform으로 인프라 코드화
- CloudWatch Alarm / SNS 운영 고도화
