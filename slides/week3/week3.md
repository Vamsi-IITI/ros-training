---
title: Week 3
revealOptions:
    width: 1600
    height: 900 
---
# Week 3

---

## Recap of last week

----

#### Creating a ROS Node

<pre><code>ros::init(argc, argv, "NODE_NAME")</code></pre> <!-- .element: class="fragment" data-fragment-index="1" --> 

----

#### Writing A ROS Publisher
4 steps:

<ol>
<li>Need a <code>ros::NodeHandle</code></li> <!-- .element: class="fragment" data-fragment-index="0" --> 
</ol>
<pre><code class="c++">ros:NodeHandle nh;</code></pre> <!-- .element: class="fragment" data-fragment-index="1" --> 

<ol start="2">
<li>Need a <code>ros::Publisher</code></li> <!-- .element: class="fragment" data-fragment-index="2" --> 
</ol>
<pre><code class="c++">ros::Publisher publisher = nh.advertise&lt;MESSAGE_TYPE&gt;(TOPIC, QUEUE_SIZE)</code></pre> <!-- .element: class="fragment" data-fragment-index="3" --> 

<ol start="3">
<li>Create the message</li> <!-- .element: class="fragment" data-fragment-index="4" --> 
</ol>
<pre><code class="c++">std_msgs::Int32 msg;</code></pre> <!-- .element: class="fragment" data-fragment-index="5" --> 

<ol start="4">
<li>Publish the message</li> <!-- .element: class="fragment" data-fragment-index="6" --> 
</ol>
<pre><code class="c++">publish.publish(message)</code></pre> <!-- .element: class="fragment" data-fragment-index="7" --> 

----

#### Writing A ROS Subscriber
4 steps:

<div>
<ol>
<li>Need a <code>ros::NodeHandle</code></li> <!-- .element: class="fragment" data-fragment-index="0" --> 
</ol>
<pre><code class="c++">ros:NodeHandle nh;</code></pre> <!-- .element: class="fragment" data-fragment-index="1" --> 
</div>

<div>
<ol start="2">
<li>Need a <code>ros::Subscriber</code></li> <!-- .element: class="fragment" data-fragment-index="2" --> 
<pre><code class="c++">ros::Subscriber subscriber = nh.subscribe(TOPIC, QUEUE_SIZE, CALLBACK)</code></pre> <!-- .element: class="fragment" data-fragment-index="3" --> 
<ul>
<li>What is a callback function (For ROS subscribers)?</li> <!-- .element: class="fragment" data-fragment-index="4" --> 
<ul>
<li>A function that is called when a new message is received</li> <!-- .element: class="fragment" data-fragment-index="5" --> 
</ul>
</ul>
</ol>
</div>

<div>
<ol start="3">
<li>Create the callback function</li> <!-- .element: class="fragment" data-fragment-index="6" --> 
</ol>
<pre><code class="c++">void callbackFunction(std_msgs::Int32 msg) {
  // Stuff
}</code></pre> <!-- .element: class="fragment" data-fragment-index="7" --> 
</div>

<ol start="4">
<li><pre>ros::spin()</pre></li> <!-- .element: class="fragment" data-fragment-index="8" --> 
</ol>
<div><pre style="display: inline">ros::spin()</pre> <span>after creating the subscriber</span></div> <!-- .element: class="fragment" data-fragment-index="9" -->

---

Welcome to Week 3 of ROS training exercises!

- PID
- Launch files
- Play with the simulator

---

## Control Theory and PID
Need to understand control theory to learn about PID

---

### What is Control Theory
- Control a **system** to do what we want

----

#### Example

- Task: stop a car in front of the stop sign

How do you do that?

----

- Gauge the distance, or **error**, between current position and where you're trying to stop
- Use that information to decide how much throttle

----

- **Controller** determines how much throttle
- Goal: **Process variable (PV)** equal to **set point (SP)** by changing the **control**

----
#### Back to car example

- **Process Variable (PV)**: Car's position
- **Set Point (SP)**: Position right in front of the stop sign
- **Control**: Throttle

---

### The PID algorithm - Proportional Controller

- PID is one example of a **controller**
    - Simple but effective
- Three main parts

----

Intuition:
- Larger error → Step on throttle more
- Smaller error → Step on throttle less
- No error → Don't step on throttle
    
----
#### Proportional controller

**control** is **proportional** to the **error**

<p align="center">
    <img class="eqn" src="https://latex.codecogs.com/svg.latex?P_\text{out}&space;=&space;K_\text{p}&space;e(t)"/>
</p>

---
### Exercise: Implementing a Proportional controller in ROS
- Let's implement it in ROS
- Test using the `buzzsim` simulator

