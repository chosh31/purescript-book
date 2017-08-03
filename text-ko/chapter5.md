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

Patterns do not only appear in top-level function declarations. It is possible to use patterns to match on an intermediate value in a computation, using a `case` expression. Case expressions provide a similar type of utility to anonymous functions: it is not always desirable to give a name to a function, and a `case` expression allows us to avoid naming a function just because we want to use a pattern.

Here is an example. This function computes "longest zero suffix" of an array (the longest suffix which sums to zero):

```haskell
import Data.Array.Partial (tail)
import Partial.Unsafe (unsafePartial)

lzs :: Array Int -> Array Int
lzs [] = []
lzs xs = case sum xs of
           0 -> xs
           _ -> lzs (unsafePartial tail xs)
```

For example:

```text
> lzs [1, 2, 3, 4]
[]

> lzs [1, -1, -2, 3]
[-1, -2, 3]
```

This function works by case analysis. If the array is empty, our only option is to return an empty array. If the array is non-empty, we first use a `case` expression to split into two cases. If the sum of the array is zero, we return the whole array. If not, we recurse on the tail of the array.

## Pattern Match Failures and Partial Functions

If patterns in a case expression are tried in order, then what happens in the case when none of the patterns in a case alternatives match their inputs? In this case, the case expression will fail at runtime with a _pattern match failure_.

We can see this behavior with a simple example:

```haskell
import Partial.Unsafe (unsafePartial)

partialFunction :: Boolean -> Boolean
partialFunction = unsafePartial \true -> true
```

This function contains only a single case, which only matches a single input, `true`. If we compile this file, and test in PSCi with any other argument, we will see an error at runtime:

```text
> partialFunction false

Failed pattern match
```

Functions which return a value for any combination of inputs are called _total_ functions, and functions which do not are called _partial_.

It is generally considered better to define total functions where possible. If it is known that a function does not return a result for some valid set of inputs, it is usually better to return a value with type `Maybe a` for some `a`, using `Nothing` to indicate failure. This way, the presence or absence of a value can be indicated in a type-safe way.

The PureScript compiler will generate an error if it can detect that your function is not total due to an incomplete pattern match. The `unsafePartial` function can be used to silence these errors (if you are sure that your partial function is safe!) If we removed the call to the `unsafePartial` function above, then `psc` would generate the following error:

```text
A case expression could not be determined to cover all inputs.
The following additional cases are required to cover all inputs:

  false
```

This tells us that the value `false` is not matched by any pattern. In general, these warnings might include multiple unmatched cases.

If we also omit the type signature above:

```purescript
partialFunction true = true
```

then PSCi infers a curious type:

```text
> :type partialFunction

Partial => Boolean -> Boolean
```

We will see more types which involve the `=>` symbol later on in the book (they are related to _type classes_), but for now, it suffices to observe that PureScript keeps track of partial functions using the type system, and that we must explicitly tell the type checker when they are safe.

The compiler will also generate a warning in certain cases when it can detect that cases are _redundant_ (that is, a case only matches values which would have been matched by a prior case):

```purescript
redundantCase :: Boolean -> Boolean
redundantCase true = true
redundantCase false = false
redundantCase false = false
```

In this case, the last case is correctly identified as redundant:

```text
Redundant cases have been detected.
The definition has the following redundant cases:

  false
```

_Note_: PSCi does not show warnings, so to reproduce this example, you will need to
save this function as a file and compile it using `pulp build`.

## Algebraic Data Types

This section will introduce a feature of the PureScript type system called _Algebraic Data Types_ (or _ADTs_), which are fundamentally related to pattern matching.

However, we'll first consider a motivating example, which will provide the basis of a solution to this chapter's problem of implementing a simple vector graphics library.

Suppose we wanted to define a type to represent some simple shape types: lines, rectangles, circles, text, etc. In an object oriented language, we would probably define an interface or abstract class `Shape`, and one concrete subclass for each type of shape that we wanted to be able to work with.

However, this approach has one major drawback: to work with `Shape`s abstractly, it is necessary to identify all of the operations one might wish to perform, and to define them on the `Shape` interface. It becomes difficult to add new operations without breaking modularity.

Algebraic data types provide a type-safe way to solve this sort of problem, if the set of shapes is known in advance. It is possible to define new operations on `Shape` in a modular way, and still maintain type-safety.

Here is how `Shape` might be represented as an algebraic data type:

```haskell
data Shape
  = Circle Point Number
  | Rectangle Point Number Number
  | Line Point Point
  | Text Point String
```

The `Point` type might also be defined as an algebraic data type, as follows:

```haskell
data Point = Point
  { x :: Number
  , y :: Number
  }
```

The `Point` data type illustrates some interesting points:

- The data carried by an ADT's constructors doesn't have to be restricted to primitive types: constructors can include records, arrays, or even other ADTs.
- Even though ADTs are useful for describing data with multiple constructors, they can also be useful when there is only a single constructor.
- The constructors of an algebraic data type might have the same name as the ADT itself. This is quite common, and it is important not to confuse the `Point` _type constructor_ with the `Point` _data constructor_ - they live in different namespaces.

This declaration defines `Shape` as a sum of different constructors, and for each constructor identifies the data that is included. A `Shape` is either a `Circle` which contains a center `Point` and a radius (a number), or a `Rectangle`, or a `Line`, or `Text`. There are no other ways to construct a value of type `Shape`.

An algebraic data type is introduced using the `data` keyword, followed by the name of the new type and any type arguments. The type's constructors are defined after the equals symbol, and are separated by pipe characters (`|`).

Let's see another example from PureScript's standard libraries. We saw the `Maybe` type, which is used to to define optional values, earlier in the book. Here is it's definition from the `purescript-maybe` package:

```haskell
data Maybe a = Nothing | Just a
```

This example demonstrates the use of a type parameter `a`. Reading the pipe character as the word "or", its definition almost reads like English: "a value of type `Maybe a` is either `Nothing`, or `Just` a value of type `a`".

Data constructors can also be used to define recursive data structures. Here is one more example, defining a data type of singly-linked lists of elements of type `a`:

```haskell
data List a = Nil | Cons a (List a)
```

This example is taken from the `purescript-lists` package. Here, the `Nil` constructor represents an empty list, and `Cons` is used to create non-empty lists from a head element and a tail. Notice how the tail is defined using the data type `List a`, making this a recursive data type.

## Using ADTs

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
