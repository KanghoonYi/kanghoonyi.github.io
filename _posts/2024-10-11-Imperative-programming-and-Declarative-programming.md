---
title: Imperative programming(명령형 프로그래밍)과 Declarative programming(선언형 프로그래밍)
author: KanghoonYi
name: KanghoonYi(pour)
date: 2024-10-11 23:01:00 +0900
categories: [Programming, Computer Basics]
tags: [Computer Science, programming]
pin: false
math: true
---

Computer Engineering을 할때, 개념적인 부분에 대한 이해 없이도 뭔가를 만들어낼 수 있습니다.  
하지만 이는 곧 한계에 부딪힙니다.  
‘Imperative programming’과 ‘Declarative programming’는 개념으로 들어서는 ‘아하..’하면서 이해는 되지만, 매번 체득되지 않는.. 그런것이었습니다.  
도서 ‘Designing Data-Intensive Application’를 읽고, 이에 대해 깨닳은 바가 있어. 이를 공유하고자 합니다.

## What is it?

`Imperative programming`과 `Declarative programming`은 Computer에게 ‘명령을 내리는 방법’에 대한 얘기입니다.  
‘Imperative’와 ‘declarative’는 서로 추구하는바가 다르고, 용도가 다릅니다.

## Imperative programming(명령형 프로그래밍)

‘Imperative’는 결과를 얻기 위한 과정을 단계별로 직접 작성하는 방법입니다.

> An imperative language tells the computer to perform **certain operations** in **a certain order**. You can imagine stepping through the code line by line, evaluating conditions, updating variables, and deciding whether to go around the loop one more time.

우리가 사용하는 java, c, python, javascript,… 등등이 이에 해당합니다.  
예를 들면,  
아래와 같이, ‘animals’ array에서 shark를 찾아내기 위해, 각 단계를 programming합니다.

```javascript
function getSharks() {
	var sharks = [];
	for (var i = 0; i < animals.length; i++) {
		if (animals[i].family === "Sharks") {
			sharks.push(animals[i]);
		}
	}
	return sharks;
}
```

> 흔히, 컴퓨터가 작동하는 방식과 같다고 얘기합니다.
{: .prompt-tip }

## Declarative programming(선언형 프로그래밍)

‘Declarative’는 이미 약속된 interface로 원하는 결과물을 선언(Declarative)하는 방식입니다.

> you just specify the pattern of the data you want—what conditions the results must meet, and how you want the data to be transformed (e.g., sorted, grouped, and aggregated)—but not how to achieve that goal.
>

대표적으로 SQL, HTML, XSL 등이 이에 해당합니다.  
예를 들면,  
아래와 같이 SQL로 기대하는 결과물을 작성할 수 있습니다.  
```sql
SELECT * FROM animals WHERE family = 'Sharks';
```

혹은 HTML과 CSS 예시가 있습니다

```html
<ul>
	<li class="selected">
		<p>Sharks</p>
		<ul>
			<li>Great White Shark</li>
			<li>Tiger Shark</li>
			<li>Hammerhead Shark</li>
		</ul>
	</li>
	<li>
		<p>Whales</p>
		<ul>
			<li>Blue Whale</li>
			<li>Humpback Whale</li>
			<li>Fin Whale</li>
		</ul>
	</li>
</ul>
```

```css
li.selected > p { background-color: blue; }
```

여기서 CSS code는 특정 DOM element(HTML을 구성하는 기본요소)의 ‘background-color’를 지정하고 있습니다.  
‘background-color’를 바꾸는 과정에 대한 구체적인 지시 없이, 시스템이 이를 이해하여 결과물을 만듭니다.  

## ‘Imperative programming’과 ‘Declarative programming’ 비교

**Imperative programming**는
: computer에게 보다 구체적으로 명령을 내릴 수 있습니다.
: 대부분의 programming language가 이에 해당하며, 작성된 language에 dependency(의존성)를 갖습니다. 그래서, 각 programming language별로 별도로 programming해야 합니다.

**Declarative programming**는
: 결과물에 이르는 ‘세부 과정(implementation details)’을 숨기고, 시스템이 이를 해석해서 작동합니다.
: 특정 programming language에 dependency가 걸리지 않으며, ‘Declarative Language’를 이해하도록 language상관없이 구현할 수 있습니다.
: Language에 대해 ‘보편적인’ Optimizer를 통해 성능을 끌어올릴 수 있습니다. 이는 ‘imperative’에서 각 로직에 맞게 최적화 해줘야하는것과는 달리, 보편적인 optimizer를 통해 성능 최적화를 할 수 있습니다.
: > it also hides implementation details of the database engine, which makes it possible for the database system to introduce performance improvements without requiring any changes to queries.
: ‘parallel execution(병렬 실행)’구현이 상대적으로 쉽습니다(구현이 상대적으로 자유롭기 때문에). 이는 ‘실행에 순서가 있어’ 병렬실행 '구현'이 어려운 'Imperative'와 두드러지는 차이점입니다.
: > declarative languages often lend themselves to parallel execution.

## References

Imperative Programming
: [Imperative programming](https://en.wikipedia.org/wiki/Imperative_programming)
