# 타입 클래스

## 이 장의 목표

이 장에서는 PureScript 타입 시스템을 이루는 강력한 추상화인 타입 클래스를 소개한다.

예제로는 자료 구조에 대해 해시 값을 구하는 라이브러리를 만들 것이다. 자료 구조를 직접 참조하지 않으면서 복잡한 자료 구조에 대해 해시 값을 구하기 위한 기반 구조를 타입 클래스를 이용하여 만들어 본다.

`Prelude` 모듈을 비롯한 표준 라이브러리에 정의되어 있는 표준 타입 클래스들도 살펴볼 것이다. PureScript 코드는 간결한 표현을 위해 타입 클래스를 많이 사용하기 때문에 이미 정의된 타입 클래스들을 익혀두면 도움이 될 것이다.

## 프로젝트 설정

이 장의 소스 코드는 `src/Data/Hashable.purs` 파일이다.

프로젝트는 다음과 같은 Bower 의존성을 가진다.

- `purescript-maybe`: `Maybe` 타입이 정의되어 있다.
- `purescript-tuples`: `Tuple` 타입이 정의되어 있다.
- `purescript-either`: `Either` 타입이 정의되어 있다. 두 종류의 값 중 하나를 나타낸다. (분리합집합이라고 한다.)
- `purescript-strings`: 문자열을 다루기 위한 함수들이 정의되어 있다.
- `purescript-functions`: PureScript 함수를 정의하는데 유용한 도움 함수들이 정의되어 있다.

이 장에서 만들 `Data.Hashable` 모듈은 위의 패키지들에서 여러 모듈을 임포트한다.

## 나 자신을 보여주는 Show

처음 살펴볼 간단한 타입 클래스는 이미 여러번 본 적 있는 `show` 함수와 관련되어 있다. 이 함수는 어떤 값이든 그것을 문자열로 변환한다.

`show` 함수는 `Prelude` 모듈에 정의된 `Show` 타입 클래스에 정의되어 있다.

```haskell
class Show a where
  show :: a -> String
```

이 코드는 `Show`라는 새로운 **타입 클래스**를 선언한다. `Show` 타입 클래스는 `a` 타입 인자를 가진다.

타입 클래스의 **인스턴트**에는 타입 클래스에 정의된 함수들의 구현이 특정 타입으로 특수화되어 들어간다.

예를 들어, 아래는 `Boolean` 값에 대한 `Show` 타입 클래스의 인스턴스 구현이다. (`Prelude`에 정의되어 있다.)

```haskell
instance showBoolean :: Show Boolean where
  show true = "true"
  show false = "false"
```

이 코드는 `showBoolean`이란 이름으로 타입 클래스 인스턴스를 선언한다. PureScript에서는 JavaScript로 컴파일되었을 때 좀더 읽기 쉽도록 타입 클래스 인스턴스에 이름을 붙인다. 우리는 `Boolean` 타입이 "`Show` 타임 클래스에 속한다"고 말한다.

PSCi에서 여러 타입의 값들로 `Show` 타입 클래스를 테스트해 볼 수 있다.

```text
> import Prelude

> show true
"true"

> show 1.0
"1.0"

> show "Hello World"
"\"Hello World\""
```

`show` 함수가 여러가지 기본 타입의 값을 문자열로 변환하는 것을 확인했다. 좀더 복잡한 타입에 대해서도 `show` 함수를 사용하여 문자열로 변환할 수 있다.

```text
> import Data.Tuple

> show (Tuple 1 true)
"(Tuple 1 true)"

> import Data.Maybe

> show (Just "testing")
"(Just \"testing\")"
```

`Data.Either` 타입의 값을 `show` 함수에 전달하면 아래의 에러 메시지가 나온다.

```text
> import Data.Either
> show (Left 10)

The inferred type

    forall a. Show a => String

has type variables which are not mentioned in the body of the type. Consider adding a type annotation.
```

PSCi가 타입을 추론할 수 없다는 뜻이라기 보다는 `show`를 적용하기 위한 타입에 대해 `Show` 인스턴스가 없다는 의미다. 타입을 추론했더니 `a`라는 알 수 없는 타입이 남아있다고 말한다.

메시지의 제안에 따라 `::` 연산자를 이용하여 표현식의 타입을 덧붙이면 PSCi가 올바른 타입 클래스 인스턴스를 찾을 수 있다.

```text
> show (Left 10 :: Either Int String)
"(Left 10)"
```

`Show` 인스턴스가 정의되지 않은 타입들도 있다. 예를 들면 `->` 함수 타입이 그렇다. `Int`에서 `Int`로의 함수를 `show` 함수에 전달하면 타입 검사기가 다음과 같은 적절한 에러 메시지를 보여준다.

```text
> import Prelude
> show $ \n -> n + 1

No type class instance was found for

  Data.Show.Show (Int -> Int)
```

> ## 연습 문제
>
> 1. (쉬움) 앞 장에서 정의한 `showShape` 함수를 이용하여 `Shape` 타입에 대해 `Show` 인스턴스를 정의해보라.

## 공통 타입 클래스

이 절에서는 `Prelude`와 표준 라이브러리에 정의되어 있는 표준 타입 클래스들을 살펴보자. 이들 타입 클래스는 PureScript 코드에 자주 나타나는 표현들을 공통 패턴으로 추상화하는데 근간이 된다. 이들 타입 클래스에 정의된 함수들을 잘 이해하고 있어야 한다.

### Eq

`Eq` 타입 클래스는 `eq` 함수를 정의한다. `eq` 함수는 두 값이 동등한지 검사한다. `==` 연산자는 사실 `eq`의 별칭일 뿐이다.

