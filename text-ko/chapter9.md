# 캔버스 그래픽

## 이 장의 목표

이 장에서는 `purescript-canvas` 패키지에 집중해보자. PureScript에서 HTML5 캔버스 API를 사용하여 2D 그래픽을 생성하는 방법을 살펴볼 것이다.

## Project Setup

이 장의 예제 프로젝트에는 다음의 Bower 의존성이 추가되었다.

- `purescript-canvas`: HTML5 캔버스 API를 위한 타입이 정의되어 있다.
- `purescript-refs`: **전역 가변 참조**를 사용하는 부수 효과를 제공한다.

이 장의 소스 코드는 몇 개의 모듈로 나누어져 있다. 각 모듈에는 따로 `main` 함수가 있다. 각 절마다 다른 파일에 관련 내용이 구현되어 있고 Pulp 빌드 명령을 실행할 때 원하는 `main`을 실행하기 위해 적절히 `Main` 모듈을 지정해주어야 한다.

`html/index.html` 파일에는 `canvas` 엘리먼트가 있어서 각 예제가 이를 이용한다. `script` 엘리먼트는 컴파일된 PureScript 코드를 로드한다. 각 절의 내용을 확인해보려면 HTML 파일을 브라우저에서 열어보면 된다.

## 간단한 도형

`Example/Rectangle.purs` 파일에 포함된 예제는 소개에 적당한 간단한 예제로서 캔버스 가운데 파랑 사각형 하나를 그린다. 이 모듈은 `Control.Monad.Eff` 모듈과 `Graphics.Canvas` 모듈을 임포트한다. `Graphics.Canvas` 모듈에는 캔버스 API를 사용하는데 필요한 `Eff` 모나드 액션이 정의되어 있다.

`main` 액션이 시작되면 `getCanvasElementById` 액션을 통해 캔버스 객체의 참조를 얻어낸다. `getContext2D` 액션은 캔버스로부터 2D 렌더링 컨텍스트를 접근하는 데 사용된다.

```haskell
main = void $ unsafePartial do
  Just canvas <- getCanvasElementById "canvas"
  ctx <- getContext2D canvas
```

첫 줄에서 `unsafePartial` 함수를 호출하는 이유는 `getCanvasElementById` 함수가 부분 함수인데 그 결과를 `Just` 생성자로 매칭하고자 하기 때문이다. 예제 수준에서는 괜찮지만 실제로 사용할 때엔 `Nothing`에 대해서도 적절히 오류 메시지를 보여주도록 처리해야 할 것이다.

PSCi에서 위 액션들의 타입을 살펴보거나 혹은 문서를 참조하기 바란다.

```haskell
getCanvasElementById :: forall eff. String -> Eff (canvas :: CANVAS | eff) (Maybe CanvasElement)

getContext2D :: forall eff. CanvasElement -> Eff (canvas :: CANVAS | eff) Context2D
```

`CanvasElement`와 `Context2D`는 모두 `Graphics.Canvas` 모듈에 정의된 타입이다. `Canvas` 효과 역시 같은 모듈에 정의되어 있고, 이 모듈의 모든 액션에서 `Canvas` 효과가 사용된다.

그래픽 컨텍스트 `ctx`는 캔버스 상태를 관리하기 위한 객체이며, 기본 도형들을 그리고 스타일이나 색상을 지정하고 변형을 적용하는 메쏘드를 제공한다.

파랑색으로 채우기 스타일을 지정하기 위해 `setFillStyle` 액션을 호출한다.

```haskell
  setFillStyle "#0000FF" ctx
```

`setFillStyle` 액션은 그래픽 컨텍스트를 인자로 넘겨줘야 한다. 이 방식은 `Graphics.Canvas` 모듈의 다른 액션들에서도 쉽게 볼 수 있는 패턴이다.

마지막으로 `fillPath` 액션을 이용하여 사각형을 채운다. `fillPath` 액션의 타입은 다음과 같다.

```haskell
fillPath :: forall eff a. Context2D ->
                          Eff (canvas :: CANVAS | eff) a ->
                          Eff (canvas :: CANVAS | eff) a
```

