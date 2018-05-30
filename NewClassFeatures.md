# New Class Features
### Special member functions
********
In C++11, there a total of 6 special member functions:
+ move constructor
+ move assignment operator
+ default constructor
+ copy constructor
+ copy assignment operator
+ destructor

Under certain circumstances they are defined by the compiler even if not defined by the user. The rule may be too complicated to put in this documents so I recommend you to check [this link](http://en.cppreference.com/w/cpp/language/member_functions#Special_member_functions) for the full legal descriptions. However, some notable exceptions where the default member functions are not generated are listed here:
+ If you do provide a __destructor__ or a __copy constructor__ or a __copy assignment operator__, the compiler does not automatically provide a __move constructor__ or a __move assignment operator__.
+ If you do provide a __move constructor__ or a __move assignment operator__, the compiler does not automatically provide a __copy constructor__ or a __copy assignment operator__.

### Defaulted and Deleted Methods
********
Suppose that you wish to use a defaulted function that, due to circumstances(such as the exceptions listed above), isn’t created automatically, you can use the keyword default to explicitly declare the defaulted versions of these methods:
```cpp
class Someclass {
public:
Someclass(Someclass &&);
Someclass() = default; // use compiler-generated default constructor Someclass(const Someclass &) = default;
Someclass & operator=(const Someclass &) = default;
...
};
```
The delete keyword, on the other hand, can be used to prevent the compiler from using a particular method.
```cpp
class Someclass {
public:
Someclass() = default; // use compiler-generated default constructor
// disable copy constructor and copy assignment operator:
Someclass(const Someclass &) = delete;
Someclass & operator=(const Someclass &) = delete;
// use compiler-generated move constructor and move assignment operator:
Someclass(Someclass &&) = default;
Someclass & operator=(Someclass &&) = default;
Someclass & operator+(const Someclass &) const;
... };
```
Only the six special member functions can be defaulted, but you can use delete with any member function. Now suppose the Someclass definition is modified thusly:
```cpp
class Someclass {
public:
...
void redo(double);
void redo(int) = delete; ...
};
```
In this case, the method call `sc.redo(5)` matches the redo(int) prototype. The compiler will detect that fact and also detect that redo(int) is deleted, and it will then flag the call as a compile-time error.

### Managing Virtual Methods: override and final
*******
Suppose the base class declares a particular virtual method, and you decide to provide a different version for a derived class. This is called overriding the old version. But a redefined method in the derived class doesn't just override the base class declaration with the same function signature. Instead, it hides all base-class methods of the same name, regardless of the argument signatures.
```cpp
struct base {
    virtual void say(char c) {
        cout << "base::parameter is 'char c':"<< endl;
    }
};
struct deriv : base {
    virtual void say(char * c) {
        cout << "deriv::parameter is 'char * c': "<< endl;
    }
};
int main()
{
    char c='c';
    deriv deriv_obj;
    deriv_obj.say(c); // won't compile!
}
```
With C++11, you can use the virtual specifier override to indicate that you intend to override a virtual function. Place it after the parameter list. If your declaration does not match a base method, the compiler objects.
```cpp
struct deriv : base {
    virtual void say(char * c) override {
        cout << "deriv::parameter is 'char * c': "<< endl;
    }
};
```
Using the Apple compiler, I got the following message:
`main.cpp:11:18: 'say' marked 'override' but does not override any member functions`

The specifier final addresses a different issue.You may find that you want to prohibit derived classes from overriding a particular virtual method.
```cpp
struct base {
    virtual void say(char c) final {
        cout << "base::parameter is 'char c':"<< endl;
    }
};
```
In the derived class, we cannot define another "virtual void say(char c)" method. However, we can define "virtual void say(char * c)".
