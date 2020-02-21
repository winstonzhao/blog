---
title: Random C++ Knowledge
date: 2020-02-19 14:10:09
tags:
---

# Introduction
I've been picking up C++ tidbits of knowledge through the cpplang slack and the my own studying of the language. I felt like it might be good to colate some what I've learned so that I'll be able to remember it better.

## When to use auto_ptr/raw pointer/class member?
>If you have a unique_ptr as a class member, the object itself lives on the heap and the class only stores the (unique_)pointer to it. If you have the object directly as a class member, the object lives directly inside the class. Both approaches have their use. In general I would put the object directly into the class as a member unless there is a reason for using a unique_ptr.
>Reasons could include: a) object is huge b) class needs to be movable cheaply, c) concrete type of object is not known at compile time (e.g. a pure virtual base class), etc.

After this explanation I didn't get what the last reason was referring to.
>Well, if you have an abstract base class base_class (i.e. it has a pure virtual function), you cannot store it directly, because it cannot be instantiated. You can only instantiate derived classes. But if your class should work with any class derived from base_class, your only choice is to have a `base_class&` member or `base_class*` member. And if your class should also own the object, then it should use unique_ptr to manage that ownership. Therefore, you should use unique_ptr<base_class> in that case.

The confusion stems from Java where you can instantiate a pure virtual (abstract) class type. However, this is not possible in C++ and therefore smart pointers are useful in this use-case.

About raw pointers (different user replied):
* Never use raw pointers for any kind of ownership. That is what smart pointers are for.
* Do use raw pointers for non-owning relationships.
* You can fiddle around with references, optional reference wrappers, observer_ptr, etc. in place of raw pointers for non-owning relationships, but imo this amounts to longhand for non-owning raw pointers and can have the same issues (for example, it is possible to have dangling references). as long as you follow point #1 consistently, your codebase self-documents and says raw pointers are never owning, therefore never need to be deleted, etc.

## Questions from top comment: [Reddit Thread](https://www.reddit.com/r/cpp/comments/24lv4b/what_useful_things_should_every_c_programmer_know/)

####  'const' in all its forms and why it is useful. Depite the extra code you have to write sometimes, const makes your programs much safer as well as easier to reason about.


####  RAII. Don't only use it, design your classes to be able to be used as RAII objects (when appropriate).
Class will own a resource, we don't just mean memory - it could be file handles, network sockets, database handles, GDI objects. The 'Scope-bound' aspect means that the lifetime of the object is bound to the scope of a variable, so when the variable goes out of scope then the destructor will release the resource. A very useful property of this is that it makes for greater exception-safety.

```
RawResourceHandle* handle=createNewResource();
handle->performInvalidOperation();  // Oops, throws exception
...
deleteResource(handle); // oh dear, never gets called so the resource leaks
```

```
ManagedResourceHandle handle(createNewResource());
handle->performInvalidOperation(); // throws, stack is unwound, thus local variables are destroyed
```

The advantages of RAII as a resource management technique are that it provides encapsulation, exception safety (for stack resources), and locality (it allows acquisition and release logic to be written next to each other).

Comparing RAII with the finally construct used in Java, Stroustrup wrote that “In realistic systems, there are far more resource acquisitions than kinds of resources, so the 'resource acquisition is initialization' technique leads to less code than use of a 'finally' construct.”

#### Move Semantics


####  std::make_unique

**Advantages**
* make_unique teaches users "never say new/delete and new[]/delete[]" without disclaimers.

* make_unique shares two advantages with make_shared (excluding the third advantage, increased efficiency). First, unique_ptr<LongTypeName> up(new LongTypeName(args)) must mention LongTypeName twice, while auto up = make_unique<LongTypeName>(args) mentions it once.

* make_unique prevents the unspecified-evaluation-order leak triggered by expressions like foo(unique_ptr<X>(new X), unique_ptr<Y>(new Y)). (Following the advice "never say new" is simpler than "never say new, unless you immediately give it to a named unique_ptr".)

* make_unique is carefully implemented for exception safety and is recommended over directly calling unique_ptr constructors.

**When not use make_unique**
* Don't use make_unique if you need a custom deleter or are adopting a raw pointer from elsewhere.


## Questions from top comment: [Quora Thread](https://www.quora.com/What-are-the-must-know-things-to-be-a-c-developer-How-to-evaluate-a-c-programmer)

* `class c {};` How many member functions does it have?
![](/images/c++_special_members.png)

* Questions about the following function:
```
char *f(char *c) {
    char *p;
    while (*p++ = *c++) ;
    return p;
}
```
1. It has no documentation. What do you think it is supposed to do?
It is supposed to be `strcpy()`
```
char *strcpy(char *dest, const char *src)
{
   char *save = dest;
   while(*dest++ = *src++);
   return save;
}
```

2. Does it?
No it doens't because `char *p` is allocated on the stack and turns into UB after it is returned.

3. How do you change it so it works?
Allocate memory using `malloc()` or `new`, or allow the user to pass in a pointer for which to copy the string.

4. Once it works, what would you do to improve it?
Allow the copying of multiple bytes at once using multiple blocks, detecting whether the null terminator is there.

## Questions from second comment: [Quora Thread](https://www.quora.com/What-are-the-must-know-things-to-be-a-c-developer-How-to-evaluate-a-c-programmer)

1. Where a variable may be allocated? 
2. const to const_cast and other types of cast
3. dynamic_cast to virtual methods table:
  1. Virtual destructor
  2. Virtual inheritance
  3. Pure virtual method
  4. Abstract class
  5. Smart pointers casts
  6. RTTI
4. smart pointers to RAII
5. RAII to exceptions handling
6. incapsulation
  1. protected and private
  2. can destructor or constructor be private
7. from private constructor to patterns
  1. singleton
  2. “template method” pattern
  3. factory method
8. from patterns templates
  1. at singleton template static members, static keyword btw
  2. variadic templates
9. other new standards features
  1. auto accepting const&
  2. &&
  3. copy elision
  4. lambdas
10. from lambda to multithreading
  1. how it works on x86
  2. asm prefix lock and other atomic commands
  3. std::atomic vs interlocked
  4. what are OS synchronization objects
    * mutex
    * semaphore
    * [critical sections] optional
    * spin-locks
    * single writer multiple reader guard
  5. std::async corner cases
  6. thread pools
    * Windows OS global thread pool
    * if Windows kernel is needed than DPC and kernel questions branch, GDT, LDT, “ring 0”, etc
  7. multi-processor environment corner cases and file locks
11. STL
  1. what used
  2. what other know
12. from STL to algorithms and structures
  1. asymptotic
  2. hash table total asymptotic
  3. rb+ vs trie
  4. practice task for senior level
    * for example implement observer pattern with a task where a class method accepts callback function and calls it asynchronously
13. optional [boost and custom project libs/frameworks]
  1. what used
  2. what other know
