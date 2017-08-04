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
# 3장 자원 관리

### 자원관리는 객체로

\- 자원 누출을 막기위해, 생성자 안에서 자원을 획득하고, 소멸자에서 그것을 해제하는 RAII(resource aquisition is initialization) 겍체를 사용합시다.  
\- 일반적으로 널리 쓰이는 RAII 클래스는 shared_ptr , auto_ptr 있는데 shared_ptr이 복사 동작이 직관적이기 때문에 더 널리 쓰인다. auto_ptr은 복사시 원본을 null로 만들어 버린다.

### 자원관리 클래스이 복사 동작에 대해 진지하게 고찰하자

\- RAII객체의 복사는 그 객체가 관리하는 자원의 복사 문제가 있기에, 그자원을 어떻게 복사 하느냐에 따라 RAII객체의 복사 동작이 결정된다.
\- RAII객체의 복사 동작을 금지 하거나, 참조 카운팅을 해주는 선으로 마무리 할 수 있다.

### 자원 관리 클래스에서 관리되는 자원은 외부에서 접근할 수 있도록 하자.

\- RAII 클래스를 만들 때는 그 클래스가 관리하는 자원을 얻을 수 있는 방법을 열어주어야 한다.
\- 자원 접근은 명시적 또는 암시적 변환을 통해서 가능하다.

### new 및 delete를 사용할 떄는 형태를 반드시 맞추자.

new 표현식에 []을 썼으면, delete 표현식에도 []을 써야하고,
new 표현식에 []을 사요하지 않았으면, delete에는 []을 사용하지 말아야 한다.

### new로 생성한 객체를 스마트 포인터에 저장하는 코드는 별도의 한 문장으로 만들자.
```cpp
//이런 함수가 있다고 가정하자.
void processWidget(std::shared_ptr<Widget> pw, int priority);

//함수 호출
processWidget(std::shared_ptr<Widget>(new Widget), priority());

//개선 코드
std::shared_ptr<Widget> pw(new Widget);
processWidget(pw, priority()); // 자원 누출 걱정이 사라짐.
```
CPP 에서는 파라미터에서 함수 호출 순서가 컴파일러마다 다를 수 있다.
예를 들어 위에 함수를 호출 할때 순서가,
new Widget  
Priority  
std::shared_ptr  
이런 순서로 호출이 될 수 있다는 말이고 이런식 으로 호출 될때 Priority간 호출 되면서 예외가 발생하게 된다면 new Widget에서 생성된 자원이 누수가 된다.  
때문에 반드시 **스마트 포인터에 저장되는 코드는 별도의 한문장으로 만들어야 한다.**

new 로 생성한 객체를 스마트 포인터에 넣는 코드는 반드시 별도의 한문장으로 만들자. 그렇지 않으면 디버깅하기 힘든 자원 누출이 초래될 수 있다.

# 4장 설계 및 선언 

### 클래스 설계는 타입 설계와 똑같이 취급하자.
클래스 설계시 고려 할점.
- 새로 정의한 타입의 객체 생성 및 소멸
- 객체 초기화와 객체 대입
- 복사 생성자를 어떻게 정의 할 것인가
- 값에 대한 제약사항
- 상속에 대한 구조
- 암시적 과 명시적에 대한 타입 변환
- 표준 함수(컴파일러가 생성해주는 함수)중 어떤것을 허용할지 말지 
- 접근 권한

### 20. 값에 의한 전달 보다는 상수객체 참조자에 의한 전달 방식을 택하라.

**c++은 기본 적으로 값에 의한 전달 방식을 사용한다.** 이 사본은 **복사 생성자에 의해서 만들어 진다.**

> Drived Class 를 함수의 파라미터로 넘길때 Base Class로 넘기게 되면 **Base Class의 복사 생성자가 호출**되므로 **Drived class의 특징들은 모두 사라지게** 된다.  
> 이점은 굉장히 주목할 사항이다.  

```cpp
//값에 의한 전달 보다는 상수 객체 참조자에 의한 전달을 선호 합시다.
void printName(const Window& w)
{
    ...
}
```

### 21. 함수에서 객체를 반환해야 하는 경우 참조자를 반환하려고 하지 말자.  -> 이해가 조금더 필요.

지역 스택 객체에 대한 포인터나 참조자를 반환하는 일 , 혹은 힙에 할당된 객체에 대한 참조자를 반환 하는일 , 지역 정적객체에 대한 포인터난 참조자를 반환하는 일은  
그런 객체가 두 개 이상 필요해질 가능성이 있다면 절대로 하지 말자.

### 22. 데이터 맴버가 선언될 곳은 private 영역임을 명심하자.
데이터 멤버는 private 멤버로 선언하자.

