# Eff 모나드

## 이 장의 목표

지난 장에서 살펴 본 Applicative 펑터는 **부수 효과**를 다룰 수 있는 추상화 수단이었다. 누락 가능성이 있는 경우나, 오류 메시지, 혹은 검사 등. 이번 장에서는 부수 효과를 좀더 풍부하게 표현할 수 있는 새로운 추상화인 **모나드**를 소개한다.

이 장의 목표는 모나드란 추상화의 유용성과 **do 표기법**을 설명한다. 주소록 예제를 확장하여 브라우저에 UI 를 보여주고 여기서 발생하는 부수 효과를 처리하기 위한 모나드 사용법도 다룬다. 우리가 살펴 볼 모나드는 PureScript 에서 특히 중요한 `Eff` 모나드이다. `Eff` 모나드는 **네이티브** 효과를 추상화하여 감추어준다.

## 프로젝트 설정

이 장에서 사용할 프로젝트는 앞 장의 코드를 포함한다. 이 장의 프로제트 `src` 디렉토리에는 앞 장의 프로젝트 모듈들이 그대로 들어있다.

이번 프로젝트에는 새로운 Bower 의존성이 추가되었다.

* `purescript-eff`: 이 장에서 주요 주제인 `Eff` 모나드가 정의되어 있다.
* `purescript-react`: React UI 라이브러리를 사용하기 위한 바인딩이다. 주소록 애플리케이션의 UI 를 만드는 데 사용된다.

이번 프로젝트에는 `Main` 모듈이 추가되었다. 이 모듈은 애플리케이션의 시작점과 UI 를 그려주는 함수들을 포함한다.

이 프로젝트를 컴파일하려면 먼저 `npm install` 명령으로 React 를 설치해야 한다. `pulp browserify --to dist/Main.js` 명령으로 빌드하여 JavaScript 코드를 번들링한다. `html/index.html` 파일을 웹브라우저로 열면 이 프로젝트를 실행할 수 있다.

## 모나드와 do 표기법

do 표기법은 **배열 이해** 문법을 다루면서 소개했었다. 배열 이해 문법은 `Data.Array` 모듈의 `concatMap` 함수를 사용하기 위한 단축 문법이다.

다음의 예를 살펴보자. 주사위를 두 번 던져서 나온 숫자의 합이 `n`이 되는 경우를 모두 보려고 한다. 이러한 비결정적 알고리즘은 아마 다음처럼 정리할 수 있을 것이다.

* 처음 주사위를 던져서 나올 수 있는 값 하나를 선택하여 `x`라고 한다.
* 다시 주사위를 던져서 나올 수 있는 값 하나를 선택하여 `y`라고 한다.
* `x`와 `y` 합이 `n`이면 두 값을 `[x, y]` 쌍으로 반환하고, 그렇지 않으면 실패한다.

배열 이해 문법을 이용하면 위와 같은 비결정적 알고리즘을 매끄럽게 표현할 수 있다.

```haskell
import Prelude

import Control.Plus (empty)
import Data.Array ((..))

countThrows :: Int -> Array (Array Int)
countThrows n = do
  x <- 1 .. 6
  y <- 1 .. 6
  if x + y == n
    then pure [x, y]
    else empty
```

PSCi 에서 이 함수를 테스트해보자.

```text
> countThrows 10
[[4,6],[5,5],[6,4]]

> countThrows 12
[[6,6]]
```

앞 장에서는 `Maybe` Applicative 펑터에 대해 직관적인 설명을 하면서 PureScript 함수를 **누락 가능성이 있는 값**에 대해서도 적용할 수 있게 함으로서 프로그래밍 언어가 확장된 것으로 보았다. 이번에도 같은 방식으로 설명할 수 있다. **배열 모나드**를 통해 PureScript 함수가 **비결정적 선택**을 지원할 수 있게 되면서 프로그래밍 언어가 더 확장된다고 볼 수 있다.

이를 일반화해보면, 어떤 타입 생성자 `m`이 **모나드**라면 `m a` 타입의 값에 대해 do 표기법을 사용할 수 있다. 위의 배열을 사용한 예를 보면 각 줄이 하나같이 `Array a` 타입의 계산식을 포함하고 있다. 일반화하자면 do 표기법을 사용한 블록의 모든 줄들은 모나드 `m`과 어떤 타입 `a`로 만들어지는 `m a` 타입을 계산식에 포함한다. 이 때 `m`은 모든 줄에서 같아야 하며(즉, 블록 내에서 부수 효과가 하나로 고정된다.) `a` 타입은 달라질 수 있다.(각 줄의 계산식은 다른 타입의 결과를 내놓을 수 있다.)

do 표기법을 사용하는 다른 예를 살펴보자. 이번에 사용할 타입 생성자는 `Maybe`다. XML 노드를 나타내는 `XML`이란 타입이 있다고 할 때 다음과 같은 함수를 만들 수 있다.

```haskell
child :: XML -> String -> Maybe XML
```

이 함수는 어떤 노드의 자식들 중에서 특정 이름의 엘리먼트를 찾는 함수다. 해당 자식 엘리먼트가 없다면 `Nothing`을 반환한다.

여러 단계로 중첩된 엘리먼트를 찾고자 할 때 do 표기법을 이용할 수 있다. 예를 들어 XML 문서로 된 사용자 프로파일에서 주소 필드 중 도시를 찾으려고 한다.

```haskell
userCity :: XML -> Maybe XML
userCity root = do
  prof <- child root "profile"
  addr <- child prof "address"
  city <- child addr "city"
  pure city
```

`userCity` 함수는 `profile` 자식 엘리먼트를 찾고 그 아래 `address` 엘리먼트를 찾고 그 아래 `city` 엘리먼트를 찾는다. 이 중 어느 하나라도 없다면 `Nothing`을 반환한다. 제대로 찾는다면 `city` 엘리먼트를 `Just`로 감싸서 반환한다.

마지막 줄의 `pure` 함수는 `Applicative` 펑터마다 정의된 것이다. `Maybe` 역시 Applicative 펑터이면서 `pure`가 `Just`로 정의되어 있기 때문에 마지막 줄을 `Just city`라고 바꾸어도 결과는 같다.

## Monad 타입 클래스

`Monad` 타입 클래스는 다음처럼 정의되어 있다.

```haskell
class Apply m <= Bind m where
  bind :: forall a b. m a -> (a -> m b) -> m b

class (Applicative m, Bind m) <= Monad m
```

핵심이 되는 함수는 `bind`다. 이 함수는 `Bind` 타입 클래스에 정의되어 있다. `<$>`나 `<*>` 연산자가 `Functor`와 `Apply`의 `map`과 `apply`에 매핑되듯, Prelude 에서는 `Bind` 타입 클래스의 `bind` 함수 별칭으로 `>>=` 연산자를 매핑시켜 놓았다.

`Monad` 타입 클래스는 `Bind`와 `Applicative` 타입 클래스를 더한 것이다.

`Bind` 타입 클래스의 예를 몇 가지 살펴보자. 배열에 대한 `Bind` 인스턴스는 다음과 같다.

```haskell
instance bindArray :: Bind Array where
  bind xs f = concatMap f xs
```

배열 이해 문법이 `concatMap` 함수와 연관되어 있다는 것이 여기서 설명된다.

`Maybe` 타입 생성자에 대한 `Bind` 인스턴스는 다음과 같다.

```haskell
instance bindMaybe :: Bind Maybe where
  bind Nothing  _ = Nothing
  bind (Just a) f = f a
```

do 표기법의 블록을 따라가는 중에 값이 누락되는 경우가 발생하면 마지막까지 `Nothing`이 전달되는 것이 설명된다.

이제 `Bind` 타입 클래스와 do 표기법의 관계를 살펴보자. do 표기법으로 작성된 간단한 코드 블록을 가지고 앞 줄의 계산 결과가 `bind`를 통해 다음 줄로 전달되는 방법을 알아볼 것이다.

