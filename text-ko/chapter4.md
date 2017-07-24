# 재귀, Map과 Fold

## 이 장의 목표

이 장에서는 알고리즘을 구조화하는 방법으로서 재귀함수를 들여다 볼 것이다. 재귀는 함수형 프로그래밍에서 기본적인 기법이며, 이 책 전체에 걸쳐 사용할 것이다.

PureScript 표준 라이브러리의 함수들도 몇 개 살펴볼 것이다. 특히 `map`과 `fold` 함수들, 그리고 조금 특화된 `filter`나 `concatMap` 함수들을 다룬다.

이번 장을 이끌어갈 예제는 가상의 파일 시스템을 다루기 위한 라이브러리이다. 파일 시스템을 구성하는 파일들의 특성을 계산하는 함수들을 작성하면서 이 장에서 배우게 될 기법들을 적용해 보자.

## 프로젝트 설정

이 장의 소스 코드는 `src/Data/Path.purs`와  `src/FileOperations.purs` 파일이다.

`Data.Path` 모듈에는 가상 파일 시스템의 모델이 정의되어 있다. 이 모듈의 내용을 수정할 일은 없다.

`FileOperations` 모듈에는 `Data.Path`를 사용하는 함수들이 정의되어 있다. 이 장에 포함된 연습 문제를 완성하려면 이 파일을 수정하면 된다.

프로젝트는 다음과 같은 Bower 의존성을 가진다.

- `purescript-maybe`: `Maybe` 타입 생성자가 정의되어 있다.
- `purescript-arrays`: 배열을 사용하기 위한 함수들이 정의되어 있다.
- `purescript-strings`: JavaScript 문자열을 다루기 위한 함수들이 정의되어 있다.
- `purescript-foldable-traversable`: 배열이나 다른 자료 구조들을 폴딩(fold)할 수 있는 함수들이 정의되어 있다.
- `purescript-console`: 콘솔 출력 함수들이 정의되어 있다.

## 시작하기

프로그래밍에서 재귀는 매우 중요한 기법이다. 특히 순수 함수형 프로그래밍에서 더 일반적이다. 그 이유는 재귀를 사용하면 프로그램에서 상태 변경을 줄일 수 있기 때문이다.

재귀는 **분할 정복** 전략(어떤 입력을 처리하려고 할 때 그 입력을 작게 나누어 처리한 다음, 그 부분 결과를 합쳐서 최종 결과를 얻는다.)과도 관련이 깊다.

PureScript에서 재귀의 간단한 예 몇 가지를 보자.

다음은 일반적인 **팩토리얼 함수**이다. 

```haskell
fact :: Int -> Int
fact 0 = 1
fact n = n * fact (n - 1)
```

팩토리얼 함수는 문제를 작은 문제로 나누어 풀고 있음을 알 수 있다. 주어진 정수보다 1 작은 정수에 대해 팩토리얼을 계산한다. 0에 도달하면 문제를 더 나눌 필요없이 답이 명확하다.

또다른 보편적 예로서 **피보나찌 함수**를 보자.

```haskell
fib :: Int -> Int
fib 0 = 1
fib 1 = 1
fib n = fib (n - 1) + fib (n - 2)
```

역시 문제를 작은 문제로 나누어 풀고 있다. 이번에는 `fib (n - 1)`과 `fib (n - 2)` 이렇게 두 개의 작은 문제를 먼저 풀어서 각각의 부분 결과를 더한다.

## 배열에 재귀 사용하기

재귀 함수는 `Int` 타입에만 국한되지 않는다. 나중에 **패턴 매칭**을 다룰 때 다양한 데이터 타입에 대해 재귀 함수를 정의하는 방법을 다루게 된다. 하지만 지금은 수나 배열에 대해서만 살펴보기로 하자.

입력이 0이냐 아니냐로 분기한 것과 마찬가지로 배열에 대해서도 입력이 비어있거나 그렇지 않은 경우로 분기한다. 다음 예는 재귀를 사용하여 배열의 길이를 계산하는 함수이다.

```haskell
import Prelude

import Data.Array (null)
import Data.Array.Partial (tail)
import Partial.Unsafe (unsafePartial)

length :: forall a. Array a -> Int
length arr =
  if null arr
    then 0
    else 1 + length (unsafePartial tail arr)
```

