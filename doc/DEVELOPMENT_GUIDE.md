# Miner & Blacksmith - 소스 분석, 사용법 및 개발 방향 문서

이 문서는 **Miner & Blacksmith (광부 & 대장장이)** 웹 게임의 소스 코드 분석 결과, 페이지 분리 아키텍처, 게임 사용 방법, 그리고 향후 개발 방향성 및 유지보수 가이드를 담고 있습니다.

---

## 1. 프로젝트 개요 (Project Overview)

**Miner & Blacksmith**는 웹 브라우저 환경에서 동작하는 캐주얼 시뮬레이션 RPG 게임입니다.  
플레이어는 광부(Miner)로서 던전을 탐험하며 다양한 광석을 채굴하고, 대장장이(Blacksmith)로서 광석을 제련하여 강력한 장비와 부적을 제작합니다. 더 깊은 던전층으로 내려갈수록 더 희귀한 금속을 얻을 수 있습니다.

* **타겟 환경**: Web Browser (Desktop / Mobile Responsive)
* **주요 특징**:
  * **페이지 분리 아키텍처**: 로그인/랜딩 전용 페이지(`login.html`)와 게임 실행 전용 페이지(`game.html`)가 분리된 깔끔한 UI 구조.
  * **게스트 & 회원가입 지원**: 회원가입 없이 로컬 저장 기반으로 플레이하거나, Supabase 계정 로그인으로 클라우드 멀티 디바이스 연동 가능.
  * **자체 오디오 엔진**: 외부 라이브러리 없이 Web Audio API 기반의 앰비언트 BGM 및 타격 SFX(Dynamic Base64 WAV Fallback 지원) 합성.

---

## 2. 프로젝트 아키텍처 및 기술 스택 (Architecture & Tech Stack)

### 2.1 기술 스택
* **Frontend**: HTML5, Vanilla CSS3 (Custom Variables, Flexbox, CSS Grid, SVG Animation), Vanilla JavaScript (ES6+)
* **Backend / Serverless**: Node.js (Vercel Serverless Function - `/api/config.js`)
* **Database & Auth**: Supabase (Supabase Auth & PostgreSQL - `game_saves` 테이블)
* **Audio Engine**: Web Audio API (합성 BGM) & Base64 Dynamic WAV Generation (Fallback 채광 SFX)
* **Storage**: LocalStorage (`minerBlacksmithSave`, `minerBlacksmithGuest`) + Supabase Cloud Auto-sync

### 2.2 디렉터리 구조 및 파일 역할
```text
miner-blacksmith-game/
├── index.html              # 스마트 라우터 (세션 파악 후 login.html 또는 game.html 이동)
├── login.html              # 로그인 / 회원가입 / 게스트 시작 랜딩 페이지
├── game.html               # 메인 게임 실행 페이지 (프로필 바, 채광, 탐험, 대장간, 상점)
├── api/
│   └── config.js           # Vercel Serverless API (Supabase 환경변수 전달)
└── doc/
    ├── README.md           # 문서 목차 안내
    ├── DEVELOPMENT_GUIDE.md# 시스템 분석, 아키텍처 및 상세 개발 가이드 (본 문서)
    └── IMPLEMENTATION_PLAN.md # 페이지 분리 구현 계획서
```

---

## 3. 핵심 소스 코드 분석 (Core Source Code Analysis)

### 3.1 라우팅 및 세션 관리 (`index.html`, `login.html`, `game.html`)
* **스마트 라우터 (`index.html`)**:
  * 접속 시 Supabase 세션 유효성을 파악하고, 유효 세션 또는 게스트 표식이 있는 경우 `game.html`로 직행하며, 그 외에는 `login.html`로 자동 연결합니다.
* **로그인 랜딩 (`login.html`)**:
  * 이메일/비밀번호 인증(`signInWithPassword`, `signUpWithPassword`)과 **게스트 모드 시작 (`startGuest`)** 기능을 지원합니다.
  * 게스트 시작 시 `minerBlacksmithGuest` 플래그를 로컬 스토리지에 설정하고 `game.html`로 진입합니다.
