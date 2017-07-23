# 함수와 레코드

## 이 장의 목표

이 장에서는 PureScript 프로그램을 구성하는 핵심 요소인 함수와 레코드를 살펴본다. 그리고 PureScript 프로그램을 어떻게 구조화하는지 타입이 어떤 식으로 개발을 도와주는지도 살펴본다.

연락처 목록을 관리하는 간단한 주소록 애플리케이션을 만들어 본다. 이 과정에서 PureScript 문법에 관한 몇가지 개념들도 소개할 것이다.

우리가 만들 애플리케이션의 사용자 인터페이스는 일단 인터랙티브 모드인 PSCi를 이용한다. 하지만 JavaScript로 완전한 인터페이스를 덧붙일 수도 있다.
사실 이 책 뒤에서 입력 검증이나 저장하기/불러오기 같은 기능을 구현할 때 JavaScript 프론트엔드를 붙여볼 것이다.

## 프로젝트 설정

이 장의 소스코드는 `src/Data/AddressBook.purs` 파일에 작업한다. 이 파일은 아래처럼 모듈 선언과 임포트 문으로 시작한다.

```haskell
module Data.AddressBook where

import Prelude

import Control.Plus (empty)
import Data.List (List(..), filter, head)
import Data.Maybe (Maybe)
```

임포트한 모듈들을 살펴보자.

- `Control.Plus` 모듈에는 `empty` 값이 정의되어 있다.
- `Data.List` 모듈은 Bower를 이용하여 `purescript-lists` 패키지를 설치해야 사용할 수 있다. 이 모듈에는 연결 리스트와 관련된 함수들이 정의되어 있다.
- `Data.Maybe` 모듈에는 값이 있거나 없을 수 있음을 나타내는 데이터 타입과 이에 관련된 함수들이 정의되어 있다.

임포트를 잘 보면 모듈 뒤에 괄호로 임포트하는 것들을 표시하고 있다. 임포트 내용을 구체적으로 표시하는 것은 좋은 습관이며 임포트 충돌을 피하는 방법이기도 하다.

이 책의 소스 코드 저장소에서 코드를 복제했다면 Pulp를 이용하여 아래처럼 프로젝트를 빌드할 수 있다.

```text
$ cd chapter3
$ bower update
$ pulp build
```

## 기본 타입들

PureScript는 JavaScript의 기본 타입인 number, string, boolean에 대응하는 세 가지 기본 타입들을 정의하고 있다. 이 타입들은 `Prim` 모듈에 정의되어 있는데, 따로 임포트하지 않아도 모든 모듈에 자동으로 임포트된다. 각 타입은 `Number`, `String`, `Boolean`이며, PSCi에서 `:type` 명령으로 확인해 볼 수 있다.

```text
$ pulp psci

> :type 1.0
Number

> :type "test"
String

> :type true
Boolean
```

PureScript는 정수, 문자, 배열, 레코드, 함수와 같은 다른 타입들도 기본으로 정의하고 있다.

정수는 부동 소숫점 실수를 나타내는 `Number` 타입과 달리 소숫점이 없다.

```text
> :type 1
Int
```

문자를 나타낼 때는 작은따옴표를 사용한다. (문자열은 큰따옴표를 사용한다.)

```text
> :type 'a'
Char
```

배열은 JavaScript 배열에 대응한다. 하지만 JavaScript 배열과는 달리 PureScript 배열은 요소들이 모두 같은 타입이어야 한다.

```text
> :type [1, 2, 3]
Array Int

> :type [true, false]
Array Boolean

> :type [1, false]
Could not match type Int with Boolean.
```

앞 예제 마지막에 타입 체커가 에러를 표시하는 것을 볼 수 있다. 위 에러는 배열 요소들의 타입을 통일하려는 시도가 실패했음을 의미한다.

레코드는 JavaScript 오브젝트에 대응한다. 레코드를 표시할 때는 JavaScript의 오브젝트를 표시할 때와 같은 문법을 사용한다.

```text
> author = { name: "Phil", interests: ["Functional Programming", "JavaScript"] }

> :type author
{ name :: String
, interests :: Array String
}
```

타입을 보면 우리가 정의한 객체가 두 개의 필드를 가지고 있음을 보여준다. `name` 필드는 `String` 타입이고, `interests` 필드는 `Array String` 타입이다. (`String` 타입 요소들의 배열을 의미한다.)

레코드의 필드는 점(.) 뒤에 필드 이름을 붙여 접근할 수 있다. 

```text
> author.name
"Phil"

> author.interests
["Functional Programming","JavaScript"]
```

