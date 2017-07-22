# 처음 시작하기

## 이 장의 목표

이 장의 목표는 PureScript 개발 환경을 설정하고 첫 번째 PureScript 프로그램을 작성해 보는 것이다.

첫 프로젝트로 우리는 PureScript 라이브러리를 만들어 본다. 직각 삼각형의 빗변 길이를 계산하는 함수 하나로 구성된 매우 단순한 라이브러리다.

## 필요한 도구들

PureScript 개발 환경을 설정하기 위해 사용할 도구들은 다음과 같다.

- [`purs`](http://purescript.org) - PureScript 컴파일러.
- [`npm`](http://npmjs.org) - Node 패키지 매니저. 다른 도구들을 설치하는 데 필요하다.
- [Pulp](https://github.com/bodil/pulp) - PureScipt 프로젝트를 관리하기 위한 여러가지 작업들을 자동화하는 명령줄 도구.

이제 도구를 설치하고 설정하는 과정을 살펴보자.

## PureScript 설치하기

PureScript 컴파일러는 [PureScript 웹사이트](http://purescript.org)에서 배포 중인 플랫폼 별 바이너리를 다운로드하여 설치하는 방법을 추천한다.

컴파일러를 실행 파일 경로에 추가한 다음 명령줄에서 실행해보자.

```text
$ purs
```

컴파일러를 설치하는 다른 방법도 있다.

- npm 이용: `npm install -g PureScript`.
- 소스로 직접 빌드하기. PureScript 웹사이트에 방법이 설명되어 있다.

## 도구들 설치하기

`npm`을 사용할 것이기 때문에 Node.js가 이미 설치되어 있어야 한다.

`npm`을 이용하여 Pulp 명령줄 도구와 Bower 패키지 관리자를 설치한다.

```text
$ npm install -g pulp bower
```

이제 `pulp`와 `bower` 명령이 여러분의 실행 파일 경로에 추가되었다. 이것만으로
PureScript 프로젝트를 생성할 도구가 모두 갖춰졌다.

## Hello, PureScript!

단순한 것부터 시작하자. Pulp 명령줄 도구를 이용하여 Hello World! 프로그램을 컴파일하고 실행한다.

빈 디렉토리부터 만들고 여기서 `pulp init` 명령을 실행한다.

```text
$ mkdir my-project
$ cd my-project
$ pulp init

* Generating project skeleton in ~/my-project

$ ls

bower.json	src		test
```

Pulp는 `src`와 `test` 디렉토리와 `bower.json` 설정 파일을 만들어낸다.
`src` 디렉토리에는 소스 파일을, `test` 디렉토리에는 테스트를 둘 것이다.
`test` 디렉토리는 이 책 뒷 부분에서 사용할 것이다.

`src/Main.purs` 파일을 아래처럼 수정한다.

```haskell
module Main where

import Control.Monad.Eff.Console

main = log "Hello, World!"
```

프로그램은 몇 줄 되지 않지만 몇 가지 핵심 아이디어를 엿볼수 있다.

- 파일 첫 줄은 모듈 헤더로 시작한다. 모듈 이름은 대문자로 시작하며, 여러 단어로 구성할 때는 점으로 구분한다. 여기서는 한 단어로 되어 있지만 `My.First.Module`도 모듈 이름으로 사용할 수 있다.
- 모듈을 임포트할 때는 전체 이름을 사용한다. 여기서는 `log` 함수를 사용하기 위해 `Control.Monad.Eff.Console` 모듈을 임포트했다.
- `main` 프로그램의 정의를 보면 함수를 적용하는 모양이 보인다. PureScipt는 함수명과 인자 사이를 공백으로 띄워서 함수를 호출한다.

위의 코드는 다음 명령으로 빌드하고 실행시켜 볼 수 있다.

```text
$ pulp run

* Building project in ~/my-project
* Build successful.
Hello, World!
```

짝짝짝~ 여러분이 PureScript로 첫 번째 프로그램을 컴파일하고 실행시킨 것을 축하한다.

## 브라우저 용으로 컴파일하기

Pulp는 PureScript 코드를 웹 브라우저에서 사용하기 위한 형태의 JavaScript로 바꿔주기도 한다. `pulp browserify` 명령을 이용하면 된다.

```text
$ pulp browserify

* Browserifying project in ~/my-project
* Building project in ~/my-project
* Build successful.
* Browserifying...
```

위 명령의 실행 결과로 많은 양의 JavaScript 코드가 출력될 것이다. 이 출력 결과는 
[Browserify](http://browserify.org/)가 출력하는 내용이다. Browserify는  _Prelude_라고 하는 PureScript의 표준 라이브러리를 포함하여 `src` 디렉토리 내용을 변환한다.
이렇게 생성된 JavaScript 코드를 파일에 저장하거나 HTML 문서에 포함시킬 수 있다.
이 프로그램의 경우에는 브라우저 콘솔에 "Hello, World!"를 출력할 것이다.

## 죽은 코드 제거하기

`pulp build` 명령도 있다. `-O` 옵션과 함께 이 명령을 사용하면 **죽은 코드 제거** 기능이 동작한다. 최종 결과물에 불필요한 코드를 제거하는 기능이다. 이 기능을 사용하면 빌드 결과 JavaScript 코드가 훨씬 작아진다.

```text
$ pulp build -O --to output.js

* Building project in ~/my-project
* Build successful.
* Bundling Javascript...
* Bundled.
```

이렇게 생성된 코드는 HTML 문서에 추가하여 사용할 수 있다. `output.js` 파일을 열어보면
다음과 비슷한 형태로 컴파일된 모듈을 볼 수 있다.

```javascript
(function(exports) {
  "use strict";

  var Control_Monad_Eff_Console = PS["Control.Monad.Eff.Console"];

  var main = Control_Monad_Eff_Console.log("Hello, World!");
  exports["main"] = main;
})(PS["Main"] = PS["Main"] || {});
```

컴파일된 JavaScript 코드를 보면 PureScript 컴파일러가 어떤 식으로 JavaScript 코드를 생성하는지 엿볼 수 있다.

- 모듈은 오브젝트 하나로 변환된다. 래퍼 함수에 의해 생성되는 이 오브젝트는 모듈이 외부로 노출하는 값들을 멤버로 가진다.
- PureScript는 원래의 이름을 가능한 유지한다.
- PureScript 함수 적용은 JavaScript 함수 적용(함수 호출)로 변환된다.
- 모든 모듈이 정의된 다음에 아무런 인자 없이 `main` 함수가 실행된다.
- PureScript 코드는 런타임 라이브러리 의존성이 없다. 컴파일러가 생성하는 코드는 모두 다른 PureScript 모듈들에서 비롯된다.

PureScript가 생성하는 코드는 단순하고, 읽고 이해할 수 있는 수준이다. 실제로 코드를 생성하는 과정은 껍데기를 단순 변환하는 것에 가깝다. PureScript 언어에 대해 깊이 알지 못하더라도 어떤 식으로 JavaScript 코드가 생성될지 예상하는 건 어렵지 않다.

## CommonJS 모듈로 컴파일하기

Pulp는 CommonJS 모듈을 생성할 수도 있다. Node로 개발하는 경우나 큰 프로젝트에서 CommonJS 모듈 스타일로 컴포넌트를 나누는 경우에 유용하다.

CommonJS 모듈로 빌드하려면 `-O` 옵션 없이 `pulp build` 명령을 사용한다.

```text
$ pulp build

* Building project in ~/my-project
* Build successful.
```

생성된 모듈은 따로 지정하지 않으면 `output` 디렉토리에 생성된다. PureScript 모듈 하나가 CommonJS 모듈 하나로 컴파일되어 각자의 하위 디렉토리 아래에 저장된다.

## Bower로 의존성 추적하기

이 장의 목표였던 `diagonal` 함수를 작성하려면 제곱근을 계산할 수 있어야 한다.
`purescript-math` 패키지에는 JavaScrpt의 `Math` 객체에 정의된 함수들에 대한 타입이 정의되어 있다. 패키지를 설치해보자.

```text
$ bower install purescript-math --save
```

`--save` 옵션을 사용하면 의존성이 `bower.json` 설정 파일에 추가된다.

`purescript-math` 라이브러리의 소스는 `bower_components` 디렉토리에 추가되며,
여러분의 프로젝트를 컴파일할 때 포함된다.

## 빗변 길이 계산하기

`diagonal` 함수를 작성해보자. 외부 라이브러리를 이용하는 예제가 될 것이다.

우선 `Math` 모듈을 임포트해야 한다. `src/Main.purs` 파일의 상단에 다음 줄을 추가한다.

```haskell
import Math (sqrt)
```

`Prelude` 모듈도 임포트해야 한다. `Prelude` 모듈에는 덧셈, 곱셈과 같은
매우 기본적인 연산들이 정의되어 있다.

```haskell
import Prelude
```

이제 `diagnoal` 함수를 다음처럼 정의한다.

```haskell
diagonal w h = sqrt (w * w + h * h)
```

방금 작성한 함수에 타입을 정의할 필요가 없다. 컴파일러가 `diagonal` 함수의 정의로부터
두 개의 수를 입력받아 다시 수를 반환한다는 사실을 유추해 낼 수 있기 때문이다.
하지만 문서화라는 측면에서 타입 정의를 덧붙이는 것이 좋은 습관이다. 

이제 `main` 함수를 수정하여 방금 작성한 `diagonal` 함수를 사용해보자.

```haskell
main = logShow (diagonal 3.0 4.0)
```

프로젝트를 컴파일하고 실행시키기 위해 `pulp run` 명령을 사용한다.

```text
$ pulp run

* Building project in ~/my-project
* Build successful.
5.0
```

## 인터랙티브 모드에서 테스트하기

PureScript 컴파일러는 PSCi라고 하는 인터랙티브 REPL도 제공한다.
여러분의 코드를 테스트하거나 새로운 아이디어를 실험해보고자 할 때 매우 유용하다.
PSCi를 이용하여 `diagonal` 함수를 테스트해 보자.

`pulp psci` 명령으로 REPL을 시작한다. Pulp는 소스 모듈을 자동으로 읽어들인다.

```text
$ pulp psci
>
```

`:?`를 입력하면 사용가능한 명령의 목록을 보여준다.


```text
> :?
The following commands are available:

    :?                        Show this help menu
    :quit                     Quit PSCi
    :reset                    Reset
    :browse      <module>     Browse <module>
    :type        <expr>       Show the type of <expr>
    :kind        <type>       Show the kind of <type>
    :show        import       Show imported modules
    :show        loaded       Show loaded modules
    :paste       paste        Enter multiple lines, terminated by ^D
```

탭 키를 누르면 여러분의 코드와 Bower 의존성, 그리고 Prelude 모듈에 정의된 모든 함수들을 목록으로 보여준다.

우선 `Prelude` 모듈을 임포트한다.

```text
> import Prelude
```

이제 수식을 입력하여 계산해 볼 수 있다.

```text
> 1 + 2
3

> "Hello, " <> "World!"
"Hello, World!"
```

PSCi 상에서 `diagonal` 함수를 사용해보자.

```text
> import Main
> diagonal 5.0 12.0

13.0
```

PSCi 상에서 함수를 정의할 수도 있다.


```text
> double x = x * 2

> double 10
20
```

지금 입력하는 내용들의 문법을 다 이해하지 못하더라도 걱정하지 마라. 책을 읽어가면서 금방 이해할 수 있게 된다.

마지막으로 `:type` 명령을 이용하면 여러분이 입력한 표현식(expression)의 타입을 확인할 수도 있다.

```text
> :type true
Boolean

> :type [1, 2, 3]
Array Int
```

인터랙티브 모드에서 이것저것 시도해보라. 하다가 막히면 `:reset` 명령으로 이미 로드된 모든 모듈들을 해지시킬 수 있다.

> ## 연습 문제
>
> 1. (쉬움) `pi` 상수를 사용해보라. `Math` 모듈에 정의되어 있다. 주어진 반지름으로 원의 넓이를 계산하는 함수 `circleArea`를 작성해보라. 작성한 함수를 PSCi로 테스트해보라. (**힌트**: `import Math` 문장을 수정하여 `pi`를 임포트할 수 있다.)
> 1. (보통) `bower install` 명령으로 `purescript-globals` 패키지를 의존성으로 설치해보라. 패키지 안의 함수들을 PSCi로 테스트해보라. (**힌트**: PSCi 모드에서 `:browse` 명령을 이용하면 특정 모듈의 내용을 살펴볼 수 있다.)

## 결론

이 장에서 우리는 Pulp 도구를 이용하여 간단한 PureScript 프로젝트를 생성하였다.

처음으로 PureScript 함수를 하나 작성해보았다. 이렇게 만든 프로그램은 브라우저나 Node에서 이용할 수 있는 JavaScript 프로그램으로 컴파일하여 실행할 수 있다.

이 장에서 설정한 개발 환경을 이용하여 앞으로 우리가 만들 코드를 컴파일하고 빌드하고 테스트할 것이다. 그러니 여기서 살펴본 도구나 방법들에 익숙해지기 바란다.
