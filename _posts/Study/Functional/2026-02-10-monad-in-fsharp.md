---
title: "Monad in F#"
date: 2026-02-10 12:20:30 +0900
categories: ["Study", "Functional"]
tags: ["Functional", "F#"]
math: true
---

### Before Reading

[저번 글](https://cloudholic.github.io/posts/monad-1/)에서 모나드에 대해 알아봤다. 모나드를 한 마디로 요약하면 side-effect를 함수형 언어의 방식으로 처리하는 이론적 배경이자 디자인 패턴이다. 그만큼 함수형 언어에서의 핵심 개념이기 때문에 함수형이 주인 언어에서는 이 모나드를 사용할 다양한 방법들을 매우 직설적으로 제공해준다. 실제로 하스켈에서는 이 모나드가 이름 그대로 Monad라고 정의되어 있으며, 그 내용도 모나드의 수학적 정의와 크게 다르지 않다. 그런데 유독 .NET 기반의 함수형 언어인 F#에서는 모나드라는 단어가 직접적으로 표시되지 않고 숨겨져 있다.

### Computation Expression

우선 F#에선 모나드 대신 "Computation Expression", 즉 계산식이라는 명칭을 사용한다. 명칭뿐만 아니라 사용하는 형태도 좀 특이한데, 다음과 같이 사용한다.

``` fsharp
builder-expr { cexpr }
```

`builder-expr`에 그 계산식의 이름이 들어가며, 이후 중괄호 안에 계산식 내에서 해야하는 작업들이 들어가는 형태다. 그리고 계산식을 좀 더 편하게 쓸 수 있도록, 계산식 내에서만 사용 가능한 특수한 키워드들을 제공한다. 이들은 `let!`, `do!`, `and!` 등과 같이 원래의 키워드에 `!`가 붙은 형태로, 공통적으로 계산식 속에 있는 그 값을 직접 다루게 해준다.

``` fsharp
let result = 
    option {
        let! x = Some 5
        let! y = Some 3
        return x + y
    }
```

위 코드에서, `x`와 `y`에는 각각 `5`와 `3`이 바인딩되고, 그로 인해 `x + y`연산을 정상적으로 수행할 수 있게 해준다. 만일 `let!` 대신 `let`을 사용했다면, `x`는 `Some 5`, `y`는 `Some 3`이 그대로 바인딩될 것이라 `x + y`를 계산하기 위해 또 복잡한 방법을 사용해야 했을 것이다.
(사실 F#에서 `option` 계산식은 기본적으로 제공하는 계산식은 아니다. 하지만 대충 어떤 느낌인진 알지 않는가?)

그럼 이제 F#에서 기본으로 제공하는 아래 4가지 계산식들을 보면서 실제 사용법을 파악해보자.


#### seq

`seq` 계산식은 Sequence Builder이다. 즉, 계산식 안의 구문을 통해 `seq<'T>` 타입을 만들어내며, 이 `seq<'T>`라는 타입은 배열, 리스트, 기타 다른 모든 컬렉션의 부모격이 된다. 즉, C#의 `IEnumerable<T>`와 동일하다고 볼 수 있다. 

``` fsharp
let seqA = seq { 0..15 }    // 0, 1, ..., 15

let seqB = seq {
    yield 1
    yield 2
    for x in [3..10] do
        yield x * x
}   // 1, 2, 9, 16, 25, ..., 100

let seqC = seq {
    for _ in 1..10 do
        yield! seq { 1; 2; 3; 4; 5 }
}   // 1, 2, 3, 4, 5, 1, 2, 3, 4, 5, ..., 1, 2, 3, 4, 5
```

`seqA`와 `seqB`는 코드와 그 결과로 나올 시퀀스만 봐도 무슨 일을 하는지 손쉽게 알 수 있으니 넘어가자. 참고로 `yield` 키워드는 C#에서의 그것과 동일하다. 

`seqC`의 경우 `yield!`라는 키워드가 쓰였는데, 이는 `let`과 `let!`의 관계와 동일하다. 즉, 계산식 안의 내부 원소들을 꺼내서 `yield` 작업을 수행하는 것이라고 보면 된다. 이중 시퀀스를 단일 시퀀스로 펼치는 작업이기 때문에 이를 flatten이라고 부른다.

그 외에도 `Seq.empty`, `Seq.init`, `Seq.singleton` 등 다양한 보조 함수들이 정의되어 있다. 그런데 주의할 점은 `seq`에는 `let!` 바인딩이 존재하지 않는다. 이에 대한 자세한 내용은 후술한다.


#### async

`async`는 요즘 대부분의 언어는 다 갖고 있다는 바로 그 비동기 지원과 관련된 계산식이다. 사실 생각해보면 당연한게, 비동기 작업이야말로 정말 대표적인 "필수불가결한 side-effect"에 해당한다. 그러니 이를 함수형 언어의 방식으로 처리하려면 모나드 형태로 처리할 수 밖에 없다.

``` fsharp
let urlList = [ "Microsoft.com", "http://www.microsoft.com/"
                "MSDN", "http://msdn.microsoft.com/"
                "Bing", "http://www.bing.com" ]

let fetchAsync(name, url: string) =
    async {
        try
            let uri = new System.Uri(url)
            let httpClient = new HttpClient()
            let! html = httpClient.GetStringAsync(uri) |> Async.AwaitTask
            printfn "Read %d characters for %s" html.Length name
        with
            | ex -> printfn "%s" (ex.Message);
    }

let runAll() =
    urlList
    |> Seq.map fetchAsync
    |> Async.Parallel
    |> Async.RunSynchronously
    |> ignore

runAll()
```

위와 같이 우리가 생각하는 그 비동기 작업 그대로 사용하면 된다. 다만 몇 가지 특이한 점이 존재한다.

``` fsharp
let (result1 : Async<byte[]>) = stream.AsyncRead(bufferSize)

let! (result2 : byte[]) = stream.AsyncRead(bufferSize)
```
우선 계산식 안의 값을 직접 바인딩을 시켜주는 `let!`의 특성상 `async` 계산식을 `let!`으로 바인딩하게 되면 C#에서의 `await`와 같은 효과를 내게 된다. 즉, 해당 비동기 작업을 완료하고 그 값을 가져올 때까지 기다리게 된다. 또한 `use!` 키워드를 쓸 수 있는데, 이는 C#에서의 `await using`과 동일하다. 즉, `use`는 C#의 `IDisposable`을, `use!`는 C#의 `IAsyncDisposable`에 대응된다고 보면 된다.


#### task

`task` 계산식 역시 C#에서 너무나도 익숙한 `Task` 관련 지원이 포함되어 있다. 그런데 왜 `async`와 `task`가 분리되었을까? 실제로 둘은 하는 일이 비슷하고 C#에서도 둘을 크게 구분하진 않는다. F# 공식 문서에서는 .NET의 타 언어와의 interop, 혹은 (특히 C#의) `Task`와 직접적으로 연동할 일이 있을때 `task` 계산식을 쓰라고 권장하고 있다. 그 외에는 `async` 계산식과 개념적으로도 동일하고, 쓰는 것도 동일하다. 


#### query

`query`는 LINQ를 지원하기 위한 계산식이다. 즉, 안에서 C#에서의 그 LINQ 문법을 사용하고, 그 결과를 얻어낼 수 있다. `seq`에서 `yield`를 통해 하나씩 얻어내던 것처럼, `query`에서는 `select`로 같은 역할을 수행할 수 있다.

```fsharp
query {
    for customer in db.Customers do
        select customer
}
```

`query`는 `seq`와 비슷하게 `let!`이 제공되지 않는다. 그 대신 `query` 안에서 쓸 수 있는 다양한 전용 연산자들이 Query Operator라는 이름으로 제공되지만, 이 글에서 이걸 다루진 않는다. 해당 내용들은 LINQ나 SQL에 익숙하다면 어렵지 않게 사용할 수 있으니 궁금하면 관련 문서를 찾아보자.


### Custom Computation Expression

그럼 F#에서 지원하는 이 기본 계산식들 외에 추가로 계산식을 만들고 싶으면 어떻게 해야 할까? 아래와 같이 Builder `type`을 만들면 된다.

``` fsharp
type OptionBuilder() =
    member _.Bind(x, f) = Option.bind f x
    member _.Return(x) = Some x
    member _.ReturnFrom(x) = x
    member _.Zero() = None

let option = OptionBuilder()

let result =
    option {
        let! x = Some 5
        let! y = Some 3
        return x + y
    }   // Some 8
```

직관적으로 크게 어려운 방법은 아니다. 다만 위에서 `option` 계산식을 만들 때 정의한 `Bind`나 `Return`, `ReturnFrom`, `Zero` 등의 함수들은 다 이미 이름이 정의된 함수들이며, 이러한 함수를 구현해야 CE 전용 키워드들을 제대로 사용할 수 있게 된다. 다행히도 모든 함수를 다 구현할 필요는 없다. 필요한 함수들만 구현해두면 된다.

그럼 이제 정의된 함수들 중 중요한 것들만 살펴보자.

#### Bind

``` fsharp
member Bind: M<'T> * ('T -> M<'U>) -> M<'U>
```

Monad에서 정의된, `>>=` 연산자에 해당하는 함수이다. 계산식 내에서의 `let!` 혹은 `do!`로 시작하는 일련의 구문이 `Bind`로 변환되게 된다.

``` fsharp
builder {
    let! x = m
    return x + 1
}

builder.Bind(m, fun x -> builder.Return(x + 1))
```

즉, 위의 코드에서 `builder { ... }` 구문과 `builder.Bind(...)` 구문은 동일하다.


#### Return, ReturnFrom

```fsharp
member Return: 'T -> M<'T>
```

`Return`은 위의 `Bind`와 마찬가지로 역시 모나드에서 정의된 그 함수이며, 당연하지만 `Bind`와 `Return`이 같이 정의되어야 이 계산식이 모나드가 된다. 계산식 내에서는 `return` 키워드와 동일하다.

``` fsharp
builder {
    return 42
}

builder.Return(42)
```


``` fsharp
member ReturnFrom: M<'T> -> M<'T>
```

`ReturnFrom`은 이미 계산식에 있는 값을 그대로 반환해준다. 계산식 내에서는 `return!`키워드와 동일하다.

``` fsharp
builder {
    return! someComputation
}

builder.ReturnFrom(someComputation)
```


#### Zero

``` fsharp
member Zero: unit -> M<'T>
```

비어있는 계산식의 값을 정의한다. 즉, `if` ~ `then` 구문에서 `else`가 비어있으면 `Zero`가 호출되게 된다.

``` fsharp
builder {
    if condition then
        return 42
}

if condition then
    builder.Return(42)
else
    builder.Zero()
```


#### MergeSources

``` fsharp
member MergeSources: (M<'T> * M<'U>) -> M<'T * 'U>
```

여러 값을 병렬로 바인딩하게 해준다. 계산식 내에서 `and!` 키워드와 동일하다.

``` fsharp
builder {
    let! x = m1
    and! y = m2
    return x + y
}
```


#### Delay, Run

``` fsharp
member Delay: (unit -> M<'T>) -> Delayed<'T>
```

`Delay`는 계산식을 함수로 감싸 즉시 계산하지 않고 지연시키도록 한다. 리턴값이 `Delayed<'T>`라는 타입인데, 이 조건을 만족하기만 한다면 어떤 타입이든 올 수 있다. 일반적으론 `M<'T>` 혹은 `unit -> M<'T>`가 들어가게 되며, 디폴트 값도 `M<'T>`이다. 계산식 내에서 직접 부를 방법은 없고 문맥에 따라 `return`과 `delay` 중 하나로 변환된다.


``` fsharp
member Run: Delayed<'T> -> M<'T> // or M<'T> -> 'T
```

그렇게 지연된 계산을 실행하게 해주는 함수가 `Run`이다. `Delayed<'T>`를 계산해서 `M<'T>`를 가져오게 하거나, 혹은 `M<'T>`를 계산해서 그 안에 있는 `'T`를 꺼내게 한다.

``` fsharp
let result = 
    builder { 
        return 42
    }

builder.Run(builder.Delay(fun () -> builder.Return(42)))
```

위의 코드에선 `builder` 계산식의 결과를 `result`에 즉시 바인딩하도록 요구하고 있으니 계산식으로 감싸서 주는 것이 아니라 `Run`을 수행하게 된다.


#### Combine

``` fsharp
member Combine: M<'T> * Delayed<'T> -> M<'T> // or M<unit> * M<'T> -> M<'T>
```

두 개의 계산을 이어붙인다. 보통 한 계산식 내에서 큰 연관은 없는 작업을 연달아 수행할 때 `Combine`이 사용된다.

``` fsharp
builder {
    do! operation1  // M<unit>
    do! operation2  // M<unit>
    return 42       // M<int>
}

builder.Combine(
    builder.Bind(operation1, fun() ->
        builder.Bind(operation2, fun() ->
            builder.Return(42))))
```


#### While, For

``` fsharp
member While: (unit -> bool) * Delayed<'T> -> M<'T> // or (unit -> bool) * Delayed<unit> -> M<unit>
member For: seq<'T> * ('T -> M<'U>) -> M<'U> // or seq<'T> * ('T -> M<'U>) -> seq<M<'U>>
```

이름 그대로 계산식 내에서 `while`, `for` 루프로 변환된다. 사용법도 동일하다.

``` fsharp
builder {
    while condition() do
        do! operation
}

builder.While(condition, builder.Delay(fun () -> operation))
```

``` fsharp
builder {
    for x in [1..10] do
        do! operation x
}

builder.For([1..10], fun x -> operation x)
```


#### TryWith, TryFinally

``` fsharp
member TryWith: Delayed<'T> * (exn -> M<'T>) -> M<'T>
member TryFinally: Delayed<'T> * (unit -> unit) -> M<'T>
```

각각 `try ... with`, `try ... finally` 구문에 대응된다. 각각의 함수의 2번째 인자를 잘 보면 `TryWith`는 예외를 받아서 무언가 수행을 해야하고, 또 리턴을 제시해야 하니까 `exn -> M<'T>`로, `TryFinally`는 예외가 있든 없든 상관없이 반드시 수행해야 하고, 또 리턴값을 요구하지 않으므로 `unit -> unit`을 받고 있음을 알 수 있다.

``` fsharp
builder {
    try
        return! operation
    with
    | ex -> return! handleError ex
}

builder.TryWith(
    builder.Delay(fun () -> operation),
    fun ex -> handleError ex
)
```

``` fsharp
builder {
    try
        return! operation
    finally
        cleanup()
}

builder.TryFinally(
    builder.Delay(fun () -> operation),
    fun () -> cleanup()
)
```

그리고 위 두 예제 모두 계산식을 선언만 했지 그 값을 즉시 받아오는 코드가 없으므로 `try` 본문이 `Delay`로 변환되었음을 알 수 있다.


#### Using

``` fsharp
member Using: 'T * ('T -> M<'U>) -> M<'U> when 'T :> IDisposable
```

`use`키워드에 대응되며, C#의 `using`과도 동일하다. 즉 `IDisposable` 객체의 할당 및 자원해제까지 담당해준다.

``` fsharp
builder {
    use file = File.OpenRead("sample.txt")
    return! processFile file
}

builder.Using(
    File.OpenRead("sample.txt"),
    fun file -> processFile file
)
```


#### Yield, YieldFrom

``` fsharp
member Yield: 'T -> M<'T>
member YieldFrom: M<'T> -> M<'T>
```

각각 `yield`, `yield!`에 대응된다.

``` fsharp
seq {
    yield 1
    yield 2
}

seq.Yield(1)
seq.Combine(seq.Yield(1), seq.Yield(2))
```

``` fsharp
seq {
    yield! [1; 2; 3]
    yield 4
}

seq.YieldFrom([1; 2; 3])
seq.Combine(seq.YieldFrom([1; 2; 3]), seq.Yield(4))
```


### Computation Expression != Monad

위에서 어떻게 계산식을 만들 수 있는지 대략 살펴봤다. 그런데 가만 살펴보면 이상한 점이 있다. 위의 모든 함수들의 구현은 전부 선택적 요소라고 했는데, 그러면 모나드의 핵심 요소인 `Return`과 `Bind`를 구현하지 않아도 될까? 

일단 `Return`의 경우에는 반드시 필요해보인다. 계산식이 계산 결과를 내지 못한다면 무슨 의미가 있을까? 

그런데 `Bind`의 경우는 이야기가 좀 다르다. 모나드에서 `Bind`는 반드시 필요하다. `Bind` 함수가 있어야 모나드를 유지하면서 안의 값을 마음대로 변경할 수 있기 때문이다. 즉, `Bind`가 없다면 우리는 모나드 내의 변수들을 다룰 수 없고 그저 그대로 리턴하는 것만 가능해진다.

그런데 F#의 계산식은 모나드가 아니라 계산식이다. 말장난같지만 `Bind` 함수의 구현은 필수가 아니라는 점이 이를 증명한다. 그럼 `Bind`가 없는 계산식은 어떤 의미가 있을까? 아까 앞에서 `seq`와 `query`는 `let!`을 사용할 수 없다고 했었다. 그 이유가 바로 `seq`와 `query`는 `Bind`를 구현하지 않았기 때문이다. `Bind`가 없으니 그 시작점이 될 `let!` 또한 사용 불가능하다. 그런데 `seq`와 `query`는 안의 값을 조작하려는 용도가 아니라, 값을 끝없이 생성해내기 위한 용도로 사용된다. 즉, 이전 값에 의존해서 순차적인 계산을 할 필요가 없는 녀석들이라 얘네들한테는 자기 자신이 모나드인지 아닌지가 그리 중요하지 않다. 어차피 안쓸거니까! 그래서 구현도 하지 않은 것이다.

이쯤에서 알 수 있는 사실은, F#의 계산식은 모나드가 아니다. 그런데 모나드는 F#의 계산식으로 구현할 수 있다. 즉, 계산식이 범위가 좀 더 넓은 형태인 것이다.