PureScript의 함수는 JavaScript의 함수에 대응한다. PureScript의 표준 라이브러리에 포함된 함수들이 좋은 예제가 될 것이다. 그 중 몇 개를 이 장에서 살펴볼 것이다.

```text
> import Prelude
> :type flip
forall a b c. (a -> b -> c) -> b -> a -> c

> :type const
forall a b. a -> b -> a
```

파일 최상위 수준에 함수를 정의할 때는 등호(=) 앞에 인자들을 표시하면 된다.

```haskell
add :: Int -> Int -> Int
add x y = x + y
```

함수는 인라인으로 정의할 수도 있는데, 이 때는 백슬래시(\\) 뒤에 공백으로 구분된 인자 목록을 추가한다.
PSCi에서 여러 줄에 걸친 선언을 입력하려면 `:paste` 명령으로 "붙여넣기 모드"를 활성화하면 된다. 붙여넣기 모드에서 빠져나올 때는 **Control-D**를 입력한다.

```text
> :paste
… add :: Int -> Int -> Int
… add = \x y -> x + y
… ^D
```

위처럼 정의한 함수를 인자들에 적용(apply)할 때는 함수 이름 뒤에 공백으로 구분하여 인자 두 개를 나열하면 된다.

```text
> add 10 20
30
```

## 한정 타입

앞 절에서 Prelude에 정의된 함수 몇 개의 타입을 살펴봤다. 그 중 하나인 `flip` 함수의 타입은 다음과 같다.

```text
> :type flip
forall a b c. (a -> b -> c) -> b -> a -> c
```

타입에 나타난 `forall` 키워드는 `flip` 함수가 **universally quantified type**을 가진다는 걸 나타낸다. 말이 조금 어려운데 쉽게 말하자면 `a`, `b`, `c`를 어떤 임의의 타입으로 바꾸어도 `flip`이 문제없이 동작한다는 의미다.

예를 들어 `a` 대신 `Int` 타입을, `b` 대신 `String` 타입을, `c` 대신 `String` 타입을 사용할 수 있다. 이렇게 구체적인 타입을 선택하여 `flip`을 **구체화**하면 타입이 아래와 같다.

```text
(Int -> String -> String) -> String -> Int -> String
```

타입 인자로 일반화된 타입을 구체화할 때 이를 따로 표시할 필요가 없다. 구체화는 자동으로 이뤄진다. 예를 들어 `flip`이 이미 위처럼 구체화되었다고 보고 그냥 사용할 수 있다는 얘기다.

```text
> flip (\n s -> show n <> s) "Ten" 10

"10Ten"
```

`a`, `b`, `c` 대신 어떤 타입이든 선택할 수 있다고는 하지만 타입이 일관되게 적용되어야 한다. 예를 들어 `flip` 함수의 첫 인자로 전달하는 함수의 인자 타입이 `flip` 함수의 나머지 인자 타입과 맞아야 한다. 위 코드에서 두 번째 인자로 문자열 `"Ten"`을, 세 번째 인자로 수 `10`을 전달한 것도 이 때문이다. 인자 순서가 달랐다면 문제가 된다.

```text
> flip (\n s -> show n <> s) 10 "Ten"

Could not match type Int with type String
```

## 들여쓰기에 관하여

PureScript 코드는 Haskell과 마찬가지로 들여쓰기에 의미가 있다. C 계열 언어들이 중괄호를 사용하여 코드 블럭을 구분하듯이 PureScript는 공백을 사용하여 코드 구역을 구분한다.

만약 선언이 여러 줄에 걸쳐이다면 첫 줄을 뺀 나머지 줄은 첫 줄보다 안쪽으로 들여쓰기 되어야 한다.

따라서 다음은 올바른 PureScript 코드이다.

```haskell
add x y z = x +
  y + z
```

하지만 다음은 올바르지 않다.

```haskell
add x y z = x +
y + z
```

두 번째 경우는 PureScript 컴파일러가 두 줄을 별도의 선언으로 처리한다.

일반적으로 같은 수준에서 정의되는 것들은 같은 수준으로 들여쓰기되어야 한다. 예를 들어 PSCi에서 여러 값을 한 번에 선언할 때는 같은 깊이로 들여쓰기되어야 한다. 다음은 바른 코드다.

```text
> :paste
… x = 1
… y = 2
… ^D
```

하지만 다음은 올바르지 않다.

```text
> :paste
… x = 1
…  y = 2
… ^D
```

