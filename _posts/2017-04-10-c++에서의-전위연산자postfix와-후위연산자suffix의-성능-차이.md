---
date: 2017-04-10T16:06:41+00:00
categories:
  - c++

---
[원본글:  Is there a performance difference between i++ and ++i in C++?](http://stackoverflow.com/questions/24901/is-there-a-performance-difference-between-i-and-i-in-c)

**요약 : 경우에 따라 다르지만, 특별한 이유가 없다면 i++ 대신 ++i를 쓰세요**

[우선 C부터 알아보자면](http://stackoverflow.com/questions/24886/is-there-a-performance-difference-between-i-and-i-in-c/24887#24887), 컴파일러와 시스템에 따라 다르지만 대부분 차이는 없다.  하지만 사실은 잠재적(?)으로 i++이 ++i보다 느리다. [^1]

[^1]: 사실 뇌피셜이다 (사실 각주 테스트다ㅎ)

왜냐하면 후위연산이 이루어지기 전의 값을 저장하기 위해 임시변수를 만들기 때문이다. 하지만 요즘 컴파일러들은 최적화 옵션이 켜져 있으면,

자동으로 최적화하여 결과물은 전위든, 후위든 똑같이 나온다. 또한 아래 답변에선

> So the question one should be asking is not which of these two operations is faster, it is which of these two operations expresses more accurately what you are trying to accomplish. I submit that if you are not using the value of the expression, there is never a reason to use i++ instead of ++i, because there is never a reason to copy the value of a variable, increment the variable, and then throw the copy away.

뭐가 더 빠른지말고, 어떤 목적에 맞게 써야하는지 아는게 더 적절하다고 말하고 있다.

나중에 값을 사용하지 않을거면 굳이 ++i 대신 i++를 사용하여 값을 임시변수에 복사해놓고 현재의 값을 증가시키고, 임시변수를 버릴 필요가 없기 때문이다.

C++에서 역시 일반적인 변수에서는 상관없지만, 클래스의 인스턴스의 경우 차이가 있다. 기본으로 선언된 오퍼레이터의 동작을 보자.

```cpp
Foo& Foo::operator++()   // called for ++i
{
    this-> data += 1;
    return *this;
}

Foo Foo::operator++(int ignored_dummy_value)   // called for i++
{
    Foo tmp(*this);   // variable "tmp" cannot be optimized away by the compiler
    ++(*this);
    return tmp;
}
```

전위연산 같은 경우 단순히 1을 증가시키고 리턴하는 반면 후위연산 같은 경우 복사생성자(copy constructor)를 실행시킨 뒤, 현재 값을 1 증가시키고 임시 변수를 리턴시키는데, 이때 복사생성자는
  
컴파일러에 의해 자동으로 최적화 되지 않는다.

지금까지 사람들이 보통 iterator에 전위를 쓰는지 이해하지 못했는데, 위의 글을 보고 이해하게 되었다. 음;;