```haskell
do value <- someComputation
   whatToDoNext
```

PureScript 컴파일러는 위와 같은 코드 패턴을 만나면 아래의 코드로 변환한다.

```haskell
bind someComputation \value -> whatToDoNext
```

`bind`에 대한 별칭 연산자를 사용하여 다음처럼 변환한다고 볼 수도 있다.

```haskell
someComputation >>= \value -> whatToDoNext
```

`whatToDoNext` 계산식은 앞 계산의 결과인 `value`를 사용할 수 있다.

`bind`가 여러번 발생하는 경우에도 위와 같은 규칙이 여러번 적용될 뿐이다. `userCity` 예제 코드에서 do 표기법을 걷어내고 나면 아래의 코드와 같아진다.

```haskell
userCity :: XML -> Maybe XML
userCity root =
  child root "profile" >>= \prof ->
    child prof "address" >>= \addr ->
      child addr "city" >>= \city ->
        pure city
```

do 표기법을 사용한 코드가 `>>=` 연산자를 사용한 코드보다 의미를 더 분명하게 전달하는 경우가 많다. 하지만 `>>=` 연산자를 직접 사용하여 바인딩하면 **Point-free 스타일**의 코드를 작성할 수도 있다. 가독성이 나아지는 경우에 한해서만 사용하도록 한다.

## Monad 법칙

`Monad` 타입 클래스에는 **모나드 법칙**이라고 하는 세 가지 법칙이 있다. `Monad` 타입 클래스의 인스턴스를 제대로 구현하려면 이 법칙들을 만족해야 한다.

do 표기법을 사용하여 이 법칙들을 간단히 나타낼 수 있다.

### 항등원 법칙

**오른쪽 항등원** 법칙은 세 가지 법칙 중에서 가장 단순한 법칙이다. 이 법칙에 따르면 do 표기법 블록의 마지막 줄에 있는 `pure`는 없어도 결과에 영향을 주지 않는다.

```haskell
do
  x <- expr
  pure x
```

오른쪽 항등원 법칙에 따라 위 블록은 그냥 `expr`만 사용한 것과 동등하다.

**왼쪽 항등원** 법칙은 do 표기법 블록의 맨 첫줄에 있는 `pure`는 없어도 결과에 영향을 주지 않는다는 것을 설명한다.

```haskell
do
  x <- pure y
  next
```

위 코드 블록은 단지 `next`만 사용한 것과 동등하다. (`next` 계산식에 포함된 `x`를 모두 `y`로 바꾸어야 한다.)

마지막 법칙은 **결합 법칙**이다. 이 법칙은 do 표기법 블록의 중첩을 설명한다. 아래와 같이 중첩된 do 블록이 있다고 하자.

```haskell
c1 = do
  y <- do
    x <- m1
    m2
  m3
```

위 코드는 아래와 동등하다.

```haskell
c2 = do
  x <- m1
  y <- m2
  m3
```

두 경우 모두 `m1`, `m2`, `m3`로 된 세 개의 모나드 계산식을 포함한다. `m1`의 계산 결과는 `x`로 바인드되고, `m2`의 계산 결과는 `y`로 바인드된다.

`c1`의 경우 `m1`과 `m2`를 따로 do 블록으로 묶어놓았다.

`c2`의 경우 `m1`, `m2`, `m3`가 모두 같은 do 블록에 나타난다.

결합 법칙에 따라 중첩된 do 블록을 단순하게 만들 수 있다.

**주의** do 표기법은 결국 `bind` 함수 호출로 변환되어 `c1`이나 `c2`는 아래의 `c3` 코드와도 동등한 의미를 가진다.

```haskell
c3 = do
  x <- m1
  do
    y <- m2
    m3
```

## Monad 접기

모나드를 추상적 수준으로 다루는 예를 살펴보자. `Monad` 타입 클래스에 포함되는 임의의 타입 생성자를 대상으로 하는 함수를 하나 보여줄 것이다. 이 함수를 통해 모나드를 사용하는 코드가 부수 효과를 지원하는 "더 큰 언어"라는 직관을 좀더 구체화할 수 있을 것이다. 그리고 모나드가 제공하는 일반화 능력도 알 수 있을 것이다.

이 함수는 바로 `foldM`이라는 함수다. 이 함수는 `foldl` 함수를 모나드 문맥으로 일반화한 것이다. 타입은 다음과 같다.

```haskell
foldM :: forall m a b
       . Monad m
      => (a -> b -> m a)
      -> a
      -> List b
      -> m a
```

모나드 `m`이 나타난 것만 빼면 `foldl` 함수의 타입과 똑같다.

```haskell
foldl :: forall a b
       . (a -> b -> a)
      -> a
      -> List b
      -> a
```

직관적으로 이해하자면, `foldM`은 부수 효과를 동반하는 계산을 통해 리스트를 접어나간다.

예를 들어보자. 만약 `m`으로 `Maybe`를 선택한다면 `foldM`이 하는 일은 각 단계마다 값을 계산하는데 실패할 수 있는 상황에서 값을 누적하여 결합해 나간다. 이 과정에서 어느 단계에서라도 `Nothing`을 반환하여 전체 접기 과정을 실패로 중단할 수도 있다. 접기의 결과 전체를 두고 보아도 성공하거나 실패할 수 있으므로 `Maybe` 타입이 된다.

만약 `m`으로 `Array` 타입 생성자를 선택한다면 어떻게 될까? 그러면 각 단계는 결과가 0 개 이상 여러 개가 될 수 있다는 의미다. 각 단계가 여러 개의 결과를 내 놓을 수 있으므로 이 과정을 모두 따라가면 최종 결과는 그 모든 조합이 된다. 마치 그래프를 순회하는 것처럼.

`foldM`의 구현은 리스트를 처리하는 일반적인 형태처럼 케이스 두 개로 나누어진다.

만약 리스트가 비어 있다면 최종 계산 결과로 내놓을 수 있는 것은 `a` 타입의 값인 두 번째 인자 뿐이다.

```haskell
foldM _ a Nil = pure a
```

`pure` 함수를 이용하여 `a`를 모나드 `m`으로 리프트했다.

리스트가 비어 있지 않다면 인자로 주어진 `a` 타입의 값과 `b` 타입의 값, 그리고 `a -> b -> m a` 타입의 함수를 이용하여 `m a` 타입의 값을 만들 수 있다. 함수를 적용한 결과인 `m a` 타입 값에서 계산 결과를 얻어내려면 역방향 화살표 `<-`를 이용하면 된다.

남은 일은 리스트의 꼬리에 대해 재귀 호출하는 것이다.

```haskell
foldM f a (b : bs) = do
  a' <- f a b
  foldM f a' bs
```

이 구현은 리스트에 대한 `foldl` 함수와 거의 똑같다. 단지 do 표기법을 사용하는 점만 다를 뿐이다.

이 함수를 PSCi 에서 정의하고 테스트해보자. 정수에 대한 "안전하게 나누기" 함수를 예로 들어보자. 0 으로 나누는 경우엔 실패를 나타내기 위하여 `Maybe` 타입 생성자를 이용한다.

```haskell
safeDivide :: Int -> Int -> Maybe Int
safeDivide _ 0 = Nothing
safeDivide a b = Just (a / b)
```

이 함수를 이용하여 안전하게 나누는 과정을 `foldM`으로 구현해보자.

```text
> import Data.List

> foldM safeDivide 100 (fromFoldable [5, 2, 2])
(Just 5)

> foldM safeDivide 100 (fromFoldable [2, 0, 4])
Nothing
```

`foldM safeDivide` 함수는 나누어가는 중에 어디라도 0 이 등장하면 `Nothing`을 반환한다. 나누는 과정에 아무런 문제가 없다면 최종 결과를 `Just` 생성자로 감싸서 반환한다.

## 모나드와 Applicative

