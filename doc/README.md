# Miner & Blacksmith Documentation

이 디렉터리는 Miner & Blacksmith 프로젝트의 소스 분석, 사용법, 개발 방향 및 구현 계획 문서를 포함합니다.

* 📄 [개발 가이드 및 소스 분석 문서 (DEVELOPMENT_GUIDE.md)](file:///Users/flowerhead/miner-blacksmith-game/doc/DEVELOPMENT_GUIDE.md)
* 📋 [로그인/게임 페이지 분리 구현 계획서 (IMPLEMENTATION_PLAN.md)](file:///Users/flowerhead/miner-blacksmith-game/doc/IMPLEMENTATION_PLAN.md)

---

## 주요 문서 내용 요약
1. **프로젝트 개요 & 기술 스택**: HTML5, Vanilla CSS/JS, Web Audio API, Supabase, Vercel Serverless Function
2. **핵심 시스템 소스 분석**:
   - 채광 및 드랍 가중치 엔진 (`weightedOre`)
   - 3x3 그리드 던전 탐험 및 미니맵 (`explore`)
   - 대장간 제작 및 성공률/숙련도 커브 (`craft`)
   - Web Audio API / Dynamic WAV 오디오 엔진
   - LocalStorage & Supabase Cloud Auto-sync
3. **사용법 및 플레이 가이드**: 채광, 판매/상점, 던전 탐험, 대장간 제작, 클라우드 세이브 사용법
4. **향후 개발 방향 (Roadmap)**:
   - Phase 1: Vite + TypeScript 및 ES Modules 기반 코드 모듈화
   - Phase 2: 몬스터 전투, 퀘스트, Idle 자동 채굴 요소 확장
   - Phase 3: Leaderboard, Supabase RLS, PWA 전환
5. **구현 계획서 요약**:
   - `login.html` (로그인/회원가입/게스트진입) 및 `game.html` (게임플레이) 분리 아키텍처
