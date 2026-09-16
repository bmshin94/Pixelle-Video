# 📘 Pixelle-Video 분석 & 활용 가이드 (한국어)

> 프로젝트를 처음 접한 사람이 **"이게 뭔지 / 어떻게 쓰는지 / 어떻게 돈이 되는지"** 를
> 한 번에 파악할 수 있도록 정리한 문서입니다.

---

## 🔗 저장소 주소

| 구분 | 주소 |
|---|---|
| 🍴 내 포크 | https://github.com/bmshin94/Pixelle-Video |
| ⭐ 원본 (Upstream) | https://github.com/AIDC-AI/Pixelle-Video |
| 📘 공식 문서 | https://aidc-ai.github.io/Pixelle-Video/zh |
| 📦 Windows 통합팩 | https://github.com/AIDC-AI/Pixelle-Video/releases |

- **개발사**: AIDC-AI (알리바바 AI 사업부)
- **라이선스**: Apache-2.0 (상업적 이용 / 수정 / 재배포 가능, 저작권 고지 및 NOTICE 유지 필요)
- **버전**: 0.2.0 / Python >= 3.11

---

## 1. 🎬 이게 뭐 하는 프로젝트야?

**AI 자동 숏폼(쇼츠/릴스) 영상 생성 엔진**입니다.
주제 한 줄만 입력하면 아래 과정이 전부 자동으로 돌아갑니다.

```
"왜 아침에 일찍 일어나야 할까" 입력
  ↓ ✍️  LLM이 대본 작성
  ↓ 🎨  문장별 이미지/영상 생성 (ComfyUI · Flux · WAN · Seedream · Kling)
  ↓ 🗣️  나레이션 음성 합성 (Edge-TTS · Index-TTS, 음성 클론 지원)
  ↓ 🎵  배경음악(BGM) 추가
  ↓ 🎬  ffmpeg로 최종 합성 → output/*.mp4
```

**편집 프로그램(프리미어·캡컷) 없이** 영상이 완성됩니다.

### ✨ 가장 큰 차별점: 영상 디자인이 HTML/CSS

`pixelle_video/services/frame_html.py` 를 보면
**Playwright 헤드리스 브라우저로 HTML 템플릿을 렌더링 → 이미지로 캡처 → 영상 프레임으로 사용**합니다.

👉 **웹 개발 지식만 있으면 나만의 영상 스타일을 무한 생산**할 수 있다는 뜻입니다.
현재 `templates/` 에 30여 종(세로/가로/정사각형)이 기본 내장되어 있습니다.

---

## 2. 📁 폴더 구조

| 경로 | 역할 |
|---|---|
| `pixelle_video/` | 🏭 **핵심 엔진** |
| `pixelle_video/pipelines/` | 생성 시나리오 — `standard.py`(주제→영상), `asset_based.py`(내 사진/영상 활용), `custom.py`(직접 제작용 템플릿), `linear.py`(Template Method 베이스) |
| `pixelle_video/services/` | 기능별 서비스 — `llm_service.py`(대본), `tts_service.py`(음성), `media.py`(미디어), `video.py`(ffmpeg), `frame_html.py`(HTML→이미지), `persistence.py`·`history_manager.py`(저장/이력) |
| `pixelle_video/services/api_services/` | 모델 직접 호출 — `image_gpt.py`, `image_seedream.py`, `image_dashscope.py`, `video_kling.py`, `video_seedance.py`, `vlm_*.py` |
| `pixelle_video/prompts/` | 프롬프트 모음 (대본·이미지·제목·스타일 변환) |
| `web/` | 🖥️ **Streamlit 웹 UI** (포트 8501). 파이프라인 5종: 기본 / 커스텀 소재 / 디지털휴먼 / 이미지→영상 / 동작전이 |
| `api/` | ⚡ **FastAPI REST 서버** (포트 8000) |
| `templates/` | 🎨 HTML 영상 템플릿 (1080x1920 / 1920x1080 / 1080x1080) |
| `workflows/` | ComfyUI 워크플로우 JSON — `selfhost/`(로컬) vs `runninghub/`(클라우드) |
| `bgm/` | 배경음악 보관 |
| `docs/` | 한/영/중 문서 (mkdocs) |
| `packaging/windows/` | Windows 원클릭 통합팩 빌드 스크립트 |
| `Dockerfile` · `docker-compose.yml` | 도커 실행 (API + WebUI 동시 기동) |