```haskell
class Eq a where
  eq :: a -> a -> Boolean
```

`eq` 함수나 `==` 연산자나 비교하려는 두 인자의 타입이 반드시 같아야 한다. 타입이 다른 두 값이 같은지 비교하는 것은 말이 안된다.

PSCi에서 `Eq` 타입 클래스를 확인해보자.

```text
> 1 == 2
false

> "Test" == "Test"
true
```

### Ord

`Ord` 타입 클래스는 `compare` 함수를 정의한다. `compare` 함수는 순서 비교가 가능한 두 값을 비교하는데 사용한다. 비교 연산자 `<`와 `>`, 그리고 `<=`와 `>=`는 `compare` 함수로 정의할 수 있다.

```haskell
data Ordering = LT | EQ | GT

class Eq a <= Ord a where
  compare :: a -> a -> Ordering
```

`compare` 함수로 두 값을 비교한 결과는 `Ordering` 타입의 값으로 반환된다.

- `LT`: 첫 인자가 두 번째 인자보다 작다.
- `EQ`: 두 인자가 같다.
- `GT`: 첫 인자가 두 번째 인자보다 크다.

PSCi에서 `compare` 함수도 테스트해보자.

```text
> compare 1 2
LT

> compare "A" "Z"
LT
```

### Field

`Field` 타입 클래스는 사칙연산과 같은 수치 연산자를 지원하는 타입에 사용된다. 수치 연산자들에 대한 추상화를 제공하여 이들 연산자들이 적절히 사용되도록 한다.

**참고**: `Eq`나 `Ord` 타입 클래스처럼 `Field` 타입 클래스도 PureScript 컴파일러가 특별히 취급한다. `1 + 2 * 3`과 같은 간단한 표현식은 타입 클래스에 정의된 함수를 호출하는 대신 JavaScript로 그대로 변환된다.

```haskell
class EuclideanRing a <= Field a
```

`Field` 타입 클래스는 좀더 일반적인 **수퍼 클래스**들로 구성된다. 이렇게 여러 수퍼 클래스로 나눠서 정의한 것은 어떤 타입이 `Field` 클래스의 연산 중 일부만 지원할 수도 있기 때문이다. 예를 들어 자연수 타입이 있다면 덧셈과 곱셈에 대해서는 닫힌 형태의 정의가 가능하지만 뺄셈에 대해서는 닫혀있지 않다. 따라서 자연수 타입은 `Semiring` 클래스(`Num` 클래스의 수퍼 클래스)에 속하기는 하지만 `Ring`이나 `Field` 클래스에 속하지는 않는다.

수퍼 클래스 설명은 뒤에서 더 하겠지만 수치 타입 클래스의 전체 구조를 살펴보는 것은 이 장의 범위를 벗어나므로 관심있는 독자들은 `purescript-prelude` 패키지에서 `Field` 클래스의 수퍼 클래스들을 설명한 문서를 참조하기 바란다.

### Semigroup 과 Monoid

`Semigroup` 타입 클래스는 두 값을 `append` 함수로 결함할 수 있는 타입들을 식별한다.

```haskell
class Semigroup a where
  append :: a -> a -> a
```

문자열은 문자열 이어붙이기 연산으로 세미그룹이 될 수 있다. 이어붙이기 가능한 배열도 마찬가지다. `purescript-monoid` 패키지에서 `Semigroup` 인스턴스를 더 찾아볼 수 있다.

이어붙이기 연산자 `<>`를 이미 다루었다. 이 연산자는 `append`의 별칭이다.

`Monoid` 타입 클래스(`purescript-monoid` 패키지)는 `Semigroup` 타입 클래스를 확장하여 빈 값을 나타내는 `mempty` 함수를 추가한다.

```haskell
class Semigroup m <= Monoid m where
  mempty :: m
```

문자열이나 배열은 모노이드의 예이다.

`Monoid` 타입 클래스의 인스턴스는 여기에 속하는 타입의 결과를 빈 값으로 시작하여 누적시켜 하나의 값으로 만드는 방법을 제공한다. fold를 이용하면 모노이드에 해당하는 값의 배열을 모두 결합하여 하나의 값으로 만들 수 있다.

```haskell
> import Data.Monoid
> import Data.Foldable

> foldl append mempty ["Hello", " ", "World"]
"Hello World"

> foldl append mempty [[1, 2, 3], [4, 5], [6]]
[1,2,3,4,5,6]
```

`purescript-monoid` 패키지는 모노이드나 세미그룹에 해당하는 여러 타입들의 인스턴스가 정의되어 있다. 이 장 이후로 쓰임새를 더 볼 수 있을 것이다.

### Foldable

`Monoid` 타입 클래스는 fold의 결과가 될 수 있는 타입을 식별한다고 볼 수 있다. `Foldable` 타입 클래스는 fold의 입력이 될 수 있는 타입 생성자들을 식별한다.

`Foldable` 타입 클래스는 `purescript-foldable-traversable` 패키지로 제공되며, 이 패키지에는 `Maybe`나 배열과 같은 표준 컨테이너 타입들의 인스턴스가 정의되어 있다.

`Foldable` 클래스에서 정의하는 함수들은 지금까지 본 다른 함수들보다 타입이 복잡하다.

```haskell
class Foldable f where
  foldr :: forall a b. (a -> b -> b) -> b -> f a -> b
  foldl :: forall a b. (b -> a -> b) -> b -> f a -> b
  foldMap :: forall a m. Monoid m => (a -> m) -> f a -> m
```

