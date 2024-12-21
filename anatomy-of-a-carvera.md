# Anatomy of a Carvera
<!--- TODO --->
This section provides details on _how_ the Carvera is working. 

{% hint style="info" %}
None of this info is required to be able to successfully use the machine, but you may find some of this information useful for troubleshooting, repairing, or later upgrading the machine.
{% endhint %}

## Specifications

<!--- TODO --->

## Mechanical structure

Since the Carvera arrives pre-assembled spend some time to get acquainted with the mechanical structure to fully appreciate the construction.

<!--- TODO --->

## Work area

The Carvera has a cutting area of TBA. One thing to consider is that the cutting area is somewhat larger than the bed area: 

<!--- TODO: Replace image for bed diagram --->

![](.gitbook/assets/page_11_800.png)

Some implications of the design may not be immediately apparent:

* Due to the proximity of the tool holder to the bed, part of the bed is in accessiable
<!--- TODO --->

## Stepper motors

As the name implies, stepper motors are designed to rotate by small angle increments, rather than rotating continuously as conventional motors do when they are powered. Stepper motors are ubiquitous in hobby CNCs, for a good reason: they provide the capability to move by a precise amount \(i.e. a given number of steps\), without the need for a position feedback mechanism.

The stepper motors on the Carvera are **NEMA23** \(just a fancy way to say that their front/back face size is 2.3'' square\), are of the "**bipolar**" type \(which means that they are controlled by two pairs of wires, the "A" phase and the "B" phase, and that one step is made by sending current in A and turning off B, or the other way around\), and are internally designed so that each step rotates the shaft by exactly 1.8°, which translates to 360°/1.8° = **200 steps to perform a full revolution**. 

It's the job of the **stepper driver** to generate current alternatively in the A and B phases, when instructed to move by one or more steps.

{% hint style="info" %}
The stepper drivers use a trick to optimize the torque: instead of generating a constant voltage \(and therefore current\) in a phase, they generate a higher voltage and chop it at a high frequency so that it produces the desired value on average. It's just an implementation detail, but it explains the \(normal\) humming sound that the motors emit when the machine is idle.
{% endhint %}

But it gets better: instead of just turning the current on and off completely alternatively on phases A and B to move one step at a time, if the stepper driver sends a carefully chosen variable amount of current in A and B simultaneously, then it is possible to reach \(and hold\) intermediate positions between two steps: these are called "**microsteps**", and the drivers used in the Carvera are capable of controlling **8 microsteps for each step**.

So overall, the motors can be controlled with a precision of 200 × 8 = 1600 microsteps per revolution. Combined with the ball screws, this means 0.005mm accuracy is achieved.

<!--- TODO: Discuss the closed loop functionality  --->

{% hint style="info" %}
Interestingly, stepper motors use more power when doing nothing \(_i.e._, holding a position\) than when moving. This is why they tend to get warm when the machine is on but idle. This is not a problem in itself, the motors are designed to support high temperatures but you might as well turn the machine off if it is going to stay idle for a long time, if only to save power.
{% endhint %}

It is the job of the **controller** to send commands to the stepper drivers \(which in turn will generate the right current waveforms on the motor phases\), to achieve the desired movements of the machine.

## Controller board

The brain of the Carvera is the controller board. There maybe several revisions, the one presented below is from circa 2024 \(vTBA\), the latest one may be slightly different, but fundamentals will likely be the same.

<!--- TODO: Replace Picture --->
![](.gitbook/assets/controller_overview.png)


<!--- TODO: Describe Pinout/Connectors --->


## Smoothieware Motion control software

On the Carvera, the piece of embedded software that runs in the microcontroller is a modified version of **Smoothieware**, open-source CNC motion control software \(available here: [https://github.com/Smoothieware/Smoothieware](https://github.com/Smoothieware/Smoothieware)\)

Makera has forked Smoothieware and made a number of Carvera specific modifications. The Carvera firmware is availiable here: [https://github.com/MakeraInc/CarveraFirmware](https://github.com/MakeraInc/CarveraFirmware)



What Smoothieware does is listen to incoming commands from either the Wifi Controller or USB Serial input, and act upon receiving them. Here's a very rough description of what's going on inside Smoothieware:

<!--- TODO: Is this correct? Original doc was for GRBL which Smoothieware is based. --->

![](.gitbook/assets/page_21_800_fixed.png)

* first there is a **reception buffer**, and that's a critical point because while the microcontroller can execute code with very deterministic timings, that's not the case of the host PC at the other end of the USB cable, which executes _e.g._ Carvera Controller on Windows or Mac OS X. And as everyone has experienced, from time to time Windows or MAC OS can decide to go and do something else for a little while, and this could break the flow of G-code commands into the controller. With this buffer, the control software can send a few additional G-code commands in advance of the current one, to ensure that the controller will never starve waiting for the next command from USB.
* the **G-code commands** are parsed and processed by a dedicated piece of code that generates the appropriate signals to drive the stepper driver to produce the desired motion
* the **'$' commands** are interpreted to update various internal variables
* the **realtime commands** consist of a single character, that has an immediate effect as soon as it is received, having priority over anything else Smoothieware is currently doing. For example ,  sending '!' will trigger a Feed Hold.
* Smoothieware also uses hardware input signals from the **limit switches**, from the **Carvera wireless probe**, and commands an output "**PWM**" spindle signal to drive the RPM of the spindle or laser.


Below are the three most useful GRBL commands when using the Carvera:
<!--- TODO: Are these really useful on the Carvera? --->
* **$$** prints out the current value of the settings
* **$&lt;x&gt;=&lt;val&gt;** writes value _val_ in setting _x_
* **$H** homes the machines

These commands can be typed from the G-code **console**, all G-code senders have one \(the one in Carvera Controller is called "MDI" for "Manual Data Input"\).

{% hint style="info" %}
The full set of codes supported by the machine is documented on the [Makera Wiki](https://wiki.makera.com/en/supported-codes)
{% endhint %}

## Wireless Probe

<!--- TODO --->

## Laser

<!--- TODO --->

## Spindle

<!--- TODO --->

## Vacuum

<!--- TODO --->

## 4th Axis Module

<!--- TODO --->

