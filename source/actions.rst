.. _actions:
==============
Actions
==============

Actions are functions that may be linked to dashboards, sensors, GPIO and other kinds of inputs.
Actions are callable from anywhere in your kervi application even if they resides in another process 
the framework handles process and network boundaries for you.

Define actions
--------------

You turn a function into an action via the @action decorator

.. code:: python

    from kervi.actions import action

    @action
    def my_action(p):
        print("my action", p)

You may also decorate methods in kervi controllers:

.. code:: python

    from kervi.controllers import controller
    from kervi.actions import action
    
    class GateController(Controller):
        def __init__(self, controller_id="gate_controller", name="Gate controller"):
            super().__init__(controller_id, name)

        @action()
        def open(self, speed):
            print("open gate with speed:", speed)


Execute actions
---------------

You execute an action just as an normal function or method

.. code:: python

    my_action("p1")

    gate_controller = GateController()
    gate_controller.open(10)

Actions with timeout

By default, an action is called synchronous and first returns with the result when the action is completed.
If you expect your action to operate within a special time interval, you can add the keyword parameter timeout to
the action call. If the execution time exceeds the timeout, a TimeoutError exception is raised.

.. code:: python

    @action
    def my_action():
        print("started")
        time.sleep(10)
        print("done")

    try:
        my_action(timeout=5)
    except TimeoutError:
        print("Timout occured")

Asynchronous action call
------------------------

It is possible to call an action asynchronously if you don't want to wait for the action to finish execution.
Just set the keyword argument run_async to true.

.. code:: python

    my_action(run_async=True)


Interrupts
---------

Sometimes you need to signal a running action during execution. 
To handle this situation you need to specify an interrupt function for an action.

.. code:: python

    terminate = False
    @action
    def my_action():
        print("my_action start")
        
        while not terminate:
            time.sleep(.1)

        print("my_action done")

    @my_action.set_interrupt
    def my_action_interrupt():
        global terminate
        terminate = True
        print("interrupt my_action")

    
    #call action
    my_action(run_async=True)

    #wait for five seconds
    time.sleep(5)

    #signal that the action should terminate
    my_action.interrupt() 


Interrupts support parameters

.. code:: python

    @my_action.set_interrupt
    def my_action_interrupt(p1):
        global terminate
        terminate = True
        print("interrupt my_action:", p1)

    
    #call action
    my_action(run_async=True)

    #wait for five seconds
    time.sleep(5)

    #signal that the action should terminate
    my_action.interrupt("p !")

The action decorator injects a variable "exit_action" that is false until the action is interrupted.

.. code:: python
    
    @action
    def my_action(p1="p1d", **kwargs):
        print("action", p1, kwargs.get("kw1", None))
        while not exit_action:
            print("in loop")
            time.sleep(1)
        print("action interrupted")

    
    my_action("p1x", kw1=10, run_async=True)
    #wait for five seconds
    time.sleep(5)

    #signal that the action should terminate
    my_action.interrupt()

Scheduling
----------

It is possible to schedule when an action should run

.. code:: python

    @action
    def my_action(p1):
        print("My action", p1)
        
    my_action.run_every().minute.do("P1")
    my_action.run_every(10).minutes.do("P1")
    my_action.run_every().minute.at(":17").do("P1")
    
    my_action.run_every().hour.do("P1")

    my_action.run_every().day.do("P1")
    my_action.run_every().day.at("10:30").do("P1")
    
    
    my_actionrun_every().monday.do("P1")
    my_action.run_every().wednesday.at("13:15").do("P1")

It is also possible to run an action in an time interval.

.. code:: python
    
    @action
    def my_action(p1="p1d", **kwargs):
        print("action", p1, kwargs.get("kw1", None))
        while not exit_action:
            print("in loop")
            time.sleep(1)
        print("action interrupted")

    my_action.run_every().minute.at(":58").until(":02").do("P!x", kw1=20)
    my_action.run_every().wednesday.at("13:15").do("P1")