`f`를 배열 타입 생성자로 지정해서 읽으면 좀더 편하다. `f a`를 `Array a`라고 바꿔보면 배열을 fold할 때 사용했던 함수와 타입이 같다는 걸 확인할 수 있다.

`foldMap`은 어떤가? `forall a m. Monoid m => (a -> m) -> Array a -> m`이라고 바꿔놓고 보면, 이 타입 시그너처로부터 우리는 결과 타입 `m`을 `Monoid` 타입 클래스에 속하는 어떤 타입이라도 가능하다는 것을 알 수 있다. 필요한 것은 배열 인자 타입 `a`로부터 결과 타입인 모노이드로 변환하는 함수이다. 일단 모노이드 구조를 이용하면 배열 요소들을 모두 누적 결합하여 하나의 값으로 만들 수 있다.

PSCi에서 `foldMap`을 테스트해 보자.

```text
> import Data.Foldable

> foldMap show [1, 2, 3, 4, 5]
"12345"
```

문자열 모노이드를 선택하여 문자열을 모두 이어붙였다. 인자로 사용한 `show` 함수는 배열의 요소인 `Int` 값들을 `String` 값으로 변환해주고, 그 결과 문자열을 모두 연결하여 단일 문자열로 결과를 얻었다.

배열만 `Foldable`에 속하는 것은 아니다. `purescript-foldable-traversable` 패키지에는 `Maybe`나 `Tuple`과 같은 타입들에 대한 `Foldable` 인스턴스가 정의되어 있다. `purescript-list` 같은 패키지는 자체적인 타입 정의와 함께 해당 타입의 `Foldable` 인스턴스 정의를 제공한다. `Foldable` 클래스는 순서가 의미를 가지는 컨테이너 타입이라는 개념을 나타낸다.

### Functor 그리고 타입 클래스의 법칙들

`Prelude`에는 부수효과(side-effect)를 동반하는 프로그래밍에 대한 함수형 스타일을 지원하는 타입 클래스들도 정의되어 있다. `Functor`, `Applicative`, `Monad` 클래스들이다. 이들 추상화에 대한 내용은 이 책 뒤에서 다루기로 하고, 여기서는 `Functor` 타입 클래스의 정의만 살펴보자. `Functor` 타입 클래스에는 `map` 함수가 정의되어 있다.

```haskell
class Functor f where
  map :: forall a b. (a -> b) -> f a -> f b
```

`map` 함수(별칭 연산자는 `<$>`)는 함수를 자료 구조에 적용할 수 있게 "리프트(lift)"시켜준다. "리프트"의 정확한 정의는 사용하는 자료 구조에 따라 달라진다. 우리는 이미 몇 가지 사례를 살펴보았다.

```text
> import Prelude

> map (\n -> n < 3) [1, 2, 3, 4, 5]
[true, true, false, false, false]

> import Data.Maybe
> import Data.String (length)

> map length (Just "testing")
(Just 7)
```

적용되는 자료 구조에 따라 서로 다르게 동작는 `map` 함수를 어떻게 이해하면 좋을까?

어떤 컨테이너가 있고, 여기에 담긴 요소(들)에 함수를 적용한 다음 그 결과로 다시 새로운 컨테이너를 기존과 같은 구조로 만들어 준다는 식의 직관적 이미지를 그릴 수 있을 것이다. 하지만 이러한 이미지를 더 정확하게 표현하는 방법은 없을까?

`Functor` 클래스의 인스턴스는 **Functor 법칙**이라고 하는 몇 가지 법칙을 만족해야만 한다.

- `map id xs = xs`
- `map g (map f xs) = map (g <<< f) xs`

첫 번째 법칙은 **항등원 법칙**이다. 이 법칙에 의하면 항등 함수를 해당 자료 구조에 적용할 수 있도록 리프트하면 기존 자료 구조와 완전히 동일한 자료 구조를 내 놓아야 한다. 항등 함수는 입력 값을 변환하지 않으므로 쉽게 이해할 수 있다.

두 번째 법칙은 **합성 법칙**이다. 이 법칙에 의하면 자료 구조에 어떤 함수를 리프트하여 적용한 다음 또다른 함수를 리프트하여 적용했을 때, 그 결과는 두 함수를 합성한 다음 리프트하여 적용했을 때와 같아야 한다.

"리프트"가 어떻게 정의되든 어떤 함수를 특정 자료 구조에 적용하도록 리프트한다면 위의 규칙들을 지켜서 정의되어야 한다.

표준 타입 클래스 다수가 이런 식의 법칙들을 동반한다. 특정 타입 클래스가 법칙들을 함께 제공한다면 우리는 해당 클래스의 인스턴스들을 좀더 일반화시켜 검토해볼 수 있다. 관심있는 독자들은 이미 살펴본 다른 표준 타입 클래스들에 어떤 법칙들이 수반되는지 확인해보길 바란다.

> ## 연습 문제
>
> 1. (쉬움) 아래의 뉴타입 정의는 복소수를 나타낸다.
>
>     ```haskell
>     newtype Complex = Complex
>       { real :: Number
>       , imaginary :: Number
>       }
>     ```
>
>     `Complex` 타입에 대해 `Show`와 `Eq` 인스턴스를 정의해보라.

## 타입 지정

타입 클래스를 이용하여 함수의 타입을 한정지을 수 있다. 예를 들어 입력 변수 세 개가 모두 같은지 확인하는 함수를 작성하려고 한다면, `Eq` 타입 클래스의 인스턴스를 사용하여 세 값이 같은지 비교할 수 있다.

```haskell
threeAreEqual :: forall a. Eq a => a -> a -> a -> Boolean
threeAreEqual a1 a2 a3 = a1 == a2 && a2 == a3
```

