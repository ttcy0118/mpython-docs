:mod:`machine` --- Hardwares related function
====================================================

.. module:: machine
   :synopsis: hardwares related function

The ``machine`` module contains specific functions related to the hardware on a specific circuit board. Most of the functions in this module allow direct and unrestricted access to and control the hardware blocks on the system (such as CPU, timer, bus, etc.).
Improper use can cause failures, locks, board crashes, and, in extreme cases, hardware damage.

.. _machine_callbacks:

Comments on :mod:`machine` module functions and callback used by class methods：All these callbacks should be considered to be executed in the context of the interrupt.
This is true for both physical devices with ID> = 0 and “virtual” devices with negative IDS (for example, - 1)（these “virtual” devices are still thin pads above real and real hardware interrupts）.
See :ref:`interrupt handler <isr_rules>`



 .. toctree::
   :maxdepth: 1

   machine.Pin.rst
   machine.ADC.rst 
   machine.TouchPad.rst
   machine.PWM.rst
   machine.UART.rst
   machine.I2C.rst
   machine.SPI.rst
   machine.Timer.rst
   machine.RTC.rst
   machine.WDT.rst






Reset correlation function
-----------------------

.. function:: reset()

    It has the same effect as pressing the external reset button

.. function:: reset_cause()

    Cause for Reset

    ==================== ==============  ====================================  
     Cause for Reset     Numerial value  Definition
     PWRON_RESET          1              Power on and restart 
     HARD_RESET           2              Hard restart
     WDT_RESET            3              Watchdog timer restart 
     DEEPSLEEP_RESET      4              Restart from sleep 
     SOFT_RESET           5              Soft restart 
    ==================== =============  ====================================  



Interrupt correlation function
---------------------------

.. function:: disable_irq()

    Disable interrupt request. Returns the previous IRQ status, which should be treated as opaque  :func:`enable_irq()` before calling :func:`disable_irq()` , 
    This return value should be passed to the function to restore the interrupt to its original state.


.. function:: enable_irq(state)

    Re-enable interrupt request.  :func:`state` Parameter should be the last call  :func:`disable_irq()` Value returned when function.

Power correlation function
-----------------------

.. function:: freq()

    Return CPU frequency in Hz

.. function:: idle()

   Provides a clock for the CPU to help reduce power consumption at any time in the short or long term. Once any interrupt is triggered, the peripheral continues to work and execute
   （On many ports, this includes system timer interrupts that occur at regular intervals in milliseconds）.

.. function:: sleep()

   .. note:: This function is not recommended. You can use lightsleep() without parameters. 

.. function:: deepsleep()

    Stop execution to try to enter low power state. 

    It time_ms  is specified, this is the maximum amount of time (in milliseconds) that sleep will last.

  Error reporting bilingual control, then this will be the longest sleep duration（in milliseconds）. Or sleep can last indefinitely.

    Whether there is time or not, if there is an event to be handled, the execution can be recovered at any time. This type of event or wake-up source should be configured before hibernation, such as `Pin` change or  `RTC` timeout. 

    ``lightsleep`` and ``deepsleep`` precise behavior and power saving function of depends on the underlying hardware to a large extent, but the general attributes are:：

        - lightsleep has full ram and state retention. After wake-up, resume execution from the point where sleep is requested, and all subsystems can run.
        - Deep sleep may not retain ram or any other state of the system (such as peripheral devices or network interfaces). fter wake-up, resume execution from the main script, similar to hard reset or power on reset. The `reset_cause()` function will return `machine.DEEPSLEEP` , which can be used to distinguish deep sleep wake-up from other resets.
    


.. function:: wake_reason()

    Return to the wake-up reason.
        
    ==================== ===============  ====================================  
    Wake-up reason       Numerial Value   Definition
    PIN_WAKE/EXT0_WAKE     2              Single RTC_GPIO wake up
    EXT1_WAKE              3              Multiple RTC_GPIO wake up
    TIMER_WAKE             4              Timer wake up
    TOUCHPAD_WAKE          5              Touchpad wake up
    ULP_WAKE               6              ULP wake up
    ==================== ===============  ====================================  



Other functions
-----------------------



.. function:: unique_id()

    Byte string that returns the unique identifier of board/ SoC. If the underlying hardware allows it, it will change from board/ SoC instance to another instance. 
    Length varies by hardware (if you need a short ID, use a substring of the full value). In some micropython ports, the ID corresponds to the network MAC address.

    >>> machine.unique_id()
    b'\xccP\xe3\x90\xeb\xd4'

.. function:: time_pulse_us(pin, pulse_level, timeout_us=1000000)

   Test the duration of the external pulse level at the given pin and return the duration of the external pulse level in microseconds. ``pulse_level`` =1 test high level duration, pulse_level=0 test low level duration. 
    When the set level is inconsistent with the current pulse level, the timing will start when the input level is consistent with the set level. If the set level is consistent with the current pulse level, the timing will start immediately. 
    When pin level and set level are always opposite, it will wait for timeout, timeout returns - 2. When the pin level and the setting level are the same all the time, it will also wait for the timeout, and the timeout will return to - 1, ``timeout_us`` is the timeout.

.. function:: rng()

    Returns a random number generated by 24 bit software.

.. _machine_constants:

Constant
---------

IRQ wake up value
^^^^^^^^

.. data:: machine.SLEEP

    2

.. data:: machine.DEEPSLEEP

    4

Restart reason
^^^^^^^

.. data:: machine.PWRON_RESET
          machine.HARD_RESET
          machine.WDT_RESET
          machine.DEEPSLEEP_RESET
          machine.SOFT_RESET


Wake up reason
^^^^^^^^

.. data:: machine.PIN_WAKE
          machine.EXT0_WAKE
          machine.EXT1_WAKE
          machine.TIMER_WAKE
          machine.TOUCHPAD_WAKE
          machine.ULP_WAKE