Linking to dashboards
---------------------

It is possible to link actions to dashboards.
A linked action will show up as a button on the panel.

.. code:: python

    @action
    def my_action():
        print("this is my action")

    my_action.link_to_dashboard("app", "gate")
    
You can send parameters to the action.

.. code:: python 

    from kervi.actions import action

    @action(name="My action")
    def my_action(p):
        print("my_action is called with:", p)

    my_action.link_to_dashboard("app", "gate", action_parameters=["x"])


If an interrupt function is set for the action it will be called when the button is released. 

.. code:: python 

    from kervi.actions import action

    @action(name="My action")
    def my_action(p):
        print("my_action is called with:", p)

    @my_action.set_interrupt
    def my_action_interrupt():
        print("my_action interrupt called")

    my_action.link_to_dashboard("app", "gate", action_parameters=["x"])

You are able to specify parameters that should be send in the interrupt.

.. code:: python 

    from kervi.actions import action

    @action(name="My action")
    def my_action(p):
        print("my_action is called with:", p)

    @my_action.set_interrupt
    def my_action_interrupt(p):
        print("my_action interrupt called: ", p)

    my_action.link_to_dashboard("app", "gate", action_parameters=["x"], interrupt_parameters=["i"])

Other keyword parameters you can use in link_to_dashboard:

    * *link_to_header* (``str``) -- Link this action to the header of the panel.

    * *label_icon* (``str``) -- Icon that should be displayed together with label.

    * *label* (``str``) -- Label text, default value is the name of the action.

    * *flat* (``bool``) -- Flat look and feel.

    * *inline* (``bool``) -- 
        Display button and label in its actual size
        If you set inline to true the size parameter is ignored.
        The action will only occupy as much space as the label and input takes.

    
    * *on_text* (``string``) -- Text to display when switch is on.
    * *off_text* (``string``) -- Text to display when switch is off.
    * *on_icon* (``string``) -- Icon to display when switch is on.
    * *off_icon* (``string``) -- Icon to display when switch is off.

    * *button_icon* (``string``) -- Icon to display on button.
    * *button_text* (``string``) -- Text to display on button, default is name.

    * *action_parameters* (``list``) -- list of parameters to pass to the action.

    * *interrupt_enabled* (``bool``) -- If true the button will send interrupt to action on off. Default true if an interrupt is specified for the action.
    * *interrupt_parameters* (``list``) -- List of parameters to pass to the interrupt function of the action.


Linking to values
-----------------

It is possible to link an action to gpio or sensors or other kervi values.
When the value of linked source changes the Action is executed or interrupted.

.. code:: python

    my_action.link_to(GPIO["GPIO2"])

In the code above the action is linked to gpio2 if it goes high it executes the action.
When the gpio2 goes low the action is interrupted.

Here is another example where the action is linked to a sensors.
When the sensor value is 10 the action is executed.

.. code:: python  

    my_action.link_to(
        temp_sensor,
        trigger_value = 10
    )

You can also use an lambda expression as trigger.

.. code:: python

    my_action.link_to(
        temp_sensor,
        trigger_value = lambda x: x > 10
    )

If you want to pass the value of the linked source you can do by setting the pass_value parameter
Now the action is called every time the source changes.

.. code:: python

    my_action.link_to(
        temp_sensor,
        pass_value = true
    )

It is also possible to pass additional parameters to the action when it is triggered.

.. code:: python

    my_action.link_to(
        temp_sensor,
        trigger_value = lambda x: x > 10,
        action_parameters = ["20", 30]
    )

You can also specify when the interrupt should fire.

.. code:: python

    my_action.link_to(
        temp_sensor,
        trigger_value = lambda x: x > 10,
        action_parameters = ["20", 30]
        trigger_interrupt_value: lambda x: x < 5,
        interrupt_parameters = [0]
    )



System actions
--------------

