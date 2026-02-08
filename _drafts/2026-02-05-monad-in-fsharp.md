---
title: "Monad in F#"
date: 2026-02-08 13:28:30 +0900
categories: ["Study", "Functional"]
tags: ["Functional", "F#"]
math: true
---

### Before Reading

![저번 글](https://cloudholic.github.io/posts/monad/)에서 모나드에 대해 알아봤다. 모나드를 한 마디로 요약하면 side-effect를 함수형 언어의 방식으로 처리하는 이론적 배경이자 디자인 패턴이다. 그만큼 함수형 언어에서의 핵심 개념이기 때문에 함수형이 주인 언어에서는 이 모나드를 사용할 다양한 방법들을 매우 직설적으로 제공해준다. 실제로 하스켈에서는 이 모나드가 이름 그대로 Monad라고 정의되어 있으며, 그 내용도 모나드의 수학적 정의와 크게 다르지 않다. 그런데 유독 .NET 기반의 함수형 언어인 F#에서는 모나드라는 단어가 직접적으로 표시되지 않고 숨겨져 있다.

### Computation Expression

우선 F#에선 모나드 대신 "Computation Expression", 즉 계산식이라는 명칭을 사용한다. 명칭뿐만 아니라 사용하는 형태도 좀 특이한데, 다음과 같이 사용한다.

``` fsharp
builder-expr { cexpr }
```

`builder-expr`에 그 계산식의 이름이 들어가며, 이후 중괄호 안에 계산식 내에서 해야하는 작업들이 들어가는 형태다. F#에서 기본으로 제공하는 계산식들을 보면서 실제 사용법을 파악해보자.


#### seq

`seq`를 이해하기 위해선 F#의 `seq<'T>` 타입에 대해 알아야 한다. 크게 어려운 개념은 아닌데, 배열, 리스트, 기타 다른 모든 컬렉션의 부모격이 되는 놈이시다. 즉, C#의 `IEnumerable<T>`와 동일하다고 볼 수 있다. 그리고 `seq` 


#### async


#### task


#### query