---

## 3. 🚀 설치 및 사용법

### 방법 A — Windows 통합팩 (가장 쉬움) ⭐
1. Releases에서 통합팩 다운로드 → 압축 해제
2. `start.bat` 더블클릭
3. 브라우저에서 `localhost:8501` 자동 오픈
4. 「⚙️ 시스템 설정」에서 API 키 입력 → 저장
5. 주제 입력 → 「🎬 영상 생성」

> Python·uv·ffmpeg 설치 불필요 (전부 포함)

### 방법 B — 소스 실행 (macOS / Linux)
```bash
# 사전 준비
brew install ffmpeg          # 필수
# uv 설치: https://docs.astral.sh/uv/getting-started/installation/

# 실행 (의존성 자동 설치)
uv run streamlit run web/app.py
```

### 방법 C — Docker
```bash
docker-compose up -d
# WebUI: localhost:8501 / API: localhost:8000
# config.yaml 이 없으면 config.example.yaml 에서 자동 생성됨
```

### 실제 사용 흐름
```
① 생성 모드     : "AI 생성"(주제만 입력) 또는 "고정 문안"(내 대본 사용)
② 음성 설정     : TTS 워크플로우 선택 / 참고 음성 업로드 시 음성 클론
③ 비주얼 설정   : 이미지 워크플로우 + 프롬프트 접두어(스타일 고정) + 템플릿
④ BGM 선택     : 없음 / 내장 / bgm 폴더에 직접 추가
⑤ 생성         : 진행률 확인 → output/ 폴더에 mp4 저장
```

---

## 4. 🧩 플러그인? 스킬? MCP? → **전부 아님, 독립 웹앱입니다**

| 구분 | 여부 | 근거 |
|---|---|---|
| Claude 플러그인 | ❌ | `.claude-plugin/` 없음 |
| Claude 스킬 | ❌ | `SKILL.md` 없음 |
| MCP 서버 | ❌ | 아래 참고 |

### 🕵️ 확인된 사실
`pyproject.toml` 에 `fastmcp>=2.0.0` 이 **의존성으로 선언되어 있지만,
실제 코드에서는 단 한 곳도 사용하지 않습니다.**
(전체 `.py` 검색 결과 `mcp` 관련 코드 0건 — 형제 프로젝트 **Pixelle-MCP** 에서 분기되며 남은 잔여 의존성)

### ✅ 대신 REST API 서버가 존재
| 엔드포인트 | 기능 |
|---|---|
| `POST /video/generate/async` | 영상 생성 (비동기) |
| `POST /video/generate/sync` | 영상 생성 (동기) |
| `POST /tts/synthesize` | 음성 합성 |
| `POST /image/generate` | 이미지 생성 |
| `POST /content/narration` · `/title` · `/image-prompt` | 대본·제목·프롬프트 생성 |
| `POST /frame/render` | HTML 템플릿 → 이미지 렌더 |
| `GET /tasks/{task_id}` | 작업 진행 상황 |
| `GET /resources/workflows/*` · `/templates` · `/bgm` | 사용 가능 리소스 목록 |

> 💡 이 REST API 덕분에 **MCP 서버로 감싸는 작업이 매우 간단**합니다.

---

## 5. 💳 API 토큰이 필요한가?

### 🔴 필수 1개 — LLM (대본 작성)
```yaml
llm:
  api_key: ""
  base_url: ""
  model: ""
```
| 선택지 | 비용 |
|---|---|
| **Ollama (로컬)** | **0원** 🆓 |
| Qwen(통의천문) | 매우 저렴 ⭐ 가성비 |
| DeepSeek | 저렴 |
| GPT-4o | 고가 / 고품질 |

### 🟡 이미지·영상 — 3가지 중 택 1
| 방식 | 키 | 비용 |
|---|---|---|
| **ComfyUI 로컬** | 불필요 | **무료** (GPU 필요) |
| **RunningHub 클라우드** | `runninghub_api_key` | 유료 / 설치 0 |
| **모델 API 직접 호출** | `openai`, `dashscope`, `ark`, `kling(access+secret)` | 유료 / 고품질 |

### 🟢 음성(TTS) — 무료
Edge-TTS는 별도 키 없이 사용 가능하며 **한국어를 지원**합니다.

### 💸 최소 비용 조합
```
Ollama(무료) + ComfyUI 로컬(무료) + Edge-TTS(무료) = 전기세만
```

