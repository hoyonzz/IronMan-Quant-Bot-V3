# 🌊 IronMan-Quant-Bot(V3.0)

> **"단순한 트레이딩 봇을 넘어, AI가 룰(Rule)을 스스로 정의하고 API로 통신하는 '적응형 마이크로서비스(MSA) 데이터 레이크'"**

<p align="center">
  <img src="https://img.shields.io/badge/Python-3.11+-blue.svg" alt="Python">
  <img src="https://img.shields.io/badge/FastAPI-0.100+-009688.svg" alt="FastAPI">
  <img src="https://img.shields.io/badge/Docker-Compose-2496ED.svg" alt="Docker">
  <img src="https://img.shields.io/badge/PostgreSQL-15.0-336791.svg" alt="PostgreSQL">
  <img src="https://img.shields.io/badge/AI-Gemini%202.5%20Pro-purple.svg" alt="Gemini">
</p>

---

## 1. 프로젝트 개요 (Overview)

**IronMan-Quant-Bot(V3.0)**는 정적이고 하드코딩된 규칙에 의존하는 기존 퀀트 시스템의 한계를 극복하기 위해 설계된 **Backend API Server 및 Data Lake**입니다. 
단일 스크립트 기반의 봇(Bot) 형태를 탈피하여, 대용량 금융 데이터를 적재하고 LLM(Gemini)이 시장 체제(Regime)에 따라 동적으로 필터링 파라미터를 조정하는 **Data-driven Architecture**를 구현했습니다.

### 🎯 무엇을 고민했는가? (Problem & Solution)

**기존 시스템의 한계:**
- ❌ **Hardcoded Threshold**: "VIX가 30 이상이면 매수 안함" 같은 코드가 박혀 있어, 시장 구조 변화(Regime Shift) 시 즉각 대응 불가.
- ❌ **Scalability Issue**: SQLite 기반의 단일 스크립트는 향후 웹 프론트엔드 연동이나 다중 전략 확장 시 병목 발생.
- ❌ **Environment Conflict**: "내 로컬에서는 돌아가는데 서버에서는 안 도는" 의존성 문제.

**해결책:**
- ✅ **Dynamic Parameter Engine**: AI가 매일 시장의 다차원 매크로 데이터를 분석하여 파라미터(기준점)를 산출하고 DB에 주입. 로직 코드는 단 한 줄도 수정할 필요가 없습니다.
- ✅ **Microservices & API**: FastAPI 기반으로 구축되어, 향후 어떤 프론트엔드(React, Vue)나 외부 서비스와도 즉시 연동 가능한 확장성 제공.
- ✅ **Containerization**: Docker & Docker-compose를 도입하여, 명령어 한 줄로 DB와 API 서버가 동일한 환경에서 배포되는 인프라(IaC) 구축.

---

## 2. Key Architecture (핵심 기술 및 구조)

시스템은 역할이 철저히 분리된 **4-Pillars 구조**로 작동합니다.

### 🗄️ Pillar 1: The Lake (Data Persistence Layer)
*   **Core Tech:** `PostgreSQL`, `SQLAlchemy (ORM)`, `Alembic`
*   **Role:**
    *   **다차원 데이터 적재**: 종목 정보, 시계열 가격 데이터뿐만 아니라 하이일드 스프레드, 환율 등 거시경제 지표를 RDBMS에 정규화하여 저장.
    *   **JSONB 활용**: AI의 추론 근거(`ai_reasoning`) 등 비정형 데이터를 유연하게 저장.

### 🧠 Pillar 2: The Brain (Dynamic AI Engine)
*   **Core Tech:** `Google Gemini 2.5 Pro`, `LangChain(optional)`
*   **Role:**
    *   매크로 DB를 읽어 들여 현재 시장 체제(Bull/Bear/Volatile) 판단.
    *   오늘 스캐너가 사용할 **스마트머니 통과 기준(%) 및 켈리 베팅 비중**을 동적으로 결정하여 DB에 기록(Update).

### ⚡ Pillar 3: The Core API (FastAPI Server)
*   **Core Tech:** `FastAPI`, `Pydantic`, `Uvicorn`, `Asyncio`
*   **Role:**
    *   비동기(Async) 기반의 초고속 스캐너 및 RESTful API 엔드포인트 제공.
    *   하드코딩된 상수가 아닌, DB에 기록된 **당일의 동적 파라미터를 쿼리 변수로 주입**하여 조건에 맞는 종목(Top Picks) 도출.

### 🐳 Pillar 4: The Infra (Containerization)
*   **Core Tech:** `Docker`, `Docker-compose`
*   **Role:**
    *   앱과 DB를 컨테이너로 격리하여 OS 환경에 구애받지 않는 재현성 100% 보장.
    *   `Volume Mount`를 통한 데이터 영속성 유지 및 `Port Forwarding`을 통한 네트워크 제어.

---

## 3. Migration Report (Legacy Bot ➡️ MSA Data Lake)

| 구분 | V2 (Legacy Bot) | **V3 (IQB Data Lake)** | 백엔드 관점의 개선 효과 |
|:---|:---|:---|:---|
| **아키텍처** | 단일 스크립트 (Monolithic) | **마이크로서비스 (MSA 기반 API)** | 프론트엔드 분리 및 타 서비스 연동의 확장성 확보 |
| **인프라** | 로컬 환경 의존 | **Docker Containerization** | 배포 자동화 및 환경 충돌 원천 차단 |
| **데이터베이스**| SQLite (File-based) | **PostgreSQL (Server-based)** | 다중 접속(Concurrency) 처리 및 정형/비정형(JSON) 데이터 복합 설계 |
| **로직 제어** | 코드 내부의 If-Else 룰 | **Data-driven Dynamic Logic** | 소스 코드 재배포 없이 AI의 판단만으로 시스템 룰 실시간 변경 |

---

## 4. Directory Structure (폴더 구조)
```text
IronMan-Quant-Bot(3.0)
├── docker-compose.yml       # 인프라 오케스트레이션 (PostgreSQL + FastAPI)
├── Dockerfile               # FastAPI 컨테이너 빌드 레시피
├── .env                     # 환경 변수 (DB 비번, API Key 등 - 깃허브 제외)
├── app/                     # 메인 애플리케이션 (FastAPI)
│   ├── main.py              # API 진입점 (Entry Point)
│   ├── api/                 # 라우터 (Endpoints: /scan, /macro 등)
│   ├── core/                # 설정 및 의존성 주입 (Config, DB Session)
│   ├── models/              # SQLAlchemy 스키마 (PostgreSQL 테이블 매핑)
│   ├── schemas/             # Pydantic 모델 (요청/응답 데이터 검증)
│   ├── services/            # 비즈니스 로직 (AI 판단, 수급 필터링 등)
│   └── utils/               # 외부 통신 (KIS API, Telegram 알림 등)
└── requirements.txt         # 파이썬 패키지 목록
