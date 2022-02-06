---
title: 'Introduction to testing UIs'
tocTitle: 'Introduction'
description: 'UI 테스트를 위한 최신 개발 방법'
commit: 'f9a12b8'
---

<div class="aside">이 가이드는 JavaScript와 React, 그리고 Storybook에 <b>숙련된 개발자</b>를 위해 작성되었다. 만약 스토리북 사용에 익숙하지 않다면 <a href="/intro-to-storybook">Intro to Storybook</a>을 방문해 기초부터 배워보기를 추천한다!
</div>

<br/>

//아래 한 문단은 의역했습니다.

UI 테스트는 까다롭다. 사용자는 새로운 기능이 자주 출시되기를 기대한다. 하지만 새로운 기능이 출시된다는 건 이전보다 더 많은 양의 UI와 state를 테스트해야 한다는 뜻이다. 모든 테스팅 도구는 "쉽고, 신뢰할 수 있으며, 빠를 것" 을 보장하지만, 그 어려움에 대해서는 가볍게 넘어간다.

선두적인 프론트엔드 팀을 어떻게 따라갈 수 있을까? 그들은 대체 어떤 테스트 전략과 메소드를 사용할까? 우리는 스토리북 커뮤니티에서 10팀을 골라 - wilio, Adobe, Peloton, Shopify 등 - 그들을 조사해보았다.

이 가이드는 규모있는 엔지니어링 팀이 사용하는 UI 테스트 기술을 소개한다. 이를 통해 테스트의 적용 범위, 설정 및 유지보수의 균형을 유지하는 실용적인 방법을 배울 수 있을 것이다. 또한 테스트 과정에서 피해야 할 함정도 알려주도록 하겠다.

## 어떤 것들을 테스트해야 할까?

널리 사용되는 JavaScript 프레임워크는 모두 [컴포넌트 기반 (componet-driven)](https://www.componentdriven.org/) 으로 동작한다. 즉, UI는 가장 작은 단위인 Atomic 컴포넌트부터 시작해서 점진적으로 페이지가 되는 "bottom-up" 방식으로 구성된다.

오늘날 UI를 구성하는 모든 조각이 컴포넌트라는 것을 기억하자. 물론 페이지도 마찬가지다. 페이지와 버튼의 차이점은 데이터를 소비하는 방식뿐이다.

결국 UI 테스트는 곧 컴포넌트 테스트인 셈이다. 


<img style="max-width: 400px;" src="/ui-testing-handbook/component-testing.gif" alt="Component hierarchy: atomic, compositions, Pages and Apps" />

유닛 테스트(Unit Test), 통합 테스트(integration), e2e 테스트 등으로 구분하는 건 다소 모호할 수 있다. 그 대신 UI에서 어떤 요소들이 테스트가 가능한지 살펴보자. 

#### 비주얼 (Visual)

비주얼 테스트(Visual test)는 컴포넌트가 일련의 props 및 state에 대해 올바르게 렌더링되는지 확인한다. 그들은 모든 컴포넌트의 스크린샷을 찍은 뒤 커밋과 커밋을 비교하여 변경 사항을 식별한다.

#### 구성 요소 (Composition)

컴포넌트들은 데이터의 흐름을 따라 서로 연결되어 있다. 상위 레벨의 컴포넌트나 페이지에서 비주얼 테스트를 실행해보면 그들의 긴밀한 연결을 확인할 수 있을 것이다.  


#### 상호작용 (Interaction)

상호작용 테스트(Interaction test)는 이벤트가 의도한 대로 처리되는지 검증하는 작업이다. 분리한 컴포넌트를 렌더링한 다음, 클릭이나 입력 같은 사용자 동작을 시뮬레이션한다. 그 뒤 최종적으로 state가 올바르게 업데이트 되었는지 확인한다.  

#### 접근성 (Accessibility)

접근성 테스트는 시각장애, 청각장애 등 다양한 장애와 관련된 사용성 문제를 검증한다. 노골적인 접근성 위반을 포착할 수 있도록 Axe 같은 자동화 도구를 QA의 첫번째 줄에 사용한다. 그런 다음 인간의 주의가 필요한 까다로운 문제를 위해 실제 디바이스에서 수동으로 QA를 수행한다.

#### 유저 플로우 (User flow)

간단한 작업이라도 사용자는 여러 컴포넌트에 걸쳐 일련의 단계를 완료해야 한다. 이는 또 다른 잠재적인 실패가 될 수 있다. Cypress 및 Playwright와 같은 도구를 사용하면 전체 애플리케이션에 대해 테스트를 실행하여 이러한 상호 작용을 확인할 수 있다.

## 워크플로우에 대한 이해

UI에서 테스트가 필요한 다양한 측면을 다루었지만, 이를 워크 플로우에 결합하는 것은 까다롭다. 무언가 틀렸다간 UI 개발 과정이 고되게 느껴질 수 있다. 변경사항이 있을 때마다 테스트가 중단되기 때문에 모든 툴에 테스트 케이스를 복제해야 하고, 이 모든 것은 "유지 관리의 악몽"이 된다.

우리가 조사한 팀들은 팀의 규모와 기술 스택이 각기 달랐음에도 불구하고 비슷한 전략을 가지고 있었다. 그 내용을 취합해 효율적인 워크플로우로 만들면 다음과 같다: 



- 📚 [Storybook](http://storybook.js.org/)을 이용한 **컴포넌트 분리.** . Write test cases where each state is reproduced using props and mock data.
- ✅ [Chromatic](https://www.chromatic.com/)을 이용한 **시각적 버그 포착 및 구성요소 확인.** 
- 🐙 [Jest](https://jestjs.io/)와 [Testing Library](https://testing-library.com/)를 이용한 **interaction 검증.**
- ♿️ [Axe](https://www.deque.com/axe/)를 이용한 **접근성 심사.**
- 🔄 [Cypress](https://www.cypress.io/)를 이용해 e2e 테스트 코드를 작성하여 **유저 플로우 검증.**
- 🚥 [GitHub Actions](https://github.com/features/actions)을 통해 자동으로 테스트를 실행해 **회귀 포착.**

![](/ui-testing-handbook/ui-testing-workflow.png)

## 테스트를 시작해보자.

다음 장에서 우리는 테스트 스택의 각 계층에 대해 더 깊이 파고들어 이 테스트 전략을 구현하는 메커니즘에 대해 알아볼 것이다. 하지만 먼저 테스트해볼 무언가가 필요하다. Taskbox 앱을 예시로 사용하겠다. Taskbox는 Asana와 비슷한 작업 관리 앱이다.

![](/ui-testing-handbook/taskbox.png)

UI를 테스트하는 방법에 집중하고 있으므로, 구현의 세부 정보는 중요하지 않다. 또한 여기서 React를 사용했지만, 이러한 테스트 개념은 모든 컴포넌트 기반 프레임워크로 확장될 수 있다.

코드를 직접 보고 싶다면 이 저장소 https://github.com/chromaui/ui-testing-guide-code 를 fork한 다음, 아래의 커맨드를 따라하면 된다.

```sh
# Clone the forked repository
git clone https://github.com/<your_github_username>/ui-testing-guide-code

cd ui-testing-guide-code

# Install dependencies
yarn
```
