# Miner & Blacksmith - 소스 분석, 사용법 및 개발 방향 문서

이 문서는 **Miner & Blacksmith (광부 & 대장장이)** 웹 게임의 소스 코드 분석 결과, 게임 사용 방법, 그리고 향후 개발 방향성 및 유지보수 가이드를 담고 있습니다.

---

## 1. 프로젝트 개요 (Project Overview)

**Miner & Blacksmith**는 웹 브라우저 환경에서 동작하는 캐주얼 시뮬레이션 RPG 게임입니다.  
플레이어는 광부(Miner)로서 던전을 탐험하며 다양한 광석을 채굴하고, 대장장이(Blacksmith)로서 광석을 제련하여 강력한 장비와 부적을 제작합니다. 더 깊은 던전층으로 내려갈수록 더 희귀한 금속을 얻을 수 있습니다.

* **타겟 환경**: Web Browser (Desktop / Mobile Responsive)
* **주요 특징**: 단일 파일 형태의 가벼운 클라이언트 구조, 외부 종속성이 최소화된 Web Audio 합성 BGM/SFX, Supabase 기반 클라우드 세이브 지원.

---

## 2. 프로젝트 아키텍처 및 기술 스택 (Architecture & Tech Stack)

### 2.1 기술 스택
* **Frontend**: HTML5, Vanilla CSS3 (Custom Variables, Flexbox, CSS Grid, SVG Animation), Vanilla JavaScript (ES6+)
* **Backend / Serverless**: Node.js (Vercel Serverless Function - `/api/config.js`)
* **Database & Auth**: Supabase (Supabase Auth & PostgreSQL - `game_saves` 테이블)
* **Audio Engine**: Web Audio API (합성 BGM) & Base64 Dynamic WAV Generation (Fallback 채광 SFX)
* **Storage**: LocalStorage (`minerBlacksmithSave`) + Supabase Cloud Auto-sync

### 2.2 디렉터리 구조
```
miner-blacksmith-game/
├── api/
│   └── config.js        # Vercel Serverless API (Supabase 환경변수 전달)
├── doc/
│   └── DEVELOPMENT_GUIDE.md # 개발 및 분석 문서 (본 문서)
└── index.html           # 메인 게임 UI, 스타일, 게임 엔진 및 로직 통합 파일
```

---

## 3. 핵심 소스 코드 분석 (Core Source Code Analysis)

### 3.1 채광 시스템 (Mining System)
* **광석 정보 (`ores`)**:
  * 돌(Stone), 구리(Copper), 철(Iron), 은(Silver), 미스릴(Mithril), 아다만트(Adamant), 오리하르콘(Orichalcum), 성철(Starsteel) 등 8개 티어로 구성.
* **드랍 가중치 엔진 (`weightedOre`)**:
  * 단순 확률이 아닌 `광부 레벨`, `현재 던전 깊이`, `방 위험도`, `곡괭이 희귀 보너스(rareBoost)`를 실시간 반영하여 가중치 계산.
  * 잠긴 광석 티어 대비 플레이어 레벨/채광력이 부족할 경우 페널티 적용.
* **10회 연속 채광 (`mine(10)`)**:
  * 현재 방의 남은 광맥(`vein`) 수를 검사하고 최대 채광 가능 수만큼 연타 가능.

### 3.2 던전 탐험 시스템 (Dungeon System)
* **방 구조 (`roomTypes` & `defaultDungeon`)**:
  * 8가지 유형의 방(광산 입구, 구리빛 갈림길, 푸른 수정 광맥, 별빛 보석 광맥 등).
* **그리드 기반 탐험 (`explore`)**:
  * 동/서/남/북 및 '깊은 길' 이동 지원.
  * 이동 시 던전 깊이(`depth`) 및 방 위험도(`danger`) 상승, 위험도에 따른 낙석 이벤트(골드 손실) 발생.
  * 방문한 위치 좌표(`visited`)를 기록하여 `3x3` 미니맵(`miniMap`) 렌더링.
  * 광맥 고갈 시 대장간 귀환(`campBtn`)을 통해 안전 지역으로 복귀 가능.