> ⚠️ **`config.yaml` 은 절대 커밋 금지!** (`.gitignore` 등록 완료 — 강제 add 주의)

---

## 6. ⭐ 왜 GitHub에서 인기가 있나

1. **알리바바 공식 팀(AIDC-AI) 작품** — 개인 프로젝트 대비 신뢰도·지속성이 다름
2. **"AI 숏폼 자동 생성"이 최상위 트렌드 키워드** — MoneyPrinterTurbo·NarratoAI 계보의 후발 개선작
3. **HTML 템플릿이라는 명확한 차별점** — 웹 개발자면 누구나 커스터마이징 가능
4. **완전 무료 운영 가능** (Ollama + ComfyUI)
5. **모듈 교체 자유도** — LLM·이미지·영상·TTS 전부 갈아끼움, 벤더 락인 없음
6. **진입 장벽 최소화** — Windows 통합팩 · Docker · 다국어 문서 · 영상 튜토리얼 · 커뮤니티
7. **연구 백그라운드** — SIGGRAPH Asia / ACL 논문 다수 연계

---

## 7. 🤖 로컬 에이전트 구축에 도움이 되는가 → **매우 그렇다**

### 방향 1 — MCP 서버로 감싸기 (최우선 추천)
이미 REST API가 있으므로 얇은 래퍼만 추가하면 됩니다.
```python
from fastmcp import FastMCP   # 이미 의존성에 포함되어 있음

mcp = FastMCP("pixelle-video")

@mcp.tool()
async def create_short_video(topic: str, template: str = "image_default"):
    """주제를 받아 숏폼 영상을 생성한다"""
    ...  # api/routers/video.py 로직 호출
```
→ AI 어시스턴트에게 **"'겨울철 건강관리' 주제로 세로 영상 만들어줘"** 라고 말하면 바로 실행 가능.

### 방향 2 — 에이전트 설계 레퍼런스로 학습
| 배울 점 | 위치 |
|---|---|
| 멀티스텝 워크플로우 설계 | `pipelines/linear.py` (Template Method) |
| 진행률 실시간 보고 | `models/progress.py` |
| 멀티 프로바이더 추상화 | `services/api_services/` |
| 프롬프트 분리 관리 | `prompts/` |
| 병렬 처리 + 동시성 제한 | `pipelines/standard.py` (asyncio.Semaphore) |
| 작업 상태 영속화 | `services/persistence.py`, `history_manager.py` |
| LLM 비정형 응답 방어 | 커밋 이력의 `fix: null content` 계열 참고 |

### 방향 3 — 부품 단위 재활용
- `frame_html.py` → HTML을 이미지로 변환 (카드뉴스·썸네일·리포트 생성기에 그대로 사용 가능)
- `services/video.py` → ffmpeg 래퍼 (병합·자르기·BGM·오버레이)
- `tts_service.py`, `tts_voices.py` → 무료 다국어 TTS

---

## 8. 💰 수익화 아이디어

### 원가 구조 (60초 / 컷 8개 기준, 추정치)
| 구성 | 방식 | 대략 원가 |
|---|---|---|
| 대본 | Qwen/DeepSeek | 10~50원 |
| 대본 | Ollama 로컬 | 0원 |
| 이미지 8장 | ComfyUI 로컬 | 0원 |
| 이미지 8장 | RunningHub / API | 300~2,000원 |
| 영상 클립 | Kling / Seedance | 2,000~10,000원 |
| 음성 · BGM · 합성 | Edge-TTS + ffmpeg | 0원 |

> 단가는 제공사·모델·시점에 따라 달라지므로 반드시 직접 확인 필요.

**핵심 인사이트**
1. 이미지 기반은 거의 무료, 영상 기반은 고비용 → **초기에는 이미지 템플릿으로 시작**
2. GPU 보유 시 원가가 0에 수렴 → **가격 결정권이 내게 있음 (마진 90%+)**

