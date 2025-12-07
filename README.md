# 📬 Noti to Notion

여러 웹사이트의 공지사항과 이벤트를 자동으로 크롤링하여 Notion 데이터베이스에 저장하는 자동화 도구입니다.

## ✨ 주요 기능

- **다중 소스 크롤링**: 여러 웹사이트에서 공지사항을 한 번에 수집
- **중복 방지**: Notion DB에 이미 있는 게시물은 자동으로 건너뜀
- **자동 실행**: GitHub Actions를 통한 정기적 자동 실행 지원
- **테스트 모드**: DRY_RUN 모드로 실제 저장 없이 테스트 가능

### 현재 지원하는 소스

| 소스 | 설명 | 태그 |
|------|------|------|
| 서울산업진흥원(SETI) | 교육/연수 공지사항 | `study` |
| 강원경제진흥원(GETI) | 교육/연수 공지사항 | `study` |
| 국립과천과학관 | 공지/공고 | `science` |

---

## 🚀 시작하기

### 1단계: Notion Integration 생성

1. [Notion Integrations](https://www.notion.so/my-integrations) 페이지로 이동
2. **+ New integration** 클릭
3. Integration 이름 입력 (예: `Noti Crawler`)
4. **Submit** 클릭
5. **Internal Integration Secret** 복사 (나중에 사용)

### 2단계: Notion 데이터베이스 생성

Notion에서 새 데이터베이스를 생성하고 다음 속성(Property)들을 추가하세요:

| 속성 이름 | 타입 | 설명 |
|-----------|------|------|
| `Name` | Title | 게시물 제목 |
| `URL` | URL | 원본 링크 |
| `Person` | Person | 담당자 (선택) |
| `Date` | Date | 수집 일시 |
| `CreationDate` | Date | 원본 게시 날짜 |
| `Tag` | Multi-select | 소스별 태그 |

### 3단계: 데이터베이스에 Integration 연결

1. 생성한 데이터베이스 페이지 우측 상단 **···** 클릭
2. **Connections** → **Connect to** 선택
3. 1단계에서 만든 Integration 선택

### 4단계: 데이터베이스 ID 확인

데이터베이스 페이지 URL에서 ID를 확인합니다:

```
https://www.notion.so/myworkspace/a8aec43384f447ed84390e8e42c2e089?v=...
                                  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
                                  이 부분이 DATABASE_ID
```

---

## ⚙️ 설정

### 코드 수정

`notitonotion.py` 파일에서 다음 값들을 본인 환경에 맞게 수정하세요:

```python
# 데이터베이스 ID (본인의 Notion DB ID로 변경)
DATABASE_ID = "your-database-id-here"

# Person 속성의 사용자 ID (선택 - 사용하지 않으면 해당 줄 삭제)
"Person": {"people": [{"object": "user", "id": "your-user-id-here"}]}
```

### 환경 변수 설정

| 변수명 | 설명 | 필수 |
|--------|------|------|
| `NOTION_AUTH_TOKEN` | Notion Integration Secret | ✅ |
| `DRY_RUN` | `true`로 설정 시 테스트 모드 | ❌ |

---

## 💻 로컬 실행

### 필요 조건

- Python 3.8+
- pip

### 설치

```bash
# 저장소 클론
git clone https://github.com/skywind99/noti_to_notion.git
cd noti_to_notion

# 의존성 설치
pip install -r requirements.txt
```

### 실행

```bash
# 환경 변수 설정
export NOTION_AUTH_TOKEN="your-notion-secret-here"

# 테스트 모드로 실행 (실제 저장 안 함)
export DRY_RUN=true
python notitonotion.py

# 실제 실행
export DRY_RUN=false
python notitonotion.py
```

---

## 🤖 GitHub Actions로 자동화

### 1단계: Fork 또는 Clone

이 저장소를 Fork하거나 본인의 GitHub 저장소에 코드를 업로드하세요.

### 2단계: Secrets 설정

1. GitHub 저장소 → **Settings** → **Secrets and variables** → **Actions**
2. **New repository secret** 클릭
3. 다음 Secret 추가:

| Name | Value |
|------|-------|
| `NOTION_AUTH_TOKEN` | Notion Integration Secret |

### 3단계: Workflow 파일 확인

`.github/workflows/` 폴더에 워크플로우 파일이 있습니다. 스케줄을 수정하려면:

```yaml
on:
  schedule:
    # 매일 오전 9시 (KST) 실행 = UTC 0시
    - cron: '0 0 * * *'
  workflow_dispatch:  # 수동 실행 가능
```

### 4단계: Actions 활성화

1. 저장소 → **Actions** 탭
2. 워크플로우 활성화
3. **Run workflow**로 수동 테스트

---

## 🔧 커스터마이징

### 새로운 소스 추가하기

1. 파싱 함수 작성:

```python
def parse_new_source():
    # 웹사이트 크롤링 로직
    response = session.get(YOUR_URL, headers=headers)
    soup = BeautifulSoup(response.content, 'html.parser')
    
    items = []
    # 파싱 로직 작성
    for row in soup.select('your-selector'):
        items.append({
            "title": "게시물 제목",
            "link": "URL",
            "date": "YYYY-MM-DD",
            "tag": "your-tag"
        })
    return items
```

2. `update_notion_with_new_posts()` 함수의 sources에 추가:

```python
sources = [
    ("Website", parse_website),
    ("RSS", parse_rss),
    ("Science", parse_science_notices),
    ("NewSource", parse_new_source),  # 새로 추가
]
```

### 태그 커스터마이징

각 소스별로 다른 태그를 지정할 수 있습니다:

```python
items.append({
    "title": title,
    "link": link,
    "date": iso_date,
    "tag": "custom-tag"  # 원하는 태그명
})
```

---

## 📁 프로젝트 구조

```
noti_to_notion/
├── notitonotion.py      # 메인 스크립트
├── requirements.txt     # Python 의존성
└── .github/
    └── workflows/       # GitHub Actions 워크플로우
```

---

## 🐛 문제 해결

### "Notion API error: 401"
- `NOTION_AUTH_TOKEN`이 올바르게 설정되었는지 확인
- Integration이 데이터베이스에 연결되었는지 확인

### "Notion API error: 404"
- `DATABASE_ID`가 올바른지 확인
- 데이터베이스 URL에서 ID를 다시 복사

### "Website fetch error"
- 대상 웹사이트가 접속 가능한지 확인
- 웹사이트 구조가 변경되었을 수 있음 (셀렉터 수정 필요)

### SSL 관련 에러
- 코드에 이미 맞춤형 SSL Adapter가 포함되어 있어 대부분의 SSL 이슈를 처리합니다

---

## 📝 라이선스

이 프로젝트는 자유롭게 사용, 수정, 배포할 수 있습니다.

---

## 🙋 기여하기

1. 이 저장소를 Fork
2. 새 브랜치 생성 (`git checkout -b feature/new-source`)
3. 변경사항 커밋 (`git commit -m 'Add new source'`)
4. 브랜치에 Push (`git push origin feature/new-source`)
5. Pull Request 생성

---

## 📮 문의

이슈나 질문이 있으시면 [GitHub Issues](https://github.com/skywind99/noti_to_notion/issues)에 등록해 주세요!
