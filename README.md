# Path Planning

**The goals / steps of this project are the following:**

  - Design a path planner that is able to create smooth, safe paths for
    the car to follow along a 3 lane highway with traffic. 
    
  - A successful path planner will be able to keep inside its lane, avoid
    hitting other cars, and pass slower moving traffic all by using
    localization, sensor fusion, and map data.
    
#### Here I will consider the points from the project individually and describe how I addressed each point in my implementation

##### 1. The car is able to drive at least 4.32 miles without an incident.

![](https://i.imgur.com/zqBxxdv.png)

##### 2. The car drives according to the speed limit.

The car drives ~30 MPH which was how it is able to slow down without max acceleration or jerk, when a car in front of it slow down unexpectedly. 
The other cars on the road drive ~5 MPH faster but are willing to accept more jerk and more negative acceleration (and possibly accidents).

##### 3. Max Acceleration and jerk are not exceeded.

The car does not exceed a total acceleration of 10 m/s^2 and a jerk of 10 m/s^3 as described above.

##### 4. Car does not have collisions.

The car does not come into contact with any of the other cars on the road, and has a stopping distance that is needed because of latency and the slowing distance (from fast velocity to a slower velocity). I tried to anticipate collisions by flagging cars (inside a safty buffer) that I thought might change lanes unexpectedly (they slow down and get closer to the ego car's lane).

##### 5. The car stays in its lane, except for the time between changing lanes.

I programmed the car to decide which lane will be better (without near accidents) and not to move to other lanes unless it needs to.
The car doesn't spend more than a 3 second length outside the lane during changing lanes, and every other time the car stays inside one of the 3 lanes on the right hand side of the road.

##### 6. The car is able to change lanes

The car is able to smoothly change lanes when it makes sense to do so, such as when behind a slower moving car and an adjacent lane is clear of other traffic.
I gave the car an ability to change lanes, but it usually chooses to stay on the middle lane because of fast moving cars that come from behind it on the adjacent lanes.

![](https://i.imgur.com/SiL0XhO.png)

#### The Model

The code for the model is in the file: Vehicle.cpp.
The model contains 3 parts:

##### A finite state machine

There are 6 states : “CA”(constant acceleration), “KL”(keep lane), “PLCL”
(prepare lane change left), “LCL” (lane change left), “PLCR” (prepare lane change
right) and “LCR” (lane change right).

![](https://i.imgur.com/Hldmbba.png)

This is a slightly different state machine then what was described in class:
The state “CA” (constant speed) is the first state and it leads only to the state “KL”. 
“KL” (keep lane) has following states which depend on the location of the car. 
If the car is in the middle lane then the following states will be “PLCL”, “PLCR” 
and “KL”. If the car is not in the middle lane, then only the right states are added.
The state of “KL” is added first, so the car will prefer to stay in the lane, if there is a
choice between 2 lanes with the same cost.
Both “”PLCL” and “PLCR” have 2 following states:
“LCL” or “LCR” (changing lane) and “KL” if the situation changes.
Both “”LCL” and “LCR” have only one following state: “KL”.

![](https://i.imgur.com/B3j9KB5.png)

##### Calculation of the cost for each state (which is translated to a lane number) And decision about the best state (with minimum cost)

For each state I calculate 3 costs: collision cost, efficiency cost and buffer cost.
The code for the calculation is in a separate file : cost_functions.cpp.
Collision cost is regarding to the possibility of a collision in the target lane.
Efficiency cost is regarding to future velocity of the car in the target lane. Will the car
be able to drive faster or maintain its current speed.
Buffer cost is regarding to how far will the car be from other cars 
when it will move to the target lane.
After calculating the costs for each possible following state I am multiplying them with weights according to their importance (collision has of course maximum weight).
I sum all the costs and assign the new total cost to the corresponding following state.
After that I choose the state with the minimum costs.

![](https://i.imgur.com/RQi8HRN.png)

##### Updating the car with the new state and all that follows (acceleration and etc)

After the decision on the best state, the lane and acceleration are being updated:

![](https://i.imgur.com/nrlMEbb.png)

#### Reflection

##### Problems / issues I faced in my implementation of this project:

**_1.Handling multiple unexpected different scenarios on the highway, when the car’s program is already set._**

  **a.** When driving, there might be multiple unexpected scenarios and interaction with other cars. If there is a driver then he/she can drive accordingly. 
However, this is not the case with an autonomous vehicle, where the car is already programmed. 
This means that I as the developer needed to anticipate many scenarios.

  **b.** There is also a need to describe every unexpected situation in details. 
For example : the car before the autonomous car is stopping unexpectedly. 
If there was a driver than he/she can understand what they are seeing. 
But for an autonomous car there a detailed description is needed, so I needed to explain to myself what is actually happening so a program can understand : "the velocity of the car before you is less than (threshold - which threshold?) and / or the distance between you is smaller than (threshold - which threshold?).

  **c.** After identifying the problem there is also the matter of the solution which is not always definitive and might require additional measurements / resources and time that is not always given. For example : in order to decide which is safer -  to turn to another lane or to slow down in the current lane we need to get additional measurements of the velocity of the cars in all the lanes.
This needs to be done for multiple and sometimes unexpected scenarios

**_2. Acquiring information that is missing from the sensor-fusion data._**

Sometimes, not all the information that is needed to perform calculations is given. For example: the acceleration of other cars on the road is needed to calculate their velocity.
Therefore, I needed to get the data using physics equations from other measurements and make calculated assumptions based on the given data.

**_3. Calculating the stopping distance and the best negative acceleration._**

It takes time and distance for a car to respond to a situation on the road. Since that there are many possible situations and solutions, there is a need to take into account the best negative acceleration (so there won't be jerk) and the stopping distance before every definite decision on a solution. 