### 수익 모델 6종 비교
| 모델 | 시작까지 | 초기비용 | 월 수익 잠재력 | 난이도 | 추천 |
|---|---|---|---|---|---|
| **A. 로컬 비즈니스 대행** | 2~4주 | 거의 0 | 200~600만원 | ⭐⭐ | 🥇 |
| **B. SaaS 웹서비스** | 2~3개월 | 100~300만원 | 200~1,000만원+ | ⭐⭐⭐⭐ | 🥈 |
| **C. 템플릿 판매** | 1~2주 | 0 | 30~150만원 | ⭐ | 🥉 |
| **D. 채널 직접 운영** | 3~6개월 | 0 | 편차 큼 | ⭐⭐⭐ | ⚠️ |
| **E. 한국화 포크 → 커뮤니티** | 1~2개월 | 0 | 100~500만원 | ⭐⭐⭐ | 👍 |
| **F. 버티컬 SaaS(업종 특화)** | 3~4개월 | 200~500만원 | 500~3,000만원 | ⭐⭐⭐⭐⭐ | 💎 |

> 수치는 성공 시 기준의 참고값이며 보장된 수익이 아닙니다.

#### A. 로컬 비즈니스 숏폼 대행 🥇
- **타겟**: 병원·치과·학원·부동산·카페·헬스장·미용실
- **상품**: 베이직 8개 20만원 / 스탠다드 20개 40만원 / 프리미엄 30개+업로드대행+리포트 70만원 (월)
- **활용 기능**: `asset_based` 파이프라인 (사장님 실제 사진·영상으로 제작 → AI 생성 이미지보다 전환율 높음)
- **선행 작업**: 한국어 자막/폰트 최적화, 업종별 프롬프트 세트
- **시작법**: 샘플 10개 제작 → 인스타 계정 → 지역 업체 대상 무료 3개 제안 → 계약 전환

#### B. SaaS 웹서비스 🥈
- **요금제 예시**: 무료(월 3개·워터마크) / 라이트 9,900원 / 프로 29,000원 / 팀 99,000원
- **이미 완성된 부분**: 생성 엔진, REST API, 비동기 작업 큐 + 진행률, 히스토리
- **추가 개발**: React 프론트, 인증, 결제(토스·아임포트·Stripe), 사용량 제한, 스토리지, 큐 워커 분리
- **주의**: GPU 서버 비용 · 무료 플랜 어뷰징 방어

#### C. 템플릿/에셋 판매 🥉
- HTML 기반이라 디자인 대량 생산 용이
- 한국형 템플릿팩 29,000원 / 업종별 팩 19,000원 / 프롬프트 프리셋 12,000원 / 번들 79,000원
- 판매처: 크몽 · 탈잉 · Gumroad 등
- **빈틈**: 원본은 중국 스타일 위주 → 한국 감성 디자인 수요 존재

#### D. 콘텐츠 채널 직접 운영 ⚠️
- 아이디어: 책 요약 · 심리학 · 역사 · 건강 · 재테크 · 해외 영어 채널(CPM 우위)
- **리스크**: 유튜브/틱톡의 대량생산 저품질 콘텐츠 정책 강화 → 수익 창출 거절·채널 제재 가능
- **대응**: 기획·검수는 사람이, 고유 음성 확보, AI 사용 표시, 양보다 질

#### E. 한국 시장 특화 포크 👍
- 한국어 자막/폰트 최적화, 클로바 TTS 연동, 한국형 템플릿, SNS 자동 업로드
- 오픈소스 공개 → 커뮤니티 형성
- 수익화: 강의 · 전자책 · 컨설팅(건당 50~200만원) · 호스팅 버전(오픈코어)

#### F. 버티컬 SaaS 💎
```
"영상 툴"          → 경쟁 과다, 월 1만원
"치과 전용 환자 유입 자동화" → 경쟁 희소, 월 50~150만원
```
- 치과 예시 구성: 시술별 대본 템플릿 · **의료광고 심의 체크** · 후기 영상(개인정보 마스킹) · 지역 키워드 · 다채널 배포 · 성과 리포트
- 다른 후보: 부동산 매물 영상, 학원 합격 후기, **쇼핑몰 상품 상세 → 릴스 변환**, 숙박업 객실 홍보

### 🗺️ 실행 로드맵
```
0~1개월 (검증)
  □ 영상 20개 직접 제작해보기
  □ 한국어 자막/폰트 문제 해결
  □ 업종별 샘플 3세트 (카페 / 병원 / 학원)
  □ 인스타 계정 개설 + 샘플 업로드
  □ 지인 업체 1곳 무료 제작 → 반응 확인

1~3개월 (첫 매출)
  □ [A] 고객 3~5곳 확보 → 월 100~200만원
  □ [C] 템플릿팩 제작·등록 (병행)
  □ 배치 생성 자동화 스크립트
  □ 고객 피드백 수집

3~6개월 (확장)
  □ 반응 좋은 업종 선정 → [F] 버티컬 특화
  □ 또는 [B] SaaS 셀프서비스화
  □ [E] 한국화 포크 공개로 유입 확보
```

