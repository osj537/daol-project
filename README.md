# 🗂️ Daol — AI 문서 인식·분류 자동화 플랫폼

> LLM(Gemma)과 OCR을 결합해 문서 텍스트 추출부터 AI 요약까지 전 과정을 자동화한 풀스택 플랫폼

![서비스 메인 화면 스크린샷 또는 GIF 삽입 위치]

[![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white)](https://python.org)
[![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=flat&logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com)
[![React](https://img.shields.io/badge/React-61DAFB?style=flat&logo=react&logoColor=black)](https://react.dev)
[![MariaDB](https://img.shields.io/badge/MariaDB-003545?style=flat&logo=mariadb&logoColor=white)](https://mariadb.org)
[![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat&logo=docker&logoColor=white)](https://docker.com)

---

## 📌 프로젝트 개요

법률·일반 문서의 수동 분류 및 요약 업무를 자동화하기 위해 개발한 플랫폼입니다.
EasyOCR로 텍스트를 추출하고, LLM(Ollama/Gemma)으로 요약한 결과를 SSE 스트리밍으로 실시간 제공합니다.

- **개발 기간**: 2026.02 ~ 2026.04 (3개월)
- **참여 인원**: 인턴십 프로젝트 (글로벌 아카데미 더다올디앤씨)
- **역할**: 풀스택 (프론트엔드 + 백엔드 + AI 파이프라인)

---

## ✨ 주요 기능

- 📄 **문서 자동 처리**: HWPX(XML), PDF 등 포맷별 텍스트 추출 자동화
- 🤖 **AI 요약·번역**: LLM(Gemma) 기반 요약·번역, SSE 스트리밍으로 실시간 출력
- 🔐 **보안 문서 관리**: 숫자 4자리 비밀번호 검증 및 ACL(접근 제어) 적용
- 📡 **실시간 모니터링**: Discord Webhook으로 에러·주요 이벤트 즉시 알림
- 🧠 **OCR 정확도 개선**: 15만 장 합성 학습 데이터 기반 EasyOCR 파인튜닝

---

## 🛠️ 기술 스택

| 분류         | 기술                                   |
| ------------ | -------------------------------------- |
| Frontend     | React, react-hot-toast                 |
| Backend      | Python, FastAPI, SQLAlchemy ORM        |
| AI / Library | EasyOCR, Pillow, LMDB, OpenCV, PyMuPDF |
| 실시간 통신  | SSE (Server-Sent Events)               |
| 모니터링     | Discord Webhooks                       |
| DB / DevOps  | MariaDB, Docker, GitHub                |

---

## 👤 본인 기여 (오상진)

| 분류                     | 담당 내용                                                                                           |
| ------------------------ | --------------------------------------------------------------------------------------------------- |
| **SSE 텍스트 추출**      | `StreamingResponse` 기반 SSE 스트림, `start → page/ocr_progress → done/error` 이벤트 순서 설계      |
| **SSE 요약**             | 토큰 단위 스트리밍, 프론트엔드 30ms 배치 플러시로 렌더링 횟수 제한 (최대 33회/초)                   |
| **SSE 번역**             | 캐시 히트 시 DB 저장 결과를 단어 단위로 재스트리밍, LLM 재호출 없이 타이핑 효과 유지                |
| **문서 CRUD API**        | 편집·삭제·공개/비공개 토글 API 풀스택 구현, `can_user_access_document()` ACL 권한 체계              |
| **보안 처리**            | 중요 문서 4자리 숫자 비밀번호 검증, `is_important` 상태에 따른 password 자동 초기화                 |
| **Discord 모니터링**     | 전역 `HTTPException` 핸들러로 에러 메시지·상태코드·발생 경로·사용자 ID 자동 전송                    |
| **마이페이지 일괄 삭제** | `Promise.all()` 병렬 DELETE 요청, 완료 후 로컬 상태 즉시 필터링                                     |
| **챗봇 UI**              | 좌하단 고정 패널, TOOLTIP_TIPS 12초 로테이션, 카테고리 탭 필터링, 정적 Q&A 구조                     |
| **알림 시스템**          | `alert()` 대신 `react-hot-toast` 전역 등록, `toast.success / toast.error` 통일                      |
| **OCR 파인튜닝**         | PDF·HWP·HWPX 포맷별 텍스트 추출 → Pillow 합성 이미지 15만 장 → LMDB 저장 → EasyOCR 파인튜닝 전 과정 |
| **학습 최적화**          | `ThreadPoolExecutor(16 workers)` 병렬 생성, 5,000개마다 commit으로 메모리 과부하 방지               |

---

## 🔥 트러블슈팅

### 1. OCR 인식률 문제 — 모델이 아닌 데이터 문제였다

**문제 상황**
EasyOCR 기본 모델이 한국어 법률문서에서 오인식을 빈번하게 발생시킴

**원인 분석**
모델 자체의 한계보다 법률 특화 학습 데이터 부족이 원인이라고 판단

**해결 방법**
실제 법률·일반 문서에서 텍스트를 추출 후, Pillow로 노이즈·가우시안 블러·랜덤 색상 등 다양한 환경을 시뮬레이션한 합성 이미지 15만 장 생성.
TPS-ResNet-BiLSTM-Attn 구조로 EasyOCR 파인튜닝 진행

**결과**
파인튜닝 후 한국어 법률문서 인식 정확도 체감 수준으로 향상

---

### 2. AI 요약 대기 중 사용자 피드백 부재

**문제 상황**
요약 완료 후 한 번에 화면에 표시 → 응답 대기 중 사용자가 진행 여부를 알 수 없음

**해결 방법**
SSE 기반 토큰 단위 스트리밍으로 구조 전환.
프론트엔드에서 토큰마다 `setState` 대신 `setInterval 30ms` 배치 플러시 적용 → 렌더링 횟수 최대 33회/초로 제한하면서도 화면이 끊기지 않도록 조정

**결과**
사용자가 처리 진행 상황을 실시간 확인 가능, 체감 응답 속도 크게 향상

---

### 3. 번역 중복 LLM 호출 비용 문제

**문제 상황**
동일 문서를 같은 모델로 재번역할 때마다 LLM을 새로 호출하는 낭비 발생

**해결 방법**
요청 시 DB에 동일 모델로 번역된 결과가 있으면 LLM 호출 없이 기존 텍스트를 단어 단위로 split해 token 이벤트로 재전송 → 타이핑 효과 유지하면서 캐시 처리

**결과**
중복 요청 시 LLM 호출 제거, 응답 속도 대폭 향상

---

### 4. 운영 중 장애 대응 속도 문제

**문제 상황**
배포 후 이슈 발생 시 로그를 직접 뒤져야 해서 대응이 늦어지는 문제

**해결 방법**
전역 `HTTPException` 핸들러 구성 → 에러 메시지, 상태코드, 발생 경로, 사용자 ID를 Discord Webhook으로 즉시 전송.
`status_code >= 500`은 error(빨강), 4xx는 warning(주황)으로 레벨 구분

**결과**
배포 후 이슈 대응 속도 체감 수준으로 개선

---

## 🏗️ 시스템 아키텍처

```
[사용자]
   │
   ▼
[React Frontend]
   │  REST API / SSE 스트리밍
   ▼
[FastAPI Backend]
   ├── 문서 파싱 (HWPX / HWP / PDF)
   ├── EasyOCR 텍스트 추출 (SSE 진행률 스트리밍)
   ├── LLM 요약·번역 (Ollama / Gemma, SSE 토큰 스트리밍)
   ├── SQLAlchemy ORM (MariaDB)
   └── Discord Webhook (전역 에러 핸들러)

[OCR 파인튜닝 파이프라인]
   실제 문서 텍스트 추출 (PDF·HWP·HWPX)
   → Pillow 합성 이미지 생성 (15만 장, ThreadPoolExecutor 16 workers)
   → LMDB 저장 (5,000개마다 commit)
   → EasyOCR 파인튜닝 (TPS-ResNet-BiLSTM-Attn)
```

---

## 🚀 실행 방법

```bash
# 레포 클론
git clone https://github.com/kimm9487/daol_minipro
cd daol_minipro

# 백엔드 실행
pip install -r requirements.txt
uvicorn main:app --reload

# 프론트엔드 실행
cd frontend
npm install
npm start
```

> Docker 사용 시:
>
> ```bash
> docker-compose up --build
> ```

---

## 📎 관련 링크

- 🔗 [GitHub Repository](https://github.com/kimm9487/daol_minipro)
- 📓 [Notion 프로젝트 문서](https://granite-engineer-6d1.notion.site/AI-Ollama-Gemma-2ae6f9ef939f80bc9cfcdd212058bb8e)

---

> 이 README는 본인 기여 범위를 중심으로 작성되었습니다.
> 스크린샷 및 GIF는 실제 서비스 화면으로 교체해 주세요.