첫 번째 인자가 그래픽 컨텍스트다. 두 번째 인자는 경로(path)를 그리기 위한 또다른 액션이다. 경로를 그리기 위해 여기서는 `rect` 액션을 호출한다. `rect` 액션도 그래픽 컨텍스트를 인자로 넘겨줘야 하며 추가로 사각형의 위치와 크기 정보를 제공해야 한다.

```haskell
  fillPath ctx $ rect ctx
    { x: 250.0
    , y: 250.0
    , w: 100.0
    , h: 100.0
    }
```

메인 모듈의 이름인 `Example.Rectangle`을 지정하여 사각형 예제를 빌드한다.

```text
$ mkdir dist/
$ pulp build -O --main Example.Rectangle --to dist/Main.js
```

`html/index.html` 파일을 열어서 캔버스 가운데 파랑 사각형이 그려지는지 확인해보자.

## 행 다형성 사용하기

경로를 그리는 방법은 많다. `arc` 함수는 원호를 그리고, `moveTo`, `lineTo`, `closePath` 함수는 선분들로 연결된 경로를 그리는 데 사용할 수 있다.

`Shape.purs` 파일은 직사각형, 호, 삼각형 이렇게 세 개의 도형을 그린다.

이미 본 것처럼 `rect` 함수는 인자를 받을 때 레코드 형식으로 받았다. 사실 직사각형 속성에 대한 타입 별칭이 이미 정의되어 있다.

```haskell
type Rectangle =
  { x :: Number
  , y :: Number
  , w :: Number
  , h :: Number
  }
```

`x`와 `y` 속성은 top-left 코너의 위치를 나타내고, `w`와 `h` 속성은 각각 너비와 높이를 나타낸다.

원호를 그리려면 `arc` 함수를 이용한다. 다음 타입의 레코드를 전달하면 된다.

```haskell
type Arc =
  { x     :: Number
  , y     :: Number
  , r     :: Number
  , start :: Number
  , end   :: Number
  }
```

`x`와 `y` 속성은 원호의 중심을, `r` 속성은 반지름을 나타낸다. 그리고 `start`와 `end` 속성은 원호가 시작되는 지점과 끝나는 지점을 라디안 값으로 나타낸다.

예를 들어 아래 코드는 `(300, 300)` 지점을 중심으로 반지름이 `50`인 원호의 조각을 채운다.

```haskell
  fillPath ctx $ arc ctx
    { x      : 300.0
    , y      : 300.0
    , r      : 50.0
    , start  : Math.pi * 5.0 / 8.0
    , end    : Math.pi * 2.0
    }
```

`Rectangle`과 `Arc` 레코드 타입에는 모두 `x`와 `y` 속성이 모두 `Number` 타입으로 선언되어 있다. 두 경우 모두 캔버스 상위 한 지점을 나타낸다. 따라서 두 레코드 타입에 대해 행 다형성으로 동작하는 함수를 작성할 수 있다.

예를 들어 `Shapes` 모듈에 정의된 `translate` 함수는 도형의 `x`와 `y` 속성을 수정함으로써 수평이동 시키는 함수다.

```haskell
translate
  :: forall r
   . Number
  -> Number
  -> { x :: Number, y :: Number | r }
  -> { x :: Number, y :: Number | r }
translate dx dy shape = shape
  { x = shape.x + dx
  , y = shape.y + dy
  }
```

이 함수의 타입이 행 다형성을 보여주고 있다. `translate` 함수는 `x`와 `y` 속성이 있는 어떤 레코드 타입이라도 인자로 받을 수 있으며, 반환값은 입력 인자의 타입과 같다. 여기서 `x`와 `y` 속성만 갱신되고 다른 속성들은 수정되지 않고 그대로 반환된다.

여기 사용된 문법은 **레코드 갱신 문법**이다. `shape { ... }` 표현식은 `shape` 레코드를 기본으로 하여 중괄호에 선언된 필드들만 지정된 값으로 갱신하여 새로운 레코드를 만든다. 중괄호 내부의 각 필드는 이름과 값이 등호로 구분된다. (레코드 생성 문법에서는 콜론이 사용된다.)