`Monad` 타입 클래스의 모든 인스턴스들은 `Applicative` 타입 클래스의 인스턴스이기도 하다. 두 타입 클래스는 서브클래스-수퍼클래스 관계에 있기 때문이다.

하지만 `Monad` 인스턴스만 있으면 `Applicative` 인스턴스 구현이 자동으로 제공된다. 바로 `ap` 함수 덕분이다.

```haskell
ap :: forall m a b. Monad m => m (a -> b) -> m a -> m b
ap mf ma = do
  f <- mf
  a <- ma
  pure (f a)
```

`Monad` 타입 클래스의 법칙을 만족하는 `m`이라면 위의 `ap` 구현은 `Applicative` 인스턴스의 올바른 구현이 된다.

흥미를 느끼는 독자라면 `ap` 함수가 `Array`, `Monad`, `Either e` 타입에 대해 `apply`가 될 수 있는지 확인해보기 바란다.

만약 모든 모나드가 Applicative 펑터이기도 하다면 Applicative 펑터에 대해 그렸던 직관을 모나드에 적용할 수도 있어야 한다. 특히 모나드 프로그래밍 역시 부수 효과를 다루는 더 큰 "확장된 언어"로 프로그래밍하는 것으로 볼 수 있어야 한다. 그리고 이 새로운 언어로 진입하기 위한 수단으로 `map`이나 `apply` 함수를 이용하여 함수들을 리프트할 수 있어야 한다.

하지만 모나드는 그냥 Applicative 펑터를 사용할 때보다 더 많은 일을 가능하게 한다. 결정적인 차이는 do 표기를 이용할 때 뚜렷이 나타난다. `userCity` 예제를 다시 꺼내어보자. XML 문서로 된 사용자 프로필에서 도시 정보를 찾아내는 함수였다.

```haskell
userCity :: XML -> Maybe XML
userCity root = do
  prof <- child root "profile"
  addr <- child prof "address"
  city <- child addr "city"
  pure city
```

do 표기법을 사용하면 두 번째 계산식에서 첫 번째 계산식의 결과인 `prof`를 이용할 수 있다. 세 번째 계산식에서 두 번째 계산식의 결과인 `addr`를 사용할 수도 있다. 이렇게 앞 계산 결과를 그 다음 계산식에서 사용하는 것이 `Applicative` 타입 클래스로는 불가능하다.

`pure`와 `apply` 만으로 `userCity` 함수를 작성하려해도 할 수가 없다. Applicative 펑터는 함수의 인자들이 서로에 독립적인 경우만 지원하며 모나드는 계산식이 서로 의존적인 경우도 지원한다.

지난 장에서 `Applicative` 타입 클래스가 병렬성을 표현할 수 있다고 했었다. 이것이 가능한 이유는 바로 리프트하는 함수의 인자들이 서로 독립적이기 때문이다. `Monad` 타입 클래스는 앞 계산의 결과를 다음 계산이 사용할 수 있기 때문에 병렬성을 표현할 수 없다. 대신 모나드는 부수 효과를 순차적으로 합성해야 한다.

> ## 연습 문제
> 
> 1. (쉬움) `Data.Array` 모듈(`purescript-arrays` 패키지)에서 `head`와 `tail` 함수의 타입을 살펴보고 이들 함수를 이용하여 `third` 함수(배열의 세 번째 요소를 반환)를 작성해보라. do 표기법으로 작성한다. 반환 타입은 `Maybe`일 것이다.
> 1. (보통) 동전 묶음이 주어졌을 때 동전들로 만들 수 있는 금액의 모든 경우를 계산하는 함수 `sums`를 `foldM`을 이용하여 작성해보라. 동전 묶음은 각 동전의 금액을 포함하는 배열로 주어진다. PSCi에서 다음처럼 확인할 수 있어야 한다.
> 
>     ```
>     > sums []
>     [0]
>     > sums [1, 2, 10]
>     [0,1,2,3,10,11,12,13]
>     ```
> 
>     **힌트**: `foldM`을 이용하면 이 함수는 한 줄로 작성할 수 있다. `nub`이나 `sort` 함수를 이용하여 최종 결과에서 중복을 제거하고 정렬하면 된다.
> 1. (보통) `Maybe` 모나드에 대해 `ap` 함수와 `apply` 연산자가 올바르게 동작함을 확인해보라.
> 1. (보통) `purescript-maybe` 패키지에 정의된 `Maybe` 타입의 `Monad` 인스턴스가 모나드 법칙을 만족하는지 증명해보라.
> 1. (보통) 리스트 처리 함수인 `filter`를 일반화하여 `filterM` 함수를 작성해보라. 이 함수의 타입은 다음과 같다.
> 
>     ```haskell
>     filterM :: forall m a. Monad m => (a -> m Boolean) -> List a -> m (List a)
>     ```
> 
>     `Maybe`와 `Array` 모나드에 대해 이 함수가 잘 동작하는지 PSCi에서 테스트해보라.
> 1. (어려움) 모든 모나드는 다음처럼 정의된 `Functor` 인스턴스를 기본으로 가진다.
> 
>     ```haskell
>     map f a = do
>       x <- a
>       pure (f x)
>     ```
> 
>     모나드 법칙을 이용하여 모든 모나드에 대해 아래의 식이 성립함을 보여라.
> 
>     ```haskell
>     lift2 f (pure a) (pure b) = pure (f a b)
>     ```
> 
>     이미 살펴본 `ap` 함수로 `Applicative` 인스턴스가 정의되었다. 그리고 `lift2` 함수는 아래처럼 정의되어 있다.
> 
>     ```haskell
>     lift2 :: forall f a b c. Applicative f => (a -> b -> c) -> f a -> f b -> f c
>     lift2 f a b = f <$> a <*> b
>     ```

## 네이티브 효과

이제부터 살펴볼 모나드는 PureScript에서 가장 핵심이 되는 `Eff` 모나드다.

`Eff` 모나드는 Prelude와 `Control.Monad.Eff` 모듈에 정의되어 있다. `Eff` 모나드는 **네이티브** 부수 효과를 관리하기 위해 사용된다.

네이티브 부수 효과란 무엇을 말하는가? 일반적으로 PureScript 표현식은 부수 효과를 포함하지 않는다는 점에서 임의의 부수 효과를 유발할 수 있는 JavaScript 표현식과 차별화된다. JavaScript 표현식에 허용되지만 PureScript에서 배제하는 이러한 부수 효과들을 네이티브 부수 효과라고 하며, 다음의 효과들이 여기에 해당한다.

* 콘솔 IO
* 난수 생성
* 예외
* 가변 상태 읽기/쓰기

브라우저 환경이라면 다음의 부수 효과들도 포함한다.

* DOM 조작
* XMLHttpRequest / AJAX 호출
* 웹소켓 사용
* 로컬 저장소 읽기/쓰기

지금까지 살펴본 아래의 부수 효과들은 네이티브 부수 효과는 아니었다.

* 누락 가능한 값. `Maybe` 자료 형으로 표현됨.
* 오류. `Either` 자료 형으로 표현됨.
* 다중 값 함수. 배열이나 리스트로 표현됨.

네이티브냐 아니냐의 구분이 모호한 것이 사실이다. 예를 들어 오류 메시지를 예외 형태로 나타낸다면 네이티브 부수 효과라고 할 수 있다. 이 경우는 `Eff` 모나드로 표현해야 한다. 하지만 오류 메시지를 `Either`로 나타낼 수도 있으며 `Either`로 표현된 오류 메시지는 JavaScript 런타임이 발생시키는 부수 효과가 아니기 때문에 `Eff` 모나드를 사용하기에 적합하지 않다. 즉 오류 메시지라는 효과 자체가 네이티브 속성을 가지는 것은 아니고 런타임에서 어떻게 구현하느냐에 따라 네이티브 여부가 결정된다.

## 부수 효과와 순수성

