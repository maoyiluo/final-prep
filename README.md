# Final Exam Preparation

The problems below are similar to what you can expect for the final
exam. Though note that:

* There are more exercises than what I expect you to be able to
  complete in 2 hours.
  
* Some of the exercises are a bit harder than what you'd see in the exam.

## Lambda Calculus

1. For each of the following lambda calculus terms, compute the set of
   free variables and show an alpha-renaming of the term such that no
   variable is bound more than once.
   
   1. *(λ x. x y) z*
      
   1. *(λ x. (λ x. x) x) (λ x. x)*
      
   1. *(λ x. (λ x. x) x) (λ x. x) y*   

1. Show the reductions to normal form of the following lambda calculus
   terms, assuming the definitions provided in the class notes:

   1. *`or` `false` `1`*

   1. *`iszero` `1` `2` `3`*

   1. *`pred` `1`*
      
   Show each beta-reduction step.

1. A list can be represented in the lambda calculus by its *fold*
   function. For example, a list *xs* containing *`1`*, *`2`*, and
   *`3`* becomes a function that takes two arguments *f* and *z* and
   returns *f `1` (f `2` (f `3` z))*:

   *xs = λ f z. f `1` (f `2` (f `3` z))*

   Now, if one wants to implement, e.g., a function that sums up all
   the elements in a list *xs* containing Church numerals, this can be
   done as follows:

   *`sum` = λ xs. xs `plus` `0`*

   In particular, applying `sum` to the list *xs* defined above yields:

   *`sum` xs → ... → `plus` `1` (`plus` `2` (`plus` `3` `0`)) → ... → `6`*

   1. Write a lambda term `nil` that represents the empty list `()`.

   2. Write a lambda term `cns` that takes an element *hd* and a list
      *tl* (represented as a fold function) and returns a list formed
      by prepending *hd* on *tl*.
   
   3. Write a lambda term `isnil` that takes a list *xs* and returns
      `true` if *xs* is the empty list and `false` otherwise.
   
   4. Write a lambda term `head` that takes a non-empty list *xs* and
      returns the head of the list (i.e. the left-most element).

   5. Write a lambda term `tail` that takes a non-empty list *xs* and
      removes the remainder of *xs* obtained after removing its
      head. This is quite a bit harder and requires a trick analogous
      to the one used to define `pred` on Church numerals (see the
      class notes).


## Type Inference

For each of the following OCaml functions, decide whether they are
well-typed. If yes, provide the type of the function. Otherwise,
explain the type error. You do not need to provide any typing
constraints or calculate the unifier explicitly.


1. 
   ```ocaml
   let f x y z = if x then not y else z
   ```

2. 
   ```ocaml
   let g x = if x then x + 1 else x - 1
   ```
 
3.
   ```ocaml
   let rec h x = if x then h x + 1 else h x - 1
   ```
 
4. 
   ```ocaml 
   let i f = f true + f 0
   ```
 
5. 
   ```ocaml
   let j x =
     let f y = x in
     f true + f 0
   ```
 
6. 
   ```ocaml
   let rec k = function
   | [] -> 0
   | x :: xs -> x + k xs
   ```
 

7. 
   ```ocaml
   let l = function
   | None -> false
   | Some x -> x + 1
   ```
 
8.
   ```ocaml
   let rec m = function
   | None -> 0
   | Some x -> x
   | [] -> 0
   | x :: xs -> x + m xs
   ```
   
## Type Inhabitation

Assume we are given the following algebraic data type:

```ocaml
type ('a,'b) either = 
  | Left of 'a 
  | Right of 'b
```

Write OCaml functions that satisfy the following polymorphic type
signatures:

* `f: ('a, 'a) either -> 'a`

* `g: ('a, 'b) either * ('a, 'c) either -> ('a, 'b * 'c) either`

* `h: 'a -> ('a -> 'b) -> ('a -> 'c) -> 'b * 'c`

* `i: 'a -> (('b -> 'a) -> 'c) -> 'c`

## OCaml Modules and Functors

1. Write a module signature `SetType` that declares a type `elem` and
   a type `t` where the values of `t` represent sets over values of
   type `elem`. Your signature should support the following
   operations:
   
   1. A value `empty` representing the empty set
   
   2. A function `add` for inserting an element into a set
   
   3. A function `remove` for removing an element from a set
   
   4. A function `mem` for checking whether a given element is a
      member of a given set.
      
   5. A function `fold` that implements a fold function on sets. For
      example, if the type `elem` is `int`, then 
      
      ```ocaml
      fold (fun acc x -> acc + x) 0 s
      ```
      
      should compute the sum of all the elements stored in the set `s`.

2. Consider the following module signature for ordered types:

   ```ocaml
   module type OrderedType =
     sig
       type t
       val compare : t -> t -> int
     end
   ```
   
   as well as the following algebraic data type for binary search trees:

   ```ocaml
   type tree = 
   | Leaf
   | Node of elem * tree * tree
   ```

   Write a functor that takes a module of type `OrderedType` and then
   uses the above binary search tree ADT to implement a module of type
   `SetType`:

   ```ocaml
   module MakeSet (O: OrderedType) : SetType with type elem = O.t = 
     struct
       type elem = O.t

       type tree = 
       | Leaf
       | Node of elem * tree * tree
       
       type t = tree
       ...
   end
   ```

   Implement `fold` using an in-order traversal of the tree.

## Dynamic Dispatch

Consider the following Scala program:

```scala
class A {
  private def m1(s: String): A = {
    println("A.m1(String)")
    new A
  }
  def m2(): Unit = println("A.m2()")
  def m3(): A = {
    println("A.m3()")
    new A
  }
}

object A { def m(a: A): A = a.m1("Hello") }

class B extends A {
  private def m1(s: String): B = {
    println("B.m1(String)")
    this
  }
  override def m2(): Unit = println("B.m2()")
  override def m3(): A = {
    println("B.m3()")
    this
  }
}

object Overriding extends App {
  val a1: A = new B

  val a2: A = A.m(a1)

  a2.m2()

  val a3: A = a1.m3()

  a3.m2()
}
```

1. What is the dynamic type of the variables `a1`, `a2`, and `a3` in
   `Overriding`?

1. What does each of the four method calls in `Overriding` print when
   it is executed? Explain.

## Data Layouts and Vtables

Consider the following Scala classes.

```scala
class Base(private val name: String, val c: Char) {
  
  override def toString: String = name
  
  def m1(o: Any): Any = { 
    println("Base.m1(Any)")
    o
  }
  
  def m2(o: Base): Unit = println("Base.m2(Base)")  
  
  private def m3(s: String): Unit = println("Base.m3(String)")
} 

class Derived(val x: Int, val name: String) extends Base(name, 'c') { 
  private val y: Int = 0
 
  override def m1(o: Any): Derived = { 
    println("Derived.m1(Any)") 
    this
  }
  
  def m4(o: Derived): Unit = println("Derived.m4(Derived)")
}
```

1. What are the correctly ordered entries in `Derived`'s data layout and vtable? Please use
   ```scala
   <classname>.<fieldname> : <type>
   ```
   notation for data layout entries, and
   
   ```scala
   <classname>.<methodname>(<type>, ..., <type>): <type>
   ```
   notation for vtable entries. Here, `<classname>` refers to the name
   of the class where the field, respectively, method has been
   declared. For simplicity, you may assume that the only method that
   `Base` inherits from `AnyRef` is `toString`.

## Scala Generics

Consider the following two generic Scala classes:

```scala
class CoVar[+T](x: T) {
  def method1: T = x
  def method2(y: T): List[T] = List(x,y)
}

class ContraVar[-T](x: T) {
  def method1: T = x
  def method2(y: T): List[T] = List(x,y)
}
```

The Scala compiler will reject both classes because at least one
method in each class violates the variance annotation of the type
parameter `T` of that class because the type occurs in a position with
opposite variance.

1. For each class, indicate the occurrences of the type parameter in
   the code that violates the variance annotation.
   
1. Change each class such that the violations of variances are
   fixed. You are _not allowed_ to change any of the following:

   * the variance annotation of the type parameter T
   * the type of the class parameter `x`;
   * the parameter names and implementations of the methods.

   However, you are allowed to introduce additional generic type
   parameters and type bounds to individual methods or the entire
   class. You are also allowed to change the parameter types and
   return types of methods. If you change any of the methods' types,
   the types should remain as precise as possible (e.g., you are not
   allowed to use the types `Any` or `Nothing`). 
   
   Your resulting code needs to be well-typed.
   
Hints:
  
* Recall that the type `List[A]` is covariant in `A`.

* You might find it more difficult to fix the class `ContraVar`. Note
  that you can introduce an additional type parameter for the whole
  class to replace the violating occurrences of `T` and then use a
  type bound to bound this new parameter appropriately using `T`.

## Memory Management

Consider the following C++ program:
```C++
 1: #include "ptr.h"
 2: 
 3: struct A;
 4:
 5: struct B {
 6:   Ptr<A> a;
 7:   B(Ptr<A>& _a) : a(_a) { }
 8:   B(A* _a) : a(_a) { }
 9: };
10:
11: struct A {
12:   Ptr<B> b;
13:   A(Ptr<B>& _b) : b(_b) { }
14: };
15:
16: int main(void) {
17:   Ptr<B> pn = 0;
18:
19:   Ptr<B> pb = new B(0);
20:
21:   Ptr<A> pa = new A(pb);
22:
23:   pb->a = pa;
24:
25:   pa->b = pn;
26:
27:   return 0;
28: }
```

Here, we assume the implementation of `Ptr` from [Class 13](https://github.com/nyu-pl-sp19/class13/blob/master/src/ptr.h).

1. When we trace the execution of this program, we observe that the following
   methods (respectively constructors and destructors) are called in
   the given order:
  
   ```c++
   Ptr<B>(B*)
   B(A*)
   Ptr<A>(A*)
   Ptr<B>(B*)
   A(Ptr<B>&)
   Ptr<B>(const Ptr<B>&)
   Ptr<A>(A*)
   Ptr<B>::operator->()
   Ptr<A>::operator=(const Ptr<A>&)
   Ptr<A>::operator->()
   Ptr<B>::operator=(const Ptr<B>&)
   ~Ptr<A>
   ~Ptr<B>
   ~B
   ~Ptr<A>
   ~A
   ~Ptr<B>
   ~Ptr<B>
   ```
  
   For each call, explain where the call is coming from. Here is an
   example for the first entry:
  
   `Ptr<B>(B*)` Call to `Ptr` constructor when `pn` is initialized on
   line 17.
  
   Note that some of the method calls happen on the same line in the program.

1. Draw diagrams that illustrate the memory state of the program
   (including the stack and heap contents) right before each of the
   following lines in the program is executed:
   
   * line 25
   
   * line 27

