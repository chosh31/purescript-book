# Applicative Validation

## 이 장의 목표

이 장에서 처음 만나게 될 **Applicative 펑터**는 매우 중요한 추상화 개념이다. 이름 그대로 `Applicative` 타입 클래스로 정의되는데, 이름 때문에 겁먹을 필요는 없다. 웹 입력 양식의 데이터를 검사하는 실용적 예를 통해 Applicative 펑터의 유용성을 살펴볼 것이다. 이 기술을 잘 적용하면 군더더기없이 간단하고 선언적 형태로 입력 양식 데이터를 검사할 수 있다.

**Traversable 펑터**라고 하는 `Traversable` 타입 클래스도 다룰 것이다. 이 타입 클래스가 실무에서 자연스럽게 유도되는 과정을 보게 될 것이다.

이 장의 예제 코드는 3장의 주소록 예제에서 이어진다. 기존의 데이터 타입을 확장하고 데이터를 검사하기 위한 함수들을 작성할 것이다. 이런 함수들은 웹 UI에서 데이터 입력 양식으로 입력되는 데이터를 검사하고 에러 메시지를 보여주는데 사용할 수 있다.

## 프로젝트 설정

이 장의 소스코드는 `src/Data/AddressBook.purs`, `src/Data/AddressBook/Validation.purs` 파일로 구성된다.

지금까지 보지 못한 새로운 Bower 의존성이 있다.

- `purescript-control`: `Applicative`처럼 제어 흐름을 추상화하는 타입 클래스와 함수들이 정의되어 있다.
- `purescript-validation`: 이 장의 주제인 **Applicative Validation**과 관련된 정의들을 포함한다.

`Data.AddressBook` 모듈에는 이 프로젝트의 데이터 타입과 `Show` 인스턴스가 정의되어 있고, `Data.AddressBook.Validation` 모듈에는 이들 타입에 대한 검사 규칙들이 정의되어 있다.

## 함수 적용의 일반화

**Applicative 펑터** 개념을 설명하기 위해 먼저 `Maybe` 타입 생성자부터 살펴보자.

이 장의 소스코드에 정의된 `address` 함수는 타입이 아래와 같다.

```haskell
address :: String -> String -> String -> Address
```

이 함수는 각각 거리명, 도시, 주를 나타내는 문자열 세 개로부터 `Address` 타입의 값을 만들어낸다.

PSCi에서 이 함수를 적용하고 그 결과를 바로 확인할 수 있다.

```text
> import Data.AddressBook

> address "123 Fake St." "Faketown" "CA"
Address { street: "123 Fake St.", city: "Faketown", state: "CA" }
```

그런데 만일 거리명이나 도시, 혹은 주를 나타내는 문자열이 없을 수도 있고, 이러한 경우를 나타내기 위해 `Maybe` 타입을 사용한다면 어떻게 될까?

예를 들어 도시가 누락된 상태로 이 함수를 바로 적용하려하면 타입 검사기가 다음의 오류 메시지를 보여줄 것이다.

```text
> import Data.Maybe
> address (Just "123 Fake St.") Nothing (Just "CA")

Could not match type

  Maybe String

with type

  String
```

`address`  함수는 문자열을 인자로 취하는데 `Maybe String`을 전달했으니 이 오류는 당연한 결과다.

하지만 `Maybe` 타입으로 표현된 값에 대해서 `address` 함수를 어떤 식으로든 호출할 수 있으면 좋겠다. 그리고 실제로 가능하다. `Control.Apply` 모듈에 정의된 `lift3` 함수가 딱 여기에 필요한 함수다.

```text
> import Control.Apply
> lift3 address (Just "123 Fake St.") Nothing (Just "CA")

Nothing
```

이 경우에는 그 결과가 `Nothing`이다. 필요한 인자 중 하나(도시)가 빠져있기 때문이다. 만약 필요한 문자열 세 개가 모두 `Just` 생성자에 담겨 전달된다면 결과를 제대로 얻을 수 있을 것이다.

```text
> lift3 address (Just "123 Fake St.") (Just "Faketown") (Just "CA")

Just (Address { street: "123 Fake St.", city: "Faketown", state: "CA" })
```

`lift3`이란 이름은 인자를 세 개 가지는 함수를 리프트(lift)한다는 것을 의미한다. `Control.Apply` 모듈에는 인자 갯수 별로 비슷한 함수들이 정의되어 있다.

## 임의 인자 갯수의 함수를 리프트하기

인자 갯수가 몇 개 되지않는 함수들은 `lift2`나 `lift3` 같은 함수들을 이용하면 된다. 그런데 인자 갯수가 몇 개가 되든 적용할 수 있게 이 개념을 확장할 수 있을까?

`lift3` 함수 타입을 살펴보면 힌트를 얻을 수 있다.

```text
> :type lift3
forall a b c d f. Apply f => (a -> b -> c -> d) -> f a -> f b -> f c -> f d
```

`Maybe`를 사용한 예제에서는 `f` 타입 생성자 자리에 `Maybe`가 사용된 것이고 이때 `lift3` 함수는 아래처럼 특수화된다.

