===========
Treelib API
===========

Library for manipulating trees in Python made up of dicts and lists.


Goals
=====

The primary goal of this library is to make it less unwieldy to manipulate trees
made up of Python dicts and lists.

For example, say we want to get a value deep in the tree. we could do this::

  value = tree['a']['b']['c']


That'll throw a ``KeyError`` if any of those bits are missing. So you could
handle that::

  try:
      value = tree['a']['b']['c']
  except KeyError:
      value = None


Alternatively, you could do this::

  value = tree.get('a', {}).get('b', {}).get('c': None)


These work, but both are unwieldy especially if you're doing this a lot.

Similarly, setting things deep is also unenthusing::

  tree['a']['b']['c'] = 5


The safer form is this::

  tree.setdefault('a', {}).setdefault('b', {})['c'] = 5


This library aims to make sane use cases for tree manipulation easier to read
and think about.


Paths
=====

A path is a string specifying a period-delimited list of edges. Edges can be:

1. a key (for a dict)
2. an index (for a list)

Example paths::

  a
  a.[1].foo_bar.Bar
  a.b.[-1].Bar


Paths can be composed using string operations since they're just strings.

FIXME(willkg): Add diagram showing a tree with edges specified by a path.


Key
---

Keys are identifiers that are:

1. composed entirely of ascii alphanumeric characters, hyphens, and underscores
2. at least one character long

For example, these are all valid keys::

  a
  foo
  FooBar
  Foo-Bar
  foo_bar


Index
-----

Indexes indicate a 0-based list index. They are:

1. integers
2. wrapped in ``[`` and ``]``
3. can be negative

For example, these are all valid indexes::

  [0]
  [1]
  [-50]


API
===

.. py:func:: tree_get(tree, path, default=None)

   Given a tree consisting of dicts and lists, returns the value specified by
   the path.

   Some things to know about ``tree_get()``:

   1. It doesn't alter the tree.
   2. Once it hits an edge that's missing, it returns the default.

   Examples:

   >>> tree_get({'a': 1}, 'a')
   1
   >>> tree_get({'a': 1}, 'b')
   None
   >>> tree_get({'a': {'b': 2}}, 'a.b')
   2
   >>> tree_get({'a': {'b': 2}}, 'a.b.c', default=55)
   55
   >>> tree_get({'a': {'1': 2}}, 'a.1')
   2
   >>> tree_get({'a': [1, 2, 3]}, 'a.[1]')
   2
   >>> tree_get({'a': [{}, {'b': 'foo'}]}, 'a.[1].b')
   'foo'


.. py:func:: tree_set(tree, path, value, mutate=True, create_missing=False)

   Given a tree consisting of dicts and lists, sets the item specified by path
   to the specified value.

   If one of the edges doesn't exist, then this raises either a ``KeyError``
   for dicts or a ``IndexError`` for lists.

   :arg boolean mutate: If ``mutate`` is ``True`` (the default), then this
       changes the tree in place and returns the mutated tree.

       If ``mutate`` is ``False``, then this does a deepcopy of the tree,
       changes the copy, and returns the copy. This is expensive.

   :arg boolean create_missing: If ``create_missing`` is ``False`` (the default),
      then this will raise a ``KeyError`` for failed dict keys and
      ``IndexError`` for failed list indexes.

      If ``create_missing`` is ``True``, and this isn't
      the last item in the path, then this will create the intermediary
      dict/list.

      If the next edge is a key, it'll create a dict. If the next edge is an
      index, then it'll create a list filling in ``None`` for the required
      indices.

      Here are some examples.

      This sets ``a`` to 5. This isn't affected by ``create_missing``.

      >>> tree_set({}, 'a', value=5, create_missing=True)
      {'a': 5}
      >>> tree_set({}, 'a', value=5, create_missing=False)
      {'a': 5}

      This tries to traverse ``a``, but it doesn't exist and it's not the last
      edge in the path. The next edge is ``b``, which is a key, so it first sets
      ``a`` to an empty dict, then proceeds.

      >>> tree_set({}, 'a.b', value=5, create_missing=True)
      {'a': {'b': 5}}

      This tries to traverse ``a``, but it doesn't exist and it's not the last
      edge in the path. The next edge is ``[2]``, which is an index, so it first
      sets ``a`` to a list of 3 ``None`` values, then proceeds.

      >>> tree_set({}, 'a.[2]', value=5, create_missing=True)
      {'a': [None, None, 5]}

      This is similar, but with a negative index.

      >>> tree_set({}, 'a.[-1]', value=5, create_missing=True)
      {'a': [5]}

      This creates missing indices in an existing list.

      >>> tree_set({'a': []}, 'a.[2]', value=5, create_missing=True)
      {'a': [None, None, 5]}


   Examples:

   These don't mutate the tree:

   >>> tree = {'a': {'b': {'c': 1}}}
   >>> tree_set(tree, 'a', value=5, mutate=False)
   {'a': 5}
   >>> tree_set(tree, 'a.b.c', value=[], mutate=False)
   {'a': {'b': {'c': []}}}

   These raise errors if an edge is missing:

   >>> tree_set({}, 'a.b.c', value=5)
   KeyError ...
   >>> tree_set({}, 'a.[1].b', value=5)
   IndexError ...

   These create missing edges and indexes:

   >>> tree_set({}, 'a.b.c', value=5, create_missing=True)
   {'a': {'b': {'c': 5}}}
   >>> tree_set({}, 'a.[1].b', value=5, create_missing=True)
   {'a': [None, {'b': 5}]}


.. py:func:: tree_flatten(tree)

   Flattens a tree into a dict with keys of paths.

   >>> tree_flatten({'a': 1})
   {'a': 1}
   >>> tree_flatten({'a': {'b': 1, 'c': 2}})
   {'a.b': 1, 'a.c': 2}
   >>> tree_flatten({'a': [{'b': 1}, {'c': 2}]})
   {'a.[0].b': 1, 'a.[1].c': 2}

   .. Note::

      At this point, a flattened tree can't be used using ``tree_get`` and
      ``tree_set``.


.. py:func:: tree_setdefault(tree, default_tree)

   FIXME


.. py:func:: tree_validate(tree, schema)

   FIXME


.. py:func:: tree_traverse(tree, fun)

   FIXME


Research and Inspirations
=========================

Python ``defaultdict``
----------------------

Python has a defaultdict

https://docs.python.org/3/library/collections.html#defaultdict-objects

This doens't handle lists and dicts well, though.

We'd have to either create the original data structure as a defaultdict, or
convert it to one.

If you try to get something deep from a defaultdict, it mutates the
structure.

It doesn't easily support composable paths.


jq processor
------------

jq has interesting filter syntax.

https://stedolan.github.io/jq/manual/#Basicfilters


Creating a new subclass of Python ``dict``
------------------------------------------

We could do that and add ``get_path`` and ``set_path``, but I wonder if we can
get the utility we want without having to box/unbox data.

If we're just working with dicts and lists and standard Python things, then
``json.dumps`` and other things just work without us having to do anything about
them.