### 3.3 대장간 제련 시스템 (Forge System)
* **제작 레시피 (`recipes`)**:
  * 구리 주괴, 철 곡괭이, 은 부적, 미스릴 망치, 아다만트 드릴, 오리하르콘 왕관, 성철 코어.
* **성공률 및 숙련도 계산 (`craft`)**:
  * 성공률 = $\min(95\%, 48\% + \text{smithLevel} \times 4.5\% + \text{mastery} \times 0.3\% + \text{forgeBoost})$.
  * 성공 시: 골드 획득 및 장비 특수 효과 적용 (채광력 상승, 희귀 금속 확률 증가, 제련 성공률 증가).
  * 실패 시: 재료는 소모되나 대장장이 숙련도(`mastery`)가 추가 상승하여 다음 제작 성공률 보정.

### 3.4 사운드 및 오디오 엔진 (Web Audio API & Sound Engine)
* **합성 BGM (`startBgm`)**:
  * `AudioContext`의 OscillatorNode 3개를 튜닝하여 웅장하고 어두운 앰비언트 드론 사운드 생성.
  * BiquadFilterNode 및 LFO(Low Frequency Oscillator)를 연결하여 주파수 변조 효과 구현.
* **합성/폴백 SFX (`playMineSound`, `buildMineSoundWav`)**:
  * Web Audio 지원 시 금속 및 타격음 합성.
  * 미지원 환경을 대비해 Data URI 형식의 Base64 PCM WAV 파일 동적 생성 알고리즘 내장.

### 3.5 데이터 동기화 및 클라우드 (Data Sync & Supabase)
* **로컬 동기화 (`save`, `load`)**:
  * 게임 진행 시 `localStorage`에 JSON 형태 즉시 저장.
* **클라우드 동기화 (`initSupabase`, `scheduleCloudSave`, `saveCloudNow`, `loadCloudSave`)**:
  * `/api/config`를 호출하여 Vercel 환경변수로부터 Supabase URL/Key 획득.
  * `https://esm.sh/@supabase/supabase-js@2`를 ES Module Dynamic Import로 로드.
  * 로그인 세션 감지 시 `game_saves` 테이블에 `user_id`를 PK로 하여 게임 상태 `state`를 디바운스(750ms) 기반 `upsert` 수행.

---

## 4. 플레이 및 사용 방법 (User Guide)

1. **게임 시작 및 기본 채광**:
   * 브라우저에서 `index.html`을 실행합니다.
   * `⛏ 채광` 버튼을 클릭하여 돌과 구리를 채굴하고 광부 레벨을 올립니다.
   * 필요시 `⛏⛏ 10회 채광`으로 빠르게 광맥을 소모합니다.

2. **광석 판매 및 상점 이용**:
   * `광석 판매` 버튼으로 1~3티어(돌, 구리, 철) 흔한 광석을 팔아 골드를 수급합니다.
   * `상점` 탭에서 **튼튼한 곡괭이**, **탐광꾼 곡괭이** 등을 구매하여 채광력과 희귀 광석 발견율을 높입니다.

3. **던전 탐험**:
   * `북쪽`, `동쪽`, `서쪽`, `남쪽` 및 `깊은 길` 버튼을 눌러 던전 속으로 들어갑니다.
   * 더 깊은 방일수록 위험도가 높아지지만, 고급 광맥(미스릴, 아다만트, 성철 등)이 등장합니다.
   * 광맥이 고갈되면 `대장간 귀환`을 클릭해 입구로 돌아와 광맥을 리셋합니다.

4. **대장간 제작**:
   * 대장간 패널에서 레시피를 선택하고 `제작` 버튼을 클릭합니다.
   * 대장장이 레벨과 숙련도가 올라갈수록 미스릴 망치, 성철 코어 등 최종 장비 제작 확률이 상승합니다.