When the kervi application has loaded and started all processes it calls the app_main action this is your hook where you can
start your application logic. In the same way app_exit action is called upon termination of the kervi application.

It is optional for you to define these actions in your application.

.. code:: python

    @action
    def app_main():
        #start your application logic here


    @action
    def app_exit():
        #application logic that clean up and reset devices

You can control the application via the following actions

.. code:: python

    #stop the application
    Actions["app.stop"]

    #restart the application
    Actions["app.restart"]

    #shut down application device (Raspberry pi)
    Actions["app.shutdown"]

    #reboot application device (Raspberry pi)
    Actions["app.reboot"]


You can connect a battery sensor to the *shutdown action* and initiate a shutdown of your device when the battery running low.

.. code:: python

    from kervi.application import Application

    APP = Application()
    
    from kervi.sensors import Sensor
    from kervi.devices.sensors.CW2015 import CW2015CapacityDeviceDriver

    capacity_sensor = Sensor("CW2015_capacity", "CW2015 capacity", CW2015CapacityDeviceDriver())
    
    #link sensor to action.
    APP.actions.shutdown.link_to(capacity_sensor, trigger_value=lambda x: x<10)

    #display a battery icon in the header of your application. 
    capacity_sensor.set_ui_parameter("value_icon", [
        {
            "range":[0, 5],
            "icon":"battery-empty"
        },
        {
            "range":[5, 25],
            "icon":"battery-quarter"
        },
        {
            "range":[20, 50],
            "icon":"battery-half"
        },
        {
            "range":[5, 75],
            "icon":"battery-three-quarters"
        },
        {
            "range":[75, 100],
            "icon":"battery-full"
        }
    ])
    capacity_sensor.link_to_dashboard("*", "header_right", display_unit=False, show_sparkline=False, show_value=False)

    APP.run()

Complete example
-----------------

This is a complete example that shows a gate controller that controls a motor and have two end stop switches.
The end stops are linked to GPIO2 and GPIO3. 

There are to two actions move_gate and stop_gate these are linked to the "gate" panel on the app dashboard.
The move_gate action is also linked GPIO4 and GPIO5. When GPIO4 is triggered the gate opens and closes when gpio5 is triggered.

