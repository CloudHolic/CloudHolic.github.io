---
title: "Monad in F#"
date: 2026-02-05 13:28:30 +0900
categories: ["Study", "Functional"]
tags: ["Functional", "F#"]
math: true
---

### Before starting

저번 글에서 모나드에 대해 알아봤다. 한 마디로 요약하면 모나드는 side-effect를 함수형 언어의 방식으로 처리하는 이론적 배경이자 디자인 패턴이다. 그리고 하스켈에서는 이 모나드가 이름 그대로 Monad라고 정의되어 있으며, 모나드의 수학적 정의와 크게 다르지 않다. 그런데 다른 함수형 언어인 F#에서는 모나드라는 개념이 직접적으로 표시되지 않고 숨겨져 있다.

### Computation Expression

