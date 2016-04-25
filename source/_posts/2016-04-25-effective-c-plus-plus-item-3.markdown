---
layout: post
title: "[Effective C++] Item 3. 낌새만 보이면 const를 들이대 보자!"
date: 2016-04-25 13:17:27 +0900
comments: true
categories: [C++, Effective C++] 
---
### 포인터와 `const`

`const` 키워드는 들어갈 수 있겠다 싶은 곳이면 어디든 붙일 수 있다.

포인터 타입에 대해서는 포인터 자체를 상수로, 혹은 포인터가 가리키는 대상을 상수로, 혹은 둘 다 지정할 수 있고, 아무것도 지정하지 않을 수도 있다.

```cpp
char greeting[] = "Hello";
char* p = greeting; // 비상수 포인터, 비상수 데이터
const char* p = greeting; // 비상수 포인터, 상수 데이터
char* const p = greeting; // 상수 포인터, 비상수 데이터
const char* const p = greeting; // 상수 포인터, 상수 데이터
```

`const` 키워드가 `*`의 왼쪽에 있으면 포인터가 가리키는 대상이 상수,
`const`가 `*`의 오른쪽에 있으면 포인터 자체가 상수이다.

`const`가 `*`의 양쪽에 다 있으면 포인터가 가리키는 대상 및 포인터가 전부 상수이다.

 포인터가 가리키는 대상을 상수로 만들 때 const 를 사용하는 스타일이 다를 수 있는데,
 
```cpp
void f1(const Widget *pw);
void f2(Widget const *pw);
```

위의 두 함수의 매개 변수 타입은 동일하다.

### STL과 `const`

STL 반복자(iterator)는 기본적인 동작 원리가 T* 포인터와 흡사합니다.

```cpp
std::vector<int> vec;
// ...
const std::vector<int>::iterator iter = vec.begin(); // iter는 T* const 처럼 동작합니다.
*iter = 10; // OK, iter가 가리키는 대상을 변경한다.
++iter; // 에러! iter는 상수

std::vector<int>::const_iterator cIter = vec.begin(); // cIter는 const T* 처럼 동작합니다.
*cIter = 10; // 에러! cIter가 가리키는 대상이 상수
++cIter; // OK, cIter를 변경한다.
```

`const`의 가장 강력한 용도는 함수 선언에 쓸 경우이다.

`const`는 함수 반환 값, 각각의 매개 변수, 멤버 함수 앞에 붙을 수 있고, 함수 전체에 대해 `const`의 성질을 붙일 수 있다.

### 함수 반환 값을 상수로 만들기
```cpp
class Rational { ... };
const Rational operator*(const Rational& lhs, const Rational& rhs);
```

함수의 반환 값이 상수일 때, 다음과 같은 어처구니 없는 실수를 막아준다.

```cpp
Rational a, b, c;
// ...
(a * b) = c; // 에러!
```

위와 같은 경우는 바로 알아챌 수 있겠지만, 다음과 같은 경우는 비교적 많이 일어난다.

```cpp
if (a * b = c) ... // 타이핑 실수, 원래 비교하려고 했던건데...
```

위의 코드는 `a`와 `b`가 기본 제공 타입이었다면 문법 위반에 걸리는 코드이지만, 사용자 정의 타입이기 때문에 허용된다.

### 상수 멤버 함수

멤버 함수에 붙는 `const` 키워드는 "해당 멤버 함수가 상수 객체에 대해 호출될 함수이다" 라는 사실을 알려준다.
이런 함수가 중요한 이유는 두 가지이다.

1. **클래스의 인터페이스를 이해하기가 쉬워짐**: 해당 클래스 객체의 상태를 변화시킬 수 없는 함수는 무엇이며, 변화시킬 수 있는 함수는 무엇인지 클래스 선언만 보고 바로 알 수 있다.

2. **해당 클래스의 상수 객체를 사용할 수 있게 해줌**: C++ 프로그램의 실행 성능을 높이는 핵심 기법중의 하나인 "상수 객체에 대한 참조자"를 쓰려면, 상수 멤버 함수가 준비되어 있어야 한다.