`translate` 함수는 `Rectangle` 타입이나 `Arc` 타입의 레코드에 모두 사용할 수 있다.

세 번째로 살펴볼 경로 예제는 선분들로 만들어지는 경로다. 아래 코드를 보자.

```haskell
  setFillStyle "#FF0000" ctx

  fillPath ctx $ do
    moveTo ctx 300.0 260.0
    lineTo ctx 260.0 340.0
    lineTo ctx 340.0 340.0
    closePath ctx
```

경로를 그리는 데 사용된 함수들은 `moveTo`와 `lineTo`, `closePath`다.

- `moveTo`: 지정된 좌표로 경로의 현재 위치를 옮긴다.
- `lineTo`: 현재 위치에서 지정된 위치까지 선분을 그리고, 현재 위치를 새 지점으로 갱신한다.
- `closePath`: 현재 위치와 시작 위치를 연결하는 선분을 그리면서 경로 그리기를 완료한다.

위 코드는 채워진 이등변삼각형을 그린다.

메인 모듈을 `Example.Shapes`로 지정하여 예제를 빌드한다.

```text
$ pulp build -O --main Example.Shapes --to dist/Main.js
```

그리고 `html/index.html` 파일을 다시 열어서 결과를 확인하자. 캔버스에 세 가지 다른 도형이 그려지는지 확인해보자.

> ## 연습 문제
>
> 1. (쉬움) `strokePath`와 `setStrokeStyle` 함수를 위의 예제들에 적용해보자.
> 1. (쉬움) 복잡한 경로를 그리기 위해서는 do 블록에서 `fillPath`와 `strokePath` 함수를 함께 사용하면 된다. `Rectangle` 예제를 수정하여 `fillPath` 한 번에 사각형 두 개가 나란히 나오도록 바꿔보자. 또, 원호 그리기와 선분 잇기를 이용하여 부채꼴을 그려보자.
> 1. (보통) 다음처럼 2차원 공간의 한 지점을 나타내는 레코드 타입이 있다.
>
>     ```haskell
>     type Point = { x :: Number, y :: Number }
>     ```
>
>     아래 타입에 따라 `renderPath` 함수를 정의해보자. 이 함수는 주어진 좌표 배열로 닫힌 경로를 그린다.
>
>     ```haskell
>     renderPath
>       :: forall eff
>        . Context2D
>       -> Array Point
>       -> Eff (canvas :: Canvas | eff) Unit
>     ```
>
>     `f` 함수가 `0`에서 `1` 사이의 값을 받아서 `Point`를 반환한다고 할 때, 주어진 `f`에 대해 `renderPath` 함수를 이용하여 그래프를 그리는 액션 함수를 작성해보라. 그래프를 구성하는 좌표는 `f` 함수에 대해 적정 수준의 샘플링으로 구할 수 있다.
>
>     ```haskell
>     f :: Number -> Point
>     ```
>
>     `f`를 바꿔가며 여러가지 경로를 그려보자.

## 무작위로 원 그리기

`Example/Random.purs` 파일에 포함된 예제는 `Eff` 모나드를 이용하여 난수 생성과 캔버스 조작이라는 두 가지 다른 부수 효과를 합친다. 이 예제는 무작위로 100개의 원을 캔버스에 그린다.

`main`  액션은 그래픽 컨텍스트를 가져온 다음 스트로크와 채우기 스타일을 지정한다.

```haskell
  setFillStyle "#FF0000" ctx
  setStrokeStyle "#000000" ctx
```

`for_` 함수를 이용하여 `0`부터 `100`까지의 정수로 루프를 만든다.

```haskell
  for_ (1 .. 100) \_ -> do
```

루프의 몸체가 되는 do 블록에서는 `0`과 `1` 사이의 난수를 세 개 선택한다. 이 값은 중심의 좌표 `x`, `y`와 반지름 `r`이 된다.

```haskell
    x <- random
    y <- random
    r <- random
```

원 하나를 그리기 위해 위의 세 값으로 `Arc`를 만들고 마지막으로 경로를 채우고 스트로크를 그린다.

