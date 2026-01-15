---
title: "양자 알고리즘 1 - 큐비트"
date: 2025-03-09 13:18:08
categories: ["Quantum Algorithm"]
tags: ["Quantum"]
---

### Before starting

이 시리즈는 양자 알고리즘을 다루고 있지만 양자역학을 다루진 않는다. 양자 알고리즘을 이해하는 데에 있어 필요한 최소한의 선수지식을 철저히 내 시선에서 이해한 대로 작성하는 연재글이니 참고할 것.

### Simple Prerequisites

본격적인 이야기를 시작하기 전에, 양자 컴퓨팅에서 잘 써먹는 양자의 특징적인 현상 몇 개를 먼저 정리한다.  

**양자 중첩(Superposition)**이란 양자가 가능한 상태가 있으면 이들 모두의 상태를 동시에 갖고 있을 수도 있음을 뜻하며, 그 값을 측정하는 순간에 1개의 상태로 정해짐을 뜻한다.  
보통 구슬 모양의 자석에 비유하는데, 만일 이 구슬에 어떠한 표시도 없다면 어디가 N극이고 어디가 S극인지는 측정하기 전에는 아무도 모를 것이다. 즉, N극과 S극이 "중첩"된 상태가 되는 것이다. 그럼 만일 이 구슬을 측정하기 위해 위쪽이 N극, 아래쪽이 S극으로 이루어진 측정 터널에 통과시켰다고 생각해보자.  만약 터널의 중심에서 N극까지의 거리와 S극까지의 거리가 동일하다면 구슬의 절반은 위로 휘게 되고, 나머지 절반은 아래로 휘게 될 것이다. 그리고 한번 아래로 휜 상태로 통과한 구슬을 그대로 다시 통과시켜도 여전히 아래로 휘게 될 것은 자명하다. 이런 식으로 양자를 측정하게 되면 1가지 상태로 고정되고, 별도의 조작이 없으면 그 고정된 상태가 계속 유지된다.  
그럼 이 양자를 다시 측정되기 전 상태로 돌려놓는 방법은 없을까? 방법은 의외로 간단한데, 측정에 사용한 터널을 $$90^\circ$$ 돌려서 한번 아래로 휜 구슬을 다시 통과시키면, 이 구슬들은 이제 절반은 왼쪽으로, 절반은 오른쪽으로 휘게 될 것이다. 즉 측정된 양자를 다시 중첩된 상태로 돌려놓는 것 또한 가능하며, 양자 컴퓨팅에서 이 연산(Hadamard transform이라고 한다.)은 굉장히 중요하다. 약간의 과장을 보태면 양자 컴퓨팅을 하는 가장 주요한 이유가 이 중첩 상태이기 때문.  

**양자 얽힘(Entanglement)**이란 서로 연관된 두 개의 양자 중 하나의 상태가 관측되면 다른 하나의 상태를 즉각적으로 알 수 있음을 뜻한다다. 이 즉각적이라는건 말 그대로 즉시 알 수 있다는 의미로, 어떠한 딜레이도 소요되지 않는다. 자석을 잘라서 2개의 자석으로 만들었다고 가정하자. 위에서 했던 것과 동일하게 측정해서 한 쪽 자석의 극의 방향을 알게 됐으면, 다른 쪽 자석은 굳이 측정하지 않아도 극의 방향이 어디를 향하는지는 바로 알 수 있게 된다. 그리고 이 과정에선 어떠한 딜레이도 없으며, 거리의 제한 또한 없다는 것 역시 자명하다. 이런 식으로 서로 얽혀있는 양자는 별도의 과정 없이도 서로 연관된 상태가 될 수 있다.  
이 얽힘 상태를 이용해서, 한 쪽의 상태를 측정해서 고정시키면 여기에 얽혀있는 다른 쪽의 양자의 정보를 바탕으로 측정된 양자의 정보를 복원할 수 있다. 이건 마치 양자가 거리와 시간을 무시하고 복사된 듯한 효과를 주기에 **양자 순간이동(Teleportation)**으로 불리지만, 양자가 물리적으로 순간이동하는 것은 아니고 그 양자가 갖고 있던 정보만 옮겨진다는 개념이다.  


### Qubit

