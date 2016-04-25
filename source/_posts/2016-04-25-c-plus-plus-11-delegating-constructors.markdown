---
layout: post
title: "[C++ 11] Delegating Constructors"
date: 2016-04-25 13:29:10 +0900
comments: true
categories: [C++, C++ 11]
---
### 생성자 위임 (C++11)

아래와 같은 클래스를 가정해보자.

```cpp
class Foo {
public:
    Foo(char x) : _x(x) { } 
    Foo(char x, int y) : _x(x), _y(y) { }
    ...
    
private:
    char _x;
    int _y;
};
```

여기에서는 데이터 멤버 수가 적기 때문에 크게 문제될 것이 없어 보이지만, 데이터 멤버가 늘어나면 날수록 멤버 초기화 리스트에 보이는 멤버들 수는 기하 급수적으로 늘어날 것이다.

C++03 에서는 아래와 같은 방법들로 이를 해결했었다.

1. 데이터 멤버에 기본 값을 넣을 수 있는 경우에. 기본 인자를 설정하여 여러 버전의 생성자를 만드는 방법

```cpp
class Foo {
public:
    Foo(char x, int y=0) : _x(x), _y(y) { } // Foo(char)와 Foo(char, int)
    ...
 };
```

2. 대입으로도 초기화가 가능한 데이터 멤버들을 별도의 함수로 빼내어(대게 `private` 함수) 초기화를 시키는 방법

```cpp
class Foo {
 public:
   Foo(char x);
   Foo(char x, int y);
   ...
   
 private:
   ...
   void construct(char x, int y);
 };

 Foo::Foo(char x)
 {
     construct(x, 0);
     ...
 }

 Foo::Foo(char x, int y)
 {
     construct(x, y);
     ...
 }

 void Foo::construct(char x, int y)
 {
     _x = x;
     _y = y;
     ...
 }
```

1번의 방법이 깔끔하긴 하지만 인자의 순서에 신경을 써야 한다는 단점이 있다. 기본 인자들은 마지막에 넣어주어야 하기 때문이다.

2번은 코드 중복을 피할 수는 있지만, 효율성을 잃게 된다.

C++11에서는 아주 멋지게 이를 해결할 방법이 존재한다.

```cpp
class Foo {
public: 
    Foo(char x, int y) : _x(x), _y(y) { }
    Foo(char x) : Foo(x, 0) { } // 생성자 위임
};
```

위처럼 생성자 위임을 통해 다른 버전의 생성자를 호출하는 것이 가능해진다.

또한 아래와 같은 방법도 가능하다.

```cpp
class Foo {
public:
    Foo() { }
    explicit Foo(char x) : _x(x) { }

private:
    char _x = 0;
};
```

기본 생성자를 호출하는 경우에도 `_x`는 `0`으로 초기화될 것이다. 또한 `Foo(char)` 생성자는 효율성을 위해 `_x`의 초기화를 두번 수행하지 않고 한번만 수행한다. (`Foo(char)` 생성자에 `explicit` 키워드를 붙이는 이유는 [Effective C++ 항목 24] 참조)

기본 클래스의 생성자로부터 생성자를 위임하고자 할 경우에는 다음과 같은 방법을 사용할 수도 있다.

```cpp
class BaseClass {
public:
    BaseClass(int value);
};

class DerivedClass : public BaseClass {
public:
    using BaseClass::BaseClass;
};
```

`using` 키워드를 사용하여 `DerivedClass`에서 `BaseClass`의 생성자를 사용하고 있다.