PureScript 키워드 중에는 `where`, `of`, `let` 처럼 여러 선언은 추가할 수 있는 새로운 코드 블록을 유발하는 것들이 있다. 이 경우도 선언들이 더 깊게 들여쓰기 되어야 한다.

```haskell
example x y z = foo + bar
  where
    foo = x * y
    bar = y * z
```

위에서 `foo`와 `bar` 선언은 `example` 선언보다 더 깊게 들여쓰기 되어 있다.

이 규칙에 예외가 있는데, 바로 소스 파일 맨 위에 있는 `module` 선언 뒤의 붙은 `where` 키워드의 경우다.

## 타입 직접 정의하기

PureScript를 이용하여 새로운 문제를 해결하기 위한 첫 단계는 바로 우리가 작업할 값들을 표현할 타입을 정의하는 것이다. 먼저 주소록에 사용할 레코드 타입을 정의해보자.

```haskell
type Entry =
  { firstName :: String
  , lastName  :: String
  , address   :: Address
  }
```

위 코드는 `Entry`라고 하는 **타입 별명**을 정의한다. 이제 `Entry`라는 타입은 등호 오른편에 있는 레코드 타입과 동등하다. 이 레코드 타입은 `firstName`, `lastName`, `address` 이렇게 세 개의 필드를 가진다. 앞의 두 필드는 `String` 타입이고, `address` 필드는 `Address` 타입이다. 아래는 `Address` 타입의 정의이다.

```haskell
type Address =
  { street :: String
  , city   :: String
  , state  :: String
  }
```

레코드의 필드에 다른 레코드가 올 수 있다는 점을 알 수 있다.

이제 주소록 전체를 나타내기 위한 세 번째 타입 별명을 정의할 것이다. 주소록은 단순히 연결 리스트로 표현할 수 있다.

```haskell
type AddressBook = List Entry
```

`List Entry`는 **배열** 자료 구조를 의미하는 `Array Entry`와는 다른 타입이다.

## 타입 생성자와 카인드(Kind)

`List`는 **타입 생성자**의 하나다. 어떤 값도 `List`라는 타입을 바로 가질 수는 없다. 대신 타입 `a`가 있다고 했을 때 `List a`라는 타입은 말이 된다. 말하자면 `List`는 **타입 인자**로서 `a`를 취하고, 그 결과로 `List a`라는 새로운 타입을 **생성**하는 셈이다.

함수 적용과 마찬가지로 타입 생성자를 다른 타입에 적용할 때는 타입 생성자에 다른 타입들을 나란히 연결하면 된다. `List Entry`는 `List` 타입 생성자를 `Entry` 타입에 적용시킨 것이다. 그 결과는 엔트리 항목들의 리스트를 나타낸다.

어떤 값을 `List` 타입으로 잘못 선언하려하면 (타입을 직접 지정하려면 `::` 타입 지정 연산자를 이용한다.) 컴파일러는 다음과 같이 처음 보는 에러를 뱉어낼 것이다.

```text
> import Data.List
> Nil :: List
In a type-annotated expression x :: t, the type t must have kind Type
```

이는 **카인드 에러**이다. 여러가지 값(value)을 **타입(type)**으로 구분지을 수 있는 것과 마찬가지로, 타입들은 **카인드(kind)**로 구분지을 수 있다. 값에 타입을 잘못 지정했을 때 **타입 에러**를 유발하듯이 타입에 카인드를 잘못 사용했을 때는 **카인드 에러**가 발생한다.

`Type`이라고 하는 특별한 카인드가 있다. 이것이 `Number`나 `String`과 같은 모든 타입들을 나타낸다.

타입 생성자는 다른 카인드에 속한다. 예를 들어 `Type -> Type` 카인드는 타입을 타입으로 매핑하는 함수를 의미한다. `List`가 여기에 해당한다. 이제 앞에서 본 카인드 에러가 어떻게 나온 것인지 설명할 수 있다. 모든 값들은 `Type` 카인드의 타입이 지정되어야 한다. 그런데 `List`의 카인드는 `Type -> Type`이기 때문에 맞지 않다. 그래서 위와 같은 카인드 에러가 나온 것이다.

타입이나 타입 생성자의 카인드를 알고 싶을 때는 PSCi에서 `:kind` 명령을 사용하면 된다.

```text
> :kind Number
Type

> import Data.List
> :kind List
Type -> Type

> :kind List String
Type
```

PureScript의 **카인드 시스템**은 좀더 흥미로운 카인드도 지원한다. 이 책 뒤에서 살펴볼 것이다.

## 주소록 항목 보여주기

