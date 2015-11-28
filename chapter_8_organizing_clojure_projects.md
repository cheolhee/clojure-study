
Chapter 8
---
# Organizing and Building Clojure Projects




## Project Geography



먼저 클로저 프로젝트를 물리적인 파일 배치와 코드베이스상의 기능적인 조직 관점에서 어떤 식으로 조직화할지 정해야 한다. 네임스페이스(이름공간)에 대한 얘기다.


### Defining and Using Namespaces
쉽게 말해 서울 사는 김서방인가, 부산 사는 김서방인가..

namespace는...

* Symbol들의 다이나믹 매핑.
* 간단히 말해 Java의 패키지, Python,Ruby의 모듈과 유사하다.

모든 클로저 코드는 네임스페이스 안에서 정의된다. 네임스페이스 선언을 생략하면 디폴트 **user** 네임스페이스로 매핑된다.

클로저에는 네임스페이스를 위한 다양한 함수와 매크로가 있다.


#### in-ns

def와 그와 유사한 것(defn 같은)들은 현재의 네임스페이스 안에서 var를 선언한다.

```
*ns*
;= #<Namespace user>
(defn a [] 42)
;= #'user/a
```

**in-ns** 를 사용하면 다른 네임스페이스로 전환할 수 있다. ( 존재하지 않은 네임스페이스였다면 새로 생성된다. )

```
(in-ns 'physics.constants)
;= #<Namespace physics.constants>
(def ^:const planck 6.62606957e-34)
;= #'physics.constants/planck
```

하지만 새로 만든 네임스페이스에 뭔가 잘못됐음을 알 수 있다.

```
(+ 1 1)
;= 컴파일 예외 발생!!
```

**+**함수(기타 clojure.core 네임스페이스로 선언된 모든 함수)는 **user** 네임스페이스 있지 않으면 작동하지 않는다. namespace-qualified 심볼이면 접근 가능하다.

```
(clojure.core/range -20 20 4);= (-20 -16 -12 -8 -4 0 4 8 12 16)```


#### refer

네임스페이스가 이미 로드됐다고 가정하고, refer를 사용하면 해당 네임스페이스의 var들을 우리 네임스페이스에서 사용할 수 있게 된다.

```
user/a;= #<user$a user$a@6080669d>(clojure.core/refer 'user)
;= nil(a);= 42
```
이제 a를 로컬에서 선언한 것처럼 쓸 수 있다.


refer는 단순히 임포트를 하는 것보다 더 많은 일을 할 수 있다.
옵셔널 키워드를 사용하면 현재 네임스페이스에 매핑시, 어떤 var를 제외하거나 포함하거나 이름을 바꿀 수 있다.

```
(clojure.core/refer 'clojure.core :exclude '(range)
:rename '{+ add - sub / div * mul})
;= nil
(-> 5 (add 18) (mul 2) (sub 6))
;= 40
(range -20 20 4)
;= #<CompilerException java.lang.RuntimeException:
;= Unable to resolve symbol: range in this context, compiling:(NO_SOURCE_PATH:1)>

```
이제 range를 제외한 clojure.core의 함수를 사용할 수 있고, 연산자 함수는 다른 이름이로 변경됐다.


### require and use

다른 네임스페이스에 선언된 함수나 데이터를 사용하고자 할 때 , require 와 use는 ...

1. 문제의 네임스페이스가 로드됐는지 확인.
2. 필요에 따라 네임스페이스명의 알리아스를 확립.
3. 다른 네임스페이스의 var들을 참조할 수 있게 암묵적으로 **refer** 해 줌.

**require** 는 (1),(2) 를 제공하고, **use** 는 그 위에다가 간편하게 방법으로 (3)을 제공한다. 


REPL을 새로 열어서  집합 함수 union을 사용해 본다.

```
(clojure.set/union #{1 2 3} #{4 5 6});= #<ClassNotFoundException java.lang.ClassNotFoundException: clojure.set>
```

에러 발생. require를 해줘야만 네임스페이스가 로드된다.

```
(require 'clojure.set);= nil
(clojure.set/union #{1 2 3} #{4 5 6}) ;= #{1 2 3 4 5 6}
```

매번 full-qualified 심볼을 입력하려면 아주 번거롭다. 다행히 require는 알리아스 기능이 있다.

```
(require '[clojure.set :as set]) ;= nil(set/union #{1 2 3} #{4 5 6});= #{1 2 3 4 5 6}
```

네임스페이스의 앞부분이 동일하다면 한 번만 입력하는 방법도 가능하다.

```
(require '(clojure string [set :as set]))

```