PureScript와 같은 순수 언어에서 자연스럽게 나오는 질문이 있다. 바로 부수 효과 없이 어떻게 의미를 가지는 코드를 작성할 수 있는가 하는 것이다.

이 질문에 대한 답은 이렇다. PureScript는 부수 효과를 철저히 배제하지 않는다. 순수 계산식과 부수 효과를 동반하는 계산식을 구분하기 위해 부수 효과를 타입 시스템으로 나타내는 것을 목표로 한다. 이런 관점에서 언어는 여전히 순수하다 할 수 있다.

부수 효과를 유발하는 값은 순수한 값과 다른 타입을 가진다. 부수 효과를 유발하는 값을 함수의 인자로 직접 전달할 수가 없다. 이것이 허용된다면 부수 효과가 의도하지 않은 순간에 실행될 수도 있기 때문이다.

`Eff` 모나드로 관리되는 부수 효과가 실제로 발현되기 위해서는 JavaScript에서 `Eff eff a` 타입의 계산식을 실행해야만 한다.

Pulp 빌드 도구나 그 밖의 도구들은 애플리케이션이 시작될 때 `main` 계산식을 호출하기 위한 부가적인 JavaScript 코드를 쉽게 생성할 수 있게 도와준다. `main`은 `Eff` 모나드의 계산식이어야 한다.

이렇게 함으로써 우리는 어떤 부수 효과를 사용하는지 정확하게 알 수 있다. 정확하게 `main`에서 사용하는 효과들만 사용하게 된다. `Eff` 모나드를 이용하면 `main`에서 허용할 부수 효과를 제한할 수도 있다. 예를 들어 `main` 함수의 타입을 통해 애플리케이션이 콘솔 입출력만 사용하도록 제한할 수 있다.

## Eff 모나드

`Eff`모나드의 목적은 부수 효과들을 잘 정의된 타입으로 나타내고 동시에 효율적인 JavaScript를 생성할 수 있게 하기 위함이다. 이 모나드는 **확장가능한 효과** 모나드라고도 불린다. 그 이유를 곧 설명하겠다.

`purescript-random` 패키지를 사용하는 다음 예제를 보자. 이 패키지는 난수 생성 함수를 정의하고 있다.

```haskell
module Main where

import Prelude

import Control.Monad.Eff.Random (random)
import Control.Monad.Eff.Console (logShow)

main = do
  n <- random
  logShow n
```

`src/Main.purs` 파일이 위와 같다면 Pulp를 이용하여 컴파일하고 실행할 수 있다.

```text
$ pulp run
```

위 명령을 실행하면 `0`과 `1` 사이에서 랜덤하게 선택한 값 하나가 콘솔에 출력될 것이다.

이 프로그램은 do 표기법을 사용하여 JavaScript가 제공하는 두 가지 네이티브 효과(난수 생성과 콘솔 IO)를 결합하였다.

## 확장가능한 효과

PSCi에서 위 모듈을 임포트하고 `main`의 타입을 검사해보자.

```text
> import Main

> :type main
forall eff. Eff (console :: CONSOLE, random :: RANDOM | eff) Unit
```

타입이 꽤 복잡하다. 하지만 PureScript의 레코드와 비교하면 쉽게 설명할 수 있다.

레코드 타입을 사용하는 간단한 함수를 하나 보자.

```haskell
fullName person = person.firstName <> " " <> person.lastName
```

이 함수는 `firstName`과 `lastName` 속성을 가지는 레코드로부터 전체 이름에 해당하는 문자열을 만들어낸다. 이 함수의 타입을 PSCi에서 확인해보면 다음과 같다.

```haskell
forall r. { firstName :: String, lastName :: String | r } -> String
```

이 타입은 이렇게 읽으면 된다. `fullName` 함수는 `firstName`과 `lastName` 필드와 **그 밖의 필드 속성을 가지는** 레코드를 입력으로 받아 `String`을 반환한다.

다시 말해, `fullName` 함수는 레코드에 필드가 더 많아도 `firstName`과 `lastName`이 있기만 하다면 문제없이 동작한다.

```text
> firstName { firstName: "Phil", lastName: "Freeman", location: "Los Angeles" }
Phil Freeman
```

아까 살펴본 `main`의 타입도 비슷하게 해석할 수 있다. `main`은 **부수 효과를 가지는 계산식**이며, 난수 생성과 콘솔 IO를 지원하기만 한다면 **다른 어떤 부수효과를 가지는** 환경이면 실행하여 `Unit` 타입의 값을 계산할 수 있다.

이것이 바로 "확장가능한 효과"라는 이름을 가지는 이유다. 우리가 필요한 특정 부수 효과들을 지원하기만 한다면 얼마든지 부수 효과를 확장할 수 있다.

## 효과 넘나들기

이같은 확장성 덕분에 `Eff` 모나드는 여러가지 다른 부수 효과를 넘나들 수 있다.

앞서 사용한 `random` 함수의 타입은 아래와 같다.

```haskell
forall eff1. Eff (random :: RANDOM | eff1) Number
```

효과 집합을 나타내는 `(random :: RANDOM | eff1)` 부분이 `main`과 다르다.

하지만 `random` 함수의 타입을 인스턴스화 하면서 일치시킬 수 있다. 만약 `eff1`을 `(console :: CONSOLE | eff)`로 선택한다면 두 함수의 효과 집합이 순서만 바꿔서 똑같아질 수 있다.

아래와 같은 타입의 `logShow` 함수도 특수화하여 `main` 함수의 효과 집합에 일치시킬 수 있다.

```haskell
forall eff2. Show a => a -> Eff (console :: CONSOLE | eff2) Unit
```

이번에는 `eff2`를 `(random :: RANDOM | eff)`로 선택하면 된다.

여기서 중요한 점은 `random`과 `logShow` 함수의 타입이 서로 섞어 쓸 수 있는 부수 효과들을 표시한다는 점이다. 각 함수가 나타내는 부수 효과들을 결합하여 더 큰 부수 효과 집합을 가지는 더 큰 계산식을 만들 수 있다.

여기서 `main` 함수의 타입을 미리 지정하지 않았다는 점을 주목하기 바란다. `random`과 `logShow` 함수의 다형 타입을 참고하여 `main`의 가장 일반화된 타입을 찾아내는 일은 컴파일러의 몫이다.

## Eff의 카인드

`main`의 타입은 지금껏 살펴본 다른 타입들과는 다르다. 이를 설명하자면 `Eff`의 **카인드(Kind)**를 따져봐야 한다. 값을 타입으로 분류할 수 있는 것처럼 타입도 카인드로 분류할 수 있다는 사실을 떠올려보자. 지금까지는 `Type`(타입의 카인드)과 `->`(타입 생성자를 위한 카인드를 만들 때 사용)로 만들 수 있는 카인드만 다루었다.

`Eff`의 카인드를 알기 위해 PSCi에서 `:kind` 명령을 사용해보자.

```text
> import Control.Monad.Eff

> :kind Eff
# Control.Monad.Eff.Effect -> Type -> Type
```

지금까지 보지 못했던 두 가지 카인드가 등장했다.

`Control.Monad.Eff.Effect`는 **효과**를 나타내는 카인드다. 말하자면 여러 유형의 부수 효과 타입들에 대한 **타입 수준에서의 표식**인 셈이다. 앞서 `main` 함수에서 살펴본 두 가지 표식이 바로 `Control.Monad.Eff.Effect` 카인드를 가진다.

```text
> import Control.Monad.Eff.Console
> import Control.Monad.Eff.Random

> :kind CONSOLE
Control.Monad.Eff.Effect

> :kind RANDOM
Control.Monad.Eff.Effect
```

`#`는 카인드 생성자이며 순서없는 표식 집합을 나타내는 **행(row)**을 만들 때 사용한다.

즉 `Eff` 는 효과 행과 반환 타입을 인자로 가진다. 다시 말해, `Eff`의 첫 인자는 순서없는 부수 효과 타입 표식들의 집합이며, 두 번째 인자는 계산 결과의 반환 타입이다.