----
#### Launching buzzsim with `roslaunch`
- This time we'll use `roslaunch` to launch the simulator
```bash
roslaunch igvc_training_exercises week3.launch
```

- Syntax quite similar to `rosrun`
    - `roslaunch PACKAGE_NAME LAUNCH_FILE`
- Instead of `buzzsim`, there's a `week3.launch`

---
#### `.launch` files
- File located in the [igvc_training_exercises/launch](../igvc_training_exercises/launch) folder

```launch
<launch>
    <node pkg="igvc_buzzsim" type="buzzsim" name="buzzsim">
        <param name="config_path" value="$(find igvc_training_exercises)/config/week3/world.yml" />
        <param name="world_name" value="stationary" />
<!--        <param name="world_name" value="moving" />-->
<!--        <param name="world_name" value="circle" />-->
    </node>

    <node pkg="igvc_training_exercises" type="week3" name="week3" output="screen">
    </node>
</launch>
```

----

- Two `<node>` tags
    - Specifies a node to launch
    - `buzzsim` from `igvc_buzzsim` package
    - `week3` from `igvc_training_exercises` package
    
----

- Three `<param>` tags
    - One is commented out with `<!-- -->`
- We'll get to these `<param>` tags later

----

- What does a `.launch` file do?

<p>Lets you define nodes to launch and their respective parameters.</p> <!-- .element: class="fragment" data-fragment-index="0" -->

----

#### The `buzzsim` environment for the exercise
<img src="https://i.imgur.com/43gpYx0.png"/>

Goal: Move the bottom turtle to the top turtle using PID

---

#### Relevant rostopics and the message types
How do I find out which topics are relevant?

<div><code class="inline">rostopic list</code> to find the list of topics</div><!-- .element: class="fragment" data-fragment-index="0" -->
<div><code class="inline">rostopic info</code> to find the type of a topic</div><!-- .element: class="fragment" data-fragment-index="1" -->

----

- `/oswin/velocity`
    - Send velocity commands to the bottom turtle
- `/oswin/ground_truth`
    - Get where the bottom turtle is
- `/kyle/ground_truth`
    - Get where the top turtle is

---