```haskell
forall a b c d. (a -> b -> c -> d) -> Maybe a -> Maybe b -> Maybe c -> Maybe d
```

타입을 보면 이 함수는 인자가 세 개인 함수를 입력으로 받아 그것을 리프트한 새로운 함수를 반환한다. 반환된 함수는 인자와 결괏값 모두 `Maybe`로 감싸져 있다.

분명 아무 타입 생성자 `f`에 대해서 이런 식으로 리프트할 수 있는 것은 아닐 것이다. 그렇다면 `Maybe` 타입의 어떠한 특성이 이런 식의 리프트를 가능하게 한 것일까? 앞서 `Maybe`로 특수화하는 과정에서 `f`에 적용된 `Apply` 타입 클래스 제약을 지울 수 있었다. `Apply`는 `Prelude`에 다음처럼 정의되어 있다.

```haskell
class Functor f where
  map :: forall a b. (a -> b) -> f a -> f b

class Functor f <= Apply f where
  apply :: forall a b. f (a -> b) -> f a -> f b
```

`Apply` 타입 클래스는 `Functor`의 서브클래스고, `apply`라는 함수를 추가한다. `<$>` 연산자가 `map` 함수의 별칭으로 정의된 것처럼 `Prelude`에는 `<*>` 연산자가 `apply` 함수의 별칭으로 정의되어 있다. 앞으로도 자주 보겠지만 이 두 연산자는 함께 사용되는 경우가 많다.

`apply` 함수의 타입을 보면 `map`의 타입과 매우 흡사하다. `map`은 함수를 인자로 받는 대신 `apply`의 첫번째 인자는 타입 생성자 `f`에 감싸진 함수라는 점이다. 이 함수가 어떻게 사용되는지 보기 전에 우선 `Maybe` 타입에 대해 `Apply` 타입 클래스가 어떻게 정의되는지부터 보자.

```haskell
instance functorMaybe :: Functor Maybe where
  map f (Just a) = Just (f a)
  map f Nothing  = Nothing

instance applyMaybe :: Apply Maybe where
  apply (Just f) (Just x) = Just (f x)
  apply _        _        = Nothing
```

타입 클래스의 인스턴스를 보면 함수 적용 대상뿐만아니라 적용하려는 함수가 없을 수도 있으며, 함수와 인자가 모두 값이 정의된 경우에만 최종 결과가 정의된다는 것을 알 수 있다.

이제 `map`과 `apply`를 사용하여 인자 갯수별로 함수를 리프트해보자.

인자가 하나인 함수는 간단히 `map`만 사용하면 된다.

인자 갯수가 두 개인 함수를 보자. `a -> b -> c` 처럼 커리된 함수 `g`가 있다. 생략된 괄호를 표시하면 이 타입은 `a -> (b -> c)`가 된다. 이제 인자가 하나인 함수로 보고 `map`을 `g`에 적용하면 그 결과는 `f a -> f (b -> c)` 타입의 새로운 함수가 된다. (여기서 `f`는 `Functor` 인스턴스가 정의되어 있어야 한다.) 이 결과 함수에 첫 번째 인자로 `f a`를  부분 적용하면 남는 것은 `f (b -> c)`, 즉 `f`로 감싸진 함수가 된다. 만약 `f`에 대해 `Apply` 인스턴스가 정의되어 있다면 `apply`를 사용하여 두 번째 인자로 `f b`를 적용할 수 있다. 그 결과는 `f c`가 된다.

길게 설명한 것을 모아보면, `x :: f a`와 `y :: f b`인 두 값이 있을 때 `(g <$> x) <*> y` 표현식의 결과가 `f c`이다. (이 표현식은 `apply (map g x) y`와 동등하다.) 우선순위 규칙에 따라 앞 부분의 괄호를 생략할 수 있으므로 표현식은 다음과 같다. `g <$> x <*> y`.

위 내용을 일반화하면 첫 번째 인자에 `<$>`를 사용하고 남은 인자들에 `<*>` 한다고 볼 수 있으며, 결국 `lift3`은 다음처럼 생각할 수 있다.

```haskell
lift3 :: forall a b c d f
       . Apply f
      => (a -> b -> c -> d)
      -> f a
      -> f b
      -> f c
      -> f d
lift3 f x y z = f <$> x <*> y <*> z
```

`lift3` 함수의 정의가 타입과 일치하는지 독자들이 직접 확인해보길 바란다.

이제 `address` 함수를 `Maybe`로 리프트할 때 `<$>`와 `<*>`를 직접 사용할 수 있다.

```text
> address <$> Just "123 Fake St." <*> Just "Faketown" <*> Just "CA"
Just (Address { street: "123 Fake St.", city: "Faketown", state: "CA" })

> address <$> Just "123 Fake St." <*> Nothing <*> Just "CA"
Nothing
```

인자 갯수가 다른 함수들에 대해서도 위 방법처럼 `Maybe`로 리프트할 수 있을 것이다.

## Applicative 타입 클래스

`Apply` 타입 클래스와 연관된 타입 클래스로 `Applicative`가 있다.

```haskell
class Apply f <= Applicative f where
  pure :: forall a. a -> f a
```