`const` 키워드가 있고 없고의 차이만 있는 멤버 함수들은 오버로딩이 가능하다.

```cpp
class TextBlock {
public:
    // ...
    
    // 상수 객체에 대한 operator[]                   _____
    const char& operator[](std::size_t position) const
    { return text[position]; }
    
    // 비상수 객체에 대한 operator[]
    char& operator[](std::size_t position)
    { return text[position]; }

private:
    std::string text;
};
```

위 처럼 선언된 `TextBlock`의 `operator[]`는 다음과 같이 쓸 수 있다.

```cpp
TextBlock tb("Hello");
std::cout << tb[0]; // 비상수 멤버 호출

const TextBlock ctb("World");
std::cout << ctb[0]; // 상수 멤버 호출
```

위 예제는 이해를 돕기위한 용도가 크고, 실제 프로그램에서 상수 객체가 자주 생기는 경우는 **상수 객체에 대한 포인터 혹은 참조자가 함수의 인자로 전달될 때**이다.

```cpp
void print(const TextBlock& ctb)
{
    if (ctb.empty()) return;
    std::cout << ctb[0];
    // ...
}
```

또한 상수 객체와 비상수 객체의 쓰임새가 아래처럼 달라질 수 있다.

```cpp
std::cout << tb[0]; // OK
tb[0] = 'x'; // OK
std::cout << ctb[0]; // OK
ctb[0] = 'x'; // 에러!
```

상수 객체의 반환 타입이 `const char&`이기 때문에 넷째줄에서 에러가 발생했다.

비상수 버전의 `operator[]`는 참조자를 반환해야 한다. 만약 그냥 `char`를 반환한다면, 다음 문장은 컴파일되지 않는다.

```cpp
tb[0] = 'x';
```

기본 제공 타입을 반환하는 함수의 반환 값을 수정하는 일은 불가능하다. 만약 된다고 해도, **'값에 의한 반환'**을 수행하는 C++의 특성상 수정되는 값은 `tb.text[0]`의 사본이지 `tb.text[0]`의 값이 아니다.


### 비트수준 상수성과 논리적 상수성

- 비트수준 상수성(물리적 상수성): 멤버 함수가 `const`이려면 그 객체의 어떤 데이터 멤버(정적 멤버는 제외)도 바꾸지 않아야 함.

C++에서 정의하고 있는 상수성도 비트수준 상수성이고, 컴파일러 차원에서 비트수준 상수성을 지키는 것은 매우 쉽지만, 포인터가 가리키는 대상을 변경하는 경우, 컴파일러가 잡아내지 못한다.

```cpp
class CTextBlock {
public:
    // ...

    // 부적절한(그러나 비트수준 상수성을 지키는 것처럼 보여져서 허용되는) 선언
    char& operator[](std::size_t position) const
    { return pText[position]; }
private:
    char *pText;
```

`operator[]` 내부 코드만 보면 `pText`를 바꿀 수 없다는 것은 확실하다. 그러나 다음과 같은 코드에서는

```cpp
const CTextBlock cctb("Hello");
char *pc = &cctb[0];
*pc = 'J'; // OK, cctb는 이제 "Jello"라는 값을 가짐
```

상수 객체에 대고 상수 멤버 함수 밖에 호출한 적이 없는데 내부의 값이 변해 버렸다.

논리적 상수성은 이런 황당한 상황을 보완하는 개념으로 나오게 되었다.

- **논리적 상수성**: 상수 멤버 함수라고 해서 객체를 조금도 바꿀 수 없는 것이 아니라, 일부는 바꿀 수 있되, 사용자측이 알아채지 못하게만 하면 상수 멤버 자격이 있다.

예를 들어, 클래스 내부의 캐시 데이터를 업데이트하는 용도가 이런 경우에 해당된다.

```cpp
class CTextBlock {
public:
    // ...
    std::size_t length() const;
private:
    char *pText;
    std::size_t textLength; // 바로 직전에 계산한(캐시된) 텍스트 길이
    bool lengthIsValid; // 이 길이(캐시)가 현재 유효한가?
};

std::size_t CTextBlock::length() const
{
    if (!lengthIsValid) {
        textLength = std::strlen(pText); // 에러!
        lengthIsValid = true; // 에러!
    }
    return textLength;
}
```