타입 선언을 보면 `forall`을 사용하여 정의한 보통의 다형 타입과 비슷하다. 하지만 `=>` 굵은 화살표 왼쪽에 `Eq a`가 타입 클래스 제약으로 추가되어 있다.

이 타입을 보고 우리는 `threeAreEqual` 함수의 `a` 타입 자리에 `Eq` 인스턴스가 정의된 타입이라면 어떤 타입이든 사용할 수 있다는 것을 알 수 있다. (인스턴스는 임포트되어 있어야 한다.)

타입 클래스 제약은 여러 개가 될 수 있다. 타입 클래스의 인스턴스가 단순 타입 변수에만 한정되는 것도 아니다. 아래의 또다른 예는 `Ord`와 `Show` 인스턴스를 이용하여 두 값을 비교하는 함수이다.

```haskell
showCompare :: forall a. Ord a => Show a => a -> a -> String
showCompare a1 a2 | a1 < a2 =
  show a1 <> " is less than " <> show a2
showCompare a1 a2 | a1 > a2 =
  show a1 <> " is greater than " <> show a2
showCompare a1 a2 =
  show a1 <> " is equal to " <> show a2
```

타입 클래스 제약을 여러 개 추가하기 위해 `=>` 기호가 여러번 사용된 것을 볼 수 있다. 마치 커리 함수의 인자 나열과 비슷하다. 두 기호를 헷갈리지 않아야 한다.

- `a -> b`는 **타입** `a`를 입력받아 **타입** `b`를 출력하는 함수를 나타내며
- `a => b`는 타입 `b`에 적용되는 **제약** `a`를 의미한다.

PureScript 컴파일러는 명시적으로 타입을 지정하지 않으면 타입 클래스 제약에 대해서도 추론을 시도한다. 함수를 정의하면서 가능한 일반화하고 싶을 때 컴파일러의 추론 기능이 유용하다.

PSCi를 열어 `Semiring` 타입 클래스를 추론하는 것을 테스트 해 보자.

```text
> import Prelude

> :type \x -> x + x
forall a. Semiring a => a -> a
```

여기서 이 함수를 `Int -> Int` 타입으로 지정할 수도 있고, `Number -> Number` 타입으로 지정할 수도 있다. 하지만 PSCi가 찾아낸 가장 일반화된 타입은 `Semiring` 타입 클래스라는 걸 보여준다. 따라서 우리는 이 함수가 `Int`나 `Number`에 대해 사용할 수 있다.

## 인스턴스 겹침

PureScript는 타입 클래스 인스턴스와 관련하여 **인스턴스 겹침 규칙**이라는 것을 따른다. 어떤 함수를 호출하는 자리에 타입 클래스 인스턴스가 필요하다면 PureScript는 올바른 인스턴스를 찾기 위해 타입 검사기가 추론해낸 타입을 이용한다. 이 때, 해당 타입에 대한 인스턴스는 정확히 하나만 있어야 한다. 그렇지 않으면 컴파일러가 경고를 보여준다.

이 상황을 연출하기 위해 서로 충돌하는 타입 클래스 인스턴스가 정의된 타입을 만들어보자. 아래 코드에서 우리는 `T` 타입에 대해 `Show`라는 인스턴스를 두 개 만들어 서로 겹치도록 했다.

```haskell
module Overlapped where

import Prelude

data T = T

instance showT1 :: Show T where
  show _ = "Instance 1"

instance showT2 :: Show T where
  show _ = "Instance 2"
```

이 모듈만 컴파일하면 경고 없이 잘 컴파일된다. 하지만 `show` 함수를 `T` 타입의 값에 대해 사용한다면, 즉 컴파일러가 `Show` 인스턴스를 찾아야 한다면, "인스턴스 겸침 규칙"에 의해 아래와 같은 경고를 보여준다.

```text
Overlapping instances found for Prelude.Show T
```

인스턴스 겹침 규칙은 타입 클래스 인스턴스를 자동으로 선택하는 과정을 예측 가능하게 도와준다. 어떤 타입에 대한 타입 클래스 인스턴스가 둘 이상 정의된 상황이 허용된다면 임포트 순서에 따라 어떤 것이 선택될지 알 수 없고 결과적으로는 프로그램이 실행될 때 어떻게 동작할 지 예측할 수 없게 된다. 피하고 싶은 결과다.

어떤 타입 클래스의 인스턴스가 두 개 정의되는 것이 정말 필요한 상황이라면 기존 타입을 감싸는 뉴타입(newtype)을 정의하는 것이 일반적인 접근 방법이다. 서로 다른 뉴타입은 서로 다른 타입 클래스 인스턴스를 가질 수 있기 때문에 인스턴스 겹침 규칙의 영향을 받지 않는다. PureScript 표준 라이브러리인 `purescript-maybe`에서도 `Maybe a` 타입에 대해 `Monoid` 타입 클래스 인스턴스를 여러 개 정의할 때 이 방법을 사용하였다.

## 인스턴스 의존성

함수를 정의할 때 타입 클래스로 제약을 추가하여 해당 인스턴스를 사용할 수 있듯이 타입 클래스 인스턴스를 정의할 때도 다른 타입 클래스 인스턴스를 사용할 수 있다. 이 방법은 추론 타입을 이용하여 구현을 재사용할 수 있기 때문에 매우 유용하다.

예를 들어 `Show` 타입 클래스에서 배열에 대해 인스턴스를 정의하는 경우를 보자. 배열의 요소들에 대해 `show` 함수를 적용할 수만 있다면 우리는 쉽게 배열에 대한 `show` 함수를 정의할 수 있다.

```haskell
instance showArray :: Show a => Show (Array a) where
  ...
```

