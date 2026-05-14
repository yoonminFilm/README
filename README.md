# Yoonmin Film Archive

<div align="center">
  <img src="./img/banner-logo.png" alt="Yoonmin Film Archive" width="40%"/>
  <br/><br/>
  <p><strong>Photography Portfolio & Archive Platform</strong></p>
  <p>작품과 커머셜 프로젝트를 한곳에 담아내는 디지털 갤러리</p>
  <br/>
  <a href="https://www.yoonminfilm.co.kr">www.yoonminfilm.co.kr</a>
</div>

<br/>

## Overview

Yoonmin Film Archive는 사진 작가 윤성민의 포트폴리오 웹 애플리케이션입니다.
프로젝트별 갤러리, 카테고리 기반 작품 탐색, 문의 시스템, 관리자 대시보드를 제공합니다.

<br/>

## Tech Stack

<div align="center">
  <img src="https://img.shields.io/badge/Next.js-16-000000?style=for-the-badge&logo=next.js&logoColor=white"/>
  <img src="https://img.shields.io/badge/TypeScript-3178C6?style=for-the-badge&logo=typescript&logoColor=white"/>
  <img src="https://img.shields.io/badge/Tailwind_CSS-v4-06B6D4?style=for-the-badge&logo=tailwindcss&logoColor=white"/>
  <img src="https://img.shields.io/badge/DynamoDB-4053D6?style=for-the-badge&logo=amazondynamodb&logoColor=white"/>
  <img src="https://img.shields.io/badge/S3-569A31?style=for-the-badge&logo=amazons3&logoColor=white"/>
  <img src="https://img.shields.io/badge/Lambda-FF9900?style=for-the-badge&logo=awslambda&logoColor=white"/>
  <img src="https://img.shields.io/badge/Vercel-000000?style=for-the-badge&logo=vercel&logoColor=white"/>
</div>

<br/>

| Layer | Technology |
|-------|-----------|
| **Frontend** | Next.js 16 (App Router), React 19, Framer Motion |
| **Styling** | Tailwind CSS v4, Pretendard Variable Font |
| **Database** | DynamoDB (Single Table Design) |
| **Storage** | S3 (presigned upload, public read) |
| **Image Processing** | Lambda + sharp (auto resize on upload) |
| **Auth** | JWT (HttpOnly cookie) + bcrypt |
| **Notification** | Resend (email on inquiry) |
| **Deploy** | Vercel (frontend) + AWS Lambda (image resize) |

<br/>

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                        Client                           │
│              (Browser / Mobile)                         │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────────────────┐
│                    Vercel (Edge)                         │
│  ┌──────────┐  ┌───────────┐  ┌──────────────────────┐  │
│  │ Middleware│─▶│ App Router│  │  Static / ISR Pages  │  │
│  │ (JWT Auth)│  │ API Routes│  │  (project/[slug])    │  │
│  └──────────┘  └─────┬─────┘  └──────────────────────┘  │
└──────────────────────┼──────────────────────────────────┘
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
   ┌──────────┐  ┌──────────┐  ┌──────────┐
   │ DynamoDB │  │    S3    │  │  Resend  │
   │          │  │          │  │  (Email) │
   │ Projects │  │ originals│  └──────────┘
   │ Images   │  │ thumbnails│
   │ Inquiries│  │ medium   │
   └──────────┘  └────┬─────┘
                      │ S3 Event
                      ▼
                ┌──────────┐
                │  Lambda  │
                │  (sharp) │
                │ auto     │
                │ resize   │
                └──────────┘
```

<br/>

## Features

### Photo Book Carousel
메인 페이지에서 프로젝트별 이미지를 저널 스타일 그리드로 자동 회전.
레이아웃은 랜덤으로 선택되며, 다음 슬라이드를 미리 로드합니다.

### Category-based Project View
프로젝트 내 카테고리(예: Paris, Italy, Swiss)별 섹션 전환.
우측 네비게이션으로 빠른 이동, IntersectionObserver로 현재 위치 추적.

### Admin Dashboard
프로젝트 CRUD, 이미지 업로드(drag & drop + presigned URL), 카테고리 관리, 문의 관리.
변경 시 Next.js 캐시 자동 갱신 (revalidatePath).

### Image Pipeline
1. 관리자가 이미지 업로드 → S3 `originals/` presigned PUT
2. Lambda 트리거 → sharp로 자동 리사이즈
   - `thumbnails/` — 400px, WebP, q80
   - `medium/` — 1200px, WebP, q85
3. 메타데이터 DynamoDB 저장

### Inquiry System
About 페이지 문의 폼 → DynamoDB 저장 → Resend 이메일 알림.
Rate limit: 50/day, 200/week.

<br/>

## Database Schema

**Single Table Design** — 하나의 테이블(`yoonminfilm`)에 모든 엔티티 저장.

| Entity | PK | SK | GSI1PK | GSI1SK |
|--------|----|----|--------|--------|
| Project | `PROJECT` | `PROJECT#<order>#<id>` | `SLUG#<slug>` | `PROJECT#<order>` |
| Image | `PROJECT#<id>` | `IMAGE#<order>#<imgId>` | — | — |
| Inquiry | `INQUIRY` | `INQ#<time>#<id>` | — | — |

<br/>

## Security

| Layer | Implementation |
|-------|---------------|
| API Auth | JWT middleware — 모든 write API 인증 필수 |
| Password | bcrypt hash (HttpOnly, Secure, SameSite cookie) |
| S3 Write | presigned URL only (10분 유효) |
| S3 Read | public (이미지 CDN 용도) |
| Rate Limit | 문의 POST 50/day, 200/week |
| Secrets | 환경변수 (.gitignore, Vercel encrypted) |

<br/>

## Version History

### V2 — 2026.05 (Current)
Next.js 풀스택 재구축.

- Next.js 16 App Router + TypeScript + Tailwind CSS v4
- DynamoDB Single Table Design
- Admin 대시보드 (프로젝트 CRUD, 이미지/카테고리 관리)
- S3 presigned upload + Lambda 자동 리사이즈
- JWT 인증 + middleware API 보호
- Photo book 스타일 메인 캐러셀
- Resend 이메일 알림
- Vercel + 커스텀 도메인 (yoonminfilm.co.kr)

### V1 — 2024.11 ~ 2025.11
React CRA + AWS Serverless 초기 버전.

- React 18 SPA (CRA) + Bootstrap
- Express + Lambda (Serverless Framework)
- S3 JSON 파일 기반 데이터 저장
- 카카오톡 알림 연동

<br/>

## Contact

- **Site**: [www.yoonminfilm.co.kr](https://www.yoonminfilm.co.kr)
- **Email**: yoonmin.tech@gmail.com
- **Instagram**: [@yoonmin_film](https://www.instagram.com/yoonmin_film/)