* **게임 실행 페이지 (`game.html`)**:
  * 상단에 슬림한 **플레이어 프로필 바(`updateUserUI`)**가 위치하여 로그인 계정 이메일 또는 `🎮 게스트 플레이어` 상태를 직관적으로 보여줍니다.

### 3.2 채광 시스템 (Mining System)
* **광석 정보 (`ores`)**:
  * 돌(Stone), 구리(Copper), 철(Iron), 은(Silver), 미스릴(Mithril), 아다만트(Adamant), 오리하르콘(Orichalcum), 성철(Starsteel) 등 8개 티어로 구성.
* **드랍 가중치 엔진 (`weightedOre`)**:
  * 단순 확률이 아닌 `광부 레벨`, `현재 던전 깊이`, `방 위험도`, `곡괭이 희귀 보너스(rareBoost)`를 실시간 반영하여 가중치 계산.
  * 잠긴 광석 티어 대비 플레이어 레벨/채광력이 부족할 경우 페널티 적용.
* **10회 연속 채광 (`mine(10)`)**:
  * 현재 방의 남은 광맥(`vein`) 수를 검사하고 최대 채광 가능 수만큼 연타 가능.

### 3.3 던전 탐험 시스템 (Dungeon System)
* **방 구조 (`roomTypes` & `defaultDungeon`)**:
  * 8가지 유형의 방(광산 입구, 구리빛 갈림길, 푸른 수정 광맥, 별빛 보석 광맥 등).
* **그리드 기반 탐험 (`explore`)**:
  * 동/서/남/북 및 '깊은 길' 이동 지원.
  * 이동 시 던전 깊이(`depth`) 및 방 위험도(`danger`) 상승, 위험도에 따른 낙석 이벤트(골드 손실) 발생.
  * 방문한 위치 좌표(`visited`)를 기록하여 `3x3` 미니맵(`miniMap`) 렌더링.
  * 광맥 고갈 시 대장간 귀환(`campBtn`)을 통해 안전 지역으로 복귀 가능.

### 3.4 대장간 제련 시스템 (Forge System)
* **제작 레시피 (`recipes`)**:
  * 구리 주괴, 철 곡괭이, 은 부적, 미스릴 망치, 아다만트 드릴, 오리하르콘 왕관, 성철 코어.
* **성공률 및 숙련도 계산 (`craft`)**:
  * 성공률 = $\min(95\%, 48\% + \text{smithLevel} \times 4.5\% + \text{mastery} \times 0.3\% + \text{forgeBoost})$.
  * 성공 시: 골드 획득 및 장비 특수 효과 적용 (채광력 상승, 희귀 금속 확률 증가, 제련 성공률 증가).
  * 실패 시: 재료는 소모되나 대장장이 숙련도(`mastery`)가 추가 상승하여 다음 제작 성공률 보정.

### 3.5 사운드 및 오디오 엔진 (Web Audio API & Sound Engine)
* **합성 BGM (`startBgm`)**:
  * `AudioContext`의 OscillatorNode 3개를 튜닝하여 웅장하고 어두운 앰비언트 드론 사운드 생성.
  * BiquadFilterNode 및 LFO(Low Frequency Oscillator)를 연결하여 주파수 변조 효과 구현.
* **합성/폴백 SFX (`playMineSound`, `buildMineSoundWav`)**:
  * Web Audio 지원 시 금속 및 타격음 합성.
  * 미지원 환경을 대비해 Data URI 형식의 Base64 PCM WAV 파일 동적 생성 알고리즘 내장.

### 3.6 데이터 동기화 및 클라우드 (Data Sync & Supabase)
* **로컬 동기화 (`save`, `load`)**:
  * 게임 진행 시 `localStorage`에 JSON 형태 즉시 저장 (`minerBlacksmithSave`).
