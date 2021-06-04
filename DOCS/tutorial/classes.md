### Navigation

-   [index](https://docs.python.org/3/genindex.html "General Index")
-   [modules](https://docs.python.org/3/py-modindex.html "Python Module Index") |
-   [next](stdlib.html "10. Brief Tour of the Standard Library") |
-   [previous](errors.html "8. Errors and Exceptions") |
-   ![](../_static/py.png)
-   [Python](https://www.python.org/) »
-   [3.9.5 Documentation](https://docs.python.org/3/index.html) »
-   [The Python Tutorial](index.html) »
-   

    |

<span id="tut-classes"></span>

<span class="section-number">9. </span>Classes<a href="#classes" class="headerlink" title="Permalink to this headline">¶</a>
============================================================================================================================

Classes provide a means of bundling data and functionality together. Creating a new class creates a new *type* of object, allowing new *instances* of that type to be made. Each class instance can have attributes attached to it for maintaining its state. Class instances can also have methods (defined by its class) for modifying its state.

Compared with other programming languages, Python’s class mechanism adds classes with a minimum of new syntax and semantics. It is a mixture of the class mechanisms found in C++ and Modula-3. Python classes provide all the standard features of Object Oriented Programming: the class inheritance mechanism allows multiple base classes, a derived class can override any methods of its base class or classes, and a method can call the method of a base class with the same name. Objects can contain arbitrary amounts and kinds of data. As is true for modules, classes partake of the dynamic nature of Python: they are created at runtime, and can be modified further after creation.

In C++ terminology, normally class members (including the data members) are *public* (except see below <a href="#tut-private" class="reference internal"><span class="std std-ref">Private Variables</span></a>), and all member functions are *virtual*. As in Modula-3, there are no shorthands for referencing the object’s members from its methods: the method function is declared with an explicit first argument representing the object, which is provided implicitly by the call. As in Smalltalk, classes themselves are objects. This provides semantics for importing and renaming. Unlike C++ and Modula-3, built-in types can be used as base classes for extension by the user. Also, like in C++, most built-in operators with special syntax (arithmetic operators, subscripting etc.) can be redefined for class instances.

(Lacking universally accepted terminology to talk about classes, I will make occasional use of Smalltalk and C++ terms. I would use Modula-3 terms, since its object-oriented semantics are closer to those of Python than C++, but I expect that few readers have heard of it.)

<span id="tut-object"></span>

<span class="section-number">9.1. </span>A Word About Names and Objects<a href="#a-word-about-names-and-objects" class="headerlink" title="Permalink to this headline">¶</a>
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Objects have individuality, and multiple names (in multiple scopes) can be bound to the same object. This is known as aliasing in other languages. This is usually not appreciated on a first glance at Python, and can be safely ignored when dealing with immutable basic types (numbers, strings, tuples). However, aliasing has a possibly surprising effect on the semantics of Python code involving mutable objects such as lists, dictionaries, and most other types. This is usually used to the benefit of the program, since aliases behave like pointers in some respects. For example, passing an object is cheap since only a pointer is passed by the implementation; and if a function modifies an object passed as an argument, the caller will see the change — this eliminates the need for two different argument passing mechanisms as in Pascal.

<span id="tut-scopes"></span>

<span class="section-number">9.2. </span>Python Scopes and Namespaces<a href="#python-scopes-and-namespaces" class="headerlink" title="Permalink to this headline">¶</a>
------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Before introducing classes, I first have to tell you something about Python’s scope rules. Class definitions play some neat tricks with namespaces, and you need to know how scopes and namespaces work to fully understand what’s going on. Incidentally, knowledge about this subject is useful for any advanced Python programmer.

Let’s begin with some definitions.

A *namespace* is a mapping from names to objects. Most namespaces are currently implemented as Python dictionaries, but that’s normally not noticeable in any way (except for performance), and it may change in the future. Examples of namespaces are: the set of built-in names (containing functions such as <a href="https://docs.python.org/3/library/functions.html#abs" class="reference internal" title="abs"><code class="sourceCode python"><span class="bu">abs</span>()</code></a>, and built-in exception names); the global names in a module; and the local names in a function invocation. In a sense the set of attributes of an object also form a namespace. The important thing to know about namespaces is that there is absolutely no relation between names in different namespaces; for instance, two different modules may both define a function `maximize` without confusion — users of the modules must prefix it with the module name.

By the way, I use the word *attribute* for any name following a dot — for example, in the expression `z.real`, `real` is an attribute of the object `z`. Strictly speaking, references to names in modules are attribute references: in the expression `modname.funcname`, `modname` is a module object and `funcname` is an attribute of it. In this case there happens to be a straightforward mapping between the module’s attributes and the global names defined in the module: they share the same namespace! <a href="#id2" id="id1" class="footnote-reference brackets">1</a>

Attributes may be read-only or writable. In the latter case, assignment to attributes is possible. Module attributes are writable: you can write `modname.the_answer                     = 42`. Writable attributes may also be deleted with the <a href="https://docs.python.org/3/reference/simple_stmts.html#del" class="reference internal"><code class="xref std std-keyword docutils literal notranslate">del</code></a> statement. For example, `del                     modname.the_answer` will remove the attribute `the_answer` from the object named by `modname`.

Namespaces are created at different moments and have different lifetimes. The namespace containing the built-in names is created when the Python interpreter starts up, and is never deleted. The global namespace for a module is created when the module definition is read in; normally, module namespaces also last until the interpreter quits. The statements executed by the top-level invocation of the interpreter, either read from a script file or interactively, are considered part of a module called <a href="https://docs.python.org/3/library/__main__.html#module-__main__" class="reference internal" title="__main__: The environment where the top-level script is run."><code class="sourceCode python">__main__</code></a>, so they have their own global namespace. (The built-in names actually also live in a module; this is called <a href="https://docs.python.org/3/library/builtins.html#module-builtins" class="reference internal" title="builtins: The module that provides the built-in namespace."><code class="sourceCode python">builtins</code></a>.)

The local namespace for a function is created when the function is called, and deleted when the function returns or raises an exception that is not handled within the function. (Actually, forgetting would be a better way to describe what actually happens.) Of course, recursive invocations each have their own local namespace.

A *scope* is a textual region of a Python program where a namespace is directly accessible. “Directly accessible” here means that an unqualified reference to a name attempts to find the name in the namespace.

Although scopes are determined statically, they are used dynamically. At any time during execution, there are 3 or 4 nested scopes whose namespaces are directly accessible:

-   the innermost scope, which is searched first, contains the local names

-   the scopes of any enclosing functions, which are searched starting with the nearest enclosing scope, contains non-local, but also non-global names

-   the next-to-last scope contains the current module’s global names

-   the outermost scope (searched last) is the namespace containing built-in names

If a name is declared global, then all references and assignments go directly to the middle scope containing the module’s global names. To rebind variables found outside of the innermost scope, the <a href="https://docs.python.org/3/reference/simple_stmts.html#nonlocal" class="reference internal"><code class="xref std std-keyword docutils literal notranslate">nonlocal</code></a> statement can be used; if not declared nonlocal, those variables are read-only (an attempt to write to such a variable will simply create a *new* local variable in the innermost scope, leaving the identically named outer variable unchanged).

Usually, the local scope references the local names of the (textually) current function. Outside functions, the local scope references the same namespace as the global scope: the module’s namespace. Class definitions place yet another namespace in the local scope.

It is important to realize that scopes are determined textually: the global scope of a function defined in a module is that module’s namespace, no matter from where or by what alias the function is called. On the other hand, the actual search for names is done dynamically, at run time — however, the language definition is evolving towards static name resolution, at “compile” time, so don’t rely on dynamic name resolution! (In fact, local variables are already determined statically.)

A special quirk of Python is that – if no <a href="https://docs.python.org/3/reference/simple_stmts.html#global" class="reference internal"><code class="xref std std-keyword docutils literal notranslate">global</code></a> or <a href="https://docs.python.org/3/reference/simple_stmts.html#nonlocal" class="reference internal"><code class="xref std std-keyword docutils literal notranslate">nonlocal</code></a> statement is in effect – assignments to names always go into the innermost scope. Assignments do not copy data — they just bind names to objects. The same is true for deletions: the statement `del                     x` removes the binding of `x` from the namespace referenced by the local scope. In fact, all operations that introduce new names use the local scope: in particular, <a href="https://docs.python.org/3/reference/simple_stmts.html#import" class="reference internal"><code class="xref std std-keyword docutils literal notranslate">import</code></a> statements and function definitions bind the module or function name in the local scope.

The <a href="https://docs.python.org/3/reference/simple_stmts.html#global" class="reference internal"><code class="xref std std-keyword docutils literal notranslate">global</code></a> statement can be used to indicate that particular variables live in the global scope and should be rebound there; the <a href="https://docs.python.org/3/reference/simple_stmts.html#nonlocal" class="reference internal"><code class="xref std std-keyword docutils literal notranslate">nonlocal</code></a> statement indicates that particular variables live in an enclosing scope and should be rebound there.

<span id="tut-scopeexample"></span>

### <span class="section-number">9.2.1. </span>Scopes and Namespaces Example<a href="#scopes-and-namespaces-example" class="headerlink" title="Permalink to this headline">¶</a>

This is an example demonstrating how to reference the different scopes and namespaces, and how <a href="https://docs.python.org/3/reference/simple_stmts.html#global" class="reference internal"><code class="xref std std-keyword docutils literal notranslate">global</code></a> and <a href="https://docs.python.org/3/reference/simple_stmts.html#nonlocal" class="reference internal"><code class="xref std std-keyword docutils literal notranslate">nonlocal</code></a> affect variable binding:

    def scope_test():
        def do_local():
            spam = "local spam"

        def do_nonlocal():
            nonlocal spam
            spam = "nonlocal spam"

        def do_global():
            global spam
            spam = "global spam"

        spam = "test spam"
        do_local()
        print("After local assignment:", spam)
        do_nonlocal()
        print("After nonlocal assignment:", spam)
        do_global()
        print("After global assignment:", spam)

    scope_test()
    print("In global scope:", spam)

The output of the example code is:

    After local assignment: test spam
    After nonlocal assignment: nonlocal spam
    After global assignment: nonlocal spam
    In global scope: global spam

Note how the *local* assignment (which is default) didn’t change *scope\_test*’s binding of *spam*. The <a href="https://docs.python.org/3/reference/simple_stmts.html#nonlocal" class="reference internal"><code class="xref std std-keyword docutils literal notranslate">nonlocal</code></a> assignment changed *scope\_test*’s binding of *spam*, and the <a href="https://docs.python.org/3/reference/simple_stmts.html#global" class="reference internal"><code class="xref std std-keyword docutils literal notranslate">global</code></a> assignment changed the module-level binding.

You can also see that there was no previous binding for *spam* before the <a href="https://docs.python.org/3/reference/simple_stmts.html#global" class="reference internal"><code class="xref std std-keyword docutils literal notranslate">global</code></a> assignment.

<span id="tut-firstclasses"></span>

<span class="section-number">9.3. </span>A First Look at Classes<a href="#a-first-look-at-classes" class="headerlink" title="Permalink to this headline">¶</a>
--------------------------------------------------------------------------------------------------------------------------------------------------------------

Classes introduce a little bit of new syntax, three new object types, and some new semantics.

<span id="tut-classdefinition"></span>

### <span class="section-number">9.3.1. </span>Class Definition Syntax<a href="#class-definition-syntax" class="headerlink" title="Permalink to this headline">¶</a>

The simplest form of class definition looks like this:

    class ClassName:
        <statement-1>
        .
        .
        .
        <statement-N>

Class definitions, like function definitions (<a href="https://docs.python.org/3/reference/compound_stmts.html#def" class="reference internal"><code class="xref std std-keyword docutils literal notranslate">def</code></a> statements) must be executed before they have any effect. (You could conceivably place a class definition in a branch of an <a href="https://docs.python.org/3/reference/compound_stmts.html#if" class="reference internal"><code class="xref std std-keyword docutils literal notranslate">if</code></a> statement, or inside a function.)

In practice, the statements inside a class definition will usually be function definitions, but other statements are allowed, and sometimes useful — we’ll come back to this later. The function definitions inside a class normally have a peculiar form of argument list, dictated by the calling conventions for methods — again, this is explained later.

When a class definition is entered, a new namespace is created, and used as the local scope — thus, all assignments to local variables go into this new namespace. In particular, function definitions bind the name of the new function here.

When a class definition is left normally (via the end), a *class object* is created. This is basically a wrapper around the contents of the namespace created by the class definition; we’ll learn more about class objects in the next section. The original local scope (the one in effect just before the class definition was entered) is reinstated, and the class object is bound here to the class name given in the class definition header (`ClassName` in the example).

<span id="tut-classobjects"></span>

### <span class="section-number">9.3.2. </span>Class Objects<a href="#class-objects" class="headerlink" title="Permalink to this headline">¶</a>

Class objects support two kinds of operations: attribute references and instantiation.

*Attribute references* use the standard syntax used for all attribute references in Python: `obj.name`. Valid attribute names are all the names that were in the class’s namespace when the class object was created. So, if the class definition looked like this:

    class MyClass:
        """A simple example class"""
        i = 12345

        def f(self):
            return 'hello world'

then `MyClass.i` and `MyClass.f` are valid attribute references, returning an integer and a function object, respectively. Class attributes can also be assigned to, so you can change the value of `MyClass.i` by assignment. `__doc__` is also a valid attribute, returning the docstring belonging to the class: `"A                       simple                       example                       class"`.

Class *instantiation* uses function notation. Just pretend that the class object is a parameterless function that returns a new instance of the class. For example (assuming the above class):

    x = MyClass()

creates a new *instance* of the class and assigns this object to the local variable `x`.

The instantiation operation (“calling” a class object) creates an empty object. Many classes like to create objects with instances customized to a specific initial state. Therefore a class may define a special method named <a href="https://docs.python.org/3/reference/datamodel.html#object.__init__" class="reference internal" title="object.__init__"><code class="sourceCode python"><span class="fu">__init__</span>()</code></a>, like this:

    def __init__(self):
        self.data = []

When a class defines an <a href="https://docs.python.org/3/reference/datamodel.html#object.__init__" class="reference internal" title="object.__init__"><code class="sourceCode python"><span class="fu">__init__</span>()</code></a> method, class instantiation automatically invokes <a href="https://docs.python.org/3/reference/datamodel.html#object.__init__" class="reference internal" title="object.__init__"><code class="sourceCode python"><span class="fu">__init__</span>()</code></a> for the newly-created class instance. So in this example, a new, initialized instance can be obtained by:

    x = MyClass()

Of course, the <a href="https://docs.python.org/3/reference/datamodel.html#object.__init__" class="reference internal" title="object.__init__"><code class="sourceCode python"><span class="fu">__init__</span>()</code></a> method may have arguments for greater flexibility. In that case, arguments given to the class instantiation operator are passed on to <a href="https://docs.python.org/3/reference/datamodel.html#object.__init__" class="reference internal" title="object.__init__"><code class="sourceCode python"><span class="fu">__init__</span>()</code></a>. For example,

    >>> class Complex:
    ...     def __init__(self, realpart, imagpart):
    ...         self.r = realpart
    ...         self.i = imagpart
    ...
    >>> x = Complex(3.0, -4.5)
    >>> x.r, x.i
    (3.0, -4.5)

<span id="tut-instanceobjects"></span>

### <span class="section-number">9.3.3. </span>Instance Objects<a href="#instance-objects" class="headerlink" title="Permalink to this headline">¶</a>

Now what can we do with instance objects? The only operations understood by instance objects are attribute references. There are two kinds of valid attribute names: data attributes and methods.

*data attributes* correspond to “instance variables” in Smalltalk, and to “data members” in C++. Data attributes need not be declared; like local variables, they spring into existence when they are first assigned to. For example, if `x` is the instance of `MyClass` created above, the following piece of code will print the value `16`, without leaving a trace:

    x.counter = 1
    while x.counter < 10:
        x.counter = x.counter * 2
    print(x.counter)
    del x.counter

The other kind of instance attribute reference is a *method*. A method is a function that “belongs to” an object. (In Python, the term method is not unique to class instances: other object types can have methods as well. For example, list objects have methods called append, insert, remove, sort, and so on. However, in the following discussion, we’ll use the term method exclusively to mean methods of class instance objects, unless explicitly stated otherwise.)

Valid method names of an instance object depend on its class. By definition, all attributes of a class that are function objects define corresponding methods of its instances. So in our example, `x.f` is a valid method reference, since `MyClass.f` is a function, but `x.i` is not, since `MyClass.i` is not. But `x.f` is not the same thing as `MyClass.f` — it is a *method object*, not a function object.

<span id="tut-methodobjects"></span>

### <span class="section-number">9.3.4. </span>Method Objects<a href="#method-objects" class="headerlink" title="Permalink to this headline">¶</a>

Usually, a method is called right after it is bound:

    x.f()

In the `MyClass` example, this will return the string `'hello                       world'`. However, it is not necessary to call a method right away: `x.f` is a method object, and can be stored away and called at a later time. For example:

    xf = x.f
    while True:
        print(xf())

will continue to print `hello                       world` until the end of time.

What exactly happens when a method is called? You may have noticed that `x.f()` was called without an argument above, even though the function definition for `f()` specified an argument. What happened to the argument? Surely Python raises an exception when a function that requires an argument is called without any — even if the argument isn’t actually used…

Actually, you may have guessed the answer: the special thing about methods is that the instance object is passed as the first argument of the function. In our example, the call `x.f()` is exactly equivalent to `MyClass.f(x)`. In general, calling a method with a list of *n* arguments is equivalent to calling the corresponding function with an argument list that is created by inserting the method’s instance object before the first argument.

If you still don’t understand how methods work, a look at the implementation can perhaps clarify matters. When a non-data attribute of an instance is referenced, the instance’s class is searched. If the name denotes a valid class attribute that is a function object, a method object is created by packing (pointers to) the instance object and the function object just found together in an abstract object: this is the method object. When the method object is called with an argument list, a new argument list is constructed from the instance object and the argument list, and the function object is called with this new argument list.

<span id="tut-class-and-instance-variables"></span>

### <span class="section-number">9.3.5. </span>Class and Instance Variables<a href="#class-and-instance-variables" class="headerlink" title="Permalink to this headline">¶</a>

Generally speaking, instance variables are for data unique to each instance and class variables are for attributes and methods shared by all instances of the class:

    class Dog:

        kind = 'canine'         # class variable shared by all instances

        def __init__(self, name):
            self.name = name    # instance variable unique to each instance

    >>> d = Dog('Fido')
    >>> e = Dog('Buddy')
    >>> d.kind                  # shared by all dogs
    'canine'
    >>> e.kind                  # shared by all dogs
    'canine'
    >>> d.name                  # unique to d
    'Fido'
    >>> e.name                  # unique to e
    'Buddy'

As discussed in <a href="#tut-object" class="reference internal"><span class="std std-ref">A Word About Names and Objects</span></a>, shared data can have possibly surprising effects with involving <a href="https://docs.python.org/3/glossary.html#term-mutable" class="reference internal"><span class="xref std std-term">mutable</span></a> objects such as lists and dictionaries. For example, the *tricks* list in the following code should not be used as a class variable because just a single list would be shared by all *Dog* instances:

    class Dog:

        tricks = []             # mistaken use of a class variable

        def __init__(self, name):
            self.name = name

        def add_trick(self, trick):
            self.tricks.append(trick)

    >>> d = Dog('Fido')
    >>> e = Dog('Buddy')
    >>> d.add_trick('roll over')
    >>> e.add_trick('play dead')
    >>> d.tricks                # unexpectedly shared by all dogs
    ['roll over', 'play dead']

Correct design of the class should use an instance variable instead:

    class Dog:

        def __init__(self, name):
            self.name = name
            self.tricks = []    # creates a new empty list for each dog

        def add_trick(self, trick):
            self.tricks.append(trick)

    >>> d = Dog('Fido')
    >>> e = Dog('Buddy')
    >>> d.add_trick('roll over')
    >>> e.add_trick('play dead')
    >>> d.tricks
    ['roll over']
    >>> e.tricks
    ['play dead']

<span id="tut-remarks"></span>

<span class="section-number">9.4. </span>Random Remarks<a href="#random-remarks" class="headerlink" title="Permalink to this headline">¶</a>
--------------------------------------------------------------------------------------------------------------------------------------------

If the same attribute name occurs in both an instance and in a class, then attribute lookup prioritizes the instance:

    >>> class Warehouse:
            purpose = 'storage'
            region = 'west'

    >>> w1 = Warehouse()
    >>> print(w1.purpose, w1.region)
    storage west
    >>> w2 = Warehouse()
    >>> w2.region = 'east'
    >>> print(w2.purpose, w2.region)
    storage east

Data attributes may be referenced by methods as well as by ordinary users (“clients”) of an object. In other words, classes are not usable to implement pure abstract data types. In fact, nothing in Python makes it possible to enforce data hiding — it is all based upon convention. (On the other hand, the Python implementation, written in C, can completely hide implementation details and control access to an object if necessary; this can be used by extensions to Python written in C.)

Clients should use data attributes with care — clients may mess up invariants maintained by the methods by stamping on their data attributes. Note that clients may add data attributes of their own to an instance object without affecting the validity of the methods, as long as name conflicts are avoided — again, a naming convention can save a lot of headaches here.

There is no shorthand for referencing data attributes (or other methods!) from within methods. I find that this actually increases the readability of methods: there is no chance of confusing local variables and instance variables when glancing through a method.

Often, the first argument of a method is called `self`. This is nothing more than a convention: the name `self` has absolutely no special meaning to Python. Note, however, that by not following the convention your code may be less readable to other Python programmers, and it is also conceivable that a *class browser* program might be written that relies upon such a convention.

Any function object that is a class attribute defines a method for instances of that class. It is not necessary that the function definition is textually enclosed in the class definition: assigning a function object to a local variable in the class is also ok. For example:

    # Function defined outside the class
    def f1(self, x, y):
        return min(x, x+y)

    class C:
        f = f1

        def g(self):
            return 'hello world'

        h = g

Now `f`, `g` and `h` are all attributes of class `C` that refer to function objects, and consequently they are all methods of instances of `C` — `h` being exactly equivalent to `g`. Note that this practice usually only serves to confuse the reader of a program.

Methods may call other methods by using method attributes of the `self` argument:

    class Bag:
        def __init__(self):
            self.data = []

        def add(self, x):
            self.data.append(x)

        def addtwice(self, x):
            self.add(x)
            self.add(x)

Methods may reference global names in the same way as ordinary functions. The global scope associated with a method is the module containing its definition. (A class is never used as a global scope.) While one rarely encounters a good reason for using global data in a method, there are many legitimate uses of the global scope: for one thing, functions and modules imported into the global scope can be used by methods, as well as functions and classes defined in it. Usually, the class containing the method is itself defined in this global scope, and in the next section we’ll find some good reasons why a method would want to reference its own class.

Each value is an object, and therefore has a *class* (also called its *type*). It is stored as `object.__class__`.

<span id="tut-inheritance"></span>

<span class="section-number">9.5. </span>Inheritance<a href="#inheritance" class="headerlink" title="Permalink to this headline">¶</a>
--------------------------------------------------------------------------------------------------------------------------------------

Of course, a language feature would not be worthy of the name “class” without supporting inheritance. The syntax for a derived class definition looks like this:

    class DerivedClassName(BaseClassName):
        <statement-1>
        .
        .
        .
        <statement-N>

The name `BaseClassName` must be defined in a scope containing the derived class definition. In place of a base class name, other arbitrary expressions are also allowed. This can be useful, for example, when the base class is defined in another module:

    class DerivedClassName(modname.BaseClassName):

Execution of a derived class definition proceeds the same as for a base class. When the class object is constructed, the base class is remembered. This is used for resolving attribute references: if a requested attribute is not found in the class, the search proceeds to look in the base class. This rule is applied recursively if the base class itself is derived from some other class.

There’s nothing special about instantiation of derived classes: `DerivedClassName()` creates a new instance of the class. Method references are resolved as follows: the corresponding class attribute is searched, descending down the chain of base classes if necessary, and the method reference is valid if this yields a function object.

Derived classes may override methods of their base classes. Because methods have no special privileges when calling other methods of the same object, a method of a base class that calls another method defined in the same base class may end up calling a method of a derived class that overrides it. (For C++ programmers: all methods in Python are effectively `virtual`.)

An overriding method in a derived class may in fact want to extend rather than simply replace the base class method of the same name. There is a simple way to call the base class method directly: just call `BaseClassName.methodname(self,                     arguments)`. This is occasionally useful to clients as well. (Note that this only works if the base class is accessible as `BaseClassName` in the global scope.)

Python has two built-in functions that work with inheritance:

-   Use <a href="https://docs.python.org/3/library/functions.html#isinstance" class="reference internal" title="isinstance"><code class="sourceCode python"><span class="bu">isinstance</span>()</code></a> to check an instance’s type: `isinstance(obj,                         int)` will be `True` only if `obj.__class__` is <a href="https://docs.python.org/3/library/functions.html#int" class="reference internal" title="int"><code class="sourceCode python"><span class="bu">int</span></code></a> or some class derived from <a href="https://docs.python.org/3/library/functions.html#int" class="reference internal" title="int"><code class="sourceCode python"><span class="bu">int</span></code></a>.

-   Use <a href="https://docs.python.org/3/library/functions.html#issubclass" class="reference internal" title="issubclass"><code class="sourceCode python"><span class="bu">issubclass</span>()</code></a> to check class inheritance: `issubclass(bool,                         int)` is `True` since <a href="https://docs.python.org/3/library/functions.html#bool" class="reference internal" title="bool"><code class="sourceCode python"><span class="bu">bool</span></code></a> is a subclass of <a href="https://docs.python.org/3/library/functions.html#int" class="reference internal" title="int"><code class="sourceCode python"><span class="bu">int</span></code></a>. However, `issubclass(float,                         int)` is `False` since <a href="https://docs.python.org/3/library/functions.html#float" class="reference internal" title="float"><code class="sourceCode python"><span class="bu">float</span></code></a> is not a subclass of <a href="https://docs.python.org/3/library/functions.html#int" class="reference internal" title="int"><code class="sourceCode python"><span class="bu">int</span></code></a>.

<span id="tut-multiple"></span>

### <span class="section-number">9.5.1. </span>Multiple Inheritance<a href="#multiple-inheritance" class="headerlink" title="Permalink to this headline">¶</a>

Python supports a form of multiple inheritance as well. A class definition with multiple base classes looks like this:

    class DerivedClassName(Base1, Base2, Base3):
        <statement-1>
        .
        .
        .
        <statement-N>

For most purposes, in the simplest cases, you can think of the search for attributes inherited from a parent class as depth-first, left-to-right, not searching twice in the same class where there is an overlap in the hierarchy. Thus, if an attribute is not found in `DerivedClassName`, it is searched for in `Base1`, then (recursively) in the base classes of `Base1`, and if it was not found there, it was searched for in `Base2`, and so on.

In fact, it is slightly more complex than that; the method resolution order changes dynamically to support cooperative calls to <a href="https://docs.python.org/3/library/functions.html#super" class="reference internal" title="super"><code class="sourceCode python"><span class="bu">super</span>()</code></a>. This approach is known in some other multiple-inheritance languages as call-next-method and is more powerful than the super call found in single-inheritance languages.

Dynamic ordering is necessary because all cases of multiple inheritance exhibit one or more diamond relationships (where at least one of the parent classes can be accessed through multiple paths from the bottommost class). For example, all classes inherit from <a href="https://docs.python.org/3/library/functions.html#object" class="reference internal" title="object"><code class="sourceCode python"><span class="bu">object</span></code></a>, so any case of multiple inheritance provides more than one path to reach <a href="https://docs.python.org/3/library/functions.html#object" class="reference internal" title="object"><code class="sourceCode python"><span class="bu">object</span></code></a>. To keep the base classes from being accessed more than once, the dynamic algorithm linearizes the search order in a way that preserves the left-to-right ordering specified in each class, that calls each parent only once, and that is monotonic (meaning that a class can be subclassed without affecting the precedence order of its parents). Taken together, these properties make it possible to design reliable and extensible classes with multiple inheritance. For more detail, see <a href="https://www.python.org/download/releases/2.3/mro/" class="reference external">https://www.python.org/download/releases/2.3/mro/</a>.

<span id="tut-private"></span>

<span class="section-number">9.6. </span>Private Variables<a href="#private-variables" class="headerlink" title="Permalink to this headline">¶</a>
--------------------------------------------------------------------------------------------------------------------------------------------------

“Private” instance variables that cannot be accessed except from inside an object don’t exist in Python. However, there is a convention that is followed by most Python code: a name prefixed with an underscore (e.g. `_spam`) should be treated as a non-public part of the API (whether it is a function, a method or a data member). It should be considered an implementation detail and subject to change without notice.

Since there is a valid use-case for class-private members (namely to avoid name clashes of names with names defined by subclasses), there is limited support for such a mechanism, called *name mangling*. Any identifier of the form `__spam` (at least two leading underscores, at most one trailing underscore) is textually replaced with `_classname__spam`, where `classname` is the current class name with leading underscore(s) stripped. This mangling is done without regard to the syntactic position of the identifier, as long as it occurs within the definition of a class.

Name mangling is helpful for letting subclasses override methods without breaking intraclass method calls. For example:

    class Mapping:
        def __init__(self, iterable):
            self.items_list = []
            self.__update(iterable)

        def update(self, iterable):
            for item in iterable:
                self.items_list.append(item)

        __update = update   # private copy of original update() method

    class MappingSubclass(Mapping):

        def update(self, keys, values):
            # provides new signature for update()
            # but does not break __init__()
            for item in zip(keys, values):
                self.items_list.append(item)

The above example would work even if `MappingSubclass` were to introduce a `__update` identifier since it is replaced with `_Mapping__update` in the `Mapping` class and `_MappingSubclass__update` in the `MappingSubclass` class respectively.

Note that the mangling rules are designed mostly to avoid accidents; it still is possible to access or modify a variable that is considered private. This can even be useful in special circumstances, such as in the debugger.

Notice that code passed to `exec()` or `eval()` does not consider the classname of the invoking class to be the current class; this is similar to the effect of the `global` statement, the effect of which is likewise restricted to code that is byte-compiled together. The same restriction applies to `getattr()`, `setattr()` and `delattr()`, as well as when referencing `__dict__` directly.

<span id="tut-odds"></span>

<span class="section-number">9.7. </span>Odds and Ends<a href="#odds-and-ends" class="headerlink" title="Permalink to this headline">¶</a>
------------------------------------------------------------------------------------------------------------------------------------------

Sometimes it is useful to have a data type similar to the Pascal “record” or C “struct”, bundling together a few named data items. An empty class definition will do nicely:

    class Employee:
        pass

    john = Employee()  # Create an empty employee record

    # Fill the fields of the record
    john.name = 'John Doe'
    john.dept = 'computer lab'
    john.salary = 1000

A piece of Python code that expects a particular abstract data type can often be passed a class that emulates the methods of that data type instead. For instance, if you have a function that formats some data from a file object, you can define a class with methods `read()` and `readline()` that get the data from a string buffer instead, and pass it as an argument.

Instance method objects have attributes, too: `m.__self__` is the instance object with the method `m()`, and `m.__func__` is the function object corresponding to the method.

<span id="tut-iterators"></span>

<span class="section-number">9.8. </span>Iterators<a href="#iterators" class="headerlink" title="Permalink to this headline">¶</a>
----------------------------------------------------------------------------------------------------------------------------------

By now you have probably noticed that most container objects can be looped over using a <a href="https://docs.python.org/3/reference/compound_stmts.html#for" class="reference internal"><code class="xref std std-keyword docutils literal notranslate">for</code></a> statement:

    for element in [1, 2, 3]:
        print(element)
    for element in (1, 2, 3):
        print(element)
    for key in {'one':1, 'two':2}:
        print(key)
    for char in "123":
        print(char)
    for line in open("myfile.txt"):
        print(line, end='')

This style of access is clear, concise, and convenient. The use of iterators pervades and unifies Python. Behind the scenes, the <a href="https://docs.python.org/3/reference/compound_stmts.html#for" class="reference internal"><code class="xref std std-keyword docutils literal notranslate">for</code></a> statement calls <a href="https://docs.python.org/3/library/functions.html#iter" class="reference internal" title="iter"><code class="sourceCode python"><span class="bu">iter</span>()</code></a> on the container object. The function returns an iterator object that defines the method <a href="https://docs.python.org/3/library/stdtypes.html#iterator.__next__" class="reference internal" title="iterator.__next__"><code class="sourceCode python"><span class="fu">__next__</span>()</code></a> which accesses elements in the container one at a time. When there are no more elements, <a href="https://docs.python.org/3/library/stdtypes.html#iterator.__next__" class="reference internal" title="iterator.__next__"><code class="sourceCode python"><span class="fu">__next__</span>()</code></a> raises a <a href="https://docs.python.org/3/library/exceptions.html#StopIteration" class="reference internal" title="StopIteration"><code class="sourceCode python"><span class="pp">StopIteration</span></code></a> exception which tells the `for` loop to terminate. You can call the <a href="https://docs.python.org/3/library/stdtypes.html#iterator.__next__" class="reference internal" title="iterator.__next__"><code class="sourceCode python"><span class="fu">__next__</span>()</code></a> method using the <a href="https://docs.python.org/3/library/functions.html#next" class="reference internal" title="next"><code class="sourceCode python"><span class="bu">next</span>()</code></a> built-in function; this example shows how it all works:

    >>> s = 'abc'
    >>> it = iter(s)
    >>> it
    <iterator object at 0x00A1DB50>
    >>> next(it)
    'a'
    >>> next(it)
    'b'
    >>> next(it)
    'c'
    >>> next(it)
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
        next(it)
    StopIteration

Having seen the mechanics behind the iterator protocol, it is easy to add iterator behavior to your classes. Define an <a href="https://docs.python.org/3/reference/datamodel.html#object.__iter__" class="reference internal" title="object.__iter__"><code class="sourceCode python"><span class="fu">__iter__</span>()</code></a> method which returns an object with a <a href="https://docs.python.org/3/library/stdtypes.html#iterator.__next__" class="reference internal" title="iterator.__next__"><code class="sourceCode python"><span class="fu">__next__</span>()</code></a> method. If the class defines `__next__()`, then <a href="https://docs.python.org/3/reference/datamodel.html#object.__iter__" class="reference internal" title="object.__iter__"><code class="sourceCode python"><span class="fu">__iter__</span>()</code></a> can just return `self`:

    class Reverse:
        """Iterator for looping over a sequence backwards."""
        def __init__(self, data):
            self.data = data
            self.index = len(data)

        def __iter__(self):
            return self

        def __next__(self):
            if self.index == 0:
                raise StopIteration
            self.index = self.index - 1
            return self.data[self.index]

    >>> rev = Reverse('spam')
    >>> iter(rev)
    <__main__.Reverse object at 0x00A1DB50>
    >>> for char in rev:
    ...     print(char)
    ...
    m
    a
    p
    s

<span id="tut-generators"></span>

<span class="section-number">9.9. </span>Generators<a href="#generators" class="headerlink" title="Permalink to this headline">¶</a>
------------------------------------------------------------------------------------------------------------------------------------

<a href="https://docs.python.org/3/glossary.html#term-generator" class="reference internal"><span class="xref std std-term">Generators</span></a> are a simple and powerful tool for creating iterators. They are written like regular functions but use the <a href="https://docs.python.org/3/reference/simple_stmts.html#yield" class="reference internal"><code class="xref std std-keyword docutils literal notranslate">yield</code></a> statement whenever they want to return data. Each time <a href="https://docs.python.org/3/library/functions.html#next" class="reference internal" title="next"><code class="sourceCode python"><span class="bu">next</span>()</code></a> is called on it, the generator resumes where it left off (it remembers all the data values and which statement was last executed). An example shows that generators can be trivially easy to create:

    def reverse(data):
        for index in range(len(data)-1, -1, -1):
            yield data[index]

    >>> for char in reverse('golf'):
    ...     print(char)
    ...
    f
    l
    o
    g

Anything that can be done with generators can also be done with class-based iterators as described in the previous section. What makes generators so compact is that the <a href="https://docs.python.org/3/reference/datamodel.html#object.__iter__" class="reference internal" title="object.__iter__"><code class="sourceCode python"><span class="fu">__iter__</span>()</code></a> and <a href="https://docs.python.org/3/reference/expressions.html#generator.__next__" class="reference internal" title="generator.__next__"><code class="sourceCode python"><span class="fu">__next__</span>()</code></a> methods are created automatically.

Another key feature is that the local variables and execution state are automatically saved between calls. This made the function easier to write and much more clear than an approach using instance variables like `self.index` and `self.data`.

In addition to automatic method creation and saving program state, when generators terminate, they automatically raise <a href="https://docs.python.org/3/library/exceptions.html#StopIteration" class="reference internal" title="StopIteration"><code class="sourceCode python"><span class="pp">StopIteration</span></code></a>. In combination, these features make it easy to create iterators with no more effort than writing a regular function.

<span id="tut-genexps"></span>

<span class="section-number">9.10. </span>Generator Expressions<a href="#generator-expressions" class="headerlink" title="Permalink to this headline">¶</a>
-----------------------------------------------------------------------------------------------------------------------------------------------------------

Some simple generators can be coded succinctly as expressions using a syntax similar to list comprehensions but with parentheses instead of square brackets. These expressions are designed for situations where the generator is used right away by an enclosing function. Generator expressions are more compact but less versatile than full generator definitions and tend to be more memory friendly than equivalent list comprehensions.

Examples:

    >>> sum(i*i for i in range(10))                 # sum of squares
    285

    >>> xvec = [10, 20, 30]
    >>> yvec = [7, 5, 3]
    >>> sum(x*y for x,y in zip(xvec, yvec))         # dot product
    260

    >>> unique_words = set(word for line in page  for word in line.split())

    >>> valedictorian = max((student.gpa, student.name) for student in graduates)

    >>> data = 'golf'
    >>> list(data[i] for i in range(len(data)-1, -1, -1))
    ['f', 'l', 'o', 'g']

Footnotes

 <span class="brackets"><a href="#id1" class="fn-backref">1</a></span>   
Except for one thing. Module objects have a secret read-only attribute called <a href="https://docs.python.org/3/library/stdtypes.html#object.__dict__" class="reference internal" title="object.__dict__"><code class="sourceCode python">__dict__</code></a> which returns the dictionary used to implement the module’s namespace; the name <a href="https://docs.python.org/3/library/stdtypes.html#object.__dict__" class="reference internal" title="object.__dict__"><code class="sourceCode python">__dict__</code></a> is an attribute but not a global name. Obviously, using this violates the abstraction of namespace implementation, and should be restricted to things like post-mortem debuggers.

### [Table of Contents](https://docs.python.org/3/contents.html)

-   <a href="#" class="reference internal">9. Classes</a>
    -   <a href="#a-word-about-names-and-objects" class="reference internal">9.1. A Word About Names and Objects</a>
    -   <a href="#python-scopes-and-namespaces" class="reference internal">9.2. Python Scopes and Namespaces</a>
        -   <a href="#scopes-and-namespaces-example" class="reference internal">9.2.1. Scopes and Namespaces Example</a>
    -   <a href="#a-first-look-at-classes" class="reference internal">9.3. A First Look at Classes</a>
        -   <a href="#class-definition-syntax" class="reference internal">9.3.1. Class Definition Syntax</a>
        -   <a href="#class-objects" class="reference internal">9.3.2. Class Objects</a>
        -   <a href="#instance-objects" class="reference internal">9.3.3. Instance Objects</a>
        -   <a href="#method-objects" class="reference internal">9.3.4. Method Objects</a>
        -   <a href="#class-and-instance-variables" class="reference internal">9.3.5. Class and Instance Variables</a>
    -   <a href="#random-remarks" class="reference internal">9.4. Random Remarks</a>
    -   <a href="#inheritance" class="reference internal">9.5. Inheritance</a>
        -   <a href="#multiple-inheritance" class="reference internal">9.5.1. Multiple Inheritance</a>
    -   <a href="#private-variables" class="reference internal">9.6. Private Variables</a>
    -   <a href="#odds-and-ends" class="reference internal">9.7. Odds and Ends</a>
    -   <a href="#iterators" class="reference internal">9.8. Iterators</a>
    -   <a href="#generators" class="reference internal">9.9. Generators</a>
    -   <a href="#generator-expressions" class="reference internal">9.10. Generator Expressions</a>

#### Previous topic

[<span class="section-number">8. </span>Errors and Exceptions](errors.html "previous chapter")

#### Next topic

[<span class="section-number">10. </span>Brief Tour of the Standard Library](stdlib.html "next chapter")

### This Page

-   [Report a Bug](https://docs.python.org/3/bugs.html)
-   [Show Source](https://github.com/python/cpython/blob/3.9/Doc/tutorial/classes.rst)

### Navigation

-   [index](https://docs.python.org/3/genindex.html "General Index")
-   [modules](https://docs.python.org/3/py-modindex.html "Python Module Index") |
-   [next](stdlib.html "10. Brief Tour of the Standard Library") |
-   [previous](errors.html "8. Errors and Exceptions") |
-   ![](../_static/py.png)
-   [Python](https://www.python.org/) »
-   [3.9.5 Documentation](https://docs.python.org/3/index.html) »
-   [The Python Tutorial](index.html) »
-   

    |

© [Copyright](https://docs.python.org/3/copyright.html) 2001-2021, Python Software Foundation.  
The Python Software Foundation is a non-profit corporation. [Please donate.](https://www.python.org/psf/donations/)  
  
Last updated on May 30, 2021. [Found a bug](https://docs.python.org/3/bugs.html)?  
Created using [Sphinx](https://www.sphinx-doc.org/) 2.4.4.