`Applicative`는 `Apply`의 서브클래스이며 `pure` 함수를 추가한다. `pure`는 값을 하나 입력받아 `f` 타입 생성자로 감싸진 값을 만든다.

다음은 `Maybe`의 `Applicative` 인스턴스다.

```haskell
instance applicativeMaybe :: Applicative Maybe where
  pure x = Just x
```

Applicative 펑터가 함수를 리프트하는 펑터라고 본다면 `pure` 함수는 인자가 없는 함수를 리프트하는 것으로 생각할 수도 있다.

## Applicative를 직관적으로 이해하기

PureScript의 함수는 순수 함수라서 부수 효과(side-effect)를 허용하지 않는다. Applicative 펑터를 이용하면 부수 효과를 `f`라는 타입으로 표시할 수 있어서 "프로그래밍 언어"로서의 표현력이 더 커진다.

예를 들어 `Maybe` 펑터는 값을 계산해 낼 수 없는 경우라는 부수 효과를 나타낸다. `Either err`는 `err` 타입의 오류 발생 가능성을 나타내며, `r ->` 화살표 펑터는 전역 설정값을 읽어들이는 부수 효과를 나타낼 수 있다. 지금은 `Maybe` 펑터만 살펴보자.

펑터 `f`가 부수 효과를 포함하는 더 확장된 프로그래밍 언어를 나타낸다고 할 때 `Apply`나 `Applicative` 인스턴스는 값들과 함수 적용이라는 작은 프로그래밍 언어(PureScript)를 새로운 언어의 요소들을 새로운 언어로 리프트하는 방법을 제공한다.

`pure`는 (부수 효과 없는) 순수한 값을 더 큰 언어로 리프트하며, 함수는 `map`이나 `apply`를 이용하여 리프트할 수 있다.

여기서 이런 궁금증이 생길 수도 있다. 만약 `Applicative`를 이용하여 PureScript의 함수와 값을 리프트하여 새로운 언어로 확장한다고 했는데, 여기서 언어가 더 확장되었다는 것이 무엇을 의미하는가? 이 궁금증에 대한 답은 `f` 펑터에 달려있다. 만약 `f a` 타입이면서 `pure x`로 표현할 수 없는 표현식이 있다면 그 표현식은 오직 확장된 언어에만 존재하는 셈이다.

`f`를 `Maybe`로 놓고 보면 `Nothing` 표현식이 이러한 값에 해당한다. `Nothing`은 어떤 `x`를 사용하더라도 `pure x`라는 형태로는 나타낼 수 없다. 따라서 PureScript 언어가 누락된 값을 표현하기 위한 `Nothing`이라는 새로운 항이 추가되면서 더 확장된 것이라고 생각할 수 있다.

## 다른 효과들

다른 `Applicative` 펑터의 예를 더 살펴보자.

PSCi를 열어서 아래처럼 간단한 함수 하나를 정의해보자. 이 함수는 전체 이름을 구성하는 문자열을 세 개 입력받는다.

```text
> import Prelude

> fullName first middle last = last <> ", " <> first <> " " <> middle

> fullName "Phillip" "A" "Freeman"
Freeman, Phillip A
```

이제 이 함수를 이용하는 (매우 간단한) 웹 서비스를 만든다고 가정해보자. 쿼리 인자가 세 개 필요하며 이들 값은 사용자가 입력해야 한다. 쿼리 인자가 누락되는 경우를 나타내기 위해 `Maybe` 타입을 사용한다. 이제 `fullName` 함수를 `Maybe` 펑터로 리프트하여 누락된 값에 대해서도 동작하도록 만들 수 있다.

```text
> import Data.Maybe

> fullName <$> Just "Phillip" <*> Just "A" <*> Just "Freeman"
Just ("Freeman, Phillip A")

> fullName <$> Just "Phillip" <*> Nothing <*> Just "Freeman"
Nothing
```

인자가 하나라도 `Nothing`이면 `Nothing`을 반환한다.

입력 인자가 올바르지 않은 경우 사용자에게 바로 피드백을 줄 수 있기 때문에 이러한 동작은 나쁘지 않다. 하지만 어떤 인자에 문제가 있는지 알려줄 수 있다면 더 낫지 않을까?

`Maybe`로 리프트하는 대신 `Either String`으로 리프트할 수 있다. `Either String`으로 리프트하면 오류 메시지를 반환할 수 있다. 먼저 `Maybe` 타입을 `Either String`으로 변환할 수 있는 연산자를 정의해보자.

```text
> :paste
… withError Nothing  err = Left err
… withError (Just a) _   = Right a
… ^D
```

**주의**: `Either err`라는 Applicative 펑터에서 `Left` 생성자는 오류를, `Right` 생성자는 성공을 나타낸다.

이제 `Either String`으로 리프트하여 각 인자에 대해 적절한 오류 메시지를 제공할 수 있다.

```text
> :paste
… fullNameEither first middle last =
…   fullName <$> (first  `withError` "First name was missing")
…            <*> (middle `withError` "Middle name was missing")
…            <*> (last   `withError` "Last name was missing")
… ^D

> :type fullNameEither
Maybe String -> Maybe String -> Maybe String -> Either String String
```