주소록 항목을 문자열로 보여주는 함수를 하나 작성해 보자.
이번에는 먼저 함수의 타입부터 선언할 것이다. 타입을 꼭 선언할 필요는 없지만 이렇게 하는 것은 좋은 습관이다. 타입 자체로 어느 정도 문서 역할도 하기 때문이다. 사실 PureScript 컴파일러는 최상위 수준의 선언들 타입을 지정하지 않으면 결고 메시지를 보여준다. 타입 선언을 추가하려면 함수 이름 뒤에 `::`와 함께 타입을 적어주면 된다.

```haskell
showEntry :: Entry -> String
```

위 타입 시그너처를 보면 `showEntry`라는 함수가 `Entry` 하나를 인자로 받고 결과로는 `String`를 반환한다는 것을 알 수 있다. 아래는 `showEntry` 함수의 정의이다.

```haskell
showEntry entry = entry.lastName <> ", " <>
                  entry.firstName <> ": " <>
                  showAddress entry.address
```

이 함수는 주소록 항목을 나타내는 `Entry` 타입의 세 필드를 하나의 문자열로 이어붙인다. 이때 `address` 필드를 `String`으로 변환하기 위해 `showAddress` 함수를 이용한다. `showAddress` 함수도 비슷하게 정의할 수 수 있다.

```haskell
showAddress :: Address -> String
showAddress addr = addr.street <> ", " <>
                   addr.city <> ", " <>
                   addr.state
```

함수 정의는 이름으로 시작하고, 그 뒤에 인자들의 이름을 나열한다. 함수의 결과는 등호 기호 뒤에 나타난다. 필드를 접근할 때엔 점(.) 다음에 필드 이름을 붙인다. PureScript에서 문자열끼리 이어붙일 때는 다이아몬드 연산자(`<>`)를 사용한다.

## 일찍 자주 테스트하라

PSCi 인터랙티브 모드는 여러분이 더 빠르게 프로토타입을 만들고 즉각적인 피드백을 얻을 수 있게 도와준다. 조금 전에 구현한 두 함수가 기대한대로 동작하는지 확인해보자.

먼저 우리가 작성한 프로그램을 빌드한다.

```text
$ pulp build
```

이제 PSCi를 시작하고 `import` 명령으로 우리가 만든 모듈을 임포트한다.

```text
$ pulp psci

> import Data.AddressBook
```

레코드 리터럴을 이용하여 주소 하나를 생성한다. 이건 마치 JavaScript에서 익명의 객체 하나를 만드는 것과 비슷하게 보인다. 이 항목에 이름을 부여한다.

```text
> address = { street: "123 Fake St.", city: "Faketown", state: "CA" }
```

이제 우리가 작성한 함수를 적용해보자.

```text
> showAddress address

"123 Fake St., Faketown, CA"
```

`showEntry` 함수를 테스트하기 위해 위 주소를 포함하는 주소록 엔트리 하나를 만들고 여기에 함수를 적용해보자.

```text
> entry = { firstName: "John", lastName: "Smith", address: address }
> showEntry entry

"Smith, John: 123 Fake St., Faketown, CA"
```

## 주소록 만들기

이제 유틸리티 함수를 몇 개 더 작성해 보자. 주소록에 항목이 하나도 없는 빈 상태를 나타내는 값이 필요하다.
이는 빈 리스트로 나타낸다.

```haskell
emptyBook :: AddressBook
emptyBook = empty
```

기존 주소록에 새로운 항목을 추가하는 함수도 필요하다.
이 함수를 `insertEntry`라고 부르자. 먼저 타입을 따져보자.

```haskell
insertEntry :: Entry -> AddressBook -> AddressBook
```

이 타입 시그너처를 보면 `insertEntry` 함수가 첫 인자로 `Entry`를, 두 번째 인자로 `AddressBook`을 받으며, 결과로는 새로운 `AddressBook`을 반환한다는 것을 알 수 있다.

우리는 기존 주소록을 바로 수정하지 않는다. 그 대신 새로운 주소록을 반환한다. 반환하는 새 주소록이 기존 항목들과 함께 새 항목을 포함한다. 이렇게 해서 `AddressBook`은 **불변 자료 구조**가 된다. PureScript에서 불변 자료 구조는 매우 중요한 개념이다. 자료가 변경될 수 있음은 코드의 부작용이며, 이로 인해 코드의 동작을 효과적으로 추론하기 어려워진다. 그래서 우리는 가급적 순수 함수와 불변 자료 구조를 우선시 한다.