5. **클라우드 저장 및 동기화**:
   * 상단 인증 카드에서 이메일과 비밀번호를 입력하고 `회원가입` 또는 `로그인`합니다.
   * 로그인 상태에서는 채광, 탐험, 제작 시 진행 상황이 자동으로 클라우드(`game_saves`)에 저장되어 타 기기에서도 이어할 수 있습니다.

---

## 5. 개발 방향성 및 리팩토링 로드맵 (Development Roadmap)

현재 단일 `index.html`로 구성되어 있어 초기 접근성은 뛰어나지만, 향후 스케일업과 유지보수를 위해 아래 방향으로의 리팩토링 및 기능 확장을 권장합니다.

### Phase 1: 아키텍처 및 코드 모듈화 (Short-term)
* [ ] **단일 파일 분리**:
  * `css/style.css`: UI/컴포넌트 및 애니메이션 스타일 이관.
  * `src/core/`: `MiningEngine.js`, `ForgeEngine.js`, `DungeonEngine.js` 게임 로직 분리.
  * `src/audio/`: `AudioEngine.js` (BGM, SFX 관리자 분리).
  * `src/services/`: `SupabaseService.js` 인증 및 클라우드 세이브 관리.
* [ ] **빌드 도구 도입 (Vite + TypeScript)**:
  * 타입 안정성 확보 (Game State, Recipe, Ore 인터페이스 정의).
  * 번들링 및 자산 최적화.

### Phase 2: 게임 콘텐츠 및 시스템 확장 (Mid-term)
* [ ] **던전 몬스터 & 전투 시스템 (Combat System)**:
  * 던전 탐험 중 위험도에 따라 몬스터(슬라임, 고블린, 락 몬스터 등) 등장.
  * 제작한 무기/방어구를 장착하여 전투를 치르는 턴제 또는 방치형 전투 구현.
* [ ] **퀘스트 및 일일 목표 (Quest System)**:
  * "철 광석 50개 채굴", "은 부적 3개 제작" 등 미션 완수 시 골드 및 특수 칭호 보상.
* [ ] **방치형(Idle) 자동 채광기/용광로**:
  * 접속하지 않아도 광석이 채굴되는 자동 채굴 인프라 추가.

### Phase 3: 소셜 및 보안/인프라 강화 (Long-term)
* [ ] **리더보드 (Leaderboard System)**:
  * Supabase DB에 `leaderboards` 테이블 구축 후 최고 던전 깊이 및 대장장이 레벨 랭킹 표시.
* [ ] **Supabase 보안 정책 (RLS) 강화**:
  * `game_saves` 테이블 RLS(Row Level Security) 적용하여 본인 유저 ID의 데이터만 접근/수정 가능하도록 설정.
* [ ] **PWA (Progressive Web App) 전환**:
  * Service Worker 추가로 모바일 앱처럼 설치 및 완전 오프라인 기능 지원.

---

## 6. 환경 설정 및 배포 가이드 (Setup & Deployment)

### 6.1 Vercel 환경 변수 설정
본 프로젝트를 Vercel에 배포할 경우 다음 환경 변수를 등록해야 클라우드 저장이 활성화됩니다.
* `SUPABASE_URL`: Supabase 프로젝트 URL (예: `https://xxxx.supabase.co`)
* `SUPABASE_ANON_KEY`: Supabase 익명 API 키 (Anon Key)

### 6.2 Supabase 테이블 스키마
Supabase SQL Editor에서 아래 DDL을 실행하여 `game_saves` 테이블을 생성합니다.

```sql
CREATE TABLE public.game_saves (
  user_id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  state JSONB NOT NULL,
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT timezone('utc'::text, now()) NOT NULL
);

-- Row Level Security (RLS) 활성화 권장
ALTER TABLE public.game_saves ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can manage their own save game" 
ON public.game_saves 
FOR ALL 
USING (auth.uid() = user_id) 
WITH CHECK (auth.uid() = user_id);
```