이 함수는 배열이 비어있는지 여부에 따라 분기하기 위해 `if .. then .. else` 표현식을 사용하였다. `null` 함수는 배열이 비어있으면 `true`를 반환한다. 빈 배열은 길이가 0이고, 비어 있지 않은 배열은 꼬리(첫 요소를 뺀 나머지 배열)의 길이에 1을 더해서 길이를 계산한다.

이런 식으로 배열의 길이를 구하는 것은 당연히 비효율적이다. 하지만 다음 연습 문제를 푸는데 도움이 될 것이다.

> ## 연습 문제
>
> 1. (쉬움) 입력 값이 짝수이면 `true`를 반환하는 재귀 함수를 작성해 보라.
> 1. (보통) 배열에서 짝수의 갯수를 반환하는 재귀 함수를 작성해 보라. **힌트**: `unsafePartial head`를 사용하면 비어 있지 않은 배열의 첫 요소를 얻어낼 수 있다.(`head`는 `Data.Array.Partial`에서 임포트한다.)

## Map

`map` 함수는 배열에 대한 재귀 함수이다. 배열의 각 요소들에 어떤 함수를 일괄적으로 적용하여 변환하기 위해 사용한다. 즉 배열의 내용을 바꾸기는 하지만 그 모양(배열의 길이)은 그대로 유지한다.

이 책 뒤에서 **타입 클래스**를 다룰 때 보겠지만 `map` 함수는 **펑터(functor)**라고 하는 타입 생성자들의 클래스에 대해 모양을 유지하면서 변환하는 좀더 일반적인 패턴의 한 예이다.

PSCi에서 `map` 함수를 사용해 보자.

```text
$ pulp psci

> import Prelude
> map (\n -> n + 1) [1, 2, 3, 4, 5]
[2, 3, 4, 5, 6]
```

`map` 함수를 사용하는 형태를 보자. 첫 번째 인자로 전달하는 함수는 배열의 각 요소들을 변환할 함수이고, 두 번째 인자로 배열이 전달된다.

## 중위 연산자