새로운 함수는 `Maybe`로 감싸진 인자를 셋 받아서, `String` 타입의 오류 메시지 혹은 `String` 타입의 결괏값을 반환한다.

여러가지 입력으로 새 함수를 테스트해보자.

```text
> fullNameEither (Just "Phillip") (Just "A") (Just "Freeman")
(Right "Freeman, Phillip A")

> fullNameEither (Just "Phillip") Nothing (Just "Freeman")
(Left "Middle name was missing")

> fullNameEither (Just "Phillip") (Just "A") Nothing
(Left "Last name was missing")
```

세 인자들 중에서 처음으로 누락된 인자에 대한 정보를 오류 메시지로 제공하는 걸 알 수 있다. 세 인자가 모두 정상이라면 제대로 된 값을 반환한다. 하지만 누락된 입력값이 둘 이상 되더라도 첫 번째 항목만 오류로 알려준다.

```text
> fullNameEither Nothing Nothing Nothing
(Left "First name was missing")
```

이 정도로도 훌륭하다고 할 수 있지만 누락된 인자의 목록을 원한다면 `Either String`보다 강력한 무언가가 필요하다. 그것이 무언인지는 이 장 뒷부분에서 알아보자.

## 효과들 합치기

이번 절에서는 Applicative 펑터를 추상적으로 사용하는 예를 살펴보자. 특정되지 않은 Applicative 펑터 `f`로 나타난 부수 효과들을 합치는 함수를 만들 것이다.

먼저 그 의미를 알아보자. 어떤 타입 `a`가 있고, 여기에 한겹 씌어진 `f a` 값의 리스트가 있다고 하자. 즉 `List (f a)` 타입의 인자를 받는다는 얘기다. 직관적으로 보자면 이 리스트는 `f`라는 부수 효과를 가지는 계산식의 리스트이며 각 계산의 결과는 `a` 타입의 값이다. 이 계산들을 순서대로 모두 실행한다면 `List a`의 값을 얻을 수 있을 것이다. 하지만 이 과정은 모두 `f`라는 효과가 따라붙는다. 다시 말하자면 `List (f a)`의 인자를 받아서 `f (List a)`로 변환할 수 있다는 얘기다. 리스트로 나타난 효과들을 모두 "합친" 것이다.

크기가 `n`으로 고정된 리스트라면 `n` 개의 인자를 받아서 길이가 `n`인 리스트를 만드는 함수를 만들 수 있다. 예를 들어 `n`이 `3`이라면 `\x y z -> x : y : z : Nil` 함수가 그렇다. 이 함수는 `a -> a -> a -> List a` 타입이 된다. 펑터 `f`의 `Applicative` 인스턴스를 이용하여 이 함수를  리프트하면 `f a -> f a -> f a -> f (List a)` 타입의 함수를 얻을 수 있다. 하지만 어떤 `n`에 대해서도 이런 함수를 만들 수 있으므로 리스트 입력에 대해서 리프트하는 것도 가능할 것이라고 기대할 수 있다.

즉 다음과 같은 함수를 작성할 수 있어야 한다.

```haskell
combineList :: forall f a. Applicative f => List (f a) -> f (List a)
```

이 함수는 인자 리스트, 그것도 부수 효과가 딸린 값들을 입력으로 받아 각 부수 효과들을 모두 적용하여 하나의 리스트를 감싼 형태로 반환한다.

이 함수를 작성하기 위해 입력 리스트의 길이를 따져보자. 우선 리스트가 빈 리스트라면 실행할 효과가 하나도 없으므로 `pure`를 이용하여 빈 리스트를 리프트하기만 하면 된다.

```haskell
combineList Nil = pure Nil
```

사실 이 정의 외에 다르게 구현할 방법은 없다.

자 이제 리스트에 값이 하나 이상 있는 경우를 보자. 리스트의 머리와 꼬리를 떼어서 보면 머릿값은 `f a` 타입의 감싸진 값이고 꼬리는 여전히 `List (f a)` 타입이다. 꼬리 부분은 재귀적으로 효과를 합쳐나갈 수 있고, 그 결과는 `f (List a)`가 될 것이다. 이제 `<$>`와 `<*>`를 이용하여 `Cons` 생성자를 리프트하여 머리와 새로운 꼬리에 적용할 수 있다.

```haskell
combineList (Cons x xs) = Cons <$> x <*> combineList xs
```

이번에도 주어진 타입을 만족하기 위해서는 이 밖에 다른 구현을 떠올릴 수 없다.

`Maybe` 타입에 대해 PSCi로 이 함수를 테스트해보자.

```text
> import Data.List
> import Data.Maybe

> combineList (fromFoldable [Just 1, Just 2, Just 3])
(Just (Cons 1 (Cons 2 (Cons 3 Nil))))

> combineList (fromFoldable [Just 1, Nothing, Just 2])
Nothing
```