위 코드는 효율적인 측면을 고려하여 `textLength`의 값을 캐시해 놓는다.
그리고 사용자는 `length()`가 어떤식으로 구현되어 있던 간에 `pText`의 값이 변하지 않을 것을 알고 함수를 호출할 것이다.
이때 이 함수는 논리적 상수성을 지키고 있는 것이다.
그러나 컴파일러가 비트수준 상수성을 강제하고 있기 때문에 컴파일 에러가 발생한다.
이를 해결할 방법은 mutable 키워드를 이용하는 것이다.
이 키워드가 붙은 데이터 멤버는 상수 멤버 함수 안에서도 수정이 가능해진다.

```cpp
class CTextBlock {
public:
    // ...
    std::size_t length() const;
private:
    char *pText;
    mutable std::size_t textLength; // 바로 직전에 계산한(캐시된) 텍스트 길이
    mutable bool lengthIsValid; // 이 길이(캐시)가 현재 유효한가?
};

std::size_t CTextBlock::length() const
{
    if (!lengthIsValid) {
        textLength = std::strlen(pText); // OK
        lengthIsValid = true; // OK
    }
    return textLength;
}
```

#### 상수 멤버 및 비상수 멤버 함수에서 코드 중복 현상을 피하는 방법

`TextBlock::operator[]`가 기능이 확장되어(경계 검사, 접근 데이터 로깅, 무결성 검증 등) 코드가 비대해지면, 아무리 개별 함수로 빼낸다 해도 코드 중복이 생길 수 밖에 없다.

```cpp
class TextBlock {
public:
    // ...
    
    const char& operator[](std::size_t position) const
    { 
        // 경계 검사
        // 접근 데이터 로깅
        // 무결성 검증
        return text[position];
    }
    
    char& operator[](std::size_t position)
    {
        // 경계 검사
        // 접근 데이터 로깅
        // 무결성 검증
        return text[position];
    }

private:
    std::string text;
};
```

코드 중복은 절대악이다. 컴파일 시간, 유지보수 시간, 바이너리 크기 부풀림 등...
캐스팅을 써서 위와 같은 코드 중복을 피할 수 있다.
캐스팅은 기본적으로 피해야할 기법이지만, 코드 중복이 더 해악이므로 캐스팅을 사용한다.

```cpp
class TextBlock {
public:
    // ...
    
    const char& operator[](std::size_t position) const
    { 
        // 경계 검사
        // 접근 데이터 로깅
        // 무결성 검증
        return text[position];
    }
    
    char& operator[](std::size_t position)
    {
        return
            const_cast<char&>(
                static_cast<const TextBlock&>
                    (*this)[position]
            );
    }

private:
    std::string text;
};
```

첫번째 캐스팅인 `static_cast`는 무한 재귀 호출을 피하기 위해, 즉 상수 버전의 `operator[]`를 호출하기 위해 사용되었고, 두번째 캐스팅인 `const_cast`는 결과에서 `const`를 떼어내기 위해 사용되었다.

물론 C 스타일의 캐스팅을 사용하면 괄호 만으로도 끝나겠지만, **C 스타일의 캐스팅 보다는 C++ 스타일의 캐스팅을 사용해야 한다**(항목 27 참조).

비상수 멤버 함수에서 상수 멤버 함수를 호출하는 이유는 반대로 생각하면 간단해 진다.

만약 **상수 멤버 함수 쪽에서 비상수 멤버 함수 쪽을 호출했다면, 상수 멤버 함수로써는 비트 수준 상수성이 간단하게 깨지게 되는 것**이기 때문이다.

> - `const`를 붙여 선언하면 컴파일러가 사용상의 에러를 잡아내는 데 도움을 준다.
> - 컴파일러는 비트수준 상수성을 지켜야 하지만, 우리는 논리적인 상수성을 사용하여 프로그래밍해야 한다.
> - 상수 멤버 함수 및 비상수 멤버 함수가 서로 동일한 기능으로 구현되어 있을 경우, 비상수 버전이 상수 버전을 호출하도록 만들자.
