# 주식 투자 에이전트 - 기술 스택 명세

## 1. 언어 및 런타임

| 항목 | 선택 | 이유 |
|-----|------|------|
| **메인 언어** | Python 3.11+ | 금융 라이브러리 생태계 최강, LLM SDK 지원 |
| **패키지 관리** | `uv` 또는 `poetry` | 의존성 재현성 보장 |
| **가상환경** | venv (프로젝트별 격리) | |

---

## 2. 데이터 수집 레이어

### 2.1 시장 데이터

| 라이브러리/API | 용도 | 비고 |
|-------------|------|------|
| `pykrx` | 한국 주가, 거래량, 시가총액 | 무료, KRX 공식 데이터 |
| `FinanceDataReader` | 국내/해외 주가 통합 조회 | 무료 |
| `한국투자증권 Open API` | 실시간 시세, 주문 실행 | 계좌 필요, 무료 |

### 2.2 재무/공시 데이터

| 라이브러리/API | 용도 | 비고 |
|-------------|------|------|
| `dart-fss` | DART 공시 수집 | DART API 키 필요 (무료) |
| `OpenDartReader` | DART 재무제표 파싱 | dart-fss 보완 |

### 2.3 뉴스/대안 데이터

| 라이브러리/API | 용도 | 비고 |
|-------------|------|------|
| `BeautifulSoup4` + `requests` | 네이버 금융 뉴스 크롤링 | |
| `feedparser` | RSS 피드 수집 | 한경, 머니투데이 등 |
| `selenium` (선택) | 동적 페이지 크롤링 | 필요 시 추가 |

### 2.4 매크로 데이터

| 라이브러리/API | 용도 | 비고 |
|-------------|------|------|
| `한국은행 ECOS API` | 금리, 환율, 경제지표 | 무료 API 키 필요 |
| `pykrx` | KOSPI/KOSDAQ 지수 | |

---

## 3. 데이터 저장 레이어

| 저장소 | 용도 | 선택 이유 |
|-------|------|---------|
| **SQLite** | 주가 시계열, 재무데이터 | 개발 단계 - 설치 불필요, 파일 기반 |
| **PostgreSQL** (운영 전환 시) | 데이터량 증가 시 마이그레이션 | |
| **JSON 파일** | 전략 설정, 에이전트 상태 저장 | 가독성 좋음 |
| **로컬 파일시스템** | 뉴스 원문, 공시 PDF | |

> **결정:** 개발 초기는 SQLite로 시작, 데이터량이 100만 행 이상이 되면 PostgreSQL 전환 검토

---

## 4. 데이터 처리 레이어

| 라이브러리 | 용도 |
|---------|------|
| `pandas` | 데이터 조작, 시계열 처리 |
| `numpy` | 수치 계산 |
| `pandas-ta` | 기술적 지표 계산 (RSI, MACD, BB 등) |
| `scipy` / `statsmodels` | 통계 분석, 회귀 분석 |

---

## 5. LLM 레이어

### 5.1 옵션 비교

| 옵션 | 장점 | 단점 | 비용 |
|-----|------|------|------|
| **Claude API (Anthropic)** | 긴 컨텍스트(200K), 분석력 우수, 한국어 강함 | API 비용 발생 | 유료 (토큰당 과금) |
| **GPT-4o (OpenAI)** | 성능 우수, 생태계 풍부 | API 비용 발생 | 유료 |
| **Gemini Pro (Google)** | 무료 티어 있음, 긴 컨텍스트 | 한국어 다소 약함 | 무료~유료 |
| **Ollama (로컬)** | 무료, 데이터 외부 전송 없음 | 성능 낮음, GPU 필요 | 무료 |

### 5.2 권장 구성

```
[권장] Claude API (claude-sonnet) 사용
- 이유 1: 한국 재무/경제 맥락 이해도 높음
- 이유 2: 200K 토큰 컨텍스트 → 긴 재무제표 전달 가능
- 이유 3: 구조화된 JSON 출력 안정성 높음
- 비용 최적화: 일상적 분석은 Haiku, 전략 수립은 Sonnet 사용
```

### 5.3 LLM 통합

| 라이브러리 | 용도 |
|---------|------|
| `anthropic` (Python SDK) | Claude API 호출 |
| `langchain` (선택) | 복잡한 체인 구성 시 |

---

## 6. 백테스트 레이어