고전적인(즉, Non-quantum) 컴퓨터에서 정보를 다루는 최소 단위는 bit이며 다들 알겠지만 이 bit가 다룰 수 있는 정보는 0과 1의 2가지이다.  

그렇다면 양자 컴퓨터에서는 어떨까? 양자 컴퓨터에서 정보를 다루는 최소 단위는 quantum bit, 즉 qubit라고 한다. 양자를 다루는 컴퓨터답게 qubit는 양자 역학의 원리를 이용한 bit이다. 이 말은 즉, qubit 1개에 여러 값을 공존시킬 수 있으며, 측정된 순간 그 중 어느 하나로 값이 정해진다는 의미이다.  
그런데 양자 1개에 여러 값을 공존시킬 수 있으면 굳이 0과 1로만 나눌 필요도 없지 않을까? 그래서 보다 일반적인 개념으로 qudit라는 것이 등장한다. 굳이 0과 1에 한정하지 않고, 양자 1개가 임의의 N개의 값을 가지는 시스템이다. $$N = 2$$라면 qubit고, $$N = 3$$이라면 qutrit라고 부르는 식이다. 그런데 $$N > 2$$일 경우 하드웨어적으로 이를 효율적으로 구현하기도 어렵고, 소프트웨어적으로도 이를 잘 써먹기 어렵다. 그래서 보통은 그나마 친숙하고 설계하기도 편한 qubit로 대부분의 양자컴퓨터 및 알고리즘을 설계하고 있다.  

그렇다면 만일 qubit를 중첩 상태 없이 0과 1로만 고정시켜서 사용하면 고전적인 컴퓨팅과 동일한게 아닌가? 맞긴 하다. 실제로 중첩을 아예 배제한 채로 qubit를 사용하게 되면 이는 bit와 하등 다를게 없어진다. 이 말은 양자컴퓨터는 (이론상) 고전 컴퓨터가 할 수 있는 모든 일을 할 수 있다. 하지만 지금 qubit는 매우 비싼 자원이라 그렇게 쓸 수 없다. bit는 현재 펑펑 써도 괜찮은 자원이지만 (1GB가 약 86억 bit이다) qubit는 1개 단위로 쓰는 매우 비싼 몸이시다...  

그럼 이런 qubit를 어떻게 수학적으로 기술할 수 있을까? 보통 양자는 Dirac notation으로 기술한다. qubit 또한 양자이므로 이 표기법을 그대로 쓰는 편이다. 다만 qubit는 측정된 값이 0 혹은 1로만 나오도록 가공되었으므로 다음과 같이 간단하게 기술할 수 있다.  
$$\ket{\Psi} = \left[\begin{array}{c}\Psi_{1}\\\Psi_{2}\end{array}\right],\space\space|\Psi_{1}|^{2} + |\Psi_{2}|^{2} = 1$$  
즉, 2차원 공간에서 길이가 1인 벡터를 이용하여 기술한다. 또한 측정 결과로 나올 수 있는 0, 1값은 다음과 같이 정의된다.  
$$\ket{0} = \left[\begin{array}{c}1\\0\end{array}\right],\space\space\ket{1} = \left[\begin{array}{c}0\\1\end{array}\right]$$  
따라서 임의의 qubit는 다음과 같이 표현할 수 있다.  
$$\ket{\Psi} = \left[\begin{array}{c}\Psi_{1}\\\Psi_{2}\end{array}\right]=\Psi_{1}\ket{0} + \Psi_{2}\ket{1}$$  
0의 상태와 1의 상태가 각각 $$\Psi_{1}$$, $$\Psi_{2}$$만큼씩 중첩되어 있으며 이들을 각각 제곱하면 관측할 때 0이 나올 확률과 1이 나올 확률을 알 수 있다. 당연하지만 그 확률의 합은 1이어야 한다.  

이제 아래의 두 qubit를 생각해보자.  
$$\ket{\Psi_{1}} = \frac{1}{\sqrt{2}}\ket{0} + \frac{1}{\sqrt{2}}\ket{1},\space\space\ket{\Psi_{2}} = \frac{1}{\sqrt{2}}\ket{0} - \frac{1}{\sqrt{2}}\ket{1}$$  
qubit $$\Psi_{1}$$과 $$\Psi_{2}$$는 모두 각각의 상태가 나올 확률이 50%씩이다. 그럼 같은 것이 아니냐고 할 수 있지만, 측정의 방향의 따라서 이 두 qubit는 다른 값을 가질 수 있기 때문에 다른 상태로 취급된다. 이와 같이, 방향까지 중요하게 다뤄지는 구형으로 생각해야 하며 실제로도 그렇다. 그러면 2차원 공간에 한정지어서 생각할 이유도 없지 않을까?  

