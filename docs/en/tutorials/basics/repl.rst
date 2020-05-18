REPL
=====

One of the main advantages of using micropython is the interactive REPL，REPL（read-evaluate-print loop）.
REPL is very helpful for learning a new programming language, as it can respond to programs written by beginners immediately, it means that you can execute the code and view the results immediately without the tedious steps of compiling and uploading first.
If the MPython Board wants REPL to work on windows, you need to install the cp2104 serial driver first.


Serial Connection
----------

To access through USB serial, you need to use serial terminal software. On windows, kitty and xshell are good choices. When the serial port baud rate is set to 115200, you can start playing MicroPython.

After the connection is established through the serial port, you can test whether it works normally by pressing enter several times. If it works normally, you can see the Python REPL prompt, which is expressed as  ``>>>`` .

使用 REPL
----------

一旦有提示，您就可以开始尝试了！按Enter键后，可在提示符处键入任何内容。
MicroPython将运行您输入的代码并打印结果（如果有的话）；如果输入的文本出错，则会打印出错误消息。

尝试在提示符下输入以下内容::

    >>> print('hello mPython')
    hello mPython


请注意，您无需键入 ``>>>`` 箭头，它们表示您应在此提示符后键入文本，其下一行是响应的内容。

如果您已经了解了一些python，现在可以尝试一些基本命令。例如::

    >>> 1+2
    3
    >>> 1/2
    0.5
    >>> 12*34
    408


可以尝试下载mPython的OLED显示屏上显示字符::

    >>> from mpython import *
    >>> oled.DispChar('hello,world!',0,0)
    >>> oled.show()
    >>> 

.. Note::

    ``oled.DispChar(str,x,y)``   ``str`` 为要显示的字符串， ``x`` 、``y`` 为显示起点的x、y坐标。
    然后用 ``oled.show()`` 刷新屏幕后，字符串即可显示在OLED显示屏上。您可以尝试在其他位置显示任意字符串。



行编辑
~~~~~~~~~~~~

您可以使用向左和向右箭头键移动光标来编辑当前输入的行；按Home键或ctrl-A将光标移动到行的开头，按End或ctrl-E移动到行的末尾；Delete键或退格键用来删除。

输入历史记录
~~~~~~~~~~~~~

REPL会记住您输入的一定数量的前几行文本（ESP32上最多8行）。
要调用上一行，请使用向上和向下箭头键。

Tab键
~~~~~~~~~~~~~~

Tab键可以查看模块中所有成员列表。这对于找出模块或对象具有的函数和方法非常有用。
假设您在上面的例子中导入了machine然后键入 ``.`` 再按Tab键以查看machine模块所有成员列表::

    >>> machine.
    __class__       __name__        ADC             DAC
    DEEPSLEEP       DEEPSLEEP_RESET                 EXT0_WAKE
    EXT1_WAKE       HARD_RESET      I2C             PIN_WAKE
    PWM             PWRON_RESET     Pin             RTC
    SLEEP           SOFT_RESET      SPI             Signal
    TIMER_WAKE      TOUCHPAD_WAKE   Timer           TouchPad
    UART            ULP_WAKE        WDT             WDT_RESET
    deepsleep       disable_irq     enable_irq      freq
    idle            mem16           mem32           mem8
    reset           reset_cause     sleep           time_pulse_us
    unique_id       wake_reason
    >>> machine.


行继续和自动缩进
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

您键入的某些内容将需要“继续”，也就是说，需要更多行文本才能生成正确的Python语句。在这种情况下，
提示符将更改为``...``并且光标将自动缩进正确的数量，以便您可以立即开始键入下一行。
通过定义以下函数来尝试此操作::


    >>> def toggle(p):
    ...    p.value(not p.value())
    ...    
    ...    
    ...    
    >>>

在上面，您需要连续按三次Enter键才能完成复合语句（即三条线上只有点）。完成复合语句的另一种方法是按退格键到达行的开头，然后按Enter键。 （如果您输错了并且想要退出，那么按ctrl-C，所有行都将被忽略。）

您刚刚定义函数功能为翻转引脚电平。您之前创建的pin对象应该仍然存在
（如果没有，则需重新创建它），您可以使用以下命令翻转LED::

    >>> toggle(pin)

现在让我们在一个循环中翻转LED（如果您没有LED，那么您可以打印一些文本而不是调用切换，看看效果）：

    >>> import time
    >>> while True:
    ...     toggle(pin)
    ...     time.sleep_ms(500)
    ...    
    ...    
    ...    
    >>>

这将以1Hz（半秒开，半秒关）翻转LED。要停止切换按 ``ctrl-C`` ，这将引发键盘中断异常并退出循环。


粘贴模式
~~~~~~~~~~

按 ``ctrl-E`` 将进入特殊粘贴模式，您可将一大块文本复制并粘贴到REPL中。如果按ctrl-E，您将看到粘贴模式提示::

    paste mode; Ctrl-C to cancel, Ctrl-D to finish
    === 

然后，您可以粘贴（或键入）您的文本。请注意，没有任何特殊键或命令在粘贴模式下工作（例如Tab或退格）
，它们只是按原样接受。按 ``ctrl-D`` 完成输入文本并执行。

其他控制命令
~~~~~~~~~~~~~~~~~~~~~~

还有其他四个控制命令：

* 空白行上的Ctrl-A将进入原始REPL模式。这类似于永久粘贴模式，除了不回显字符。

* 空白处的Ctrl-B转到正常的REPL模式。

* ``Ctrl-C`` 取消任何输入，或中断当前运行的代码。

* 空白行上的 ``Ctrl-D`` 将执行软重启。


