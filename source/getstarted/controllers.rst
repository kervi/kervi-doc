===========
Controllers
===========

Kervi controllers reacts to one or more inputs and computes the value of one or more outputs.  
The input could come from the user via the web based UI, sensors or other application logic.

Below is the code extended with a simple fan controller that starts when the cpu temperature is over 20 degrees and reach 
max speed when the temperature is 80 degrees.


.. code:: python
  
    if __name__ == '__main__':

        from kervi.application import Application
        
        app = Application()

        #create sensors
        from kervi.sensors.sensor import Sensor
        from kervi.devices.platforms.common.sensors.cpu_use import CPULoadSensorDeviceDriver
        from kervi.devices.platforms.common.sensors.cpu_temp import CPUTempSensorDeviceDriver

        #create a senors that uses CPU load device driver
        cpu_load_sensor = Sensor("CPULoadSensor","CPU", CPULoadSensorDeviceDriver())
        
        #link to dashboard
        cpu_load_sensor.link_to_dashboard("*", "header_right")
        cpu_load_sensor.link_to_dashboard(type = "value", show_sparkline=True, link_to_header=True)
        cpu_load_sensor.link_to_dashboard(type="chart")

        #create a senors that uses CPU temp device driver
        cpu_temp_sensor = Sensor("CPUTempSensor","CPU temp", CPUTempSensorDeviceDriver())
        
        #link to dashboard
        cpu_temp_sensor.link_to_dashboard("*", "header_right")
        cpu_temp_sensor.link_to_dashboard(type = "value", show_sparkline=True, link_to_header=True)
        cpu_temp_sensor.link_to_dashboard(type="chart")

        from kervi.controllers.controller import Controller
        from kervi.values import NumberValue
        
        class FanController(Controller):
            def __init__(self):
                Controller.__init__(self, "fan_controller", "Fan")

                self.temp = self.inputs.add("temp", "Temperature", NumberValue)
                self.temp.min = 0
                self.temp.max = 150
                
                self.fan_speed = self.outputs.add("fan_speed", "Fanspeed", NumberValue)

            def input_changed(self, changed_input):
                temp = self.temp.value
                if temp <= 20:
                    self.fan_speed.value = 0
                else:
                    speed = (temp / 80) * 100
                    if speed > 100:
                        speed = 100
                    self.fan_speed.value = speed

        fan_controller = FanController()

        #link the fan controllers temp input to cpu temperature sensor
        #The temp sensor is loaded in another process and linked via its id
        fan_controller.temp.link_to(cpu_temp_sensor)

        #Show the controller input
        fan_controller.temp.link_to_dashboard()
        
        #link the controller output the UI
        fan_controller.fan_speed.link_to_dashboard()

        app.run()
