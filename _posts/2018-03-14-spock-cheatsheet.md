---
date: 2018-03-14T19:12:33+00:00
category: tdd
---


#  SPOCK, SPOCK-GENESIS

# Reference
> https://stackoverflow.com/questions/2665812/what-is-mocking
> 
> https://semaphoreci.com/community/tutorials/stubbing-and-mocking-in-java-with-the-spock-testing-framework
> 
> http://spockframework.org/spock/docs/1.0/interaction_based_testing.html
> 
> http://spockframework.org/spock/docs/1.1/spock_primer.html
> 
> http://groovy-lang.org/style-guide.html

 

## 구조

테스트 케이스 작성을 위해 Spock Specificiation을 상속받아 구현하면 된다.
 
```groovy
class MyTest extends Specification {
  // fields
  // fixture methods
  // feature methods
  // helper methods
}

```

### Fields

```groovy
def obj = new ClassUnderSpecification()
def coll = new Collaborator()
/* feature method 끼리 공유하지 않음 */


@Shared def res = new VeryExpensiveResource()
/* feature method 끼리 공유함 */
// setupSpec과 동일하게 한번만 설정되므로 이후 변경 불가능

static final PI = 3.141592654
/* 상수를 나타냄, 공유됨 */

// Shared와 static의 차이는 없지만 상수일 경우에만 static 사용을 권장
```

### Fixture methods

Feature method에서 공용으로 사용하는 객체등을 설정하는 부분
```groovy
def setup() {}          
// feature method가 실행되기 전 매번 실행
def cleanup() {}
// feature method가 실행된 뒤 매번 실행
def setupSpec() {}
// 첫 feature method가 실행되기 전 한번 실행
def cleanupSpec() {}
// 마지막 feature method가 실행 후 한번 실행
```

### Feature methods

예상한대로 기능이 정상적으로 작동하는지 확인하는 각 케이스를 의미


#### Phase

 **Feature Method의 기본 컨셉**

> `Setup` ->  `Stimulus` -> `Response` -> `CleanUp`


1.Setup : 기능 실행을 위한 환경 설정

2.Stimulus : 기능 실행, **필수** 

3.Response: 결과값이 예상과 맞는지 검증, **필수**  

4.CleanUp: 설정한 환경 초기화 

#### Block

given: when, then... etc

 Spock에서 `Phase` 를 구현한  즉, *Feature Method 구성하는 키워드들*

 >`setup`, `when`, `then`, `expect`, `cleanup`, `where` 


