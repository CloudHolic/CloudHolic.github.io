---
title: "Monad 4 - Distributive Law"
date: 2026-03-15 23:49:00 +0900
categories: ["Study", "Functional"]
tags: ["Functional"]
math: true
---

## Before starting

이 글은 [3편](https://cloudholic.github.io/posts/monad-3/)에서 다뤘던 내용 중, 모나드 간의 분배법칙에 관한 보충 설명이다. 굳이 해당 내용을 글 하나로 뺄 정도로, 꽤나 길고 복잡한 내용이 섞여 있다.


## Functor, Applicative의 합성

먼저, 3편에서는 Functor와 Applicative는 합성에 대해 닫혀있다고 했는데, 왜 그런지부터 확인해보자.


Functor의 경우, 두 Functor F, G를 합성해도 다음과 같이 Functor를 구현할 수 있다.

``` haskell
fmap :: (a -> b) -> F(G a) -> F(G b)
fmap f = fmap_F (fmap_G f)
```

즉, `fmap`의 인자로 주어진, 값 변환 함수 `a -> b`를 안쪽까지 밀어넣기만 하면 되므로, Functor는 합성 연산에 대해서 닫혀있음을 알 수 있다.


Applicative의 경우, 두 Applicative F, G를 합성해도 다음과 같이 Applicative를 구현할 수 있다.

``` haskell
pure :: a -> F(G a)
pure = pure_F . pure_G

(<*>) :: F(G (a ->b)) -> F(G a) -> F(G b)
(<*>) f x = fmap_F (<*>_G) f <*>_F x
```

여기도 같은 원리로, 각각의 레이어에서 pure와 Apply를 순차적으로 적용하면 되니 Applicative도 합성 연산에 대해서 닫혀있다.


이러한 구현 외에도, Functor와 Applicative는 둘 다 컨텍스트 내부의 값을 변환할 수만 있으면 되기 때문에, 즉 필요한 연산이 단방향이기 때문에 그냥 순차적으로 적용하면 되는 것이다.


## What about Monad?

### Join

그럼 모나드에선 어떻게 바뀔까? 모나드에 대한 합성을 정의하기 전에, `join`함수를 살펴봐야 한다.

우리는 여태 모나드를 `return`과 `Bind`로 정의했었다. 그러나 사실, 수학적으로는 `return`과 `join`으로 정의된다. `join`은 다음과 같이 정의된 함수다.

``` haskell
join :: M (M a) -> M a
```

즉, 2중첩으로 된 모나드에서 중첩을 한 개 지워내는 함수이다. 모나드는 Functor이기 때문에 `fmap`이 제공된다. 그런데 `fmap`을 반복적으로 적용하다보면 컨텍스트가 계속 중첩되게 된다. `fmap`은 컨텍스트를 추가만 하기 때문이다. 그래서 컨텍스트를 유지하면서 계산을 하기 위해서는 이 중첩된 컨텍스트를 제거할 방법이 필요하다. 그렇게 해야지만 비로소 계산을 자유롭게 할 수 있어 모나드라고 부를 수 있게 된다.

그리고 우리가 흔히 알고 있는 `Bind`는 사실 `fmap`과 `join`으로 구현이 가능하다.

``` haskell
m >>= f = join (fmap f m)
```

당연하지만, 컨텍스트가 몇 번 중첩되든 한 겹만 남기고 제거하기 위해서는 지울 때도 한 겹씩 제거해야만 한다. 그렇기 때문에 `join`함수는 반드시 1개 단위로 컨텍스트를 제거해야 한다.


### Monad Composition

그럼 이제 모나드의 합성을 생각해보자.

``` haskell
return :: a -> M(N a)
return = return_M . return_N

join :: M(N(M(N a))) -> M(N a)
```

우선, 위에서의 고찰을 통해 `Bind` 대신 `join`에 대해서만 생각해도 충분하다. 그리고 `return`의 경우 합성이 매우 잘 된다. 사실 Applicative의 `pure`와 동일해서 더 볼 필요도 없다.

문제는 `join`인데, `M(N(M(N a))) -> M(N a)`를 구현해야 하고, 우리가 가진건 다음의 두 함수 뿐이다.

``` haskell
join_M :: M(M a) -> M a
join_N :: N(N a) -> N a
```

이걸 사용해서 줄이기 위해서는 `N(M a) -> M(N a)`가 필요하다. 이게 있어야지 `M(N(M(N a))) -> M(M(N(N a))) -> M(N(N a)) -> M(N a)`의 순서를 거쳐 `M . N`에 대한 `join`을 구현할 수 있다.


즉, 모나드의 합성을 정의하려면 함수 `N(M a) -> M(N a)`가 필요하며, 이는 분배법칙에 해당한다. 결국 모나드에서 분배법칙이 적용되느냐가 모나드의 합성이 가능한지로 귀결된다.


## Distributive Law between Monads

근데 모나드 간의 분배법칙이 성립하는지 아닌지는 직관적으로 알긴 너무 어렵다. 그래서 실제로 어떻게 증명되었나를 살펴볼 건데, 여기서는 2006년에 발표된 Varacca & Winskel의 논문 [Distributing Probability over Non-determinism](https://www.researchgate.net/publication/220173520_Distributing_Probabililty_over_Nondeterminism)에서 제시된 방법을 소개할 것이다. 


어떤 것이 불가능함을 증명하는 가장 대표적인 방법은 귀류법이다. 또한 불가능을 증명할 때에는 반례를 하나 찾기만 해도 충분하다. 이들의 증명은 여기서 출발한다.

논의를 시작하기 앞서, $\lambda: D \circ P \Rightarrow P \circ D$라는 분배법칙이 성립하기 위해서는 다음의 4개의 조건을 모두 만족해야 한다는 공리가 존재한다.

- A1: $\lambda \circ \eta^D_{P(X)} = P(\eta^D_X)$
- A2: $\lambda \circ D(\eta^P_X) = \eta^P_{D(X)}$
- A3: $\lambda \circ \mu^D_{P(X)} = P(\mu^D_X) \circ \lambda_{D(X)} \circ D(\lambda_X)$
- A4: $\lambda \circ D(\mu^P_X) = \mu^P_{D(X)} \circ (P(\lambda_X)) \circ \lambda_{P(X)}$

여기서 $\eta$는 각 모나드의 unit을 나타내는 자연 변환이고, $\mu$는 모나드의 곱셈을 의미하는 자연 변환이다. 위의 4가지 공리는 각각 D의 unit 보존, P의 unit 보존, D의 곱셈 보존, P의 곱셈 보존을 의미한다.


즉, 두 모나드를 적절히 찾아서 이들은 분배법칙이 성립하지 않는다는 것을 증명하면 되는 것이고, 이들 간의 분배법칙이 성립하지 않는다는 것을 증명하기 위해 귀류법을 적용하여 일단 된다고 가정하고 모순을 이끌어낼 것이다.

여기서 정한 두 개의 모나드는 다음과 같다.

- Finite Nonempty Powerset Monad P: 확률과 무관하게 시스템이 여러 선택지 중 하나를 선택하는 상황
- Finite Valuation Monad D: $p$의 확률로 A를, $(1-p)$의 확률로 B를 선택하는 등의 확률적 상황

즉, 이들 사이의 분배법칙이 성립한다는 소리는, "집합들의 확률분포"를 "확률분포들의 집합"으로 바꿀 수 있다는 이야기이다. 그럼 이러한 조건을 만족하는 분배법칙 $\lambda$가 무엇을 만족해야 하는지 살펴보자.


### Step 1

위의 공리 A1에 의해서, 집합 $S$에서 확률이 1이 되는 point mass $\sigma_S$에 대해 $\lambda$는 다음을 만족해야 한다.

$$\lambda_X(\sigma_S) = \lbrace \sigma_x \mid x \in S \rbrace$$

예를 들어 $S = \lbrace a, b \rbrace$라면 $\lambda_X(\sigma_{\lbrace \lbrace a, b \rbrace \rbrace}) = \lbrace \sigma_a, \sigma_b \rbrace$가 된다.