```haskell
    let path = arc ctx
         { x     : x * 600.0
         , y     : y * 600.0
         , r     : r * 50.0
         , start : 0.0
         , end   : Math.pi * 2.0
         }
    fillPath ctx path
    strokePath ctx path
```

`Example.Random` 모듈을 지정하여 이 예제를 빌드할 수 있다.

```text
$ pulp build -O --main Example.Random --to dist/Main.js
```

`html/index.html` 파일을 브라우저로 열어서 동작을 확인해보자.

## 변형

캔버스는 간단한 도형을 그리는 것보다 더 많은 일을 할 수 있다. 캔버스는 그리기 전에 도형을 변형할 수도 있다. 수평 이동, 회전, 확대, 기울임과 같은 변형이 가능하다.

`purescript-canvas` 라이브러리는 이러한 변형을 다음 함수로 지원한다.

```haskell
translate :: forall eff
           . TranslateTransform
          -> Context2D
          -> Eff (canvas :: Canvas | eff) Context2D

rotate    :: forall eff
           . Number
          -> Context2D
          -> Eff (canvas :: Canvas | eff) Context2D

scale     :: forall eff
           . ScaleTransform
          -> Context2D
          -> Eff (canvas :: CANVAS | eff) Context2D

transform :: forall eff
           . Transform
          -> Context2D
          -> Eff (canvas :: CANVAS | eff) Context2D
```

`translate` 액션은 `TranslateTransform` 레코드로 지정된 만큼 좌표를 이동시킨다.

`rotate` 액션은 원점을 기준으로 첫 번째 인자가 지정하는 만큼 회전시킨다. 회전 각도는 라디안 단위를 사용한다.

`scale` 액션은 원점을 기준으로 확대 혹은 축소시킨다. `ScaleTransform` 레코드로 `x` 축과 `y`축의 확대 수준을 다르게 지정할 수 있다.

`transform`은 다른 액션들보다 더 일반화된 액션이다. 행렬로 정의된 아핀 변환을 적용시킨다.

이 변형 액션들을 호출한 다음에 그려지는 모든 도형은 해당 변형이 적용되어 그려진다.

사실 각 액션 함수들이 하는 일은 컨텍스트의 현재 변형에 새로운 변형을 곱하기로 누적시키는 것이다. 여러 변형을 순차적으로 적용시키면 실제로는 역순으로 각 효과가 반영된다.

```haskell
transformations ctx = do
  translate { translateX: 10.0, translateY: 10.0 } ctx
  scale { scaleX: 2.0, scaleY: 2.0 } ctx
  rotate (Math.pi / 2.0) ctx

  renderScene
```

위와 같이 세 가지 액션이 순차적으로 적용되면 실제로 해당 그림은 회전, 확대, 이동 순서로 적용된다.

## 컨텍스트 유지하기

한 장면의 일부를 그리면서 변형을 적용하는 경우에는 일부를 그린 다음 변형을 원래대로 되돌리는 것이 일반적이다.

캔버스 API가 제공하는 `save`와 `restore` 메쏘드는 캔버스의 상태를 스택처럼 관리할 수 있게 해준다. `purescript-canvas`는 다음 함수를 이용하여 이 기능을 사용할 수 있게 한다.

```haskell
save
  :: forall eff
   . Context2D
  -> Eff (canvas :: CANVAS | eff) Context2D

restore
  :: forall eff
   . Context2D
  -> Eff (canvas :: CANVAS | eff) Context2D
```

`save` 액션은 컨텍스트의 현재 상태를 스택에 추가한다. 상태에는 현재 스타일이나 변형도 포함된다. `restore` 액션은 상태 스택에서 최신 상태를 꺼내어 복원한다.

이 두 함수를 이용하면 현재 상태를 저장한 다음 몇가지 스타일과 변형을 적용하여 그림을 그리고 마지막에 다시 원래의 변형과 상태로 되돌릴 수 있다. 예를 들어 다음 함수는 인자로 받은 캔버스 액션을 실행하기 전에 회전을 먼저 적용하고, 액션을 실행한 다음 변형을 원래대로 되돌린다.

