<project_specification>

<project_name>Kakao travel - 여행 맥락 번역 모바일 웹</project_name>

<overview>
Kakao travel은 유럽 여행 중 스마트폰 브라우저에서 최소 클릭으로 음성 대화 번역과 카메라 번역을 제공하는 모바일 친화 웹사이트입니다.

핵심 기능은 두 가지입니다:
1. 턴 기반 음성 대화 — 탭 한 번으로 말하고, 번역 결과를 큰 글씨로 상대방에게 보여줌 (식당 주문, 호텔 체크인, 길 묻기). TTS는 젊은 여성 목소리로 자연스럽게 재생.
2. 카메라 번역 — 메뉴판/표지판을 촬영하면 단순 직역이 아닌 맥락 해석 제공 (음식 설명, 재료, 행동 가이드)

CRITICAL: 서버 없음. Gemini API를 클라이언트에서 직접 호출. API 키는 localStorage에 저장.
CRITICAL: 모바일 브라우저(iOS Safari, Android Chrome) 우선. 별도 설치 없이 URL 접속 즉시 사용 가능. 모바일 화면비/세로 모드에 최적화된 반응형 레이아웃.
CRITICAL: 최소 UI. 페이지 열면 바로 마이크 또는 카메라. 메뉴 탐색 없이 1-2탭으로 결과까지.
CRITICAL: TTS 음성은 일관되게 "젊은 여성(young female)" 보이스로 재생. 시스템 보이스 목록에서 언어별 여성 보이스를 자동 선택하고, 발화 톤은 약간 밝고 또렷하게 (rate 1.0, pitch 1.05~1.15).
</overview>

<scope_boundaries>
  <in_scope>
    - 턴 기반 음성 대화 번역 (한국어 ↔ 영어/독일어/프랑스어/이탈리아어)
    - 카메라 촬영 → Gemini Vision → 맥락 번역
    - 번역 결과 큰 글씨 전체화면 표시 (상대방에게 보여주기)
    - TTS 자동 재생 (젊은 여성 목소리)
    - 모바일 우선 반응형 레이아웃 (iOS Safari, Android Chrome)
    - API 키 설정 (localStorage)
    - 대화 히스토리 (세션 내)
  </in_scope>
  <out_of_scope>
    - 서버/백엔드
    - 사용자 계정/인증
    - 오프라인 번역
    - 동시통역 (실시간 스트리밍)
    - 다국어 UI (UI는 한국어 고정)
    - PWA 설치 / 홈 화면 추가 / Service Worker 오프라인 캐시
    - 데스크탑 전용 UI (모바일 우선이며, 데스크탑은 모바일 폭 컨테이너로 표시)
  </out_of_scope>
  <future_considerations>
    - 자주 쓰는 표현 즐겨찾기 (Phase 2)
    - 상황별 프롬프트 자동 전환: 식당/교통/숙박 (Phase 2)
    - 번역 히스토리 영속 저장 (Phase 3)
    - 다른 언어 추가 (Phase 3)
  </future_considerations>
</scope_boundaries>

<technology_stack>
  <frontend>
    <framework>Vanilla TypeScript — 프레임워크 없음, 빠른 개발과 경량화</framework>
    <build_tool>Vite v6</build_tool>
    <styling>단일 CSS 파일, CSS 변수 기반 Kakao 디자인 토큰</styling>
  </frontend>
  <apis>
    <gemini>Google Gemini API (gemini-2.0-flash) — 텍스트 번역 + Vision 이미지 해석</gemini>
    <web_speech_stt>Web Speech API SpeechRecognition — 음성→텍스트</web_speech_stt>
    <web_speech_tts>Web Speech API SpeechSynthesis — 텍스트→음성. 젊은 여성 보이스를 우선 선택(아래 voice_selection 정책 참고).</web_speech_tts>
  </apis>
  <mobile_web>
    <viewport>viewport=device-width, initial-scale=1, viewport-fit=cover, maximum-scale=1 (확대 잠금으로 입력 시 폰트 점프 방지)</viewport>
    <responsive>모바일 우선. max-width 480px 컨테이너 + 데스크탑은 좌우 검은 여백으로 모바일 폭 유지. 가로 모드는 그대로 동작하되 디자인 최적화 대상은 세로 모드.</responsive>
    <safe_area>env(safe-area-inset-*) 모든 가장자리 요소에 적용 (노치/Dynamic Island/홈 인디케이터)</safe_area>
    <theme_color>meta name="theme-color" content="#0d1117" — 모바일 브라우저 상단 바 색상 통일</theme_color>
    <favicon>SVG 파비콘 + 192/512 PNG (브라우저 탭/북마크용, 홈 화면 추가는 out_of_scope)</favicon>
  </mobile_web>
  <hosting>GitHub Pages 또는 Vercel — 무료, HTTPS 필수 (Web Speech / 카메라 / 마이크 권한이 secure context를 요구)</hosting>
  <note>CRITICAL: 서버 없음. 모든 API 호출은 클라이언트에서 직접.</note>
  <note>CRITICAL: Service Worker / manifest.json / 홈 화면 추가 / 오프라인 캐시는 사용하지 않는다. 일반 모바일 웹사이트로만 동작.</note>
</technology_stack>

<file_structure>
Kakao-travel/
├── index.html                 # 페이지 진입점 (모바일 메타/뷰포트/theme-color 포함)
├── src/
│   ├── main.ts                # 앱 초기화, 뷰 라우팅
│   ├── views/
│   │   ├── conversation.ts    # 음성 대화 뷰
│   │   ├── camera.ts          # 카메라 번역 뷰
│   │   └── settings.ts        # API 키 설정 뷰
│   ├── services/
│   │   ├── gemini.ts          # Gemini API 호출 (텍스트 + Vision)
│   │   ├── speech.ts          # STT + TTS 래퍼 (젊은 여성 보이스 선택 로직 포함)
│   │   └── storage.ts         # localStorage 관리 (API 키, 설정)
│   ├── utils/
│   │   ├── prompts.ts         # 맥락 번역 프롬프트 템플릿
│   │   ├── languages.ts       # 언어 코드, 표시명, Web Speech 매핑
│   │   └── voices.ts          # 언어별 젊은 여성 보이스 선택/캐시 유틸
│   └── styles/
│       └── main.css           # 전체 스타일 (Kakao 디자인 토큰 + 모바일 우선 반응형)
├── public/
│   └── favicon/               # 파비콘 + 192/512 PNG (브라우저 탭/북마크용, 홈 화면 추가 미지원)
├── tests/
│   ├── gemini.test.ts         # 번역 프롬프트 + 응답 파싱 테스트
│   ├── prompts.test.ts        # 맥락 프롬프트 생성 테스트
│   └── fixtures/
│       └── sample-translations.json  # 더미 번역 데이터
├── vite.config.ts
├── tsconfig.json
├── package.json
├── PLAN.md                    # 프로젝트 계획 (기존)
├── PROJECT_PLAYBOOK.md        # 진행 노하우 레퍼런스 (기존)
└── README.md
</file_structure>