`insertEntry` 함수를 구현하려면 `Data.List`의 `Cons`를 사용해야 한다.
PSCi에서 `:type` 명령을 사용하여 `Cons`의 타입을 살펴보자.

```text
$ pulp psci

> import Data.List
> :type Cons

forall a. a -> List a -> List a
```

타입 시그너처를 보면 `Cons`가 어떤 임의의 타입 `a`의 값과 그 타입의 요소들로 된 리스트를 두 인자로 받아서 같은 타입의 새로운 리스트를 반환한다는 것을 알 수 있다. 여기서 `a`를 `Entry`라는 타입으로 구체화해 보자.

```haskell
Entry -> List Entry -> List Entry
```

여기서 `List Entry`는 `AddressBook`과 같다. 따라서 위 타입은 아래와 동등하다.

```haskell
Entry -> AddressBook -> AddressBook
```

우리가 선언한 타입과 일치한다. 따라서 단순히 `Cons`만 적용하면 새로운 `AddressBook`을 얻을 수 있다. 우리가 원하던 바로 그것이다.

아래는 `insertEntry`의 구현이다.

```haskell
insertEntry entry book = Cons entry book
```

등호의 왼쪽으로부터 `entry`와 `book`이라는 두 인자가 스코프에 추가되며,
`Cons` 함수에 인자들을 적용하여 결과를 만들어낸다.

## 커리된 함수

PureScript에서 함수는 모두 인자를 하나만 가진다. 앞에서 본 `insertEntry` 함수는 마치 인자를 두 개 가지는 것 같지만 실제로는 단지 **커리(curry)된 함수**일 뿐이다.

`insertEntry` 함수의 타입에 사용된 `->` 연산자는 오른쪽 우선으로 결합한다. 즉 컴파일러가 봤을 때 이 함수의 타입은 아래처럼 읽힌다는 얘기다.

```haskell
Entry -> (AddressBook -> AddressBook)
```

다시 말해, `insertEntry` 함수는 함수를 반환하는 함수다. `Entry` 타입의 인자를 하나 받아서 새로운 함수를 반환한다. 반환되는 결과 함수는 다시 `AddressBook` 타입의 인자를 하나 받아서 새로운 `AddressBook`을 반환한다.

그렇다면 우리는 `insertEntry` 함수에 첫 번째 인자만 전달하여 **부분 적용**도 할 수 있다는 얘기다. PSCi에서 인자 하나만 적용했을 때의 타입을 알아보자.

```text
> :type insertEntry entry

AddressBook -> AddressBook
```

에상한대로 결과 타입은 함수이다. 이제 우리는 그 결과 함수를 두 번째 인자에 적용할 수 있다.

```text
> :type (insertEntry entry) emptyBook
AddressBook
```

여기서 괄호는 불필요하다. 위 코드는 아래와 똑같다.

```text
> :type insertEntry example emptyBook
AddressBook
```

함수 적용은 왼쪽 우선 결합이기 때문이다. 이러한 문법 특성은 인자가 두 개 이상 필요할 때에도 단순히 인자를 하나씩 나열하기만 해도 문제없이 동작하는 이유이기도 하다.

이제부터 이 책에서 이야기하는 "인자 두 개를 갖는 함수"는 사실 커리된 함수를 의미한다. 즉 인자를 하나 받아서 또다른 함수를 반환하는 함수라는 얘기다.

`insertEntry` 정의를 다시 살펴보자.


```haskell
insertEntry :: Entry -> AddressBook -> AddressBook
insertEntry entry book = Cons entry book
```

등호 오른쪽에 괄호를 억지로 나타내보면 `(Cons entry) book`이 된다.
즉 `insertEntry entry`의 결과인 함수는 인자(`book`)를 받아서 그대로 다시 `(Cons entry)` 함수에 전달한다. 하지만 만약 두 함수가 모든 입력에 대해 서로 같은 값을 반환한다면 사실 두 함수는 같은 함수이다. 따라서 그대로 전달하기만 했던 `book` 인자는 등호 양쪽에서 제거할 수 있다.

```haskell
insertEntry :: Entry -> AddressBook -> AddressBook
insertEntry entry = Cons entry
```

하지만 이제는 같은 논리에 따라 `entry`를 양쪽에서 제거할 수 있다.

```haskell
insertEntry :: Entry -> AddressBook -> AddressBook
insertEntry = Cons
```

이러한 과정을 **에타(eta) 변환**이라고 한다. 이 방법을 다른 몇가지 방법들과 함께 적용하면 함수를 **포인트 프리 형식**으로 재작성할 수 있다. 포인트 프리 형식(point-free form)은 인자를 직접 참조하지 않으면서 함수를 정의하는 형식을 말한다.

