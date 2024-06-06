---
title: 'Python descriptor details'
date: 2024-06-06 17:00:06
tags: Python
categories: Python
hide: true
---

### Python描述符类descriptor详解

> ##### 结论：
> 
> - Python中对象的属性以key-value的形式保存在`__dict__`字典中，原始的访问属性的方式为字典lookup；
> - Python为了实现不同的属性访问方式，提供了**描述符协议（descriptor protocal）**，类似C++中的操作符重载，将`.`操作符从`__dict__[key]`的属性访问方式重载为描述符协议中的`__get__()`函数调用方式，从而实现自动绑定类实例到类成员函数的第一个参数等功能。
> - 描述符协议的使用方法类似`装饰器`，即在原函数执行前后做一系列判断，然后在返回或调用原函数。



##### 1. 类成员函数如何实现self自动绑定第一个参数？

```
class Fruit():
	def color(self):
		pass

fruit = Fruit()
fruit.color()  # 不需要显示传入参数，自动将fruit实例对象作为第一个参数传入
```

在自定义类class中定义成员函数时，会很自然地将self作为第一个形参；但是通过实例对象调用类内的成员原函数时，并不会显示传入实例对象，那为什么成员函数内还能访问实例对象内的属性呢？

首先，通过`fruit.color()`中的`.`操作符访问对象属性时，会自动调用这个对象的`__getattribute__(obj, name)`方法，即:

```
def object_getattribute(obj, name):
# python中对象调用__getattribute__的简单逻辑实现
# obj: 要访问的对象自身
# name: 要访问的属性名称
	obj_type = type(obj)  # Fruit类
	cls_var = getattr(obj_type, name, None)  # Fruit类中名为name的属性
	descriptor_get = getattr(type(cls_var), '__get__', None) 
	# 注意此时找的是type(cls_var)中的__get__方法，type(cls_var)不是Fruit类，而是cls_var属性对应的descriptor类，如下文定义的MethodType类等

	# 有__get__且有__set__或__delete__时，是data descriptor, 调用descriptor中的__get__方法
	if descriptor_get is not None:
		if (hasattr(type(cls_var), '__set__') or hasattr(type(cls_var), '__delete__')):
			return descriptor_get(cls_var, obj, obj_type)  
	# 没有__get__时，先看对象本身是否有该属性
	if hasattr(obj, '__dict__') and name in obj.__dict__:
		return obj.__dict__[name]
	# 只有__get__且对象本身没有该属性时，是non-data descriptor，也调用descriptor中的__get__方法
	if descriptor_get is not None:
		return descriptor_get(cls_var, obj, obj_type) 
	# 实例对象所属的类中有该属性
	if cls_var is not None:
		return cls_var
	# 都没有，则返回找不到属性错误
	raise AttributeError(name)
```

简单概括，当实例对象通过`.`操作符访问某属性时，会优先看该实例对象所属类是否有该属性，如果有**且该属性所属的类**实现了__get__方法和__set__或__delete__方法，则调用该__get__方法；如果没有该属性则看实例对象本身是否有这个属性，如果有则直接返回该属性（字典lookup方式）；如果实例本身没有这个对象，但是实例所属类有该属性且该属性所属类有__get__方法，还是调用__get__方法获取属性；如果实例所属类中有该属性但是该属性所属类没有__get__方法，则直接返回该属性，最后都没有则返回属性找不到error。

以上面的Fruit类中的成员函数color举个例子，`fruit.color`经过__getattribute__函数处理，实际效果会变为`type(fruit).__dict__['color'].__get__(fruit, type(fruit))`：

1. `type(fruit).__dict__['color']`得到的是`Fruit.color`对象，这是一个`function`**描述符类实例**，这个function描述符类待会儿我们在下文会定义。
2. function描述符类的实例对象`Fruit.color`中定义了`__get__`函数，但是没有`__set__`或`__delete__`函数；
3. `fruit.__dict__`并没有`color`属性（实例对象的__dict__中保存的是自己内部的变量，即self里定义的变量）；
4. 自身没有该属性且有`__get__`方法，则调用该`__get__(Fruit.color, fruit, Fruit)`方法；

需要注意的是真正的python解释器在执行整个过程时，并不是直接调用__getattribute__方法，而是当找不到属性时有额外操作进行辅助查找，这里只是为了说明descriptor的作用。

因为descriptor类的存在，导致`__getattribute__`访问实例时会根据不同情况使用不同调用方式：直接__dict__进行lookup或者调用相应`__get__`函数，那么将实例对象self作为成员函数第一个参数又是如何实现的呢？答案就是上文中的**描述符类function**。

