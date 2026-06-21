# Intelligent-control
This is for project II, ECTE441, Intelligent control, University of Wollongong

My Unserstanding

The objective of this project is to design an intelligent controller for a mobile robot so that it can navigate through a 2D map with obstacle regions and reach the target destination safely. The robot must avoid unsafe obstacle areas while moving toward the target, without using coordinate-based path planning.

In this project, I designed a TSK/Sugeno fuzzy controller to control the robot. The robot block has two control inputs: the left wheel speed and the right wheel speed. It also provides five output signals, including the odometer, distance to target, angle to target, obstacle location, and obstacle distance signal.

The fuzzy controller uses selected sensor outputs from the robot block, mainly the angle to target, obstacle location, and ultrasonic obstacle distance signal, to generate a steering control signal. This signal is then used to adjust the speed difference between the left and right wheels, allowing the robot to turn left, turn right, move straight, and avoid obstacles.

One of the main difficulties in this project is understanding the real range and meaning of the sensor parameters. Although the project instruction provides the general range of each parameter, these values should not be treated as simple linear values. For example, the ultrasonic signal does not increase smoothly from zero as the robot approaches an obstacle. Therefore, display blocks were added in Simulink to observe the actual sensor output values during simulation. This helped to understand how close the robot was to obstacles and to design more suitable membership functions for the fuzzy controller.

This project also includes simulation testing under different initial robot angles to evaluate whether the controller can guide the robot to the target while avoiding obstacle collisions.

Project Objective

The robot starts from the required initial position:

X = 3.4
Y = 5.8

The goal is to guide the robot to the estimated target location while avoiding irregular polygon obstacle areas.
The controller does not use coordinate-based path planning. Instead, it only uses the sensor outputs from the robot block to make real-time control decisions.

The main control objectives are:

Move the robot toward the target.
Avoid obstacles detected by the robot sensors.
Stop the robot when it is close enough to the target.
Collect input-output data from the fuzzy controller for ANFIS training.
Robot Block

The robot block has two input signals:

Input	Description
vl	Left wheel speed
vr	Right wheel speed

The valid range for both wheel speeds is:

-2 <= vl, vr <= 2

The robot block provides five output signals:

Output	Description
distance to target	Estimated distance between the robot and target
odometer	Accumulated travelled distance
ultrasonic	Nonlinear front obstacle distance signal
theta	Angle between robot heading and target bearing
ob_loc / activeSensor	Obstacle location indicator
Fuzzy Controller Design

The fuzzy controller uses three robot sensor outputs as inputs:

theta
ob_loc
ultrasonic

The controller generates one output signal:

x = steering control signal

This output is combined with a constant base speed of 1.5 to generate the final wheel speeds:

Left wheel speed  = 1.5 + x
Right wheel speed = 1.5 - x

The steering logic is:

If x < 0, the right wheel is faster than the left wheel, so the robot turns left.
If x > 0, the left wheel is faster than the right wheel, so the robot turns right.
If x = 0, both wheels have the same speed, so the robot moves straight.

A switch block is also used with the distance to target signal.
When the robot is close enough to the target, both wheel speeds are set to zero:

If distance to target < 0.05:
    vl = 0
    vr = 0

This allows the robot to stop automatically near the destination.

Sensor Interpretation

A key difficulty in this project is that the sensor values should not be treated as simple linear values.
During testing, I added display blocks in Simulink to observe the real output values from the robot block. This helped me tune the membership functions more accurately.

Theta

theta represents the angle between the robot heading direction and the target direction.

In this controller:

Theta range	Meaning
Negative theta	Target is on the left side
Around zero	Target is approximately in front of the robot
Positive theta	Target is on the right side
Obstacle Location: ob_loc

The ob_loc signal indicates where the nearest obstacle is detected:

ob_loc value	Meaning
-9	No obstacle detected
-5	Obstacle on the left side
-2.5	Obstacle on the left/front-left side
0	Obstacle in the front/middle
2.5	Obstacle on the right/front-right side
5	Obstacle on the right side
Ultrasonic

The ultrasonic signal is nonlinear.
It does not increase smoothly from zero in all cases.

From my simulation observation:

0 means no obstacle is detected.
The first detected obstacle value is usually around 14.
Values around 14–20 mean the obstacle is still relatively far.
Values above 20 mean the robot is getting close to the obstacle.
Values around 35 are already very close to the obstacle boundary.
Collision cases may produce larger values, often around 50–80.

Because of this nonlinear behaviour, I did not divide the ultrasonic range evenly.
Instead, I used three practical membership functions:

None
Faraway
Near

This made the fuzzy controller more suitable for the actual robot sensor behaviour.

Fuzzy Rule Design

The fuzzy controller uses 35 rules to handle different combinations of:

theta
ob_loc
ultrasonic

The rules are designed to balance two behaviours:

Moving toward the target.
Avoiding obstacles when they are detected.

For example:

If no obstacle is detected, the robot mainly follows the target direction using theta.
If an obstacle is detected in front of the robot and the ultrasonic value is high, the robot turns away.
If the obstacle is on the left or right side, the controller adjusts the wheel speeds to avoid moving into the obstacle.
After avoiding the obstacle, the robot turns back toward the target.
ANFIS Controller

For Task 2, an ANFIS controller was trained using the data collected from the Sugeno fuzzy controller.

The ANFIS training data includes:

theta
ob_loc
ultrasonic
Sugeno controller output

The ANFIS model was generated using grid partition with the same input membership function numbers as the original fuzzy controller:

theta:      5 MFs
ob_loc:     6 MFs
ultrasonic: 3 MFs

This creates:

5 × 6 × 3 = 90 rules

The ANFIS model was trained for 50 epochs using the hybrid training method.
After training, the ANFIS output was able to approximate the original Sugeno fuzzy controller output.

Simulation Results

The fuzzy controller was tested using six different initial heading angles while keeping the same starting position.

Case	Initial angle	Odometer	Time lasted

Case 1	3.140	11840	185 s    
Case 2	1.507	12180	188.05 s    
Case 3	-0.126	16210	231.95 s    
Case 4	-0.879	8877	161.55 s    
Case 5	-2.072	12830	198.95 s    
Case 6	-2.952	11630	181.5 s

All six test cases successfully reached the target without entering the obstacle regions.

Case 4 produced the shortest travelled distance and shortest completion time.
Case 3 produced the longest route, showing that the initial heading angle can affect the path and navigation efficiency.

Important Observation About Odometer

During testing, I found that the odometer value may continue increasing even after the wheel speed inputs are cut off by the switch block.

Therefore, when recording the final odometer value, the simulation should be stopped manually after the robot reaches the target. Otherwise, the odometer value may not accurately represent the actual travelled distance before reaching the target.