`insertEntry`의 경우 **에타 변환**의 결과로 매우 간결한 정의를 얻을 수 있었다. "`insertEntry`는 리스트에서의 `Cons`이다." 하지만 포인트 프리 형식이 일반적으로 더 낫다고 얘기하는 것은 논란의 여지가 있다.

## 주소록 검색하기

미니멀리즘 주소록 애플리케이션에 마지막으로 추가할 함수는 이름으로 검색하여 일치하는 항목을 반환하는 함수이다. 이 함수는 작은 함수들을 합성하여 프로그램을 완성해 나간다는 함수형 프로그래밍의 핵심 아이디어를 잘 보여줄 것이다.

어떻게 구현할까? 우선 성과 이름이 일치하는 항목들을 필터링한다. 그런 다음 필터링 결과 리스트의 머리, 즉 첫 번째 항목을 반환한다.

구현에 대한 개략적 아이디어로부터 함수의 타입을 계산해 낼 수 있다. 우선 PSCi를 열어서 `filter`와 `head` 함수의 타입을 알아보자.

```text
$ pulp psci

> import Data.List
> :type filter
forall a. (a -> Boolean) -> List a -> List a

> :type head
forall a. List a -> Maybe a
```

타입이 말하는 바를 제대로 이해하기 위해 하나하나 떼어 살펴보자.

`filter`는 인자가 두 개인 커리된 함수이다. 첫 번째 인자는 리스트 요소를 입력받아 `Boolean` 값을 결과로 반환하는 함수이다. 두 번째 인자는 리스트이다. 결과로 반환하는 값은 또다른 리스트이다.

`head`는 리스트 하나를 인자로 받는다. 반환값의 타입은 아직 보지 못한 `Maybe a`라는 타입이다. `Maybe a`는 `a` 타입의 값이 있거나 혹은 없을 수도 있음을 나타낸다. JavaScript나 다른 언어들이 `null`을 사용하여 값이 없음을 나타내는 것과 비슷하지만 타입 안정성을 제공하는 방법이다. 나중에 더 자세히 살펴볼 것이다.

`filter`와 `head`는 universally quantified type이어서 PureScript 컴파일러가 쓰임새에 따라 구체화한다. 우리 경우는 다음과 같다.

```haskell
filter :: (Entry -> Boolean) -> AddressBook -> AddressBook

head :: AddressBook -> Maybe Entry
```

우리는 성과 이름으로 검색할 것이므로 성과 이름을 구현할 함수의 인자로 받아야 한다.

또, `filter`를 사용하려면 첫 번째 인자로 넘길 함수도 필요하다. 이 함수를 `filterEntry`라고 하자. 그럼 `filterEntry` 함수는 `Entry -> Boolean` 타입이어야 한다. `filter filterEntry`처럼 적용하면 그 결과는 `AddressBook -> AddressBook` 타입이 된다. 이 함수의 결과를 `head` 함수에 전달하면 우리가 원하는 결과 타입인 `Maybe Entry`를 얻게 된다.

위 사실들을 종합하면 우리가 구현할 함수(`findEntry`라고 하자)의 타입 시그너처를 결정할 수 있다.

```haskell
findEntry :: String -> String -> AddressBook -> Maybe Entry
```

이 타입 시그너처가 말하는 바를 알아보자. `findEntry` 함수는 문자열 두 개로 이름(first name)과 성(last name)을 취하고, 추가로 `AddressBook`도 인자로 받는다. `Entry`를 반환하지만 값이 없을 수도 있다. 이름이 주소록에서 발견되는 경우에만 제대로 된 값을 가질 것이다.

다음은 `findEntry` 함수의 정의이다.

```haskell
findEntry firstName lastName book = head $ filter filterEntry book
  where
    filterEntry :: Entry -> Boolean
    filterEntry entry = entry.firstName == firstName && entry.lastName == lastName
```

이제 코드를 하나하나 따라가보자.

`findEntry` 함수는 `firstName`, `lastName`, `book` 이라는 세 이름을 스코프에 가져온다. 앞의 두 이름은 문자열이고 마지막 `book`은 `AddressBook`이다.

정의를 구성하는 등호 오른쪽 부분은 `filter`와 `head` 함수를 조합한다. 우선 항목 리스트를 필터링하고, 그 결과에 `head` 함수를 적용한다.

