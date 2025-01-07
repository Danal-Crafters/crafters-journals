![img](https://miro.medium.com/v2/resize:fit:700/1*xvBkQs9Ye_MncN0LKcOb4g.png)



# 개요

웹서비스를 개발할 때 프론트엔드 영역에서는 여러 라이브러리 중에 React.js(이하 리액트)를 채택하여 많이 사용하고 있습니다.

실제로 상당수의 국내 서비스에서 프론트엔드 영역은 리액트를 사용하고 있고, 프론트엔드 신입 개발자라면 선택이 아닌 필수적으로 갖춰야 할 역량이 되어가고 있습니다.

어떤 문제를 해결하고자 리액트를 사용하고, 얻게 되는 이점은 무엇이며 다른 대안은 없는지 혹은 미래에도 리액트를 사용하게 될 것 같은지 되짚어보며 생각한 주관적인 내용을 글로 옮겨 봤습니다.

# 첫 단추

리액트를 처음 공부할 때 공식 레퍼런스가 아닌 한글로 정리된 블로그의 글을 읽다 보면 잘못된 정보를 습득하게 되는 경우가 종종 있습니다.

예시로 ‘리액트를 사용하는 이유’ 라는 키워드로 검색을 했을 때 나오는 정보중 대표적으로 아래 세 가지가 있습니다

1 . 컴포넌트를 만들어 UI를 공통화하고 재사용 할 수 있다
2 . 리액트는 빠르다
3 . 재사용성이 높아 생산성이 증가된다

‘리액트를 왜 사용하시나요’ 라는 질문에 세 답변 중 맞는 답변은 무엇일까요?
세 답변 모두 리액트의 특징은 맞을 수 있지만, 리액트를 사용하는 근본적인 이유는 될 수 없습니다.

## **1. 컴포넌트를 만들어 UI를 공통화하고 재사용 할 수 있다**

리액트의 가장 큰 특징 중 컴포넌트 단위의 개발을 통한 UI 재사용 및 공통화는 리액트만의 특징 혹은 장점이 아닙니다. Vue.js Angular.js Svelte 등 여러 라이브러리에서 컴포넌트 방식의 모델링을 지원합니다.

심지어 라이브러리를 사용하지 않고도 Web API를 사용해 재사용 가능한
웹 컴포넌트를 구현할 수 있습니다.

```html
  <greeting-message name="Alice"></greeting-message>
  <greeting-message name="Bob"></greeting-message>

  <script>
    // 커스텀 엘리먼트 정의
    class GreetingMessage extends HTMLElement {
      constructor() {
        super();
        // Shadow DOM을 사용하여 내부 내용 캡슐화
        this.attachShadow({ mode: 'open' });
      }

      // 속성 변경을 감지
      static get observedAttributes() {
        return ['name'];
      }

      // 속성 값 변경 시 처리
      attributeChangedCallback(name, oldValue, newValue) {
        if (name === 'name') {
          this.render(newValue);
        }
      }

      // 렌더링 함수
      render(name) {
        this.shadowRoot.innerHTML = `
          <style>
            p {
              font-size: 20px;
              color: #333;
            }
          </style>
          <p>Hello, ${name}!</p>
        `;
      }

      // 요소가 연결되었을 때 호출되는 콜백
      connectedCallback() {
        this.render(this.getAttribute('name') || 'Guest');
      }
    }

    // 커스텀 엘리먼트 등록
    customElements.define('greeting-message', GreetingMessage);
  </script>
```

그리고 실제로 유튜브에서도 리액트가 아닌 web components 기반의 라이브러리인 polymer.js를 사용하고, 깃허브에서는 리액트와 HTML 템플릿 라이브러리인 lit-html을 함께 사용하기도 합니다

## 2. 리액트는 빠르다

`빠르다`라는 기준이 없는 추상적인 평가에서 참인지 거짓인지 답을 내릴 수 없습니다. 여러 가지 기준과 상황에 따라 빠를 수도 있습니다.
하지만 기본적으로 언어 그 자체인 javascript와 javascript로 쓰여진 라이브러리인 리액트를 비교한다면 당연히 순수한 자바스크립트만을 사용했을 때 보다 빠를 수 없습니다.

![img](https://miro.medium.com/v2/resize:fit:700/1*2rhXLxbIHKRBDZaZnvKDLw.png)

[js-framework benchmark](https://krausest.github.io/js-framework-benchmark/current.html)

또한 다른 라이브러리와 프레임워크를 비교한 벤치마크 표를 참고했을 때 리액트의 performance는 대부분 상위권에 속하지 않습니다.

![img](https://miro.medium.com/v2/0*iNKql1EJPom_4AEn.jpeg)

[solid with react](https://filipjerga.medium.com/solid-js-vs-react-js-what-is-better-ab25b2338c61)

그리고 얼마 전까지 순수 리액트만을 사용했을 때보다 70%가 빠르다는 문장으로 주목을 받았던 million.js와 virtual DOM을 사용하지 않고 높은 성능을 갖춘 solid.js와 같은 라이브러리도 존재합니다.
https://dev.to/composite/reactwa-solidyi-caijeom-topabogi-4e23

![img](https://miro.medium.com/v2/resize:fit:700/0*rlq_iAaZVqQpYB4_)

[million.js](https://million.dev/)

(물론 million.js는 React.js의 대안이 아닙니다 리액트의 성능을 높여주는 라이브러리입니다.)

## 3 . 재사용성이 높아 생산성이 증가된다

이 역시 여러 상황과 조건에 따라 참이 될 수도 거짓일 수도 있습니다.
작은 규모의 서비스나 빠르게 만들어야 하는 프로토타입일 때는 리액트를 사용하여 UI를 컴포넌트 단위로 설계하는고 개발하는 것은 오히려 불필요한 파편화나 오버헤드가 발생할 여지가 있습니다.

또한 당연한 말이지만 어느 정도 규모가 있는 서비스에서도 컴포넌트의 추상화를 ‘**잘**’ 해야지만 재사용성이 높아지는 것이고 무분별한 컴포넌트 생성은 오히려 개발자로 하여금 복잡도가 높아지거나 성능 최적화에 걸림돌이 될 수 있습니다

위처럼 세 가지 특징을 비판적인 관점으로 바라봤을 땐 리액트의 순수한 장점으로 보이지 않을 수 있습니다

보통 리액트의 특징을 사용했을 때 얻게 되는 장점으로 오인하고 있기 때문이기도 합니다.

# 리액트의 선언적인 방식

리액트를 사용하는 근본적인 이유는 `**선언적 프로그래밍`**이 가능하다는 부분에 있습니다.

리액트를 사용했을 때와 그렇지 않을 때 예시 코드를 간단하게 두 가지 준비했습니다

```html
  <p>현재 카운트: <span id="countValue">0</span></p>
  <button onclick="addCount()">카운트 증가</button>
  
  <script>
    let count = 0;
    const countDisplay = document.getElementById('countValue');
    
    function addCount(){
      count++;
      render();
    }
  
    function render(){
      countDisplay.textContent = count;
    }
  </script>
```

버튼을 클릭했을 때 count라는 변수를 증가시고, count를 기준으로 요소의 textContent를 변경하는 render 함수를 호출하여 UI를 업데이트하고 있습니다. 한 개의 요소를 변경하는 코드이며 어느 정도 규모가 있는 서비스를 위와 같이 구현한다면 복잡도는 크게 증가할 것입니다.

다음은 리액트를 사용한 코드입니다.

```javascript
const [count, addCount] = useState(0);
return (
  <p>현재 카운트: <span>{count}</span></p>
  <button onClick={() => addCount(count++)}>카운트 증가</button>
)
```

앞선 자바스크립트와 마찬가지로 count++를 통해 카운트를 증가시킵니다.
증가된 count 변수는 컴포넌트의 return문(JSX) 내에 포함되어 있고, 리액트에서 내부적으로 state가 변경되었을 때 리렌더링이 발생하면서 DOM Update를 해주기 때문에 직접 count에 따른 UI 변경 코드를 작성해 주지 않아도 됩니다.

리액트의 컴포넌트 로직에서는 사용자에게 무엇을 렌더링 하여 보여줄지만 신경 쓰면 됩니다. 즉 view를 선언해놓고 state만 변경하면 됩니다.

# 단면과 단면

복잡한 서비스에서 HTML view를 update 하기 위한 로직을 직접 작성해야 하는 것과 아닌 것은 분명 유의미한 차이가 있을 수 있습니다.

그렇다고 선언적인 코드를 작성하기 위해서라는 한 가지의 이유로 리액트를 선택하고 앞서 말씀드린 세 가지 특징이 장점이 아니라는 이야기 또한 당연히 아닙니다.

## 리액트와 웹 컴포넌트

![img](https://miro.medium.com/v2/resize:fit:700/1*qlWpepAJbonqIBWANpNfDg.png)

https://legacy.reactjs.org/docs/web-components.html

레거시 리액트 문서에 기재된 웹 컴포넌트에 관한 이야기입니다.
리액트를 단순 캡슐화된 UI 컴포넌트를 위한 라이브러리로 생각한다면 차이가 실감 나지 않을 수 있지만, 웹 컴포넌트와 리액트는 각각 해결하고자 하는 문제가 다릅니다.

![img](https://miro.medium.com/v2/resize:fit:700/1*O0SDEDxtMC3Nl4LtpOYyUg.png)

https://github.com/facebook/react/issues/11347

리액트에서 또한 웹 컴포넌트를 다양한 접근 방식으로 지원하기 위한 계획이 과거부터 현재까지 지속적으로 진행되오고 있습니다.

## 리액트는 결코 느리지 않다

과거 `리액트는 보통의 경우 빠르다` 라는 주장을 한 유명한 개발자는 리액트의 핵심 개발팀의 일원이며, 리액트와 관련된 여러 가지 중요한 개념을 제시한 인물로 잘 알려져 있습니다.

그가 리액트가 `빠르다` 라고 말한 이유를, 리액트가 virtual DOM을 사용해 실제 DOM에 접근하는 횟수를 최소화하고, UI 업데이트를 최적화하는 방식으로 동작하기 때문이라고 설명합니다.

리액트가 일반적으로 성능 면에서 우수하지만, 실제 성능은 개발자가 작성한 코드와 애플리케이션의 특정 요구 사항에 따라 달라질 수 있다는 것입니다.

또한 리액트보다 성능이 좋은 도구도 여러 가지가 존재하지만, 보통의 경우 리액트 자체도 빠르고 결코 느리지 않다는 것은 정립이 된 내용이기도 합니다.

# 내가 리액트를 사용하는 이유

리액트를 사용한 서비스를 개발해 본 개발자라면 보통의 경우 한두 가지의 장점 때문이 아닌 종합적으로 만족도가 높기에 다시 사용을 하고 저명한 대표적인 라이브러리로 자리했다고 생각을 합니다.

우선 리액트를 보통 프레임워크가 아닌 라이브러리라고 부릅니다.
실제로 리액트를 새로운 서비스를 만드는 것이 아닌, 기존에 사용하던 서비스에서 부분적으로 사용하여 점진적인 개선이 가능하며, 자유도 또한 높습니다. 자유도가 높다는 것은 리액트를 사용한 서비스 개발이 객관적인 한 가지의 정답에 국한되어 있지 않다는 것을 의미합니다.
물론 현재는 어느 정도 정형화된 리액트 디자인 패턴이 다수 존재하기도 합니다.

특정 기술을 도입하여 사용할 때 풍부한 레퍼런스와 커뮤니티는 생산성과 지속성에 큰 영향을 줍니다. 리액트의 경우 Meta(Facebook)에서 지속적으로 개발과 관리를 하는 오픈소스 프로젝트이며, 현재 Vercel 팀과도 긴밀히 협업을 통해 문제를 해결해나가고 있습니다.

높은 접근성과 사용성 그리고 안정성이 있는 라이브러리로써 생산성 증대가 어느 정도 보장되어 있는 라이브러리를 마다할 필요는 없다고 생각합니다.

# 결론

리액트를 단순히 다들 많이 사용해서, 국밥 라이브러리여서라는 이유보단 조금 더 원초적인 접근으로 시작을 해보는 것 또한 나쁘지 않다고 생각을 합니다.

현재 리액트는 단순 프론트엔드 영역에서 UI를 개발하기 위한 라이브러리 이상의 역할을 하고 있습니다. 단순 브라우저 런타임에 국한되지 않고 RSC(React Server Component)를 사용하거나 리액트 기반의 프레임워크인 Next.js Remix Astro 등 많은 선택지 또한 존재합니다.

새로운 페러다임이 등장했을 때도 여전히 최고의 라이브러리로 자리매김한다는 보장은 없지만, 현시점 높은 생산성을 제공하기 위한 라이브러리임은 틀림없고 앞으로도 당분간 많은 개발자가 선호하는 라이브러리일 것입니다.

# 참조

https://filipjerga.medium.com/solid-js-vs-react-js-what-is-better-ab25b2338c61
https://dev.to/composite/reactwa-solidyi-caijeom-topabogi-4e23|
https://dev.to/nikl/react-is-slower-than-vanilla-js--pfo
https://krausest.github.io/js-framework-benchmark/current.html
https://developer.mozilla.org/en-US/docs/Web/API/Web_components
https://pyjun01.github.io/v/million-js/
