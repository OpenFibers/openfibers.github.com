---
layout: post
title: "C++ 10分钟速成"
date: 2020-05-07 10:33:56 +0800
comments: true
categories: 
---


## IDE 选择

### PC
MinGW  
VSCode  

### macOS
CLion (花钱)  
Xcode  

文章例子编译环境：  
方言： GNU++14[-std=gnu++14]
Standard library：libc++ (LLVM C++ standard library with C++11 support)


## 1. 指针

#### 声明

```cpp
int main() {
    string food = "Pizza";
    string* ptr = &food;
    cout << food << "\n";
    cout << &food << "\n";
    cout << ptr << "\n";
    cout << &ptr << "\n";
    cout << sizeof(ptr) << "\n";
    cout << sizeof(food) << "\n";
  return 0;
}
```

#### 执行结果

```
Pizza
0x7ffeefbff560
0x7ffeefbff560
0x7ffeefbff558
8
24
```

#### 类型

```
food  std::__1::string    "Pizza"	
ptr   std::__1::string *  "Pizza" 0x00007ffeefbff560
```

#### 内存 (macOS, big-endian)

```
0x7ffeefbff558          0x7ffeefbff560
⬇️                      ⬇️
60 F5 BF EF FE 7F 00 00 0A 50 69 7A 7A 61 00 00
`  õ  ¿  ï  þ  .  .  .     P  i  z  z  a  .  . 

00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
.  .  .  .  .  .  .  .  .  .  .  .  .  .  .  .  
```

<!--more-->


## 2. 引用

#### 声明

```
void swapNums(int &x, int &y) {
    cout << "address of ref " << &x << " " << &y << "\n";
    int z = x;
    x = y;
    y = z;
}

int main() {
    int firstNum = 10;
    int secondNum = 20;

    cout << "Before swap: " << "\n";
    cout << firstNum  << " " << secondNum << "\n";
    cout << "address of num " << &firstNum << " " << &secondNum << "\n";

    // Call the function, which will change the values of firstNum and secondNum
    swapNums(firstNum, secondNum);

    cout << "After swap: " << "\n";
    cout << firstNum  << " " << secondNum << "\n";
    cout << sizeof(firstNum) << "\n";
    cout << sizeof(secondNum) << "\n";
    return 0;
}
```

#### 执行结果

```
Before swap: 
10 20
address of num 0x7ffeefbff578 0x7ffeefbff574
address of ref 0x7ffeefbff578 0x7ffeefbff574
After swap: 
20 10
4
4
```

#### 类型

```
firstNum    int     10
secondNum   int     20
x           int &   0x00007ffeefbff578
y	          int &   0x00007ffeefbff574
```

#### 内存 (macOS, big-endian)
```
0x7ffeefbff574  0x00007ffeefbff578
⬇              ⬇️️
14  00  00  00  0A  00  00  00
20  0   0   0   10  0   0   0
```


## 3. OOP: Contructor and Initialize

```
class MyClass {     // The class
  public:           // Access specifier
    MyClass() {     // Constructor
      cout << "Hello World!";
    }
};

int main() {
  MyClass myObj;    // Create an object of MyClass (this will call the constructor)
  return 0;
}
```

#### 执行结果

```
Hello World
```


## 4. 方法默认参数

```
void myFunction(string country = "Norway") {
  cout << country << "\n";
}

int main() {
  myFunction("Sweden");
  myFunction("India");
  myFunction();
  myFunction("USA");
  return 0;
}

// Sweden
// India
// Norway
// USA
```

## 5. 继承

```
// Base class
class Vehicle {
  public:
    string brand = "Ford";
    void honk() {
      cout << "Tuut, tuut! \n" ;
    }
};

// Derived class
class Car: public Vehicle {  // 继承方式：public
  public:
    string model = "Mustang";
};

int main() {
  Car myCar;
  myCar.honk();
  cout << myCar.brand + " " + myCar.model;
  return 0;
}
```

#### 执行结果

```
Tuut, tuut! 
Ford Mustang
```

#### 继承权限

子类中直接继承获得的父类方法，访问权限遵循最小权限原则。  
权限范围： private < protected < public
继承获得的方法，其权限 = min(父类方法访问权限，继承方式)
举例：  

```
// Base class
class Vehicle {
  public:
    string brand = "Ford";
    void honk() {
      cout << "Tuut, tuut! \n" ;
    }
};

// Derived class
class Car: private Vehicle {  // 继承方式为 private
  public:
    string model = "Mustang";
};

int main() {
  Car myCar;
  myCar.honk();  // 子类权限 = min(public, private)，也就是 private. 这里会产生编译错误
  return 0;
}
```



## 6. 多重继承

```
// Base class
class MyClass {
  public: 
    void myFunction() {
      cout << "Some content in parent class." ;
    }
};

// Another base class
class MyOtherClass {
  public: 
    void myOtherFunction() {
      cout << "Some content in another class." ;
    }
};

// Derived class 
class MyChildClass: public MyClass, public MyOtherClass {
};

int main() {
  MyChildClass myObj;
  myObj.myFunction();
  myObj.myOtherFunction();
  return 0;
}
```

#### 如果 MyClass 和 MyOtherClass 有同名函数，则允许继承但不允许调用

```
// Base class
class MyClass {
  public:
    void myFunction() {
      cout << "Some content in parent class." ;
    }
};

// Another base class
class MyOtherClass {
  public:
    void myFunction() {
      cout << "Some content in another class." ;
    }
    void myOtherFunction() {
      cout << "Some content in another class." ;
    }
};

// Derived class
class MyChildClass: public MyClass, public MyOtherClass {
};

int main() {
  MyChildClass myObj;
  myObj.myFunction();  // err at compile time
  myObj.myOtherFunction();  // can execute properly
  return 0;
}
```

#### 多重继承时的权限修饰符

和单继承的权限修饰同理，遵循权限相乘规则。

```
// Base class
class MyClass {
  public:
    void myFunction() {
      cout << "Some content in parent class." ;
    }
};

// Another base class
class MyOtherClass {
  public:
    void myOtherFunction() {
      cout << "Some content in another class." ;
    }
};

// Derived class
class MyChildClass: private MyClass, public MyOtherClass {
  public:
    MyChildClass(){
      myFunction();  // can execute properly
    }
};

int main() {
  MyChildClass myObj;
  myObj.myFunction();  // 权限 = min(public, private) 也就是 private，err at compile time
  myObj.myOtherFunction();  // can execute properly
  return 0;
}
```