술어 함수 `filterEntry`는 `where` 절 내부에 보조 함수로 정의된다.
이렇게 하면 `filterEntry` 함수가 `findEntry` 함수 정의 내부에서만 사용가능하고 바깥에서는 사용할 수 없다. 또 `filterEntry` 함수에서는 바깥 함수(`findEntry`)의 인자들을 사용할 수 있어서 `firstName`과 `lastName`를 이용하여 인자로 주어진 `Entry`가 일치하는지 여부를 판별할 수 있다.

최상위 선언과 마찬가지로 `filterEntry` 함수의 타입 시그너처를 꼭 선언할 필요는 없다. 하지만 타입 선언이 문서화 역할도 해주기 때문에 덧붙이는 방식을 추천한다.

## 이항 연산자 스타일의 함수 적용

`findEntry` 정의를 보면 약간 다른 형태의 함수 적용이 보인다. `head` 함수를 `filter filterEntry book` 표현식에 적용하면서 `$` 기호를 이항 연산자처럼 사용하였다.

`$`를 사용한 표현식은 `head (filter filterEntry book)`처럼 기존의 일반적인 함수 적용 스타일을 사용한 것과 동일하다.

`($)`는 Prelude에 정의된 `apply`라고 하는 일반적인 함수의 별칭일 뿐이다. 이 함수는 다음처럼 정의되어 있다.

```haskell
apply :: forall a b. (a -> b) -> a -> b
apply f x = f x

infixr 0 apply as $
```

`apply`는 함수와 값을 인자로 받아서 함수에 값을 적용한 결과를 반환한다. 
`infixr` 키워드를 사용하여 `apply`에 대한 별명으로 `($)`를 사용한다고 선언했다.

하지만 일반적인 함수 적용 방식 대신 왜 굳이 `$`를 사용할 필요가 있을까?
그 이유는 `$`가 오른쪽 우선으로 결합되며 연산자 우선순위가 가장 낮기 때문이다. 이 말은 `$`를 사용하면 괄호를 여러번 중첩해야 하는 상황을 피할 수 있다는 뜻이다.

예를 들어 아래처럼 여러 단계로 중첩하여 함수를 적용하는 경우가 있다. 여기서는 종업원의 상사를 얻어낸 뒤, 상사의 주소를 얻어내고, 다시 그 주소 중에서 도로명을 얻어내고 있다.

```haskell
street (address (boss employee))
```

위 코드는 `$` 연산자를 이용하면 좀더 읽기 쉬운 코드가 된다. (좀더 읽기 쉽다는 데 동의하지 않는 이들도 있다.)

```haskell
street $ address $ boss employee
```

## 함수 합성

`insertEntry` 함수를 에타 변환으로 단순화할 수 있었던 것과 마찬가지로 `findEntry` 함수의 정의도 단순화할 수 있다.

`book` 인자를 보자. `filter filterEntry` 함수에 전달되고, 그 결과가 다시 `head` 함수에 전달된다. 바꿔말하면 `book` 인자는 `filter filterEntry` 함수와 `head` 함수를 **합성**한 함수에 전달되는 셈이다.

PureScript에서는 함수 합성 연산자로 `<<<`와 `>>>`를 사용한다. 각각 "역방향 합성"과 "순방향 합성"을 의미한다.

이제 `findEntry`를 합성 연산자를 이용하여 재정의할 수 있다. 이 경우는 역방향 합성을 사용하여 아래와 같이 쓸 수 있다.

```
(head <<< filter filterEntry) book
```

이렇게 쓰고 보면 에타 변환이 가능하다는 것을 알 수 있다. 따라서 `findEntry` 함수의 최종 형태는 다음과 같아진다.

```haskell
findEntry firstName lastName = head <<< filter filterEntry
  where
    ...
```

혹은 다음처럼 순방향 합성을 사용해도 결과는 같다.

```haskell
filter filterEntry >>> head
```

어떻게 하든 마지막 정의는 `findEntry` 함수가 무엇인지 명확히 알려준다. "`findEntry` 함수는 필터링 함수와 `head` 함수의 합성이다."

어떤 모양이 더 읽기 쉬운지에 대한 판단은 여러분에게 맡겨두겠다. 대신 함수들을
호출하는 대상만으로 보는 것이 아니라 그 자체로 조작할 수 있음을 인식하는 것은 도움이 될 것이다. 함수 하나가 하나의 작업을 실행하고, 최종 결과는 이런 함수들을 합성하고 조립함으로써 얻을 수도 있다.

## 테스트, 테스트, 테스트, ...

이제 동작하는 애플리케이션의 핵심이 만들어졌다. PSCi에서 확인해 보자.

```text
$ pulp psci

> import Data.AddressBook
```

