# 마크다운 도장깨기 - 노트부터 ppt까지 (2)

안녕하세요,   
돌아온 마크다운 짱팬 BE플랫폼개발팀 정찬미입니다.  
지난 포스팅에서 소개해드린 마크다운, '백문이불여일타' 해보셨을까요?!  
시도해보신 분, 해보려다 까먹으신 분, 이미 잘 알고 계신 분 모두 좋습니다.  

오늘은 마크다운으로 프리젠테이션자료를 만들 수 있는 방법을 소개해드리겠습니다.  
어렵지 않습니다. 마크다운 문법만 안다면 쉽게 따라할 수 있습니다.
그것이 바로 마프(`Marp`) 입니다.

마프는 css를 커스텀해 테마로 사용할 수도 있습니다.   
예시로 보여드리려고 다날 다크 테마를 만들어봤습니다.
다날 테마 적용하는 방법도 소개할테니 끝까지 읽어주세요!

---

## Marp란?

> **Marp**는 Markdown을 기반으로 프레젠테이션을 제작할 수 있는 오픈소스 도구입니다.  


```
---
marp: true
---
```

VS Code 기준으로 플러그인을 설치하고, 마크다운 문서 맨위에 marp를 사용한다는 헤더를 넣기만 하면 marp가 적용됩니다. 이제  Markdown으로 작성하면 PPT가 뚝딱 만들어집니다.  
모든 내용이 Github에 공개된 오픈소스이고 무료로 사용할 수 있습니다!