만약 타입 클래스 인스턴스가 다른 여러 개의 인스턴스를 사용해야 한다면 `=>` 기호 왼쪽에 괄호로 묶고 쉼표로 구분하여 나열하면 된다.

```haskell
instance showEither :: (Show a, Show b) => Show (Either a b) where
  ...
```

위 두 개의 타입 클래스 인스턴스는 `purescript-prelude` 라이브러리에 정의되어 있다.

프로그램이 컴파일 될 때 `show` 함수 인자의 타입을 추론하고 그 타입으로 `Show` 타입 클래스에 대한 올바른 인스턴스가 선택된다. 선택된 인스턴스는 인스턴스들의 복잡한 관계에 의해 결정되지만 그 복잡성이 개발자에게 노출되지는 않는다.

> ## 연습 문제
>
> 1. (쉬움) 어떤 타입 `a`에 대해 절대 비어있지 않은 배열을 나타내기 위해 아래처럼 타입을 선언했다.
>
>     ```haskell
>     data NonEmpty a = NonEmpty a (Array a)
>     ```
>
>     `NonEmpty a` 타입에 대한 `Eq` 인스턴스를 작성해보라. `Eq a`와 `Eq (Array a)`의 인스턴스를 재사용해야 할 것이다.
> 1. (보통) `NonEmpty a` 타입에 대한 `Semigroup` 인스턴스를 작성해보라. `Array`에 대한 `Semigroup` 인스턴스를 재사용하면 된다.
> 1. (보통) `NonEmpty`에 대한 `Functor` 인스턴스를 작성해보라.
> 1. (보통) `a` 타입에 대해 `Ord` 인스턴스가 주어졌을 때 해당 타입의 어떤 값보다 더 큰 "무한대"를 추가하여 아래처럼 확장된 타입을 정의할 수 있다.
>
>     ```haskell
>     data Extended a = Finite a | Infinite
>     ```
>
>     `Extended a`  타입에 대한 `Ord` 인스턴스를 작성해보라. `a` 타입의 `Ord`  인스턴스를 재사용하면 된다.
> 1. (어려움) `NonEmpty` 타입에 대한 `Foldable` 인스턴스를 작성해보라. **힌트**: 배열의 `Foldable` 인스턴스를 재사용하라.
> 1. (어려움) 순서가 있는 컨테이너를 정의하는 타입 생성자 `f`가 있을 때 (즉, `f`는 `Foldable` 인스턴스가 정의되어 있다.) 우리는 컨테이너 맨 앞에 요소가 하나 더 추가된 새로운 컨테이너 타입을 정의할 수 있다.
>
>     ```haskell
>     data OneMore f a = OneMore a (f a)
>     ```
>
>     `OneMore f` 컨테이너의 요소들도 순서가 있다.(맨 앞에 추가된 요소가 `f`의 다른 요소들보다 앞에 있다고 본다.) `OneMore f` 타입에 대해 `Foldable` 인스턴스를 작성해보라.
>
>     ```haskell
>     instance foldableOneMore :: Foldable f => Foldable (OneMore f) where
>       ...
>     ```

## 다중 인자 타입 클래스

타입 클래스의 타입 인자가 꼭 하나만 되는 것은 아니다. 하나인 경우가 많기는 하지만 사실 타입 클래스는 타입 인자를 **0개부터 여러 개**까지 가질 수 있다.

타입 인자를 2개 갖는 타입 클래스를 하나 살펴보자.

```haskell
module Stream where

import Data.Array as Array
import Data.Maybe (Maybe)
import Data.String as String

class Stream stream element where
  uncons :: stream -> Maybe { head :: element, tail :: stream }

instance streamArray :: Stream (Array a) a where
  uncons = Array.uncons

instance streamString :: Stream String Char where
  uncons = String.uncons
```

`Stream` 모듈에 `Stream` 클래스가 정의되어 있다. 이 타입 클래스는 여러 요소들을 스트림처럼 가지며 `uncons` 함수를 이용하여 스트림으로부터 맨 앞의 요소를 하나 꺼낼 수 있는 타입을 식별한다.

`Stream` 타입 클래스는 스트림 타입뿐만아니라 요소 타입도 인자로 지정할 수 있다. 이렇게 함으로써 스트림 타입은 같지만 요소 타입이 다른 경우에 대해서도 인스턴스를 따로 정의할 수 있다.

이 모듈에는 클래스 인스턴스가 두 개 정의되어 있다. 하나는 배열에 대해 정의되어 있고 다른 하나는 문자열에 대해 정의되어 있다.

이제 우리는 임의의 스트림에 대해서 동작하는 함수들을 정의할 수 있다. 예를 들어 스트림의 요소가 `Monoid`인 경우 전체를 누적 결합하는 함수를 정의할 수 있다.

```haskell
import Prelude
import Data.Monoid (class Monoid, mempty)

foldStream :: forall l e m. Stream l e => Monoid m => (e -> m) -> l -> m
foldStream f list =
  case uncons list of
    Nothing -> mempty
    Just cons -> f cons.head <> foldStream f cons.tail
```

PSCi에서 여러가지 다른 타입의 `Stream`과 다른 종류의 `Monoid`에 대해 `foldStream`을 테스트해보라.

## 함수적 의존 관계

다중 인자 타입 클래스는 매우 유용하게 사용될 수 있지만 타입이 너무 복잡해져서 타입 추론에 문제가 생기기도 한다. 앞에서 본 `Stream` 클래스에 대해 일반화된 `tail` 함수를 정의하는 경우를 살펴보자.

```haskell
genericTail xs = map _.tail (uncons xs)
```