먼저 텅 빈 주소록에서 뭔가를 검색해 보자. (물론 찾지 못했다는 결과가 나올 것이다.)

```text
> findEntry "John" "Smith" emptyBook

No type class instance was found for

    Data.Show.Show { firstName :: String
                   , lastName :: String
                   , address :: { street :: String
                                , city :: String
                                , state :: String
                                }
                   }
```

에러가 나왔다. 이 에러는 PSCi가 `Entry` 타입의 값을 문자열로 출력할 방법을 모르겠다는 뜻이니 크게 걱정할 필요 없다.

`findEntry` 함수의 결과 타입이 `Maybe Entry`인데 우리는 `Entry`를 `String`으로 바꾸기 위해 따로 함수를 만들었다.

`showEntry` 함수는 인자로 `Entry`를 필요로 하는데 지금 우리가 반환하는 결과 타입은 `Maybe Entry`이다. (`Entry` 타입의 값이 있거나 혹은 없을 수도 있음을 의미한다.) 값이 있을 때 `showEntry`를 적용하고 없을 때는 없다고 표시할 수 있어야 한다.

다행히 `Prelude` 모듈에서는 방법을 제공한다. `map` 연산자를 사용하여 `Maybe`와 같은 타입 생성자가 적용된 경우에도 함수를 사용할 수 있게 한다. (`map` 함수 및 그와 관련된 다른 함수들은 이 책 뒤에서 펑터(functor)를 이야기할 때 또 다루겠다.)

```text
> import Prelude
> map showEntry (findEntry "John" "Smith" emptyBook)

Nothing
```

좀 낫다. `Nothing`이 반환되었는데, 이는 원하는 값이 없다는 것을 나타낸다. 우리가 원하는 결과이다.

편의를 위해 `Maybe Entry`를 `String`으로 출력할 도움 함수를 하나 만들자. 그러면 매번 `map showEntry`를 사용할 필요가 없다.

```text
> printEntry firstName lastName book = map showEntry (findEntry firstName lastName book)
```

이제 비어 있지 않은 주소록을 만들고, 다시 검색해 보자.
앞서 만들었던 항목을 여기서 다시 사용한다.

```text
> book1 = insertEntry entry emptyBook

> printEntry "John" "Smith" book1

Just ("Smith, John: 123 Fake St., Faketown, CA")
```

이번에는 결과가 제대로 된 값을 포함하고 있다. `book1`에 다른 이름의 항목을 추가하여 `book2`를 만들고, 각 이름으로 검색해 보라.

> ## Exercises
>
> 1. (쉬움) `findEntry` 함수를 잘 이해하고 있는지 확인하기 위해 정의의 각 부분을 떼어내어 타입을 표시해 보라. 예를 들어 `head` 함수는 사용된 위치에서 구체화되어 `AddressBook -> Maybe Entry` 타입이 된다.
> 1. (중간) 주소록에서 도로명으로 검색하는 함수를 작성해 보라. `findEntry` 함수의 구현을 참고하라. PSCi에서 작성한 함수를 테스트해 보라.
> 1. (중간) `AddressBook`에 어떤 이름이 나타나는지만 확인하는 함수를 작성해 보라. 반환값은 Boolean 값이다. **힌트**: 어떤 리스트가 비어있는지 여부를 확인하는 `Data.List.null` 함수의 타입을 PSCi를 이용하여 알아보라. 
> 1. (어려움) 주소록에서 같은 성과 이름을 가지는 항목들을 제거해 주는 `removeDuplicates`라는 함수를 작성해 보라. **힌트**: 리스트에서 비교 함수를 이용하여 중복 요소를 제거하는 `Data.List.nubBy` 함수의 타입을 PSCi에서 확인해 보라.

## 결론

이 장에서 우리는 다음의 함수형 프로그래밍 개념들을 다루었다.

- 인터랙티브 모드 PSCi를 이용하여 함수들이나 새로운 아이디어를 확인해 보기.
- 타입은 정확성을 확인하는 도구이면서 동시에 구현 도구이기도 하다.
- 여러 개의 인자를 가지는 함수를 나타내기 위해 커리된 함수를 사용하기.
- 작은 컴포넌트들을 합성하는 방법으로 프로그램 만들기.
- `where` 절을 이용하여 코드를 간결하게 구조화하기.
- null 대신 `Maybe` 타입 이용하기.
- 에타 변환, 함수 합성을 이용하여 코드가 의도를 드러내도록 리팩터링하기.

다음 장부터는 이러한 개념들을 토대로 새로운 개념들을 배워보자.
