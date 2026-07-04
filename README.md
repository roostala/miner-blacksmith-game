# Miner & Blacksmith (광부 & 대장장이) ⛏️🔥

**Miner & Blacksmith**는 브라우저 환경에서 동작하는 웹 기반 시뮬레이션 RPG 게임입니다.  
플레이어는 광부(Miner)로서 던전을 탐험하고 다양한 광석을 채굴하며, 대장장이(Blacksmith)로서 광석을 제련하여 강력한 장비와 부적을 제작합니다.

---

## 🌟 주요 특징 (Key Features)

* **분리된 로그인 & 게임 아키텍처**:
  * `login.html`: 세련된 게이밍 테마의 로그인/회원가입 랜딩 페이지 및 게스트 플레이 지원.
  * `game.html`: 플레이어 프로필 바, 채광, 던전 탐험, 대장간 제작, 인벤토리/상점이 통합된 메인 게임 페이지.
  * `index.html`: 접속 세션 상태를 자동 파악하여 페이지를 리다이렉트하는 스마트 라우터.
* **로그인 & 게스트 모드 양방향 지원**:
  * **로그인 플레이어**: Supabase Cloud DB(`game_saves`)를 통한 실시간 진행 상황 멀티 디바이스 동기화.
  * **게스트 플레이어**: 회원가입 없이 브라우저 `LocalStorage` 기반 즉시 게임 플레이 가능.
* **다채로운 게임 엔진**:
  * **동적 드랍 가중치 엔진 (`weightedOre`)**: 광부 레벨, 던전 깊이, 위험도, 곡괭이 사양에 따른 광석 드랍률 계산.
  * **그리드 던전 탐험 (`explore`)**: 3x3 미니맵, 방별 위험도/광맥 수 관리, 낙석 피격 시스템.
  * **대장간 제련 & 숙련도 (`craft`)**: 성공률 보정 커브 및 실패 시 숙련도 보상 시스템.
* **Web Audio 기반 사운드 엔진**: 외부 음원 파일 없이 Web Audio API로 앰비언트 BGM 및 타격 SFX(Dynamic Base64 WAV Fallback 지원) 합성.

---

## 📁 디렉터리 구조 (Directory Structure)

```text
miner-blacksmith-game/
├── README.md               # 프로젝트 메인 README (본 문서)
├── index.html              # 스마트 진입점 및 세션 라우터 (Smart Router)
├── login.html              # 로그인 / 회원가입 / 게스트 시작 랜딩 페이지
├── game.html               # 메인 게임 실행 페이지 (채광, 탐험, 대장간, 상점)
├── api/
│   └── config.js           # Vercel Serverless API (Supabase 환경변수 전달)
└── doc/
    ├── README.md           # 문서 목차 안내
    ├── DEVELOPMENT_GUIDE.md# 시스템 분석, 아키텍처 및 상세 개발 가이드
    └── IMPLEMENTATION_PLAN.md # 페이지 분리 구현 계획서
```

---

## 🚀 빠른 시작 방법 (Quick Start)

### 1. 로컬 실행
별도의 빌드 과정 없이 웹 서버를 구동하여 바로 실행할 수 있습니다.

```bash
# Python 내장 HTTP 서버 구동 예시
python3 -m http.server 8080

# 또는 npx serve 구동
npx serve -l 8080
```

브라우저에서 `http://localhost:8080` 으로 접속하면 세션 파악 후 `login.html` 또는 `game.html`로 자동 진입합니다.

---

## 🎮 게임 조작 및 플레이 방법

1. **시작 화면 (`login.html`)**:
   * 이메일/비밀번호로 로그인 및 회원가입하거나, **'🎮 게스트 모드로 바로 시작'** 버튼을 클릭합니다.
2. **채광 (`game.html`)**:
   * `⛏ 채광` 및 `⛏⛏ 10회 채광` 버튼으로 광석을 수급하고 광부 레벨을 올립니다.
   * `광석 판매`로 1~3티어 광석을 팔아 골드를 확보하고, `상점`에서 성능 좋은 곡괭이를 구매합니다.
3. **던전 탐험**:
   * 동/서/남/북 및 `깊은 길`로 이동하여 새로운 방을 발견합니다.
   * 깊은 층일수록 위험도가 커지지만 미스릴, 아다만트, 성철 등 희귀 광석 채굴 확률이 증가합니다.
   * 광맥 고갈 시 `대장간 귀환`으로 입구 복귀 및 광맥을 재생성합니다.
4. **대장간 제작**:
   * 수집한 광석으로 곡괭이, 부적, 왕관, 코어 등을 제련합니다.
   * 제작 성공 시 능력치 상승 및 골드 보상을 받으며, 실패 시에도 숙련도가 상승하여 다음 성공률이 높아집니다.

---

## 📚 상세 문서 (Documentation Links)

* 📄 [개발 가이드 및 소스 분석 문서 (DEVELOPMENT_GUIDE.md)](file:///Users/flowerhead/miner-blacksmith-game/doc/DEVELOPMENT_GUIDE.md)
* 📋 [로그인/게임 페이지 분리 구현 계획서 (IMPLEMENTATION_PLAN.md)](file:///Users/flowerhead/miner-blacksmith-game/doc/IMPLEMENTATION_PLAN.md)