위 코드는 아래와 같은 오류 메시지를 유발한다.

```text
The inferred type

  forall stream a. Stream stream a => stream -> Maybe stream

has type variables which are not mentioned in the body of the type. Consider adding a type annotation.
```

`Stream` 타입 클래스의 타입 인자들 중에서 `element` 타입이 `genericTail` 함수 정의에 사용되지 않았기 때문에 타입을 결정지을 수 없다는 불만이다.

뿐만아니라 `genericTail` 함수의 정의에 해당하는 표현식을 특정 스트림 타입의 값에 직접 적용할 수도 없다.

```text
> map _.tail (uncons "testing")

The inferred type

  forall a. Stream String a => Maybe String

has type variables which are not mentioned in the body of the type. Consider adding a type annotation.
```

위 표현식에서는 컴파일러가 `streamString` 인스턴스를 선택할 수 있어야 할것 같다. `String`은 결국 `Char` 값의 스트림이지 다른 타입의 스트림이 될 수 없기 때문이다.

컴파일러는 이와 같은 유도 과정을 자동으로 이끌어낼 수 없기 때문에 `streamString` 인스턴스로 결정지을 수 없다. 하지만 우리가 타입 클래스를 정의하면서 힌트를 추가하는 방법으로 컴파일러를 도와줄 수 있다.

```haskell
class Stream stream element | stream -> element where
  uncons :: stream -> Maybe { head :: element, tail :: stream }
```

여기서 `stream -> element` 부분을 **함수적 의존 관계**라고 한다. 함수적 의존 관계는 다중 인자 타입 클래스의 타입 인자들 사이에 함수적 관계가 있음을 나타낸다. 컴파일러는 함수적 의존 관계를 보고 스트림 타입에서 스트림 요소 타입으로의 함수가 있다는 것을 알 수 있다. 즉, 컴파일러가 스트림 타입만 알게 되면 요소 타입도 결정지을 수 있다.

이 힌트를 통해 컴파일러는 `genericTail` 함수의 올바른 타입을 추론할 수 있다.

```text
> :type genericTail
forall stream element. Stream stream element => stream -> Maybe stream

> genericTail "testing"
(Just "esting")
```

함수적 의존 관계는 다중 인자 타입 클래스로 API를 설계하는 경우에 매우 유용하다.

## 인자 없는 타입 클래스

타입 인자가 없는 타입 클래스를 정의할 수도 있다. 인자 없는 타입 클래스는 컴파일 시점에 함수들의 단언(assertion) 역할을 한다. 타입 시스템을 통해 코드가 만족해야 하는 속성들을 추적할 수 있게 한다.

부분 함수를 살펴볼 때 본 적 있는 `Partial` 타입 클래스가 주요 사례에 해당한다. `Data.Array.Partial` 모듈에 정의된 `head`나 `tail` 함수가 부분 함수인 것은 이미 알고 있다.

```haskell
head :: forall a. Partial => Array a -> a

tail :: forall a. Partial => Array a -> Array a
```

`Partial` 타입 클래스에 해당하는 인스턴스는 없다는 점을 알아둬야 한다. `head`  함수를 직접 사용하려 하면 다음과 같은 타입 오류를 일으킨다.

```text
> head [1, 2, 3]

No type class instance was found for

  Prim.Partial
```

부분 함수를 사용하는 다른 함수들에도 `Partial` 제한을 걸어줘야 한다.

```haskell
secondElement :: forall a. Partial => Array a -> a
secondElement xs = head (tail xs)
```

`unsafePartial` 함수는 부분 함수를 일반 함수처럼 사용할 수 있게 해 준다. 대신 안전하지 않을 수 있다. 이 함수는 `Partial.Unsafe` 모듈에 정의되어 있다.

```haskell
unsafePartial :: forall a. (Partial => a) -> a
```

타입에 나타난 `Partial` 제약이 바깥의 `forall` 부분이 아니라 괄호 안쪽에 있다. 즉 `unsafePartial` 함수는 부분적인 값을 일반적인 값으로 바꿔주는 함수다.

## 수퍼클래스

타입 클래스의 인스턴스를 정의할 때 다른 타입 클래스 인스턴스를 사용하여 정의한 것처럼 타입 클래스 자체를 정의할 때 다른 타입 클래스를 사용할 수 있다. 여기서는 **수퍼클래스**를 말한다.

두 타입 클래스가 있는데, 어느 한쪽 클래스의 모든 인스턴스가 반드시 다른 한쪽 클래스의 인스턴스여야 하는 경우에 후자에 해당하는 클래스를 전자 클래스의 수퍼클래스라고 한다. 클래스 정의에서 수퍼클래스를 지정하려면 왼쪽을 바라보는 두꺼운 화살표를 사용한다.

이미 살펴본 몇가지 클래스들이 수퍼클래스 관계를 가진다. 대표적으로 `Eq` 클래스는 `Ord` 클래스의 수퍼클래스이고, `Semigroup` 클래스는 `Monoid` 클래스의 수퍼클래스다. `Ord` 클래스의 모든 인스턴스마다 `Eq` 인스턴스도 정의되어야 한다. 이런 수퍼클래스 관계는 쉽게 파악된다. 예를 들어 두 값의 크기가 어느 한쪽이 크거나 작다고 할 수 없는 경우에 우리는 `Eq` 클래스를 이용하여 실제로 같은지 결정할 수 있다.

