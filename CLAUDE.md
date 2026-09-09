# French Study — 우리 토끼 프랑스어 공부 🐰

한국어 사용자를 위한 프랑스어 학습 웹앱. YouTube 영상 또는 프랑스어 스크립트를 넣으면
Gemini API가 문장별 교재(해석·단어·구문·발음 팁·예문)를 만들어 주고, 문장별 구간 반복·
AI 원어민 TTS·발음 녹음 비교·복습 문장 모음 기능으로 학습한다.

- **배포 주소**: https://baelab-create.github.io/french-study/
- **저장소**: https://github.com/baelab-create/french-study (`main` 브랜치, push하면 GitHub Pages 자동 배포)
- **현재 완성도**: 기능 완성 상태로 실사용 중. 백엔드 없음, 순수 정적 사이트.

## 기술 스택 / 구조

- 프레임워크·빌드 도구 없음. **단일 `index.html`** (HTML+CSS+JS 전부 포함) + `assets/banner.jpg`(상단 배너).
- 외부 의존성: YouTube IFrame API(런타임 로드), Google Gemini API(REST 직접 호출).
- 데이터 저장: 전부 **브라우저 안** — localStorage(교재/진도/설정, 키 접두어 `fs_`) + IndexedDB `fs_recordings`(녹음·TTS 캐시). 서버 DB 없음.
- Gemini API 키는 사용자가 설정 화면에서 입력하며 localStorage에만 저장된다. **저장소에 키가 들어가면 절대 안 됨.**

## 실행 / 배포

```bash
# 로컬 실행 (아무 정적 서버나 가능)
python3 -m http.server 8643 -d .
# → http://localhost:8643

# 배포 = main에 push (GitHub Pages가 1~2분 내 자동 반영)
git push
```

Claude Code 브라우저 프리뷰용 설정은 `~/Downloads/.claude/launch.json`의 `french-study` 항목(포트 8643)에 있다 — 이 파일은 저장소 밖이므로 다른 기기에서는 필요 시 다시 만들 것.

## 주요 설계 결정과 이유

1. **english-study 기반 포크**: https://baelab-create.github.io/english-study/ 의 index.html을 복사해 프랑스어용으로 개조했다 (2026-09-01). 구조·기능이 거의 같으므로 한쪽에서 고친 버그는 다른 쪽에도 적용을 검토할 것.
2. **localStorage 키 접두어 `fs_`** (english-study는 `es_`): 두 앱이 같은 origin(baelab-create.github.io)이라 접두어를 분리하지 않으면 교재 데이터가 서로 섞인다. IndexedDB 이름도 `fs_recordings`로 분리.
3. **JSON 필드명 `english`에 프랑스어 텍스트가 들어간다**: english-study 코드를 그대로 물려받아 필드명이 `english`다(레거시 호환). Gemini 프롬프트·스키마에 "the field is named 'english' for legacy reasons but must contain the French text"로 명시되어 있다. 전면 리네임은 기존 사용자의 localStorage 교재가 깨지므로 하지 않았다.
4. **입력 소스 A/B 탭**: A) YouTube 영상 분석, B) 프랑스어 스크립트 붙여넣기. B로 만든 교재는 `videoId:null`/`isText:true`이고, 학습 화면에서 `body.text-lesson` 클래스로 플레이어·구간재생 UI를 숨기고 AI TTS 중심으로 동작한다.
5. **디자인**: 사용자 제공 토끼·곰돌이 배너(`assets/banner.jpg`) 톤에 맞춘 핑크 팔레트 — bg `#fdf4ee`, accent `#ec6a97`, 본문 브라운 `#5a4232`. 학습 카드의 한글 해석·단어 칩·구문·발음 텍스트는 가독성을 위해 검정(`#1b1b1b`)으로 통일 (사용자 요청).
6. **예문(examples) 필드**: 문장마다 A1-A2 수준 예문 1~2개(`{french, korean}`)를 Gemini가 생성, "📝 예문으로 익히기" 블록에 AI 음성 재생 버튼과 함께 표시. 예문 추가 이전에 만든 교재에는 예문이 없다(재분석해야 생김).
7. **타임스탬프 정확도**: Gemini의 영상 직접 분석은 싱크가 어긋날 수 있어, YouTube "스크립트 표시" 텍스트나 SRT/VTT를 붙여넣으면 자막의 공식 타임스탬프를 그대로 쓰는 경로(권장)를 따로 두었다.

## 알려진 이슈

- **임베드 차단 영상**: 영상 소유자가 외부 사이트 재생을 막아 둔 영상(플레이어 에러 101/150)은 교재는 생성되지만 페이지 안에서 재생이 안 된다. YouTube 정책이라 우회 불가. 개선 아이디어(미구현): 에러를 감지해 친절한 안내 + "YouTube에서 열기" 버튼 표시.
- 샘플 교재(`sample_fr`)의 videoId는 Steve Jobs 연설(영어 영상) 플레이스홀더 — 문장과 영상 내용이 일치하지 않는다. 화면 구경용이라고 제목에 명시되어 있음.
- 사용자 데이터가 브라우저에만 있으므로 기기를 옮기면 교재/진도가 따라오지 않는다. 설정 화면의 내보내기/가져오기(JSON)로 수동 이전.

## 다음에 할 만한 작업

- 임베드 차단(101/150) 에러 감지 후 안내 UI (위 참고)
- english-study에 역이식할 만한 개선: 예문 기능, 녹음 재생 버튼 강조 등

## git에 없는 것 / 다른 기기에서 필요한 것

- **코드는 전부 git에 있다.** 수동으로 옮길 파일 없음.
- Gemini API 키: 각 기기 브라우저에서 설정 화면에 한 번씩 입력 (https://aistudio.google.com/apikey 에서 발급).
- 학습 데이터(교재·진도·복습): 브라우저별로 따로 저장됨. 옮기려면 설정 → 내보내기 → 새 기기에서 가져오기.
- GitHub 인증: 각 기기에서 `gh auth login` (계정 baelab-create) 또는 기존 git 인증 사용.
