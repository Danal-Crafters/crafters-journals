# 월간 크래프터스 아카이브 리포지토리

## 월간 크래프터스?
- 매월 1회 테크니컬라이팅 기법을 반영한 포스팅을 공유하는 저널입니다. 
- 매월 첫째주 금요일을 출간 목표로 잡고 있습니다.

## 컨플루언스 글 마이그레이션 하는 법

1. Confluence의 페이지 소스 HTML 가져오기
Confluence 페이지를 엽니다.
페이지 오른쪽 상단 메뉴에서 "페이지 더보기" 또는 "Export" 옵션을 선택합니다.  
**"HTML Export"**를 선택하여 해당 페이지를 HTML로 저장합니다.

2. HTML 소스를 Markdown으로 변환
HTML 파일을 Markdown으로 변환하기 위해 다음과 같은 오픈소스 도구나 파이썬 스크립트를 사용할 수 있습니다:

3. Pandoc 사용
Pandoc은 HTML을 Markdown으로 변환할 수 있는 강력한 CLI 도구입니다.  
  - **설치 및 사용 방법**
  - Pandoc을 설치합니다.
    ```bash
    sudo apt-get install pandoc  # Ubuntu
    brew install pandoc          # macOS
    ```
  - HTML 파일을 Markdown으로 변환합니다.
    ```bash
    pandoc input.html -f html -t markdown -o output.md
    ```
