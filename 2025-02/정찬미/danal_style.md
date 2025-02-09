---
marp: true
paginate: true
theme: danal_style
math: katex
---

<!-- Cover Slide -->
<section class="cover-slide">
  <h1>Hello, Marp!</h1>
  <p>Markdown으로 만드는 프레젠테이션</p>
</section>

---

<!-- 목차 -->
# 목차

<hr/>

- Marp란?
- Marp의 주요 특징
- 설치 및 활용 방법
- 테마 커스터마이징
- 활용 사례 및 예제
- 참고 자료
- Q & A

---

<!-- 여기 발표 스크립트를 넣고 export할수도 있습니다-->
# Marp란?

<hr/>

> ### Markdown 기반 프레젠테이션 제작 도구

<br> 


### 단순하고  쉽습니다!

- GUI 없이 Markdown 만으로 아이디어를 빠르게 PPT  형태로 만들어낼수 있습니다.
 
 ### **다양한 출력 포맷 지원**
- HTML, PDF, PPT 등 여러 포맷으로 내보내기가 가능합니다.

###  **오픈소스 프로젝트**

- 누구나 자유롭게 사용하고 기여할 수 있습니다.

 
 [Marp 공식 사이트](https://marp.app/) 

---


# 그외 Marp의 특징

<hr/>


### 일반적인 Markdown 문법 채택
- **CommonMark 채택**: GitHub 등에서 사용되는 표준 마크다운 문법과 호환됩니다.

###  커스터마이징
- **테마 커스텀**: CSS/SCSS를 이용해 자신만의 테마를 만들 수 있습니다.

- [Marp core의 테마 예제](https://github.com/marp-team/marp-core/blob/main/themes/example.md)

---

<!-- 설치 및 활용 방법 -->
# Marp 설치 및 활용 방법

<hr/>

### VS Code 환경에서 시작하기

1. **Marp for VS Code 확장 설치**
   - [Marp for VS Code](https://marketplace.visualstudio.com/items?itemName=marp-team.marp-vscode)를 설치합니다.

2. **Markdown 파일 설정**
   - `.md`파일에 아래와 같이 헤더?를 추가해 Marp 모드를 활성화합니다.
     ```markdown
     ---
     marp: true
     theme: danal_style
     ---
     ```

3. **슬라이드 작성 및 미리보기**
   - `---` 구분자를 사용해 각 슬라이드를 나누고, VS Code 기본기능인 Markdown 프리뷰를 통해 바로 결과를 확인할 수 있습니다.

자세한 사용법은 [Marp Markdown 문서](https://github.com/marp-team/marp-core?tab=readme-ov-file#marp-markdown)에서 확인하세요.

---

<!-- 테마 커스터마이징 -->
# 테마 커스터마이징

<hr/>

Marp는 기본 테마 외에도 나만의 CSS를 적용할 수 있습니다.

- **기본 테마 수정**: SCSS를 활용해 폰트, 색상, 레이아웃을 자유롭게 조정.
- **VS Code 설정 연동**: `"markdown.marp.themes"` 설정을 통해 외부 CSS 파일을 불러올 수 있습니다.
- **실시간 미리보기**: 변경 사항은 즉시 프리뷰에 반영되어 빠른 피드백을 받을 수 있습니다.

참고) 다날 스타일 커스텀 테마는 [GitHub Markdown Dark CSS](https://github.com/sindresorhus/github-markdown-css/blob/main/github-markdown-dark.css)와 Marp 기본 테마를 조합해 제작되었습니다.

---

<!-- 활용 사례 및 예제 -->
# 활용 사례 및 예제

<hr/>

### 코드 예제
Marp 슬라이드에서 코드 블록은 구문 하이라이팅이 적용되어 가독성이 뛰어납니다.

```kotlin
fun main(args: Array<String>) {
    println("Hello, Marp!")
}
```

---

# 이미지 및 인포그래픽
<hr/>  

활용할 수 있는 아이콘을 만들고 배경 이미지를 적용해봤습니다.

<div class="infographic">
  <div class="icon">마</div>
  <div class="text">마크다운으로</div>
  <div class="icon">프</div>
  <div class="text">프리젠테이션 끝내자</div>
</div>

![bg](https://images.rawpixel.com/image_800/czNmcy1wcml2YXRlL3Jhd3BpeGVsX2ltYWdlcy93ZWJzaXRlX2NvbnRlbnQvbHIvcm0zMDktYWV3LTAxM18xXzEuanBn.jpg)

---

# 표 활용

<hr/>  

표로 데이터를 깔끔하게 정리할 수 있습니다.

| AA       | BBBB   |
|:----:|:----:|
| 설치       | Marp CLI  |
| 문법       | Markdown |
| 커스터마이징 | CSS/SCSS |

---

# 수식 

<hr/>  

Render inline math such as $ax^2+bc+c$.

$$ I_{xx}=\int\int_Ry^2f(x,y)\cdot{}dydx $$

$$
f(x) = \int_{-\infty}^\infty
    \hat f(\xi)\,e^{2 \pi i \xi x}
    \,d\xi
$$

---

#  References 

<hr/>
  

- **Marp 공식 사이트**: [https://marp.app/](https://marp.app/)
- **Marp core 테마 예제**: [예제 마크다운](https://github.com/marp-team/marp-core/blob/main/themes/example.md)
- **Marp Markdown 사용법**: [Marp Markdown](https://github.com/marp-team/marp-core?tab=readme-ov-file#marp-markdown)
- **Github Markdown Dark CSS**: [Dark CSS](https://github.com/sindresorhus/github-markdown-css/blob/main/github-markdown-dark.css)

---

<section class="cover-slide">
  <h1>Q & A</h1>
  <p>질문은 언제나 환영입니다!</p>
</section>
