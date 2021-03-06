=====
Kervi
=====
  
.. image:: images/kervi_frontx.png

Easy Python framework for robotic and automation projects.
The framework runs on all platforms that supports Python and have hardware support for Raspberry pi.
Wire up sensors, controllers and other devices to your Raspberry PI and link them to web based dashboards
and internal application logic.

Knowledge about web servers, html or web programming is not needed.
UI configuration is done in python code and the framework creates the web ui.

Features of the framework are:

* Python classes for handling sensors, displays, motors, gpio and other hardware.
* Python classes for handling user interactions from controllers on dashboards.
* Integrated camera handling (Raspberry PI camera at the moment)
* Device driver library for common devices 
* Very easy dashboard setup.
* Dashboard UI is responsive and scales from mobile phones to desktop.
* Multi process oriented by spreading over more than one core.
* Multi device oriented by connecting multiple Raspberry Pi's in one large application. 
* No web development knowledge is required as dashboard layout is handled in python. 

Below is a complete example that shows how to display a sensor on a dashboard with a chart

.. code-block:: python

    if __name__ == '__main__':
        from kervi.application import Application
        APP = Application()
        
        from kervi.sensor import Sensor
        from kervi.devices.sensors.system import CPULoadSensorDeviceDriver
        
        cpu_sensor = Sensor("CPULoadSensor","CPU", CPULoadSensorDeviceDriver())
        cpu_sensor.link_to_dashboard(link_to_header=True)
        cpu_sensor.link_to_dashboard(type="chart")

        APP.run()

.. toctree::
   :maxdepth: 4
   :includehidden:
   :caption: Contents:
   :hidden:

   install
   getstarted/index
   dashboards
   sensors
   controllers
   signals
   actions
   camera
   motors
   gpio
   display
   messaging
   configuration
   security
   hal
   devices/index
   multi_process
   distributed
   
   service
   cli
   ipc
   plugins/index
   kervi_api
   contribute
   
