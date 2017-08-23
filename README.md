# CarND-Path-Planning-Project
Self-Driving Car Engineer Nanodegree Program

### Goals
In this project your goal is to safely navigate around a virtual highway with other traffic that is driving +-10 MPH of the 50 MPH speed limit. You will be provided the car's localization and sensor fusion data, there is also a sparse map list of waypoints around the highway. The car should try to go as close as possible to the 50 MPH speed limit, which means passing slower traffic when possible, note that other cars will try to change lanes too. The car should avoid hitting other cars at all cost as well as driving inside of the marked road lanes at all times, unless going from one lane to another. The car should be able to make one complete loop around the 6946m highway. Since the car is trying to go 50 MPH, it should take a little over 5 minutes to complete 1 loop. Also the car should not experience total acceleration over 10 m/s^2 and jerk that is greater than 50 m/s^3.

## Write-up

### The car drives according to the speed limit.

The car is set to drive at a target speed of 49.5 miles an hour ```main.cpp line 206```. This target speed is used in spacing the points along the spline ```main.cpp line 377 - 394, specifically line 378```. The controller receives a message every 0.02 seconds. By multiplying this value with the speed in m/s ```target speed / 2.24``` we get the distance between the points.

### Max Acceleration and Jerk are not Exceeded.

Just setting / changing the target speed makes the car respond immediately and therefor greatly exceeds the acceleration and jerk limits. To ensure this doesn't happen we use a filter we update the target speed every cycle by using the following formula:

``` 92% * current car speed + 8% * target car speed (line 164)```

The ratio between the current car speed and the target speed are chosen imperically.

### Car does not have collisions.

Every cycle the sensor fusion data is looped ```main.cpp line 255 - 279``` here we check whether there are other cars in the same lane ```line 264 - 269``` and if so we check if there are any cars in front of us closer than 30 meters ```line 265```. If there is a car closer than 30 meters in front of us we set the ```too_close``` flag to true and we adjust the speed of the car to match the speed of the car in front minus 10 mph. This difference is chosen imperically and is necessary to make sure not to collide with breaking cars.

If there are cars too close in front of us we first of all adjust the speed: ```line 303```.

If we are no longer too close to a car: go back to the target speed of 49.5 mph ```line 306```.

### The car is able to change lanes


In the same cycle we check for cars to the left of us ```line 269 - 273``` and to the right of us ```line 273 - 276``` if there are cars here we save the distance to the nearest car (in front or behind us) in respectively ```LCL``` or ```LCR```.

We check whether there is a possibility to swerve left or right. To do this we need at least 15m of clear distance in the lane ```line 281```. If there is a possibility to change the lane, we will only initiate this if we ourselves are in the center of our current lane (so we don't decide to switch back mid action) ```line 283```. And then we go through a small decision tree:

- If we can only go left: go left ```line 284 - 288```
- If we can only go right: go right ```line 288 - 292```
- If we can go left or right, but there's more space left: go left ```line 292 - 296```
- If we can go left or right, but there's more space right: go right ```line 296 - 300```

### Model description

I was greatly helped by the walkthrough video posted by udacity. I have implemented the path model they suggest here. It starts by creating two vectors of points ```ptsx``` for the x values and ```ptsy``` for the y values.

If there are not enough previous points recorded yet we initiate this with points based on the current cars position ```line 325 & 327``` and a previous point calculated back using the cars current position and heading and going one step back ```line 321 - 322```.

If there are enough previous points we push does to the points vectors and also calculate the heading based on the previous and previous previous point ```line 329 - 338```.

Next we create three future points respectively 30, 60 and 90 meters ahead of the current possition and we use getXY to convert the Frenet coordinates to Cartesian coordinates ```line 340 - 350```.

We can use these point vectors to create a spline. But to do so we have to shift the coordinates from map coordinates to local (car) coordinates ```line 352 - 358```.

We create the spline using Kluge's spline header ```line 360 - 365```. And now we need to find the points on the spline spaced in such a way that the car will travel at the target speed. ```line 372 - 384```. We then need to shift the coordinates back to map coordinates ```line 386 - 390``` and we need to send them to the car ```line 392 - 397```.

### Reflection

The walkthrough was very usefull to get a good start. Using the future points and incorporating the lane number and speed in there was a great way to easily build the request lane shifting and speed controlling functions on top. It was fairly easy to build the lane switching and speed controls, but I spent quite some time optimising them in such a way that it felt efficient and fast. Checking whether the car was in the center of it's own lane ```line 283``` proved very helpful in preventing the car from making to irredical lane shifts.

### Updates

The Udacity reviewer brought to my attention that the car sometimes changes lanes to a lane where the car is actually closer and than immediately changes back (uncomfortable). This was because the distance in order to change  a lane was set to 15 meters, where the distance to keep to the car in front was set at 30 meters. I have adjusted this by setting the lane changing distance to 35 meters, but at the same time multiplying the distance when the value is negative ```line 263 - 265``` this ensures the car can overtake when the distance to a car in the next lane is 35 meters in front, or 17.5 meters behind. Keeping a safe distance to both :)