.. code:: python

    if __name__ == '__main__':
        import time
        from kervi.application import Application

        APP = Application()
        
        from kervi.dashboards import Dashboard, DashboardPanel
        Dashboard(
            "app",
            "App",
            [
                DashboardPanel("gate", title="Gate")
            ],
            is_default=True
        )

        Dashboard(
            "settings",
            "Settings",
            [
                DashboardPanel("gate", width="200px", title="Gate")
            ]
        )

        from kervi.controllers import Controller
        from kervi.actions import action
        from kervi.values import NumberValue, BooleanValue
        
        class GateController(Controller):
            def __init__(self, controller_id="gate_controller", name="Gate controller"):
                super().__init__(controller_id, name)

                self.gate_speed = self.inputs.add("speed", "Gate speed", NumberValue)
                self.gate_speed.value = 100
                self.gate_speed.min = 0
                self.gate_speed.persist_value = True

                self.lo_end_stop = self.inputs.add("lo_end_stop", "low end stop", BooleanValue)
                self.hi_end_stop = self.inputs.add("hi_end_stop", "High end stop", BooleanValue)

                self.gate_motor_speed = self.outputs.add("gate_motor_speed", "Gate motor speed", NumberValue)

                self._stop_move = False

            @action(name="Move gate")
            def move_gate(self, open=True):
                if open:
                    print("open gate")
                    if not self.hi_end_stop.value:
                        self._stop_move = False
                        self.gate_motor_speed.value = self.gate_speed.value
                        while not self._stop_move and not self.hi_end_stop.value:
                            time.sleep(0.1)
                        self.gate_motor_speed.value = 0
                    if self.hi_end_stop.value:
                        print("Gate open")
                    else:
                        print("Gate stopped")
                else:
                    print("close gate:")
                    if not self.lo_end_stop.value:
                        self._stop_move = False
                        self.gate_motor_speed.value = -1 * self.gate_speed.value
                        while not self._stop_move and not self.lo_end_stop.value:
                            time.sleep(0.1)
                        self.gate_motor_speed.value = 0
                    if self.lo_end_stop.value:
                        print("Gate closed")
                    else:
                        print("Gate stopped")

            @move_gate.set_interrupt
            def move_gate_interrupt(self):
                print("stop gate:")
                self._stop_move = True

            def controller_start(self):
                print("gate controller is started")
                self.gate_motor_speed.value = 0

            def input_changed(self, changed_input):
                pass

        gate_controller = GateController()
        gate_controller.move_gate.link_to_dashboard("app", "gate", inline=True, button_text=None, button_icon="arrow-up", label=None, action_parameters=[True], )
        gate_controller.move_gate.link_to_dashboard("app", "gate", inline=True, button_text=None, button_icon="arrow-down", label=None, action_parameters=[False])
        
        gate_controller.link_to_dashboard("settings", "gate")

        from kervi.devices.motors.dummy_motor_driver import DummyMotorBoard

        motor_board = DummyMotorBoard()
        gate_controller.gate_motor_speed.link_to(motor_board.dc_motors[0].speed)

        from kervi.hal import GPIO
        GPIO["GPIO2"].define_as_input()
        GPIO["GPIO3"].define_as_input()

        gate_controller.lo_end_stop.link_to(GPIO["GPIO2"])
        gate_controller.hi_end_stop.link_to(GPIO["GPIO3"])

        gate_controller.move_gate.link_to(GPIO["GPIO4"], action_parameters = [True])
        gate_controller.move_gate.link_to(GPIO["GPIO5"], action_parameters = [False])

        APP.run()


The result is an app with two dashboards "app" where the gate is controlled and "settings" where the speed 
of the gate speed could be set.  

.. image:: images/gate_controller.png
    :width: 35%
.. image:: images/gate_controller_settings.png
    :width: 35%

Multi process
------------

If you want to call an action that is defined in another process within your kervi application you use the Actions list.

.. code:: python

    from kervi.actions import Actions

    Actions["my_action"]("x")


The @action decorator takes the optional parameters action_id.
.. code:: python

    from kervi.actions import action

    @action(action_id="alternative_id")
    def my_action(p):
        print("my action", p)

You now call it via:

.. code:: python

    from kervi.actions import Actions

    Actions["alternative_id"]("x")


This example shows how to set up at simple robot and control it via an external script of commands.

It consists of two scripts robot.py that should be executed on the robot 
and a script robot_task.py that holds a series of actions that the robot should perform.

robot.py is a kervi application that you should execute on your Raspberry pi.

.. code:: python
    
    if __name__ == '__main__':
        from kervi.application import Application

        APP = Application()

        from kervi.controllers.steering import MotorSteering
        from kervi.devices.motors.dummy_motor_driver import DummyMotorBoard

        motor_board = DummyMotorBoard()
        
        #MotorSteering is a build controller for handling robots with two motors.
        steering = MotorSteering()
        
        steering.left_speed.link_to(motor_board.dc_motors[0])
        steering.right_speed.link_to(motor_board.dc_motors[1])

        @action
        def app_exit():
            steering.stop()

        APP.run()

The other python script robot_tasks.py is a kervi module that connects to the
kervi application in robot.py. You can run it on the robot itself or another computer
on your local network.

.. code:: python

    if __name__ == '__main__':
        from kervi.module import Module
        from kervi.actions import action, Actions
        module = Module()
        
        
        @action
        def module_main():
            #move for 5 seconds at 100% speed
            Actions["steering.move"](100, duration=5)
            #turn right at 50% speed
            Actions["steering.rotate"](50, degree=90)

        module.run()