![bloch_sphere]({{ "/assets/quantum/bloch_sphere.png" | absolute_url }})  

그래서 위와 같은 구체로도 qubit를 정의할 수 있으며, 이를 Bloch sphere라고 부른다. 3차원 구체형이기 때문에 축은 기존의 $$\ket{0}$$, $$\ket{1}$$ 외에도 $$\ket{+}$$, $$\ket{-}$$, $$\ket{i}$$, $$\ket{-i}$$의 4가지가 더 있으며, 다음과 같이 정의한다.  
$$\ket{+} = \frac{1}{\sqrt{2}}\ket{0} + \frac{1}{\sqrt{2}}\ket{1},\space\space\ket{-} = \frac{1}{\sqrt{2}}\ket{0} - \frac{1}{\sqrt{2}}\ket{1},\space\space\ket{i} = \frac{1}{\sqrt{2}}\ket{0} + \frac{i}{\sqrt{2}}\ket{1},\space\space\ket{-i} = \frac{1}{\sqrt{2}}\ket{0} - \frac{i}{\sqrt{2}}\ket{1}$$  
그리고 임의의 qubit에 대한 일반적인 형태는 다음과 같이 정의할 수 있다.  
$$\ket{\Psi} = \cos\theta\ket{0} + \sin\theta e^{i\varphi}\ket{1}$$  


### Quantum Logic Gate

지금까지는 qubit 1개에 대해서만, 그것도 어떤 식으로 표현하는가에 대해서만 알아봤다. qubit는 어디까지나 정보만을 가지고 있으므로 이제 이 정보를 어떤 연산을 이용해서 다룰 수 있는지 알아보자.

#### Unitary operation

1개의 qubit는 2x1 행렬로 표기되므로 이를 대상으로 하는 unitary operation은 2x2 행렬로 표기된다.

- **I**(Identity) Gate  
    $$I = \left[\begin{array}{cc}1&0\\0&1\end{array}\right]$$  
    I Gate는 qubit를 그대로 유지시키는 항등연산자이다.

- **H**(Hadamard) Gate  
    $$H = \frac{1}{\sqrt{2}}\left[\begin{array}{cc}1&1\\1&-1\end{array}\right]$$  
    H Gate는 qubit를 다시 50:50의 중첩 상태로 바꿔주는 연산자이다. 위에도 말했지만 이미 측정된 결과만 가지고 연산을 수행하는 것은 고전적인 bit와 다를게 없기 때문에, 측정된 qubit를 다시 중첩상태로 돌려놓는 이 연산자는 양자 컴퓨팅에서 매우 중요하다.

- Pauli **X**, **Y**, **Z** Gates / Phase Shift Gate / Rotation Gates  
    $$X = \left[\begin{array}{cc}0&1\\1&0\end{array}\right],\space Y = \left[\begin{array}{cc}0&-i\\i&0\end{array}\right],\space Z = \left[\begin{array}{cc}1&0\\0&-1\end{array}\right]$$  
    이 연산들은 bloch sphere에서 각각 X, Y, Z축으로 뒤집는 연산이다. 굳이 각 연산의 효과를 적자면 X 연산은 0과 1이 등장할 확률을, Y 연산은 phase를, 그리고 Z 연산은 확률의 방향을 반전시킨다. 다만 X 연산자의 경우 고전 컴퓨팅에서의 NOT gate와 그 결과가 동일하기 떄문에 X 연산자를 NOT 연산자로도 부른다.  
    Z 연산자의 일반화된 버전으로, 반전이 아니라 특정 각도만큼 회전시키는 연산을 생각해볼 수 있다.  
    $$P(\varphi) = \left[\begin{array}{cc}1&0\\0&e^{i\varphi}\end{array}\right]$$  
    이 연산은 Phase shift gate라고 부르며, qubit의 phase만 변화시키는 연산이다. 특별한 케이스로 $$P(\pi/4)$$를 S Gate, $$P(\pi/8)$$를 T Gate로 부른다. 당연히히 $$S = T^2$$이다.  
    조금 더 일반화된 버전으로, X, Y, Z 축을 중심으로 각각 일정 각도만 회전시키는 연산을 생각해볼 수 있다.  
    $$R_{x}(\theta)  = \left[\begin{array}{cc}\cos\frac{\theta}{2}&-i\sin\frac{\theta}{2}\\-i\sin\frac{\theta}{2}&\cos\frac{\theta}{2}\end{array}\right]$$  
    $$R_{y}(\theta)  = \left[\begin{array}{cc}\cos\frac{\theta}{2}&-\sin\frac{\theta}{2}\\\sin\frac{\theta}{2}&\cos\frac{\theta}{2}\end{array}\right]$$  
    $$R_{z}(\theta)  = \left[\begin{array}{cc}e^{\frac{-i\theta}{2}}&0\\0&e^{\frac{i\theta}{2}}\end{array}\right]$$  