```haskell
rotated ctx render = do
  save ctx
  rotate Math.pi ctx
  render
  restore ctx
```

이처럼 고차함수를 이용하여 추상화시킨 예로는 `withContext` 함수가 있다. 이 액션 함수는 다른 캔버스 액션을 실행하면서 원래의 컨텍스트 상태를 유지시킨다.

```haskell
withContext
  :: forall eff a
   . Context2D
  -> Eff (canvas :: CANVAS | eff) a
  -> Eff (canvas :: CANVAS | eff) a
```

`withContext` 함수를 이용하여 `rotated` 함수를 구현할 수도 있다.

```haskell
rotated ctx render =
  withContext ctx do
    rotate Math.pi ctx
    render
```

## 전역 가변 상태

이 절에서 다룰 내용은 `purescript-refs` 패키지가 `Eff` 모나드를 이용하여 제공하는 새로운 효과다.

`Control.Monad.Eff.Ref` 모듈에는 전역 가변 참조를 위한 `Ref`라는 타입 생성자와 이와 연관된 `REF` 효과가 정의되어 있다.

```text
> import Control.Monad.Eff.Ref

> :kind Ref
Type -> Type

> :kind REF
Control.Monad.Eff.Effect
```

`Ref a` 타입의 값은 `a` 타입의 값을 담을 수 있는 가변 참조 셀을 나타낸다. 앞 장에서 다루었던 `STRef h a`와 매우 비슷하다. `runST` 함수를 이용하여 `ST` 효과를 없앨 수 있었던 것과 달리 `Ref` 효과는 따로 핸들러를 제공하지 않는다. `ST`는 안전한 지역적 가변 값을 추적하는 데 사용할 수 있지만 `Ref`는 전역 가변 값을 추적하는데 사용한다. 그렇기 때문에 자주 사용해서는 안된다.

`Example/Refs.purs` 파일에 포함된 예제는 `REF` 효과를 이용하여 `canvas` 엘리먼트에서 발생하는 마우스 클릭 이벤트를 추적한다.

이 코드는 먼저 `newRef` 액션을 이용하여 `0` 값을 가지는 새로운 참조를 만든다.

```haskell
  clickCount <- newRef 0
```

클릭 이벤트 핸들러에서는 `modifyRef` 액션을 사용하여 클릭 횟수를 갱신한다.

```haskell
    modifyRef clickCount (\count -> count + 1)
```

`readRef` 액션은 클릭 횟수를 읽을 때 사용한다.

```haskell
    count <- readRef clickCount
```

`render` 함수에서는 클릭 횟수에 따라 다른 변형을 적용하여 사각형을 그린다.

```haskell
    withContext ctx do
      let scaleX = Math.sin (toNumber count * Math.pi / 4.0) + 1.5
      let scaleY = Math.sin (toNumber count * Math.pi / 6.0) + 1.5

      translate { translateX: 300.0, translateY:  300.0 } ctx
      rotate (toNumber count * Math.pi / 18.0) ctx
      scale { scaleX: scaleX, scaleY: scaleY } ctx
      translate { translateX: -100.0, translateY: -100.0 } ctx

      fillPath ctx $ rect ctx
        { x: 0.0
        , y: 0.0
        , w: 200.0
        , h: 200.0
        }
```

이 액션은 `withContext` 함수를 이용하여 원래의 변형을 유지한 채로 몇 가지 변형을 순차적으로 적용한다. (변환이 적용되는 순서는 반대라는 점을 잊지말자.)

- 사각형은 `(-100, -100)`만큼 이동하여 사각형 중심이 원점이 된다.
- 사각형은 원점을 기준으로 확대된다.
- 사각형은 클릭 횟수 * 10도 만큼 회전된다.
- 사각형은 `(300, 300)`만큼 이동하여 사각형 중심이 캔버스 중심이 된다.

예제를 다음 명령으로 빌드한 다음 `html/index.html` 파일을 열어보자.

```text
$ pulp build -O --main Example.Refs --to dist/Main.js
```

캔버스를 여러번 클릭하면 녹색 사각형이 캔버스 중심을 기준으로 회전하는 모습을 볼 수 있다.