#### Writing a proportional controller
- Implement the proportional controller in
[igvc_training_exercises/src/week3/main.cpp](https://github.com/RoboJackets/ros-training/blob/master/code/igvc_training_exercises/src/week3/main.cpp).

----

How?
1. Create a subscriber to the ground truth topics <!-- .element: class="fragment" data-fragment-index="0" -->
2. Create a publisher to the velocity topic <!-- .element: class="fragment" data-fragment-index="1" -->
3. Save Kyle's pose in a global variable <!-- .element: class="fragment" data-fragment-index="2" -->
<li>In the callback for <pre class="inline">/oswin/ground_truth</pre>, calculate the error</li> <!-- .element: class="fragment" data-fragment-index="3" -->
<li>Multiply the error by <pre class="inline">Kp</pre> to calculate the velocity</li> <!-- .element: class="fragment" data-fragment-index="4" -->
6. Publish the velocity to the velocity topic <!-- .element: class="fragment" data-fragment-index="5" -->


----
##### Subscribe to the ground truth topics for both turtles
Create a subscriber for `/oswin/ground_truth` and `/kyle/ground_truth` to find where the turtles are

- Make sure to `#include` the message type!
    - Clion should yell at you if you don't. Listen to Clion.

----
##### Save the position in a global variable
Save the position for `/kyle/ground_truth` in global variable `g_kyle_pose`

----
##### Sanity check - everything is still working
How do I check to make sure the subscribers are working?
<div>Use <pre class="inline">ROS_INFO_STREAM</pre> to print out the coordiantes</div> <!-- .element: class="fragment" data-fragment-index="1" -->

----

##### Publish to the `/oswin/velocity` topic to control the bottom turtle
- Create a publisher for the `/oswin/velocity`
- Put the publisher in the global variable `g_velocity_pub`
    - So that we can access it in the callback

----
##### Calculate the error in position
Calculate the error in x-coordinate between the two turtles

----
##### Implement a proportional controller in the callback for `/oswin/ground_truth`

<ol>
<li>Create a variable <code class="inline">kp</code> to store the
<img class="eqn-inline" src="https://latex.codecogs.com/svg.latex?\inline&space;K_p" />
coefficient.</li>
<li>Calculate 
<img class="eqn-inline" src="https://latex.codecogs.com/svg.latex?\inline&space;P_{out}&space;=&space;K_p&space;\,&space;e(t)" />
</li>
<li>Publish
<img class="eqn-inline" src="https://latex.codecogs.com/svg.latex?\inline&space;P_{out}" />
to the publisher created earlier</li>
</ol>

----
With `kp = 1`, after you compile and `roslaunch` you should then see the bottom turtle move up and stop exactly where
top turtle is.

----
What happens with different values of `kp`?

---

#### ROS parameters
- It's a pain to recompile every time you change a coefficient
- ROS parameters allow you to change parameters by specifying them in a `.launch` file

----

- `ros::NodeHandle` has a `.param(PARAM_NAME, VARIABLE)` method you can call
    - Give you the value of a parameter

----
To make `kp` a parameter:

```c++
double g_kp;
...
int main(int argc, char** argv)
{
  ros::init(argc, argv, "week3");

  ros::NodeHandle private_nh{"~"};
  pnh.getParam("kp", g_kp);
  ROS_INFO_STREAM("kp: " << g_kp);
}
```

- We pass `"~"` as a parameter to the `ros::NodeHandle` constructor
    - Makes `ros::NodeHandle` refer to the node's __private namespace__
    - Namespace between the `<node>` tags in a `.launch` file.

----
Also need to set the parameters in the `.launch` file

----

- Add a `<param>` tag between the `<node>` tag that launches the `week3` node:

```launch
<node pkg="igvc_training_exercises" type="week3" name="week3" output="screen">
    <param name="kp" value="1.0" />
</node>
```

----

- Correct value of the `kp` parameter is printed
- Can change the value in the `week3.launch` without recompiling

---

#### Visualizing data with `rqt_plot`
- Useful to visualize the error by plotting a graph of it

----

- Create a publisher of type `std_msgs::Float64` with the topic "error"
- Publish the "error" variable calculated in the callback for `/oswin/ground_truth`

How can you verify that it is publishing correctly? 
<pre><code>rostopic echo /error</code></pre> <!-- .element: class="fragment" data-fragment-index="1" -->

----

- ROS comes with the tool `rqt_plot` for plotting graphs and visualizing data
```bash
rqt_plot
```
- Add the `/error` topic to the graph
    - Type `/error` at the top
----
Launch the simulation and the controller. You should see a graph like below:
<img src="https://i.imgur.com/DFSYlmb.png" />

----
- Try playing with `kp` again
    - Verify that the graph shows what you see happening in the simulator.

---

### The PID Algorithm - Integral Controller
- Problem: We're trying to catch up to a car that's driving away from us?
    - What if our error 
<img class="eqn-inline" src="https://latex.codecogs.com/svg.latex?\inline&space;K_p&space;\,&space;e(t)" />
is equal to the target car's velocity?
<li>Because both cars have same velocity, error won't decrease</li> <!-- .element: class="fragment" data-fragment-index="1" -->

----

- Intuition:
    - If error wasn't decreasing, step harder on the throttle
    - The longer the error is present, the harder you step on the throttle.

----

- Can describe this behavior with the **integral** of the error
    - If the error is present for longer, the integral of the error will increase
    - Increase our control effort
    
----

This is an **integral controller**, where the **control** is proportional to the **integral** of the **error**

<p align="center">
    <img class="eqn"
    src="https://latex.codecogs.com/svg.latex?I_\text{out}&space;=&space;K_\text{i}&space;\,&space;\int_0^t&space;e(\tau)&space;\,d\tau" />
</p>

----

- Integrals are _hard_
- Easier to think of this in terms of **discrete** time
    - time happens in small steps
- Instead of computing an **integral**, we compute a **sum** of the error
    - Multiply it by the time step between each interval
    
<img src="https://www.physicsforums.com/attachments/graph-jpg.22137/" />

----

<p align="center">
    <img class="eqn" src="https://latex.codecogs.com/svg.latex?I_\text{out}&space;=&space;K_\text{i}&space;\,&space;\sum_{\tau=0}^{t}&space;e(\tau)&space;\,\Delta&space;t" title="I_\text{out} = K_\text{i} \, \sum_{\tau=0}^{t} e(\tau) \,\Delta t" />
</p>

or calculating recursively

<p align="center">
    <img class="eqn" src="https://latex.codecogs.com/svg.latex?\begin{align*}&space;I_\text{out}&space;=&&space;K_\text{i}&space;\cdot&space;\;&space;\textrm{accumulator}_t&space;\\&space;\textrm{accumulator}_t&space;=&&space;\;&space;\textrm{accumulator}_{t-1}&space;&plus;&space;e(t)&space;\;&space;\Delta&space;t&space;\end{align*}" title="\begin{align*} I_\text{out} =& K_\text{i} \cdot \; \textrm{accumulator}_t \\ \textrm{accumulator}_t =& \; \textrm{accumulator}_{t-1} + e(t) \; \Delta t \end{align*}" />
</p>

---

### Exercise: A moving target, and implementing an integral controller
- Let's try a moving target this time
- Ctrl-/ to comment out the `<param name="world_name" value="stationary" />` line
- Do the same thing on the line below to uncomment `<param name="world_name" value="moving" />`

----

It should look like this:
```launch
<launch>
    <node pkg="igvc_buzzsim" type="buzzsim" name="buzzsim">
        <param name="config_path" value="$(find igvc_training_exercises)/config/week3/world.yml" />
<!--        <param name="world_name" value="stationary" />-->
        <param name="world_name" value="moving" />
<!--        <param name="world_name" value="circle" />-->
    </node>

    <node pkg="igvc_training_exercises" type="week3" name="week3" output="screen">
        <param name="kp" value="1.0" />
    </node>
</launch>
```

----

- If you `roslaunch` again, you should see the top turtle start to move
- Does your proportional controller work?
- Verify what you're seeing by checking `rqt_plot`.

----

The error that you're seeing is called **steady state error**. As we explained earlier, the **integral controller**
helps to solve this.

---

## But wait, we're not going to implement this
The proof is trivial and left as an exercise for the reader

You thought I was going to do all the hard work for you guys? Well too bad, we're moving on to Derivative.

----

### The PID Algorithm - Derivative Controller

- Imagine we're back in the first scenario where we're trying to stop right in from the stop sign
- We're driving a massive truck
- What problems does our *proportional controller* have?
<ul>
<li>We keep our foot on the throttle even when we're going full speed in front of the stop sign</li> <!-- .element: class="fragment" data-fragment-index="0" --> 
<li>Result in overshoot, oscillating back and forth</li> <!-- .element: class="fragment" data-fragment-index="0" --> 
</ul>
----

- Intuition:
    - Step on the throttle at the beginning to get the truck moving
    - As the truck begins to accelerate, use the throttle less
    - The faster you're approaching the target position, the more you ease off the throttle, or even apply the brakes

----

Another example:
- Scenario: opening a valve to fill a cup of water
- If the cup is filling up too quickly, close the tap to prevent water from overflowing
- On the other hand, if the cup broke a hole opened up at the bottom and water started to leak 
    - Since error is now _increasing_ quickly, the derivative term will try to open up the tap more

----

- Component of control that is proportional to the **derivative** of the error
    - The faster the error is _decreasing_, the more we _ease off_ on our **control**
    - The faster the error is _increasing_, the more we _increase_ our **control**
----

This is a **derivative controller**, where the **control** is proportional to the **derivative** of the **error**.

<p align="center">
    <img class="eqn"
    src="https://latex.codecogs.com/svg.latex?D_\text{out}&space;=&space;K_\text{d}&space;\,&space;\frac{d\,&space;e(t)}{dt}" />
</p>

----

For the **discrete time** version, calculate the derivative with **finite differences**:

<p align="center">
    <img class="eqn"
    src="https://latex.codecogs.com/svg.latex?D_{out}&space;=&space;K_d&space;\;&space;\frac{e(t)&space;-&space;e(t-1)}{\Delta&space;t}" title="D_{out} = K_d \; \frac{e(t) - e(t-1)}{\Delta t}" />
</p>

----

### Exercise: Not overshooting, and derivative

- This is left as an exercise for the reader again

----

- Similar to last week, check out the markdown version on [github](https://github.com/RoboJackets/ros-training/blob/master/code/instructions/week3.md)
    - Contains additional exercises (implementing the **I** and **D** parts of PID)
- Any questions?

---

## Summary

----

- [PID control and basic control theory](#the-pid-algorithm---proportional-controller)
    + Controlling a **Process Variable** to equal some **Set Point**
    + The **P**roportional **I**ntegral **D**erivative algorithm as a simple controller
    + Understanding what a controller does in the context of a car
    
----

- [`roslaunch` and `.launch` files](#launching-buzzsim-with-roslaunch)
    + Using `roslaunch <ros package> <launch file>` to launch a `.roslaunch` file
    + `<node>` tags to define ROS nodes to be launched in a `.roslaunch` file
    
----

- [ROS parameters](#ros-parameters)
    + Using `nh.getParam` to get parameters defined in a launch file
    + Adding parameters to the launch file with the `<launch>` tag
    
----

- [Written our own PID controller](#exercise-implementing-a-proportional-controller-in-ros)
    + Implementing something we just learnt about in theory!
    + And it works!
    
----

## Next Week
We'll learn
about reference frames, the IMU, and localization with dead reckoning by integrating IMU data.

---

See you next week!
