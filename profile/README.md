# 🐯 AJOU-CHATBOT — LangGraph + RAG 기반 아주대 신입생 도우미 챗봇



<div align="center">
  
![Image](https://github.com/user-attachments/assets/467dd9ae-69c0-428d-9c54-12afc4aea22a)

**📌 팀 래기(RAGGY)** — 전공자 11조  

</div>

---

## 🎯 Problem → Solution

대학에 처음 입학한 신입생은 “졸업요건, 수강신청, 장학, 휴학, 일정” 등 정보를 여러 사이트(포털/단과대/학과/요람)에 흩어져 있는 형태로 찾아야 합니다. 이로 인해 **정보 접근성 저하**, **학업 차질**, **행정실 반복 문의 증가** 문제가 발생합니다. 본 프로젝트는 **LangGraph로 대화 흐름을 제어**하고 **RAG로 정확한 문서 근거**를 찾아 **즉답**을 제공합니다.

---

## 🧩 What this repo contains

- **Backend (FastAPI)** — RAG/검색, LangGraph 파이프라인, API
- **Frontend (React + Vite + TS)** — 챗 UI, 카테고리 선택, 결과 뷰
- **Pipelines (Airflow)** — 공지 크롤링 → 전처리 → 임베딩 → Chroma 업서트
- **Vector / RDBMS** — ChromaDB (embeddings), PostgreSQL (원천/메타/요람/공지)

---

## 🏗️ Monorepo Structure

```
.
├─ backend/
│  ├─ app/
│  │  ├─ agents/         # LangGraph nodes (intent, clarify, retrieve, context, classify, answer)
│  │  ├─ api/            # FastAPI routers
│  │  ├─ core/           # config, logging, deps
│  │  ├─ data/           # loaders, preprocessors (LlamaParse/Gemini output handlers)
│  │  ├─ domain/         # schemas, DTOs
│  │  ├─ graphs/         # graph definitions / wiring
│  │  ├─ models/         # db models (SQLAlchemy) + vector store clients
│  │  ├─ scripts/        # one-off build/index jobs
│  │  ├─ services/       # retrieval, rerank, prompt, notice service
│  │  └─ utils/          # text cleaning, tokenization, etc.
│  ├─ storage/           # chroma persist dirs
│  ├─ docker/            # optional docker assets
│  ├─ airflow/           # DAGs: crawl → preprocess → embed
│  ├─ .env
│  ├─ requirements.in
│  └─ requirements.txt
│
└─ frontend/
   └─ src/
      ├─ components/     # UI components (Chat, MessageBubble, CategorySelector, TypingDots)
      ├─ lib/            # api.ts, categories.ts (utils & API calls)
      ├─ assets/         # images, svg
      ├─ types.ts        # shared types
      ├─ App.tsx         # app root
      └─ main.tsx        # Vite entry
```

---

## 🛠️ Tech Stack

**Backend**
- Python 3.10+ / FastAPI / Uvicorn
- SQLAlchemy (Async) + PostgreSQL
- ChromaDB (persistent)
- LangChain / **LangGraph**
- Embedding: **BAAI/bge-m3**
- Rerank: **BAAI/bge-reranker-v2-m3**
- Tokenization: **konlpy(Okt)**, **kiwipiepy**
- Scheduler: **Airflow**
- Parsing: **LlamaParse** + **Gemini 2.5 Pro** (표/멀티컬럼 PDF 구조화)

**Frontend**
- React + Vite + TypeScript
- Lightweight chat UI (streaming typing dots, category selector)

**LLMs (role-split)**
- Agents / pipeline: **GPT-4o-mini**
- Final answerer: **Claude 3.5 Sonnet**, **Gemini 1.5 Flash**

---

## 🧠 Core Pipeline (LangGraph)

```
Intent → Clarify → Retrieve → BuildContext → Classify → Answer
```

- **Hybrid Retrieval**: Dense(**bge-m3**) + Lexical(**BM25**) → **wRRF** 융합  
- **Parent–Child Chunks**: 자식 청크로 검색하고 **부모 문맥**을 반환해 맥락 보존  
- **Cross-Encoder Rerank**: 상위 K 후보를 bge-reranker로 점수화해 Top-N 확정  
- **Confidence Gate**: wRRF 신뢰도 미만이면 “관련 정보 없음”을 즉시 반환  
- **Heuristic Fast-Path**: 전과/행정 고정 응답은 LLM 호출 없이 즉시 답변  
- **Notices Automation**: Airflow로 매일 **크롤–전처리–임베딩** 자동화, 점진 업서트  
  (공지 테이블을 전체/단과대/학과로 분리 저장, 메타데이터로 필터링)

---

## 🛣️ 향후 개선 Roadmap

- [ ] 전 학과/전 학년 스케일업 (현재 신입생/일부 단과대 중심)  
- [ ] **Semantic Caching** (Redis + gRPC)로 **지연/비용** 절감  
- [ ] 공지 **포스터/OCR** 요약으로 커버리지 극대화  
- [ ] 섹션별 few-shot/루브릭 강화 및 DPO 데이터 축적

---

## 🧑‍🤝‍🧑 Team

| 역할        | 이름     | 담당 내용 |
|-------------|----------|-----------|
| 팀장        | 이승주   | 전체 설계, 프론트엔드, 요람 처리 |
| 공지 담당   | 조명아   | 공지 수집, 파이프라인 구성 |
| 공통 안내   | 안채연   | 학사 공통 Q&A 설계, 응답 설계 |


---

## 🏆 수상

![Image](https://github.com/user-attachments/assets/1de3b0cc-6fe1-4e07-a128-35a14363b9ec)

**2025년도 ICT챌린지 프로젝트 ICT경진대회 전공자팀 대상(1위)**