> ## 연습 문제
>
> 1. (쉬움) Write a higher-order function which strokes and fills a path simultaneously. Rewrite the `Random.purs` example using your function.
> 1. (보통) Use the `RANDOM` and `DOM` effects to create an application which renders a circle with random position, color and radius to the canvas when the mouse is clicked.
> 1. (보통) Write a function which transforms the scene by rotating it around a point with specified coordinates. _Hint_: use a translation to first translate the scene to the origin.

## L-Systems

In this final example, we will use the `purescript-canvas` package to write a function for rendering _L-systems_ (or _Lindenmayer systems_).

An L-system is defined by an _alphabet_, an initial sequence of letters from the alphabet, and a set of _production rules_. Each production rule takes a letter of the alphabet and returns a sequence of replacement letters. This process is iterated some number of times starting with the initial sequence of letters.

If each letter of the alphabet is associated with some instruction to perform on the canvas, the L-system can be rendered by following the instructions in order.

For example, suppose the alphabet consists of the letters `L` (turn left), `R` (turn right) and `F` (move forward). We might define the following production rules:

```text
L -> L
R -> R
F -> FLFRRFLF
```

If we start with the initial sequence "FRRFRRFRR" and iterate, we obtain the following sequence:

```text
FRRFRRFRR
FLFRRFLFRRFLFRRFLFRRFLFRRFLFRR
FLFRRFLFLFLFRRFLFRRFLFRRFLFLFLFRRFLFRRFLFRRFLF...
```

and so on. Plotting a piecewise-linear path corresponding to this set of instruction approximates a curve called the _Koch curve_. Increasing the number of iterations increases the resolution of the curve.

Let's translate this into the language of types and functions.

We can represent our choice of alphabet by a choice of type. For our example, we can choose the following type:

```haskell
data Alphabet = L | R | F
```

This data type defines one data constructor for each letter in our alphabet.

How can we represent the initial sequence of letters? Well, that's just an array of letters from our alphabet, which we will call a `Sentence`:

```haskell
type Sentence = Array Alphabet

initial :: Sentence
initial = [F, R, R, F, R, R, F, R, R]
```

Our production rules can be represented as a function from `Alphabet` to `Sentence` as follows:

```haskell
productions :: Alphabet -> Sentence
productions L = [L]
productions R = [R]
productions F = [F, L, F, R, R, F, L, F]
```

This is just copied straight from the specification above.

Now we can implement a function `lsystem` which will take a specification in this form, and render it to the canvas. What type should `lsystem` have? Well, it needs to take values like `initial` and `productions` as arguments, as well as a function which can render a letter of the alphabet to the canvas.

Here is a first approximation to the type of `lsystem`:

```haskell
forall eff. Sentence
         -> (Alphabet -> Sentence)
         -> (Alphabet -> Eff (canvas :: Canvas | eff) Unit)
         -> Int
         -> Eff (canvas :: CANVAS | eff) Unit
```

The first two argument types correspond to the values `initial` and `productions`.

The third argument represents a function which takes a letter of the alphabet and _interprets_ it by performing some actions on the canvas. In our example, this would mean turning left in the case of the letter `L`, turning right in the case of the letter `R`, and moving forward in the case of a letter `F`.

The final argument is a number representing the number of iterations of the production rules we would like to perform.

The first observation is that the `lsystem` function should work for only one type of `Alphabet`, but for any type, so we should generalize our type accordingly. Let's replace `Alphabet` and `Sentence` with `a` and `Array a` for some quantified type variable `a`:

```haskell
forall a eff. Array a
           -> (a -> Array a)
           -> (a -> Eff (canvas :: CANVAS | eff) Unit)
           -> Int
           -> Eff (canvas :: CANVAS | eff) Unit
```

The second observation is that, in order to implement instructions like "turn left" and "turn right", we will need to maintain some state, namely the direction in which the path is moving at any time. We need to modify our function to pass the state through the computation. Again, the `lsystem` function should work for any type of state, so we will represent it using the type variable `s`.

We need to add the type `s` in three places:

```haskell
forall a s eff. Array a
             -> (a -> Array a)
             -> (s -> a -> Eff (canvas :: CANVAS | eff) s)
             -> Int
             -> s
             -> Eff (canvas :: CANVAS | eff) s
```

Firstly, the type `s` was added as the type of an additional argument to `lsystem`. This argument will represent the initial state of the L-system.

The type `s` also appears as an argument to, and as the return type of the interpretation function (the third argument to `lsystem`). The interpretation function will now receive the current state of the L-system as an argument, and will return a new, updated state as its return value.

In the case of our example, we can define use following type to represent the state:

```haskell
type State =
  { x :: Number
  , y :: Number
  , theta :: Number
  }
```

The properties `x` and `y` represent the current position of the path, and the `theta` property represents the current direction of the path, specified as the angle between the path direction and the horizontal axis, in radians.

The initial state of the system might be specified as follows:

```haskell
initialState :: State
initialState = { x: 120.0, y: 200.0, theta: 0.0 }
```

Now let's try to implement the `lsystem` function. We will find that its definition is remarkably simple.

It seems reasonable that `lsystem` should recurse on its fourth argument (of type `Int`). On each step of the recursion, the current sentence will change, having been updated by using the production rules. With that in mind, let's begin by introducing names for the function arguments, and delegating to a helper function:

```haskell
lsystem :: forall a s eff
         . Array a
        -> (a -> Array a)
        -> (s -> a -> Eff (canvas :: CANVAS | eff) s)
        -> Int
        -> s
        -> Eff (canvas :: CANVAS | eff) s
lsystem init prod interpret n state = go init n
  where
```

The `go` function works by recursion on its second argument. There are two cases: when `n` is zero, and when `n` is non-zero.

In the first case, the recursion is complete, and we simply need to interpret the current sentence according to the interpretation function. We have a sentence of type `Array a`, a state of type `s`, and a function of type `s -> a -> Eff (canvas :: CANVAS | eff) s`. This sounds like a job for the `foldM` function which we defined earlier, and which is available from the `purescript-control` package:

```haskell
  go s 0 = foldM interpret state s
```

What about in the non-zero case? In that case, we can simply apply the production rules to each letter of the current sentence, concatenate the results, and repeat by calling `go` recursively:

```haskell
  go s n = go (concatMap prod s) (n - 1)
```

That's it! Note how the use of higher order functions like `foldM` and `concatMap` allowed us to communicate our ideas concisely.

However, we're not quite done. The type we have given is actually still too specific. Note that we don't use any canvas operations anywhere in our implementation. Nor do we make use of the structure of the `Eff` monad at all. In fact, our function works for _any_ monad `m`!

Here is the more general type of `lsystem`, as specified in the accompanying source code for this chapter:

```haskell
lsystem :: forall a m s
         . Monad m
        => Array a
        -> (a -> Array a)
        -> (s -> a -> m s)
        -> Int
        -> s
        -> m s
```

We can understand this type as saying that our interpretation function is free to have any side-effects at all, captured by the monad `m`. It might render to the canvas, or print information to the console, or support failure or multiple return values. The reader is encouraged to try writing L-systems which use these various types of side-effect.

This function is a good example of the power of separating data from implementation. The advantage of this approach is that we gain the freedom to interpret our data in multiple different ways. We might even factor `lsystem` into two smaller functions: the first would build the sentence using repeated application of `concatMap`, and the second would interpret the sentence using `foldM`. This is also left as an exercise for the reader.

Let's complete our example by implementing its interpretation function. The type of `lsystem` tells us that its type signature must be `s -> a -> m s` for some types `a` and `s` and a type constructor `m`. We know that we want `a` to be `Alphabet` and `s` to be `State`, and for the monad `m` we can choose `Eff (canvas :: CANVAS)`. This gives us the following type:

```haskell
interpret :: State -> Alphabet -> Eff (canvas :: CANVAS) State
```

To implement this function, we need to handle the three data constructors of the `Alphabet` type. To interpret the letters `L` (move left) and `R` (move right), we simply have to update the state to change the angle `theta` appropriately:

```haskell
interpret state L = pure $ state { theta = state.theta - Math.pi / 3 }
interpret state R = pure $ state { theta = state.theta + Math.pi / 3 }
```

