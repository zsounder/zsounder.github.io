sf上有朋友问题了unnamed namespace的问题,unnamed namespace，不具名命名空间或者说匿名空间,Google C++ Style中有如下说明：
> Unnamed Namespaces
  Unnamed namespaces are allowed and even encouraged in .cc files, to avoid runtime naming conflicts.
  Do not use unnamed namespaces in .h files.
  ```c++
  namespace {                           
  // This is in a .cc file.
  // The content of a namespace is not indented
  enum { kUnused, kEOF, kError };       // Commonly used tokens.
  bool AtEof() { return pos_ == kEOF; }  // Uses our namespace's EOF.
  }  // namespace
  ```
  
  
unnamed namespace可以在某个给定的文件内不连续，但是不能跨越多个文件。也就是说，未命名的命名空间仅在特定的文件内部有效，其作用范围不会横跨多个不同文件。

  ```c++
  //inc.php
#pragma once
#include <iostream>
#include <typeinfo>

namespace {
    int i = 10;
    class TestClass {
    };
}

void one_local_variable_info();
void two_local_variable_info();

const std::type_info& one_get_type_info_TestClass();
const std::type_info& two_get_type_info_TestClass();


void one_pass_TestClass(TestClass * t);
void two_pass_TestClass(TestClass * t);

// one.cpp
#include "inc.h"

const std::type_info& one_get_type_info_TestClass()
{
    return typeid(TestClass);
}

void one_local_variable_info(){
    i = 11;
    std::cout << "addr: " << &i << " value: " << i << std::endl;
}

//two.cpp
#include "inc.h"
#include <iostream>
#include <typeinfo>
using namespace std;

const std::type_info& two_get_type_info_TestClass()
{
    return typeid(TestClass);
}

void two_local_variable_info(){
    std::cout << "addr: " << &i << " value: " << i << std::endl;
}

//main.cpp
#include "inc.h"
using namespace std;

int main()
{
    one_local_variable_info();
    two_local_variable_info();

    const std::type_info& t1 = one_get_type_info_TestClass();
    const std::type_info& t2 = two_get_type_info_TestClass();
    std::cout << "one.cpp TestClass type_info name: " << t1.name() <<"  hashcode: "<< t1.hash_code()<<endl;
    std::cout << "two.cpp TestClass type_info name: " << t2.name() <<"  hashcode: "<< t2.hash_code()<<endl;
    cout <<"one.cpp and two.cpp, TestClass type_info "<<(t1==t2?"same":"not same")<<endl;
    return 0;
}
```

输出如下：
```
addr: 0x1000020f8 value: 11
addr: 0x1000020fc value: 10
one.cpp TestClass type_info name: N12_GLOBAL__N_19TestClassE  hashcode: 4294974928
two.cpp TestClass type_info name: N12_GLOBAL__N_19TestClassE  hashcode: 4294974960
one.cpp and two.cpp, TestClass type_info not same
```
cpp文件作为编译单元，one.cpp与two.cpp include同一个头文件，虽然TestClass在两个单元中生成的名字相同，但type_info的hash_code(c11支持)并不相同。
同样i变量在两个cpp单元中都有独立一份拷贝。