이제 `main`의 타입을 읽을 수 있다.

```haskell
forall eff. Eff (console :: CONSOLE, random :: RANDOM | eff) Unit
```

`Eff`의 첫 번째 인자는 `(console :: CONSOLE, random :: RANDOM | eff)`이다. 이것이 의미하는 것은 `CONSOLE` 효과와 `RANDOM` 효과를 포함하는 행이다. 파이프 기호(`|`)는 효과 표식들과 임의의 다른 부수 효과들을 나타내는 **행 변수** `eff`를 구분해준다.

두 번째 인자는 `Unit`인데, 이것은 계산 결과의 반환 타입을 의미한다.

## 레코드와 행

`Eff`의 카인드를 따져봄으로써 드디어 확장가능한 효과와 레코드 사이의 연결 고리를 볼 수 있다.

레코드를 사용하는 예제 함수를 다시 살펴보자.

```haskell
fullName :: forall r. { firstName :: String, lastName :: String | r } -> String
fullName person = person.firstName <> " " <> person.lastName
```

이 함수의 타입에서 화살표 왼쪽 부분의 카인드는 `Type` 이어야 한다. `Type` 카인드의 타입들만 값을 가질 수 있기 때문이다.

사실 레코드 타입을 표현하는 중괄호는 편의 문법일 뿐이다. PureScript 컴파일러가 보게 되는 실제 타입은 다음과 같다.

```haskell
fullName :: forall r. Record (firstName :: String, lastName :: String | r) -> String
```

중괄호가 사라지고 그 자리에 `Record`라는 생성자가 등장했다. `Record`는 `Prim` 모듈에 정의된 내장 타입 생성자다. `Record`의 카인드를 보자.

```text
> :kind Record
# Type -> Type
```

`Record` 타입 생성자는 **타입의 행**을 인자로 취한다. 레코드를 인자로 가지는 함수의 행 다형성을 가능한 것이 바로 이 때문이다.

타입 시스템 입장에서는 확장가능한 효과를 다루는 방식이나 행 다형성(혹은 **확장가능한 레코드**)을 다루는 방식이 동일하다. 단지 차이점은 표식에 나타나는 타입의 **카인드**뿐이다. 레코드는 타입의 행을 인자로 받으며, `Eff`는 효과의 행을 인자로 가진다.

이러한 타입 시스템 기능을 이용하면 타입 생성자의 행을 인자로 가지는 타입이나 다른 행의 행을 인자로 가지는 타입도 만들 수 있다.

## 작게 나눠진 효과들

`Eff` 모나드를 사용할 때는 직접 타입을 지정하지 않는 것이 일반적이다. 왜냐하면 효과 행은 타입 추론 가능하기 때문이다. 하지만 특정 계산식이 사용할 효과를 명시적으로 지정할 목적으로 컴파일러에게 타입을 알려줄 수도 있다.

난수 출력 예제에 다음처럼 **닫힌** 효과 행을 타입으로 지정할 수 있다.

```haskell
main :: Eff (console :: CONSOLE, random :: RANDOM) Unit
main = do
  n <- random
  print n
```

`eff`라고 했던 행 변수가 없어졌다. 이제 `main`을 작성하면서 실수로라도 다른 유형의 효과를 사용할 수 없다. 이렇게 함으로써 코드가 사용할 부수 효과를 통제할 수 있다.

## 핸들러와 액션

`print`와 `random` 같은 함수를 **액션**이라고 부른다. 액션은 `Eff` 타입의 결과를 내놓는다. 이 함수들의 목적은 새로운 효과를 도입하는 것이다.

이와 대비되는 **핸들러**라고 하는 함수들이 있다. `Eff` 타입을 입력으로 취하는 함수들이다. 액션이 필요한 효과 집합을 **추가**하는 반면 핸들러는 효과 집합에서 특정 효과들을 **제거**한다.

`purescript-exceptions` 패키지를 보자. 여기엔 `throwException`와 `catchException`라는 함수가 정의되어 있다.

```haskell
throwException :: forall a eff
                . Error
               -> Eff (exception :: EXCEPTION | eff) a

catchException :: forall a eff
                . (Error -> Eff eff a)
               -> Eff (exception :: EXCEPTION | eff) a
               -> Eff eff a
```

`throwException` 함수는 액션이다. `Eff`가 함수의 결과 타입에 있으며, 새로운 `EXCEPTION` 효과를 추가한다.

`catchException` 함수는 핸들러다. `Eff`가 함수의 두 번째 인자에 있으며, 결과적으로 `EXCEPTION` 효과를 제거한다.

핸들러 덕분에 타입 시스템은 특정 효과의 적용 범위를 제한할 수 있다. 효과를 사용하는 코드라도 핸들러로 감싸면 해당 효과가 허용되지 않는 코드 블록에서 사용할 수 있다.

`Exception` 효과를 예로 들어보자. 어떤 코드 조각에서 예외를 던지려고 한다. 이 코드를 `catchException`으로 감싸면 예외가 허용되지 않는 코드 블록에도 포함시킬 수 있다.

JSON 문서를 통해 애플리케이션의 설정을 읽어들이는 경우를 보자. JSON 문서를 읽는 과정에 예외가 발생할 수도 있다. 문서를 읽어들여 설정을 파싱하는 함수를 만든다면 타입이 아래와 같을 것이다.

```haskell
readConfig :: forall eff. Eff (exception :: EXCEPTION | eff) Config
```

이 함수를 `main`에서 이용할 때 `catchException` 핸들러 함수로 `EXCEPTION` 효과를 처리하여 오류가 발생하면 로그로 남기고 기본 설정값을 반환하도록 할 수 있다.

```haskell
main = do
    config <- catchException printException readConfig
    runApplication config
  where
    printException e = do
      log (message e)
      pure defaultConfig
```

`purescript-eff` 패키지에 정의된 `runPure` 핸들러는 아무런 부수 효과를 가지지 않는 순수 계산식을 입력으로 받아 순수한 값을 안전하게 계산해낸다.

```haskell
type Pure a = Eff () a

runPure :: forall a. Pure a -> a
```

## 가변 상태

가장 기본이 되는 라이브러리에는 `ST`라고 하는 또다른 효과가 정의되어 있다.

`ST` 효과는 가변 상태를 조작하기 위해 사용한다. 순수 함수형 프로그래머라면 가변 상태를 공유하는 것이 문제를 일으킨다는 것쯤은 잘 알고 있을 것이다. `ST` 효과는 타입 시스템을 이용하여 가변 상태를 **지역화**함으로써 공유를 제한한다.

`ST` 효과는 `Control.Monad.ST` 모듈에 정의되어 있다. 동작 원리를 이해하려면 액션 함수들의 타입을 보아야 한다.


```haskell
newSTRef :: forall a h eff. a -> Eff (st :: ST h | eff) (STRef h a)

readSTRef :: forall a h eff. STRef h a -> Eff (st :: ST h | eff) a

writeSTRef :: forall a h eff. STRef h a -> a -> Eff (st :: ST h | eff) a

modifySTRef :: forall a h eff. STRef h a -> (a -> a) -> Eff (st :: ST h | eff) a
```

`newSTRef`는 `STRef h a` 타입의 가변 참조 셀을 새로 만든다. 이 가변 셀의 값은 `readSTRef` 로 읽을 수 있다. `writeSTRef`와 `modifySTRef` 액션을 이용하면 가변 셀의 값을 수정할 수 있다. 여기서 타입 `a`는 실제로 셀에 저장되는 값의 타입이고 타입 `h`는 타입 시스템 상의 메모리 영역(혹은 힙)을 지정하는 용도로 사용된다.

예를 살펴보자. 중력의 영향을 받아 떨어지는 물체의 움직임을 시뮬레이션하려고 한다. 간단한 갱신 함수를 반복하는 것으로 시뮬레이션을 진행할 수 있다.

