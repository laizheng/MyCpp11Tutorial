# Lambda Functions
Suppose you wish to generate a list of random integers and determine how many of them are divisible by 3 and how many are divisible by 13. The following section demonstrate using three approaches for passing information to an STL algorithm: function pointers, functors, and lambdas.

#### The How of Function Pointers, Functors, and Lambdas
Generating the list is pretty straightforward. One option is to use a vector<int> array to hold the numbers and use the STL generate() algorithm to stock the array with random numbers:
```cpp
#include <vector>
#include <algorithm>
#include <cmath>
...
std::vector<int> numbers(1000); std::generate(vector.begin(), vector.end(), std::rand);
```
In this case, the function object is a pointer to the standard rand() function.
Next, we use the std::count_if() combined with the three types of function objects to achieve our goals.
```cpp
// Using function pointers
bool f3(int x) {return x % 3 == 0;}
bool f13(int x) {return x % 13 == 0;}
int count3 = std::count_if(numbers.begin(), numbers.end(), f3);
int count13 = std::count_if(numbers.begin(), numbers.end(), f13);

// Using functors
class f_mod {
  private:
  int dv; public:
  f_mod(int d = 1) : dv(d) {}
  bool operator()(int x) {return x % dv == 0;}
};
count3 = std::count_if(numbers.begin(), numbers.end(), f_mod(3));
count13 = std::count_if(numbers.begin(), numbers.end(), f_mod(13));

// Using Lambda functions
count3 = std::count_if(numbers.begin(), numbers.end(), [](int x){return x % 3 == 0;});
count13 = std::count_if(numbers.begin(), numbers.end(), [](int x){return x % 13 == 0;});
```
Notice in this example, there is no declared return type. Instead, the return type is the type that decltype would deduce from the return value, which would be bool in this case. If the lambda doesn’t have a return statement, the type is deduced to be void.

The automatic type deduction for lambdas works only if the body consists of a single return statement. Otherwise, you need to use the new trailing-return-value syntax:
```cpp
[](double x)->double {int y = x; return x – y;} // return type is double
```

#### Named Lambdas
you can create a name for the anonymous lambda and then use the name twice.
```cpp
auto mod3 = [](int x){return x % 3 == 0;} // mod3 a name for the lambda
count1 = std::count_if(n1.begin(), n1.end(), mod3);
count2 = std::count_if(n2.begin(), n2.end(), mod3);
```
You even can use this no-longer-anonymous lambda as an ordinary function.
```cpp
bool result = mod3(z); // result is true if z % 3 == 0
```
Unlike an ordinary function, however, __a named lambda can be defined inside a function__.The actual type for mod3 will be some implementation-dependent type that the compiler uses to keep track of lambdas.

#### Relative efficiencies
The relative efficiencies of the three approaches boils down to what the compiler chooses to inline. Here, the function pointer approach is handicapped by the fact that compilers traditionally don’t inline a function that has its address taken because the concept of a function address implies a non-inline function.With functors and lambdas, there is no apparent contradiction with inlining.

#### Capturing variables on stack
Lambdas have some additional capabilities. In particular, a lambda can access by name any automatic variable in scope. Variables to be used are captured by having their names listed within the brackets.
+ If just the name is used, as in [z], the variable is accessed by value.
+ If the name is preceded by an &, as in [&count], the variable is accessed by reference.
+ Using [&] provides access to all the automatic variables by reference,
+ Using [=] provides access to all the automatic variables by value.

You also can mix and match. For instance, [ted, &ed] would provide access to ted by value and ed by reference, [&, ted] would provide access to ted by value and to all other automatic variables by reference, and [=, &ed] would provide access by reference to ed and by value to the remaining automatic variables.

Using the previous counting example, we can use this technique to count elements divisible by 3 and elements divisible by 13 using a single lambda expression:
```cpp
int count3 = 0;
int count13 = 0;
std::for_each(numbers.begin(), numbers.end(),
              [&](int x){count3 += x % 3 == 0; count13 += x % 13 == 0;});
```