`Maybe`로 특수화했을 때 이 함수는 리스트의 모든 요소들이 `Just`일 때만 `Just`를 반환한다. 하나라도 `Nothing`이면 `Nothing`을 반환한다. 누락 가능성을 지원하는 확장 언어 관점에서 봤을 때 어떤 계산식의 리스트가 있고 이들 계산식이 모두 제대로 계산되었을 때만 그 결과들을 모두 모아 반환한다는 것은 직관적으로 쉽게 이해할 수 있다.

하지만 `combineList` 함수는 어떠한 `Applicative`에 대해서도 동작한다. 예를 들어 오류를 보고하기 위한 `Either err` 계산식의 리스트에 대해서도 사용할 수 있고, 전역 설정을 읽어들이는 `r ->` 계산식의 리스트에 대해서도 사용할 수 있다.

`combineList` 함수는 뒤에 `Traversable` 펑터를 다룰 때 다시 등장할 것이다.

> ## 연습 문제
>
> 1. (쉬움) `lift2` 함수를 이용하여 수치 연산자들(`+`, `-`, `*`, `/`)을 `Maybe` 펑터로 리프트한 함수들을 작성해보라.
> 1. (보통) 위에서 본 `lift3` 정의로부터 타입이 올바른지 직접 검사해보라.
> 1. (어려움) `forall a f. Applicative f => Maybe (f a) -> f (Maybe a)`타입의 `combineMaybe` 함수를 작성해보라.

## Applicative 검사

이 장의 소스 코드에는 주소록 애플리케이션에 사용할 수 있는 몇가지 자료형이 정의되어 있다. 그 중에서 핵심이 되는 함수는 `Data.AddressBook` 모듈의 다음 함수들이다.

```haskell
address :: String -> String -> String -> Address

phoneNumber :: PhoneType -> String -> PhoneNumber

person :: String -> String -> Address -> Array PhoneNumber -> Person
```

여기서 `PhoneType`은 ADT로 정의되어 있다.

```haskell
data PhoneType = HomePhone | WorkPhone | CellPhone | OtherPhone
```

이 함수들을 이용하여 주소록 항목에 해당하는 `Person` 값을 만들 수 있다. `Data.AddressBook` 모듈에는 다음과 같은 예제가 이미 정의되어 있다.

```haskell
examplePerson :: Person
examplePerson =
  person "John" "Smith"
         (address "123 Fake St." "FakeTown" "CA")
  	     [ phoneNumber HomePhone "555-555-5555"
         , phoneNumber CellPhone "555-555-0000"
  	     ]
```

PSCi에서 이 값들을 테스트해보자.

```text
> import Data.AddressBook

> examplePerson
Person
  { firstName: "John",
  , lastName: "Smith",
  , address: Address
      { street: "123 Fake St."
      , city: "FakeTown"
      , state: "CA"
      },
  , phones: [ PhoneNumber
                { type: HomePhone
                , number: "555-555-5555"
                }
            , PhoneNumber
                { type: CellPhone
                , number: "555-555-0000"
                }
            ]
  }
```

앞 절에서 살펴본 `Either String` 펑터를 이용하면 `Person` 자료형을 검사할 수 있다. 예를 들어 성과 이름이 비어있지 않은지 다음처럼 검사할 수 있다.

```haskell
nonEmpty :: String -> Either String Unit
nonEmpty "" = Left "Field cannot be empty"
nonEmpty _  = Right unit

validatePerson :: Person -> Either String Person
validatePerson (Person o) =
  person <$> (nonEmpty o.firstName *> pure o.firstName)
         <*> (nonEmpty o.lastName  *> pure o.lastName)
         <*> pure o.address
         <*> pure o.phones
```

`nonEmpty` 함수는 문자열이 빈 문자열인지 검사한다. 만약 빈 문자열이라면 `Left` 생성자를 이용하여 오류를 반환하고, 그렇지 않다면 `Right` 생성자를 이용하여 성공을 표시한다. (`unit`은 아무 의미 없는 빈 값이다.) 성공/실패 뒤에 `*>` 연산자를 이용하여 검사 항목을 추가할 수 있다. 여기서는 `pure` 함수를 이용하여 입력 값을 그대로 통과시켰다.

`address`나 `phones` 필드에 대해서는 아무런 검사를 하지 않고 `person` 함수를 호출한다.

이 함수를 PSCi에서 호출해보면 동작하기는 하지만 어떤 필드에 문제가 있는지 보여주지 않아서 효용성이 떨어진다.

```haskell
> validatePerson $ person "" "" (address "" "" "") []
(Left "Field cannot be empty")
```

`Either String` 펑터는 오직 처음 발생하는 실패 오류만 반환하기 때문이다. 성과 이름 모두 비어 있다면 오류가 두 개 나오는 것이 더 나을 것이다.

`purescript-validation` 라이브러리에서 제공하는 또다른 Applicative 펑터를 이용할 수 있다. 이 펑터는 `V`라고 한다. 오류 항목이 **semigroup**이라면 오류를 누적해 주는 기능이 있다. 예를 들어 `V (Array String)`처럼 `String` 타입의 오류를 배열에 담는다면 오류가 발생할 때마다 배열 끝에 추가하여 마지막에 오류 배열을 반환한다.

`Data.AddressBook.Validation` 모듈은 `V (Array String)` Applicative 펑터를 이용하여 `Data.AddressBook` 모듈의 자료형들을 검사한다.

