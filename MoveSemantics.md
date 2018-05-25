# Move
## Values Categories
Each C++ expression (an operator with its operands, a literal, a variable name, etc.) is characterized by two independent properties: a __type__ and a __value category__. In practice we usually only concern two types, __rvalue__ and __lvalue__.   

Full legal descriptions of value category:   
<a href="http://en.cppreference.com/w/cpp/language/value_category">http://en.cppreference.com/w/cpp/language/value_category</a>  

### lvalue
An lvalue is an expression, such as a variable name or a dereferenced pointer, that represents data for which the program __can obtain an address__. Originally, an lvalue was one that could appear on the left side of an assignment statement.

```cpp
// Lvalues can appear on the left side of
// the built-in assignment operator:
a = 0;
// The address of lvalues can be taken:
int* a_ptr = &a;
// Lvalues can bind to lvalue references:
int& a_ref = a;
// Function call returning lvalue reference:
int& bar() { static int i = 0; return i; }
&bar();
bar() = 5;
```
### rvalue
Rvalue reference, indicated by using &&, can bind to rvalues—that is, values that can appear on the right-hand side of an assignment expression but for which one cannot apply the address operator.

Example of rvalues:
```cpp
int x = 10;
int y = 23;
// Rvalues can NOT appear on the left side of
// the built-in assignment operator:
5 = 0;
bar() = 0;
// The address of rvalues can NOT be taken:
&5;
&bar();
// Rvalues do NOT bind to lvalue references:
int& lv_ref0 = 5;
int& lv_ref1 = bar();
// Rvalues bind to rvalue references. Interestingly, binding an rvalue to an rvalue
// reference results in the value being stored in a location whose address can be
// taken.That is, although you can’t apply the & operator to 13, you can apply it to r1.
int && r1 = 13;
// r2 really binds to is the value to which x + y evaluates at that time.
// That is, r2 binds to the value 23, and r2 is unaffected by subsequent changes to x or y.
int && r2 = x + y;
// Function returning non-reference value
int bar() { return 5; }
double && r3 = bar();
```


### The Fuller Picture in C++11

ValueCategories:
![ValueCategories](https://github.com/laizheng/learn_cpp11/blob/master/value_cat.png")


With the introduction of move semantics in C++11, value categories were redefined to characterize two independent properties of expressions:
1. __Has identity__: it's possible to determine whether the expression refers to the same entity as another expression, such as by comparing addresses of the objects or the functions they identify (obtained directly or indirectly);
2. __Can be moved from__: move constructor, move assignment operator, or another function overload that implements move semantics can bind to the expression.   

In C++11, expressions that:  
* Have identity and cannot be moved from are called __lvalue__ expressions.
* Have identity and can be moved from are called __xvalue__ expressions.
* Do not have identity and can be moved from are called __prvalue__ ("pure rvalue") expressions.
* Do not have identity and cannot be moved from are not used.

The expressions that have identity are called "glvalue expressions" (glvalue stands for "generalized lvalue"). Both lvalues and xvalues are glvalue expressions.
The expressions that can be moved from are called "rvalue expressions". Both prvalues and xvalues are rvalue.

    #include <iostream>
    void foo(int&)
    {
        std::cout << "non-const lvalue ref\n";
    }
    void foo(int&&)
    {
        std::cout << "non-const rvalue ref\n";
    }
    void bar(const int&)
    {
        std::cout << "const lvalue ref\n";
    }
    int main()
    {
        int a = 0;
        foo(a);
        foo(5);
        bar(a);
        bar(5);
    }
