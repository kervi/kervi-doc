.. _device_sensors:

=======
Sensors
=======

Sensors in Kervi are managed via the class Sensor that handles reading from different kind of probes and devices. 

Kervi Device library
--------------------

A part of the Kervi framework is the device library that holds drivers for common devices. 
To use a device from the library you need to import the device module and apply the driver to a sensor class. 

.. code:: python

    from kervi.sensors import Sensor
    from kervi.devices.sensors import BMP085
    SENSOR_1 = Sensor("roomtemp1", "Room 1", BMP085.BMP085DeviceDriver(BMP085.TEMPERATURE_SENSOR))

Linking sensors
---------------------

A sensor is linked to a dashboard by calling the method link_to_dashboard. 

.. code-block:: python

    from kervi.sensor import Sensor
    from kervi_devices.sensors import BMP085
    SENSOR_1 = Sensor("roomtemp1", "Room 1", BMP085.BMP085DeviceDriver(BMP085.TEMPERATURE_SENSOR))
    
    SENSOR_1.link_to_dashboard("system", "cpu", type="value", link_to_header=True)
    SENSOR_1.link_to_dashboard("system", "cpu", type="chart")

Beside from linking to dashboards a sensor may also be linked to controller inputs.

.. code-block:: python

    FAN_CONTROLLER.temp.link_to(SENSOR_1)

For full information about linking take a look at the section :ref:`dynamic_values-label`.

Implementing a sensor device
----------------------------
Below is an example that shows the steps to create a sensor device. 

A sensor device inherits from either kervi.hal.SensorDeviceDriver or kervi.hal.I2CSensorDeviceDriver.
Fill in the abstract properties and the method read_value.

.. code-block:: python

    import time
    from kervi.hal import I2CSensorDeviceDriver

    class TSL2561SDeviceDriver(I2CSensorDeviceDriver):
        def __init__(self, address=0x39, bus=None):
            I2CSensorDeviceDriver.__init__(self, address, bus)
            self.gain = 0
            self.i2c.write8(0x80, 0x03)
            self.pause = 0.8
            self.gain = 0

        @property
        def device_name(self):
            return "TLS2561"
        
        @property
        def type(self):
            return "lux"

        @property
        def unit(self):
            return "LUX"

        @property
        def max(self):
            return 1000000

        @property
        def min(self):
            return 0
  
        def read_value(self):
            """Grabs a lux reading either with autoranging (gain=0) or with a specified gain (1, 16)"""
            if self.gain == 1 or self.gain == 16:
                self.set_gain(self.gain)
                ambient = self.read_full()
                ir_reading = self.read_ir()
            elif self.gain == 0:
                self.set_gain(16)
                ambient = self.read_full()
                if ambient < 65535:
                    ir_reading = self.read_ir()
                if ambient >= 65535 or ir_reading >= 65535:
                    self.set_gain(1)
                    ambient = self.read_full()
                    ir_reading = self.read_ir()

            if self.gain == 1:
                ambient *= 16
                ir_reading *= 16

            ratio = (ir_reading / float(ambient))

            if (ratio >= 0) & (ratio <= 0.52):
                lux = (0.0315 * ambient) - (0.0593 * ambient * (ratio**1.4))
            elif ratio <= 0.65:
                lux = (0.0229 * ambient) - (0.0291 * ir_reading)
            elif ratio <= 0.80:
                lux = (0.0157 * ambient) - (0.018 * ir_reading)
            elif ratio <= 1.3:
                lux = (0.00338 * ambient) - (0.0026 * ir_reading)
            elif ratio > 1.3:
                lux = 0

            return lux
        
        #private methods use in the driver

        def set_gain(self, gain=1):
            """ Set the gain """
            if gain == 1:
                self.i2c.write8(0x81, 0x02)
            else:
                self.i2c.write8(0x81, 0x12)

            time.sleep(self.pause)
        
        def read_word(self, reg):
            try:
                wordval = self.i2c.read_U16(reg)
                newval = self.i2c.reverse_byte_order(wordval)
                return newval
            except IOError:
                print("Error accessing 0x%02X: Check your I2C address" % self.i2c.address)
                return -1


        def read_full(self, reg=0x8C):
            """Reads visible+IR diode from the I2C device"""
            return self.read_word(reg)

        def read_ir(self, reg=0x8E):
            """Reads IR only diode from the I2C device"""
            return self.read_word(reg)