`map` 함수를 사용할 때 변환 함수와 배열 사이에 `map`을 넣을 수도 있다. 이 때는 함수 이름을 백틱(`)으로 감싸줘야 한다.

```text
> (\n -> n + 1) `map` [1, 2, 3, 4, 5]
[2, 3, 4, 5, 6]
```

**중위 함수 적용** 문법이다. 어떤 함수라도 이런 식으로 중위 연산자처럼 사용할 수 있다. 대부분은 인자가 두 개 있는 함수에 사용하는 것이 적당하다.

`map` 함수를 배열에 사용하는 경우, 이와 똑같은 역할을 하는 연산자도 있다. `<$>`라고 하는 이 연산자는 다른 이항 연산자들처럼 중위 표기로 사용할 수 있다.

```text
> (\n -> n + 1) <$> [1, 2, 3, 4, 5]
[2, 3, 4, 5, 6]
```

`map`  함수의 타입을 보자.

```text
> :type map
forall a b f. Functor f => (a -> b) -> f a -> f b
```

`map`의 타입을 보면 사실 이 장에서 필요한 것 이상으로 더 일반적이다. 일단 이번 장에서는 `map`의 타입이 다음처럼 덜 일반적인 것으로 보자.

```text
forall a b. (a -> b) -> Array a -> Array b
```

`a`와 `b` 이렇게 두 개의 타입이 결정되어야 `map` 함수를 사용할 수 있다.
`a`는 입력 배열의 요소들 타입이고, `b`는 출력 배열의 요소들 타입이다.
즉 `map`이 배열 요소의 타입을 유지할 필요가 없다는 얘기다. 예를 들어 `map`이나 `<$>`을 사용하여 정수 배열을 문자열 배열로 바꿀 수 있다.

```text
> show <$> [1, 2, 3, 4, 5]

["1","2","3","4","5"]
```

중위 연산자인 `<$>`가 특수 문법인 것처럼 보이긴 하지만 사실 보통의 PureScript 함수에 대한 별칭일 뿐이다. 이 함수가 중위 표현 문법으로 적용되어 있지만 괄호로 감싸서 다른 함수들처럼 사용할 수도 있다. `map` 자리에 `(<$>)`를 넣어도 된다.

```text
> (<$>) show [1, 2, 3, 4, 5]
["1","2","3","4","5"]
```

중위로 표기할 함수들 이름은 기존 함수들에 별칭으로 정의된다.
예를 들어 `Data.Array` 모듈에 정의된 중위 연산자 `(..)`는 `range` 함수의 별칭으로 정의되었다.

```haskell
infix 8 range as ..
```

이제 이 연산자를 다음처럼 사용한다.

```text
> import Data.Array

> 1 .. 5
[1, 2, 3, 4, 5]

> show <$> (1 .. 5)
["1","2","3","4","5"]
```

**주의**: 중위 연산자는 도메인 특정 언어(DSL)을 정의할 때 매우 유용한 도구이다. 하지만 과도하게 사용할 경우 코드를 처음 접하는 이들에게 가독성을 해칠 수 있다. 새로운 연산자를 정의할 때는 신중하게 판단하라.

위 예제에서 `1 .. 5`를 괄호로 감쌌는데, 사실 괄호가 꼭 필요하진 않다. `Data.Array` 모듈에서 `..` 연산자를 정의할 때 `<$>`보다 우선순위를 더 높게 지정했기 때문이다. `..` 연산자를 정의할 때 `infix` 뒤에 8이라고 우선순위를 지정했다. `<$>` 보다 우선순위가 더 높기 때문에 괄호를 사용하지 않아도 된다.


```text
> show <$> 1 .. 5
["1","2","3","4","5"]
```

중위 연산자의 결합 방향(왼쪽/오른쪽)을 지정하고자 할 때는 `infixl`이나 `infixr` 키워드를 사용하면 된다.

## 배열 필터링

The `Data.Array` module provides another function `filter`, which is commonly used together with `map`. It provides the ability to create a new array from an existing array, keeping only those elements which match a predicate function.

For example, suppose we wanted to compute an array of all numbers between 1 and 10 which were even. We could do so as follows:

```text
> import Data.Array

> filter (\n -> n `mod` 2 == 0) (1 .. 10)
[2,4,6,8,10]
```

X> ## Exercises
X>
X> 1. (Easy) Use the `map` or `<$>` function to write a function which calculates the squares of an array of numbers.
X> 1. (Easy) Use the `filter` function to write a function which removes the negative numbers from an array of numbers.
X> 1. (Medium) Define an infix synonym `<$?>` for `filter`. Rewrite your answer to the previous question to use your new operator. Experiment with the precedence level and associativity of your operator in PSCi.

## Flattening Arrays

Another standard function on arrays is the `concat` function, defined in `Data.Array`. `concat` flattens an array of arrays into a single array:

```text
> import Data.Array

> :type concat
forall a. Array (Array a) -> Array a

> concat [[1, 2, 3], [4, 5], [6]]
[1, 2, 3, 4, 5, 6]
```

There is a related function called `concatMap` which is like a combination of the `concat` and `map` functions. Where `map` takes a function from values to values (possibly of a different type), `concatMap` takes a function from values to arrays of values.

Let's see it in action:

```text
> import Data.Array

> :type concatMap
forall a b. (a -> Array b) -> Array a -> Array b

> concatMap (\n -> [n, n * n]) (1 .. 5)
[1,1,2,4,3,9,4,16,5,25]
```

Here, we call `concatMap` with the function `\n -> [n, n * n]` which sends an integer to the array of two elements consisting of that integer and its square. The result is an array of ten integers: the integers from 1 to 5 along with their squares.

Note how `concatMap` concatenates its results. It calls the provided function once for each element of the original array, generating an array for each. Finally, it collapses all of those arrays into a single array, which is its result.

`map`, `filter` and `concatMap` form the basis for a whole range of functions over arrays called "array comprehensions".

## Array Comprehensions

Suppose we wanted to find the factors of a number `n`. One simple way to do this would be by brute force: we could generate all pairs of numbers between 1 and `n`, and try multiplying them together. If the product was `n`, we would have found a pair of factors of `n`.

We can perform this computation using an array comprehension. We will do so in steps, using PSCi as our interactive development environment.

The first step is to generate an array of pairs of numbers below `n`, which we can do using `concatMap`.

Let's start by mapping each number to the array `1 .. n`:

```text
> pairs n = concatMap (\i -> 1 .. n) (1 .. n)
```

We can test our function

```text
> pairs 3
[1,2,3,1,2,3,1,2,3]
```

This is not quite what we want. Instead of just returning the second element of each pair, we need to map a function over the inner copy of `1 .. n` which will allow us to keep the entire pair:

```text
> :paste
… pairs' n =
…   concatMap (\i ->
…     map (\j -> [i, j]) (1 .. n)
…   ) (1 .. n)
… ^D

> pairs' 3
[[1,1],[1,2],[1,3],[2,1],[2,2],[2,3],[3,1],[3,2],[3,3]]
```

This is looking better. However, we are generating too many pairs: we keep both [1, 2] and [2, 1] for example. We can exclude the second case by making sure that `j` only ranges from `i` to `n`:

```text
> :paste
… pairs'' n =
…   concatMap (\i ->
…     map (\j -> [i, j]) (i .. n)
…   ) (1 .. n)
… ^D
> pairs'' 3
[[1,1],[1,2],[1,3],[2,2],[2,3],[3,3]]
```

Great! Now that we have all of the pairs of potential factors, we can use `filter` to choose the pairs which multiply to give `n`:

```text
> import Data.Foldable

> factors n = filter (\pair -> product pair == n) (pairs'' n)

> factors 10
[[1,10],[2,5]]
```

This code uses the `product` function from the `Data.Foldable` module in the `purescript-foldable-traversable` library.

Excellent! We've managed to find the correct set of factor pairs without duplicates.

## Do Notation

However, we can improve the readability of our code considerably. `map` and `concatMap` are so fundamental, that they (or rather, their generalizations `map` and `bind`) form the basis of a special syntax called _do notation_.

_Note_: Just like `map` and `concatMap` allowed us to write _array comprehensions_, the more general operators `map` and `bind` allow us to write so-called _monad comprehensions_. We'll see plenty more examples of _monads_ later in the book, but in this chapter, we will only consider arrays.

We can rewrite our `factors` function using do notation as follows:

```haskell
factors :: Int -> Array (Array Int)
factors n = filter (\xs -> product xs == n) $ do
  i <- 1 .. n
  j <- i .. n
  pure [i, j]
```

The keyword `do` introduces a block of code which uses do notation. The block consists of expressions of a few types:

- Expressions which bind elements of an array to a name. These are indicated with the backwards-facing arrow `<-`, with a name on the left, and an expression on the right whose type is an array.
- Expressions which do not bind elements of the array to names. The last line `pure [i, j]` is an example of this kind of expression.
- Expressions which give names to expressions, using the `let` keyword.

This new notation hopefully makes the structure of the algorithm clearer. If you mentally replace the arrow `<-` with the word "choose", you might read it as follows: "choose an element `i` between 1 and n, then choose an element `j` between `i` and `n`, and return `[i, j]`".

In the last line, we use the `pure` function. This function can be evaluated in PSCi, but we have to provide a type:

```text
> pure [1, 2] :: Array (Array Int)
[[1, 2]]
```

In the case of arrays, `pure` simply constructs a singleton array. In fact, we could modify our `factors` function to use this form, instead of using `pure`:

```haskell
factors :: Int -> Array (Array Int)
factors n = filter (\xs -> product xs == n) $ do
  i <- 1 .. n
  j <- i .. n
  [[i, j]]
```

and the result would be the same.

## Guards

One further change we can make to the `factors` function is to move the filter inside the array comprehension. This is possible using the `guard` function from the `Control.MonadZero` module (from the `purescript-control` package):

```haskell
import Control.MonadZero (guard)

factors :: Int -> Array (Array Int)
factors n = do
  i <- 1 .. n
  j <- i .. n
  guard $ i * j == n
  pure [i, j]
```

Just like `pure`, we can apply the `guard` function in PSCi to understand how it works. The type of the `guard` function is more general than we need here:

```text
> import Control.MonadZero

> :type guard
forall m. MonadZero m => Boolean -> m Unit
```

In our case, we can assume that PSCi reported the following type:

```haskell
Boolean -> Array Unit
```

For our purposes, the following calculations tell us everything we need to know about the `guard` function on arrays:

```text
> import Data.Array

> length $ guard true
1

> length $ guard false
0
```

That is, if `guard` is passed an expression which evaluates to `true`, then it returns an array with a single element. If the expression evaluates to `false`, then its result is empty.

This means that if the guard fails, then the current branch of the array comprehension will terminate early with no results. This means that a call to `guard` is equivalent to using `filter` on the intermediate array. Depending on the application, you might prefer to use `guard` instead of a `filter`. Try the two definitions of `factors` to verify that they give the same results.

X> ## Exercises
X>
X> 1. (Easy) Use the `factors` function to define a function `isPrime` which tests if its integer argument is prime or not.
X> 1. (Medium) Write a function which uses do notation to find the _cartesian product_ of two arrays, i.e. the set of all pairs of elements `a`, `b`, where `a` is an element of the first array, and `b` is an element of the second.
X> 1. (Medium) A _Pythagorean triple_ is an array of numbers `[a, b, c]` such that `a² + b² = c²`. Use the `guard` function in an array comprehension to write a function `triples` which takes a number `n` and calculates all Pythagorean triples whose components are less than `n`. Your function should have type `Int -> Array (Array Int)`.
X> 1. (Difficult) Write a function `factorizations` which produces all _factorizations_ of an integer `n`, i.e. arrays of integers whose product is `n`. _Hint_: for an integer greater than 1, break the problem down into two subproblems: finding the first factor, and finding the remaining factors.

## Folds

Left and right folds over arrays provide another class of interesting functions which can be implemented using recursion.

Start by importing the `Data.Foldable` module, and inspecting the types of the `foldl` and `foldr` functions using PSCi:

```text
> import Data.Foldable

> :type foldl
forall a b f. Foldable f => (b -> a -> b) -> b -> f a -> b

> :type foldr
forall a b f. Foldable f => (a -> b -> b) -> b -> f a -> b
```

These types are actually more general than we are interested in right now. For the purposes of this chapter, we can assume that PSCi had given the following (more specific) answer:

```text
> :type foldl
forall a b. (b -> a -> b) -> b -> Array a -> b

> :type foldr
forall a b. (a -> b -> b) -> b -> Array a -> b
```

In both of these cases, the type `a` corresponds to the type of elements of our array. The type `b` can be thought of as the type of an "accumulator", which will accumulate a result as we traverse the array.

The difference between the `foldl` and `foldr` functions is the direction of the traversal. `foldl` folds the array "from the left", whereas `foldr` folds the array "from the right".

Let's see these functions in action. Let's use `foldl` to sum an array of integers. The type `a` will be `Int`, and we can also choose the result type `b` to be `Int`. We need to provide three arguments: a function `Int -> Int -> Int`, which will add the next element to the accumulator, an initial value for the accumulator of type `Int`, and an array of `Int`s to add. For the first argument, we can just use the addition operator, and the initial value of the accumulator will be zero:

```text
> foldl (+) 0 (1 .. 5)
15
```

In this case, it didn't matter whether we used `foldl` or `foldr`, because the result is the same, no matter what order the additions happen in:

```text
> foldr (+) 0 (1 .. 5)
15
```

Let's write an example where the choice of folding function does matter, in order to illustrate the difference. Instead of the addition function, let's use string concatenation to build a string:

```text
> foldl (\acc n -> acc <> show n) "" [1,2,3,4,5]
"12345"

> foldr (\n acc -> acc <> show n) "" [1,2,3,4,5]
"54321"
```

This illustrates the difference between the two functions. The left fold expression is equivalent to the following application:

```text
((((("" <> show 1) <> show 2) <> show 3) <> show 4) <> show 5)
```

whereas the right fold is equivalent to this:

```text
((((("" <> show 5) <> show 4) <> show 3) <> show 2) <> show 1)
```

## Tail Recursion

Recursion is a powerful technique for specifying algorithms, but comes with a problem: evaluating recursive functions in JavaScript can lead to stack overflow errors if our inputs are too large.

It is easy to verify this problem, with the following code in PSCi:

```text
> f 0 = 0
> f n = 1 + f (n - 1)

> f 10
10

> f 100000
RangeError: Maximum call stack size exceeded
```

This is a problem. If we are going to adopt recursion as a standard technique from functional programming, then we need a way to deal with possibly unbounded recursion.

PureScript provides a partial solution to this problem in the form of _tail recursion optimization_.

_Note_: more complete solutions to the problem can be implemented in libraries using so-called _trampolining_, but that is beyond the scope of this chapter. The interested reader can consult the documentation for the `purescript-free` and `purescript-tailrec` packages.

The key observation which enables tail recursion optimization is the following: a recursive call in _tail position_ to a function can be replaced with a _jump_, which does not allocate a stack frame. A call is in _tail position_ when it is the last call made before a function returns. This is the reason why we observed a stack overflow in the example - the recursive call to `f` was _not_ in tail position.

In practice, the PureScript compiler does not replace the recursive call with a jump, but rather replaces the entire recursive function with a _while loop_.

Here is an example of a recursive function with all recursive calls in tail position:

```haskell
fact :: Int -> Int -> Int
fact 0 acc = acc
fact n acc = fact (n - 1) (acc * n)
```

Notice that the recursive call to `fact` is the last thing that happens in this function - it is in tail position.

## Accumulators

One common way to turn a function which is not tail recursive into a tail recursive function is to use an _accumulator parameter_. An accumulator parameter is an additional parameter which is added to a function which _accumulates_ a return value, as opposed to using the return value to accumulate the result.

For example, consider this array recursion which reverses the input array by appending elements at the head of the input array to the end of the result.

```haskell
reverse :: forall a. Array a -> Array a
reverse [] = []
reverse xs = snoc (reverse (unsafePartial tail xs))
                  (unsafePartial head xs)
```

This implementation is not tail recursive, so the generated JavaScript will cause a stack overflow when executed on a large input array. However, we can make it tail recursive, by introducing a second function argument to accumulate the result instead:

```haskell
reverse :: forall a. Array a -> Array a
reverse = reverse' []
  where
    reverse' acc [] = acc
    reverse' acc xs = reverse' (unsafePartial head xs : acc)
                               (unsafePartial tail xs)
```

In this case, we delegate to the helper function `reverse'`, which performs the heavy lifting of reversing the array. Notice though that the function `reverse'` is tail recursive - its only recursive call is in the last case, and is in tail position. This means that the generated code will be a _while loop_, and will not blow the stack for large inputs.

To understand the second implementation of `reverse`, note that the helper function `reverse'` essentially uses the accumulator parameter to maintain an additional piece of state - the partially constructed result. The result starts out empty, and grows by one element for every element in the input array. However, because later elements are added at the front of the array, the result is the original array in reverse!

Note also that while we might think of the accumulator as "state", there is no direct mutation going on. The accumulator is an immutable array, and we simply use function arguments to thread the state through the computation.

## Prefer Folds to Explicit Recursion

If we can write our recursive functions using tail recursion, then we can benefit from tail recursion optimization, so it becomes tempting to try to write all of our functions in this form. However, it is often easy to forget that many functions can be written directly as a fold over an array or similar data structure. Writing algorithms directly in terms of combinators such as `map` and `fold` has the added advantage of code simplicity - these combinators are well-understood, and as such, communicate the _intent_ of the algorithm much better than explicit recursion.

For example, the `reverse` example can be written as a fold in at least two ways. Here is a version which uses `foldr`:

```text
> import Data.Foldable

> :paste
… reverse :: forall a. Array a -> Array a
… reverse = foldr (\x xs -> xs <> [x]) []
… ^D

> reverse [1, 2, 3]
[3,2,1]
```

Writing `reverse` in terms of `foldl` will be left as an exercise for the reader.

X> ## Exercises
X>
X> 1. (Easy) Use `foldl` to test whether an array of boolean values are all true.
X> 2. (Medium) Characterize those arrays `xs` for which the function `foldl (==) false xs` returns true.
X> 3. (Medium) Rewrite the following function in tail recursive form using an accumulator parameter:
X>
X>     ```haskell
X>     import Prelude
X>     import Data.Array.Partial (head, tail)
X>     
X>     count :: forall a. (a -> Boolean) -> Array a -> Int
X>     count _ [] = 0
X>     count p xs = if p (unsafePartial head xs)
X>                    then count p (unsafePartial tail xs) + 1
X>                    else count p (unsafePartial tail xs)
X>     ```
X>
X> 4. (Medium) Write `reverse` in terms of `foldl`.

## A Virtual Filesystem

In this section, we're going to apply what we've learned, writing functions which will work with a model of a filesystem. We will use maps, folds and filters to work with a predefined API.

The `Data.Path` module defines an API for a virtual filesystem, as follows:

- There is a type `Path` which represents a path in the filesystem.
- There is a path `root` which represents the root directory.
- The `ls` function enumerates the files in a directory.
- The `filename` function returns the file name for a `Path`.
- The `size` function returns the file size for a `Path` which represents a file.
- The `isDirectory` function tests whether a function is a file or a directory.

In terms of types, we have the following type definitions:

```haskell
root :: Path

ls :: Path -> Array Path

filename :: Path -> String

size :: Path -> Maybe Number

isDirectory :: Path -> Boolean
```

We can try out the API in PSCi:

```text
$ pulp psci

> import Data.Path

> root
/

> isDirectory root
true

> ls root
[/bin/,/etc/,/home/]
```

The `FileOperations` module defines functions which use the `Data.Path` API. You do not need to modify the `Data.Path` module, or understand its implementation. We will work entirely in the `FileOperations` module.

## Listing All Files

Let's write a function which performs a deep enumeration of all files inside a directory. This function will have the following type:

```haskell
allFiles :: Path -> Array Path
```

We can define this function by recursion. First, we can use `ls` to enumerate the immediate children of the directory. For each child, we can recursively apply `allFiles`, which will return an array of paths. `concatMap` will allow us to apply `allFiles` and flatten the results at the same time.

Finally, we use the cons operator `:` to include the current file:

```haskell
allFiles file = file : concatMap allFiles (ls file)
```

_Note_: the cons operator `:` actually has poor performance on immutable arrays, so it is not recommended in general. Performance can be improved by using other data structures, such as linked lists and sequences.

Let's try this function in PSCi:

```text
> import FileOperations
> import Data.Path

> allFiles root

[/,/bin/,/bin/cp,/bin/ls,/bin/mv,/etc/,/etc/hosts, ...]
```

Great! Now let's see if we can write this function using an array comprehension using do notation.

Recall that a backwards arrow corresponds to choosing an element from an array. The first step is to choose an element from the immediate children of the argument. Then we simply call the function recursively for that file. Since we are using do notation, there is an implicit call to `concatMap` which concatenates all of the recursive results.

Here is the new version:

```haskell
allFiles' :: Path -> Array Path
allFiles' file = file : do
  child <- ls file
  allFiles' child
```

Try out the new version in PSCi - you should get the same result. I'll let you decide which version you find clearer.

X> ## Exercises
X>
X> 1. (Easy) Write a function `onlyFiles` which returns all _files_ (not directories) in all subdirectories of a directory.
X> 1. (Medium) Write a fold to determine the largest and smallest files in the filesystem.
X> 1. (Difficult) Write a function `whereIs` to search for a file by name. The function should return a value of type `Maybe Path`, indicating the directory containing the file, if it exists. It should behave as follows:
X>
X>     ```text
X>     > whereIs "/bin/ls"
X>     Just (/bin/)
X>     
X>     > whereIs "/bin/cat"
X>     Nothing
X>     ```
X>
X>     _Hint_: Try to write this function as an array comprehension using do notation.

## Conclusion

In this chapter, we covered the basics of recursion in PureScript, as a means of expressing algorithms concisely. We also introduced user-defined infix operators, standard functions on arrays such as maps, filters and folds, and array comprehensions which combine these ideas. Finally, we showed the importance of using tail recursion in order to avoid stack overflow errors, and how to use accumulator parameters to convert functions to tail recursive form.