![Marp](https://raw.githubusercontent.com/marp-team/marp-vscode/main/docs/screenshot.png)

## Marp 설치 및 사용법

Marp팀에서는 VS Code 를 사용하는 걸 추천합니다.  
설치 방법은 매우 간단합니다.

### **1. VS Code 확장 프로그램 설치**
- VS Code의 **Extensions**(확장 프로그램)에서 `Marp for VS Code`를 검색하고 설치합니다.
- 또는 아래 링크에서 직접 설치할 수 있습니다.  
  👉 [Marp for VS Code](https://marketplace.visualstudio.com/items?itemName=marp-team.marp-vscode)  

### **2. Markdown 파일에서 Marp 사용하기**
- `.md` 파일을 만들고, 맨 위에 `---`를 추가하여 슬라이드를 구분하면 끝!
- `Ctrl + Shift + P` 를 눌러 `Markdown Preview to the side` 메뉴를 선택해 미리보기를 보며 작업할 수 있습니다.

### **3. PDF 또는 PPT로 내보내기**
- 작성한 슬라이드를 PDF, PPTX 파일로 변환할 수 있습니다.
- `Ctrl + Shift + P`를 눌러 `Marp: Export slide deck`을 검색하고 선택하면 됩니다.

## Marp 특징

### **단순하고 효율적이다**

무엇보다 복잡한 GUI 없이 Markdown만으로 프레젠테이션을 제작할 수 있습니다.  
PowerPoint도 물론 좋은 툴이고, 확장성이 좋습니다. 저도 초등학생땐 마우스 피하기 게임
만들고, 대학생때는 PowrPoint 템플릿 만든 나름 헤비유저였습니다.

![](https://i.namu.wiki/i/mrzqLyow6lZ8uaGUSKIQ-t-5drkKc8c2tPsjeoW-N77hB8BRTmAjPp9gDlll0ocGqUOv6C5R2Iaa1JdGLo_KaA.webp)  

그러나 PowerPoint는 기능이 많고 GUI 기반이라 수기 작업이 많습니다.  
단축키와 다양한 기능을 자유자재로 사용하는 PPT 마스터가 되기까진 험난한 길이 있습니다.  
반면 마프는 마크다운만 알면 누구나 쉽게 사용할 수 있습니다.

### **보편적인 마크다운 문법 채택**  

Marp는 CommonMark 기반의 마크다운 문법을 채택하고 있습니다.  
즉, 우리가 GitHub, Notion 등에서 사용하는  
마크다운 문법을 거의 그대로 사용할 수 있다는 뜻입니다.   
약간 특이한 렌더링 방식을 사용하는 툴이라면 가끔식 유저 입장에서 왜 안되는건지 곤란할 때도 있으니까요.

>  **CommonMark란?**  
>
> CommonMark는 마크다운의 공식 표준 규격으로,  
GitHub에서도 채택하고 있는 마크다운 문법입니다.  


### **Marp에서 지원하는 추가 문법**  

일반적인 CommonMark에 더해, Marp가 추가로 제공하는 기능들입니다.
 
- **슬라이드 구분**: 기본적으로 `---` 를 사용하여 새로운 슬라이드를 만들 수 있음 
  ```
  ---
  marp: true
  ---
  # 첫번쨰 슬라이드
  ---
  # 두번쨰 슬라이드
  ```

- **배경 이미지 설정**:  bg로 슬라이드의 배경이미지와 위치, 크기를 설정할 수 있음
  ```markdown
  ![bg left:50%](https://source.unsplash.com/random)
  ```

- **수식**:  

```
  ---
  marp: true
  math: katex
  ---
```

   - MathJax와 Katex를 지원합니다.  맨위 헤더부분에  `math: katex` 를 추가하면 됩니다.
   - 마프팀은 MathJax를 선호하지만, Katex가 렌더링 속도가 빠르고 지원 문법이 더 다양합니다.  
  - auto shrink 기능이 있어 수식이 길어져도 자동으로 폭에 맞추어 줄어듭니다.  
    
    ```markdown
    $$ E = mc^2 $$
    ```
    
> 🔍  이렇듯 Marp는 단순히 마크다운을 PPT로 변환하는 기능만 있지 않습니다.
**여러가지 유용한 기능이 추가된 발표용 마크다운**이라고 생각하시면 됩니다.


### **테마 커스텀 가능**  

Marp는 기본적으로 **3가지 공식 테마**를 제공합니다.  

1. **default (기본 테마)**  
   - 심플한 스타일로, 검은색 텍스트에 흰 배경을 사용  
   - 표준적인 슬라이드 느낌  
   - `theme: default`
   


2. **gaia**  
   - 큰 제목과 여백이 강조된 디자인  
   - 노란 배경
   - `theme: gaia`

3. **uncover**  
   - 어두운 배경과 강한 색상의 텍스트  
   - `theme: uncover`

테마 적용 방법도 간단합니다.
```markdown
---
marp: true
theme: gaia
---
```


##  커스텀 테마 만들기

세가지 기본 테마가 깔끔하긴한데, 개인적으로는 글자가 일반적인 PPT 폰트보다 훨씬 크다거나, 색상조합이 클래식하다는 인상을 받았습니다.  

Marp는 CSS와 SCSS를 사용하여 새로운 테마를 만들 수도 있습니다.  

예를 들어, 특정 배경 색상을 입히고 싶다면, 이렇게 하시면 됩니다.

```
---
marp: true
theme: default
backgroundColor: #1e1e1e
color: white
---
```

좀 더 세부적으로 테마를 조정하고 싶다면 **Custom Theme 파일**을 만들어 적용할 수도 있습니다.

그래서, 짧은 css 가방끈으로 다날 다크 테마를 만들어봤습니다. 제가 쓰려고 만들었으니 아무도 안쓰셔도 슬퍼하지 않을겁니다!

이제 VS Code에 커스텀 테마 적용하는 방법을 소개하겠습니다. 

### 1. 현재폴더에 다날 테마 파일을 다운로드합니다.

첨부한 `danal_style.scss` 파일을 다운로드 받고, 현재 VS Code에 열려있는 폴더에 넣어주세요.

  ```css
 * Marp Danal Style Dark Theme.
 *
 * @theme danal_style
 * ...
  ```
  
### 2.   테마 파일을 VS Code에 적용합니다.

   - `Ctrl + Shift + P`(Mac은 `Command + Shift + P` )를 눌러 Command Palette를 열고, `Marp: Show Quick Pick Of Marp Commands...`을 검색합니다.
 - `Open Extension Settings`를 선택하고, `Marp: Themes`를 클릭합니다.
- `Marp: Themes` 설정에서 `./danal_style.scss`를 추가로 기입합니다. (`./`는 현재 위치한 폴더를 의미합니다. )

### 3. 마크다운 파일에서 해당 테마를 적용합니다.
   ```
   ---
   marp: true
   theme: danal_style
   ---
   ```


이제 슬라이드 전체가 다날 스타일로 적용됩니다!

> 📌 **왜 Marp 테마는 SCSS를 사용할까요?**  
> Marp의 테마는 일반 CSS가 아니라 **SCSS**(Sassy CSS)로 작성됩니다.  
> 이유는 SCSS가 **변수**와 **중첩 스타일링**을 지원해서 테마를 쉽게 커스텀할 수 있기 때문입니다.  
> Marp의 기본 테마도 github markdown css를 오버라이디한 형태의 SCSS로 만들어졌으며, `marp-core`의 GitHub 저장소에서 확인할 수 있습니다.

## Marp를 어디에 활용하면 좋을까요?

제가 직접 해보거나 들어본 사례들을 소개해드리겠습니다.

### 기술 발표
- 코드를 많이 포함하는 발표에 유용합니다.
- GitHub에서 마크다운 문서로 작성하고 바로 발표할 수 있어 효율적입니다.

### 논문 및 학술 발표

- LaTeX보다 가벼운 대안으로 사용할 수도 있습니다.
- 수식을 포함한 발표에 유용합니다.
- 인공지능 대학원을 다니는 친구는 마프로 논문 발표자료를 만드니 생산성이 대폭 향상되었다고 합니다.
- 수식 템플릿을 사용해 수학 학습지를 만든 사례도 있습니다. (유료)

### 팀 교육 자료
-  팀 내에서 **교육자료**를 빠르게 공유하고 수정할 수 있습니다.
- Git과 연동하면 **버전 관리**도 편리합니다.

---

프리젠테이션 자료를 준비하는 일은 시간과 노력이 많이 들어가는 작업입니다.  
이제 마프로 보다 가벼운 발표생활하시길 응원합니다.
마크다운 시리즈는 2편으로 끝내려합니다.  
사용하시다 궁금한 점이 있으면 언제든 물어봐주세요!

---

## **References**

- [Marp 공식 사이트](https://marp.app/)
- [Marp가 CommonMark 기반이라는 내용](https://github.com/marp-team/marp-core?tab=readme-ov-file#marp-markdown)
- [Github Markdown Dark CSS](https://github.com/sindresorhus/github-markdown-css/blob/main/github-markdown-dark.css)
- [Marp default css](https://github.com/marp-team/marp-core/blob/main/themes/default.scss)
