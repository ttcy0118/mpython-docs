:mod:`ubluetooth` --- Bluetooth LE
=========================================

.. module:: ubluetooth
   :synopsis: BLE wireless function

The module provides low power Bluetooth control interface. Currently, it supports BLE in central, peripheral, broadcast and observer roles, and the device can run in multiple roles at the same time. 

This API is designed to match the low-power Bluetooth protocol and provide building blocks for higher-level abstractions such as specific device types.

.. note:: The module is still under development, and its classes, functions, methods, and constants may change.


BLE class
---------

BLE
-----------

.. class:: BLE()

    Returns BLE object

Configure
-------------

.. method:: BLE.active([active])

    (optional) change the active state of the ble radio and return to the current state. 

    The radio must be active before any other method of this kind can be used.

.. method:: BLE.config('param')
            BLE.config(param=value, ...)

    Get or set the configuration value of BLE interface. To get a value, the parameter name should be quoted in string quotes and only one parameter should be queried at a time. To set values, use key syntax, one or more parameters can be set at a time.

    Current supported value are:

    - ``'mac'``: Return the MAC address of the device. If the device has a fixed address (for example, PYBD), return it. Otherwise (for example, ESP32), a random address is generated when the ble interface is active.
    - ``'rxbuf'``: Sets the size, in bytes, of the internal buffer used to store incoming events. This buffer is a global buffer for the entire ble driver, so it can handle incoming data for all events, including all characteristics. Increasing this value can better handle incoming burst data (for example, scan results) and enable the central device to receive larger characteristic values.

Event processing
--------------

.. method:: BLE.irq(handler, trigger=0xffff)

    Register callbacks for events in the BLE stack. The handler receives two parameters, the ``event`` (see the event code below) and the ``data`` （which are specific event tuples of values) .

    The optional *trigger* parameter allows you to mask events of interest to the program. The default value of all events.

   

    Note: Items in the tuples of ``addr``, ``adv_data`` and ``uuid`` are referred data management :mod:`ubluetooth` Module (that is, the same instance will be reused multiple times to call the event handler).
    If your program wants to use this data outside of the handler, it must first copy them, for example, using ``bytes(addr)`` or ``bluetooth.UUID(uuid)`` .

    An event handler shows all possible events::

        def bt_irq(event, data):
            if event == _IRQ_CENTRAL_CONNECT:
                # The central device is already connected to this peripheral device
                conn_handle, addr_type, addr = data
            elif event == _IRQ_CENTRAL_DISCONNECT:
                # The central device is disconnected from this peripheral device
                conn_handle, addr_type, addr = data
            elif event == _IRQ_GATTS_WRITE:
                # The central device has written this feature or descriptor
                conn_handle, attr_handle = data
            elif event == _IRQ_GATTS_READ_REQUEST:
                # Read request from central device. Note: This is a hardwareIRQ
                # Return to NONE to reject the read operation
                # Note: This event do not support ESP32.
                conn_handle, attr_handle = data
            elif event == _IRQ_SCAN_RESULT:
                # Result of 1 scan
                addr_type, addr, connectable, rssi, adv_data = data
            elif event == _IRQ_SCAN_COMPLETE:
                # Scan duration completed or stopped manually
                pass
            elif event == _IRQ_PERIPHERAL_CONNECT:
                #  gap_connect() connected
                conn_handle, addr_type, addr = data
            elif event == _IRQ_PERIPHERAL_DISCONNECT:
                # Connected peripherals are disconnected
                conn_handle, addr_type, addr = data
            elif event == _IRQ_GATTC_SERVICE_RESULT:
                # Call gattc_discover_services() every service found
                conn_handle, start_handle, end_handle, uuid = data
            elif event == _IRQ_GATTC_CHARACTERISTIC_RESULT:
                # Call gattc_discover_services() every feature found
                conn_handle, def_handle, value_handle, properties, uuid = data
            elif event == _IRQ_GATTC_DESCRIPTOR_RESULT:
                # call gattc_discover_descriptors() every descriptor found
                conn_handle, dsc_handle, uuid = data
            elif event == _IRQ_GATTC_READ_RESULT:
                # gattc_read() completed
                conn_handle, value_handle, char_data = data
            elif event == _IRQ_GATTC_WRITE_STATUS:
                # gattc_write() completed
                conn_handle, value_handle, status = data
            elif event == _IRQ_GATTC_NOTIFY:
                # Peripheral has issued a notification request
                conn_handle, value_handle, notify_data = data
            elif event == _IRQ_GATTC_INDICATE:
                #Peripheral sends out instruction request
                conn_handle, value_handle, notify_data = data

