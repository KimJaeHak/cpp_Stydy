# 1장 
### Const를 들이대자.

- Const를 붙여 선언하면 컴파일러가 사용상의 에러를 잡는데 도움을 준다.  
- 컴파일러는 비트수준 상수성을 지켜야 한다, 우리는 개념적인(논리적인)상수성 을 사용해서 프로그래밍 한다.
- 상수 맴버 및 비상수 맴버가 기능적으로 똑같이 구현되어 있을때 코드 중복을 피하는 것이 좋은데, 이때 **비상수 에서 상수함수를 호출하는 방식**으로 구현한다.  


### 객체를 사용하기 전에 반드시 초기화 하자

- 기본제공 타입의 객체는 직접 초기화 한다, 경우에 따라서 다르기 때문.  
- 생성자에서 대입문으로 맴버를 초기화 하지 말고 초기화 리스트를 사용하자, 초기화 리스트에서 데이터를 나열할 때는 클래스에 맴버 데이터가 나열된 순서대로 한다.
- 비지역 정적 객체들의 초기화 순서 문제의 해결은 비지역 정적 객체를 지역 정적 객체로 바꾸면된다.

```cpp
class FileSystem
{
public:
    std::size_t numDisks() const
}

extern FileSystem tps; 

//위이 코드를 아래와 같이 바꾼다.

class FileSystem {....};

FileSystem& tps()
{
    static FileSystem fs;
    return fs;
}

//사용하기 위해 호출하면 초기화 한다.
//초기화 하지 않고 사용하는 일이 발생하지 않도록 한다.
```

# 2장

### c++이 만들어 호출해 버리는 함수들을 조심하자.
- 복사 생성자(copy constructor)  
- 복사 대입 연산자(copy assignment operator)  
- 소멸자(destructor)  

```cpp
class Empty{}; //사용자가 이렇게 선언하면

//컴파일러는 이렇게 만든다.
class Empty
{
public:
    Empty(){...}
    Empty(const Empty& ref){...}
    ~Empty(){...}
    Empty& operator=(const Empty& ref){...}
}
```
- 컴파일러는 경우에 따라 클래스에 대해 기본생성자, 복사생성자, 복사대입연산자, 소멸자를 암시적으로 만들어 놓을 수 있다.

### 컴파일러가 만들어낸 함수가 필요 없다면 사용을 금지하자.

```cpp
//1. private로 선언한다.
//2. 선언만 하고 구현하지 않는다.
class HomeForSale
{
public:
    ...
private:
    HomeForSale(const HomeForSale&);
    HomeForSale& operator=(const HomeForSale&); //선언에서 매개변수 이름은 선택이다.
}

```

### 다형성을 가진 기본 클래스에서는 소멸자를 반드시 **가상 소멸자**로 선언

즉 어떤 함수가 Virtual 함수를 하나라도 가지고 있다면, 반드시 소멸자는 가상 소멸자로 선언한다.

순수 가상함수로 소멸자를 선언 하는 방법도 있는데, 어떤 함수를 abstract(추상) class로 선언하려는데 마땅히 만들어야 할 순수 가상 함수가 없을때  
소멸자를 순수가상 함수로 만들 수 있다.  
**이때 주의해야 할 점은** 순수 가상 함수에 정의를 꼭 구현해야한다. 그렇지 않으면 링크 Error 가 발생하게 된다.

### 예외가 소멸자를 떠나지 못하도록 붙들어 놓자

소멸자에서는 예외가 빠져나가면 안된다. 만약 소멸자 안에서 호출된 함수가 예외를 던질 가능성이 있다면, 소멸자에서 모두 삼켜 버리거나 프로그램을 종료시켜야 한다.
예외가 발생할 소지가 있는 함수는 소멸자에 넣지 않는 것이 좋겠다.\

### 객체 생성 및 소멸 과정중에는 절대로 가상 함수를 호출하지 말자.

파생클래스 객체의 기본 클래스가 생성되는 동안에는 그 객체의 타입은 기본클래스다. 무슨 말인고 하니  
기본 클래스가 생성되는 동안에는 Virtual 함수등을 호출해도 파생클래스의 함수가 호출 되는 것이 아니고 기본 클래스의 것이 호출된다는 것이다.
특히나 순수 가상 함수를 호출하게 되면 링크 에러가 나거나 혹시 비가상 함수를 정의해서 순수 가상함수를 호출하게 되면 프로그램이 종료되는 현상도 발생한다.
```cpp
//link Error
class Test
{
public:
    Test()
    {
        MyFunc(); //링크 에러 발생
    }
    
    virtual void MyFunc() const =0;
}

//프로그램 종료
class Test
{
public:
    Test()
    {
       Init(); //프로그램을 종료하게 만듬
               //비가상 함수에서 가상함수를 호출.
    }
    
    void Init();    
    virtual void MyFunc() const =0;
}

Test::Init()
{
    MyFunc();
}
```

### 대입 연산자는 *this의 참조자를 반환하게 하자

```cpp
class Widget
{
public:
    Widget& operator=(const Widget& rhs)
    {
        ...
        return *this;
    }
}
```
### operator= 에서는 자기대입에 대한 처리가 빠지지 않도록 하자.

```cpp

class Widget
{
private:
    Bitmap *pb;
}

//자기 자신이 들어올 경우.
//안전하지 않게 구현됨
Widget& Widget::operator=(const Widget& rhs)
{
    delete pb;   //이부분에서 left , right 모두 bitmap 이 NULL이 된다.
    pb = new Bitmap(*rhs.pb);
    return *this;
}

//개선 코드
Widget& Widget::operator=(const Widget& rhs)
{
    Bitmap *pOrig = pb;
    pb = new Bitmap(*rhs.pb);
    delete pOrig;
}

```

** 같은 객체가 대입 되는 경우에 처리를 항상 생각하자.

### 객체의 모든 부분을 빠짐없이 복사하자.

```cpp
class Base
{
private:
    int baseNum;
public:
    Base(const Base& rhs)
    Base& operator=(const Base& rhs);    
}

class Drived : public Base //상속
{
private:
    int drivedNum;
public:
    Drived(const Drived& rhs);
    Drived& operator=(const Drived& rhs);
}

Drived::Drived(const Drived& rhs) : Base(rhs) //베이스 클래스의 복사 생성자를 꼭 호출.
{
    drivedNum = rsh.drivedNum;
}

Drived& operator=(const Drived& rhs)
{
    Base::operator=(rhs) //베이스 클래스의 opreator를 호출
    drivedNum = rhs.drivedNum;
    
    return *this;
}
```
# 3장
