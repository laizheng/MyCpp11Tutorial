# Lambda Functions
Constructs a closure: an unnamed function object capable of capturing variables in scope.

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

#### The `function` wrapper
The following section explains the following line of code:
```cpp
std::function<double(double)> ef1 = dub;
```
In C++, all the following is called `callable`:
+ The name of a function.
+ The pointer to a function.
+ A function object.
+ A name assigned to a lambda expression.

The `callable` has certain `call signature`, which is described by the return type followed by a comma-separated list of parameter types enclosed in a pair of parentheses. For example, the following function:
```cpp
double dub(double x) {return 2.0*x;}
```
Its `call signature` is `double(double)`. The abundance of callable types can lead to template inefficiencies. To see this, let’s examine a simple case.
```cpp
template <typename T, typename F>
T use_f(T v, F f)
{
  static int count = 0;
  count++;
  std::cout << " use_f count = " << count
            << ", &count = " << &count << std::endl;
  return f(v);
}

class Fp {
private:
  double z_;
public:
  Fp(double z = 1.0) : z_(z) {}
  double operator()(double p) {return z_*p; }
};

class Fq {
private:
  double z_;
public:
  Fq(double z = 1.0) : z_(z) {}
  double operator()(double q) {return z_+ q; }
};

double dub(double x) {return 2.0*x;}
double square(double x) {return x*x;}
int main() {
  using std::cout;
  using std::endl;
  double y = 1.21;
  cout << "Function pointer dub:\n";
  cout << " " << use_f(y, dub) << endl;
  cout << "Function pointer square:\n";
  cout << " " << use_f(y, square) << endl;
  cout << "Function object Fp:\n";
  cout << " " << use_f(y, Fp(5.0)) << endl;
  cout << "Function object Fq:\n";
  cout << " " << use_f(y, Fq(5.0)) << endl;
  cout << "Lambda expression 1:\n";
  cout << " " << use_f(y, [](double u) {return u*u;}) << endl;
  cout << "Lambda expression 2:\n";
  cout << " " << use_f(y, [](double u) {return u+u/2.0;}) << endl;
  return 0;
}
```
The call signatures for all the 6 callable are the same. So it might seem that F would be the same type for all six calls to use_f() and that the template would be instantiated just once. However, there are more than 1, and 4 as a matter of fact, templates instantiated, increasing code size.

Output of the above program:
```
Function pointer dub:
use_f count = 1, &count = 0x402028
2.42
Function pointer square:
use_f count = 2, &count = 0x402028
1.1
Function object Fp:
use_f count = 1, &count = 0x402020
6.05
Function object Fq:
use_f count = 1, &count = 0x402024
6.21
Lambda expression 1:
use_f count = 1, &count = 0x405020
1.4641
Lambda expression 2:
use_f count = 1, &count = 0x40501c
1.815
```
We can use the `std::function` template, declared in the functional header file, to specify an object in terms of a call signature, and it can be used to wrap a function pointer, function object, or lambda expression having the same call signature:
```cpp
std::function<double(double)> fw1 = dub;
```
Then all six calls to `use_f()` can be made with the same type (`function<double(double)>`) for `F`, resulting in just one instantiation.
```cpp
double dub(double x) {return 2.0*x;}
double square(double x) {return x*x;}
int main()
{
  	using std::cout;
  	using std::endl;
  	using std::function;

    double y = 1.21;
    function<double(double)> ef1 = dub;
    function<double(double)> ef2 = square;
    function<double(double)> ef3 = Fq(10.0);
    function<double(double)> ef4 = Fp(10.0);
    function<double(double)> ef5 =  [](double u) {return u*u;};
    function<double(double)> ef6 =  [](double u) {return u+u/2.0;};
    cout << use_f(y, function<double(double)>(dub)) << endl;
    cout << use_f(y, fdd(sqrt)) << endl;
    cout << use_f(y, ef3) << endl;
    cout << use_f(y, ef4) << endl;
    cout << use_f(y, ef5) << endl;
    cout << use_f(y, ef6) << endl;

    std::cin.get();
    return 0;
}
```
There are two other options to use function wrapper. First we can use a temporary `function<double(double)>` object as an argument to the `use_f()` function:
```cpp
typedef function<double(double)> fdd; // simplify the type declaration
cout << use_f(y, fdd(dub)) << endl; // create and initialize object to dub
cout << use_f(y, fdd(square)) << endl;
...
```
Second, we can modify the template to use `std::function`:
```cpp
template <typename T>
T use_f(T v,  std::function<T(T)> f)
{
    static int count = 0;
    count++;
    cout << "  use_f count = " << count << ", &count = " << &count << endl;
    return f(v);
}
// Function calls look like this:
cout << " " << use_f<double>(y, dub) << endl;
```
