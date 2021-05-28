reprshed
========

A toolshed for writing great ``__repr__`` methods quickly and easily.

Replace boilerplate like this:

.. code:: python

    @reprlib.recursive_repr('<...>')
    def __repr__(self):
        return f'{type(self).__name__}({self._foo!r}, bar={self._bar!r})'

with just this:

.. code:: python

    def __repr__(self):
        return reprshed.pure(self, self._foo, bar=self._bar)


Versioning
----------

This library's version numbers follow the `SemVer 2.0.0
specification <https://semver.org/spec/v2.0.0.html>`_.


Installation
------------

::

    pip install reprshed


Usage
-----

Import:

.. code:: python

    import reprshed

For a "pure" class (a class whose exact state can be recreated purely
by passing the right constructor arguments), use ``reprshed.pure``.

For an "impure" class (a class whose exact state is not reproducible),
use ``reprshed.impure``.

Examples!

.. code:: python

    >>> class MyClass:
    ...     def __init__(self, foo, bar, qux):
    ...         self._foo = foo
    ...         self._bar = bar
    ...         self._qux = qux
    ...     def __repr__(self):
    ...         return reprshed.pure(self, self._foo, self._bar, self._qux)

The first argument to ``reprshed.pure`` must be ``self``, because
``reprshed`` uses that to get the class name. All other arguments
are formatted as arguments to that class constructor:

.. code:: python

    >>> MyClass('foo', 'bar', qux='qux')
    MyClass('foo', 'bar', 'qux')

Formatting the ``repr`` as constructor keyword arguments is easy too:

.. code:: python

    >>> class MyClass:
    ...     def __init__(self, foo, bar):
    ...         self._foo = foo
    ...         self._bar = bar
    ...     def __repr__(self):
    ...         return reprshed.pure(self, foo=self._foo, bar=self._bar)
    ... 
    >>> MyClass(1, bar='qux')
    MyClass(foo=1, bar='qux')

You also get recursion protection automatically on all
common Python implementations:

.. code:: python

    >>> class MyClass:
    ...     def __init__(self, number, foo):
    ...         self.number = number
    ...         self.foo = foo
    ...     def __repr__(self):
    ...         return reprshed.pure(self, self.number, foo=self.foo)
    ... 
    >>> my_instance = MyClass(1, 'whatever')
    >>> my_instance
    MyClass(1, foo='whatever')
    >>> my_instance.foo = my_instance
    >>> my_instance
    MyClass(1, foo=<...>)
    >>> my_instance.foo = MyClass(2, [None, my_instance, 3, '4'])
    >>> my_instance
    MyClass(1, foo=MyClass(2, foo=[None, <...>, 3, '4']))

Using ``reprshed.impure`` is the same, only the output format changes:

.. code:: python

    >>> class MyClass:
    ...     def __repr__(self):
    ...         return reprshed.impure(self, 1234, 'foo', bar=0, qux='qux')
    ... 
    >>> MyClass()
    <MyClass 1234 'foo' bar=0 qux='qux'>

If you need to take more manual control, you can use ``reprshed.raw``:

.. code:: python

    >>> class MyClass:
    ...     def __repr__(self):
    ...         return reprshed.impure(self, foo=5, bar=reprshed.raw('qux()'))
    ... 
    >>> MyClass()
    <MyClass foo=5 bar=qux()>

By passing ``reprshed.raw`` as a positional argument, you can get
arbitrary formatting inside the ``repr`` if you really need to:

.. code:: python

    >>> class MyClass:
    ...     def __repr__(self):
    ...         return reprshed.impure(self, reprshed.raw('a b () c,{d,e}'))
    ... 
    >>> MyClass()
    <MyClass a b () c,{d,e}>

You can even use ``reprshed.raw`` to override the class name:

.. code:: python

    >>> class MyClass:
    ...     def __repr__(self):
    ...         return reprshed.impure(reprshed.raw('fake name'))
    ... 
    >>> MyClass()
    <fake name>