### 23. 멤버 함수보다는 비멤버 비프렌드 함수와 더 가까워지자.
멤버 함수보다는 비멤버 비프렌드 함수를 자주 쓰도록 하자, 캡슐화 정도가 높아지고, 패키징 유연성과 확장성이 늘어난다.

### 24. 타입 변환이 모든 매개변수에 대해 적용되어야 한다면 비멤버 함수를 선언하자.
```cpp
class Rational
{
public:
    Rational(int numerator = 0, int denominator = 1);
    
    const Rational operator*(const Rational& rhs) const;
 
}

void main()
{
    Rational a(1,8);
    Rational b(2,5);
    
    Rational result = a*b;
    
    result = a*2; // ok
    result = 2*a; // Error!!
}
```
위의 코드에서 a*2는 에러가 아닌데 왜 2*a 는 왜 빌드 에러가 발생하는 걸까? 원인은 2가지가 있다고 생각한다.
첫 번째는 바로 c++에서는 암시적 형변환이 있는데 함수의 인자로 받는 타입과 동일한 생성자를 지니고 있다면, 임시 객체를 생성해서 암시적으로 형변환을 하는 특징이 있습니다.
무슨 말인고 하니. 

```cpp
result = a*2 // 이 코드는 아래와 같이 바뀔수 있다.

result = a.operator*(2) //step 1
result = a.operator*(Rational(2)) //step 2 //int형의 인자를 받는 생성자가 존재하기 때문에 암시적 형변환.

```
하지만 result = 2*a같은 경우는 왜 에러가 발생할까? 이유는 왼쪽의 대상 인자는 매개변수 리스트에 들어가 있지 않기 때문입니다.
그렇다면 어떻게 왼쪽 값도 매개변수 리스트에 넣을까? 비멤버 함수로 선언하면 됩니다. 

```cpp
//개선 코드
//friend로 선언할 필요는 없다. Rational의 public interface만 가지고도 구현을 할 수 있기 때문이다.

class Relational
{
    ...
}

const Rational operator*(const Rational& lhs, const Rational rhs)
{
    ...
}


```
> 어떤 함수에 들어가는 모든 매개변수(this 포인터가 가르키는 객체도 포함해서) 타입 변환을 해 줄 필요가 있다면, 그 함수는 비멤버 함수 여야 한다.  

### 25. 예외를 던지지 않는 swap 에 대한 지원도 생각해 보자 => 이해 안감


# 5장 구현

### 26. 변수의 정의는 최대한 늦추자.
```cpp 

//A 방법
Widget W;
for(int i=0; i<n ++i)
{
    w= i;
}

//B 방법
for(int i = 0; i<n ++i)
{
    Widget w(i);
}
```
1. 대입이 생성자-소멸자 쌍보다 비용이 덜들고
2. 전체 코드에서 수행 성능에 민감한 부분을 건드리는 중

위의 두가지 경우가 아니라면 B 방법으로 가는 것이 좋다.

> 변수의 정의는 늦출 수 있을떄 까지 늦추는 것이 효율이 좋다.

### 27. 캐스팅은 절약, 또절약! 잊지 말자.

c++ 네 가지로 이루어진 새로운 형태의 캐스트 연산자.
- cosnt_cast<T>(); //객체이 상수성을 없애는 용도로 사용한다.
- dynamic_cast<T>(); //안전한 더운캐스팅을 할 때 사용하는 연산자.(런타임 비용이 높은 캐스트 연산자)
- reinterpret_cast<T>(); //포인터를 int로 바꾸는 등의 하부 수준 캐스팅을 위해 만들어진 연산자.
- static_cast<T>()  //암시적 변환을 강제로 진행할 때 사용한다.

* 다른 방법이 가능하다면 캐스팅은 피하자, 수행성능에 민감한 코드에서 dynamic_cast는 다시한번 사용을 자제하자.
* 캐스팅이 어쩔 수 없이 필요하다면, 함수 안에 숨길 수 있도록 해보자, 최소한 사용자는 캐스팅을 넣지 않고 함수를 호출 할 수 있게된다.  
* 구형 스타일의 캐스트보다 위의 신형 스타일을 사용하자.

### 28. 내부에서 사용하는 개체에 대한 핸들을 반환하는 코드는 피하자.
어떤 객체의 내부요소에 대한 핸들(참조자, 포인터, 반복자)을 반환하는 것은 되도록 피하자, 캡슐화를 높이고, 상수 멤버 함수가 객체의 상수성을 유지한 채로 동작할 수 있도록 하며,
**무효참조 핸들이 생기는 경우를 최소화** 할 수 있다.

### 29. 예외 안전성이 확보되도록 노력하자.

강력한 예외 안정성 보장은 복사-후-맞바꾸기 이지만 모든함수에 대해 실용적인 것은 아니다.  

### 30. 인라인 함수는 미주알고주알 따져서 이해해 두자.