### ⚖️ 법적 · 정책 주의사항
- 🔴 **AI 생성물 표시**: 유튜브 AI 합성 콘텐츠 공개 표시 / 광고성 콘텐츠는 표시광고법상 "광고" 표기 필수
- 🔴 **의료·금융·법률 광고**: 의료광고는 사전 심의 대상, 금융·투자도 규제 존재
- 🔴 **저작권**: BGM 상업 이용 라이선스 확인, AI 이미지의 모델사 약관 확인, 실제 인물은 초상권 동의
- 🔴 **음성 클론**: 타인 목소리 무단 클론 금지 (본인 또는 명시적 동의만)
- 🟢 **Apache 2.0**: 프로젝트 자체의 상업적 이용·수정·재배포는 자유 (저작권 고지 및 NOTICE 유지)

### 결론
> **[모델 A]로 현금 흐름을 만들고 → 고객에게서 발견한 진짜 문제를 [모델 F]로 자동화**
>
> "영상 생성 기술"은 이미 흔합니다. 수익은 **특정 업종의 반복 업무를 끝까지 없애주는 것**에서 나옵니다.

---

## 9. 🛠️ React / PHP로 만들 수 있나?

### 결론: **React 프론트 + 기존 파이썬 엔진 조합이 최선** ⭐
```
┌──────────────────┐   HTTP   ┌──────────────────────┐
│  React 프론트엔드  │ ───────▶ │ Pixelle-Video 엔진     │
│  로그인/결제/UI   │          │ api/ (이미 완성)      │
└──────────────────┘          └──────────────────────┘
```
```javascript
const res = await fetch('http://localhost:8000/video/generate/async', {
  method: 'POST',
  body: JSON.stringify({ topic: '겨울철 건강관리' })
});
const { task_id } = await res.json();   // 이후 /tasks/{task_id} 폴링
```

### Node.js 전체 재작성 시 대체 기술
| 기능 | 파이썬 | Node 대체 | 난이도 |
|---|---|---|---|
| HTML→이미지 | Playwright | Puppeteer | 쉬움 |
| 영상 합성 | ffmpeg-python | fluent-ffmpeg | 쉬움 |
| LLM 호출 | openai | openai (js) | 쉬움 |
| 이미지 API | httpx | fetch/axios | 쉬움 |
| TTS | edge-tts | msedge-tts | 보통 |
| ComfyUI 연동 | comfykit | 직접 구현 | 어려움 |

### PHP는 비추천
- 영상 생성은 5~10분 소요 → 요청·응답 모델과 부적합 (Laravel Queue + Redis 필수)
- 병렬 이미지 생성이 핵심인데 async 지원이 약함
- HTML 렌더링(`spatie/browsershot`)도 내부적으로 Node Puppeteer 호출
- AI 라이브러리 생태계 빈약
- **대안**: 엔진은 파이썬 유지, PHP는 회원·결제 영역만 담당

### 최종 비교
| 방식 | 기간 | 난이도 | 추천 |
|---|---|---|---|
| React UI + 파이썬 엔진 | 1~2주 | ⭐⭐ | 🥇 |
| 파이썬 엔진 직접 개조 | 며칠 | ⭐ | 🥈 |
| Node.js 전체 재작성 | 2~3개월 | ⭐⭐⭐⭐ | 🥉 |
| PHP 전체 재작성 | 3개월+ | ⭐⭐⭐⭐⭐ | ❌ |

---

## 10. ✅ 다음에 할 일 (우선순위)

1. 🔤 **한국어 자막/폰트 최적화** — 영업·판매의 전제 조건. 가장 먼저 처리
2. 🎨 **한국 감성 템플릿 5종 제작** — 샘플 영상 제작용
3. 🏥 **업종별 프롬프트 세트** (치과 / 카페 / 학원)
4. ⚡ **배치 생성 스크립트** — 영상 다량 일괄 생성
5. 🤖 **MCP 서버 래퍼** — AI 어시스턴트에서 직접 호출
6. 📄 **영업용 제안서 / 단가표**

---

*이 문서는 저장소 분석 결과를 바탕으로 정리되었습니다. 비용·수익 수치는 참고용 추정치이며 실제와 다를 수 있습니다.*
