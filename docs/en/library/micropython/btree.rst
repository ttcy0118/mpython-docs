:mod:`btree` -- Simple BTree database
=====================================

.. module:: btree
   :synopsis: simple BTree database

 ``btree`` module uses external storage (disk file, or random access stream in general) to realize a simple key value database.
 Key sorting is stored in the database. In addition to effective retrieval of a single key value, the database also supports efficient ordered range scanning (using keys within a given range to retrieve values).
 In the aspect of application program interface, B-tree database should work in a way similar to the standard `dict` type as much as possible. One obvious difference is that both the key and the value is needed.
 Is a `bytes` object（therefore, if you want to store other types of objects, you need to serialize them to `bytes` first）. 


 This module is based on the 1.xx version of the famous berkelydb library.

Example::

    import btree

    # First, we need to open a stream which holds a database 
    # This is usually a file, but can be in-memory database 
    # using uio.BytesIO, a raw flash partition, etc. 
    # Oftentimes, you want to create a database file if it doesn't
    # exist and open if it exists. Idiom below takes care of this.
    # Usually, if there is no database file, you need to create one; if there is, then just open it. The following idioms take this into account.
    # DO NOT open database with "a+b" access mode.
    
    try:
        f = open("mydb", "r+b")
    except OSError:
        f = open("mydb", "w+b")

    # Now open a database itself 
    db = btree.open(f)

    # The keys you add will be sorted internally in the database 
    db[b"3"] = b"three"
    db[b"1"] = b"one"
    db[b"2"] = b"two"

    # Assume that any changes are cached in memory unless
    # explicitly flushed (or database closed). Flush database
    # at the end of each "transaction". 
    # Any changes are assumed to be cached in memory unless explicitly flushed (or the database is shut down). Refresh the database at the end of each process.
    db.flush()

    # Prints b'two'
    print(db[b"2"])

    # Iterate over sorted keys in the database, starting from b"2"
    # until the end of the database, returning only values. 
    # Iterate over the sorted keys in the database, starting from b“2” to the end of the database, and only return values.
    # Mind that arguments passed to values() method are *key* values. 
    # Prints:
    #   b'two'
    #   b'three'
    for word in db.values(b"2"):
        print(word)

    del db[b"2"]

    # No longer true, prints False 
    print(b"2" in db)

    # Prints:
    #  b"1"
    #  b"3"
    for key in db:
        print(key)

    db.close()

    # Don't forget to close the underlying stream! 
    f.close()


Function
---------

.. function:: open(stream, \*, flags=0, cachesize=0, pagesize=0, minkeypage=0)

   Open a database from a random access ``stream``(similar to an open file). All other parameters are optional and are only keywords, and allow adjustment of advanced parameters of database operation (most users do not need this):

   * *flags* - currently unused
   * *cachesize* - Recommended maximum memory cache size in bytes. For a board with sufficient memory, using a larger value may improve performance. This value is only the recommended value. If the value is set too low, the module may occupy more memory.
   * *pagesize* - Page size for nodes in BTree. The acceptable range is 512-65536. If 0, the size of the underlying I/O block is used (the best compromise between memory usage and performance).
   * *minkeypage* - The minimum number of keys stored per page. The default value is 0 equals 2. 

   Returns a B-tree object that implements a dictionary protocol (method set) and some of the following additional methods.

Method
-------

.. method:: btree.close()

   Close the database. Shutting down the database at the end of processing is mandatory because some unwritten data may remain in the cache. Note：This does not close the underlying flow that was opened with the database, which should be closed separately (this is also mandatory to ensure that data flushed from the buffer enters the underlying storage).

.. method:: btree.flush()

   Flushes any data in the cache to the underlying stream. 

.. method:: btree.__getitem__(key)
            btree.get(key, default=None)
            btree.__setitem__(key, val)
            btree.__detitem__(key)
            btree.__contains__(key)

   Standard dictionary method. 

.. method:: btree.__iter__()

   BTree objects can be iterated directly (similar to dictionaries) to access all keys in order.

.. method:: btree.keys([start_key, [end_key, [flags]]])
            btree.values([start_key, [end_key, [flags]]])
            btree.items([start_key, [end_key, [flags]]])

   These methods are similar to the standard dictionary methods, but you can also use optional parameters to iterate over a key sub scope rather than the entire database.
   NOte：Among the three methods, *start_key* and *end_key* parameters represent key values. For example, the value ``values()`` method iterates over the values corresponding to a given key range.
   No *start_key* value means “from the first key”, no *end_key*  value or its value is none means “until the end of the database”。
   *start_key* ，not includes *end_key* ，Can include *end_key* in the iteration by passing the tag of `btree.INCL` . 
   To iterate in the down key direction by passing the `btree.DESC` tag. Tag value can be the same as ORed。

Constant
---------

.. data:: INCL

    `keys()`, `values()`, `items()` Method, specifying that the scan should contain the end key.
    
.. data:: DESC

    `keys()`, `values()`, `items()` Method, specifies that the scan should follow the down direction of the key.