`Data.AddressBook.Validation` 모듈에 정의된 것들 중 일부를 보면 아래와 같다.

```haskell
type Errors = Array String

nonEmpty :: String -> String -> V Errors Unit
nonEmpty field "" = invalid ["Field '" <> field <> "' cannot be empty"]
nonEmpty _     _  = pure unit

lengthIs :: String -> Number -> String -> V Errors Unit
lengthIs field len value | S.length value /= len =
  invalid ["Field '" <> field <> "' must have length " <> show len]
lengthIs _     _   _     =
  pure unit

validateAddress :: Address -> V Errors Address
validateAddress (Address o) =
  address <$> (nonEmpty "Street" o.street *> pure o.street)
          <*> (nonEmpty "City"   o.city   *> pure o.city)
          <*> (lengthIs "State" 2 o.state *> pure o.state)
```

`validateAddress` 함수는 `Address` 자료형을 검사한다. `street`와 `city` 필드가 비어있지 않은지, 그리고 `state` 필드의 길이가 2가 맞는지를 검사한다.

`nonEmpty`와 `lengthIs` 검사 함수에서 사용하는 `invalid` 함수는 `Data.Validation` 모듈에 정의된 것이며 오류를 나타내기 위해 사용한다. 여기서는 오류를 `Array String` 세미그룹으로 표현하고 있으므로 `invalid` 함수의 인자로 문자열 배열을 전달하였다.

이제 이 함수를 PSCi에서 테스트해보자.

```text
> import Data.AddressBook
> import Data.AddressBook.Validation

> validateAddress $ address "" "" ""
(Invalid [ "Field 'Street' cannot be empty"
         , "Field 'City' cannot be empty"
         , "Field 'State' must have length 2"
         ])

> validateAddress $ address "" "" "CA"
(Invalid [ "Field 'Street' cannot be empty"
         , "Field 'City' cannot be empty"
         ])
```

이번에는 검사에서 발견된 전체 오류 목록을 볼 수 있다.

## 정규표현식 검사기

`validatePhoneNumber` 함수는 정규표현식으로 입력 값을 검사한다. 여기서 핵심이 되는 검사 함수는 `matches`이며 이 함수는 `Data.String.Regex` 모듈의 `Regex`를 이용하여 입력 값을 검사한다.

```haskell
matches :: String -> R.Regex -> String -> V Errors Unit
matches _     regex value | R.test regex value =
  pure unit
matches field _     _     =
  invalid ["Field '" <> field <> "' did not match the required format"]
```

여기 사용된 `pure`는 검사를 성공적으로 통과했음을 의미하고 `invalid`는 오류 배열을 이용하여 실패를 나타낸다.

`validatePhoneNumber` 함수는 `matches` 함수를 이용한다.

```haskell
validatePhoneNumber :: PhoneNumber -> V Errors PhoneNumber
validatePhoneNumber (PhoneNumber o) =
  phoneNumber <$> pure o."type"
              <*> (matches "Number" phoneNumberRegex o.number *> pure o.number)
```

올바른 값과 문제 있는 값으로 이 함수를 테스트해보자.

```text
> validatePhoneNumber $ phoneNumber HomePhone "555-555-5555"
Valid (PhoneNumber { type: HomePhone, number: "555-555-5555" })

> validatePhoneNumber $ phoneNumber HomePhone "555.555.5555"
Invalid (["Field 'Number' did not match the required format"])
```

> ## 연습 문제
>
> 1. (쉬움) 정규표현식 검사기를 이용하여 `Address` 타입의 `state` 필드가 정확히 대문자 두 개로 되어 있는지 확인하여라. **힌트**: 소스 코드에서 `phoneNumberRegex` 함수를 참고하라.
> 1. (보통) `matches` 검사기를 이용하여 문자열 전체가 모두 공백문자로 되어 있지 않은지 검사하는 함수를 작성해보라. 이미 정의된 검사기 중에서 `nonEmpty`를 사용하는 곳을 이 함수로 교체하여라.

## Traversable 펑터

마지막으로 살펴볼 검사기는 `validatePerson`이다. 이 함수는 지금까지 살펴본 검사기들을 모두 합쳐서 `Person` 자료형 전체를 검사한다.

```haskell
arrayNonEmpty :: forall a. String -> Array a -> V Errors Unit
arrayNonEmpty field [] =
  invalid ["Field '" <> field <> "' must contain at least one value"]
arrayNonEmpty _     _  =
  pure unit

validatePerson :: Person -> V Errors Person
validatePerson (Person o) =
  person <$> (nonEmpty "First Name" o.firstName *>
              pure o.firstName)
         <*> (nonEmpty "Last Name"  o.lastName  *>
              pure o.lastName)
	       <*> validateAddress o.address
         <*> (arrayNonEmpty "Phone Numbers" o.phones *>
              traverse validatePhoneNumber o.phones)
```

마지막 줄에 `traverse` 함수가 새로이 등장했다.

`traverse` 함수는 `Data.Traversable` 에 정의된 `Traversable` 타입 클래스에서 추가한 함수다.