```
# 首先定义一个成员函数类MethodType:
# 注意这不是descriptor，只是为了实现自动绑定实例对象作为参数而定义的类
class MethodType():
	def __init__(self, func, obj):
		self.__func__ = func  # 缓存函数
		self.__obj__ = obj  # 缓存实例对象
	
	def __call__(self, *args, **kwargs):
		func = self.__func__
		obj = self.__obj__
		return func(obj, *args, **kwargs)  # 将实例对象作为第一个参数，调用func
```

注意这个类并不是描述符类，其没有实现`__get__`等方法，只是为了实现绑定对象到函数第一个参数的功能，其更像是一个**装饰器**，在原color函数执行前后做一些操作。然后，还需要定义一个描述符类function来实现不同情况的属性获取方法：

```
class function():
	def __get__(self, obj, obj_type=None):
		if obj is None:
			return self
		return MethodType(self, obj)
```

定义了`function`描述符类之后，可以直接使用`def`关键字定义函数，注意`def`关键字函数等价于实例化`function`类并赋值给类变量。例如：`def color() <=> color = function()`，即创建一个描述符实例并赋值给color属性。当通过类名或类实例对象访问color属性时则会产生不同效果：

```
# 通过类名访问
Fruit.color  # 返回<function Fruit.color>
fruit.color  # 返回<bound method Fruit.color of <__main__.Fruit object>>
```

可以看到，通过实例访问color属性（成员函数，描述符类实例）时，产生如下过程：

1. `object_getattribute()`决定调用`function`描述符类中的`__get__()`函数来访问`color`属性；
2. `obj`不为None，返回`MethodType`实例，即`bound method`类型；
3. `MethodType`中将实例对象绑定到原函数color()的第一个参数上；

至此，类实例对象调用成员函数，自动绑定实例对象本身(self)成为函数第一个参数的功能已经实现。但是在Python中，也能通过类自身（即用类名的方式）去访问属性，会有不一样的效果。类似地，有`object_getattribute`方法，也有`type_getattribute`方法，即类名通过`.`操作符访问属性时，会通过此方法决定如何访问。

```
def type_getattribute(cls, name):
	v = object.__getattribute__(cls, name)
	if hasattr(v, '__get__'):
		return v.__get__(None, cls)
	return v
```

可以看到，当通过类直接访问属性时，也是会调到`object.__getattribute__`函数，但是此时传入的不是实例对象，而是类对象本身，所以`descriptor_get`为None，直接看类内是否有`name`这个属性，如果有的话则返回并检查该属性是否有`__get__`方法，有则执行描述符类的`__get__`方法。例如`Fruit.color`，执行了`function`类内的`__get__`方法，但是此时传入的是obj是None，所以直接返回func自身，即`function`而不是`bound function`。

###### 总结：

1. `.`操作符访问属性时会被转化为调用`__getattribute__`方法，来决定是执行原始的`__dict__`内的lookup方法，还是执行描述符descriptor的`__get__`方法。
2. 类对象和实例对象有不同的`__getattribute__`方法，会产生不同的效果：
   
   * 类对象调用：`Fruit.color`  <==>  `Fruit.__dict__['color'].__get__(None, Fruit)`
   * 实例对象调用：`fruit.color`  <==>  `type(fruit).__dict__['color'].__get__(fruit, type(fruit))`
3. Python中自定义类会默认实现上述功能的function descriptor，所以类内的成员函数可以直接通过实例变量调用，自动绑定对象本身作为第一个参数，不需要在显示传参。

##### 2. 什么是描述符类descriptor

> 定义：一个Python对象中如果实现了下列三个方法中的任意一个，则是描述符对象（descriptor）：
> 
> * `__get__(self, obj, type=None)`
> * `__set__(self, obj, value)`
> * `__delete__(self, obj)`
>   其中`self`是当前描述符对象的实例，`obj`是传入的实例，即调用描述符函数的实例（如`fruit`），`type`则是传入的实例所属的类（如`Fruit`）。这三种方法也被称为描述符协议（descriptor protocal）。如果只实现了`__get__`方法，被称为non-data descriptor，如果还实现了`__set__`或者`__delete__`中的一种，则是data descriptor.

描述符主要用于**控制对属性的访问方式**：是使用原始的`__dict__`内lookup的方式，还是使用描述符协议内的`__get__`方式，可以实现计算属性，懒加载属性，属性访问控制等功能。

##### 3. 描述符的应用

###### （1）classmethod

了解了描述符协议类的作用，实现`classmethod`的效果非常简单：

