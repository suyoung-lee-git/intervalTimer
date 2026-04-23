# 인터벌 타이머 iPhone 웹앱 요건서

## 프로젝트 개요

iPhone Safari에 최적화된 PWA(Progressive Web App) 형태의 인터벌 타이머.  
홈 화면에 추가하여 네이티브 앱처럼 사용 가능.

---

## 기술 스택

- **Frontend**: Vanilla HTML5 + CSS3 + JavaScript (ES6+)
- **빌드 도구**: 없음 (단일 파일 또는 최소 파일 구성)
- **PWA**: Web App Manifest + Service Worker
- **오디오**: Web Audio API (비프음 생성)
- **진동**: Vibration API
- **저장소**: localStorage (설정 영속성)

> 프레임워크 불사용 이유: iPhone Safari 호환성 극대화, 설치 불필요, 파일 크기 최소화

---

## 핵심 기능 요건

### 1. 타이머 설정

| 항목 | 설명 | 기본값 |
|------|------|--------|
| 준비 시간 (Prepare) | 운동 시작 전 카운트다운 | 10초 |
| 운동 시간 (Work) | 고강도 구간 | 40초 |
| 휴식 시간 (Rest) | 저강도 구간 | 20초 |
| 세트 수 (Sets) | 반복 횟수 | 8세트 |
| 라운드 수 (Rounds) | 세트 묶음 반복 | 1라운드 |
| 라운드 간 휴식 | 라운드 사이 휴식 시간 | 60초 |

- 각 항목은 `+` / `-` 버튼으로 조작 (터치 친화적, 최소 44px 터치 영역)
- 숫자 직접 탭하면 인라인 편집 가능
- 설정값은 localStorage에 자동 저장

### 2. 타이머 동작

- **단계 순서**: Prepare → [Work → Rest] × Sets → (라운드 간 휴식 → 반복) × Rounds
- 마지막 세트에는 Rest 생략
- 각 단계 전환 시 **3초 전 카운트다운 비프음** (3회 짧은 비프 → 1회 긴 비프)
- 시작/일시정지/재개/초기화 제어

### 3. 화면 표시

- 현재 단계명 (PREPARE / WORK / REST / ROUND REST / DONE)
- 남은 시간 (대형 숫자, mm:ss 또는 ss 형식)
- 현재 세트 / 전체 세트 (예: `SET 3 / 8`)
- 현재 라운드 / 전체 라운드
- 전체 진행률 프로그레스 바
- 단계별 배경색 구분:
  - Prepare: 파란색 (`#1E90FF`)
  - Work: 빨간색 (`#FF3B30`)
  - Rest: 초록색 (`#34C759`)
  - Round Rest: 주황색 (`#FF9500`)
  - Done: 회색 (`#8E8E93`)

### 4. 알림 / 피드백

- **소리**: Web Audio API로 비프음 생성 (외부 파일 불필요)
  - 짧은 비프: 100ms, 880Hz
  - 긴 비프: 500ms, 440Hz
  - Done 사운드: 3음계 상승
- **진동**: 각 단계 전환 시 진동 패턴 (지원 기기만)
- **화면 켜짐 유지**: Wake Lock API 사용 (미지원 시 graceful fallback)

### 5. PWA 요건

- `manifest.json`: 앱 이름, 아이콘, `display: standalone`, `theme_color`
- `service-worker.js`: 오프라인 캐싱 (Cache First 전략)
- iOS Safari 전용 메타태그:
  ```html
  <meta name="apple-mobile-web-app-capable" content="yes">
  <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
  ```
- 아이콘: 192×192, 512×512 (SVG 기반 PNG 또는 inline SVG)

---

## UI/UX 요건

### 레이아웃

```
┌─────────────────────┐
│   앱 이름 / 설정 버튼  │  ← 상단 헤더
├─────────────────────┤
│                     │
│      WORK           │  ← 단계명 (대형)
│      00:40          │  ← 남은 시간 (초대형)
│   SET 3 / 8         │  ← 세트 정보
│   ROUND 1 / 3       │
│  ████████░░░░ 60%   │  ← 프로그레스 바
│                     │
├─────────────────────┤
│  ⏸ PAUSE   🔄 RESET │  ← 컨트롤 버튼
└─────────────────────┘
```

- 단일 페이지 앱 (설정 화면은 슬라이드업 모달)
- Safe Area 대응 (`env(safe-area-inset-*)`)
- 다크 모드 전용 (운동 중 눈부심 방지)
- 가로 모드 지원

### 접근성

- 버튼 최소 터치 영역 44×44px
- 색상 외 텍스트로도 단계 구분 가능
- 화면 꺼짐 방지로 인한 배터리 소모 사용자 고지

---

## 파일 구조

```
intervalTimer/
├── index.html          # 메인 앱 (인라인 CSS/JS 허용)
├── manifest.json       # PWA 매니페스트
├── service-worker.js   # 오프라인 캐싱
├── icon-192.png        # PWA 아이콘
├── icon-512.png        # PWA 아이콘
└── REQUIREMENTS.md     # 본 문서
```

> `index.html` 하나에 CSS와 JS를 인라인으로 포함해도 무방 (파일 수 최소화)

---

## 브라우저 호환성

| 브라우저 | 버전 | 필수 |
|---------|------|------|
| iOS Safari | 15+ | ✅ 주요 타깃 |
| Chrome for iOS | 최신 | ✅ |
| Android Chrome | 90+ | ✅ |
| Desktop Chrome/Firefox | 최신 | 선택 |

- iOS 15 미만에서 Wake Lock 미지원 → 화면 dim 방지 fallback (주기적 noop touch 이벤트)
- Vibration API iOS Safari 미지원 → 소리로 대체

---

## 구현 우선순위

1. **P0** — 핵심 타이머 로직 (단계 전환, 카운트다운)
2. **P0** — 기본 UI (시간 표시, 단계 표시, 컨트롤)
3. **P1** — 설정 화면 (시간/세트 수 조정)
4. **P1** — 소리 피드백 (Web Audio API)
5. **P1** — localStorage 설정 저장
6. **P2** — PWA 매니페스트 + Service Worker
7. **P2** — Wake Lock API
8. **P3** — 진동 피드백
9. **P3** — 라운드 기능

---

## 제약사항 및 주의사항

- iOS Safari는 백그라운드 탭에서 `setInterval` 스로틀링 발생 → `visibilitychange` 이벤트로 경과 시간 보정 필요
- Web Audio API는 사용자 제스처 이후에만 초기화 가능 (iOS 정책)
- PWA 설치 후 홈 화면 실행 시 `standalone` 모드에서 URL 바 없음 → 전체 화면 활용
- `<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">` 필수
