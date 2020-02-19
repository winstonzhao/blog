---
title: Random C++ Knowledge
date: 2020-02-19 14:10:09
tags:
---

## Introduction
I've been picking up C++ tidbits of knowledge through the cpplang slack and the my own studying of the language. I felt like it might be good to colate some what I've learned so that I'll be able to remember it better.

### When to use auto_ptr/raw pointer/class member?
>If you have a unique_ptr as a class member, the object itself lives on the heap and the class only stores the (unique_)pointer to it. If you have the object directly as a class member, the object lives directly inside the class. Both approaches have their use. In general I would put the object directly into the class as a member unless there is a reason for using a unique_ptr.
>Reasons could include: a) object is huge b) class needs to be movable cheaply, c) concrete type of object is not known at compile time (e.g. a pure virtual base class), etc.

After this explanation I didn't get what the last reason was referring to.
>Well, if you have an abstract base class base_class (i.e. it has a pure virtual function), you cannot store it directly, because it cannot be instantiated. You can only instantiate derived classes. But if your class should work with any class derived from base_class, your only choice is to have a `base_class&` member or `base_class*` member. And if your class should also own the object, then it should use unique_ptr to manage that ownership. Therefore, you should use unique_ptr<base_class> in that case.

The confusion stems from Java where you can instantiate a pure virtual (abstract) class type. However, this is not possible in C++ and therefore smart pointers are useful in this use-case.

About raw pointers (different user replied):
* Never use raw pointers for any kind of ownership. That is what smart pointers are for.
* Do use raw pointers for non-owning relationships.
* You can fiddle around with references, optional reference wrappers, observer_ptr, etc. in place of raw pointers for non-owning relationships, but imo this amounts to longhand for non-owning raw pointers and can have the same issues (for example, it is possible to have dangling references). as long as you follow point #1 consistently, your codebase self-documents and says raw pointers are never owning, therefore never need to be deleted, etc.