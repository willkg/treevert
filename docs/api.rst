===========
Treelib API
===========

Library for manipulating trees in Python made up of dicts and lists.


.. py:func:: tree_get(tree, key, default=None)

   Given a tree consisting of mappings (like dict) and indexable sequences (like
   list), returns the value specified by the key.

   The key is a period-delimited sequence of edges to traverse. If one of the
   ediges doesn't exist, then the default is returned immediately.

   Examples:

   >>> tree_get({'a': 1}, 'a')
   1
   >>> tree_get({'a': 1}, 'b')
   None
   >>> tree_get({'a': {'b': 2}}, 'a.b')
   2
   >>> tree_get({'a': {'b': 2}}, 'a.b.c', default=55)
   55

   This supports sequences, too:

   >>> tree_get({'a': [1, 2, 3]}, 'a.1')
   2
   >>> tree_get({'a': {'1': 2}}, 'a.1')
   2

   Both dict and list support getitem notation, so the ``1`` works fine.


.. py:func:: tree_set(tree, key, value)

   Given a tree consisting of mappings (like dict) and indexable sequences that
   support getitem notation, sets the key to the value.

   The key is a period-delimited sequence of edges to traverse. If one of the
   ediges doesn't exist, then the edge is created using these rules:

   1. if the next edge is an integer, then it creates a list
   2. if the next edge is not an integer, then it creates a dict

   This returns the tree which is mutated in place.

   >>> tree_set({}, 'a', value=5)
   {'a': 5}
   >>> tree_set({}, 'a.b.c', value=5)
   {'a': {'b': {'c': 5}}}

   This will not create new list indexes:

   >>> tree_set({}, 'a.1', value=5)
   IndexError('list index out of range')

   This is the same error you'd get if you tried to access an index that doesn't
   exist in a list.


   .. Note::

      Other ideas

      Add a ``create_indexes=True`` argument that'll let it fill in missing
      indexes with ``None``:

      >>> tree_set({}, 'a.1.b', value=5, create_indexes=True)
      {'a': [None, {'b': 5}]}


.. py:func:: tree_flatten(tree)

   FIXME


.. py:func:: tree_validate(tree, fun)

   FIXME


.. py:func:: tree_traverse(tree, fun)

   FIXME