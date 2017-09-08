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

ADT의 생성자를 이용하여 값을 만드는 건 정말 간단하다. 일반 함수처럼 필요한 인자들에 적용하기만 하면 된다.

예를 들어 위에서 `Line` 생성자가 `Point` 값 두 개를 받는 것으로 정의되어 있으므로 이 `Line` 생성자로 `Shape` 타입의 값을 만들고자 하면 `Point` 타입의 값 두 개를 인자로 전달해야 한다.

```haskell
exampleLine :: Shape
exampleLine = Line p1 p2
  where
    p1 :: Point
    p1 = Point { x: 0.0, y: 0.0 }

    p2 :: Point
    p2 = Point { x: 100.0, y: 50.0 }
```

`p1`과 `p2` 두 개의 점을 만들기 위해 `Point` 생성자를 이용했고, 각 `Point` 생성자 호출에는 레코드 값 하나를 인자로 전달했다.

ADT 값을 생성하기 쉽다는 것은 알았고, 그럼 이제 그 값들을 어떻게 사용하는지 살펴보자. ADT와 패턴 매칭의 중요한 연결 고리가 등장하게 된다. ADT 값을 사용하기 위한 유일한 방법이 바로 생성자를 이용한 패턴 매칭이다.

예를 살펴보자. `Shape`를 `String`으로 변환하려고 한다. 그러기 위해서는 `Shape`를 생성하는 데 사용한 생성자가 무엇인지 패턴 매칭을 이용하여 찾아내야 한다. 아래처럼 할 수 있다.

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

각 생성자를 패턴으로 사용할 수 있으며 생성자에 전달한 인자들 역시 패턴을 사용하여 이름에 바인딩할 수 있다. `showShape` 함수의 첫 번째 케이스를 보자. 만약 `Shape` 값이 `Circle` 생성자에 매치된다면 `Circle` 생성자의 인자들(중심과 반지름)을 `c`와 `r`이란 이름으로 사용할 수 있다. 다른 케이스들도 마찬가지다.

`showPoint` 함수도 패턴 매칭 사례를 보여준다. 여기서는 케이스가 하나뿐이지만 패턴이 중첩되어 `Point` 생성자 패턴 내부에 레코드 패턴으로 필드 값들을 매치할 수 있다.

## 레코드 이름 재사용

`showPoint` 함수는 레코드 패턴을 매치시키며 `x`와 `y` 필드를 같은 이름으로 바인딩한다. PureScript에서는 이런 식의 패턴 매칭을 간결하게 작성할 수 있다.

```haskell
showPoint :: Point -> String
showPoint (Point { x, y }) = ...
```

레코드 필드의 이름만 지정하고 바인딩할 이름은 따로 지정할 필요가 없다. 이를 **레코드 이름 재사용(record pun)**이라고 부른다.

레코드 이름 재사용은 레코드를 **생성**하는 경우에도 사용할 수 있다. 예를 들어 이미 `x`와 `y`란 이름이 현재 스코프에 있을 때 `Point { x, y }` 형식으로 `Point`를 생성할 수 있다.

```haskell
origin :: Point
origin = Point { x, y }
  where
    x = 0.0
    y = 0.0
```

이 방법을 적절히 사용하면 코드 가독성을 향상시킬 수 있다.

> ## 연습 문제
>
> 1. (쉬움) 중심이 원점이고 반지름이 `10.0`인 원을 나타내는 `Shape` 값을 생성해보라.
> 1. (보통) 원점을 기준으로 하여 `2.0` 배율로 확대하는 함수를 작성해보라. (`Shape`를 인자로 받고 `Shape`를 반환한다.)
> 1. (보통) `Shape`에서 문자열을 추출하는 함수를 작성해보라. 결과 타입을 `Maybe String`으로 하여 만약 `Text` 생성자로 만들어진 값이 아닌 경우에는 `Nothing`을 반환해야 한다.

## 뉴타입

ADT 중에서도 특별하고 중요한 경우로 **뉴타입(newtype)**이라 불리는 것이 있다. 뉴타입은 `newtype` 키워드로 만들어진다.

뉴타입은 생성자가 **오직 하나**뿐이고, 그 생성자의 인자 역사 **오직 하나**뿐이어야 한다. 말하자면 뉴타입은 기존 타입에 새로운 이름을 부여하는 것과 같다. 실제로 뉴타입으로 만들어진 값은 런타임에 기존 타입과 완전히 똑같이 표현된다. 뉴타입은 타입 시스템 관점에서만 다르게 인식될 뿐이다. 타입 안전성을 위한 추가적 계층을 제공하기 위한 것이다.

예를 들어 `Number`에 대한 타입 수준의 별칭을 뉴타입으로 정의할 수 있다. 픽셀이나 인치같은 단위를 그렇게 만들 수 있다.