물체의 위치와 속도를 저장하기 위한 가변 참조 셀을 만들고 `for` 루프를 이용하여 셀에 저장된 값을 업데이트하면 된다. (여기서는 `Control.Monad.Eff`에 정의된 `forE` 액션을 사용한다.)

```haskell
import Prelude

import Control.Monad.Eff (Eff, forE)
import Control.Monad.ST (ST, newSTRef, readSTRef, modifySTRef)

simulate :: forall eff h. Number -> Number -> Int -> Eff (st :: ST h | eff) Number
simulate x0 v0 time = do
  ref <- newSTRef { x: x0, v: v0 }
  forE 0 (time * 1000) \_ -> do
    modifySTRef ref \o ->
      { v: o.v - 9.81 * 0.001
      , x: o.x + o.v * 0.001
      }
    pure unit
  final <- readSTRef ref
  pure final.x
```

계산이 끝나면 참조 셀의 최종 값을 읽어서 물체의 위치를 반환한다.

이 함수는 가변 상태를 이용하고 있지만 여전히 순수 함수다. 단 `ref` 참조 셀이 프로그램의 다른 영역에서 참조되지 않는다는 조건이 붙어야 된다. `ST` 효과가 하는 일이 바로 참조 셀을 바깥 영역에서 사용하지 못하게 만드는 일이다.

`ST` 효과를 가지는 계산식을 실행하려면 `runST` 함수를 사용할 수 밖에 없다.

```haskell
runST :: forall a eff. (forall h. Eff (st :: ST h | eff) a) -> Eff eff a
```

여기서 놓치지 말아야 할 것은 메모리 영역을 나타내는 `h` 타입이 함수 화살표 왼쪽의 **괄호 안쪽**으로 한정되어 있다는 점이다. `runST` 함수의 인자로 전달되는 액션은 `h`가 **어떠한 영역**이든 상관없이 동작해야 한다는 것을 의미한다.

하지만 `newSTRef`로 참조 셀을 만들 때 이미 영역 타입이 결정되어 버린다. 따라서 이렇게 생성된 참조 셀을 `runST` 바깥에서 사용하려 하면 타입 오류가 생긴다. 따라서 `runST`에서 `ST` 효과를 안전하게 제거할 수 있다.

사실 위 시뮬레이션 코드 예에서 `ST` 효과가 유일한 효과이기 때문에 `runST`와 `runPure`를 함께 사용하면 `simulate` 함수를 순수 함수로 만들 수 있다.

```haskell
simulate' :: Number -> Number -> Number -> Number
simulate' x0 v0 time = runPure (runST (simulate x0 v0 time))
```

PSCi에서 이 함수를 실행시켜보자.

```text
> import Main

> simulate' 100.0 0.0 0.0
100.00

> simulate' 100.0 0.0 1.0
95.10

> simulate' 100.0 0.0 2.0
80.39

> simulate' 100.0 0.0 3.0
55.87

> simulate' 100.0 0.0 4.0
21.54
```

`simulate` 정의에서 직접 `runST`를 호출하면 더 간단해진다.

```haskell
simulate :: Number -> Number -> Int -> Number
simulate x0 v0 time = runPure $ runST do
  ref <- newSTRef { x: x0, v: v0 }
  forE 0 (time * 1000) \_ -> do
    modifySTRef ref \o ->
      { v: o.v - 9.81 * 0.001
      , x: o.x + o.v * 0.001
      }
    pure unit
  final <- readSTRef ref
  pure final.x
```

이제 컴파일러는 참조 셀이 해당 영역 바깥으로 탈출할 수 없음을 알아차리고 안전하게 `var`로 변환할 수 있다. 아래는 `runST`로 감싸는 코드 영역을 JavaScript로 생성한 것이다.

```javascript
var ref = { x: x0, v: v0 };

Control_Monad_Eff.forE(0)((time * 1000) | 0)(function(i) {
  return function __do() {
    ref = (function(o) {
      return {
        v: o.v - 9.81 * 1.0e-3,
        x: o.x + o.v * 1.0e-3
      };
    })(ref);
    return Prelude.unit;
  };
})();

return ref.x;
```

`ST` 효과는 가변 상태를 지역적으로 사용하는 경우 짧은 JavaScript 코드를 생성해내는 방법으로 훌륭하다. 특히 `forE`, `foreachE`, `whileE`, `untilE` 등의 액션과 함께 사용하면 효율적인 루프 코드를 생성할 수 있다.

> ## 연습 문제
> 
> 1. (보통) `safeDivide` 함수에서 0으로 나누려는 경우에 `throwException`으로 예외를 던지도록 수정해보라.
> 1. (어려움) 원주율(π)의 근사값을 계산하는 방법 중 하나를 구현해보자. 이 방법은 다음과 같다.
>    1. 한 변의 길이가 1인 정사각형 내부에 무작위로 `N`개의 점을 찍는다. 
>    1. 이 점들 중 정사각형의 내접원 내부에 놓인 점의 갯수를 센다. 
>    1. 이제 `4n/N`으로 π의 근사값을 계산한다.
>
>    `RANDOM` 효과와 `ST` 효과(그리고 `forE` 함수)를 이용하여 여기서 설명한 방법으로 π의 근사값을 계산하는 함수를 작성해보라.

## DOM 효과

이 장에서 마지막으로 살펴볼 내용은 `Eff` 모나드로 DOM을 조작하는 효과를 처리하는 것이다.

PureScript로 DOM을 직접 다루거나 혹은 DOM과 관련된 라이브러리를 사용할 수 있는 패키지가 많다. 예를 들어 아래의 두 패키지가 있다.