<core_data_entities>
  <conversation_message>
    - id: string (uuid)
    - role: enum (user, partner)
    - originalText: string (음성 인식 결과)
    - translatedText: string (번역 결과)
    - fromLang: string (ko, en, de, fr, it)
    - toLang: string (ko, en, de, fr, it)
    - timestamp: number (Date.now())
  </conversation_message>

  <camera_result>
    - id: string (uuid)
    - imageDataUrl: string (base64 촬영 이미지)
    - detectedText: string (OCR 추출 텍스트)
    - translation: string (맥락 번역 결과)
    - context: string (맥락 설명 — 음식 설명, 행동 가이드 등)
    - detectedLang: string (감지된 언어)
    - timestamp: number
  </camera_result>

  <app_settings>
    - geminiApiKey: string (Gemini API 키)
    - partnerLang: string (상대방 언어, 기본값 en)
    - ttsEnabled: boolean (TTS 자동 재생, 기본값 true)
    - ttsVoiceProfile: enum (young_female_default — 현재 단일 프리셋. 향후 확장 대비 필드 보유)
    - ttsRate: number (기본값 1.0)
    - ttsPitch: number (기본값 1.1 — 약간 밝고 또렷한 젊은 여성 톤)
  </app_settings>
</core_data_entities>

<pages_and_interfaces>
  <navigation>
    하단 탭 바, 2개 탭 + 설정 아이콘. 높이 56px + safe-area-inset-bottom, 배경 #0d1117.
    - 대화 (마이크 아이콘 + "대화") — 기본 탭
    - 카메라 (카메라 아이콘 + "카메라")
    - 설정 (톱니바퀴 아이콘, 우측 상단 고정) — 탭 바에서 분리, 앱 상단 우측에 작은 아이콘으로 배치
    활성 탭: #7ee787 아이콘 + 라벨. 비활성: #8b949e.
    탭 간 좌우 스와이프 제스처 지원 (대화 ↔ 카메라).
    CRITICAL: safe-area-inset 적용 — iPhone 하단 홈 인디케이터 영역 확보.
  </navigation>

  <conversation_view>
    앱의 메인 화면. 열자마자 바로 사용 가능.

    <layout>
      상단: 언어 표시 바 + 설정 아이콘
      중앙: 대화 히스토리 (스크롤, 최신이 아래, 자동 스크롤)
      하단: 듀얼 마이크 버튼 영역
    </layout>

    <language_bar>
      높이 48px, 배경 #161b22, 좌우 패딩 16px.
      왼쪽: "한국어" (고정, 16px #e6edf3)
      중앙: 양방향 화살표 ↔ (20px, #8b949e)
      오른쪽: 상대방 언어 표시 (탭하면 언어 선택 바텀시트 열림)
        - 현재 언어 표시 + 드롭다운 화살표 ▾ (16px, #7ee787)
      우측 끝: 설정 톱니바퀴 아이콘 (24px, #8b949e, 탭하면 설정 바텀시트)
    </language_bar>

    <language_picker_sheet>
      바텀시트 (하단에서 올라옴, 배경 #161b22, 라운드 상단 16px).
      높이: 컨텐츠 fit (약 280px). 배경 딤: rgba(0,0,0,0.5).
      드래그 핸들: 상단 중앙, 36px × 4px, 배경 #30363d, 라운드 2px.
      언어 목록 (세로 배열, 각 항목 56px 높이, 풀 너비):
        - English 🇬🇧
        - Deutsch 🇩🇪
        - Français 🇫🇷
        - Italiano 🇮🇹
      선택된 언어: 우측에 ✓ 체크마크 #7ee787, 배경 #21262d.
      미선택: 텍스트 #e6edf3, 배경 투명.
      국기 이모지로 시각적 구분 → 여행 중 빠른 인지.
      시트 바깥 탭 또는 아래로 스와이프 → 닫기.
    </language_picker_sheet>

    <message_bubbles>
      user(나) 메시지: 오른쪽 정렬, 배경 #1a2332, 테두리 1px #7ee787
        - 라벨: "나" 12px #7ee787 (버블 상단)
        - 원문 (한국어): 14px, #e6edf3
        - 번역문: 18px bold, #7ee787
        - 타임스탬프: 11px #8b949e (버블 하단 우측)
      partner(상대) 메시지: 왼쪽 정렬, 배경 #21262d, 테두리 1px #30363d
        - 라벨: "상대" + 국기이모지 12px #8b949e (버블 상단)
        - 원문 (외국어): 14px, #8b949e
        - 번역문: 18px bold, #e6edf3
        - 타임스탬프: 11px #8b949e (버블 하단 좌측)
      버블 패딩: 12px 16px, 라운드: 16px, 최대폭 85%, 간격 12px.
      새 메시지 추가 시 부드러운 slide-up 애니메이션 (200ms ease-out).
      롱프레스 시 컨텍스트 메뉴:
        - "다시 읽기" (TTS 재생)
        - "전체화면으로 보기" (전체화면 결과 다시 표시)
        - "텍스트 복사" (번역문 클립보드 복사)
      햅틱: 롱프레스 시 UIImpactFeedbackGenerator medium.
    </message_bubbles>

    <dual_mic_area>
      CRITICAL: 기존 단일 마이크 → 듀얼 버튼으로 변경. 턴 혼동 제거.
      높이 140px, 배경 그라데이션 (transparent → #0d1117), 하단 고정.

      좌측 버튼 — "나" (한국어로 말하기):
        56px 원, 배경 #7ee787, 아이콘 마이크 #0d1117.
        하단 라벨: "한국어" 12px #8b949e.
        탭 → 한국어 STT 시작, 버튼 빨강 #f85149 + 펄스.

      우측 버튼 — "상대" (상대 언어로 말하기):
        56px 원, 배경 #58a6ff (블루 액센트), 아이콘 마이크 #0d1117.
        하단 라벨: 상대 언어명 (예: "English") 12px #8b949e.
        탭 → 상대 언어 STT 시작, 버튼 빨강 #f85149 + 펄스.

      중앙 구분: "또는" 텍스트 없이, 두 버튼 사이 간격 48px.
      두 버튼 사이 상단에 안내 텍스트: "누가 말하나요?" 13px #8b949e (첫 사용 시만, 이후 숨김).
      녹음 중 상태: 활성 버튼만 빨강 + 펄스, 비활성 버튼 opacity 0.3.
      녹음 중 텍스트: 버튼 위에 "듣고 있어요..." 14px #f85149 + 실시간 음성 파형 바 (5개 bar, 높이 변동 애니메이션).

      텍스트 입력 폴백:
        두 버튼 아래에 키보드 아이콘 (20px, #30363d).
        탭하면 텍스트 입력 모드 전환:
          - 텍스트 입력 필드 (배경 #161b22, 라운드 24px, 높이 44px, 플레이스홀더 "직접 입력...")
          - 전송 버튼 (화살표 아이콘, #7ee787)
          - 언어 방향 토글: "한→영" / "영→한" 작은 칩 (탭하여 전환)
          - 마이크 아이콘 탭하면 음성 모드 복귀
      햅틱: 녹음 시작/종료 시 UIImpactFeedbackGenerator light.
    </dual_mic_area>

    <fullscreen_result>
      번역 완료 시 자동으로 전체화면 오버레이 표시 (상대방에게 보여주기용).
      진입 애니메이션: scale 0.95→1.0 + fade-in 250ms ease-out.
      배경 #0d1117.

      상단 바 (48px):
        좌측: 닫기(✕) 버튼 32px, #8b949e
        우측: TTS 재생 버튼 (스피커 아이콘 32px, #7ee787) — 탭하면 다시 읽기
        TTS 재생 중: 스피커 아이콘에 파동 애니메이션

      중앙: 번역문을 화면 중앙에 최대한 크게 표시.
        폰트: 자동 크기 조정 (viewport 기준, 최소 28px ~ 최대 64px).
        색상: #e6edf3.
        번역문 위에 언어 방향 칩: "한국어 → English" 14px, 배경 #21262d, 라운드 12px, 패딩 4px 12px.

      하단 영역:
        원문 표시: 16px, #8b949e, 최대 3줄 (말줄임).
        "탭하여 닫기" 힌트: 12px #30363d (3초 후 fade-out).

      제스처:
        - 화면 아무 곳 탭하면 닫힘 → 대화 뷰로 복귀.
        - 아래로 스와이프해도 닫힘 (iOS 네이티브 dismiss 느낌).
      TTS 자동 재생 (설정에서 on/off).
      화면 자동 잠금 방지: 전체화면 표시 중 wakeLock API 요청 (상대방이 읽는 동안 화면 꺼짐 방지).
    </fullscreen_result>

    <loading_state>
      번역 처리 중:
        마이크 버튼 영역에 스켈레톤 버블 표시 (배경 #21262d, 라운드 16px, shimmer 애니메이션).
        버블 내 3개 dot 로딩 (bounce 애니메이션, #7ee787).
        "번역 중..." 텍스트 14px #8b949e.
    </loading_state>

    <empty_state>
      대화 히스토리가 없을 때:
      중앙 영역 (상하 중앙 정렬):
        마이크 아이콘 (48px, #30363d, bounce 애니메이션 — 3초 간격 subtle bounce).
        "아래 버튼을 눌러 대화를 시작하세요" 16px #8b949e.
        부가 안내: "왼쪽=내가 말하기 · 오른쪽=상대가 말하기" 13px #30363d.
    </empty_state>
  </conversation_view>

  <camera_view>
    <layout>
      전체 화면 카메라 프리뷰.
      하단: 촬영 버튼 영역 (safe-area 포함).
      상단: 반투명 그라데이션 바 (상단→투명, 높이 80px).
    </layout>

    <top_bar>
      상단 좌측: 뒤로가기(✕) 버튼 (32px, 배경 rgba(0,0,0,0.4) 원, 아이콘 #e6edf3).
      상단 우측: 플래시 토글 버튼 (32px, 같은 스타일, 아이콘 번개⚡).
        - on: 아이콘 #f0e68c (노란색), off: 아이콘 #e6edf3.
    </top_bar>

    <capture_area>
      하단 중앙: 촬영 버튼 (72px 원, 테두리 3px #7ee787, 내부 원 56px #e6edf3).
      하단 좌측: 갤러리 버튼 (44px 원, 배경 #21262d, 아이콘 사진 #e6edf3).
        - 탭하면 사진 라이브러리에서 이미지 선택 (input type=file, accept=image/*).
        - 이미 찍은 메뉴판/표지판 사진도 번역 가능.
      하단 우측: (빈 공간 — 좌우 대칭용)
      촬영 버튼 탭 시 햅틱: UIImpactFeedbackGenerator medium.
    </capture_area>

    <viewfinder_guide>
      카메라 프리뷰 중앙에 가이드 프레임 (점선 테두리, 라운드 12px, #7ee78740).
      프레임 하단 텍스트: "메뉴판이나 표지판을 프레임 안에 맞춰주세요" 13px #8b949e.
      촬영 시 프레임이 잠깐 #7ee787 실선으로 변경 (200ms) → 촬영 확인 피드백.
    </viewfinder_guide>

    <capture_flow>
      1. 카메라 탭 진입 → 후면 카메라 프리뷰 자동 시작
      2. 촬영 버튼 탭 → 이미지 캡처 + 셔터 애니메이션 (화면 잠깐 하얗게 flash 100ms)
      3. 프리뷰 freeze + 로딩 오버레이
      4. Gemini Vision 호출 → 결과 표시
    </capture_flow>

    <loading_overlay>
      촬영 이미지 위에 반투명 배경 rgba(13,17,23,0.7).
      중앙: 로딩 애니메이션 (원형 스피너 32px #7ee787 + "분석 중..." 16px #e6edf3).
      하단에 "취소" 버튼 (텍스트만, 14px #8b949e) — API 호출 abort 가능.
      예상 소요 표시: "보통 3~5초 걸려요" 13px #30363d (스피너 아래).
    </loading_overlay>

    <result_overlay>
      촬영 이미지 위에 반투명 오버레이.
      진입: 하단에서 slide-up 300ms ease-out (바텀시트 느낌).
      드래그 핸들: 상단 중앙 36px × 4px #30363d.

      상단: 감지된 언어 + 항목 수 ("Deutsch · 5개 항목 감지") 14px #8b949e.

      중앙: 맥락 번역 결과 (스크롤 가능).
        - 항목별 카드 (배경 #161b22, 라운드 12px, 패딩 16px, 간격 12px)
        - 원문: 14px #8b949e (상단)
        - 번역: 20px bold #e6edf3 (중앙)
        - 맥락 설명: 14px #7ee787 (하단, 음식이면 재료/특징/알레르기 주의, 표지판이면 행동 가이드)
        - 카드 우측 상단: 작은 TTS 아이콘 (16px, #8b949e) — 탭하면 해당 항목만 TTS 재생
        각 카드 탭 시 카드 확대 애니메이션 (살짝 scale 1.02, 200ms) → 가독성 확인.

      하단 버튼 영역 (간격 12px, 가로 배열):
        "다시 촬영" 버튼 (flex 1, 높이 48px, 배경 #21262d, 라운드 12px, 텍스트 #e6edf3)
        "닫기" 버튼 (flex 1, 높이 48px, 배경 #7ee787, 라운드 12px, 텍스트 #0d1117)
    </result_overlay>

    <empty_state>
      카메라 권한 거부 시:
        중앙 아이콘: 카메라에 빗금 (48px, #f85149).
        "카메라 접근 권한이 필요합니다" 18px #e6edf3.
        "Safari 설정 → 이 웹사이트 → 카메라 허용" 14px #8b949e.
        하단: "갤러리에서 선택" 버튼 (카메라 없이도 사진 번역 가능).
    </empty_state>
  </camera_view>

  <settings_view>
    CRITICAL: 탭에서 분리 → 바텀시트로 변경. 자주 접근하지 않는 설정을 탭에서 빼서 대화/카메라에 공간 확보.

    <layout>
      바텀시트 (하단에서 올라옴, 배경 #0d1117, 라운드 상단 16px).
      높이: 화면 60%. 배경 딤: rgba(0,0,0,0.5).
      드래그 핸들: 상단 중앙 36px × 4px #30363d.
      제목: "설정" 18px bold #e6edf3, 좌측 정렬, 패딩 16px 16px 8px.
      닫기: 우측 상단 ✕ 버튼 24px #8b949e.
      아래로 스와이프 또는 딤 영역 탭 → 닫기.
    </layout>

    <fields>
      섹션 간 구분선 (1px #21262d, 좌우 16px 마진).

      - Gemini API Key:
        라벨 "API 키" 14px #8b949e
        입력 필드: 배경 #161b22, 테두리 1px #30363d, 라운드 12px, 높이 48px, 패딩 0 16px
        포커스 시: 테두리 #7ee787, 외곽 glow (0 0 0 2px #7ee78733)
        type=password, 우측에 눈 아이콘 (탭하면 표시/숨김 토글)
        입력 필드 하단: 상태 메시지 + 저장 버튼 (pill shape, 배경 #7ee787, 텍스트 #0d1117, 높이 36px)

      - 상대방 언어:
        라벨 "상대방 언어" 14px #8b949e
        4개 버튼 가로 배열 (flex, 간격 8px, 높이 44px, 라운드 12px)
        각 버튼에 국기 이모지 + 언어명:
          🇬🇧 English / 🇩🇪 Deutsch / 🇫🇷 Français / 🇮🇹 Italiano
        활성: 배경 #7ee787, 텍스트 #0d1117, font-weight bold
        비활성: 배경 #21262d, 텍스트 #8b949e
        전환 시 햅틱: UIImpactFeedbackGenerator light.

      - TTS 자동 재생:
        라벨 "번역 후 자동 읽기" 14px #e6edf3 (좌측)
        보조 라벨 "젊은 여성 목소리" 12px #8b949e (라벨 하단, 현재 보이스 프로필 표기)
        토글 스위치 (우측, 너비 48px, 높이 28px)
          on: 배경 #7ee787, 원 #ffffff
          off: 배경 #30363d, 원 #8b949e
        전환 시 햅틱: UIImpactFeedbackGenerator light.
        토글 우측에 작은 ▶︎ 아이콘 (탭하면 현재 상대 언어로 짧은 샘플 문장을 젊은 여성 보이스로 재생 — "Hello, this is a sample voice." 등).
    </fields>

    <api_key_status>
      키 미설정: ⚠️ "API 키를 입력해주세요" 14px #f85149
      키 설정됨: ✓ "API 키 설정 완료" 14px #7ee787
      키 검증 중: 스피너 + "확인 중..." 14px #8b949e
      키 검증 실패: ✕ "API 키가 유효하지 않습니다" 14px #f85149
    </api_key_status>

    <app_info>
      시트 하단:
        앱 버전 ("Kakao travel v1.0") 12px #30363d, 중앙 정렬.
    </app_info>
  </settings_view>

  <onboarding_flow>
    최초 실행 시 (API 키 미설정 상태):
    전체화면 온보딩 오버레이 (배경 #0d1117).

    Step 1: 환영
      중앙 로고 + "Kakao travel" 24px bold #e6edf3.
      "여행 중 대화와 메뉴 번역을 도와드려요" 16px #8b949e.
      "시작하기" 버튼 (풀 너비 - 32px, 높이 52px, 배경 #7ee787, 라운드 12px, 텍스트 #0d1117 bold).

    Step 2: API 키 입력
      "Gemini API 키를 입력해주세요" 18px #e6edf3.
      API 키 입력 필드 (설정과 동일 스타일).
      "API 키 받기" 텍스트 링크 14px #58a6ff (Google AI Studio로 이동).
      "완료" 버튼.
      "나중에" 텍스트 버튼 12px #8b949e (스킵 가능, 대화 탭 진입 시 다시 안내).

    Step 인디케이터: 하단 2개 dot (현재: #7ee787, 비활성: #30363d).
  </onboarding_flow>

  <toast_notifications>
    화면 상단에서 slide-down, 자동 3초 후 fade-out.
    배경 #161b22, 테두리 1px #30363d, 라운드 12px, 패딩 12px 16px.
    좌측 아이콘 + 메시지 텍스트.
    종류:
      - 성공: ✓ 아이콘 #7ee787 (예: "설정 저장됨")
      - 에러: ✕ 아이콘 #f85149 (예: "번역 실패")
      - 정보: ℹ 아이콘 #58a6ff (예: "텍스트가 복사되었습니다")
    safe-area-inset-top 적용.
  </toast_notifications>
</pages_and_interfaces>

<core_functionality>
  <voice_conversation>
    듀얼 마이크 기반 흐름:

    "나" 버튼 탭 시:
      1. SpeechRecognition 시작 (lang="ko-KR")
      2. "나" 버튼 빨강 + 펄스 + 파형 표시, "상대" 버튼 비활성
      3. 음성 인식 결과 (한국어 텍스트) 수신
      4. Gemini API로 한국어 → 상대 언어 번역 요청
      5. 대화 버블 추가 (우측, 초록 테두리)
      6. 전체화면 결과 표시 (상대방에게 보여주기) — 상대 언어 번역문 크게
      7. TTS로 상대 언어 발음 재생 — 젊은 여성 보이스 (voice_selection 정책 적용)

    "상대" 버튼 탭 시:
      1. SpeechRecognition 시작 (lang=상대 언어 코드)
      2. "상대" 버튼 빨강 + 펄스 + 파형 표시, "나" 버튼 비활성
      3. 음성 인식 결과 (외국어 텍스트) 수신
      4. Gemini API로 상대 언어 → 한국어 번역 요청
      5. 대화 버블 추가 (좌측, 회색 테두리)
      6. 전체화면 결과 표시 안 함 (나만 보면 되니까 버블로 충분)
      7. TTS로 한국어 발음 재생 (나에게 들려주기) — 한국어 젊은 여성 보이스

    텍스트 입력 모드:
      1. 키보드 아이콘 탭 → 텍스트 입력 필드 표시
      2. 언어 방향 칩으로 번역 방향 선택 (한→상대 / 상대→한)
      3. 텍스트 입력 + 전송 → Gemini API 번역 → 버블 추가
      4. 전체화면 결과 표시 (한→상대 방향일 때만)

    언어 방향:
    - "나" 버튼: 한국어 → 상대 언어 (en/de/fr/it)
    - "상대" 버튼: 상대 언어 → 한국어
    - 버튼 선택으로 방향 결정 → 별도 언어 감지 불필요

    CRITICAL: 한 번에 한 명만 말함 (턴 기반). 동시 인식 없음.
    CRITICAL: 녹음 중 다른 버튼 탭 시 현재 녹음 중지 후 새 녹음 시작 (빠른 전환).
  </voice_conversation>

  <camera_translation>
    1. 카메라 프리뷰 시작 (navigator.mediaDevices.getUserMedia, facingMode: environment)
    2. 촬영 버튼 → canvas에 캡처 → base64 변환
    3. Gemini Vision API 호출 (이미지 + 맥락 프롬프트)
    4. 응답 파싱 → 항목별 카드 표시

    맥락 프롬프트:
    "유럽 여행 중인 한국인이 촬영한 사진입니다.
     텍스트를 감지하고 한국어로 번역해주세요.
     메뉴판이면 각 항목의 재료와 특징을 설명해주세요.
     표지판이면 어떤 행동을 해야 하는지 안내해주세요.
     JSON 형식으로 응답: [{original, translation, context}]"
  </camera_translation>

  <translation_prompts>
    음성 대화용 프롬프트:
    "여행 중 대화 번역입니다. 자연스럽고 공손한 표현으로 번역해주세요.
     문화적 맥락이 필요하면 괄호 안에 짧게 추가해주세요.
     원문: {text}
     {fromLang} → {toLang}"
  </translation_prompts>

  <voice_selection>
    CRITICAL: TTS는 모든 언어에서 "젊은 여성(young female)" 보이스를 우선 선택한다. 시스템이 제공하는 SpeechSynthesisVoice 목록은 OS/브라우저마다 다르므로, 휴리스틱 매칭으로 가장 가까운 보이스를 고른다.

    선택 알고리즘 (utils/voices.ts):
      1. window.speechSynthesis.getVoices() 결과를 lang prefix로 필터 (ko-KR, en-US, de-DE, fr-FR, it-IT 순으로 fallback).
         - voiceschanged 이벤트 후 1회 캐시. iOS Safari는 첫 호출 시 빈 배열 반환되는 경우가 있어 voiceschanged + 200ms 폴링 한 번 보강.
      2. 화이트리스트 우선 매칭 (이름 부분일치, 대소문자 무시):
         - ko-KR: "Yuna", "Heami", "SoYoung", "Sora", "Heera"
         - en-US/en-GB: "Samantha", "Ava", "Allison", "Joanna", "Salli", "Google US English" (en-US 기본 보이스가 여성)
         - de-DE: "Anna", "Petra", "Marlene", "Helena"
         - fr-FR: "Amelie", "Audrey", "Marie", "Celine"
         - it-IT: "Alice", "Federica", "Carla"
      3. 화이트리스트 미매칭 시 보이스 이름에 "female" 또는 OS가 제공하는 gender 메타가 female인 항목 선택.
      4. 그래도 없으면 해당 언어 기본 보이스 사용 + 톤 보정으로 여성 인상 보강 (pitch 1.15, rate 1.0).
      5. 최종 선택 보이스를 언어별로 메모리 캐시 (서비스 재시작 시 재계산).

    발화 파라미터:
      - rate: 1.0 (기본). 사용자 설정에서 0.9~1.1 범위 조정 가능 (Phase 2).
      - pitch: 1.10 ko-KR / 1.10 en / 1.08 de / 1.10 fr / 1.10 it — 너무 어린 인상 방지 위해 1.2 초과 금지.
      - volume: 1.0.

    검증 & 폴백:
      - 보이스 미지원/재생 실패 시 toast "음성을 재생할 수 없어요. 화면 텍스트를 확인해주세요." (info)
      - speak() 호출 직전 cancel()로 큐 비우기 (이전 발화 중첩 방지).
      - 사용자 첫 탭 이후에만 speak() 호출 (모바일 자동재생 정책 대응) — 자동 재생도 사용자 탭 트리거된 번역 완료 시점이므로 OK.

    UI 노출:
      - 설정 시트 TTS 토글 하단에 "젊은 여성 목소리" 보조 라벨 표시.
      - 설정 시트 ▶︎ 샘플 재생으로 사용자가 즉시 음성 톤 확인 가능.

    테스트 fixture:
      - utils/voices.test.ts: 가짜 SpeechSynthesisVoice 배열을 주입하여 화이트리스트 우선순위와 fallback 동작을 검증.
  </voice_selection>
</core_functionality>

<error_handling>
  <api_errors>
    - API 키 미설정: 설정 화면으로 자동 이동 + "API 키를 먼저 설정해주세요" 표시
    - API 호출 실패: 대화 버블에 "번역 실패. 다시 시도해주세요." 빨강 텍스트
    - 네트워크 없음: 상단 배너 "인터넷 연결을 확인해주세요" 배경 #f8514933
    - 재시도: 실패 시 자동 재시도 1회. 2회 연속 실패 시 사용자에게 표시.
  </api_errors>
  <permission_errors>
    - 마이크 권한 거부: "마이크 접근 권한이 필요합니다" + Safari 설정 안내
    - 카메라 권한 거부: "카메라 접근 권한이 필요합니다" + Safari 설정 안내
  </permission_errors>
  <speech_errors>
    - STT 인식 실패 (no-speech): "음성이 감지되지 않았습니다. 다시 말씀해주세요."
    - STT 인식 실패 (network): "음성 인식 서비스에 연결할 수 없습니다."
  </speech_errors>
</error_handling>

<third_party_integrations>
  <integration name="Gemini API">
    <purpose>텍스트 번역 + 이미지 맥락 해석</purpose>
    <endpoint>https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent</endpoint>
    <auth>API key (query parameter 또는 header)</auth>
    <features>
      - generateContent: 텍스트 번역 (대화 모드)
      - generateContent + inlineData: 이미지 해석 (카메라 모드)
    </features>
    <rate_limit>분당 15회 호출 상한 (클라이언트 circuit breaker)</rate_limit>
  </integration>

  <integration name="Web Speech API">
    <purpose>음성 인식(STT) + 음성 합성(TTS)</purpose>
    <platform_note>
      CRITICAL: iOS Safari/Android Chrome에서 SpeechRecognition 지원 여부를 Step 0에서 반드시 검증.
      미지원 시 Whisper API 폴백 또는 텍스트 입력 폴백 필요.
      iOS Safari는 getVoices()가 첫 호출 시 빈 배열을 줄 수 있으므로 voiceschanged 이벤트 + 폴링으로 보강.
    </platform_note>
    <stt_languages>ko-KR, en-US, de-DE, fr-FR, it-IT</stt_languages>
    <tts_languages>ko-KR, en-US, de-DE, fr-FR, it-IT</tts_languages>
    <tts_voice_policy>
      모든 언어에서 젊은 여성 보이스 우선 선택. 자세한 화이트리스트와 폴백 규칙은 core_functionality > voice_selection 참고.
      pitch 약 1.10, rate 1.0으로 일관된 톤 유지.
    </tts_voice_policy>
  </integration>
</third_party_integrations>

<aesthetic_guidelines>
  <design_philosophy>
    Kakao 다크 테마. 검은 배경에 초록 액센트.
    여행 중 한 손(엄지)으로 조작 가능한 큰 터치 타겟.
    핵심 원칙:
    1. 엄지 존 우선 — 주요 액션 버튼은 화면 하단 1/3에 배치 (iPhone 한 손 조작)
    2. 상태 명확성 — 현재 누가 말하는지, 어떤 언어인지 항상 시각적으로 명확
    3. 피드백 즉시성 — 모든 터치에 햅틱 + 시각 피드백 (사용자가 "눌렸다"를 확신)
    4. 최소 인지 부하 — 여행 중 피곤한 상태에서도 실수 없이 사용 가능
    5. 밝은 환경 대응 — 야외 햇빛 아래에서도 번역 결과 가독성 확보 (고대비)
  </design_philosophy>

  <color_palette>
    - Background: #0d1117
    - Surface: #161b22
    - Surface raised: #21262d
    - Border: #30363d
    - Text primary: #e6edf3
    - Text secondary: #8b949e
    - Accent (내 번역): #7ee787 (초록)
    - Accent alt (상대 번역): #58a6ff (파랑 — 나/상대 시각 구분)
    - Error: #f85149
    - Recording: #f85149
    - Warning: #d29922
    - Info: #58a6ff
  </color_palette>

  <typography>
    - Font family: -apple-system, BlinkMacSystemFont, system-ui, sans-serif
    - 번역 결과 (전체화면): 28px~64px (viewport 기반 자동, clamp(28px, 8vw, 64px)), bold
    - 버블 번역문: 18px, bold (기존 16→18, 여행 중 가독성)
    - 버블 원문: 14px, regular
    - 안내 텍스트: 13~14px, regular, #8b949e
    - 탭 라벨: 12px
    - 바텀시트 제목: 18px, bold
    - 토스트 메시지: 14px, medium
    CRITICAL: 모든 텍스트 line-height 1.5 이상 (한글+유럽어 혼합 시 가독성 확보).
  </typography>

  <spacing>
    기본 단위 4px. 스케일: 4, 8, 12, 16, 24, 32, 48.
    safe-area: env(safe-area-inset-top/bottom/left/right) 모든 가장자리 요소에 적용.
  </spacing>

  <touch_targets>
    - CRITICAL: 최소 탭 타겟 44x44px (Apple HIG)
    - 마이크/촬영 버튼: 56~72px (주요 액션)
    - 탭 바 아이템: 56px 높이
    - 언어 선택 버튼: 48px 높이
    - 바텀시트 항목: 56px 높이
    - 버블 롱프레스 영역: 버블 전체
    - 모든 아이콘 버튼: 최소 44x44px 터치 영역 (시각적 크기와 별개)
  </touch_targets>

  <haptic_feedback>
    CRITICAL: iPhone 사용 시 촉각 피드백으로 동작 확인감 제공.
    - 녹음 시작: UIImpactFeedbackGenerator.light
    - 녹음 종료: UIImpactFeedbackGenerator.medium
    - 촬영: UIImpactFeedbackGenerator.medium
    - 번역 완료: UINotificationFeedbackGenerator.success
    - 에러 발생: UINotificationFeedbackGenerator.error
    - 롱프레스 메뉴: UIImpactFeedbackGenerator.medium
    - 언어 변경: UIImpactFeedbackGenerator.light
    - 토글 전환: UIImpactFeedbackGenerator.light
    구현: navigator.vibrate() 또는 CSS haptic (iOS Safari가 vibrate 미지원이므로 시각 피드백 + UIImpact 호환 라이브러리로 폴백).
  </haptic_feedback>

  <animations>
    - 녹음 중 펄스: 활성 마이크 버튼 scale 1.0↔1.12, 0.8s ease-in-out infinite
    - 음성 파형: 5개 bar, 높이 8~24px 랜덤 변동, 100ms interval, 색상 #f85149
    - 전체화면 결과 진입: scale(0.95)→scale(1) + fade-in 250ms ease-out
    - 전체화면 결과 닫기: scale(1)→scale(0.95) + fade-out 200ms ease-in
    - 바텀시트 진입: translateY(100%)→translateY(0) 300ms cubic-bezier(0.32, 0.72, 0, 1)
    - 바텀시트 닫기: translateY(0)→translateY(100%) 250ms ease-in
    - 메시지 버블 추가: translateY(20px)→translateY(0) + fade-in 200ms ease-out
    - 로딩 스피너: 회전 1s linear infinite, 32px, #7ee787
    - 로딩 dot: 3개 dot bounce, 각각 0ms/200ms/400ms delay, 600ms ease-in-out
    - 셔터 flash: opacity 0→0.3→0, 150ms, 배경 #ffffff
    - 뷰 전환 (탭): 없음 (즉시, 빠른 반응 우선)
    - 토스트: slide-down 200ms + 3초 유지 + fade-out 300ms
    - 스켈레톤 shimmer: linear-gradient 이동 1.5s ease-in-out infinite
    CRITICAL: prefers-reduced-motion 미디어 쿼리 대응 → 모션 축소 시 모든 애니메이션 비활성화.
  </animations>

  <gestures>
    - 탭 간 스와이프: 좌우 스와이프로 대화 ↔ 카메라 전환 (threshold 50px)
    - 전체화면 결과 닫기: 아래로 스와이프 (threshold 100px, velocity 고려)
    - 바텀시트 닫기: 아래로 스와이프
    - 카메라 결과: 바텀시트 드래그 (중간/풀 두 단계 snap point)
    - 버블 롱프레스: 0.5초 홀드 → 컨텍스트 메뉴
    CRITICAL: 모든 스와이프 제스처에 스크롤과 충돌 방지 처리 필요.
  </gestures>

  <mobile_web_appearance>
    - theme-color 메타: #0d1117 — 모바일 브라우저 상단 바 색상 통일 (iOS Safari 15.4+, Android Chrome)
    - 파비콘: Kakao 스타일 번역 아이콘 (초록+검정), SVG + 192/512 PNG (브라우저 탭/북마크용)
    - viewport: width=device-width, initial-scale=1, viewport-fit=cover, maximum-scale=1 (노치/Dynamic Island 대응 + 입력 시 자동 확대 방지)
    - orientation: 별도 잠금 없음. 세로 모드에 디자인 최적화, 가로 모드도 깨지지 않게 유연 레이아웃.
    - 데스크탑 폴백: max-width 480px 컨테이너 + 좌우 #0d1117 여백으로 모바일 폭 유지.
    - 컬러스킴: <meta name="color-scheme" content="dark"> 으로 모바일 브라우저 폼 컨트롤도 다크.
  </mobile_web_appearance>
</aesthetic_guidelines>

<security_considerations>
  <api_key_storage>
    - API 키는 localStorage에만 저장 (소수 사용자, 개인 기기)
    - 코드에 API 키 하드코딩 금지
    - API 키 표시 시 마스킹 (sk-****1234)
  </api_key_storage>
  <api_protection>
    - 클라이언트 circuit breaker: 분당 15회 API 호출 상한
    - 연속 실패 3회 시 30초 쿨다운
    - generation counter: 비동기 결과 stale 방지
  </api_protection>
</security_considerations>

<final_integration_test>
  <test_scenario_1>
    <description>음성 대화 — "나" 버튼으로 한국어→영어 번역</description>
    <steps>
      1. 앱 열기 → 대화 탭 확인, 듀얼 마이크 버튼(나/상대) 표시 확인
      2. 언어 바에서 상대 언어 탭 → 바텀시트에서 "English" 선택
      3. "나" 버튼(초록) 탭 → 버튼 빨강 변환 + 파형 애니메이션, "상대" 버튼 비활성
      4. 한국어로 "이 자리 비어 있나요?" 말하기
      5. "나" 버튼 다시 탭 → 녹음 중지 → 로딩 dot 표시
      6. 대화 버블에 "나" 라벨 + 원문 + 영어 번역문 표시 확인
      7. 전체화면에 영어 번역 크게 표시 + "한국어 → English" 방향 칩 확인
      8. TTS 스피커 아이콘 탭 → 영어 발음 재생 확인 (젊은 여성 보이스로 들리는지 청취 확인)
      9. 아래로 스와이프 → 대화 뷰 복귀
    </steps>
  </test_scenario_1>

  <test_scenario_1b>
    <description>음성 대화 — "상대" 버튼으로 영어→한국어 번역</description>
    <steps>
      1. "상대" 버튼(파랑) 탭 → 버튼 빨강 + 파형, "나" 버튼 비활성
      2. 영어로 "This seat is available" 말하기
      3. "상대" 버튼 다시 탭 → 녹음 중지
      4. 대화 버블에 "상대 🇬🇧" 라벨 + 원문 + 한국어 번역 표시 확인
      5. 전체화면 표시 안 됨 확인 (내가 읽으면 되므로)
      6. 버블 롱프레스 → 컨텍스트 메뉴 → "다시 읽기" 탭 → TTS 재생
    </steps>
  </test_scenario_1b>

  <test_scenario_1c>
    <description>텍스트 입력 폴백</description>
    <steps>
      1. 듀얼 마이크 아래 키보드 아이콘 탭 → 텍스트 입력 모드 전환
      2. "한→영" 방향 칩 확인
      3. "화장실이 어디에 있나요?" 입력 → 전송 버튼 탭
      4. 번역 결과 버블 + 전체화면 표시 확인
      5. 마이크 아이콘 탭 → 음성 모드 복귀
    </steps>
  </test_scenario_1c>

  <test_scenario_2>
    <description>카메라 — 메뉴판 촬영 및 맥락 해석</description>
    <steps>
      1. 하단 카메라 탭 이동 (또는 좌로 스와이프) → 후면 카메라 프리뷰 + 가이드 프레임 확인
      2. 메뉴판을 프레임 안에 맞추기
      3. 촬영 버튼 탭 → 셔터 flash + 햅틱 피드백
      4. 프리뷰 freeze + 로딩 오버레이 ("분석 중..." + "보통 3~5초 걸려요")
      5. 결과 바텀시트 slide-up → 메뉴 항목별 카드 표시
      6. 각 카드에 원문 + 번역 + 맥락 설명 (재료/특징/알레르기 주의) 확인
      7. 개별 카드의 TTS 아이콘 탭 → 해당 항목만 발음 재생
      8. 스크롤하여 전체 항목 확인
      9. "다시 촬영" 탭 → 카메라 프리뷰 재시작
    </steps>
  </test_scenario_2>

  <test_scenario_2b>
    <description>카메라 — 갤러리 사진 번역</description>
    <steps>
      1. 카메라 뷰 하단 좌측 갤러리 버튼 탭
      2. 사진 라이브러리에서 메뉴판 사진 선택
      3. 로딩 → 결과 표시 (촬영과 동일 흐름)
    </steps>
  </test_scenario_2b>

  <test_scenario_3>
    <description>최초 실행 — 온보딩 + API 키 설정</description>
    <steps>
      1. API 키 없이 앱 열기 → 온보딩 화면 표시
      2. "시작하기" 탭 → API 키 입력 화면
      3. API 키 입력 → "완료" 탭
      4. ✓ "API 키 설정 완료" 토스트 표시
      5. 대화 탭으로 자동 이동 → 듀얼 마이크 정상 표시
    </steps>
  </test_scenario_3>

  <test_scenario_4>
    <description>설정 바텀시트 + 언어 변경</description>
    <steps>
      1. 언어 바 우측 설정 아이콘 탭 → 바텀시트 올라옴
      2. 상대방 언어 "🇩🇪 Deutsch" 선택 → 햅틱 피드백
      3. TTS 토글 off → 햅틱 피드백
      4. 바텀시트 아래로 스와이프 → 닫기
      5. 듀얼 마이크 "상대" 버튼 하단 라벨이 "Deutsch"로 변경 확인
    </steps>
  </test_scenario_4>
</final_integration_test>

<success_criteria>
  <functionality>
    - 한국어 음성 → 영어/독일어/프랑스어/이탈리아어 번역 동작
    - 외국어 음성 → 한국어 번역 동작
    - 카메라 촬영 → 맥락 번역 결과 표시
    - 전체화면 결과를 상대방에게 보여줄 수 있음
    - TTS 재생 동작 — 모든 언어에서 젊은 여성 보이스로 일관 재생
  </functionality>
  <user_experience>
    - 페이지 열기 → 마이크 탭까지 1탭 (듀얼 버튼으로 턴 혼동 제거)
    - 말하기 → 번역 결과까지 5초 이내
    - 촬영 → 해석 결과까지 5초 이내
    - 갤러리 사진으로도 번역 가능
    - 텍스트 입력 폴백으로 STT 불가 상황 대응
    - 모든 터치에 햅틱/시각 피드백
    - 바텀시트 기반 설정 → 화면 전환 없이 설정 변경
    - 스와이프 제스처로 탭 전환 + 전체화면 닫기
    - iOS Safari / Android Chrome 모바일 웹에서 URL 접속 즉시 사용 (설치/홈 화면 추가 불필요)
    - safe-area 대응으로 노치/Dynamic Island/홈 인디케이터 환경 지원
    - 데스크탑 브라우저에서도 모바일 폭 컨테이너로 정상 동작 (개발/공유 편의)
  </user_experience>
  <technical_quality>
    - TypeScript strict 모드
    - circuit breaker + generation counter 보호 장치
    - fixture 기반 테스트 (Gemini 응답 파싱, 프롬프트 생성)
  </technical_quality>
</success_criteria>

<key_implementation_notes>
  <critical_paths>
    - iOS Safari / Android Chrome Web Speech API (SpeechRecognition) 지원 여부가 최대 리스크
    - 미지원 시: 텍스트 입력 폴백 또는 Whisper API 폴백 필요
    - 카메라 API (getUserMedia)는 모바일 브라우저에서 지원됨 (HTTPS 필수)
    - TTS 보이스 선택: getVoices()가 첫 호출 시 빈 배열을 줄 수 있음 — voiceschanged + 폴링으로 보강하고 화이트리스트 우선 매칭으로 일관된 젊은 여성 톤 확보
  </critical_paths>

  <recommended_implementation_order>
    Step 0: 기술 검증 (iOS Safari + Android Chrome에서 STT/TTS/카메라 동작 + 보이스 목록 확인)
    Step 1: 모바일 웹 셸 (index.html 메타/뷰포트/theme-color, max-width 480px 컨테이너) + 설정 화면 (API 키 입력, 탭 네비게이션)
    Step 2: 음성 대화 MVP (마이크 → STT → 더미 번역 → 버블 표시)
    Step 3: Gemini 연동 (더미 → 실제 번역)
    Step 4: 전체화면 결과 + TTS — 젊은 여성 보이스 선택 로직(utils/voices.ts) 우선 구현
    Step 5: 카메라 번역 (촬영 → Gemini Vision → 결과 표시)
    Step 6: 마무리 (에러 처리, 보호 장치, 테스트, 모바일 다양한 화면 크기 QA)
  </recommended_implementation_order>

  <testing_strategy>
    - fixture 기반: Gemini 응답 JSON을 fixture로 저장, 파싱/표시 로직 테스트
    - 프롬프트 테스트: 언어별 프롬프트 생성 결과 검증
    - CRITICAL: 개발 중 Gemini API 호출 최소화. fixture로 먼저 완성 후 연동.
  </testing_strategy>
</key_implementation_notes>

</project_specification>