```cpp
class Person
{
public:
    int age() const{return theAge;} //암시적인 인라인 요청 : age는 클래스 정의 내부에서 정의되었습니다.
private:
    int theAge;
}
```

명시적인 inline함수 선언 방법은 함수 이름앞에 inline키워드를 붙이는 것입니다.
```cpp
//명시적
templete<typename T>
inline const T& std::max(const t& a, const T& b)
{
    return a<b ? b:a;
}
```

**인라인 함수는 대체적으로 헤더 파일에 들어있어야 함**  
inline 함수로 선언해도 조건에 따라서는 inline화가 안될 수 있다.  

>함수 인라인은 작고, 자주 호출되는 함수에 대해서만 하는것으로 하자.
>함수 템플릿이 대게 헤더 파일에 들어간다는 일반적인 부분만 생각해서 이들을 inline함수로 선언하면 안된다.

### 31. 파일 사이의 컴파일 의존성을 죄대로 줄이자.

> 정의 대신에 선언에 의존하게 만들자 => 핸들 클래스 와 인터페이스 클래스
> 라이브러리 헤더는 선언부만 갖고 있는 형태여야 한다.

```cpp
//핸들 클래스 
class Person
{
public:
    Person(const std::string& name, const Date& birthday, const Address& addr)
    std::string name() const;
    std::string virthDate() cosnt;
    std::string address() const;
    
private:
    std::tr1::shared_ptr<PersonImpl> pImpl //구현 클래스에 대한 객체 포인터
                                           //PersonImpl는 Person에 있는 함수를 모두 구현하고 호출하는 방식으로 동작한다.
    
}

//생성자에서pImpl을 초기화 해준다. 
Person::Person(cosnt std::string& name, const Date& birthday, 
                const address& addr) : pImpl(new PersonImpl(name, birthday, addr))
{

}

std::string Person::name() const
{
    return pImpl->name();
}


//인터페이스 클래스

class Person
{
public:
    virtual ~Person();
    virtual std::string name() cosnt =0;
    virtual std::string birthDate() cosnt =0 ;
    virtual std::string address() const =0 ;
}


```

# 6장 상속, 그리고 객체 지향 설계

### 33. 상속된 이름을 숨기는 일은 피하자.
```cpp
class Base
{
private:
    int x;
public:
    virtual void mf1() = 0;
    virtual void mf1(int);
    virtual void mf2();
    void mf3();
    void mf3(double);
};

class Drived:public Base
{
public:
    virtual void mf1();
    void mf3();
    void mf4();
}

void main()
{
    Drived d;
    int x; 
    d.mf1(); //Drived call
    d.mf1(x); // error hide base
    
    d.mf2(); //base call
    
    d.mf3(); //drived call
    d.mf3(x); //Error hide base
}
```

위 코드에서 확인 되었듯이 상속 받은 함수와 같은 함수의 이름이, 부모(base) class에 존재 하면 파라미터가 달라도 자식(Drived) class의해서 모두 가려지게 된다.  
그렇다면 부모 클래스의 overload 된 함수들을 자식클래스에서 계속해서 가려져야 하는 것일까? 아니다 방법이 있다.

코드를 아래와 같이 수정하면 된다.

```cpp
class Drived:public Base
{
public:
    using Base::mf1; //Base에 있는 것들 중 mf1과 mf3 이름으로 가진 것들을 
    using Base::mf3; //Derived의 유효 범위에서 볼 수 있도록 만든다.

    virtual void mf1();
    void mf3();
    void mf4();
}

void main()
{
    Drived d;
    int x; 
    d.mf1(); //Drived call
    d.mf1(x); // base call
    
    d.mf2(); //base call
    
    d.mf3(); //drived call
    d.mf3(x); //base call
}
```
**전달 함수**
```cpp
class Base
{
public:
    virtual void mf1() =0;
    virtual void mf1(int);
}

//여기서 using을 사용하게 되면 private로 상속한 Base클래스의 모든 mf1 함수가 외부로 노출되기 때문이다.
//mf1()함수만 노출 시키고 싶을때 사용하는 방법이다.
class Drived: private Base
{
public:
    virtual void mf1() //전달 함수 
    {Base::mf1();}     //Inline함수가 된다.
}
```

이제는 정상적으로 동작하는 것을 확인 할 수 있다.  

> 파생 클래스이 이름은 기본 클래스이 이름을 가린다. public상속에서는 이런 이름 가림 현상은 바람직하지 않다.
> 가려진 이름을 다시 볼 수 있게 하는 방법으로, using선언 혹은 전달 함수를 쓸 수 있다.  

### 34. 인터페이스 상속과 구현 상속의 차이를 제대로 파악하고 구별하자.
```cpp
class shape
{
public:
    virtual void draw() cosnt = 0;
    virtual void error(const std::string& msg);
    int objectID() const;
}

class Rectangle : public Shape{...};
class Ellips : public Shape{...};
```

