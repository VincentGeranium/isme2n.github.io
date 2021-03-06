---
layout: post
title:  "[함수형 자바스크립트]3. 자료구조는 적게, 일은 많이"
subtitle:   "함수형 자바스크립트"
categories: devlog
tags: nodejs javascript

---

자료구조를 순차적으로 탐색/변환하는데 쓰이는 실용적인 연산 몇가지(map, reduce, filter)를 알아보자. 대부분의 루프는 이들이 처리하는 하나의 특정 케이스에 지나지 않으므로 수동 루프를 없애는 목적으로도 쓰인다.

## 애플리케이션의 제어 흐름

프로그램이 정답에 이르기까지 거치는 경로를 `제어 흐름`이라고 한다. 명령형 프로그램은 작업 수행에 필요한 모든 단계를 노출하여 흐름이나 경로를 상세히 서술한다.

반면, 선억적 프로그램, 특히 함수형 프로그램은 독립적인 블랙박스 연산들이 단순하게 최소한의 제어 구조를 통해 연결되어 추상화 수준이 높다. 이렇게 연결한 연산들은 각자 다음 연산으로 상태를 이동시키는 고계함수에 불과하다. 실제로 함수형 프로그램은 데이터와 제어 흐름 자체를 고수준 컴포넌트 사이의 단순한 연결로 취급한다.

```js
opA().opB().opC().opD(); // 체이닝으로 간결한 연결
```

연산을 체이닝하면 간결하게 표현적인 형태로 프로그램을 작성할 수 있어 제어 흐름과 계산 로직을 분리할 수 있고 코드와 데이터를 더욱 효과적으로 관리 할 수 있다.

## 메서드 체이닝

`메서드 체이닝`은 메서드를 단일 구문으로 호출하는 객체지향적 패턴이다. 변이를 일이크지 않는 선에서 함수형 프로그래밍에서도 단일 객체 인스턴스에 속한 메서드를 체이닝하는 건 나름대로 쓸모가 있다.

```js

'Functional Programming'.substring(0,10).toLowerCase() + ' is fun';

// 위의 코드를 함수형으로 리팩토링 한다면

concat(toLowerCase(substring('Functional Programming', 1, 10)), ' is fun');

```

아래처럼한다면 속에서부터 코드를 이해해야하기 때문에 코드를 이해하기에 어렵다.

## 함수 체이닝

객체지향 프로그램은 주로 상속을 통해 코드를 재사용한다. `List`컴포넌트가 ArrayList, LinkedList... 등등 존재하는 게 그 예다.

함수형 프로그래밍은 접근 방법이 조금 다르다. 자료구조를 새로 만들어 어떤 요건을 충족시키는것이 아니라, 배열 등의 자료구조를 이용해 다수의 굵게 나뉜 고계 연산을 적용한다. 작업 내용은 다음과 같다.

- 작업을 수행하기 위해 무슨 일을 해야 하는지 기술된 함수를 인수로 받는다.
- 임시 변수의 값을 계속 바꾸면서 부수효과를 일으키는 기존 수동 루프를 대체한다. 관리할 코드가 줄고 에러가 날 만한 코드가 줄어든다.

### 람다 표현식

람다 표현식은 `=>` 화살표 함수라고도 한다. 함수형 프로그래밍은 람다 표현식과 잘 어울리는 주요 고계함수 `map, reduce, filter`를 적극 사용할 것을 권장한다. 로대시js 함수로 공부해보자.

### _.map: 데이터 변환

덩치 큰 데이터 컬렉션의 원소를 모두 변환해야 할 때가 있다. `map` 함수는 컬렉션의 원소를 전부 파싱할 경우 아주 유용한다. 항상 새로운 배열을 반환하기 때문에 불변성도 유지된다.

> tip: 로대시 함수에서 _(something)로 감싸면 객체를 로대시 함수로 자유롭게 사용할 수 있다.

### _.reduce: 결과를 수집

데이터를 변환하였다면 그 데이터로 의미 있는 결과를 도출하고 싶을것이다. 이 일은 `reduce` 함수가 전문성이 있다.

`reduce`는 원소 배열을 하나의 값으로 짜내는 고계함수이며, 원소마다 함수를 실행한 결괏값의 누적치를 계산한다.

```js

_(persons).reduce((stat, person) => {
    const country = person.address.country;
    stat[country] = _.isUndefined(stat[country]) ? 1 : stat[country] + 1;
    return stat;
}, {})

/*
{
    'US' : 2,
    'Greece' : 1,
    'Hungary' : 1
}
*/

```

맵 리듀스 조합을 이용하면 더 단순화해 작업할 수 있다.

```js
const getCountry = person => person.address.country;

const gatherStats = function (stat, criteria) {
    stat[criteria] = _.isUndefined(stat[criteria]) ? 1 : stat[criteria] + 1;
    return stat;
};

_(persons).map(getCountry).reduce(gatherStats, {});
```

### _.filter: 원하지 않는 원소를 제거

map이나 reduce는 모든 원소를 탐색하기 때문에 계산하지 않을 원소는 사전에 빼놓는게 좋다. 

```js
_(persons).filter(isValid).map(fullname);
```

선언적 개발은 개발자가 문제의 해법에 어떻게 도달해야 하는지 고민하기보다 어플리케이션이 어떤 결과를 내야하는지에 전념하게한다.

## 코드 헤아리기

함수형 흐름은 프로그램 로직을 파헤치지 않아도 뭘 하는 프로그램인지 윤곽을 잡기 쉽다. 그로인해 개발자는 코드뿐만 아니라, 결과를 내기 위해 서로 다른 단계를 드나드는 데이터의 흐름까지 더 깊이 헤아릴 수 있다.

### 선언적 코드와 느긋한 함수 체인

함수형 프로그램은 단순 함수들로 구성한다. 개별 함수가 하는 일은 작지만, 뭉치면 복잡한 작업도 해낼 수 있다. 아래 예제는 가장 많은 사람들이 사는 나라를 출력하는 예제이다.

```js
_.chain(persons)
    .filter(isValid)
    .map(_.property('address.country'))
    .reduce(gatherStats, {})
    .values()
    .sortBy('count')
    .reverse()
    .firse()
    .value()
    .name; // 'US'
```

> tip: _.chain()은 _.(something) 과 같은 일을 하지만 value()를 호출하기전까지 값을 반환하지 않는다.

## 재귀적 사고방식

복잡한 문제를 풀 수 있는 가장 뚜렷한 방법은 문제를 분해하는 것이다. 그 작은 문제들을 하나하나 풀어나가면 전체 문제도 풀 수 있다.

### 재귀란

재귀는 주어진 문제를 자기 반복적인 문제들로 잘게 분해한 다음 이들을 다시 조합해 원래 문제의 정답을 찾는 기법이다. 재귀 함수는 `기저 케이스`와 `재귀 케이스`로 이루어져있다.

`기저 케이스`는 재귀 함수가 구체적인 결괏값을 바로 계산할 수 있는 입력 집합이다. `재귀 케이스`는 함수가 자신을 호출할 때 전달한 입력 집합(점차 작아진다)을 처리한다.

재귀 케이스를 돌다가 가장 마지막에 기저케이스에 도달하면 하나의 값으로 귀결된다. 고등수학에서 배웠던 `점화식`을 떠올리면 된다.