***use*** 는 require의 모든 기능을 제공하면서 refer까지 시켜주는 것이 다르다. 
(use 'clojure.xml)은 아래 코드에 상당한 동작을 수행한다.

```
(require 'clojure.xml)(refer 'clojure.xml)

```

부가적으로, *** use ***는 모든 파라메터르 ***refer***에 넘기므로, 후자의 :exclude :only :rename 옵션도 그대로 사용할 수 있다. 

```
 # clojure.string/join 만 로드
 # clojure.set 에서 join만 빼고 로드
(use '(clojure [string :only (join) :as str][set :exclude (join)])) ;= niljoin;= #<string$join clojure.string$join@2259a735> 
intersection;= #<set$intersection clojure.set$intersection@2f7fc44f> 
str/trim;= #<string$trim clojure.string$trim@283aa791>

```


#### import

 Java의 클래스나 인터페이스를 현재 네임스페이스에 매핑시 사용한다.

 **java.lang** 패키지는 모든 네임스페이스에 디폴트로 임포트된다.

 한 패키지의 여러 클래스를 임포트하기
 
```
(Date.);= #<CompilerException java.lang.IllegalArgumentException:;= Unable to resolve classname: Date, compiling:(NO_SOURCE_PATH:1)> 
(java.util.Date.);= #<Date Mon Jul 18 12:31:38 EDT 2011>(import 'java.util.Date 'java.text.SimpleDateFormat);= java.text.SimpleDateFormat(.format (SimpleDateFormat. "MM/dd/yyyy") (Date.));= "11/28/2015"

```


```
(import '(java.util Arrays Collections)) 
;= java.util.Collections(->> (iterate inc 0)  (take 5) 
  into-array   Arrays/asList 
  Collections/max);= 4

```

드문 케이스지만, 짧은 클래스명이 동일한 두 클래스를 같은 네임스페이스에 임포트하는 것 불가능하다.

```
(import 'java.awt.List 'java.util.List);= #<IllegalStateException java.lang.IllegalStateException:;= List already refers to: class java.awt.List in namespace: user>

```

자바의 import에서 가능한 와일드 카드 지정은 클로저에서 지원하지 않는다.


#### ns

in-ns, refer, require, use, import 를 전부 아우르는 매크로이다.

다음 함수 호출들은 

```
(in-ns 'examples.ns)(clojure.core/refer 'clojure.core :exclude '[next replace remove]) (require '(clojure [string :as string]           [set :as set])           '[clojure.java.shell :as sh])(use '(clojure zip xml))        (import 'java.util.Date               'java.text.SimpleDateFormat                '(java.util.concurrent Executors                                       LinkedBlockingQueue))
```

아래의 ***ns*** 선언과 동일한 효과를 낸다.

```
(ns examples.ns
  (:refer-clojure :exclude [next replace remove]) 
  (:require (clojure [string :as string]
                     [set :as set]) 
            [clojure.java.shell :as sh])
  (:use (clojure zip xml)) 
  (:import java.util.Date
           java.text.SimpleDateFormat 
           (java.util.concurrent Executors
                                 LinkedBlockingQueue)))

```



### Namespaces and files


클로저 소스 파일을 어떻게 조직화 해야 하는지에 대한 몇가지 엄격의 규칙이 있다.


#### Use one file per namespaces.

각 네임스페이스는 별개의 파일로 존재 해야하고 네임스페이스 세그먼트의 폴더에 있어야 한다.

com.mycompany.foo라는 네임스페이스는 com.mycompany 폴더에 foo.clj 파일로서 있어야 한다. require 혹은 use 로 이 네임스페이스를 로드할 때 해당 폴더에 foo.clj 파일이 없으면 에러가 발생한다.


( 
이 부분은 Java의 패키지와 다른 점인데, Java패키지는 디렉토리 구조만을 뜻하기 때문이다.
com.mycompany.foo 라는 패키지가 있다면 com/mycompany/foo라는 디렉토리가 있어야 한다. 클래스 파일은 마지막 폴더 안에 위치하게 된다.
)



#### Use underscore in filenames when namespaces contain dashes.

네임스페이스의 '-' 는 파일명에서는 '_' 로 바꿔야 한다.
클로저는 관행상 대쉬'-'를 사용하지만, JVM은 클래스명이나 패키지명에 대쉬를 허용하지 않으므로 언더스코어'_'로 바꿔줘야 한다.


#### Start every namespace with a comprehenshive ns form.

.clj 소스 시작은 ns 폼으로 시작할 것.

#### Avoid cyclic namespace dependencies.

순환 관계의 네임스페이스 종속 관계는 만들지 말 것. Java패키지와는 다른 점이다.



#### Use declare to enable forward references.

클로저는 네임스페이스 파일 안에서 각 폼을 순서대로 로드하는데 이때 앞에서 선언된 var 를 참조해나가면서 진행된다. 따라서 define돼 있지 않은 var를 참조하면 에러가 난다.


```
(defn a [x] (+ constant (b x)));= #<CompilerException java.lang.RuntimeException:;= Unable to resolve symbol: constant in this context, compiling:(NO_SOURCE_PATH:1)>
```

declare 를 사용하면 에러를 피할 수 있다.

```
(declare constant b);= #'user/b(defn a [x] (+ constant (b x))) ;= #'user/a(def constant 42);= #'user/constant(defn b [y] (max y constant)) ;= #'user/b(a 100);= 142

```

#### Avoid single-segment namespaces.

네임스페이스는 몇 개의 세그먼트로 구성된다. 예를 들어 com.my-project.foo 는 3개의 세그먼트가 있다.

싱글 세그먼트 네임스페이스는 만들 수는 있지만, 쓰지 않는 것이 좋다.

( 자바에서도 마찬가지로 안 쓰는 것이 좋습니다. F/W가 제대로 동작하지 않는 경우도 있음. )



### A classpath primer

클래스패스는 Java에 익숙하지 않은 프로그래머라면 혼동의 도가니일 수도 있다.
classpath는 이해하기 어럽고 쓸 일은 없을지라도 알고는 있어야 한다.


#### Defining the classpath.


클래스패스를 설정하려면 커맨드라인에서 -cp 플래그와 함께 적는다.

```
java -cp '.:src:clojure.jar:lib/*' clojure.main
```

#### Classpath and the REPL.

클래스패스는 런타임에서도 조회할 수 있다.

```
$ java -cp clojure.jar clojure.main Clojure 1.3.0(System/getProperty "java.class.path") ;= "clojure.jar"
```

클래스패스는 JVM실행시 환경변수나 파라메터로 전달되는데, 런타임시에 바꿀 수는 없다.

(과거 add-classpath 함수가 있었으나 deprecated됐음.)


### Location, Location, Location

두가지 프로젝트 레이아웃 컨벤션이 있다.

1. Maven 스타일

소스 파일은 전부 src 폴더 밑으로 넣고, 언어와 역할에 따라 세부 폴더로 나눈다.

src/main : 공개 API나 외부 배포될 소스
src/test : 굳이 배포할 필요는 없는 소스

```
<project dir> 
||- src    |- main       |- clojure        |- java       |- resources        |- ...    |- test       |- clojure       |- java       |- resources        |- ...

```

클로저 소스는 src/main/clojure 폴더에 자바소는 src/main/java 둔다. 이렇게 파일 타입별로 디렉토리 구조를 구분하면 여러가지 처리작업을 더 쉽게 만들 수 있다.

단점은 경로가 길어진다는 점.


2. 프리 폼
이름을 정하기에는 프로젝트마다 다양한 구조.

```
<project dir> 
||- src |- test
<project dir> ||- src   |- java   |- clojure|- test|- resources |- web
```

Maven 스타일보다 디렉토리 경로가 단순. 파일타입이 섞일 수도 있다.
Maven이 아닌 다른 개발툴, Leiningen을 포함하여 이런 스타일의 프로젝트 레이아웃을 사용한다.



### The Functional Organization of Clojure Codebases

지금까지는 기계적인 룰에 대해 논의했다. 파일배치, 명명법, 네임스페이스와 파이명과의 관계 등.

더욱 미묘한 점은 함수적 관점에서 클로저 프로젝트를 어떻게 조직화 할 것인가다.


* 한 알고리즘 구현에는 몇 개의 함수가 사용돼야 하는가?
* 한 네임스페이스에는 몇 개의 함수가 포함돼야 하는가?
* 한 프로젝트에는 몇 개의 네임스페이스가 포함돼야 하는가?

다른 언어에 대한 질문이라면, 명시적으로 "좋은 스타일"을 정의하는 특정한 요건들이 있으므로 대답하기가 쉬운 편이다. 많은 다른 프로그래밍 언어의 프레임워크들은 플러그인/컴포넌트/익스텐션이 어떤 식으로 정의될지 결정돼 있는 편이다.

반면, 클로저에서는 특정 라이브러리나 프레임워크를 사용하는 부작용으로 어플리케이션이 특정한 구성을 갖춰야 하는 경우는 적다. 

아주 일반적은 원치을 제외하고 클로저 프로그램의 "젼형적인 구조"라는 것은 딱히 존재하지 않는다. 이것은 사람에 따라 괜찮게도 황당하게도 들릴 수 있겠다. 우리의 경우는, 문제풀이와 무관한 일에 정신이 산만해질 필요 없이, 기능이나 알고리즘, 도메인 자체에 집중할 수 있게 된다는 것을 알게 됐다.


#### Basic project organization principles

* 다른 것은 분리한다. 가급적 다른 네임스페이스로.

* 관계 있는 것은 같이 두고, 무리 없는 카테고리로 그룹핑한다.

* 세부적 구현에 관련된 데이터나 함수가 포함된 var는 private로 선언할 것.

* Don't Repear yourself

* 가능하면 레퍼런스 타입, 컬렉션, 시퀀스의 추상화한 구현을 사용하라. 그들 중 구체적인 구현 어느 것을 골라 사용하기보다는.