> 순수가상 함수를 선언하는 목적은 파생 클래스에게 함수의 인터페이스만을 물려주려는 것입니다.  

순수 가상 함수에도 정의를 제공할 수 있습니다. 하지만 호출을 하기 위해서는 반드시 클래스 이름을 한정자로 붙여주어야 합니다.
```cpp
Shape *ps = new Ellips();
ps->Shape::draw();
```
> 단순 가상함수를 선언하는 목적은 파생 클래스로 하여금 함수의 인터페이스 뿐만아니라 그 함수의  
> 기본 구현도 물려받게 하자는 것입니다.

>인터페이스 상속은 구현 상속과 다릅니다. public상속에서, 파생 클래스는 항상 기본 클래스의 인터페이스를 모두  
>물려 받습니다.  
>순수 가상 함수는 인터페이스 상속만을 허용합니다.  
>단순(비순수)가상함수는 인터페이스 상속과 더불어 기본 구현의 상속도 가능하도록 지정합니다.  
>비가상 함수는 인터페이스 상속과 구현의 상속도 가하도록 지정한다.  

### 35. 가상 함수 대신 쓸 것들도 생각해 두는 자세를 길러두자.

**비가상 인터페이스 관용구를 통한 템플릿 메서드 패턴**

```cpp
class CameCharacter
{
public:
    int healthValue() cosnt
    {
        int retVal = dohealthValue();
        return retVal;
    }
    
private:
    virtual int doHealthValue() const //Drived 클래스에서 오버라이드 할 수 있다.
    {
     ....
    }
}
```

**함수 포인터로 구현한 전략 패턴**

```cpp
class GameCharacter; //전방선언

int defaultHealthCalc(const GameCharacter& gc);

class GameCharacter
{
public:
    typedef int (*HealthCalcFunc)(const GameCharacter&);
    explicit GameCharacter(HealthCalcFrnc hcf = defaultHealthCalc) : healthFunc(hcf){} //initiallize
    
    int healthValue() const
    {
        return healthFunc(*this);
    }
        
private:
    HealthCalcFunc healthFunc;

}
```

**tr1::function으로 구현한 전략 패턴**
```cpp
class GameCharacter;
int defaultHealthCalc(const GameCharacter& gc);

class GAmeCharacter
{
public:
    typedef std::tr1::function<int (const GameCharacter&)> HealthCalcFunc;
    explicit GameCharter(HealthCalcFunc hcf = defalutHealthCalc) : healthFunc(hcf){}
    int healthValue() const
    {
        return healthFunc(*this);
    }
    
private:
    HealthCalcFunc healthFunc;
}
```

### 36. 상속받은 비가상 함수를 파생 클래스에서 재정의하는 것은 절대 금물

>객체를 가르키는 포인터의 타입에 의해서 함수가 결정되는 일이 발생 하기 때문이다.  
>이렇게 되면 프로그램 상에서 어떻게 동작할지 예측 하기가 매우 어렵다.

### 37. 어떤 함수에 대해서도 상속받은 기본 매개변수 값은 절대로 재정의 하지 말자.
>함수의 매개변수 값은 정적 바인딩 되고, 가상함수는 동적 바인딩 된다.  
객체를 가리키는 포인터 타입에 정의되어 있는 매개변수를 사용하고, 함수는 생성된 객체의 함수를 사용하는 이상한 현상이 발생하게 된다.

```cpp
class Shape
{
public
    enum ShapeColor {red, Green, Blue};
    virtual void draw(ShapeColor color = Red) const =0;
    
}

class Rectangle: public Shape
{
public:
    //기본 매개변수의 값을 변경하는 코드.
    virtual void draw(ShapeColor color = Green) cosnt;
}

void main()
{
    Shape *pr = new Rectangle;  //
    *pr.draw()  //이것은 Rectangle::draw(ShapeColor::Red)로 동작 하게 된다.
}
```
### 39. private 상속은 심사숙고해서 구사하자.

>- private상속을 대체할 방법은 존재한다. 객체 합성 기법으로 해결이 가능 하지만, 어떤 클래스의 protected 맴버를 사용해야 할 필요가 있을때는  
>private상속으로 구현 한다.
>- 객체 합성과는 달리 private 상속은 공백 기본 클래스 최적화(EBO)를 활성화 시킬 수 있다.

```cpp
//객체 합성 코드
class Widget
{
private:
    class WidgetTimer : public Timer
    {
    public
        virtual void onTick() const;
    }
    
    WidgetTimer timer; //Has-a 관계로 구현 하는 것을 객체 합성이라고 한다.
}
```

### 40. 다중 삭속은 심사숙고해서 사용하자.