일반적으로 서브클래스의 법칙들은 수퍼클래스의 법칙들을 인용하여 기술된다. 예를 들어 어떤 타입에 대한 `Ord`와 `Eq` 인스턴스가 있다고 하자. 이 타입의 두 값이 `Eq`  인스턴스를 통해 같다고 한다면 `compare`  함수는 당연히 `EQ` 결과를 내놓아야 한다. 다시 말하자면 `a == b`인 경우 반드시 `compare a b`의 결과가 `EQ`여야 한다. 법칙 수준의 관계가 `Eq`와 `Ord`의 수퍼클래스 관계를 잘 보여준다.

수퍼클래스 관계를 정의하는 또다른 이유는 두 클래스 사이에 명백히 드러나는 "is-a" 관계를 나타내기 위함이다. 서브클래스의 모든 멤버들은 수퍼클래스의 멤버이기도 하다.

> ## 연습 문제
>
> 1. (보통) 비어있지 않은 정수 배열에서 최댓값을 찾는 부분 함수를 정의해보라. 함수의 타입은 `Partial => Array Int -> Int`여야 한다. PSCi에서는 `unsafePartial`을 이용하여 테스트해볼 수 있다. **힌트**: `Data.Foldable` 모듈의 `maximum` 함수를 이용하여 정의하면 된다.
> 1. (보통) 아래의 `Action` 클래스는 어떤 타입에 대한 액션을 정의하는 다중 인자 타입 클래스다.
>
>     ```haskell
>     class Monoid m <= Action m a where
>       act :: m -> a -> a
>     ```
>
>     여기서 **액션**이라 함은 모노이드에 해당하는 값을 이용하여 다른 타입의 어떤 값을 변환하는 함수를 말한다. `Action` 클래스는 다음의 두 법칙을 만족해야 한다.
>
>     - `act mempty a = a`
>     - `act (m1 <> m2) a = act m1 (act m2 a)`
>
>     이 법칙들이 말하는 바는 액션이 `Monoid` 클래스의 연산들에 따르는 법칙들에 부합하여야 한다는 것이다.
>
>     예를 들어 자연수는 곱셈 연산에 있어서 모노이드가 된다.
>
>     ```haskell
>     newtype Multiply = Multiply Int
>
>     instance semigroupMultiply :: Semigroup Multiply where
>       append (Multiply n) (Multiply m) = Multiply (n * m)
>
>     instance monoidMultiply :: Monoid Multiply where
>       mempty = Multiply 1
>     ```
>
>     이 모노이드를 이용하여 입력 문자열을 반복하여 연결하는 액션을 정의할 수 있다. 다음의 액션 인스턴스를 정의해보라.
>
>     ```haskell
>     instance repeatAction :: Action Multiply String
>     ```
>
>     이 인스턴스는 위에서 열거한 앤션 클래스의 법칙들을 만족하는가?
> 1. (보통) `Action m a => Action m (Array a)`에 해당하는 인스턴스를 작성해보라. 배열에 대한 액션은 배열의 각 요소들에 독립적으로 적용되어야 한다.
> 1. (어려움) 아래와 같은 뉴타입이 있을 때 `Action m (Self m)`에 대한 인스턴스를 작성해보라. 여기서 모노이드 `m`은 자기자신에 대해 `append` 액션이 적용된다.
>
>     ```haskell
>     newtype Self m = Self m
>     ```
> 1. (어려움) 다중 인자 타입 클래스로 정의된 `Action` 클래스에 함수적 의존 관계가 필요한가? 필요하다면 왜 그러한지, 필요하지 않다면 왜 필요하지 않은지 이유를 설명해보라.

## 해싱 가능한 타입을 위한 타입 클래스

자료 구조에 대해 해싱을 계산하기 위한 라이브러리를 만들어보자.

이 라이브러리는 데모를 위한 것이기 때문에 완벽한 해싱 방법을 다룰 것으로 기대하지 않기 바란다.

해시 함수는 어떤 특성을 가지고 있어야 할까?

- 해시 함수는 매번 같은 값을 계산해내야 한다. 즉 같은 값에 대해서는 해시 값도 같아야 한다.
- 해시 함수는 가능한 고른 분포로 값을 계산해내야 한다.

첫번째 속성은 타입 클래스에서 이야기하는 법칙과 비슷하지만 두번째 속성은 PureScript 타입 체계만으로는 강제하기 어려운 측면이 있다. 다음과 같은 타입 클래스로 나타낼 수 있다.

```haskell
newtype HashCode = HashCode Int

hashCode :: Int -> HashCode
hashCode h = HashCode (h `mod` 65535)

class Eq a <= Hashable a where
  hash :: a -> HashCode
```

`Hashable` 타입은 `a == b`라면 `hash a == hash b`라는 법칙을 만족해야 한다.

이제부터 `Hashable` 타입 클래스와 관련된 함수들과 인스턴스들을 만들어보자.

우선 해시코드를 결합하는 방법이 필요하다.

```haskell
combineHashes :: HashCode -> HashCode -> HashCode
combineHashes (HashCode h1) (HashCode h2) = hashCode (73 * h1 + 51 * h2)
```

`combineHashes` 함수는 두 개의 해시코드를 결합하여 0~65535 범위 값으로 만든다.

`Hashable` 제약을 가지는 함수를 하나 작성해보자. 해시 함수를 사용하는 대표적인 경우는 어떤 두 값의 해시코드가 같은지 검사하는 것이다. `hashEqual` 함수는 이러한 관계를 나타낸다.

```haskell
hashEqual :: forall a. Hashable a => a -> a -> Boolean
hashEqual = eq `on` hash
```

이 함수는 `Data.Function` 모듈의 `on` 함수를 이용하여 작성되었다. 해시코드가 같은지 비교하는데, 구현 코드가 자연스럽게 읽힌다. 두 값은 `hash` 함수를 적용하여 그 결과가 같을 때 "해시가 같다"고 할 수 있다.