* **클라우드 동기화 (`initSupabase`, `scheduleCloudSave`, `saveCloudNow`, `loadCloudSave`)**:
  * `/api/config`를 호출하여 Vercel 환경변수로부터 Supabase URL/Key 획득.
  * 로그인 세션이 있을 경우 `game_saves` 테이블에 `user_id`를 PK로 하여 게임 상태 `state`를 디바운스(750ms) 기반 `upsert` 수행.

---

## 4. 플레이 및 사용 방법 (User Guide)

1. **시작 화면 및 진입 (`login.html`)**:
   * 브라우저에서 `index.html` 또는 `login.html` 접속.
   * 이메일과 비밀번호로 로그인/회원가입하거나, **'🎮 게스트 모드로 바로 시작'** 버튼을 누릅니다.

2. **게임 진행 (`game.html`)**:
   * 상단 프로필 바에서 현재 로그인 상태(`👤 계정` 또는 `🎮 게스트`) 확인.
   * `⛏ 채광` 및 `⛏⛏ 10회 채광` 버튼으로 광석을 채굴합니다.

3. **광석 판매 및 상점 이용**:
   * `광석 판매` 버튼으로 1~3티어 광석을 팔아 골드를 확보합니다.
   * `상점` 탭에서 **튼튼한 곡괭이**, **탐광꾼 곡괭이** 등을 구매해 채광력과 희귀율을 높입니다.

4. **던전 탐험**:
   * `방향 이동(동/서/남/북/깊은 길)`으로 던전을 나아갑니다.
   * 광맥 고갈 시 `대장간 귀환`으로 복귀하여 광맥을 다시 리셋합니다.

5. **대장간 제작**:
   * 광석 재료로 다양한 도구 및 왕관/코어 장비를 제작합니다. 성공 시 능력치가 부여됩니다.

6. **저장 및 로그아웃**:
   * **로그인 상태**: 채광/탐험/제작 시 자동으로 클라우드 동기화되며, `클라우드 저장` 버튼으로 즉시 저장할 수 있습니다.
   * **로그아웃 / 시작 화면**: 상단 `시작 화면 / 로그아웃` 버튼 클릭 시 로그인 랜딩 페이지로 안전하게 이동합니다.

---

## 5. 개발 방향성 및 리팩토링 로드맵 (Development Roadmap)

### Phase 1: 아키텍처 및 코드 모듈화 (Short-term)
* [x] **페이지 구조 분리 (완료)**: `login.html`, `game.html`, `index.html` 스마트 라우팅 분리.
* [ ] **JS/CSS 파일 분리**:
  * `css/style.css`: UI/컴포넌트 및 애니메이션 스타일 이관.
  * `src/core/`: `MiningEngine.js`, `ForgeEngine.js`, `DungeonEngine.js` 게임 로직 분리.
  * `src/audio/`: `AudioEngine.js` 오디오 모듈 분리.
* [ ] **Vite + TypeScript 도입**: 타입 안정성 및 모듈 번들링 최적화.

### Phase 2: 게임 콘텐츠 및 시스템 확장 (Mid-term)
* [ ] **던전 몬스터 & 전투 시스템 (Combat System)**:
  * 위험도에 따른 몬스터 인카운터 및 턴제/자동 전투 도입.
* [ ] **퀘스트 및 일일 목표 (Quest System)**:
  * 일일 미션 완수 시 골드 및 특수 칭호 보상.
* [ ] **방치형(Idle) 자동 채광 요소**:
  * 오프라인 또는 슬레이브 광부 채굴 시스템 추가.

### Phase 3: 소셜 및 보안/인프라 강화 (Long-term)
* [ ] **리더보드 (Leaderboard System)**:
  * Supabase `leaderboards` 테이블 연동 후 플레이어 랭킹 표시.
* [ ] **Supabase RLS 보안 강화**:
  * `game_saves` 테이블에 사용자 본인만 접근/수정 가능한 Row Level Security 정책 적용.
* [ ] **PWA (Progressive Web App) 전환**:
  * Service Worker 추가로 모바일 앱 설치 및 오프라인 모드 지원.