```haskell
class (Functor t, Foldable t) <= Traversable t where
  traverse :: forall a b f. Applicative f => (a -> f b) -> t a -> f (t b)
  sequence :: forall a f. Applicative f => t (f a) -> f (t a)
```

`Traversable` 타입 클래스는 **Traversable 펑터**를 정의한다. 이 클래스에서 추가하는 함수의 타입을 보면 좀 당혹스러울 수 있다. `validatePerson` 함수에서 어떻게 사용되는지 살펴보는 것으로 의미를 파악해보자.

Traversable 펑터는 `Functor`이면서 동시에 `Foldable`이기도 하다. **Foladable 펑터**는 fold 연산, 즉 전체 구조를 하나의 값으로 환원시킬 수 있는 타입 생성자라는 사실을 떠올리자. Traversable 펑터는 여기서 한걸음 나아가 해당 자료 구조에 적용되는 부수 효과들까지 모두 결합해준다.

너무 복잡하게 들린다면 배열에 국한시켜 단순화시켜보자. 배열은 Traversable 펑터다. 즉 다음 함수를 사용할 수 있다는 얘기다.

```haskell
traverse :: forall a b f. Applicative f => (a -> f b) -> Array a -> f (Array b)
```

직관적으로 보자면 어떤 임의의 Applicative 펑터 `f`가 있고, 배열 요소인 `a` 타입의 값을 부소 효과 `f`를 가지는 `b`로 반환해주는 함수가 있다면, `traverse` 함수를 이용하여 배열 `Array a`의 각 요소에 이 부수 효과를 유발하는 함수를 모두 적용시키고 그 결과를 `Array b`로 얻어낼 수 있다. 물론 이때는 `f`로 부수 효과가 따라온다.

여전히 이해되지 않는다면 Applicative 펑터 `f`를 `V Errors` 펑터로 한번 더 특수화해보자. 그럼 `traverse` 함수의 타입은 아래와 같아진다.

```haskell
traverse :: forall a b. (a -> V Errors b) -> Array a -> V Errors (Array b)
```

타입 `a`를 검사하는 함수 `f`가 있을 때, `traverse f`는 `Array a` 타입의 배열을 검사하는 함수라는 위 타입으로부터 알 수 있다. 그리고 이것이 바로 `Person` 자료형의 `phones` 필드를 검사하는데 필요한 함수다. `validatePhoneNumber` 함수를 `traverse`에 전달하기만 하면 배열의 모든 요소들을 순차적으로 검사하는 함수를 만들 수 있다.

일반화시켜 말하자면 `traverse` 함수는 어떤 자료 구조를 순회하면서 부수 효과를 수반하는 계산을 실시하고, 그 결과들을 다시 원래 모양의 자료 구조대로 모아준다.

`Traversable` 클래스에 정의된 또다른 함수인 `sequence`의 타입은 좀더 친숙하게 보인다.

```haskell
sequence :: forall a f. Applicative f => t (f a) -> f (t a)
```

사실 앞에서 구현했던 `combineList` 함수는 `sequence` 함수를 특수화한 형태에 불과하다. `t` 타입 생성자 자리에 `List`를 넣어보면 `combineList` 함수의 타입이 된다.

```haskell
combineList :: forall f a. Applicative f => List (f a) -> f (List a)
```

Traversable 펑터는 자료 구조를 순회하면서 부수 효과를 동반한 계산을 실시하고 그 부수 효과들을 합치면서 계산 결과를 모을 수 있음을 의미한다. 사실 `sequence`나 `traverse` 함수는 `Traversable` 클래스 입장에서 동등한 중요성을 가지고 있으며 하나로 다른 하나를 구현할 수 있다. 독자 스스로 연습삼아 구현해보기 바란다.

`Data.List` 모듈에는 리스트에 대한 `Traversable` 인스턴스가 정의되어 있다. 리스트에 대한 `traverse` 구현을 보자.

```haskell
-- traverse :: forall a b f. Applicative f => (a -> f b) -> List a -> f (List b)
traverse _ Nil = pure Nil
traverse f (Cons x xs) = Cons <$> f x <*> traverse f xs
```

리스트가 비어 있는 경우에는 간단히 `pure`를 이용하여 빈 리스트를 반환하면 된다. 비어 있지 않은 경우에는 머리 요소에 함수 `f`를 적용하여 `f b` 계산을 만들고, 재귀적으로 꼬리에 `traverse`를 적용한 다음, `Cons` 생성자를 펑터 `f`로 리프트하여 두 결과에 적용한다.

Traversable 펑터가 비단 배열이나 리스트에만 정의된 것은 아니고 다른 예제도 많다. `Maybe` 타입 생성자도 `Traversable` 인스턴스를 가진다. PSCi에서 확인해보자.

```text
> import Data.Maybe
> import Data.Traversable

> traverse (nonEmpty "Example") Nothing
(Valid Nothing)

> traverse (nonEmpty "Example") (Just "")
(Invalid ["Field 'Example' cannot be empty"])

> traverse (nonEmpty "Example") (Just "Testing")
(Valid (Just unit))
```