| 라이브러리 | 장점 | 단점 | 권장 용도 |
|---------|------|------|---------|
| **vectorbt** | 빠름(numpy 벡터화), 시각화 내장 | 학습 곡선 있음 | **1순위 권장** |
| `backtrader` | 기능 풍부, 커뮤니티 큰 | 느림, 객체지향적 복잡도 | 복잡한 전략 시 |
| `zipline-reloaded` | QuantConnect 스타일 | 설치 복잡, 업데이트 느림 | 비권장 |

> **결정:** `vectorbt` 우선 사용 (속도 + 시각화 균형)

---

## 7. 실행/주문 레이어

| API | 용도 | 비고 |
|----|------|------|
| **한국투자증권 Open API** | 국내 주식 매수/매도, 잔고 조회 | 무료, 개인 계좌 연동 |
| `kis_api` (비공식 wrapper) | KIS API Python 래퍼 | |

### 주문 실행 안전장치
```python
# 반드시 구현할 안전장치
MAX_ORDER_AMOUNT = 1_000_000  # 1회 최대 주문 금액
MAX_DAILY_LOSS = -50_000      # 일일 최대 손실 한도
PAPER_TRADING_MODE = True     # 기본값: 모의투자 모드
```

---

## 8. 스케줄링 및 오케스트레이션

| 라이브러리 | 용도 |
|---------|------|
| `schedule` 또는 `APScheduler` | 데이터 수집, 전략 실행 스케줄링 |
| `asyncio` | 비동기 API 호출 처리 |

### 실행 스케줄 예시
```
08:50  - 장 전 데이터 수집 (전일 종가, 뉴스, 공시)
09:00  - 시장 분석 + LLM 전략 업데이트
09:05  - 매수 시그널 스캔 → 주문 실행
10:00  - 중간 포지션 점검
14:50  - 장 마감 전 정리
15:30  - 일간 리포트 생성 + 텔레그램 발송
```

---

## 9. 알림 시스템

| 도구 | 용도 |
|----|------|
| `python-telegram-bot` | 매매 시그널, 체결 알림, 일간 리포트 |

---

## 10. 모니터링 및 시각화

| 도구 | 용도 |
|----|------|
| `matplotlib` / `plotly` | 차트, 백테스트 결과 시각화 |
| `streamlit` | 간단한 모니터링 대시보드 (웹 UI) |
| `loguru` | 로깅 (에러, 거래 기록) |

---

## 11. 프로젝트 디렉토리 구조

```
ai_stock_agent/
├── config/
│   ├── settings.py          # API 키, 설정값
│   └── strategy_config.json # 전략 파라미터
├── data/
│   ├── collectors/          # 데이터 수집 모듈
│   │   ├── market.py        # 주가 수집
│   │   ├── fundamental.py   # 재무 수집
│   │   ├── news.py          # 뉴스 수집
│   │   └── macro.py         # 매크로 수집
│   ├── storage/             # DB 연결, 저장/조회
│   └── processors/          # 전처리, 지표 계산
├── analysis/
│   ├── technical.py         # 기술적 분석
│   ├── fundamental.py       # 재무 분석
│   ├── sentiment.py         # 뉴스 감성 분석
│   └── market_phase.py      # 시장 국면 판단
├── agents/
│   ├── analysis_agent.py    # 분석 에이전트
│   ├── strategy_agent.py    # 전략 수립 에이전트
│   └── execution_agent.py   # 실행 에이전트
├── backtest/
│   ├── engine.py            # 백테스트 실행
│   └── metrics.py           # 성과 지표 계산
├── execution/
│   ├── kis_client.py        # 한국투자증권 API 클라이언트
│   ├── order_manager.py     # 주문 관리
│   └── risk_manager.py      # 리스크 관리
├── notifications/
│   └── telegram_bot.py      # 텔레그램 알림
├── dashboard/
│   └── app.py               # Streamlit 대시보드
├── tests/                   # 단위/통합 테스트
├── logs/                    # 실행 로그
├── db/                      # SQLite DB 파일
└── main.py                  # 메인 진입점
```

---

## 12. 필요 API 키 목록

| API | 발급처 | 비용 |
|----|-------|------|
| 한국투자증권 Open API | securities.koreainvestment.com | 무료 |
| DART Open API | opendart.fss.or.kr | 무료 |
| 한국은행 ECOS | ecos.bok.or.kr | 무료 |
| Claude API (Anthropic) | console.anthropic.com | 유료 (토큰당) |
| Telegram Bot Token | @BotFather | 무료 |

---

*작성일: 2026-06-10*  
*버전: v0.1 (기획 단계)*
