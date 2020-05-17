:mod:`ucryptolib` -- Encrypted Password
==========================================

.. module:: ucryptolib
   :synopsis: encrypted password

Class
-------

.. class:: aes

    .. classmethod:: __init__(key, mode, [IV])

        Initialize password object for encryption / decryption. Note:：After initialization, the password object can only be used for encryption or decryption. Running the decrypt() operation after encrypt() .

        Parameter:

            * ``key`` Encryption / decryption key (byte alike)
            * ``mode`` :

                * ``1`` (or ``ucryptolib.MODE_ECB`` if present）Electronic Codebook(ECB)
                * ``2`` (ucryptolib.MODE_CBC if present）Cipher-Block Chaining(CBC)
                * ``6`` (or ucryptolib.MODE_CTR if present) For Counter mode (CTR)

            * ``IV`` Initialization vector of CBC mode
            * For counter mode, IV is the initial value of the counter

    .. method:: encrypt(in_buf, [out_buf])

        Encryption in_buf。 If not givenout _buf， The result is returned as the newly allocated bytes object。
        Otherwise, the result is written to the variable buffer out_buf。 in_buf and out_bufcan also refer to the same variable buffer, in which case the data is encrypted in place.

    .. method:: decrypt(in_buf, [out_buf])

        Similar as `encrypt()` , but used for decryption. 