To interpret the letter `F` (move forward), we can calculate the new position of the path, render a line segment, and update the state, as follows:

```haskell
interpret state F = do
  let x = state.x + Math.cos state.theta * 1.5
      y = state.y + Math.sin state.theta * 1.5
  moveTo ctx state.x state.y
  lineTo ctx x y
  pure { x, y, theta: state.theta }
```

Note that in the source code for this chapter, the `interpret` function is defined using a `let` binding inside the `main` function, so that the name `ctx` is in scope. It would also be possible to move the context into the `State` type, but this would be inappropriate because it is not a changing part of the state of the system.

To render this L-system, we can simply use the `strokePath` action:

```haskell
strokePath ctx $ lsystem initial productions interpret 5 initialState
```

Compile the L-system example using

```text
$ pulp build -O --main Example.LSystem --to dist/Main.js
```

and open `html/index.html`. You should see the Koch curve rendered to the canvas.

> ## Exercises
>
> 1. (쉬움) Modify the L-system example above to use `fillPath` instead of `strokePath`. _Hint_: you will need to include a call to `closePath`, and move the call to `moveTo` outside of the `interpret` function.
> 1. (쉬움) Try changing the various numerical constants in the code, to understand their effect on the rendered system.
> 1. (보통) Break the `lsystem` function into two smaller functions. The first should build the final sentence using repeated application of `concatMap`, and the second should use `foldM` to interpret the result.
> 1. (보통) Add a drop shadow to the filled shape, by using the `setShadowOffsetX`, `setShadowOffsetY`, `setShadowBlur` and `setShadowColor` actions. _Hint_: use PSCi to find the types of these functions.
> 1. (보통) The angle of the corners is currently a constant (`pi/3`). Instead, it can be moved into the `Alphabet` data type, which allows it to be changed by the production rules:
>
>     ```haskell
>     type Angle = Number
>
>     data Alphabet = L Angle | R Angle | F
>     ```
>
>     How can this new information be used in the production rules to create interesting shapes?
> 1. (어려움) An L-system is given by an alphabet with four letters: `L` (turn left through 60 degrees), `R` (turn right through 60 degrees), `F` (move forward) and `M` (also move forward).
>
>     The initial sentence of the system is the single letter `M`.
>
>     The production rules are specified as follows:
>
>     ```text
>     L -> L
>     R -> R
>     F -> FLMLFRMRFRMRFLMLF
>     M -> MRFRMLFLMLFLMRFRM
>     ```
>
>     Render this L-system. _Note_: you will need to decrease the number of iterations of the production rules, since the size of the final sentence grows exponentially with the number of iterations.
>
>     Now, notice the symmetry between `L` and `M` in the production rules. The two "move forward" instructions can be differentiated using a `Boolean` value using the following alphabet type:
>
>     ```haskell
>     data Alphabet = L | R | F Boolean
>     ```
>
>     Implement this L-system again using this representation of the alphabet.
> 1. (어려움) Use a different monad `m` in the interpretation function. You might try using the `CONSOLE` effect to write the L-system onto the console, or using the `RANDOM` effect to apply random "mutations" to the state type.

## Conclusion

In this chapter, we learned how to use the HTML5 Canvas API from PureScript by using the `purescript-canvas` library. We also saw a practical demonstration of many of the techniques we have learned already: maps and folds, records and row polymorphism, and the `Eff` monad for handling side-effects.

The examples also demonstrated the power of higher-order functions and _separating data from implementation_. It would be possible to extend these ideas to completely separate the representation of a scene from its rendering function, using an algebraic data type, for example:

```haskell
data Scene
  = Rect Rectangle
  | Arc Arc
  | PiecewiseLinear (Array Point)
  | Transformed Transform Scene
  | Clipped Rectangle Scene
  | ...
```

This approach is taken in the `purescript-drawing` package, and it brings the flexibility of being able to manipulate the scene as data in various ways before rendering.

In the next chapter, we will see how to implement libraries like `purescript-canvas` which wrap existing JavaScript functionality, by using PureScript's _foreign function interface_.
