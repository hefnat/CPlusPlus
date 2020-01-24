# CPlusPlus
Based on the site http://cppquiz.org/quiz, extracted questions for easier overview.

The more :skull: you see, the higher the difficulty!

According to the C++17 standard, what is the output of the following programs? 
 
#### 1. :skull::skull:   
```
#include <iostream>

int main() {
  int a = 10;
  int b = 20;
  int x;
  x = (a, b);
  std::cout << x;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 20

The comma operator is applied on two expressions: a and b.

According to [expr.comma](https://timsong-cpp.github.io/cppwp/n4659/expr.comma#1)§8.19¶1 in the standard: "A pair of expressions separated by a comma is evaluated left-to-right; the left expression is a discarded-value expression (...) The type and value of the result are the type and value of the right operand".

The right operand here being b, with the value 20. This is then the resulting value of the expression (a, b), and 20 is assigned to x.
</p>
</details>

#### 2. :skull: 

```
#include <iostream>

class A {
public:
  A() { std::cout << 'a'; }
  ~A() { std::cout << 'A'; }
};

class B : public A {
public:
  B() { std::cout << 'b'; }
  ~B() { std::cout << 'B'; }
};

int main() { B b; }
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: abBA

The derived class constructor B() first calls the base class constructor A(), and then executes the compound-statement (the part inside {}) of its own body.

The derived class destructor ~B() first executes its own body, and then calls the base class destructor ~A().

Let's have a look at the standard:

Initialization order is defined in [class.base.init](https://timsong-cpp.github.io/cppwp/n4659/class.base.init#13)§15.6.2¶13:

"In a non-delegating constructor, initialization proceeds in the following order:
- (...)
- Then direct base classes are initialized in declaration order as they appear in the base-specifier-list (regardless of the order of the mem-initializers)
- (...)
- Finally, the compound-statement of the constructor body is executed."

Destruction order is defined in [class.dtor](https://timsong-cpp.github.io/cppwp/n4659/class.dtor#9)§15.4¶9:

"After executing the body of the destructor and destroying any automatic objects allocated within the body, a destructor for class X calls (...) the destructors for X’s non-virtual direct base classes"  

</p>
</details>

#### 3. :skull:
```
#include <iostream>
struct X {
  virtual void f() const { std::cout << "X"; }
};

struct Y : public X {
  void f() const { std::cout << "Y"; }
};

void print(const X &x) { x.f(); }

int main() {
  X arr[1];
  Y y1;
  arr[0] = y1;
  print(y1);
  print(arr[0]);
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: YX

arr is an array of X, not of pointers to X. When an object of type Y is stored in it, it is converted to X, and we lose the "Y part" of the object. This is commonly known as "slicing".
</p>
</details>

#### 4. :skull::skull:   
```
#include <iostream>

using namespace std;

template<typename T>
void f(T) {
    cout << 1;
}

template<>
void f(int) {
    cout << 2;
}

void f(int) {
    cout << 3;
}

int main() {
    f(0.0);
    f(0);
    f<>(0);
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 132

Here we have three calls to f. Which function is selected for each call? [temp.over](https://timsong-cpp.github.io/cppwp/n4659/temp.over#1)§17.8.3¶1 is surprisingly readable:

    A function template can be overloaded either by (non-template) functions of its name or by (other) function templates of the same name. 

The function template f is in our case overloaded by the non-template void f(int). It goes on:

    When a call to that name is written [...], template argument deduction [is] performed for each function template to find the template argument values (if any) that can be used with that function template to instantiate a function template specialization that can be invoked with the call arguments

So in the tree cases above, we first do template argument deduction, giving double, int and int, respectively.

Then:

    For each function template, if the argument deduction and checking succeeds, the template-arguments [...] are used to synthesize the declaration of a single function template specialization which is added to the candidate functions set to be used in overload resolution

So in all the three cases above, a specialization of the function template f is added to the candidates to overload resolution, with T equal to double, int and int, respectively.

Now look at each call:

For f(0.0), T is deduced as double, and a specialization void f(double) is added as a candidate for overload resolution. The other candidate is the non-template function void f(int). The one taking a double is a better match, and 1 is printed.

For f(0), T is deduced as int, and a specialization void f(int) is added as a candidate for overload resolution. The other candidate is the non-template function void f(int). Both are equally good matches, so is the program ill-formed?

[over.match.best](https://timsong-cpp.github.io/cppwp/n4659/over.match.best#1)§16.3.3¶1>

    F1 is defined to be a better function than another viable function F2 if [...]
    F1 is not a function template specialization and F2 is a function template specialization

void f(int) is not a function template specialization, and template<> void f(int) is, so the former is selected, and 3 is printed.

Finally we get to f<>(0). <> is an (empty) template argument list, which can be specified for template functions, but not for non-template functions. So in this case, the non-template function is not an option, we explicitly ask for the function template. This is also described in a note in [temp.arg.explicit](https://timsong-cpp.github.io/cppwp/n4659/temp.arg.explicit#4)§17.8.1¶4:

    An empty template argument list can be used to indicate that a given use refers to a specialization of a function template even when a non-template function is visible that would otherwise be used.

T is deduced to int, the specialization for int is the only candidate, and 2 is printed.
</p>
</details>

#### 5. :skull::skull::skull: 

```
#include <iostream>

int a = 1;

int main() {
    auto f = [](int b) { return a + b; };

    std::cout << f(4);
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 5  

It is clear that a is not captured explicitly, so the question is whether a should be captured implicitly (and if yes, then a default capture has to be specified). [expr.prim.lambda.capture](https://timsong-cpp.github.io/cppwp/n4659/expr.prim.lambda.capture#7)§8.1.5.2¶7 says:

    A lambda-expression with an associated capture-default that does not explicitly capture *this or a variable with automatic storage duration [...], is said to implicitly capture the entity (i.e., *this or a variable) if the compound-statement:
    - odr-uses the entity (in the case of a variable),
    - [...]

The use of a constitutes an odr-use. But since a has static storage duration rather than automatic storage duration, it is not implicitly captured.

Since a is neither explicitly nor implicitly captured, a in the lambda expression simply refers to the global variable a.

(Note: It is also disallowed to capture a explicitly, because of [expr.prim.lambda.capture](https://timsong-cpp.github.io/cppwp/n4659/expr.prim.lambda.capture#4)§8.1.5.2¶4):

    The identifier in a simple-capture is looked up using the usual rules for unqualified name lookup; each such lookup shall find an entity. An entity that is designated by a simple-capture is said to be explicitly captured, and shall be *this (when the simple-capture is “this” or “* this”) or a variable with automatic storage duration declared in the reaching scope of the local lambda expression.
</p>
</details>

#### 6. :skull::skull: 

```   
#include <iostream>

int main() {
    int i = '3' - '2';
    std::cout << i;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 1
'3' and '2' are both ordinary character literals, with type char. But we don't know which values they have! That's up to the implementation:

[lex.ccon](https://timsong-cpp.github.io/cppwp/n4659/lex.ccon#2)§5.13.3¶2:

    An ordinary character literal that contains a single c-char representable in the execution character set has type char, with value equal to the numerical value of the encoding of the c-char in the execution character set.

operator- subtracts the value of the second operand from the value of the first operand. But how can we know what the result is, when we don't know the values of the operands?

[lex.charset](https://timsong-cpp.github.io/cppwp/n4659/lex.charset#3)§5.3¶3:

    In both the source and execution basic character sets, the value of each character after 0 in the above list of decimal digits shall be one greater than the value of the previous.

(The "above list of decimal digits" is simply 0123456789.)

So we know that the difference between '3' and '2' is 1, even if we don't know their actual values. Importantly, this is only true for decimal digits, and not for instance for regular letters. There is no guarantee that 'b' - 'a' is 1.
</p>
</details>

#### 7. :skull::skull:

```
#include <iostream>

struct A {
  A() { foo(); }
  virtual ~A() { foo(); }
  virtual void foo() { std::cout << "1"; }
  void bar() { foo(); }
};

struct B : public A {
  virtual void foo() { std::cout << "2"; }
};

int main() {
  B b;
  b.bar();
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 121

Even though foo() is virtual, it is not considered to be so during the execution of constructors and destructors.

Rationale:

If an object of type B is being constructed, first the constructor of A is called, then the constructor of B. Thus, during A's constructor, the "B part" of the object has not been constructed yet, and should not be used. One could easily imagine that B::foo() would use the "B part" of the object, so it would be dangerous for A's constructor to call it.

When the object is destroyed, B's destructor is called first, then A's destructor, leading to the same problem.
</p>
</details>

#### 8. :skull:

```
#include <iostream>
using namespace std;

int foo() {
  cout << 1;
  return 1;
}

void bar(int i = foo()) {}

int main() {
  bar();
  bar();
}
```
<details><summary><b>Answer</b></summary>
<p>

####  The program is guaranteed to output: 11

Is foo called both times or just once? The C++ standard says this in [dcl.fct.default](https://timsong-cpp.github.io/cppwp/n4659/dcl.fct.default#9)§11.3.6¶9: "A default argument is evaluated each time the function is called with no argument for the corresponding parameter."

Thus, foo is called twice.

</p>
</details>

#### 9. :skull::skull:
```
#include <iostream>

int main() {
  int a = 0;
  decltype((a)) b = a;
  b++;
  std::cout << a << b;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 11

According to [dcl.type.simple](https://timsong-cpp.github.io/cppwp/n4659/dcl.type.simple#4)§10.1.7.2¶4 in the C++ standard:
"The type denoted by decltype(e) is deﬁned as follows:
- if e is an unparenthesized id-expression naming a structured binding ([dcl.struct.bind](https://timsong-cpp.github.io/cppwp/n4659/dcl.struct.bind)§11.5), decltype(e) is the referenced type as given in the specification of the structured binding declaration;
— if e is an unparenthesized id-expression or an unparenthesized class member access ([expr.ref](https://timsong-cpp.github.io/cppwp/n4659/expr.ref)§8.2.5), decltype(e) is the type of the entity named by e. If there is no such entity, or if e names a set of overloaded functions, the program is ill-formed;
— otherwise, if e is an xvalue, decltype(e) is T&&, where T is the type of e;
— otherwise, if e is an lvalue, decltype(e) is T&, where T is the type of e;
— otherwise, decltype(e) is the type of e."

Because a is encapsulated in parentheses, it doesn't qualify for the first case, it is treated as an lvalue, therefore b's type is int&, not int.

</p>
</details>

#### 10. :skull:

```
#include <iostream>
     
int main() {
    char* str = "X";
    std::cout << str;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program has a compilation error 

According to [lex.string](https://timsong-cpp.github.io/cppwp/n4659/lex.string#8)§5.13.5¶8 in the standard: "A narrow string literal has type “array of n const char”"

An array of n const char converts to a pointer to const char. A note in [conv.qual](https://timsong-cpp.github.io/cppwp/n4659/conv.qual#4)§7.5¶4 extrapolates from the preceeding normative passages that "a prvalue of type “pointer to cv1 T” can be converted to a prvalue of type “pointer to cv2 T” if “cv2 T” is more cv-qualified than “cv1 T”." In this case however, char* is less cv-qualified than const char *, and the conversion is not allowed.

Note: While most compilers still allow char const[] to char* conversion with just a warning, this is not a legal conversion since C++11.

See also http://dev.krzaq.cc/stop-assigning-string-literals-to-char-star-already/
</p>
</details>

#### 11. :skull::skull::skull:

```
#include <iostream>

struct X {
  X() { std::cout << "X"; }
};

struct Y {
  Y(const X &x) { std::cout << "Y"; }
  void f() { std::cout << "f"; }
};

int main() {
  Y y(X());
  y.f();
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program has a compilation error 

The compilation error is on the line y.f(), but the source of the problem is Y y(X());

This could be interpreted as a a variable definition (which was the intention of the programmer in this example), or as a definition of a function y, returning an object of type Y, taking a function (with no arguments, returning an object of type X) as its argument.

The compiler is required by the standard to choose the second interpretation, which means that y.f() does not compile (since y is now a function, not an object of type Y).

Wikipedia has a concise explanation: http://en.wikipedia.org/wiki/Most_vexing_parse, and the standard has more in [stmt.ambig](https://timsong-cpp.github.io/cppwp/n4659/stmt.ambig)§6.8.

To fix the problem, change Y y(X()) to either Y y{X{}} (modern C++) or Y y((X())) (pre-C++11)
</p>
</details>

#### 12. :skull::skull:

```
#include <iostream>
#include <utility>

int y(int &) { return 1; }
int y(int &&) { return 2; }

template <class T> int f(T &&x) { return y(x); }
template <class T> int g(T &&x) { return y(std::move(x)); }
template <class T> int h(T &&x) { return y(std::forward<T>(x)); }

int main() {
  int i = 10;
  std::cout << f(i) << f(20);
  std::cout << g(i) << g(20);
  std::cout << h(i) << h(20);
  return 0;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 112212

The T&& in the templated functions do not necessarily denote an rvalue reference, it depends on the type that is used to instantiate the template. If instantiated with an lvalue, it collapses to an lvalue reference, if instantiated with an rvalue, it collapses to an rvalue reference. See note [1].

Scott Meyers has written a very good article about this, where he introduces the concept of "universal references" (the official term is "forwarding reference") http://isocpp.org/blog/2012/11/universal-references-in-c11-scott-meyers

In this example, all three functions are called once with an lvalue and once with an rvalue. In all cases, calling with an lvalue (i) collapses T&& x to T& x (an lvalue reference), and calling with an rvalue (20) collapses T&& x to T&& x (an rvalue reference). Inside the functions, x itself is always an lvalue, no matter if its type is an rvalue reference or an lvalue reference.

-For the first example, y(int&) is called for both cases. Output: 11.
-For the second example, move(x) obtains an rvalue reference, and y(int&&)is called for both cases. Output: 22.
-For the third example, forward<T>(x) obtains an lvalue reference when x is an lvalue reference, and an rvalue reference when x is an rvalue reference, resulting in first a call to y(int&)and then a call to y(int&&). Output: 12.

Note [1]: [dcl.ref](https://timsong-cpp.github.io/cppwp/n4659/dcl.ref#6)§11.3.2¶6 in the standard: "If a typedef-name (§10.1.3, §17.1) or a decltype-specifier (§10.1.7.2) denotes a type TR that is a reference to a type T, an attempt to create the type “lvalue reference to cv TR” creates the type “lvalue reference to T”, while an attempt to create the type “rvalue reference to cv TR” creates the type TR." The example at the end of that paragraph is worth a look.

Note from the contributor: This demonstrates Scott Meyers's advice to use std::forward for forwarding references, and std::move for rvalue references.

</p>
</details>

#### 13. :skull::skull:

```
#include <iostream>

struct E
{
  E() { std::cout << "1"; }
  E(const E&) { std::cout << "2"; }
  ~E() { std::cout << "3"; }
};

E f()
{ 
  return E();
}

int main()
{
  f();
}
```
<details><summary><b>Answer</b></summary>
<p>

####  The program is guaranteed to output: 13

In f(), an E object is constructed, and 1 is printed. This object is then returned to main(), and one could expect the copy constructor to be called, printing 2.

However, E() is a prvalue and as such does not constitute an object just yet by [basic.lval](https://timsong-cpp.github.io/cppwp/n4659/basic.lval#1)§6.10¶1

    A prvalue is an expression whose evaluation initializes an object or a bit-field, or computes the value of the operand of an operator, as specified by the context in which it appears.

A prvalue only creates a temporary when needed, for instance to create an xvalue. In those cases, a temporary materialization conversion happens ([conv.rval](https://timsong-cpp.github.io/cppwp/n4659/conv.rval#1)§7.4¶1). In this case however, no temporary is needed, and none is created.
A pr
[stmt.return](https://timsong-cpp.github.io/cppwp/n4659/stmt.return#2)§9.6.3¶2 says:

    (...) the return statement initializes the glvalue result or prvalue result object of the (explicit or implicit) function call by copy-initialization from the operand.

And copy-initialization for a class-type by [dcl.init](https://timsong-cpp.github.io/cppwp/n4659/dcl.init#17)§11.6¶17 goes through:

    If the initializer expression is a prvalue and the cv-unqualified version of the source type is the same class as the class of the destination, the initializer expression is used to initialize the destination object.

Which means that no copy or move constructor is called at all. This implies that the copy and move constructor could very well be deleted, and the code would still compile just fine.

The output is thus 13 because of the constructor followed by the destructor call.
</p>
</details>

#### 14. :skull::skull:
```
#include <iostream>

void f(unsigned int) { std::cout << "u"; }
void f(int)          { std::cout << "i"; }
void f(char)         { std::cout << "c"; }

int main() {
    char x = 1;
    char y = 2;
    f(x + y);
}
```
<details><summary><b>Answer</b></summary>
<p>

####  The program is unspecified / implementation defined 

The type of the sum of two chars is actually not uniquely specified. We do however know it's not char.

Before being passed to operator +, the operands (x and y) go through a conversion. [expr.add](https://timsong-cpp.github.io/cppwp/n4659/expr.add#1)§8.7¶1:

    The additive operators + and - group left-to-right. The usual arithmetic conversions are performed for operands of arithmetic or enumeration type.

What are "the usual arithmetic conversions"?

[expr](https://timsong-cpp.github.io/cppwp/n4659/expr#11)§8¶11:

    Many binary operators that expect operands of arithmetic or enumeration type cause conversions and yield result types in a similar way. The purpose is to yield a common type, which is also the type of the result. This pattern is called the usual arithmetic conversions, which are defined as follows:
    - [a bunch of rules for floats, enums etc]
    - Otherwise, the integral promotions (7.6) shall be performed on both operands

So both chars go through integral promotions. Those are defined in [conv.prom](https://timsong-cpp.github.io/cppwp/n4659/conv.prom#1)§7.6¶1:

    A prvalue of an integer type other than bool, char16_t, char32_t, or wchar_t whose integer conversion rank (7.15) is less than the rank of int can be converted to a prvalue of type int if int can represent all the values of the source type; otherwise, the source prvalue can be converted to a prvalue of type unsigned int.

So a char gets converted to an int if int can fit all possible values of a char. But that's not necessarily the case!

First, int could actually be the same size as char. [basic.fundamental](https://timsong-cpp.github.io/cppwp/n4659/basic.fundamental#2)§6.9.1¶2:

    There are five standard signed integer types : “signed char”, “short int”, “int”, “long int”, and “long long int”. In this list, each type provides at least as much storage as those preceding it in the list.

Note that it says "at least as much storage", it doesn't have to be more. So for instance you could have an sixteen bit system where both char and int are sixteen bits.

Second, char can be either signed or unsigned, it's up to the implementation: [basic.fundamental](https://timsong-cpp.github.io/cppwp/n4659/basic.fundamental#1)§6.9.1¶1:

    It is implementation-defined whether a char object can hold negative values.

int is signed, so if char is also signed, all possible values of char will fit in an int. However, if char is unsigned, and int and char is the same size, char can actually hold larger values than int! In the former case, chars get promoted to ints, but in the latter case, chars get promoted to unsigned int before being summed.

So in practice, most systems will call f(int) and print i, but some might call f(unsigned int) and print u, and they would both be confirming to the standard.
</p>
</details>

#### 15. :skull:
```
#include <iostream>

struct GeneralException {
  virtual void print() { std::cout << "G"; }
};

struct SpecialException : public GeneralException {
  void print() override { std::cout << "S"; }
};

void f() { throw SpecialException(); }

int main() {
  try {
    f();
  }
  catch (GeneralException e) {
    e.print();
  }
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: G
We throw a SpecialException. It is derived from GeneralException, but is caught by value, so e will have the dynamic type GeneralException, not SpecialException. This is known as slicing.

Instead, we should have caught it by reference catch (GeneralException& e), then its dynamic type would be SpecialException, and the program would output S.

</p>
</details>

#### 16. :skull:
```
#include <iostream>

int a;

int main () {
    std::cout << a;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 0
Since a has static storage duration and no initializer, it is guaranteed to be zero-initialized. Had a been defined as a local non-static variable inside main(), this would not have happened.

Note: int a has static storage duration because it is declared at namespace scope. It does not need to have static in front of it, that would only denote internal linkage.
</p>
</details>

#### 17. :skull::skull:
```
#include <iostream>
using namespace std;

class C {
public:
  explicit C(int) {
    std::cout << "i";
  };
  C(double) {
    std::cout << "d";
  };
};

int main() {
  C c1(7);
  C c2 = 7;
}
```
<details><summary><b>Answer</b></summary>
<p>

####  The program is guaranteed to output: id
These are two examples of initialization. The first form, C c1(7), is called direct-initialization, the second, C c2 = 7, is called copy-initialization. In most cases they are equivalent, but in this example they are not, since the int constructor is explicit.

The key is in [over.match.copy](https://timsong-cpp.github.io/cppwp/n4659/over.match.copy#1)§16.3.1.4¶1 :
"[...] as part of a copy-initialization of an object of class type, a user-defined conversion can be invoked to convert an initializer expression to the type of the object being initialized. [...] the candidate functions are selected as follows:
- [...]
- When the type of the initializer expression is a class type "cv S", the non-explicit conversion functions of S and its base classes are considered. [...]
(emphasis added)

And how is direct-initialization defined?

[dcl.init](https://timsong-cpp.github.io/cppwp/n4659/dcl.init#16)§11.6¶16: "The initialization that occurs in the forms
T x(a);
T x{a};
(...) is called direct-initialization."

So the int constructor is not even considered for initialization in the second case. Instead, a standard conversion sequence is used to convert the integer literal to a double, and the double constructor (the only candidate) is used.

</p>
</details>

#### 18. :skull:
```
#include <iostream>

class A {
public:
  A() { std::cout << 'a'; }
  ~A() { std::cout << 'A'; }
};

class B {
public:
  B() { std::cout << 'b'; }
  ~B() { std::cout << 'B'; }
  A a;
};

int main() { B b; }
```
<details><summary><b>Answer</b></summary>
<p>

####  The program is guaranteed to output: abBA
Member variables are initialized before the constructor is called. The destructor is called before member variables are destroyed.
</p>
</details>

#### 19. :skull:
```
#include <iostream>

int j = 1;

int main() {
  int& i = j, j;
  j = 2;
  std::cout << i << j;
}
```
<details><summary><b>Answer</b></summary>
<p>

####  The program is guaranteed to output: 12
There are two variables named j here. The first is the global one, the second is local to main. A variable is in scope from the point of its declaration until the end of the region in which it was declared, except for when another variable with the same name is in scope.

It's easiest to first look at the scope of the local j declared inside main, which extends from the , to the }. The scope of the global j is from when it was declared until the end of the program, except for when the local j is in scope.

So the reference i refers to the global j, since the local j is not yet in scope. When we set j=2, we modify the local j, and i is not affected.

This example is almost identical to [basic.scope.declarative](https://timsong-cpp.github.io/cppwp/n4659/basic.scope.declarative#2)§6.3.1¶2 in the C++ standard, which has the following explanation:

    the identifier j is declared twice as a name (and used twice). The declarative region of the first j includes the entire example. The potential scope of the first j begins immediately after that j and extends to the end of the program, but its (actual) scope excludes the text between the , and the }. The declarative region of the second declaration of j (the j immediately before the semicolon) includes all the text between { and }, but its potential scope excludes the declaration of i. The scope of the second declaration of j is the same as its potential scope.
</p>
</details>

#### 20. :skull::skull:
```
#include <variant>
#include <iostream>
 
using namespace std;

int main() {
   variant<int, double, char> v;
   cout << v.index();
}
```
<details><summary><b>Answer</b></summary>
<p>

####  The program is guaranteed to output: 0
std::variant can hold a value of any one of its alternative types, or no value. To refer to the alternative types, it uses an index i.
[variant.ctor](https://timsong-cpp.github.io/cppwp/n4659/variant.ctor#1)§23.7.3.1¶1 in the standard:

    let i be in the range [0, sizeof...(Types)), and Ti be the ith type in Types....

In our case, T0 means int, T1 means double, and T2 means char.

Now what happens if you define a variant without initializing it with a certain type? The default constructor will pick the type T0, in our case int, and value-initialize it.
[variant.ctor](https://timsong-cpp.github.io/cppwp/n4659/variant.ctor#2)§23.7.3.1¶2:

    constexpr variant() noexcept
    Constructs a variant holding a value-initialized value of type T0 

Finally, we call index() and print the result. index() returns the index of the type of the contained value. The contained value is an int, aka T0, so 0 is returned.
[variant.status](https://timsong-cpp.github.io/cppwp/n4659/variant.status#3)§23.7.3.5¶3:

    constexpr size_t index() const noexcept;
    _Effects+: If [it doesn't contain a value], returns variant_npos. Otherwise, returns the zero- based index of the alternative of the contained value.
</p>
</details>

#### 21. :skull::skull::skull:
```
#include <iostream>

int main() {
 int a = 5,b = 2;
 std::cout << a+++++b;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program has a compilation error 
Some might expect the lexer to parse the series of characters a+++++b as follows:

a++ + ++b

but according to the [maximal munch principle](https://en.wikipedia.org/wiki/Maximal_munch), the lexer will take the longest sequence of characters to produce the next token(with a few exceptions).

[lex.pptoken](https://timsong-cpp.github.io/cppwp/n4659/lex.pptoken#3)§5.4¶3 in the C++ standard:
"the next preprocessing token is the longest sequence of characters that could constitute a preprocessing token, even if that would cause further lexical analysis to fail (...)."

So after parsing a++, it is not allowed to just parse +, it has to parse ++. The sequence is thus parsed as:

a ++ ++ + b

which is ill-formed since post-increment requires a modifiable lvalue but the first post-increment will produce a prvalue, as per [expr.post.incr](https://timsong-cpp.github.io/cppwp/n4659/expr.post.incr#1)§8.2.6¶1 in the C++ standard:
"The value of a postfix ++ expression is the value of its operand. (...) The result is a prvalue."
</p>
</details>

#### 22. :skull::skull:
```    
#include <iostream>

template <class T> void f(T &i) { std::cout << 1; }

template <> void f(const int &i) { std::cout << 2; }

int main() {
  int i = 42;
  f(i);
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 1
For the call f(i), since the type of i is int, template argument deduction deduces T = int.

The explicit specialization template <> void f(const int &i) has T = const int, which is not the same type as int, so it doesn't match.

Instead, template <class T> void f(T &i) with T = int is used to create the implicit instantiation void f<int>(int&).
</p>
</details>

#### 23. :skull:
```
#include <iostream>

struct A {
  A() { std::cout << "A"; }
};
struct B {
  B() { std::cout << "B"; }
};

class C {
public:
  C() : a(), b() {}

private:
  B b;
  A a;
};

int main()
{
    C();
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: BA
The initialization order of member variables is determined by their order of declaration, not their order in the initialization list.
</p>
</details>

#### 24. :skull::skull:
```
#include <iostream>
#include <string>

void f(const std::string &) { std::cout << 1; }

void f(const void *) { std::cout << 2; }

int main() {
  f("foo");
  const char *bar = "bar";
  f(bar);
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 22
A string literal is not a std::string, but a const char[] . If the compiler was to choose f(const std::string&), it would have to go through a user defined conversion and create a temporary std::string. Instead, it prefers f(const void*), which requires no user defined conversion.
</p>
</details>

#### 25. :skull:
```
#include <iostream>

void f(int) { std::cout << 1; }
void f(unsigned) { std::cout << 2; }

int main() {
  f(-2.5);
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program has a compilation error 
This overload is ambiguous. Why?

There are two viable functions for the call f(-2.5). For the compiler to select one, one of them needs to be better than the other, or the program is ill-formed. In our case, they are equally good, making the program ill-formed.

According to [over.match.best](https://timsong-cpp.github.io/cppwp/n4659/over.match.best)§16.3.3 in the standard, a viable one-argument function is better than another if the conversion sequence for the argument is better. So why isn't the int conversion sequence better than the unsigned conversion sequence, given that the double is signed?

All conversions are given a rank, and both "double => int" and "double => unsigned int" are of type "floating-integral conversion", which has rank "conversion". See Table 13 in the standard and [conv.fpint](https://timsong-cpp.github.io/cppwp/n4659/conv.fpint)§7.10. Since they have the same rank, no conversion is better than the other, and the program is ill-formed.
</p>
</details>

#### 26. :skull:
```
#include <iostream>

void f(float) { std::cout << 1; }
void f(double) { std::cout << 2; }

int main() {
  f(2.5);
  f(2.5f);
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 21
The type of a floating point literal is double.
</p>
</details>

#### 27. :skull:
```
#include <iostream>

int main() {
  for (int i = 0; i < 3; i++)
    std::cout << i;
  for (int i = 0; i < 3; ++i)
    std::cout << i;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 012012
Whether you post-increment or pre-increment i, its value does not change until after the loop body has executed.

</p>
</details>

#### 28. :skull:
```
#include <iostream>

class A {
public:
  void f() { std::cout << "A"; }
};

class B : public A {
public:
  void f() { std::cout << "B"; }
};

void g(A &a) { a.f(); }

int main() {
  B b;
  g(b);
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: A
As long as A::f() is not virtual, A::f() will always be called, even if the reference or pointer is actually referring to an object of type B.
</p>
</details>

#### 29. :skull:
``` 
#include <iostream>

class A {
public:
  virtual void f() { std::cout << "A"; }
};

class B : public A {
public:
  void f() { std::cout << "B"; }
};

void g(A a) { a.f(); }

int main() {
  B b;
  g(b);
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: A
g(A a) takes an object of type A by value, not by reference or pointer. This means that A's copy constructor is called on the object passed to g() (no matter if the object we passed was of type B), and we get a brand new object of type A inside g(). This is commonly referred to as slicing.
</p>
</details>

#### 30. :skull:
```
#include <iostream>

int f(int &a, int &b) {
  a = 3;
  b = 4;
  return a + b;
}

int main() {
  int a = 1;
  int b = 2;
  int c = f(a, a);
  std::cout << a << b << c;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 428
When f() is called with a as both parameters, both arguments refer to the same variable. This is known as aliasing. First, a is set to 3, then a is set to 4, then 4+4 is returned. b is never modified.
</p>
</details>

#### 31. :skull:
```
#include <iostream>

int main() {
  static int a;
  std::cout << a;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 0
Since a is a static local variable, it is automatically zero-initialized. This would not have happened if we removed the keyword static, making it a non-static local variable.

[basic.start.static](https://timsong-cpp.github.io/cppwp/n4659/basic.start.static#2)§6.6.2¶2 in the standard:

    If constant initialization is not performed, a variable with static storage duration (6.7.1) or thread storage duration (6.7.2) is zero-initialized (11.6)

a has static storage duration and is not constant initialized , so it gets zero-initialized.

[dcl.init](https://timsong-cpp.github.io/cppwp/n4659/dcl.init#6)§11.6¶6:

    To zero-initialize an object or reference of type T means:
    — if T is a scalar type (6.9), the object is initialized to the value obtained by converting the integer literal 0 (zero) to T;

So a gets initialized to 0.
</p>
</details>

#### 32. :skull:
```
#include <iostream>

class A {
public:
  A() { std::cout << "a"; }
  ~A() { std::cout << "A"; }
};

class B {
public:
  B() { std::cout << "b"; }
  ~B() { std::cout << "B"; }
};

class C {
public:
  C() { std::cout << "c"; }
  ~C() { std::cout << "C"; }
};

A a;
int main() {
  C c;
  B b;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: acbBCA
[basic.start.dynamic](https://timsong-cpp.github.io/cppwp/n4659/basic.start.dynamic#5)§6.6.3¶5 in the standard:
"It is implementation-defined whether the dynamic initialization of a non-local inline variable with static storage duration is sequenced before the first statement of main or is deferred. If it is deferred, it strongly happens before any non-initialization odr-use of that variable."

Since A() is not constexpr, the initialization of a is dynamic. There are two possibilities:
- a is initialized before main() is called, i.e. before b or c are initialized.
- a is not initialized before main(). It is however guaranteed to be initialized before the the use of any function defined in the same translation unit, i.e. before the constructors of b and c are called.

Then, b and c are initialized in order.

Before main() exits, b and c are destructed in the reverse order of their construction. Then, when main() returns, a is destructed as per §6.6.4 in the standard:
"Destructors for initialized objects (...) with static storage duration are called as a result of returning from main."
</p>
</details>

#### 33. :skull::skull:
```
#include <iostream>

class A {
public:
  A() { std::cout << "a"; }
  ~A() { std::cout << "A"; }
};

class B {
public:
  B() { std::cout << "b"; }
  ~B() { std::cout << "B"; }
};

class C {
public:
  C() { std::cout << "c"; }
  ~C() { std::cout << "C"; }
};

A a;

void foo() { static C c; }
int main() {
  B b;
  foo();
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: abcBCA
[basic.start.dynamic](https://timsong-cpp.github.io/cppwp/n4659/basic.start.dynamic#4)§6.6.3¶4 in the standard:
"It is implementation-defined whether the dynamic initialization of a non-local non-inline variable with static storage duration is sequenced before the first statement of main or is deferred. If it is deferred, it strongly happens before any non-initialization odr-use of any non-inline function or non-inline variable defined in the same translation unit as the variable to be initialized."

Since A() is not constexpr, the initialization of a is dynamic. There are two possibilities:
- a is initialized before main() is called, i.e. before b is initialized.
- a is not initialized before main(). It is however guaranteed to be initialized before the the use of any function defined in the same translation unit, i.e. before the constructor of b is called.

When execution reaches B b, it is initialized as normal. Static local variables are initialized the first time control passes through their declaration, so c is initialized next. As main() is exited, its local variable b goes out of scope, and is destroyed. Finally, all static variables are destroyed in reverse order of their initialization, first c, then a.
</p>
</details>

#### 34. :skull::skull::skull:
```
#include <iostream>
#include <exception>

int x = 0;

class A {
public:
  A() {
    std::cout << 'a';
    if (x++ == 0) {
      throw std::exception();
    }
  }
  ~A() { std::cout << 'A'; }
};

class B {
public:
  B() { std::cout << 'b'; }
  ~B() { std::cout << 'B'; }
  A a;
};

void foo() { static B b; }

int main() {
  try {
    foo();
  }
  catch (std::exception &) {
    std::cout << 'c';
    foo();
  }
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: acabBA
Static local variables are initialized the first time control passes through their declaration. The first time foo() is called, b is attempted initialized. Its constructor is called, which first constructs all member variables. This means A::A() is called, printing a. A::A() then throws an exception, the constructor is aborted, and neither b or B::a are actually considered constructed. In the catch-block, c is printed, and then foo() is called again. Since b was never initialized the first time, it tries again, this time succeeding, printing ab. When main() exits, the static variable b is destroyed, first calling the destructor printing B, and then destroying member variables, printing A. 
</p>
</details>

#### 35. :skull:
```   
#include <iostream>

class A {
public:
  virtual void f() { std::cout << "A"; }
};

class B : public A {
private:
  void f() { std::cout << "B"; }
};

void g(A &a) { a.f(); }

int main() {
  B b;
  g(b);
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: B
The "trick" here is that B::f() is called even though it is private.

As [class.access.virt](https://timsong-cpp.github.io/cppwp/n4659/class.access.virt#2)§14.5¶2 in the standard puts it: "Access is checked at the call point using the type of the expression used to denote the object for which the member function is called". The call point here being a.f(), and the type of the expression is A&.
</p>
</details>

#### 36. :skull:
```
#include <iostream>
#include <limits>

int main() {
  unsigned int i = std::numeric_limits<unsigned int>::max();
  std::cout << ++i;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 0
Unsigned integers have well defined behaviour when they overflow. When you go one above the largest representable unsigned int, you end up back at zero.

According to [basic.fundamental](https://timsong-cpp.github.io/cppwp/n4659/basic.fundamental#4)§6.9.1¶4 in the C++ standard: "Unsigned integers, declared unsigned, shall obey the laws of arithmetic modulo 2^n where n is the number of bits in the value representation of that particular size of integer."
</p>
</details>

#### 37. :skull::skull:
```
#include <iostream>
#include <limits>

int main() {
  int i = std::numeric_limits<int>::max();
  std::cout << ++i;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is undefined 
Signed integer overflow is undefined behaviour according to the standard [expr](https://timsong-cpp.github.io/cppwp/n4659/expr#4)§8¶4: "If during the evaluation of an expression, the result is not mathematically defined or not in the range of representable values for its type, the behavior is undefined."

Most implementations will just wrap around, so if you try it out on your machine, you will probably see the same as if you had done
std::cout << std::numeric_limits<int>::min();

Relying on such undefined behaviour is however not safe. For an interesting example, see http://stackoverflow.com/questions/7682477/why-does-integer-overflow-on-x86-with-gcc-cause-an-infinite-loop
</p>
</details>

#### 38. :skull:
```
#include <iostream>

int main() {
  int i = 42;
  int j = 1;
  std::cout << i / --j;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is undefined 
Integer division by zero is undefined behaviour. According to [expr.mul](https://timsong-cpp.github.io/cppwp/n4659/expr.mul#4)§8.6¶4 in the standard: "If the second operand of / or % is zero the behavior is undefined."
</p>
</details>

#### 39. :skull:
```
#include <iostream>

struct A {
  virtual std::ostream &put(std::ostream &o) const {
    return o << 'A';
  }
};

struct B : A {
  virtual std::ostream &put(std::ostream &o) const {
    return o << 'B';
  }
};

std::ostream &operator<<(std::ostream &o, const A &a) {
  return a.put(o);
}

int main() {
  B b;
  std::cout << b;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: B
This is a way to get polymorphic behaviour for operator <<.
</p>
</details>

#### 40. :skull:
```
#include <iostream>

struct A {
  A() { std::cout << "A"; }
  A(const A &a) { std::cout << "B"; }
  virtual void f() { std::cout << "C"; }
};

int main() {
  A a[2];
  for (auto x : a) {
    x.f();
  }
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: AABCBC
When the array is initialized, the default constructor is called once for each of the two objects in it.

Then we iterate over the array using auto, which in our case is deduced to be A. This means the copy constructor will be called before f() for each iteration, printing BCBC. (Just as if we had written for (A x: a).

If we want to avoid the copy constructor, we can write for (auto& x : a) instead. Then the loop would print CC. (Just as if we had written for (A& x: a).
</p>
</details>

#### 41. :skull:
```
#include <iostream>
struct X {
  X() { std::cout << "X"; }
};

int main() { X x(); }
```
<details><summary><b>Answer</b></summary>
<p>

#### This program has no output.
X x(); is a function prototype, not a variable definition. Remove the parentheses (or since C++11, replace them with {}), and the program will output X. 
</p>
</details>

#### 42. :skull::skull:
```
#include <iostream>

struct X {
  X() { std::cout << "a"; }
  X(const X &x) { std::cout << "b"; }
  const X &operator=(const X &x) {
    std::cout << "c";
    return *this;
  }
};

int main() {
  X x;
  X y(x);
  X z = y;
  z = x;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: abbc
The first line in main(), X x; is straightforward, it calls the default constructor.

The next two lines is the heart of the question: The difference between X y(x) and X z = y is not that the first calls the copy constructor, and the second calls the copy assignment operator. The difference is that the first is direct initialization ([dcl.init](https://timsong-cpp.github.io/cppwp/n4659/dcl.init#16)§11.6¶16 in the standard) and the second is copy initialization ([dcl.init](https://timsong-cpp.github.io/cppwp/n4659/dcl.init#15)§11.6¶15).

[dcl.init](https://timsong-cpp.github.io/cppwp/n4659/dcl.init#17)§11.6¶17 says: "If the initialization is direct-initialization, or if it is copy-initialization where the (...) source type is the same class as (...) the class of the destination, constructors are considered." So both our cases use the copy constructor.

Not until z = x; do we have an actual assignment that uses the assignment operator.

See http://stackoverflow.com/questions/1051379/is-there-a-difference-in-c-between-copy-initialization-and-direct-initializati/1051468#1051468 for a more detailed discussion of direct vs. copy initialization.
</p>
</details>

#### 43. :skull:
```
#include <iostream>
#include <vector>

int main() {
  std::vector<int> v1(1, 2);
  std::vector<int> v2{ 1, 2 };
  std::cout << v1.size() << v2.size();
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 12
To answer this we need to look at overload resolution of vector's constructors:

[vector.cons](https://timsong-cpp.github.io/cppwp/n4659/vector.cons#6)§26.3.11.2¶6 says (somewhat redacted):
vector(size_type n, const T& value);
Effects: Constructs a vector with n copies of value, using the specified allocator

So v1 contains one "2".

[over.match.list](https://timsong-cpp.github.io/cppwp/n4659/over.match.list)§16.3.1.7 says (in summary) that when non-aggregate classes (such as vector) are list-initialized† and have an initializer list constructor (again, like vector), that constructor is chosen, and the argument list consists of the initializer list as a single argument.
(†: [dcl.init.list](https://timsong-cpp.github.io/cppwp/n4659/dcl.init.list#1)§11.6.4¶1: List-initialization is initialization of an object or reference from a braced-init-list.)

So v2 is initialized from the elements (aka initializer-clauses) in the braced-init-list, and contains the elements "1" and "2".
</p>
</details>

#### 44. :skull:
```
#include <iostream>

int main() {
  int a = 0;
  decltype(a) b = a;
  b++;
  std::cout << a << b;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 01
According to [dcl.type.simple](https://timsong-cpp.github.io/cppwp/n4659/dcl.type.simple#4)§10.1.7.2¶4 in the C++ standard:
"The type denoted by decltype(e) is deﬁned as follows:
— if e is an unparenthesized id-expression [...], decltype(e) is the type of the entity named by e."

The type of a is int, so the type of b is also int.
</p>
</details>

#### 45. :skull::skull:
```
#include <iostream>
int main() {
  std::cout << 1["ABC"];
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: B
[expr.sub](https://timsong-cpp.github.io/cppwp/n4659/expr.sub#1)§8.2.1¶1 in the standard says "The expression E1[E2] is identical (by definition) to *((E1)+(E2))".

In our case 1["ABC"] is identical to *(1+"ABC"). Since the plus operator is commutative, this is identical to *("ABC"+1), which is identical to the more familiar "ABC"[1].
</p>
</details>

#### 46. :skull::skull:
```
#include <initializer_list>
#include <iostream>

struct A {
  A() { std::cout << "1"; }

  A(int) { std::cout << "2"; }

  A(std::initializer_list<int>) { std::cout << "3"; }
};

int main(int argc, char *argv[]) {
  A a1;
  A a2{};
  A a3{ 1 };
  A a4{ 1, 2 };
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 1133
a1 is default initialized, as described in [dcl.init](https://timsong-cpp.github.io/cppwp/n4659/dcl.init#12)§11.6¶12.

a2 doesn't actually use the initializer_list constructor with a list of zero elements, but the default constructor:
[dcl.init.list](https://timsong-cpp.github.io/cppwp/n4659/dcl.init.list#3)§11.6¶3:
List-initialization of an object or reference of type T is defined as follows:
- (...)
- Otherwise, if the initializer list has no elements and T is a class type with a default constructor, the object is value-initialized.
- Otherwise, if T is a specialization of std::initializer_list, the object is constructed as described below.

a3's and a4's constructor is chosen in overload resolution, as described in [over.match.list](https://timsong-cpp.github.io/cppwp/n4659/over.match.list)§16.3.1.7:

"When objects of non-aggregate class type T are list-initialized (...), overload resolution selects the constructor in two phases:
— Initially, the candidate functions are the initializer-list constructors (§11.6.4) of the class T and the argument list consists of the initializer list as a single argument.
— If no viable initializer-list constructor is found, overload resolution is performed again, where the candidate functions are all the constructors of the class T and the argument list consists of the elements of the initializer list."

Initializer list constructors are greedy, so even though A(int) constructor is available, the standard mandates that initializer_list<int> is prioritized, and if - and only if - it's not available, the compiler is allowed to look for other constructors. (This is why it is not recommended to provide a constructor that ambiguously overloads with an initializer_list constructor. See the answer to #4 in http://herbsutter.com/2013/05/09/gotw-1-solution/ )
</p>
</details>

#### 47. :skull::skull:
```
#include <iostream>
#include <string>
#include <future>

int main() {
  std::string x = "x";

  std::async(std::launch::async, [&x]() {
    x = "y";
  });
  std::async(std::launch::async, [&x]() {
    x = "z";
  });

  std::cout << x;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: z
The destructor of a future returned from async is required to block until the async task has finished (see elaboration below). Since we don't assign the futures that are returned from async() to anything, they are destroyed at the end of the full expression (at the end of the line in this case). [class.temporary](https://timsong-cpp.github.io/cppwp/n4659/class.temporary#4)§15.2¶4 in the standard: "Temporary objects are destroyed as the last step in evaluating the full-expression (§4.k) that (lexically) contains the point where they were created."

This means that the first async call is guaranteed to finish execution before async() is called the second time, so, while the assignments themselves may happen in different threads, they are synchronized.

Elaboration on synchronization:
According to [futures.async](https://timsong-cpp.github.io/cppwp/n4659/futures.async#5)§33.6.9¶5 of the standard:
Synchronization: Regardless of the policy argument,
[...]
If the implementation chooses the launch::async policy,
— the associated thread completion synchronizes with (§4.1) the return from the first function that successfully detects the ready status of the shared state or with the return from the last function that releases the shared state, whichever happens first.

In this case, the destructor of std::future<> returned by the async() call is "the last function that releases the shared state", therefore it synchronizes with (waits for) the thread completion.

Scott Meyers writes more about this http://scottmeyers.blogspot.com/2013/03/stdfutures-from-stdasync-arent-special.html

</p>
</details>

#### 48. :skull::skull:
```
#include <iostream>

class C {
public:
  C(int i) : i(i) { std::cout << i; }
  ~C() { std::cout << i + 5; }

private:
  int i;
};

int main() {
  const C &c = C(1);
  C(2);
  C(3);
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 127386
[class.temporary](https://timsong-cpp.github.io/cppwp/n4659/class.temporary#4)§15.2¶4 in the standard: "Temporary objects are destroyed as the last step in evaluating the full-expression (...) that (lexically) contains the point where they were created." This means that normally the temporaries returned from C(1), C(2), and C(3) should be destroyed at the end of the line.

However: [class.temporary](https://timsong-cpp.github.io/cppwp/n4659/class.temporary#6)§15.2¶6 states: "(...)when a reference is bound to a temporary. The temporary to which the reference is bound (...) persists for the lifetime of the reference", so the lifetime of the temporary returned by C(1) is extended for the lifetime of c, to the end of main(). The temporaries returned by C(2) and C(3) are still destroyed at the end of their lines of creation, so they get destroyed before the one returned by C(1).
</p>
</details>

#### 49. :skull:
```
#include <iostream>

class A;

class B {
public:
  B() { std::cout << "B"; }
  friend B A::createB();
};

class A {
public:
  A() { std::cout << "A"; }

  B createB() { return B(); }
};

int main() {
  A a;
  B b = a.createB();
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program has a compilation error 
There is a compilation error when attempting to declare A::createB() a friend of B. To declare A::createB() a friend of B, the compiler needs to know that that function exists. Since it has only seen the declaration of A so far, not the full definition, it cannot know this.
</p>
</details>

#### 50. :skull::skull:
```
#include <iostream>

using namespace std;

class A {
public:
  A() { cout << "a"; }
  ~A() { cout << "A"; }
};

int i = 1;

int main() {
label:
  A a;
  if (i--)
    goto label;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: aAaA
The Standard says this about jump statements:

[stmt.jump](https://timsong-cpp.github.io/cppwp/n4659/stmt.jump#2)§9.6¶2 Transfer [...] back past an initialized variable with automatic storage duration involves the destruction of objects with automatic storage duration that are in scope at the point transferred from but not at the point transferred to.
</p>
</details>

#### 51. :skull::skull::skull:
```
#include <iostream>

extern "C" int x;
extern "C" { int y; }

int main() {

	std::cout << x << y;

	return 0;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is undefined
According to [dcl.link](https://timsong-cpp.github.io/cppwp/n4659/dcl.link#7)§10.5¶7 in the standard : A declaration directly contained in a linkage-specification is treated as if it contains the extern specifier (§10.1.1) for the purpose of determining the linkage of the declared name and whether it is a definition.
extern "C" int x; //is just a declaration
extern "C" { int y; } //is a definition

And according to [basic.def.odr](https://timsong-cpp.github.io/cppwp/n4659/basic.def.odr#4)§6.2¶4: "Every program shall contain exactly one definition of every non-inline function or variable that is odr-used in that program outside of a discarded statement; no diagnostic required."

The result: x is never defined but it is optional for the compiler to print an error. The behaviour of this program is undefined.
</p>
</details>

#### 52. :skull::skull:
```
#include <iostream>
#include <vector>

int f() { std::cout << "f"; return 0;}
int g() { std::cout << "g"; return 0;}

void h(std::vector<int> v) {}

int main() {
    h({f(), g()});
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: fg
The goal of this question is to demonstrate that the evaluation order of elements in an initializer list is specified (as opposed to the arguments to a function call).

[dcl.init.list](https://timsong-cpp.github.io/cppwp/n4659/dcl.init.list#4)§11.6.4¶4: Within the initializer-list of a braced-init-list, the initializer-clauses, including any that result from pack expansions (§17.5.3), are evaluated in the order in which they appear.

If h took two ints instead of a vector<int>, and was called like this:
h(f(), g());
the program would be unspecified, and could either print fg or gf.
</p>
</details>

#### 53. :skull::skull::skull:
```
#include <functional>
#include <iostream>

template <typename T>
void call_with(std::function<void(T)> f, T val)
{
	f(val);
}

int main()
{
	auto print = [] (int x) { std::cout << x; };
	call_with(print, 42);
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program has a compilation error 
The compiler tries to deduce T for every parameter and checks if the deduced types match. Because a lambda is of completely different type, it cannot be matched against std::function<void(T)> and the deduction process fails.
This problem can be fixed by turning the first parameter into a so-called nondeduced context.

[temp.deduct.type](https://timsong-cpp.github.io/cppwp/n4659/temp.deduct.type#5)§17.8.2.5¶5 in the standard:

    The non-deduced contexts are:


        The nested-name-specifier of a type that was specified using a qualified-id.

        (...)

    When a type name is specified in a way that includes a nondeduced context, all of the types that comprise that type name are also nondeduced. However, a compound type can include both deduced and nondeduced types. [Example: If a type is specified as A<T>::B<T2>, both T and T2 are nondeduced. Likewise, if a type is specified as A<I+J>::X<T>, I, J, and T are nondeduced. If a type is specified as void f(typename A<T>::B, A<T>), the T in A<T>::B is nondeduced but the T in A<T> is deduced. ]

In particular, a helper struct template that typedefs the template parameter can be used:

template <typename T>
struct type_identity
{
    typedef T type;
};

This helper struct can then turn std::function<void(T)> into a nondeduced context as shown in the standard:

template <typename T>
void call_with(typename type_identity<std::function<void(T)>>::type f, T val)
{
    f(val);
}

std::type_identity is in the C++20 standard, but not C++17.

The problem can also be solved in a less general way (at each call site) by explicitly specifying the template argument:

call_with<int>(print, 42);


</p>
</details>

#### 54. :skull:
```
```
<details><summary><b>Answer</b></summary>
<p>

#### 
</p>
</details>

#### 55. :skull:
```
```
<details><summary><b>Answer</b></summary>
<p>

#### 
</p>
</details>

#### 56. :skull:
```
```
<details><summary><b>Answer</b></summary>
<p>

#### 
</p>
</details>

#### 57. :skull:
```
```
<details><summary><b>Answer</b></summary>
<p>

#### 
</p>
</details>

#### 58. :skull:
```
```
<details><summary><b>Answer</b></summary>
<p>

#### 
</p>
</details>

#### 59. :skull:
```
```
<details><summary><b>Answer</b></summary>
<p>

#### 
</p>
</details>

#### 60. :skull:
```
```
<details><summary><b>Answer</b></summary>
<p>

#### 
</p>
</details>