Event code::

    from micropython import const
    _IRQ_CENTRAL_CONNECT                 = const(1 << 0)
    _IRQ_CENTRAL_DISCONNECT              = const(1 << 1)
    _IRQ_GATTS_WRITE                     = const(1 << 2)
    _IRQ_GATTS_READ_REQUEST              = const(1 << 3)
    _IRQ_SCAN_RESULT                     = const(1 << 4)
    _IRQ_SCAN_COMPLETE                   = const(1 << 5)
    _IRQ_PERIPHERAL_CONNECT              = const(1 << 6)
    _IRQ_PERIPHERAL_DISCONNECT           = const(1 << 7)
    _IRQ_GATTC_SERVICE_RESULT            = const(1 << 8)
    _IRQ_GATTC_CHARACTERISTIC_RESULT     = const(1 << 9)
    _IRQ_GATTC_DESCRIPTOR_RESULT         = const(1 << 10)
    _IRQ_GATTC_READ_RESULT               = const(1 << 11)
    _IRQ_GATTC_WRITE_STATUS              = const(1 << 12)
    _IRQ_GATTC_NOTIFY                    = const(1 << 13)
    _IRQ_GATTC_INDICATE                  = const(1 << 14)


o save space in the firmware, these constants are not included in the :mod:`ubluetooth` . Add what you need from the list above to your program. 


Advertiser
-----------------------------

.. method:: BLE.gap_advertise(interval_us, adv_data=None, resp_data=None, connectable=True)

    Starts broadcasting at the specified time interval (in microseconds). The interval will be rounded to the nearest 625 microseconds. To stop broadcasting, set `interval_us` to NONE.

    *adv_data* and *resp_data* can be any `buffer` type (for example ``bytes``, ``bytearray``, ``str``)。
    *adv_data* is included in all broadcasts and *resp_data* is sent in response to a valid scan.
 

    Note：If *adv_data* （or  *resp_data* ）as NONE, Then reuse the data passed to the previous call ``gap_advertise`` .
    In this way, the broadcaster can use to recover the broadcast ``gap_advertise(interval_us)`` . To clear the broadcast load, pass an empty bytes，that is b''.

Scanner
-----------------------

.. method:: BLE.gap_scan(duration_ms, [interval_us], [window_us])

    Runs a scan operation for a specified duration in milliseconds.

    To scan indefinitely, set *duration_ms* set to ``0`` . To stop scanning, set *duration_ms* to ``None`` . 
    
    Use *interval_us* and *window_us* to configure the duty cycle. 
    Scanner will run *window_us* at a interval of 1 microsecond, The total duration is milliseconds. The default interval and window are 1.28 seconds and 11.25 milliseconds, respectively.

    For each scan result, *_IRQ_SCAN_RESULT* initiate this event.

    When a scan is stopped (due to the end of the duration or explicitly stopped), *_IRQ_SCAN_COMPLETE* initiate this event.



GATT Server
-----------------------------

BLE peripherals have a set of registration services. Each service may contain attributes, each with a value. A feature can also contain a descriptor, which itself has a value.

These values are stored locally and accessed through the value handle generated during service registration. They can also be read or written by remote gatt client. 
In addition, the gatt server can “notify” a feature to a connected gatt client through a connection handle.

Default maximum of 20 bytes for features and descriptors. Anything written to them by a central device will be truncated to this length. However, any local write increases the maximum size,
So if you want to write longer data, please sign up and use ``gatts_write`` . for example, gatts_write(char_handle, bytes(100))


.. method:: BLE.gatts_register_services(services_definition)

    Configure the gatt server with the specified service, replacing all existing services. 

    *services_definition* is a list of services, each of which is a binary group containing UUID and feature list.

    Each feature is a 2 or 3 element tuple that contains the `UUID`，`flags` value and an optional descriptor list.

    Each descriptor is a binary group containing UUID and a flags value.

    flags is a bitwise or combined :data:`ubluetooth.FLAG_READ`，:data:`ubluetooth.FLAG_WRITE` and :data:`ubluetooth.FLAG_NOTIFY` . As defined below:

    The return value is a list of tuples (one element per service) (each element is a value handle). The feature and descriptor handles are flattened into the same tuple in the defined order.



    Example of 2 registers services (Heart Rate, and Nordic UART)::

        HR_UUID = bluetooth.UUID(0x180D)
        HR_CHAR = (bluetooth.UUID(0x2A37), bluetooth.FLAG_READ | bluetooth.FLAG_NOTIFY,)
        HR_SERVICE = (HR_UUID, (HR_CHAR,),)
        UART_UUID = bluetooth.UUID('6E400001-B5A3-F393-E0A9-E50E24DCCA9E')
        UART_TX = (bluetooth.UUID('6E400003-B5A3-F393-E0A9-E50E24DCCA9E'), bluetooth.FLAG_READ | bluetooth.FLAG_NOTIFY,)
        UART_RX = (bluetooth.UUID('6E400002-B5A3-F393-E0A9-E50E24DCCA9E'), bluetooth.FLAG_WRITE,)
        UART_SERVICE = (UART_UUID, (UART_TX, UART_RX,),)
        SERVICES = (HR_SERVICE, UART_SERVICE,)
        ( (hr,), (tx, rx,), ) = bt.gatts_register_services(SERVICES)

    These three value handle(``hr``, ``tx``, ``rx``) available and use :meth:`gatts_read <BLE.gatts_read>`, :meth:`gatts_write <BLE.gatts_write>`,
    and :meth:`gatts_notify <BLE.gatts_notify>` 。

    Note：Advertisement must be stopped before registering service. 

