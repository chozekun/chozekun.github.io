---
layout: post
title: "[Effective C++] Item 7. 다형성을 가진 기본 클래스에서는 소멸자를 반드시 가상 소멸자로 선언하자"
date: 2016-04-25 13:17:34 +0900
comments: true
categories: [C++, Effective C++]
---
다음과 같은 다형성을 가진 클래스를 가정하자.

```cpp
class TimeKeeper {
public:
    TimeKeeper();
    ~TimeKeeper();
    ...
};

class AtomicClock: public TimeKeeper { ... };
class WaterClock: public TimeKeeper { ... };
class WristWatch: public TimeKeeper { ... };
```

사용자는 이 클래스를 이용하여 시간 계산을 하고 싶을 것이다. 그러나 이 시계가 어떤 시계인지에 대한 관심은 없다. 그렇다면 다음과 같이 팩토리 함수를 만들어 두고 사용하면 좋을 것이다.

```cpp
TimeKeeper* getTimeKeeper(); // TimeKeeper의 파생 클래스로부터 생성되는 객체의 포인터를 반환
```



```cpp
TimeKeeper *ptk = getTimeKeeper();
...
delete ptk;
```

```cpp
class TimeKeeper {
public:
    TimeKeeper();
    virtual ~TimeKeeper();
    ...
};

TimeKeeper *ptk = getTimeKeeper();
...
delete ptk;
```

```cpp
class Point {
public:
    Point(int xCoord, int yCoord);
    ~Point();
    
private:
    int x, y;
};
```

```cpp
class SpecialString: public std::string {
    ...
};
```

```cpp
SpecialString *pss = new SpecialString("Impending Doom");
std::string *ps;
...
ps = pss;
...
delete ps;
```

```cpp
class AWOV {
public:
    virtual ~AWOV() = 0;
};
```

```cpp
AWOV::~AWOV() { }
```