```haskell
newtype Pixels = Pixels Number
newtype Inches = Inches Number
```

이제 `Pixels` 타입의 값을 `Inches` 타입을 인자로 받는 함수에 전달하는 것은 불가능하다. 하지만 `Pixels` 타입의 값은 여전히 `Number`와 똑같이 표현되므로 런타임에 발생하는 오버헤드가 전혀 없다.

뉴타입은 다음 장에서 **타입 클래스(type class)**를 사용하게 될 때 중요성이 더 부각된다. 런타임에 어떤 부담도 주지 않으면서 타입에 다른 동작을 덧붙일 수 있기 때문이다.

## 벡터 그래픽 라이브러리

이제 앞에서 정의한 타입들을 이용하여 벡터 그래픽 라이브러리를 만들어보자.

`Shape`의 배열에는 `Picture`라는 타입 별칭을 부여한다.

```haskell
type Picture = Array Shape
```

디버깅을 쉽게 하기 위해 `Picture`를 문자열로 출력해주는 `showPicture` 함수를 정의하자.

```haskell
showPicture :: Picture -> Array String
showPicture = map showShape
```

`pulp build` 명령으로 모듈을 컴파일한 다음 `pulp repl`로 PSCi를 띄워 작업한 내용을 확인해보자.

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

## 감싸는 사각형 계산하기

이 모듈의 예제 코드에 포함된 `bounds` 함수는 `Picture`를 입력으로 받아 전체를 감싸는 가장 작은 사각형을 계산한다.

`Bounds` 타입은 이러한 감싸는 사각형을 나타내며, 생성자가 하나인 ADT로 정의되어 있다.

```haskell
data Bounds = Bounds
  { top    :: Number
  , left   :: Number
  , bottom :: Number
  , right  :: Number
  }
```

`bounds`는 `Data.Foldable` 모듈의 `foldl` 함수를 이용하여 `Shape` 배열(`Picture`)의 값들을 하나씩 처리함으로써 전체를 감싸는 가장 작은 사각형을 찾아낸다.

```haskell
bounds :: Picture -> Bounds
bounds = foldl combine emptyBounds
  where
    combine :: Bounds -> Shape -> Bounds
    combine b shape = union (shapeBounds shape) b
```

`fold` 함수의 기본 케이스에 해당하는 `emptyBounds`는 비어있는 감싸기 사각형을 나타내기 위해 정의된 것이다. `Picture`가 비어있는 배열인 경우, 이를 감싸는 사각형으로 사용된다.

누적 함수로 사용된 `combine`은 `where` 절에 정의되어 있다. `combine` 함수는 `foldl`의 재귀 호출로 계산된 감싸기 사각형과 배열의 다음 요소 `Shape`를 인자로 받는다. 먼저 `Shape`의 감싸기 사각형을 `shapeBounds`로 계산한 다음 두 개의 감싸기 사각형을 `union`으로 합친다. `shapeBounds` 함수는 각 `Shape`을 감싸는 사각형을 계산하기 위해 패턴 매칭을 사용한다.

> ## 연습 문제
>
> 1. (보통) 벡터 그래픽 라이브러리에 `area` 함수를 추가해보라. 이 함수는 각 `Shape`의 면적을 계산한다. 연습 문제에서는 `Text` 모양에 대해 면적을 0이라고 가정한다.
> 1. (어려움) `Shape` 타입에 `Clipped` 생성자를 추가해보라. `Clipped` 생성자는 `Picture`를 특정 사각형으로 클리핑(사각형 외부를 잘라냄)한 도형을 나타낸다. `shapeBounds` 함수를 수정하여 `Clipped` 값을 처리할 수 있게 하라. 이 생성자를 통해 `Shape`이 재귀 자료 구조가 되었다.

## 결론

이번 장에서 패턴 매칭을 다뤘다. 패턴 매칭은 함수형 프로그래밍의 기본 기능이면서도 매우 강력하다. 단순한 패턴들과 더불어 배열이나 레코드가 중첩된 복잡한 자료 구조에 대해서도 살펴봤다.

패턴 매칭과 밀접하게 관련되어 있는 대수적 자료형도 소개했다. ADT를 이용하여 자료형을 간결하게 정의할 수 있고 또 여기에 새로운 연산 함수를 쉽게 추가할 수 있었다.

**행 다형성**도 다루었다. 행 다형성을 이용하면 인자들의 타입에 대해 느슨한 JavaScript 함수들에도 쉽게 타입을 부여할 수 있다. 이 책 뒤에서 또 보게 될 것이다.

앞으로 보게 될 코드에는 ADT와 패턴 매칭이 많이 사용되기 때문에 지금 익혀둔다면 여러모로 도움이 될 것이다. 여러분 스스로의 ADT를 정의하고 관련 함수들을 정의하면서 패턴 매칭을 사용해보길 바란다.

