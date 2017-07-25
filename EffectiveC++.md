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

