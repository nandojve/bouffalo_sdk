I2C - 10-bit
====================

本 demo 主要介绍 I2C 10-bit slave 模式数据传输。

硬件连接
-----------------------------

本 demo 使用到的 gpio 参考 ``board_i2c0_gpio_init`` ，将 USB 转 I2C 模块与开发板连接，具体引脚连接方式如下表（以BL616为例）：

.. table:: 硬件连接
    :widths: 50, 50
    :width: 80%
    :align: center

    +-------------------+------------------+
    | 开发板 I2C 引脚   | USB 转 I2C 模块  |
    +===================+==================+
    | SCL(GPIO14)       | SCL              |
    +-------------------+------------------+
    | SDA(GPIO15)       | SDA              |
    +-------------------+------------------+

软件实现
-----------------------------

更详细的代码请参考 **examples/peripherals/i2c/i2c_10_bit**

.. code-block:: C
    :linenos:

    board_init();

- ``board_init`` 中会开启 I2C IP 时钟，并选择 I2C 时钟源和分频。

.. code-block:: C
    :linenos:

    board_i2c0_gpio_init();

- 配置相关引脚为 `I2C` 功能

.. code-block:: C
    :linenos:

    i2c0 = bflb_device_get_by_name("i2c0");

    bflb_i2c_init(i2c0, 400000);

- 获取 `i2c0` 句柄，并初始化 i2c0 频率为 400K

.. code-block:: C
    :linenos:

    struct bflb_i2c_msg_s msgs[2];
    uint8_t subaddr[2] = { 0x00, 0x04};
    uint8_t write_data[I2C_10BIT_TRANSFER_LENGTH];

    /* Write buffer init */
    write_data[0] = 0x55;
    write_data[1] = 0x11;
    write_data[2] = 0x22;
    for (size_t i = 3; i < I2C_10BIT_TRANSFER_LENGTH; i++) {
        write_data[i] = i;
    }

    /* Write data */
    msgs[0].addr = I2C_10BIT_SLAVE_ADDR;
    msgs[0].flags = I2C_M_NOSTOP | I2C_M_TEN;
    msgs[0].buffer = subaddr;
    msgs[0].length = 2;

    msgs[1].addr = I2C_10BIT_SLAVE_ADDR;
    msgs[1].flags = 0;
    msgs[1].buffer = write_data;
    msgs[1].length = I2C_10BIT_TRANSFER_LENGTH;

    bflb_i2c_transfer(i2c0, msgs, 2);

- 初始化发送数据(write_data)和配置从设备信息(msgs)
- ``bflb_i2c_transfer(i2c0, msgs, 2)`` 开启 i2c 传输

编译和烧录
-----------------------------

参考 :ref:`get_started`

实验现象
-----------------------------

通过串口（波特率大于115200）发送``04 00 06 01 03 55``命令给 USB 转 I2C 模块，设置 I2C 从机 10-bit 模式数据传输。
按下开发板中 RST 按键，串口打印开发板发送的 write_data 数据。
