# 주식 투자 에이전트 - 데이터 명세

## 1. 데이터 분류 체계

```
수집 데이터
├── 시장 데이터 (Market Data)         ← 가격, 거래량 등
├── 재무 데이터 (Fundamental Data)    ← 재무제표, 배당 등
├── 공시 데이터 (Disclosure Data)     ← DART 공시
├── 감성 데이터 (Sentiment Data)      ← 뉴스, 리포트
└── 매크로 데이터 (Macro Data)        ← 금리, 환율, 지수
```

---

## 2. 시장 데이터 (Market Data)

### 2.1 일별 주가 데이터 (ohlcv_daily)

| 컬럼명 | 타입 | 설명 | 예시 |
|-------|------|------|------|
| `date` | DATE | 거래일 | 2024-01-15 |
| `ticker` | VARCHAR(10) | 종목 코드 | 005930 |
| `name` | VARCHAR(50) | 종목명 | 삼성전자 |
| `open` | INTEGER | 시가 | 72000 |
| `high` | INTEGER | 고가 | 73500 |
| `low` | INTEGER | 저가 | 71800 |
| `close` | INTEGER | 종가 | 73000 |
| `volume` | BIGINT | 거래량 | 15234567 |
| `value` | BIGINT | 거래대금(원) | 1112122110000 |
| `market_cap` | BIGINT | 시가총액 | 435900000000000 |
| `shares` | BIGINT | 상장 주식 수 | 5969782550 |

**출처:** pykrx (`stock.get_market_ohlcv`)  
**수집 주기:** 일별 (장 마감 후 15:30~)  
**보관 기간:** 최소 5년 (백테스트용)

### 2.2 투자자별 매매 동향 (investor_trading)

| 컬럼명 | 타입 | 설명 |
|-------|------|------|
| `date` | DATE | 거래일 |
| `ticker` | VARCHAR(10) | 종목 코드 |
| `institution_net` | BIGINT | 기관 순매수(주) |
| `foreign_net` | BIGINT | 외국인 순매수(주) |
| `individual_net` | BIGINT | 개인 순매수(주) |

**출처:** pykrx (`stock.get_market_trading_value_by_investor`)

### 2.3 기술적 지표 (technical_indicators) — 계산값

| 컬럼명 | 타입 | 설명 |
|-------|------|------|
| `date` | DATE | 날짜 |
| `ticker` | VARCHAR(10) | 종목 코드 |
| `ma_5` | FLOAT | 5일 이동평균 |
| `ma_20` | FLOAT | 20일 이동평균 |
| `ma_60` | FLOAT | 60일 이동평균 |
| `ma_120` | FLOAT | 120일 이동평균 |
| `rsi_14` | FLOAT | RSI (14일) |
| `macd` | FLOAT | MACD |
| `macd_signal` | FLOAT | MACD 시그널 |
| `bb_upper` | FLOAT | 볼린저밴드 상단 |
| `bb_lower` | FLOAT | 볼린저밴드 하단 |
| `volume_ratio` | FLOAT | 거래량 비율 (5일 평균 대비) |

---

## 3. 재무 데이터 (Fundamental Data)

### 3.1 손익계산서 (income_statement)

| 컬럼명 | 타입 | 설명 |
|-------|------|------|
| `ticker` | VARCHAR(10) | 종목 코드 |
| `period` | VARCHAR(10) | 결산 기간 (2023Q4, 2023) |
| `period_type` | ENUM | 'annual' / 'quarter' |
| `revenue` | BIGINT | 매출액 |
| `operating_profit` | BIGINT | 영업이익 |
| `net_income` | BIGINT | 당기순이익 |
| `eps` | FLOAT | 주당순이익 |
| `operating_margin` | FLOAT | 영업이익률 (%) |

### 3.2 대차대조표 (balance_sheet)

| 컬럼명 | 타입 | 설명 |
|-------|------|------|
| `ticker` | VARCHAR(10) | 종목 코드 |
| `period` | VARCHAR(10) | 결산 기간 |
| `total_assets` | BIGINT | 총자산 |
| `total_liabilities` | BIGINT | 총부채 |
| `total_equity` | BIGINT | 자기자본 |
| `cash` | BIGINT | 현금 및 현금성 자산 |
| `debt_ratio` | FLOAT | 부채비율 (%) |

### 3.3 현금흐름표 (cash_flow)

| 컬럼명 | 타입 | 설명 |
|-------|------|------|
| `ticker` | VARCHAR(10) | 종목 코드 |
| `period` | VARCHAR(10) | 결산 기간 |
| `operating_cf` | BIGINT | 영업활동 현금흐름 |
| `investing_cf` | BIGINT | 투자활동 현금흐름 |
| `financing_cf` | BIGINT | 재무활동 현금흐름 |
| `free_cf` | BIGINT | 잉여현금흐름 (FCF) |

### 3.4 주요 밸류에이션 지표 (valuation_metrics) — 계산값

| 컬럼명 | 타입 | 설명 |
|-------|------|------|
| `date` | DATE | 기준일 |
| `ticker` | VARCHAR(10) | 종목 코드 |
| `per` | FLOAT | 주가수익비율 |
| `pbr` | FLOAT | 주가순자산비율 |
| `psr` | FLOAT | 주가매출비율 |
| `roe` | FLOAT | 자기자본이익률 (%) |
| `roa` | FLOAT | 총자산이익률 (%) |
| `dividend_yield` | FLOAT | 배당수익률 (%) |

**출처:** DART OpenAPI + pykrx 결합  
**수집 주기:** 분기 1회 (공시 후 수집)

---

## 4. 공시 데이터 (Disclosure Data)

### 4.1 공시 목록 (disclosures)

| 컬럼명 | 타입 | 설명 |
|-------|------|------|
| `rcept_no` | VARCHAR(20) | DART 접수 번호 (PK) |
| `ticker` | VARCHAR(10) | 종목 코드 |
| `corp_name` | VARCHAR(100) | 법인명 |
| `report_nm` | VARCHAR(200) | 공시 제목 |
| `rcept_dt` | DATE | 접수일 |
| `category` | VARCHAR(50) | 공시 유형 |
| `summary` | TEXT | LLM 요약 (후처리) |
| `sentiment` | ENUM | 'positive'/'neutral'/'negative' |

**공시 유형 분류:**
- 정기공시: 사업보고서, 분기보고서, 반기보고서
- 주요사항: 유상증자, 무상증자, 합병, 분할
- 지분공시: 대량보유, 임원 주요주주 특정증권
- 기타: 공정공시, 수시공시

**출처:** DART OpenAPI (`dart-fss` 라이브러리)  
**수집 주기:** 실시간 (30분 간격 폴링)

---

## 5. 감성 데이터 (Sentiment Data)

### 5.1 뉴스 (news)

| 컬럼명 | 타입 | 설명 |
|-------|------|------|
| `id` | INTEGER | PK |
| `ticker` | VARCHAR(10) | 관련 종목 코드 (NULL 가능) |
| `title` | VARCHAR(500) | 뉴스 제목 |
| `content` | TEXT | 뉴스 본문 |
| `source` | VARCHAR(100) | 출처 (네이버, 한경 등) |
| `published_at` | DATETIME | 발행 시각 |
| `url` | TEXT | 원문 URL |
| `sentiment_score` | FLOAT | 감성 점수 (-1.0 ~ 1.0) |
| `keywords` | JSON | 추출 키워드 리스트 |

**수집 대상:**
- 종목별 뉴스: 네이버 금융 뉴스
- 시황 뉴스: 코스피/코스닥 시황
- 산업별 뉴스: 반도체, 2차전지, 바이오 등 주요 섹터

**출처:** 네이버 금융 RSS / 크롤링  
**수집 주기:** 1시간 간격

---

## 6. 매크로 데이터 (Macro Data)

### 6.1 거시경제 지표 (macro_indicators)

| 컬럼명 | 타입 | 설명 | 출처 |
|-------|------|------|------|
| `date` | DATE | 기준일 | |
| `kospi` | FLOAT | KOSPI 지수 | pykrx |
| `kosdaq` | FLOAT | KOSDAQ 지수 | pykrx |
| `usd_krw` | FLOAT | 원달러 환율 | 한국은행 ECOS |
| `base_rate` | FLOAT | 한국은행 기준금리 (%) | 한국은행 ECOS |
| `treasury_3y` | FLOAT | 국고채 3년 금리 (%) | 한국은행 ECOS |
| `cpi_yoy` | FLOAT | 소비자물가 전년비 (%) | 한국은행 ECOS |
| `foreign_reserve` | BIGINT | 외환보유액 (억 달러) | 한국은행 ECOS |

**수집 주기:** 일별 (발표 시 수집)

---

## 7. 에이전트 운영 데이터

### 7.1 전략 이력 (strategies)

| 컬럼명 | 타입 | 설명 |
|-------|------|------|
| `id` | INTEGER | PK |
| `created_at` | DATETIME | 생성 시각 |
| `name` | VARCHAR(200) | 전략명 |
| `description` | TEXT | LLM이 생성한 전략 설명 |
| `rules` | JSON | 매수/매도 조건 구조체 |
| `backtest_return` | FLOAT | 백테스트 수익률 (%) |
| `backtest_mdd` | FLOAT | 최대낙폭 (%) |
| `backtest_sharpe` | FLOAT | 샤프비율 |
| `status` | ENUM | 'testing'/'active'/'retired' |

### 7.2 거래 기록 (trades)

| 컬럼명 | 타입 | 설명 |
|-------|------|------|
| `id` | INTEGER | PK |
| `strategy_id` | INTEGER | FK → strategies |
| `ticker` | VARCHAR(10) | 종목 코드 |
| `trade_type` | ENUM | 'buy'/'sell' |
| `quantity` | INTEGER | 수량 |
| `price` | INTEGER | 체결 단가 |
| `amount` | BIGINT | 체결 금액 |
| `executed_at` | DATETIME | 체결 시각 |
| `mode` | ENUM | 'paper'/'live' |
| `pnl` | FLOAT | 손익 (매도 시 기록) |

---

## 8. 데이터 보관 정책

| 데이터 종류 | 보관 기간 | 이유 |
|---------|---------|------|
| 일별 주가 | 영구 | 백테스트 기간 확장 |
| 재무제표 | 영구 | 장기 펀더멘털 분석 |
| 공시 원문 | 3년 | 스토리지 절약 |
| 뉴스 | 1년 | 감성 분석 학습 |
| 매크로 지표 | 영구 | 시장 사이클 분석 |
| 거래 기록 | 영구 | 성과 추적 |

---

## 9. 초기 데이터 수집 계획

```
[1단계 초기 적재] - 개발 시작 시 1회성 수집
 ├── KOSPI 200 구성 종목 5년치 주가 데이터
 ├── 해당 종목 재무제표 (최근 3년, 분기별)
 └── 매크로 지표 (최근 5년)

[2단계 상시 수집] - 개발 진행 중 지속
 ├── 일별 주가 업데이트 (장 마감 후)
 ├── 뉴스 수집 (1시간 간격)
 ├── 공시 수집 (30분 간격)
 └── 매크로 지표 (발표 시)
```

---

*작성일: 2026-06-10*  
*버전: v0.1 (기획 단계)*