.. method:: BLE.gatts_read(value_handle)

    Read local value handle (The value is determined by :meth:`gatts_write <BLE.gatts_write>` or gatt client write in）.

.. method:: BLE.gatts_write(value_handle, data)

    Write the local value handle, which can be read by the gatt client.


.. method:: BLE.gatts_notify(conn_handle, value_handle, [data])

    Notifies the connected gatt client that the value has changed and should issue a read value for the current value of this GATT　server.

    If data is specified, the value is sent to the GATT client device as part of the notification, avoiding the need for a separate read request. Note that this does not update the stored local values.

  If the data is specified, the value will be sent to the GATT client device as part of the notification, so as to avoid the need for a separate read request. Note that this does not update the stored local values.


.. method:: BLE.gatts_set_buffer(value_handle, len, append=False)


    Set internal buffer size (in bytes). This limits the maximum value that can be received. The default value is 20.0。
    Setting ``append`` to `True` will append all remote writes to the current value instead of replacing the current value. This can buffer up to len bytes.
    When use :meth:`gatts_read <BLE.gatts_read>` , the value will be cleared after reading. This feature is useful when implementing something, such as the Nordic UART service.



GATT Client
--------------------------

.. method:: BLE.gap_connect(addr_type, addr, scan_duration_ms=2000)

    Connect to GATT server. If it is successful, the event ``_IRQ_PERIPHERAL_CONNECT``  will be triggered.

.. method:: BLE.gap_disconnect(conn_handle)

    Disconnects the specified connection handle. Success, the even ``_IRQ_PERIPHERAL_DISCONNECT`` will be triggered.
    If the connection handle is not connected, return ``False`` , otherwise return ``True`` .


.. method:: BLE.gattc_discover_services(conn_handle)

    Querying services for connected peripherals.

    For each discovered service, the even ``_IRQ_GATTC_SERVICE_RESULT`` will be triggered.

.. method:: BLE.gattc_discover_characteristics(conn_handle, start_handle, end_handle)

    Queries the connected peripheral for features within a specified range.
    Each feature discovery will trigger the event ``_IRQ_GATTC_CHARACTERISTIC_RESULT`` .


.. method:: BLE.gattc_discover_descriptors(conn_handle, start_handle, end_handle)

    Query the connected peripheral for descriptors in the specified range.

    Every time a feature is found, the event ``_IRQ_GATTC_DESCRIPTOR_RESULT`` will be triggered.


.. method:: BLE.gattc_read(conn_handle, value_handle)

    Issues a remote read to the attached peripheral to get the specified attribute or descriptor handle.

    If successful, the event ``_IRQ_GATTC_READ_RESULT`` will be triggered.

.. method:: BLE.gattc_write(conn_handle, value_handle, data, mode=0)

    Issue a remote write to a connected peripheral for a specified feature or descriptor handle.

    - ``mode``

        -  ``mode=0`` （default）no response write operation：Writes are sent to the remote peripheral, but no acknowledgment is returned and no events are raised.
        -  ``mode=1`` i is response write: requests the remote peripheral to send a response / acknowledgement that it has received data.

    If a response is received from a remote peripheral, the event``_IRQ_GATTC_WRITE_STATUS`` will triggered. 


UUID class
----------

UUID
-----------

.. class:: UUID(value)

    Creates a UUID instance with the specified value.

    This value can be：

    - A 16 bit integer. For example ``0x2908``.
    - 128 bit UUID string. For example ``'6E400001-B5A3-F393-E0A9-E50E24DCCA9E'``.


Constant
---------

.. data:: ubluetooth.FLAG_READ
          ubluetooth.FLAG_WRITE
          ubluetooth.FLAG_NOTIFY


.. literalinclude:: /../../examples/ble/ble_advertising.py
    :caption: ble_advertising.py(BLE broadcast)
    :linenos:


.. literalinclude:: /../../examples/ble/ble_temperature.py
    :caption: This example demonstrates a simple temperature sensor peripheral
    :linenos:
