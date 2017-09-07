# 패턴 매칭

## 이 장의 목표

이 장에서는 대수적 자료형(algebraic data type)과 패턴 매칭(pattern matching)을 소개한다. PureScript 타입 시스템의 기능 중에서 "행 다형성(row polymorphism)"이라고 하는 흥미로운 기능도 가볍게 살펴볼 것이다.

패턴 매칭은 함수형 프로그래밍에서 일반적인 기술이다. 자칫 복잡해질 수 있는 함수를 케이스 별로 나눠서 기술함으로써 간결하게 작성할 수 있게 도와준다.

PureScript 타입 시스템이 지원하는 대수적 자료형은 타입을 정의할 때도 비슷한 수준의 표현력을 제공하며, 패턴 매칭과도 밀접하게 관련되어 있다.

이 장의 목표는 대수적 자료형과 패턴 매칭을 이용하여 간단한 벡터 그래픽 처리 라이브러리를 만들어보는 것이다.

## 프로젝트 설정

이 장의 소스 코드는 `src/Data/Picture.purs`이다.

프로젝트의 Bower 의존성에는 이미 살펴본 패키지들 외에 다음 패키지가 추가되어 있다.

- `purescript-globals`: JavaScript 언어에서 제공하는 전역 값과 함수를 접근할 수 있다.
- `purescript-math`: JavaScript의 Math 모듈을 접근할 수 있다.

`Data.Picture` 모듈에 정의된 `Shape` 타입은 간단한 도형을 나타내고, `Picture` 타입은 도형들의 집합을 나타낸다. 이 밖에도 이 타입들을 다루기 위한 함수들이 정의되어 있다.

이 모듈은 `Data.Foldable` 모듈의 `foldl`을 사용하여 자료 구조를 fold한다.

```haskell
module Data.Picture where

import Prelude
import Data.Foldable (foldl)
```

`Data.Pictrure` 모듈은 `Global`과 `Math` 모듈도 임포트하는데, `as` 키워드를 사용하고 있다.

```haskell
import Global as Global
import Math as Math
```

임포트할 때 `as` 키워드를 사용하면 모듈에 정의된 타입과 함수를 사용할 때 **한정 이름**을 사용해야 한다. 예를 들어 `Global.infinity`나 `Math.max` 처럼 사용해야 한다. 임포트되는 모듈이 같은 이름의 타입이나 함수를 가지고 있는 경우 충돌을 피할 때 유용하다. 해당 이름을 제공하는 모듈이 무엇인지 드러내어 코드를 더 분명하게 만드는 목적으로 사용할 수도 있다.

**주의**: 이름을 한정하여 임포트하는 경우 꼭 원래의 모듈 이름과 같을 필요는 없다. `import Math as M`과 같이 짧게 정할 수도 있고 실제로도 그렇게 많이 사용한다.

## 단순한 패턴 매칭

예제부터 살펴보자. 아래 함수는 두 정수의 최대공약수를 패턴 매칭을 이용하여 계산한다.

```haskell
gcd :: Int -> Int -> Int
gcd n 0 = n
gcd 0 m = m
gcd n m = if n > m
            then gcd (n - m) m
            else gcd n (m - n)
```

이 알고리즘은 유클리드 알고리즘이다. 온라인으로 검색해보면 위 코드와 유사한 모양의 수학 등식을 찾을 수 있을 것이다. 이것이 바로 패턴 매칭의 장점 중 하나이다. 케이스 별로 정의함으로써 마치 수학에서의 함수 정의처럼 보이는 선언적 스타일의 코드를 작성할 수 있다.

패턴 매칭으로 작성된 함수는 매칭을 위한 조건과 그 결과를 짝으로 연결한다. 각 줄은 **선택지**나 **케이스**라고 한다. 등호 왼쪽의 표현식을 **패턴**이라고 하며, 각 케이스는 하나 이상의 패턴이 공백으로 구분되어 있다. 케이스는 인자들이 만족시켜야 하는 조건과 그 조건이 만족되었을 때 계산할 표현식을 등호로 연결한다. 케이스가 선언된 순서대로 검사하여 패턴이 만족되는 첫 번째 케이스에서 반환값이 계산된다.

예를 들어 `gcd` 함수는 다음의 절차에 따라 계산된다.

- 첫 번째 케이스를 검사한다. 두 번째 인자가 0이면 이 함수는 `n`(첫 번째 인자)을 반환한다.
- 아니면, 두 번째 케이스를 검사한다. 첫 번째 인자가 0이면 이 함수는 `m`(두 번째 인자)을 반환한다.
- 아니면, 이 함수는 마지막 줄의 표현식을 계산하여 그 값을 반환한다.

패턴은 값을 이름에 바인딩할 수 있다. 위 예제의 각 줄은 `n`이나 `m` 같은 이름을 입력 인자에 바인딩한다. 다양한 유형의 패턴들을 배우면서 입력 인자에 이름을 바인딩하는 다양한 방법도 함께 살펴볼 것이다.

## 단순 패턴

위 예제는 두 가지 유형의 패턴을 보여줬다.

- 정수 리터럴 패턴: 입력 값이 `Int` 타입이고 그 값도 완전히 일치할 때 매칭된다.
- 변수 패턴: 입력 인자를 변수 이름으로 바인딩하며 매칭된다.

이 밖에도 단순 패턴으로 다음과 같은 유형이 있다.

- `Number`, `String`, `Char`, `Boolean` 리터럴
- 와일드카드 패턴: 밑줄(`_`)로 표시하며 입력이 무엇이든 매칭되며 이름으로 바인딩하지 않는다.

위의 단순 패턴들이 사용된 예제를 살펴보자.

```haskell
fromString :: String -> Boolean
fromString "true" = true
fromString _      = false

toString :: Boolean -> String
toString true  = "true"
toString false = "false"
```

이 함수들을 PSCi에서 테스트해보자.

## 패턴 가드

유클리드 알고리즘 예제에서 `m > n`인 경우와 `m <= n`인 경우를 따지기 위해 우리는 `if .. then .. else`를 사용하였다. 이런 경우에 **가드**를 사용할 수도 있다.

가드는 패턴과 더불어 케이스를 선택하기 위해 만족시켜야 조건식이다. 아래는 가드를 사용하여 유클리드 알고리즘을 구현한 것이다.

```haskell
gcd :: Int -> Int -> Int
gcd n 0 = n
gcd 0 n = n
gcd n m | n > m     = gcd (n - m) m
        | otherwise = gcd n (m - n)
```

세 번째 케이스의 첫 줄을 보면 가드를 사용하여 첫 번째 인자가 두 번째 인자보다 커야 한다는 추가 조건을 나타냈다.

예제 코드에서 보여주듯이 가드는 등호 왼쪽의 패턴 목록 다음에 파이프 문자(`|`)로 구분하여 표시한다.

> ## 연습 문제
>
> 1. (쉬움) 패턴 매칭을 사용하여 팩토리얼 함수를 작성해보라. **힌트**: 입력이 0인 케이스와 그렇지 않은 케이스로 나눠 생각해보라.
> 1. (보통) 이항 계수를 계산하기 위한 **파스칼 규칙(Pascal's Rule)**을 찾아보고 이 규칙을 이용하여 이항 계수를 계산하는 함수를 작성해보라. 패턴 매칭을 사용해야 한다.

## 배열 패턴

**배열 리터럴 패턴**을 사용하면 고정 길이의 배열을 매칭할 수 있다. 예를 들어 배열이 비어있는지 확인하기 위한 `isEmpty` 함수를 작성하고자 할 때 빈 배열 패턴(`[]`)을 첫 번째 케이스로 사용할 수 있다.

```haskell
isEmpty :: forall a. Array a -> Boolean
isEmpty [] = true
isEmpty _ = false
```

길이가 5인 배열을 매칭시키며 각 요소들을 서로 다른 방법으로 바인딩할 수도 있다.

```haskell
takeFive :: Array Int -> Int
takeFive [0, 1, a, b, _] = a * b
takeFive _ = 0
```

첫 번째 패턴을 보면 요소가 다섯 개이고, 그 중 첫 번째 요소와 두 번째 요소가 각각 0과 1인 배열을 매칭시킨다. 이 케이스는 세 번째 요소와 네 번째 요소의 곱을 반환한다. 그 외의 모든 경우에 대해 이 함수는 0을 반환한다. PSCi에서 테스트해보자.

```text
> :paste
… takeFive [0, 1, a, b, _] = a * b
… takeFive _ = 0
… ^D

> takeFive [0, 1, 2, 3, 4]
6

> takeFive [1, 2, 3, 4, 5]
0

> takeFive []
0
```

배열 리터럴 패턴을 이용하면 고정된 길이의 배열을 매칭시킬 수 있다. 하지만 PureScript에서는 길이를 지정하지 않고 배열을 매칭시킬 방법을 제공하지 않는다. 길이를 알 수 없는 불변 배열에 대해 요소들을 분해하여 바인딩하려면 좋은 성능을 기대할 수 없기 때문이다. 만약 이러한 식의 매칭이 필요하다면 `Data.List` 자료 구조를 사용하면 된다. 다른 유형의 연산들에 대해 좋은 성능을 보여주는 다른 자료 구조들도 있다.

## 레코드 패턴과 행 다형성

**레코드 패턴**은 (당연하게도) 레코드를 매칭하기 위해 사용한다.

레코드 패턴은 레코드 리터럴과 비슷하다. 다만 각 필드의 콜론 뒤 값이 들어갈 자리에 패턴이 들어간다.

예를 들어 아래의 패턴은 `first`와 `last`라는 필드를 가진 레코드를 매칭하여 두 필드의 값을 각각 `x`와 `y` 이름으로 바인딩한다.

```haskell
showPerson :: { first :: String, last :: String } -> String
showPerson { first: x, last: y } = y <> ", " <> x
```

레코드 패턴은 PureScript 타입 시스템이 제공하는 **행 다형성**이란 재미난 기능을 보여주는 좋은 예이다. 만약 `showPerson` 함수를 타입 시그너처 없이 정의하면 어떻게 될까? 타입을 어떻게 추론할까?
컴파일러가 추론해낸 타입은 우리가 준 타입과는 다르다.

```text
> showPerson { first: x, last: y } = y <> ", " <> x

> :type showPerson
forall r. { first :: String, last :: String | r } -> String
```

여기 등장한 타입 변수 `r`은 무엇인가? PSCi에서 `showPerson`을 테스트해보면 흥미로운 결과를 볼 수 있다.

```text
> showPerson { first: "Phil", last: "Freeman" }
"Freeman, Phil"

> showPerson { first: "Phil", last: "Freeman", location: "Los Angeles" }
"Freeman, Phil"
```

인자로 전달하는 레코드에 필드를 더 추가해도 `showPerson`이 잘 동작한다. 레코드에 `first`와 `last` 필드가 있고 모두 `String` 타입이기만 하다면 이 함수를 적용하는 것이 타입 문제를 일으키지 않는다. 대신 필드가 모자란 경우에는 `showPerson` 함수를 적용시킬 수 없다.

```text
> showPerson { first: "Phil" }

Type of expression lacks required label "last"
```

타입 추론으로 드러난 새로운 타입 시그너처를 읽는 방법은 이렇다. "`String` 타입의 `first`와 `last` 필드, 그리고 **그외 다른 필드들**을 가진 레코드를 입력받아 `String`을 반환한다."

이 함수는 레코드 필드의 **행(row)** `r`에 대해 다형성을 띈다. 그래서 **행 다형성**이라고 부른다.

물론 `showPerson` 함수를 다음처럼 작성할 수도 있다.

```haskell
> showPerson p = p.last <> ", " <> p.first
```

PSCi에서 테스트해보면 추론된 타입이 같을 것이다.

행 다형성은 나중에 **확장 가능한 효과**를 다룰 때 또 보게 될 것이다.

## 중첩 패턴

배열 패턴이나 레코드 패턴은 작은 패턴들을 이용하여 더 큰 패턴을 만든다. 지금까지 본 예제들에서는 배열 패턴이나 레코드 패턴 내부에 단순 패턴만 사용했다. 하지만 패턴은 얼마든지 쌓아나갈 수 있다. 패턴을 다양하게 중첩하면 매우 복잡한 자료 구조에 대한 조건도 쉽게 정의할 수 있다.

아래 코드는 레코드 패턴을 중첩하여 사용한다.

```haskell
type Address = { street :: String, city :: String }

type Person = { name :: String, address :: Address }

livesInLA :: Person -> Boolean
livesInLA { address: { city: "Los Angeles" } } = true
livesInLA _ = false
```

## 명명 패턴

패턴에는 이름을 붙일 수 있다. 중첩 패턴을 사용할 때 해당 패턴에 매칭된 값을 이름에 바인딩한다. 어떤 패턴이든 `@` 문자를 사용하여 이름을 붙일 수 있다.

아래의 예를 보면 길이가 2인 배열을 정렬하는 함수인데, 배열의 요소를 각각 `x`, `y`로 바인딩하면서 추가로 배열 전체를 `arr`이란 이름으로 바인딩하고 있다.

```haskell
sortPair :: Array Int -> Array Int
sortPair arr@[x, y]
  | x <= y = arr
  | otherwise = [y, x]
sortPair arr = arr
```

이렇게 하면 배열이 이미 정렬되어 있는 경우에 새로운 배열을 만들지 않을 수 있다.

> ## 연습 문제
>
> 1. (쉬움) 레코드 패턴을 사용하여 `Person` 타입의 두 사람이 같은 도시에 살고 있는지 확인하는 `sameCity` 함수를 작성해보라.
> 1. (보통) 행 다형성을 고려할 때 `sameCity` 함수의 가장 일반화된 타입은 무엇인가? 위에 정의한 `livesInLA` 함수의 경우는 어떠한가?
> 1. (보통) 배열 리터럴 패턴을 사용하여 인자가 하나뿐인 싱글턴 배열에서 그 단일 요소 값을 추출하는 `fromSingleton` 함수를 작성해보라. 배열이 싱글턴이 아닌 경우 이 함수는 지정된 기본 값을 반환해야 한다. 즉 이 함수의 타입은 `forall a. a -> Array a -> a`이어야 한다.

## `case` 표현식

패턴이 최상위 함수 선언에만 사용되는 것은 아니다. 계산 과정의 중간 값에 대해서도 `case` 표현식을 사용하여 패턴 매칭을 할 수 있다. 익명 함수가 가능하기 때문에 매번 이름을 지정하여 최상위 함수를 만들지 않아도 되는 것처럼 `case` 표현식을 이용하면 패턴을 사용할 수 있기 때문에 패턴 매칭을 사용할 목적 때문이라면 굳이 최상위 함수를 따로 만들 필요는 없다.

`case`를 사용하는 예를 살펴보자. 배열에서 "합이 0인 가장 긴 꼬리(longest zero suffix)"를 찾는 함수이다.

```haskell
import Data.Array.Partial (tail)
import Partial.Unsafe (unsafePartial)

lzs :: Array Int -> Array Int
lzs [] = []
lzs xs = case sum xs of
           0 -> xs
           _ -> lzs (unsafePartial tail xs)
```

다음 처럼 사용할 수 있다.

```text
> lzs [1, 2, 3, 4]
[]

> lzs [1, -1, -2, 3]
[-1, -2, 3]
```

이 함수는 두 가지 형태의 케이스 분석으로 동작한다. 먼저 배열이 비어있다면 계산 결과는 빈 배열 그대로가 되며 이를 반환한다. 비어있지 않다면 `case` 표현식으로 다시 두 가지 경우로 나누어 진행한다. 배열의 합이 0이라면 그 배열 전체를 반환하고, 그렇지 않다면 배열의 꼬리 부분에 대해 재귀적으로 살펴본다.

## 패턴 매칭 실패와 부분 함수

`case` 표현식의 패턴들을 순서대로 적용시켰는데 어떤 패턴도 입력과 맞아떨어지지 않는다면 어떻게 될까? 이러한 경우 `case` 표현식은 **패턴 매칭 실패** 런타임 오류를 발생시킨다.

이러한 실패 동작은 쉽게 살펴볼 수 있다.

```haskell
import Partial.Unsafe (unsafePartial)

partialFunction :: Boolean -> Boolean
partialFunction = unsafePartial \true -> true
```

위 함수는 입력값이 `true`인 경우만 매치된다. 이 파일을 컴파일한 다음 PSCi에서 다른 값을 전달해보면 다음과 같은 런타임 오류를 확인할 수 있다.

```text
> partialFunction false

Failed pattern match
```

모든 조합의 입력에 대해 값을 계산하고 반환하는 함수들을 "완전 함수(total function)"라고 하며, 그렇지 않은 함수들을 "부분 함수(partial function)"라고 한다.

가능하다면 완전 함수로 만드는 것이 더 낫다. 하지만 어떤 함수가 부적절한 입력에 대해 값을 계산해 낼 수 없는 경우라면 `Maybe a` 타입으로 값을 반환하는 편이 더 낫다. 실패하는 경우에 `Nothing`을 반환하면 된다. 값이 있거나 없을 수 있는 상황을 타입으로 잘 드러내는 방법이다.

PureScript 컴파일러는 패턴 매치가 불완전하여 완전 함수가 되지 않는다고 확인되는 경우 컴파일 중에 오류를 보여준다. `unsafePartial` 함수는 이런 컴파일 오류를 잠재울 목적으로 사용할 수 있다. (여러분 스스로 그 부분 함수가 안전함을 보장할 수 있어야 한다.) 위 함수 정의에서 `unsafePartial` 호출 부분을 지운다면 `psc` 컴파일러가 아래의 결과를 보여줄 것이다.

```text
A case expression could not be determined to cover all inputs.
The following additional cases are required to cover all inputs:

  false

```

`false` 값이 어떤 패턴으로도 매치되지 않는다고 알려준다. 일반적으로는 누락된 경우들을 열거하여 보여준다.

위 함수 정의에서 타입 정의 부분마저 제거해보자.

```purescript
partialFunction true = true
```

그리고 PSCi를 이용하여 타입 추론 결과를 보면 아래와 같은 특이한 타입을 알려준다.

```text
> :type partialFunction

Partial => Boolean -> Boolean
```

타입을 표시하면서 사용된 `=>`는 이 책 뒤에서 더 살펴보기로 하고, 일단 여기서는 PureScript가
어떤 형태로든 타입 시스템을 이용하여 부분 함수를 추적하고 있다는 점만 알아두자.
따라서 우리는 부분 함수들이 안전하다고 컴파일러에 명시적으로 알려주어야 한다.

컴파일러는 중복 패턴이 발견되는 경우에 경고를 보여주기도 한다. 순서 상 뒤에 열거된 케이스가 이미 앞의 케이스로 매치되는 경우를 말한다.

```purescript
redundantCase :: Boolean -> Boolean
redundantCase true = true
redundantCase false = false
redundantCase false = false
```

컴파일러는 위 코드의 마지막 케이스가 중복되었다고 콕 집어 알려준다.

```text
Redundant cases have been detected.
The definition has the following redundant cases:

  false
```

**주의**: PSCi는 경고를 보여주지 않기 때문에 이 예제와 같은 경고를 직접 확인하려면 파일에 함수를 정의한 다음 `pulp build` 명령으로 컴파일해야 한다.

## 대수적 자료형

이 절에서는 PureScript 타입 시스템의 기능 중 하나인 **대수적 자료형(Algebraic Data Type, ADT)**를 소개한다. ADT는 패턴 매칭과 깊게 관련되어 있다.

하지만 먼저 ADT의 유용성을 드러낼만한 사례를 살펴보자. 이 장에서 구현하고자 하는 간단한 벡터 그래픽 라이브러리를 구성하는 기본이 될 것이다.

선분, 직사각형, 원, 텍스트와 같은 간단한 도형들을 나타내는 타입을 정의하려고 한다. 객체지향 언어를 이용한다면 아마도 `Shape`라는 인터페이스나 추상 클래스를 정의한 각 도형에 해당하는 구체 클래스를 정의할 것이다.

하지만 이런 접근에는 중대한 결함이 있다. 추상화된 `Shape`를 다루기 위해서는 도형을 가지고 작업하고자 하는 모든 기능들을 미리 식별하여 `Shape` 인터페이스에 정의해야 한다. 그런데 이렇게 하면 나중에 새로운 기능을 추가하고자 할 때 기존 코드를 건드리지 않고 작업하기가 매우 어렵다.

사전에 도형의 종류를 모두 알고 있다면 ADT를 이용하여 이러한 문제를 타입에 안전한 방법으로 해결할 수 있다. 나중에 `Shape`에 새로운 기능을 더하고자 할 때에도 기존 코드를 건드리지 않고 쉽게 추가할 수 있다.

ADT를 이용하여 `Shape`를 정의하면 아래와 같다.

```haskell
data Shape
  = Circle Point Number
  | Rectangle Point Number Number
  | Line Point Point
  | Text Point String
```

`Point` 타입 역시 ADT로 정의할 수 있다.

```haskell
data Point = Point
  { x :: Number
  , y :: Number
  }
```

위의 `Point` 데이터 타입 정의를 보면 몇가지 흥미로운 것들이 보인다.

- ADT 생성자에 전달되는 데이터는 기본 자료형에 국한되지 않는다. 생성자에 레코드, 배열, 심지어 다른 ADT를 전달할 수도 있다.
- ADT가 생성자가 여러 개인 경우에 유용하기는 하지만 생성자가 하나인 경우에도 유용하게 쓰일 수 있다.
- ADT의 생성자는 ADT 이름과 같을 수도 있다. 오히려 매우 일반적이다. 대신 타입 생성자로서의 `Point`와 데이터 생성자로서의 `Point`를 혼동하지 않아야 한다. 이 둘은 서로 다른 이름 공간에 존재한다.

`Shape` 타입은 여러 생성자들의 합으로 정의되고, 각 생성자는 이 타입의 데이터들을 식별하는데 사용된다. 다시 말해 어떤 `Shape` 값은 중심(`Point`)과 반지름(`Number`)으로 정의되는 `Circle`일 수도 있고, `Rectangle`이거나 `Line` 혹은 `Text` 중 하나이다. `Shape` 타입의 값을 생성하는 다른 방법은 없다.

ADT의 정의는 `data` 키워드와 새 타입의 이름, 그리고 필요하다면 타입 인자들을 열거하는 것으로 시작된다. 등호 뒤에 파이프 문자(`|`)로 구분하여 타입 생성자들을 열거한다.

또다른 예는 PureScript 표준 라이브러리에서 찾아보자. 이미 봤던 `Maybe` 타입은 어떤 값이 있거나 없음을 나타내는데 사용된다. `purescript-maybe` 패키지에 아래와 같이 정의되어 있다.

```haskell
data Maybe a = Nothing | Just a
```

타입 파라미터 `a`가 사용되었다. 파이프 문자를 "or" 정도로 읽는다면 위 정의는 거의 영어에 가깝게 읽을 수 있다. "a value of type `Maybe a` is either `Nothing`, or `Just` a value of type `a`".

데이터 생성자는 재귀적 자료형을 정의하는 데 사용할 수도 있다. `a` 타입의 값들에 대한 단일 연결리스트 자료형의 정의는 다음과 같다.

```haskell
data List a = Nil | Cons a (List a)
```

이 정의는 `purescript-lists` 패키지에서 가져온 것이다. 여기서 `Nil` 생성자는 빈 리스트를 나타내고, `Cons` 생성자는 맨 앞 머리 요소와 이어지는 꼬리 리스트로 비어있지 않은 리스트를 생성하기 위해 사용된다. 꼬리 부분을 `List a` 타입으로 지정함으로써 재귀적 자료형이 되었다.

## ADT 사용하기


It is simple enough to use the constructors of an algebraic data type to construct a value: simply apply them like functions, providing arguments corresponding to the data included with the appropriate constructor.

For example, the `Line` constructor defined above required two `Point`s, so to construct a `Shape` using the `Line` constructor, we have to provide two arguments of type `Point`:

```haskell
exampleLine :: Shape
exampleLine = Line p1 p2
  where
    p1 :: Point
    p1 = Point { x: 0.0, y: 0.0 }

    p2 :: Point
    p2 = Point { x: 100.0, y: 50.0 }
```

To construct the points `p1` and `p2`, we apply the `Point` constructor to its single argument, which is a record.

So, constructing values of algebraic data types is simple, but how do we use them? This is where the important connection with pattern matching appears: the only way to consume a value of an algebraic data type is to use a pattern to match its constructor.

Let's see an example. Suppose we want to convert a `Shape` into a `String`. We have to use pattern matching to discover which constructor was used to construct the `Shape`. We can do this as follows:

```haskell
showPoint :: Point -> String
showPoint (Point { x: x, y: y }) =
  "(" <> show x <> ", " <> show y <> ")"

showShape :: Shape -> String
showShape (Circle c r)      = ...
showShape (Rectangle c w h) = ...
showShape (Line start end)  = ...
showShape (Text p text) = ...
```

Each constructor can be used as a pattern, and the arguments to the constructor can themselves be bound using patterns of their own. Consider the first case of `showShape`: if the `Shape` matches the `Circle` constructor, then we bring the arguments of `Circle` (center and radius) into scope using two variable patterns, `c` and `r`. The other cases are similar.

`showPoint` is another example of pattern matching. In this case, there is only a single case, but we use a nested pattern to match the fields of the record contained inside the `Point` constructor.

## Record Puns

The `showPoint` function matches a record inside its argument, binding the `x` and `y` properties to values with the same names. In PureScript, we can simplify this sort of pattern match as follows:

```haskell
showPoint :: Point -> String
showPoint (Point { x, y }) = ...
```

Here, we only specify the names of the properties, and we do not need to specify the names of the values we want to introduce. This is called a _record pun_.

It is also possible to use record puns to _construct_ records. For example, if we have values named `x` and `y` in scope, we can construct a `Point` using `Point { x, y }`:

```haskell
origin :: Point
origin = Point { x, y }
  where
    x = 0.0
    y = 0.0
```

This can be useful for improving readability of code in some circumstances.

X> ## Exercises
X>
X> 1. (Easy) Construct a value of type `Shape` which represents a circle centered at the origin with radius `10.0`.
X> 1. (Medium) Write a function from `Shape`s to `Shape`s, which scales its argument by a factor of `2.0`, center the origin.
X> 1. (Medium) Write a function which extracts the text from a `Shape`. It should return `Maybe String`, and use the `Nothing` constructor if the input is not constructed using `Text`.

## Newtypes

There is an important special case of algebraic data types, called _newtypes_. Newtypes are introduced using the `newtype` keyword instead of the `data` keyword.

Newtypes must define _exactly one_ constructor, and that constructor must take _exactly one_ argument. That is, a newtype gives a new name to an existing type. In fact, the values of a newtype have the same runtime representation as the underlying type. They are, however, distinct from the point of view of the type system. This gives an extra layer of type safety.

As an example, we might want to define newtypes as type-level aliases for `Number`, to ascribe units like pixels and inches:

```haskell
newtype Pixels = Pixels Number
newtype Inches = Inches Number
```

This way, it is impossible to pass a value of type `Pixels` to a function which expects `Inches`, but there is no runtime performance overhead.

Newtypes will become important when we cover _type classes_ in the next chapter, since they allow us to attach different behavior to a type without changing its representation at runtime.

## A Library for Vector Graphics

Let's use the data types we have defined above to create a simple library for using vector graphics.

Define a type synonym for a `Picture` - just an array of `Shape`s:

```haskell
type Picture = Array Shape
```

For debugging purposes, we'll want to be able to turn a `Picture` into something readable. The `showPicture` function lets us do that:

```haskell
showPicture :: Picture -> Array String
showPicture = map showShape
```

Let's try it out. Compile your module with `pulp build` and open PSCi with `pulp psci`:

```text
$ pulp build
$ pulp psci

> import Data.Picture

> :paste
… showPicture
…   [ Line (Point { x: 0.0, y: 0.0 })
…          (Point { x: 1.0, y: 1.0 })
…   ]
… ^D

["Line [start: (0.0, 0.0), end: (1.0, 1.0)]"]
```

## Computing Bounding Rectangles

The example code for this module contains a function `bounds` which computes the smallest bounding rectangle for a `Picture`.

The `Bounds` data type defines a bounding rectangle. It is also defined as an algebraic data type with a single constructor:

```haskell
data Bounds = Bounds
  { top    :: Number
  , left   :: Number
  , bottom :: Number
  , right  :: Number
  }
```

`bounds` uses the `foldl` function from `Data.Foldable` to traverse the array of `Shapes` in a `Picture`, and accumulate the smallest bounding rectangle:

```haskell
bounds :: Picture -> Bounds
bounds = foldl combine emptyBounds
  where
    combine :: Bounds -> Shape -> Bounds
    combine b shape = union (shapeBounds shape) b
```

In the base case, we need to find the smallest bounding rectangle of an empty `Picture`, and the empty bounding rectangle defined by `emptyBounds` suffices.

The accumulating function `combine` is defined in a `where` block. `combine` takes a bounding rectangle computed from `foldl`'s recursive call, and the next `Shape` in the array, and uses the `union` function to compute the union of the two bounding rectangles. The `shapeBounds` function computes the bounds of a single shape using pattern matching.

X> ## Exercises
X>
X> 1. (Medium) Extend the vector graphics library with a new operation `area` which computes the area of a `Shape`. For the purposes of this exercise, the area of a piece of text is assumed to be zero.
X> 1. (Difficult) Extend the `Shape` type with a new data constructor `Clipped`, which clips another `Picture` to a rectangle. Extend the `shapeBounds` function to compute the bounds of a clipped picture. Note that this makes `Shape` into a recursive data type.

## Conclusion

In this chapter, we covered pattern matching, a basic but powerful technique from functional programming. We saw how to use simple patterns as well as array and record patterns to match parts of deep data structures.

This chapter also introduced algebraic data types, which are closely related to pattern matching. We saw how algebraic data types allow concise descriptions of data structures, and provide a modular way to extend data types with new operations.

Finally, we covered _row polymorphism_, a powerful type of abstraction which allows many idiomatic JavaScript functions to be given a type. We will see this idea again later in the book.

In the rest of the book, we will use ADTs and pattern matching extensively, so it will pay dividends to become familiar with them now. Try creating your own algebraic data types and writing functions to consume them using pattern matching.