```
class classmethod():
	def __init__(self, f):
		self.f = f
	
	def __get__(self, obj, obj_type=None):
		if obj_type is None:
			obj_type = type(obj)
		def newfunc(*args):
			return self.f(obj_type, *args)
		return newfunc
```

类似地，`@classmethod`装饰器返回一个`classmethod`描述符实例，该描述符的作用就是当类实例变量访问原函数时，首先通过`object_getattribute`调用到上述的描述类`classmethod`，然后其`__get__`将实例`fruit`所属类对象`Fruit`作为第一个参数传入；当直接用类对象访问原函数时，首先通过`type_getattribute`调用到上述`classmethod`，然后其`__get__`直接将`Fruit`作为第一个参数传入。

###### （2）staticmethod

静态方法的实现更加简单，通过`staticmethod`描述符类，不管是类对象还是类实例对象访问属性，都直接返回原属性即可：

```
class staticmethod():
	def __init__(self, f):
		self.f = f
	
	def  __get__(self, obj, obj_type=None):
		return self.f
```

可以看到，`staticmethod`描述符类中的`__get__`函数，不管传入的实例`obj`如何，都直接返回原函数，不会将传入的实例`obj`或者`obj`所属的类绑定到第一个参数。

###### （3）property

我们可以自定义一个property描述符类，从而实现当使用`.`操作符访问属性时产生不同的效果。Python官方提供的property描述符类可以简单实现为：

```
class Property:
	def __init__(self, fget=None, fset=None, fdel=None, doc=None):
		self.fget = fget
		self.fset = fset
		self.fdel = fdel
		if doc is None and fget is not None:
			doc = fget.__doc__
		self.__doc__ =  doc
	
	# 描述符协议方法
	def __get__(self, obj, obj_type=None):
		if obj is None: 
			return self
		if self.fget is None:
			raise AttributeError("unreadable attribute")
		return self.fget(obj)

	# 描述符协议方法
	def __set__(self, obj, value):
		if self.fset is None:
			raise AttributeError("cant set attribute")
		self.fset(obj, value)

	# 描述符协议方法
	def __delete__(self, obj):
		if self.fdel is None:
			raise AttributeError("cant delete attribute")
		self.fdel(obj)

	# 实例化一个拥有fget属性的描述符对象
	def getter(self, fget):
		return type(self)(fget, self.fset, self.fdel, self.__doc__)

	# 实例化一个拥有fset属性的描述符对象
	def setter(self, fset):
		return type(self)(self.fget, fset, self.fdel, self.__doc__)

	# 实例化一个拥有fdel属性的描述符对象
	def deleter(self, fdel):
		return type(self)(self.fget, self.fset, fdel, self.__doc__)
```

`Property`描述符类在其实例内（即实例对象的`__dict__`字典内）保存了`fget`，`fset`和`fdel`函数，然后在描述符协议方法被调用时判断是否调用相应函数，从而实现对属性读写的控制。

`Property`中的`getter()`、`setter()`、`deleter()`是为了以装饰器语法糖的形式而实现的函数，从描述符类的角度看，其实只需要实现`__get__`等方法即可。所以，`Property`也有两种使用方式：

```
class Fruit():
	def __init__(self, color):
		self._color = color

	# 装饰器语法糖，此时相当于创建了一个property描述符类实例，并将`color`方法作为`get`方法传入
	# 相当于调用了getter()方法，后续通过`.`操作符即可访问该属性
	@property
	def color(self):
		return self._color
	
	# 同样相当于实例化了一个描述符对象，并将`color`方法作为`set`方法传入
	# 后续通过`.`操作符赋值即可调用到此`set`方法
	@color.setter
	def color(self, value):
		self._color = value
```

如果不使用语法糖，也可以直接使用`Property`描述符协议类：

```
class Fruit():
	...
	def get_color(self): return self._color
	def set_color(self, color): self._color = color
	def del_color(self): del self._color
	color = Property(get_color, set_color, del_color, 'property test')
```

两种方式达到的效果，都是当使用`.`操作符访问属性时，调用描述符协议类中的`__get__`等方法，而不是原始的字典lookup方法。同时，描述符协议类中的`__get__`方法等可自定义，相当于提供了一种自定义通过`.`操作符去访问类属性的能力。例如，通过自定义`__set__`方法，可以实现对属性赋值时的参数合理性检查。

##### 参考：

[1](https://docs.python.org/2/howto/descriptor.html#definition-and-introduction)
[2](https://waynerv.com/posts/python-descriptor-in-detail/)