몇가지 기본 타입에 대해 `Hashable` 인스턴스를 작성해보자. 먼저 정수 타입에 대한 인스턴스부터 시작하자. `HashCode`는 사실 정수 타입에 대한 래퍼 타입이기 때문에 `hashCode` 도움 함수를 그대로 사용할 수 있다.

```haskell
instance hashInt :: Hashable Int where
  hash = hashCode
```

`Boolean` 타입에 대한 인스턴스도 쉽게 구현할 수 있다.

```haskell
instance hashBoolean :: Hashable Boolean where
  hash false = hashCode 0
  hash true  = hashCode 1
```

정수 타입에 대한 인스턴스를 이용하면 `Char` 타입을 해싱하는 인스턴스도 만들 수 있다. `Data.Char`  모듈의 `toCharCode` 함수를 이용하면 된다.

```haskell
instance hashChar :: Hashable Char where
  hash = hash <<< toCharCode
```

배열에 대한 인스턴스를 정의하려면 `hash` 함수를 배열 요소들에 `map`시키고(물론 요소 타입 역시 `Hashable` 인스턴스가 정의되어야 한다) 그 결과 해시코드들을 `combineHashes` 함수로 결합하면 된다.

```haskell
instance hashArray :: Hashable a => Hashable (Array a) where
  hash = foldl combineHashes (hashCode 0) <<< map hash
```

더 단순한 타입의 인스턴스를 이용하여 점점 더 복잡한 타입에 대한 인스턴스를 만들고 있다. 이제 `Array` 인스턴스를 이용하여 `String` 타입에 대한 인스턴스도 만들어보자. `String`을 `Char` 배열로 바꾸기만 하면 된다.

```haskell
instance hashString :: Hashable String where
  hash = hash <<< toCharArray
```

지금까지 만든 `Hashable` 인스턴스들이 `Hashable` 타입 클래스에서 요구하는 법칙을 잘 만족할까? 이를 확인하기 위해서는 같은 값에 대해 해시코드도 같은지 봐야 한다. `Int`, `Char`, `String`, `Boolean` 같은 타입의 경우는 명백하다. 이들 값들이 `Eq`로는 같은데 해시코드가 다른 경우는 있을 수 없다.

조금 복잡한 타입들은 어떨까? `Array` 인스턴스가 타입 클래스에서 요구하는 법칙을 따르는지 살펴보자. 증명하려면 귀납적으로 접근하면 된다. 길이가 0인, 즉 `[]` 배열 두 개는 서로 같으면서 해시코드도 같다. 배열에 대한 `Eq` 정의에 따라 비어있지 않은 어떤 두 배열이 같으려면 반드시 두 배열의 머리 요소끼리, 그리고 꼬리끼리 같아야 한다. 귀납적 가정에 따라 두 꼬리가 같은 해시 값을 가진다고 하자. `Hashable a` 인스턴스에 의해 두 머리 요소는 서로 같은 해시코드를 가진다. 따라서 두 배열은 같은 해시코드를 가지고 결국 `Hashable (Array a)`는 타입 클래스가 요구하는 법칙을 만족한다.

이 장의 소스코드에는 `Maybe`나 `Tuple` 타입을 포함하여 `Hashable` 인스턴스가 더 정의되어 있다.

> ## 연습 문제
>
> 1. (쉬움) PSCi를 이용하여 이미 정의한 여러 인스턴스들을 테스트해보라.
> 1. (보통) `hashEqual` 함수를 이용하여 배열 내에 중복 요소가 있는지 검사하는 함수를 작성해보라. 이 함수는 해시가 같은 경우 값도 같을 것으로 추정한 뒤 최종적으로는 `==`를 이용하여 정말 같은지 비교해야 한다. **힌트**: `Data.Array` 모듈에 정의된 `nubBy` 함수를 이용하면 좀더 쉬울 것이다.
> 1. (보통) 아래와 같은 뉴타입에 대해 `Hashable` 인스턴스를 정의하라. `Hashable` 타입 클래스의 법칙을 만족해야 한다.
>
>     ```haskell
>     newtype Hour = Hour Int
>
>     instance eqHour :: Eq Hour where
>       eq (Hour n) (Hour m) = mod n 12 == mod m 12
>     ```
>
>     `Hour` 뉴타입의 `Eq` 인스턴스는 12로 나눈 나머지만 따진다. 즉 1과 13은 같다고 본다. `Hashable` 인스턴스가 법칙을 만족함을 증명하라.
> 1. (어려움) `Maybe`, `Either`, `Tuple` 타입의 `Hashable` 인스턴스가 법칙을 만족함을 증명하라.

## 결론

이 장에서 우리는 **타입 클래스**를 살펴봤다. 타입 클래스는 타입 수준의 추상화 수단으로서 코드 재사용성을 높여주는 강력한 도구다. PureScript 표준 라이브러리에 정의된 표준 타입 클래스들을 살펴봤고, 해시 값 계산을 위한 타입 클래스를 만들고 우리만의 라이브러리를 만들어보았다.

타입 클래스 법칙도 살펴봤다. 타입 클래스 추상화를 사용하는 코드가 만족해야 하는 속성들을 직접 증명해보기도 했다. 타입 클래스 법칙은 **등식 추론(equational reasoning)**이라고 하는 더 큰 주제의 일부분이다. 이는 프로그램을 논리적으로 추론할 수 있게 하는 프로그래밍 언어와 그 타입 시스템의 특징을 다룬다. 매우 중요한 개념이며 이 책의 남은 부분을 통해 자주 접하게 될 주제이기도 하다.