#### Binary operation

당연한 말이지만, 2개 이상의 qubit를 동시에 다룰 경우에는 이들의 tensor product로 표현한다. 즉 binary operation은 4x4 행렬로 표기된다.

- **CNOT**(Controlled Not)  
    $$CNOT = \left[\begin{array}{cccc}1&0&0&0\\0&1&0&0\\0&0&0&1\\0&0&1&0\end{array}\right]$$  
    CNOT, 혹은 NOT이 곧 X이므로 CX Gate라고도 부르는 이 연산자는 첫 번째 qubit의 값에 따라 두 번째 qubit에 행해질 연산이 달라지게 된다. 보다 구체적으로, 첫 번째 qubit가 $$\ket{0}$$이라면 아무 연산을 하지 않지만, 첫 번째 qubit가 $$\ket{1}$$이라면 두 번째 qubit에 NOT 연산을 적용한다. 같은 방식으로, CY, CZ 연산도 생각해볼 수 있다.  
    CX Gate는 아무 상관 없는 두 qubit를 서로 얽을 수 있게 해주는 중요한 연산이다. 위의 H Gate와 더불어서, 이 연산은 두 고전 컴퓨팅에서는 이룰 수 없는 연산을 수행하는 양자 컴퓨팅의 핵심 연산 중 하나이다.

- **SWAP** Gate  
    $$SWAP = \left[\begin{array}{cccc}1&0&0&0\\0&0&1&0\\0&1&0&0\\0&0&0&1\end{array}\right]$$  
    이름에서도 잘 드러나듯 두 qubit의 값을 바꿔준다.

#### Ternary operation

위와 같은 이유로 ternary operation은 8x8 행렬로 표기된다.

- **CCNOT** Gate  
    $$CCNOT = \left[\begin{array}{cccccccc}1&0&0&0&0&0&0&0\\0&1&0&0&0&0&0&0\\0&0&1&0&0&0&0&0\\0&0&0&1&0&0&0&0\\0&0&0&0&1&0&0&0\\0&0&0&0&0&1&0&0\\0&0&0&0&0&0&0&1\\0&0&0&0&0&0&1&0\end{array}\right]$$  
    CCNOT Gate는 Toffoli 연산자로도 불리며, 첫 2개의 qubit가 모두 $$\ket{1}$$이어야만 세 번째 qubit에 대해 NOT 연산을 적용한다. NOT 연산자는 곧 X 연산자이므로 CCX 연산으로도 불린다.

- **CSWAP** Gate  
    $$CSWAP = \left[\begin{array}{cccccccc}1&0&0&0&0&0&0&0\\0&1&0&0&0&0&0&0\\0&0&1&0&0&0&0&0\\0&0&0&1&0&0&0&0\\0&0&0&0&1&0&0&0\\0&0&0&0&0&0&1&0\\0&0&0&0&0&1&0&0\\0&0&0&0&0&0&0&1\end{array}\right]$$  
    CSWAP Gate는 Fredkin Gate로도 불리며, 첫 qubit가 $$\ket{1}$$이면 다음 2개의 qubit에 대해 SWAP 연산을 적용한다.
    