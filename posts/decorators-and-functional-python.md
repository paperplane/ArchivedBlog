---
title: 
date: '2013-06-28'
categories: Python
tags: ['Python','decorator']
---

##Python装饰器和函数式编程[译文]##

[原文参考](http://www.brianholdefehr.com/decorators-and-functional-python)

装饰器是Python重要特性之一。它不仅是语言的固有有用特性，也教会了我们以一种有趣的方式--函数式方式来思考。


我将尝试从底层原理解释装饰器的工作原理。为了理解装饰器我们将从几个常见主题开始。在此之后，我们将会深入探讨几个简单的装饰器以及它们如何工作。最后，将讨论一些使用装饰器的更高级方式，例如传递可选参数或者串联化使用。

首先，我们来以我认为最简单的方式来定义一个Python函数。从这个简单的定义开始，我们能够以类似简单的方式来定义装饰器。
    
    函数是一个执行特定功能的可复用的代码块。

那么，什么是装饰器呢？
    
    装饰器是一个改变其他函数的函数。
    
现在，让我们从装饰器的定义开始进行扩展，以一组先决条件解释开始。

***

###函数是第一类对象###

在Python语言中，一切皆对象。这就意味着函数能够通过名字进行引用并且像其他任何对象一样被传递。例如：

    def traveling_function():    
        print "Here I am!"

    function_dict = {    
        "func": traveling_function}
    trav_func = function_dict['func']
    trav_func()
    # >> Here I am!

这里函数traveling\_function是字典function\_dict的func键的值，而这个字典值依然可以跟普通函数一样调用。

***

###第一类函数允许是高阶函数###

我们可以像其他任何对象一样传递函数。我们可以将函数作为值传递给字典，将函数加入列表，或者让它们作为对象属性。那么我们能够把函数作为参数传递给其他函数吗？当然可以！一个可以接受其他函数作为参数或者返回另一个函数的函数称之为高阶函数。

    def self_absorbed_function():    
        return "I'm an amazing function!"
        
    def printer(func):    
        print "The function passed to me says: " + func()
        
    # Call `printer` and give it `self_absorbed_function` as an argument
    printer(self_absorbed_function)
    # >> The function passed to me says: I'm an amazing function!

数能够在函数中被调用。这就允许我们创建出许多有意思的函数，像装饰器。

###装饰器基础###

本质上，装饰器就是能够将其他函数作为参数的函数。在大多数情况下，装饰器返回一个经过包装函数修改的版本的函数。为了帮助我们理解所有这些是如何工作，让我们来看看最简单的装饰器——the identity decorator。

    def identity_decorator(func):    
        def wrapper():        
            func()    
        return wrapper
    
    def a_function():    
        print "I'm a normal function."
        
    # `decorated_function` is the function that `identity_decorator` returns, which
    # is the nested function, `wrapper`
    decorated_function = identity_decorator(a_function)
    # This calls the function that `identity_decorator` returned
    
    decorated_function()
    # >> I'm a normal function

这里，identity\_decorator对它所包装函数未作任何改变。当调用时它简单地返回函数（wrapper），并调用原始函数identity\_decorator接受被包装函数作为参数。这是个无所作为的装饰器！而关于identity\_decorator真正有趣的是wrapper能够访问func变量即使func并没有作为任何一个参数传递给它。这是由于闭包。

***

###闭包###

闭包是个花哨的术语。闭包意味着当函数一旦声明，就会维持一个指向该函数所被声明词法环境的引用。

当wrapper在之前的例子中被定义时，它已经有在它的本地作用域访问对func变量的访问权限。这意味着在wrapper（它被返回并赋值给decorated\_function）的整个生命周期中，它都能够访问func变量。一旦identity\_decorator返回，访问func的唯一途径就是通过decorated\_function。func除了在decorated\_function的闭包环境中已不再一个变量。

***

###一个简单的装饰器###

现在我们来创建一个真正有所“作为”的装饰器。这个装饰器所做的就是记录它改变的函数的调用次数。

    def logging_decorator(func):
        def wrapper():
            wrapper.count+=1
            print "The function I modify has been called {0} times(s).".format(wrapper.count)
            func()
        wrapper.count=0
        returnwrapper
        
    def a_function():
        print "I'm a normal function."
        
    modified_function=logging_decorator(a_function)

    modified_function()
    # >> The function I modify has been called 1 time(s).
    # >> I'm a normal 
    function.modified_function()
    # >> The function I modify has been called 2 time(s).
    # >> I'm a normal function.

已经说过一个装饰器改变一个函数，并且能这样思考会非常有用。但是正如你在我们的例子中所看到，logging\_decrorator所做的就是返回一个类似于a\_function的新函数，它附带纪录日志的特性。

在这个例子中，logging\_decorator不仅仅接受函数作为参数，它同时返回函数wrapper。每次logging\_decorator返回的函数被调用时，它都会增加wrapper.count的值，并打印出来，然后调用logging\_decorator包装的函数。

你可能会疑惑为什么我们的计数器是wrapper的属性而不是普通的变量。会不会是wrapper的闭包给了我们对在它的作用域内所有本地变量的访问权限？是的，但是这里有一个问题。在Python语言里，闭包对在它的函数作用域链内所有变量提供所有可读权限，但是仅仅只对可变对象（列表，字典等）提供可写权限。在Python里一个interger数字是不可变对象，因此我们不能在wrapper里递增它的值。取而代之的是我们让计数器成为wrapper的属性，一个可变对象，因此我们能够增加它的值。

***

###装饰器语法###

在上一个例子中，我们看到装饰器能够以传递函数作为参数来使用，因此用装饰器函数'wrapping'了作为参数的函数。然而，一旦你习惯了装饰器，Python同时也有一个语法模式能够让这些变得更直观和简单来阅读。

    # In the previous example, we used our decorator function by passing the
    # function we wanted to modify to it, and assigning the result to a variable
    def some_function():    
        print "I'm happiest when decorated."
        
    # Here we will make the assigned variable the same name as the wrapped function
    some_function = logging_decorator(some_function)

    # We can achieve the exact same thing with this syntax:
    @logging_decorator
    def some_function():    
        print "I'm happiest when decorated."


使用装饰器语法，下面鸟瞰发生了什么：

1.解释器找到被装饰的函数，编译some\_function，并将其命名为'some\_function'。

2.然后这个函数被传递给用来声明装饰器语法（'@'符号之后）为名称的装饰器函数（logging\_decorator）

3.装饰器函数（通常另一个函数包装原始函数）的返回值取代原始函数（some\_function）的返回值。它现在绑定名称为'some\_function'的函数。

    def identity_decorator(func):
        # Everything here happens when the decorator LOADS and is passed
        # the function as described in step 2 above
        def wrapper():
            # Things here happen each time the final wrapped function gets CALLED
            func()
        returnwrapper


有了这些步骤在脑海中，让我们来注释identity\_decorator函数以澄清相关概念

希望这些注释是有启发性的。每次只有在装饰器返回函数的里面的函数调用命令被调用时，被包装的函数被激活。那些在返回函数之外的命令将只会发生一次--在给装饰器函数第一次传递被包装函数时，也正如上面步骤2所示。


个人理解：原始函数既是装饰器函数的参数，也是被包装函数

包装函数不是装饰器函数，它包装原始函数，并且是装饰器函数的返回函数。

装饰器函数接受原始函数作为参数，返回包装函数作为结果。

***

###\*args和\**kwargs###

你之前可能曾经看到过这对令人迷惑的兄弟，下面我们来同时讨论它们。

python函数通过在参数列表中使用*args语法接受可变数量的位置参数。*args会组合所有的非关键字参数为一个能够在函数内部访问的元组参数。反过来，当*args被用作函数调用的参数列表时，它会由元组参数扩展成一系列位置参数。

    def function_with_many_arguments(*args):    
        print args
        # `args` within the function will be a tuple of any arguments we pass
        # which can be used within the function like any other tuple
        
    function_with_many_arguments('hello', 123, True)
    # >> ('hello', 123, True)

    def function_with_3_parameters(num, boolean, string):    
        print "num is " + str(num)    
        print "boolean is " + str(boolean)    
        print "string is " + string
    
    arg_list = [1, False, 'decorators']
    
    # arg_list will be expanded into 3 positional arguments by the `*` symbol
    function_with_3_parameters(*arg_list)
    # >> num is 1
    # >> boolean is False
    # >> string is decorators


要重申：在parameter list中，\*args会组合一系列参数为一个名为args的元组；在argument list中，\*args会展开这个可迭代参数为一系列它所应用的函数的位置参数。

正如你在参数展开的例子中所看到，符号\*能够用于除args以外的名字。仅仅是因为惯例才使用\*args形式来组合/展开通用参数列表。


\*\*kwargs行为类似它的兄弟\*args，但是它同关键字参数而不是位置参数一起工作。如果\**kwargs用于函数的parameter list，它会收集这个函数接受的额外关键字参数并将它们放在一个字典里面。如果在函数里面的argument list使用，它会展开成一个字典作为一系列关键字参数。


现在你理解了\*args和*\*kwargs工作原理，下面来继续学习装饰器你可能会发现这些很有用。

    def function_with_many_keyword_args(**kwargs):    
        print kwargs
    
    function_with_many_keyword_args(a='apples', b='bananas', c='cantalopes')
    # >> {'a': 'apples', 'b': 'bananas', 'c': 'cantalopes'}

    def multiply_name(count=0, name=''):    
        print name * count

    arg_dict = {'count': 3, 'name': 'Brian'}
    multiply_name(**arg_dict)
    # >> BrianBrianBrian

***

###缓存化###

缓存化是一种避免潜在重复花费计算的方式。可以通过缓存函数每次执行结果来实现。以这种方式，下次函数用同样的参数执行时，它会返回缓存中的结果，而不用额外时间去计算结果。

    from functools import wraps
    def memoize(func):    
        cache = {}    
        @wraps(func)    
        def wrapper(*args):        
            if args not in cache:           
            cache[args] = func(*args)        
            return cache[args]    
        return wrapper
    
    @memoize
    def an_expensive_function(arg1, arg2, arg3):    
        ...


在这个代码实例中你可能会注意到奇怪的@wraps装饰器。在讨论memoize整体之前将简单解释下这个装饰器。
    
使用装饰器的副作用就是被包装的函数失去它自带属性`__name__`,`__doc__`和`__module__`。wraps函数被用做为包装装饰器的返回函数的装饰器，恢复在没有装饰情况下被包装函数会失去的这三个它们本应该有的属性。例如：如果没有使用wraper装饰器an\_expensive\_function的名字（通过an\_expensive\_function.`__name__`查看）将会被'wrapper'而无法查看。

缓存化展现了装饰器的一个很好的用例。它实现了很多函数都想提供的功能，通过创建通用的装饰器，我们能够向任何能够受益于它的函数添加功能。这避免了在很多不同的地方需要重新开发此功能。通过避免重复它使我们的代码更容易维护，也更容易阅读和理解。通过阅读单个单词迅速理解这个功能。

应该注意的是缓存化仅仅适用于那些纯函数。就是那些在给定特定参数列表的前提下确保总是返回相同结果的函数。如果它取决于那些作为参数而未传递的全局变量，I/O或者可能影响返回值的其他任何东西，缓存化将会产生令人困惑的结果。同时，一个纯函数没有任何副作用。因此如果你的函数增加计数器，或者调用其他对象的方法，或者其他任何没有展现在函数产生的返回值，那么副作用就不会出现在由缓存返回的结果中。

***


###类装饰器###

最初，我们说装饰器是改变另一个函数的函数，但是它们也能够用来改变类或者方法。装饰类不是经常使用，但是在特定实例下它是元类的一个非常有用的替代工具。

    foo = ['important', 'foo', 'stuff']
    def add_foo(klass):    
        klass.foo = foo    
        return klass
        
    @add_foo
    class Person(object):    
        pass
    
    brian = Person()
    print brian.foo
    # >> ['important', 'foo', 'stuff']

现在任何Person类的对象都会有超级重要的foo属性！注意到既然我们装饰一个类，而装饰器并不返回一个函数，而是自然地返回一个类。我们来更新装饰器的定义：

    装饰器是一个改变函数，方法或者类的函数。


***

###装饰器类###

事实证明我之前是有所隐瞒的。装饰器不仅仅能够装饰类，装饰器还可以是类！唯一的要求就是装饰器的返回值是可调用的。这意味着它必须实现`__call__`方法，当你调用一个对象的时候这个方法会在后台被调用。一般函数当然隐式地设置此方法。下面重新创建identity_decorator作为类来看装饰器类如何工作。

    class IdentityDecorator(object):
        def__init__(self,func):
            self.func=func
        def__call__(self):
            self.func()
            
    @IdentityDecorator
    def a_function():
        print"I'm a normal function."a_function()
        # >> I'm a normal function

这是示例中所发生的：

    当IdentityDecorator装饰a\_function时，它表现地像一个装饰器函数。这段代码等价于举例中的装饰器语法： a\_function = IdentityDecorator(a\_function)。这个装饰器类在它所装饰的函数作为参数传递给它时被调用（被初始化）
           
    当IdentityDecorator被初始化时。它的初始化函数`__init__`被调用时接受被装饰函数作为参数。在这个例子中所有的行为就是将函数赋值给类的一个属性因此它可以在后面通过其他方法被访问。
    
    最后，当a\_function（这确实是返回的IdentityDecorator对象包装a\_function）被调用时，对象的`__call__`方法被激活。因为这仅仅是一个特性装饰器，它简单地调用它装饰的函数。

下面再一次更新装饰器的定义：
    
    装饰器是改变函数，方法和类的可调用对象。

***


###带参数的装饰器###

很多时候你需要根据实际情况更改你的装饰器行为，这时可以通过传递参数来做。

    from functools import wraps
    def argumentative_decorator(gift):
        def func_wrapper(func):
            @wraps(func)
            def returned_wrapper(*args,**kwargs):
                print"I don't like this "+gift+" you gave me!"
            return func(gift,*args,**kwargs)
        return returned_wrapper
    return func_wrapper
    
    @argumentative_decorator("sweater")
    def grateful_function(gift):
        print"I love the "+gift+"! Thank you!"
    
    grateful_function()
    # >> I don't like this sweater you gave me!
    # >> I love the sweater! Thank you!

让我们来看下如果不使用装饰器语法这个装饰器函数应该如何工作。 

    # If we tried to invoke without an argument:
    grateful_function=argumentative_function(grateful_function)
    
    # But when given an argument, the pattern changes to:
    grateful_function=argumentative_decorator("sweater")(grateful_function)

需要注意的最主要的事情就是当给定参数时，装饰器第一次激活时仅仅只有这些参数－被包装的函数不是这些普通参数其中之一。在函数调用返回时，装饰器装饰的函数通过带有参数（在这个例子中，返回的值是：argumentative\_decorator("sweater")）的初始激活的装饰器被传递给返回的函数。

详细解释：

    1. 解释器到达被装饰的函数，编译grateful_function，并且绑定它为名称grateful_function。
    2. argumentative_decorator被调用，并且传递参数"sweater"。它返回func_wrapper。
    3. func_wrapper被激活，grateful_functio作为它的参数。func_wrapper返回returned_wrapper。
    4. 最后，returned_wrapper取代原始的函数grateful_function，因此它被绑定到名为'grateful_function'。

我认为这段代码比没有装饰器参数时更难理解一些，但是如果你能花费一点时间来仔细思考，非常有希望能够理解。

***

###装饰器作为可选参数###

有很多种方式来接受带可选参数的装饰器。这取决于你是否想用位置参数，关键字参数还是都用，你将会使用到一个稍微不同的模式。下面展示一种接受可选关键字参数的使用方式：

    from functools import wraps
    GLOBAL_NAME = "Brian"
    
    def print_name(function=None, name=GLOBAL_NAME):    
        def actual_decorator(function):        
            @wraps(function)        
            def returned_func(*args, **kwargs):            
                print "My name is " + name          
                return function(*args, **kwargs)        
            return returned_func    
            
        if not function:  
            # User passed in a name argument        
            def waiting_for_func(function):            
                return actual_decorator(function)        
            return waiting_for_func    
        else:        
            return actual_decorator(function)
            
    @print_name
    def a_function():    
        print "I like that name!"
    
    @print_name(name='Matt')
    def another_function():    
        print "Hey, that's new!"
        
    a_function()
    # >> My name is Brian
    # >> I like that name!
    
    another_function()
    # >> My name is Matt
    # >> Hey, that's new!

如果我们给print\_name传递关键字参数name,它会表现得类似前面例子的argumentative\_decorator。也就是说，第一次print\_name将会调用name作为它的参数。第一次激活返回的函数将会被传递给它包装的函数。

如果我们不提供name参数，print\_name表现得将会像之前没有参数的装饰器例子。它仅仅会激活它包装的函数作为唯一参数。

print\_name有两种可能性。它检查是否接受包装的函数作为参数。如果不是，它返回函数waiting\_for\_func，这个函数将会接受被包装的函数作为参数并被激活。如果它接受函数作为参数，它将跳过中间步骤并且立即激活actual\_decorator。

***

###链式装饰器###

下面我们探讨装饰器最后一个特性：链式。可以给定函数使用超过一个装饰器。这以一种类似于多重继承来结构化的方式来结构化函数。这可能是避免抓狂的最后的方式。

当你链式装饰器时，装饰器将会按顺序从底到顶存储在堆中。被包装的函数，some\_function，被编译然后传递给它上面的第一个装饰器（logging\_decorator）。然后第一个装饰器的返回值传递给第二个装饰器。就这样它们将会继续传递给链中的每一个装饰器。

    @print_name('Sam')
    @logging_decorator
    def some_function():    
        print "I'm the wrapped function!"
    
    some_function()
    # >> My name is Sam
    # >> The function I modify has been called 1 time(s).
    # >> I'm the wrapped function!

既然这里使用的装饰器都是print一个值并且执行传递给它们的函数，这意味着链中的最后一个装饰器print\_name，将会打印被包装函数被调用的第一行输出。

***

###结论###

我认为装饰器最大的优势就是它允许你在一个稍微高的抽象层来思考。如果你开始阅读函数定义并且看到它有一个缓存化装饰器，你会很快理解你在看一个有记忆特性的函数。如果记忆代码被包括在函数体内部它会需要额外的思维解析,而且会引入可能的误解。使用装饰器同样允许你代码重用，这会节省时间，容易调试，并且使重构容易。

使用装饰器同样是学习像高阶函数和闭包这样的函数式概念的一种很好的方式。

我希望这是一个愉快而翔实的阅读!