![ Block과 Phase와 관계를 나타낸 그림](https://user-images.githubusercontent.com/14921979/38843319-606e45dc-4229-11e8-9df2-e486a081542b.png)

 *Block과 Phase와 관계를 나타낸 그림*

사용 예시

```groovy
/* setup, when, then, cleanUp */
def "set Hello Message to myFuture"() {
  setup: // Setup
	def myFuture = new MyFuture()
	
  when: // stimulus
	myFuture.setHelloMessage("Hello Future")
	
  then: // response
	myFuture.helloMessage == "Hello Future"
  
  cleanUp: // cleanUp
   myFuture.destory()
}

/* expect, where */
def "computing the maximum of two numbers"() {
  // stimulus + response, when + then
  expect: 
  
  Math.max(a, b) == c

  where: // 맨 마지막에 와야됨, Data-Driven-Testing에 주로 사용
  a << [5, 3]
  b << [1, 9]
  c << [5, 9]

// Math.max(5,1) == 5
// Math.max(3,9) == 9
}

```

#### Then
`when:` 절의 조건에서 실행한 로직을  `then:` 절에서 검증할때 사용하는 방법들

Mock 아래로 옮겨야한다


##### 실행 순서

아래 코드는 ` hello hello goodbye ` 순서대로 실행될것으로 예상하며 작성한 코드다. 

```groovy
then:
2 * subscriber.receive("hello")
1 * subscriber.receive("goodbye")
```
`hello hello goodbye` 순서대로 결과가 나올것 같지만 

- "hello" "hello" "goodbye"
- "hello" "goodbye" "hello"
- "goodbye" "hello" "hello"

같은 무작위 순서로 실행되게 된다. 

의도한대로 실행되게 수정한 코드이다.

```groovy
then:
2 * subscriber.receive("hello")

then:
1 * subscriber.receive("goodbye")

```

##### Cardinality 

```groovy
1 * subscriber.receive("hello")      // 한번 실행되야 함
0 * subscriber.receive("hello")      // 실행되면 안됨
(1..3) * subscriber.receive("hello") // 1~3번 실행되야 함
(1.._) * subscriber.receive("hello") // 최소 1번은 실행되야함
(_..3) * subscriber.receive("hello") // 최대 3번 실행될 수 있음
_ * subscriber.receive("hello")      // 실행 횟수 상관없음
```



##### Target Constraint 
```groovy
1 * subscriber.receive("hello")
// subscriber(Mock)의 receive 실행시켰다
1 * _.receive("hello")
// Mock Object 중 하나가 receive("hello")를 실행시켰다.
// (subscriber 포함) 
```

인자, 이름, 인스턴스 상관없이 메소드를 실행시킨지 확인

```groovy
1 * subscriber._(*_)
// subscriber의 인자로 리스트를 넘겨 메소드중 하나를 실행시켰을때

1 * subscriber._
// shortcut for and preferred over the above

1 * _._                  
// any method call on any mock object
1 * _                    
// shortcut for and preferred over the above
```


 
##### Method Constraint

```groovy
1 * subscriber.receive("hello") 
// recieve 메소드를 실행시켰다.
1 * subscriber./r.*e/("hello")  
// 메소드 이름이 정규 표현식 매칭된 메소드를 실행시켰다.
// (r로 시작하고 e로 끝나는 메소드 실행)

```

##### Argument Constraint 
메소드 인자로  어떤값을 받은지 체크

```groovy
1 * subscriber.receive("hello")     
// argument로 "hello"를 받아 실행했다.

1 * subscriber.receive(!"hello")
// argument로 "hello" 제외한 다른 값을 받아 실행했다.

1 * subscriber.receive()
// argument 없이 실행했다.

1 * subscriber.receive(_)
// argument 값 상관없이 '한개' 를 받아 실행했다. (null 포함)

1 * subscriber.receive(*_)
//  여러개의 argument를 받았다. (인자 값이 비어있어도 상관없음)

1 * subscriber.receive(!null)
// argument로 null을 제외한 다른 값을 받아 실행했다.

1 * subscriber.receive(_ as String)
// argument로 String형 값을 하나 받았다. (null 제외)

1 * subscriber.receive({ it.size() > 3 }) 
// argument를 3개 이상 받았다.
// it는 argumentList의 이터레이터

/* 여러개를 동시에 사용할수도 있다. */

1 * process.invoke(
"ls",
"-a",
_,
!null,
{["abcdefghiklmnopqrstuwx1"].contains(it) })
// 첫번쨰: "ls"
// 두번째: "-a"
// 세번째: 아무거나
// 네번째: 네번째 값이 "abcdefghiklmnopqrstuwx1"다

```




### Helper methods

`Feature method` 여러개가 공용으로 사용하는 메소드를 의미한다

```groovy
def "Say Hello to the future"() {
  given:
	def myFuture = new MyFuture()
  when:
	myFuture.setHelloMessage("Hello Future")
  then:
	checkIsHelloFuture(myFuture.helloMessage)
}

def "should not say Hello to the future"() {
  given:
	def myFuture = new MyFuture()
  when:
	myFuture.setHelloMessage("Hello Future")
  then:
	checkIsHelloFuture(myFuture.helloMessage)
}


// helper methods
def checkIsHelloFuture(helloMessage) {
	assert helloMessage == "Hello Future"
}
```


## 어노테이션

### Narrative
[what's in a story?](https://dannorth.net/whats-in-a-story/)

[offical-spock-story-example](https://github.com/spockframework/spock/blob/groovy-1.8/spock-report/src/test/groovy/org/spockframework/report/sample/FightOrFlightStory.groovy)

> As a `[role]`
> I want `[feature]`
> so that `[benefit]`

### Title
### Issue
### Ignore
### See

## Collaborators
[Mocks Aren't Stubs](https://martinfowler.com/articles/mocksArentStubs.html)

위의 글에서 마틴 파울러


### Mock

유닛 테스트 작성시 테스트 하려는 객체가 다른 객체와 상호작용하는 경우가 대부분인다. (없다면 의심해봐야한다...; god-class가 아닌지)

만약 테스트 대상 객체와 상호작용하는 객체가 DAO면 테스트가 곤란할뿐더러 각 테스트 실행시마다 결과가 달라질 수 있다.

유닛 테스트의 목적은 코드 품질 향상을 위한 테스트 대상 객체 하나의 메소드 단위 검증이지, 실제 기능 작동 여부를 검증하려고 하는것이 아니므로 테스트 대상 객체와 상호작용하는 객체를 더미 객체로 바꿔친다.

이때 사용하는 더미 객체를 **Mock** 이라 부른다.
 
  
실행결과보단 해당 오브젝트가 몇번 실행되는지 확인하는 용도로 쓰기에 적합하다.
(TODO: 수정필요)



#### 사용법

- Mock Object의 메소드 호출시 리턴형의 기본값을 리턴함
- Mock Object는 자기 자신과만 같음 
(hashcode, 타입을 String으로 표시한 값이 추가됨)


```groovy
def subscriber = Mock(Subscriber)
def subscriber2 = Mock(Subscriber)

//OR

Subscriber subscriber = Mock()
Subscriber subscriber2 = Mock()

```

```groovy
// Mock object 생성 따로, 상호작용 설정 따로
class PublisherSpec extends Specification {
    Publisher publisher = new Publisher()
    Subscriber subscriber = Mock()
    Subscriber subscriber2 = Mock()
    
    def setup() {
        publisher.subscribers << subscriber
        // 실제 오브젝트의 맴버 변수(리스트)에 Mock Object 주입
        publisher.subscribers << subscriber2
    }
}

```

```groovy
// Mock 객체 생성시 상호작용까지 한번에 설정
def subscriber = Mock(Subscriber) {
   1 * receive("hello")
   1 * receive("goodbye")
}

```

```groovy
// 상호작용 대상 Mock Object가 동일한 경우 한번에 설정
with(subscriber) {
    1 * receive("hello")
    1 * receive("goodbye")
}
```



```groovy

def "should send messages to all subscribers"() {
    when:
    publisher.send("hello")
    // 실제 오브젝트에서 send("hello") 호출시 subscriber.recieve("hello")를 호출함

    then:
    1 * subscriber.receive("hello")
	  // publihser.send("hello") 호출시
	  // subscriber.recieve("hello")가 한번 호출되야함
    1 * subscriber2.receive("hello")
}
```

### Stub

결과값을 직접 주입하고 싶을때

```groovy
// 단일값
subscriber.receive(_) >> "ok"
// ok
```

```groovy
// 여러값
subscriber.receive(_) >>> ["ok", "fail", "ok"]
// ok, fail, ok

// 섞어쓰기
subscriber.receive(_) >>> ["ok", "fail", "ok"] >> { throw new InternalError()} >> "oops Exception"
// ok, fail, ok, Exception, oops Exception
```

### Spy

Mock 객체의 메소드는 기본값만 반환하는 반면에 Spy는 실제 객체를 가져와 Stubbing도 할 수 있고 Mock처럼 실행 횟수 체크도 가능하다.

spock 공식 문서에서는 Spy를 사용해야될것 같은 상황을 code smell이라고 가급적이면 사용하지 않을것을 권장하고 있다.

보통 해당 객체가 2가지 이상의 책임(역할)을 가지고 있어 분리가 필요한 상황이지만, 분리가 안되었을 경우 사용한다.


### Mock + Stub

Mock Object의 메소드들은 더미값을 뱉으므로 의미있는 값이 필요할땐 섞어 써보자

```groovy
setup:
subscriber.receive("message1") >> "ok"

when:
publisher.send("message1")

then:
1 * subscriber.receive("message1")

// subscriber.receive("message1")을 실행시켰을 경우 ok를 반환하도록 변경
```

## SPOCK-GENESIS

[https://github.com/Bijnagte/spock-genesis](https://github.com/Bijnagte/spock-genesis)

spock의 데이터 제너레이터. 벌크 데이터를 만들어주는 역할

```groovy
 def 'complex pogo'() {
        expect:
            person instanceof Person
            person.gender in ['M', 'F', 'T', 'U'].collect { it as char }
            person.id > 199
            person.id < 10001
            person.birthDate >= Date.parse('MM/dd/yyyy', '01/01/1980')
            person.birthDate <= new Date()

        where:
            person << type(Person,
                    id: integer(200..10000),
                    name: string(~/[A-Z][a-z]+( [A-Z][a-z]+)?/),
                    birthDate: date(Date.parse('MM/dd/yyyy', '01/01/1980'), new Date()),
                    title: these('', null).then(Gen.any('Dr.', 'Mr.', 'Ms.', 'Mrs.')),
                    gender: character('MFTU')
            ).take(3)
    }


@Iterations(2)
    def 'limiting iterations to 2 makes it so the first 2 iterations are all that run'() {
        expect:
        s instanceof String
        i instanceof Integer
        i < 3
        where:
        s << string(~/[A-Z][a-z]+( [A-Z][a-z]+)?/)
        i << these(1,2,3,4,5,6)
    }
```

### 랜덤 Map 만들기

```groovy
def 'generate a map'() {
    when: 'defining a map with different fields'
        def myMap = map(                            
            id: getLong(),                          
            name: string,                           
            age: integer(0, 120)
            ).iterator().next() 

    then: 'we should get instances of map'
        myMap instanceof Map

    and: 'the fields should follow the generators rules'
        myMap.id instanceof Long
        myMap.name instanceof String
        myMap.age instanceof Integer
}
```

### these

```groovy
def 'generate from a specific set of values'() {
    expect: 'to get numbers from a varargs'
        these(1,2,3).take(3).collect() == [1,2,3]

    and: 'to get values from an iterable object such as a list'
        these([1,2,3]).take(2).collect() == [1,2]

    and: 'to get values from a given class'
        these(String).iterator().next() == String

    and: 'to stop producing numbers if the source is exhausted'
        these(1..3).take(10).collect() == [1,2,3]
}
```

### then, &

```groovy
def 'generate from multiple iterators in sequence'() {
    setup:
        def gen = these(1, 2, 3).then([4, 5])
    expect:
        gen.collect() == [1, 2, 3, 4, 5]
}
```

### *

```groovy
def 'multiply by int limits the quantity generated'() {
    setup:
        def gen = string * 3
    when:
        def results = gen.collect()
    then:
        results.size() == 3
}
```

### seed

```groovy

def 'setting seed returns the same values with 2 generators configured the same'() {
    given:
        def generatedA = string(10).seed(879).take(10).realized
        def generatedB = string(10).seed(879).take(10).realized
    expect:
        generatedA == generatedB
}



def 'setting seed to different values produces different sequences'() {
    given:
        def generatedA = integer.seed(879).take(4).realized
        def generatedB = integer.seed(3).take(4).realized
    expect:
        generatedA == [-1295148427, 2105117961, -922763979, 1733784787]
        generatedB == [-1155099828, -1879439976, 304908421, -836442134]
}

```

### with

```groovy
// setter?
def 'call methods on generated value using with'() {
    setup:
        def gen = date.with { setTime(1400) }

    expect:
        gen.iterator().next().getTime() == 1400
}

```

### any

```groovy
//generate values by randomly order
def 'generate any value from a given source'() {
    given: 'a source'
        def source = [1,2,null,3]

    expect: 'only that the generated value is any of the elements'
        Gen.any(source).take(2).every { n -> n in source }
}
```

### take
```groovy
Gen.string(1, 10).take(5)
```