* [`purescript-dom`](http://github.com/purescript-contrib/purescript-dom)은 브라우저의 DOM API에 대한 저수준 바인딩을 제공한다.
* [`purescript-jquery`](http://github.com/paf31/purescript-jquery)는 [jQuery](http://jquery.org) 라이브러리에 대한 바인딩을 제공한다.

다른 라이브러리를 이용하지만 추상화 수준을 더 높인 라이브러리도 있다.

* [`purescript-thermite`](http://github.com/paf31/purescript-thermite)는 `purescript-react`를 이용하고,
* [`purescript-halogen`](http://github.com/slamdata/purescript-halogen)은 가상 DOM 라이브러리 위에 타입 안정성을 더한 추상화를 제공한다.

이 장에서는 `purescript-react` 라이브러리를 이용하여 주소록 애플리케이션에 UI를 추가할 것이다. 다른 UI 라이브러리도 살펴보길 권한다.

## 주소록 UI

`purescript-react` 라이브러리를 이용하여 우리 애플리케이션을 React **컴포넌트**로 정의할 것이다. React 컴포넌트는 HTML 엘리먼트를 순수 자료 구조 형태의 코드로 표현한다. 이 자료 구조는 나중에 효율적인 방법으로 DOM 형태로 렌더링된다. 이 컴포넌트는 버튼 클릭과 같은 이벤트에 반응할 수도 있다. `purescript-react` 라이브러리는 `Eff` 모나드를 이용하여 이러한 이벤트 처리 방식을 기술한다.

React 라이브러리에 대한 완전한 튜토리얼을 제공하는 것은 이 장의 범위를 훨씬 벗어나기 때문에 독자들은 필요에 따라 라이브러리 문서를 직접 살펴보아야 할 것이다. React는 `Eff` 모나드가 실제로 사용되는 방법을 보여주는 좋은 예가 될 것이다.

주소록 UI에는 사용자가 새로운 항목을 입력할 수 있는 입력 양식이 있어야 한다. 이 양식에는 여러가지 필드들(이름, 성, 도시, 주 등)을 입력하기 위한 텍스트 상자와 입력 오류 검사의 결과를 보여줄 영역이 있어야 한다. 사용자가 텍스트 박스에 입력하면 검사 결과도 함께 업데이트하여 보여줄 것이다.

문제를 조금 단순화하기 위해 입력 양식의 모양은 고정하고, 전화 번호의 종류(집, 휴대전화, 직장 등)에 따라 별도의 텍스트 박스를 이용할 것이다.

HTML 파일은 거의 텅 빈 상태로 다음 한 줄만 있으면 된다.

```html
<script type="text/javascript" src="../dist/Main.js"></script>
```

이 줄은 Pulp로 생성되는 JavaScript 코드를 참조한다. 위 한 줄은 파일의 맨 아래쪽에 추가하여야 참조할 엘리먼트들을 안전하게 사용할 수 있다. `Main.js` 파일을 다시 만들어 내려면 Pulp에 `browserify` 명령을 추가하여 실행해야 한다. `dist` 디렉토리를 미리 만들어 두고, React는 NPM 의존성으로 추가한다.

```text
$ npm install # Install React
$ mkdir dist/
$ pulp browserify --to dist/Main.js
```

`Main` 모듈에는 `main` 함수가 정의되어 있으며 여기서 주소록 컴포넌트를 생성하고 화면에 렌더링 할 것이다. `main` 함수는 타입에 표시된 것처럼 `CONSOLE`과 `DOM` 효과만 사용한다.

```haskell
main :: Eff (console :: CONSOLE, dom :: DOM) Unit
```

먼저 `main`에서 콘솔에 상태 메시지를 로그로 남겨보자.

```haskell
main = void do
  log "Rendering address book component"
```

이제 DOM API를 사용하여 문서를 참조하는 `doc` 참조를 가져오자.

```haskell
  doc <- window >>= document
```

여기서도 여러 효과를 오가며 사용하고 있음을 알 수 있다. `log` 함수는 `CONSOLE` 효과를 사용하며 `window`와 `document` 함수는 `DOM` 효과를 사용한다. `main` 함수의 타입은 정확히 두 가지 효과를 사용하고 있음을 알려준다.

`main`은 먼저 `window` 액션으로 window 객체의 참조를 얻어내고 그 결과를 `>>=` 연산자를 통해 `document` 함수로 전달한다. `document` 함수는 window 객체를 받아서 document 객체의 참조를 반환한다.

do 표기법 정의에 따라 아래처럼 작성해도 된다.

```haskell
  w <- window
  doc <- document w
```

어느 스타일이 더 읽기 쉬운지는 개인 취향의 문제라고 볼 수 있다. 앞의 코드 형태는 **point-free** 스타일이라고 한다. (이름 붙인 인자를 사용하지 않는다.) 반면에 두 번째 형식은 window 객체에 `w`라고 이름을 붙여 사용한다.

`Main` 모듈은 주소록 **컴포넌트**를 `addressBook`으로 정의한다. 이 정의를 이해하려면 먼저 몇가지 개념부터 이해해야 한다.

React 컴포넌트를 만들기 위해서는 먼저 React **클래스**를 만들어야 한다. React 클래스는 컴포넌트의 템플릿 역할을 한다. `purescript-react` 패키지에서 제공하는 `createClass` 함수를 이용하면 React 클래스를 만들 수 있다. `createClass` 함수를 사용하려면 클래스에 대한 **명세**가 필요하다. 클래스 명세는 컴포넌트의 수명 주기를 처리하기 위한 `Eff` 액션들 모음이다. 지금 당장 필요한 것은 `Render` 액션이다.

React 라이브러리에서 제공하는 몇가지 함수들 타입을 살펴보자.

```haskell
createClass
  :: forall props state eff
   . ReactSpec props state eff
  -> ReactClass props

type Render props state eff
   = ReactThis props state
  -> Eff ( props :: ReactProps
         , refs :: ReactRefs Disallowed
         , state :: ReactState ReadOnly
         | eff
         ) ReactElement

spec
  :: forall props state eff
   . state
  -> Render props state eff
  -> ReactSpec props state eff
```

여기에서 몇 가지 흥미로운 점들을 짚어보자.

* 컴포넌트를 렌더링하는 함수의 타입을 단순하게 나타낼 수 있게 `Render` 타입 별칭을 제공한다.
* `Render` 액션은 컴포넌트의 참조(`ReactThis` 타입)를 인자로 받으며 `ReactElement` 타입을 `Eff` 모나드로 감싸서 반환한다. `ReactElement`는 렌더링하고자 하는 DOM 상태를 나타내는 자료 구조다.
* 모든 React 컴포넌트는 자신의 상태를 나타내는 타입을 정의한다. 이 상태는 버튼 클릭과 같은 이벤트에 반응하여 바뀔 수 있다. `purescript-react`에서는 컴포넌트 상태의 초기값을 `spec` 함수로 제공한다.
* `Render` 타입에서 나타난 효과 행에는 React 컴포넌트의 상태에 대한 접근을 제한하기 위한 효과가 사용되었다. 예를 들어 렌더링 중에는 `refs` 객체에 대한 접근이 허용되지 않는다.(`Disallowed`) 그리고 컴포넌트의 상태는 읽기 전용으로만 접근할 수 있다. (`ReadOnly`)

`Main` 모듈에서 정의하는 주소록 컴포넌트의 상태 타입과 초기 상태를 살펴보자.

```haskell
newtype AppState = AppState
  { person :: Person
  , errors :: Errors
  }

initialState :: AppState
initialState = AppState
  { person: examplePerson
  , errors: []
  }
```

상태에는 `Person` 레코드(입력 양식 컴포넌트에서 수정하는 내용)와 오류 목록(오류 검사를 통해 만들어진다.)이 포함되어 있다.

이제 주소록 컴포넌트 정의를 보자.

```haskell
addressBook :: forall props. ReactClass props
```

이미 언급한 것처럼 `addressBook`은 `createClass`와 `spec`을 이용하여 React 클래스를 생성할 것이다. 필요한 것은 초기 상태 값과 `Render` 액션이다. 그런데 `Render` 액션은 어떻게 만들어야 할까? 먼저 `purescript-react`에서 제공하는 몇가지 기본 액션들을 살펴보자.

```haskell
readState
  :: forall props state access eff
   . ReactThis props state
  -> Eff ( state :: ReactState ( read :: Read
                               | access
                               )
         | eff
         ) state

writeState
  :: forall props state access eff
   . ReactThis props state
  -> state
  -> Eff ( state :: ReactState ( write :: Write
                               | access
                               )
         | eff
         ) state
```

`readState`와 `writeState` 함수는 확장가능한 효과에 `ReactState` 효과를 지정하여 React 상태를 접근한다는 사실을 명시하고 있다. 게다가 세부적인 읽기/쓰기 권한조차 `ReactState` 효과의 인자에 **또다른 행**으로 나타내고 있다.

여기서 행을 이용하는 PureScript 효과 시스템의 특징이 드러난다. 행에 표시되는 효과는 꼭 싱글턴 값일 필요가 없다. 그 대신 좀더 복잡한 구조를 가질 수도 있다. 이러한 유연성 덕분에 컴파일 시점에 여러가지 유용한 제약 사항을 체크할 수 있게 된다. `purescript-react` 라이브러리가 위와 같은 제약을 사용하지 않는다면 `Render` 액션에서 상태를 업데이트하는 코드가 있을 때 실행 시점에 예외를 일으킬 수 밖에 없을 것이다. 하지만 타입 시스템에 표시된 제약 덕분에 이런 실수를 컴파일 시점에 발견할 수 있다.

`addressBook` 컴포넌트 정의를 이제 읽을 수 있을 것이다. 이 컴포넌트는 먼저 현재 상태를 읽어들인다.

```haskell
addressBook = createClass $ spec initialState \ctx -> do
  AppState { person: Person person@{ homeAddress: Address address }
           , errors
           } <- readState ctx
```

do 블록의 첫 줄이 하는 일을 보자.

* `ctx`는 `ReactThis` 참조를 가리킨다. 컴포넌트 상태를 읽거나 수정할 때엔 이 값이 필요하다.
* `AppState` 내의 레코드를 레코드 바인딩 문법으로 매치시킨다. 편의상 상태 구조체의 특정 요소들을 이름붙여 바인딩했다.

`Render` 액션은 렌더링 하고자 하는 DOM 상태를 나타내는 `ReactElement` 구조를 반환해야 한다고 했다. `Render` 액션은 몇 개의 도움 함수로 나누어 정의한다. 예를 들면 `Errors` 구조를 `ReactElement` 배열로 변환하는 `renderValidationErrors`를 도움 함수로 만들면 된다.

```haskell
renderValidationError :: String -> ReactElement
renderValidationError err = D.li' [ D.text err ]

renderValidationErrors :: Errors -> Array ReactElement
renderValidationErrors [] = []
renderValidationErrors xs =
  [ D.div [ P.className "alert alert-danger" ]
          [ D.ul' (map renderValidationError xs) ]
  ]
```

`purescript-react`에서 `ReactElement`는 대부분 `div`처럼 특정 HTML 엘리먼트를 만드는 함수로 만들어진다. 이 함수들은 HTML 엘리먼트의 속성을 나타내는 배열과 자식을 나타내는 배열을 인자로 취한다. 함수 이름 끝에 따옴표("프라임"이라고 한다)가 붙은 함수(`ul'` 같은)는 속성 배열이 생략된 버전이며, 이 경우는 기본 속성 값을 사용한다.

여기서 생성하는 값들은 보통의 자료 구조라서 `map` 같은 함수를 써서 좀더 복잡한 구조의 엘리먼트를 만들 수도 있다.

두 번째로 살펴볼 도움 함수는 `formField`이다. 이 함수는 입력 필드 하나를 나타내기 위해 텍스트 박스 하나를 포함하는 `ReactElement`를 만들어준다.

```haskell
formField
  :: String
  -> String
  -> String
  -> (String -> Person)
  -> ReactElement
formField name hint value update =
  D.div [ P.className "form-group" ]
        [ D.label [ P.className "col-sm-2 control-label" ]
                  [ D.text name ]
        , D.div [ P.className "col-sm-3" ]
                [ D.input [ P._type "text"
                          , P.className "form-control"
                          , P.placeholder hint
                          , P.value value
                          , P.onChange (updateAppState ctx update)
                          ] []
                ]
        ]
```

이번에도 더 간단한 엘리먼트를 이용하여 더 복잡한 엘리먼트를 만들었다. 각 엘리먼트마다 적절한 속성을 부여했다. 속성 중에서 `input` 엘리먼트에 적용된 `onChange` 속성을 눈여겨 보자. 이 속성은 **이벤트 핸들러**이며, 텍스트 박스의 내용을 사용자가 수정할 때 컴포넌트의 상태를 갱신하기 위해 사용된다. 이벤트 핸들러가 사용하는 세 번째 도움 함수 `updateAppState`를 보자.

```haskell
updateAppState
  :: forall props eff
   . ReactThis props AppState
  -> (String -> Person)
  -> Event
  -> Eff ( console :: CONSOLE
         , state :: ReactState ReadWrite
         | eff
         ) Unit
```

`updateAppState` 함수는 컴포넌트의 참조를 `ReactThis` 타입의 값으로 받고, 추가로 `Person` 레코드를 수정하는 함수, 그리고 처리하고자 하는 `Event` 레코드를 입력으로 받는다. 이 함수는 먼저 `change` 이벤트에 `valueOf` 도움 함수를 적용하여 텍스트 박스의 입력 내용을 꺼내고, 이 값을 이용하여 새로운 `Person` 상태를 만들어낸다.

```haskell
  for_ (valueOf e) \s -> do
    let newPerson = update s
```

그 다음 검사 함수를 실행하여 발견된 오류 항목으로 컴포넌트 상태를 갱신한다. (`writeState` 함수를 이용하면 된다.)

```haskell
    log "Running validators"
    case validatePerson' newPerson of
      Left errors ->
        writeState ctx (AppState { person: newPerson
                                 , errors: errors
                                 })
      Right _ ->
        writeState ctx (AppState { person: newPerson
                                 , errors: []
                                 })
```

여기까지 컴포넌트 구현의 기본을 살펴보았다. 하지만 컴포넌트가 어떻게 동작하는지 완전히 이해하려면 이 장에 딸린 소스 코드를 꼭 읽어봐야 할 것이다.

그리고 `pulp browserify --to dist/Main.js` 명령을 실행하고 `html/index.html`을 열어서 지금까지 작성한 UI가 동작하는 것을 꼭 확인해보라. 입력 양식에 값을 입력해보고 입력 검사 오류가 페이지에 나오는지도 확인해보라.

여기서 만든 UI는 여러가지 측면에서 개선할 점이 있다. 이 애플리케이션이 좀더 그럴싸한 모양을 갖추도록 연습 문제를 통해 개선해보자.

> ## 연습 문제
>
> 1. (쉬움) 직장 전화 번호를 입력하는 텍스트 박스를 추가해보자.
> 1. (보통) 검사 오류를 `ul` 목록으로 보여주는 대신 각 오류 항목 하나마다 `alert` 스타일을 적용한 `div`로 만들어보자.
> 1. (어려움, 확장) 여기서 만든 UI가 가진 문제점 중 하나는 입력 내용에서 검출된 오류가 해당 입력 필드 바로 옆에 나타나지 않는다는 점이다. 이 문제를 수정해보라.
>
>     **힌트**: 오류를 반환할 때 문제가 되는 필드 정보가 포함되도록 검사 함수를 확장해야 한다. `Errors` 타입을 수정해야 할 수도 있다.
>
>     ```haskell
>     data Field = FirstNameField
>                | LastNameField
>                | StreetField
>                | CityField
>                | StateField
>                | PhoneField PhoneType
>
>    data ValidationError = ValidationError String Field
>
>    type Errors = Array ValidationError
>    ```
>
>    `Errors` 구조에서 오류를 유발한 특정 `Field`를 추출하는 함수를 작성해야 할 것이다.

## 결론

이 장에서는 PureScript가 부수 효과를 다루는 것과 관련된 여러가지 개념들을 살펴보았다.

* `Monad` 타입 클래스를 보았고 이 클래스와 do 표기법이 어떻게 연결되는지도 살펴보았다.
* 모나드 법칙을 소개했고, do 표기법을 사용하여 이 법칙을 적용했을 때 코드가 어떻게 변형될 수 있는지 보았다.
* 모나드를 추상적 수준에서 다루면서 여러가지 다른 부수 효과에 대해 동작하는 코드를 작성할 수 있었다.
* 모나드가 Applicative 펑터의 한 종류이며, 부수 효과를 동반한 계산을 나타낸다는 점은 같지만 접근 방법이 다르다는 것을 살펴봤다.
* 네이티브 효과라는 개념을 정의했고, `Eff` 모나드를 이용하여 네이티브 부수 효과를 다룰 수 있음을 확인했다.
* `Eff` 모나드가 어떻게 확장가능한 효과를 지원하는지, 어떻게 같은 계산식 안에서 여러가지 다른 유형의 네이티브 효과를 함께 사용할 수 있는지 살펴봤다.
* 효과와 레코드가 사실 카인드 시스템에 의해 처리되며 확장가능한 레코드와 확장가능한 효과과 서로 연결되어 있음을 알아봤다.
* `Eff` 모나드를 이용하여 난수 생성, 예외, 콘솔 IO, 가변 상태, React를 이용한 DOM 조작 등과 같은 다양한 효과를 다루어보았다.

`Eff` 모나드는 PureScript로 진짜 프로그램을 작성하고자 할 때 가장 기본이 되는 도구다. 앞으로도 여러가지 다양한 사례에 걸쳐 부수 효과를 처리하기 위해 사용될 것이다.