`Nothing`은 순회할 대상이 없으므로 아무런 검사 없이 그대로 `Nothing`을 반환한다. `Just x`는 검사 함수를 이용하여 `x`를 검사한다. 즉 `traverse`는 `a` 타입에 대한 검사 함수를 입력받아 `Maybe a`에 대한 검사 함수를 반환한다.

`Tuple a`나 `Either a`에 대한 `Traversable` 인스턴스도 있다. 일반적으로 보자면 거의 대부분의 "컨테이너"라고 할 수 있는 타입 생성자들은 `Traversable` 인스턴스를 정의할 수 있다. 연습 문제에서 이진 트리에 대한 `Traversable` 인스턴스를 작성해보라.

> ## 연습 문제
>
> 1. (보통) 아래처럼 정의된 이진 트리에 대해 `Traversable` 인스턴스를 정의해보라. 가지의 경우 부수 효과를 합칠 때 왼쪽에서 오른쪽의 순서로 합친다.
>
>     ```haskell
>     data Tree a = Leaf | Branch (Tree a) a (Tree a)
>     ```
>
>     이 방식은 트리 순회 방식 중에서 in-order 순회에 해당한다. preorder 방식 혹은 in-order 순서를 뒤집는 경우는 어떻게 될까?
>
> 1. (보통) 코드를 수정하여 `Person` 타입의 `address` 필드를 `Data.Maybe`로 바꾸고, 나머지 함수들이 잘 동작하도록 만들어라. **힌트**: `Maybe a` 타입의 필드는 `traverse` 함수로 검사할 수 있다.
> 1. (어려움) `sequence` 함수를 `traverse` 함수로 구현해보라. `traverse` 함수를 `sequence`로 구현해보라.

## Applicative 펑터를 이용한 병렬 실행

지금까지 설명하면서 "부수 효과를 합친다"라고 했다. 하지만 "합친다"라는 표현 대신 부수 효과를 "순서대로 실행한다"라고 할 수도 있었다. Traversable 평터에서 제공하는 `sequence` 함수와도 직관적으로 잘 연결된다. 부수 효과를 줄세우고 순서대로 실행하는 방식으로 그 효과들을 합치는 것이다.

하지만 Applicative 펑터는 이보다 더 일반화된 개념을 나타낸다. Applicative 펑터의 법칙들은 계산에 수반되는 부수 효과에 대해 어떤 순서를 요구하지 않는다. 이 부수 효과들을 병렬로 실행하여도 Applicative 법칙을 위배하지 않는다.

예를 들어 `V` 검사기 펑터는 오류의 **배열**을 반환하게 했는데, `Set` 세미그룹을 선택했다 하더라도 문제될 것이 없다. 만약 `Set`를 선택했다면 여러가지 검사의 실행 순서가 더이상 의미를 가지지 않는다. 심지어 자료 구조에 대해 병렬 실행할 수도 있다.

두 번째 예제로 다룰 것은 `purescript-parallel` 패키지가 제공하는 `Parallel` 타입 클래스이다. 이 타입 클래스는 **병렬 계산**을 지원한다. `Parallel` 클래스의 함수인 `parallel`은 다른 `Applicative` 펑터로 표시되는 계산을 병렬로 진행하여 결과를 계산한다.

```haskell
f <$> parallel computation1
  <*> parallel computation2
```

여기서 `computation1`과 `computation2` 계산을 비동기적으로 시작한다. 두 계산이 모두 끝나고 결과가 준비되면 `f` 함수를 이용하여 그 결과들을 합친다.

이 주제는 나중에 **콜백 헬** 문제에 Applicative 펑터를 적용하는 과정에서 자세히 다룰 것이다.

Applicative 펑터는 부수 효과를 병렬로 결합하고자 할 때 사용하는 것이 자연스럽다.

## 결론

이 장에서는 새로운 개념들을 많이 다루었다.

- **Applicative 펑터**를 이용하여 부수 효과를 가지는 타입 생성자들을 일반화할 수 있었다.
- Applicative 펑터를 이용하여 자료 구조를 검사하는 방법을 살펴보았다. 먼저 오류 하나를 반환하는 것을 살펴본 뒤 모든 오류를 모아서 반환하도록 바꿔보았다.
- `Traversable` 타입 클래스는 **순회 가능한 펑터**라는 개념을 추상화하였다. 컨테이너에 포함된 모든 요소들에 부수 효과를 가진 계산을 적용하고 부수 효과와 계산 결과를 모을 수 있었다.

Applicative 펑터는 여러가지 문제에 깔끔한 해결책을 제시한다. 앞으로도 여러가지 활용을 더 볼 수 있다. 이 장에서는 검사기 펑터를 이용하여 검사 로직을 선언적 형태로 작성할 수 있었다. "어떻게" 검사할 것이냐가 아니라 "무엇을" 검사할 것이냐는 형태로 기술할 수 있었다. 이를 일반화하자면 Applicative 펑터는 **도메인 특성 언어(Domain specific language)**를 설계할 때 유용한 도구이다.

다음 장에서는 Applicative 펑터와 관련된 개념인 **모나드** 클래스를 살펴보고 주소록 예제를 브라우저에서 실행할 수 있게 확장할 것이다.