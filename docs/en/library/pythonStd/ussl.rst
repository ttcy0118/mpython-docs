:mod:`ussl` -- SSL/TLS module
=============================

.. module:: ussl
   :synopsis: TLS/SSL wrapper for socket objects

This module implements the corresponding :term:`CPython`  a subset of modules, as described below. Refers to CPython document for details:: `ssl <https://docs.python.org/3.5/library/ssl.html#module-ssl>`_

This module provides access to transport layer security (formerly known as “secure sockets layer”）encryption and peer-to-peer authentication tools for client and server-side network sockets.

---------

.. function:: ssl.wrap_socket(sock, server_side=False, keyfile=None, certfile=None, cert_reqs=CERT_NONE, ca_certs=None)


Adopt stream sock (Usually a  usocket.socket instead of type SOCK_STREAM），and return an instance of ssl.SSLSocket, This instance wraps the underlying flow in the SSL context. 
The returned object has the usual stream interface methods, such as ``read()`` ，``write()`` etc.
In MicroPython, objects returned do not expose socket interfaces and methods, such asrecv()，send(). 
In particular, a server-side SSL socket should be created from the normal socket returned on the ``accept()`` non SSL listening server socket.




.. warning::

    Some implementations of the module do not validate the server certificate, which makes the established SSL connection prone to man in the middle attack.
    
Error
----------

.. data:: ssl.SSLError

   This exception does not exist. Instead, it uses its base classOSError。
   

Constant
---------

.. data:: ssl.CERT_NONE
          ssl.CERT_OPTIONAL
          ssl.CERT_REQUIRED

    The value supported by cert_reqs parameter.
