# CarND-Path-Planning-Project-KKP
Self-Driving Car Engineer Nanodegree Program

[image1]: PP5.png 

![Path_Planning][image1]

### Summary.
In this project, the path planning algorithm implemented using C++ is introduced in order to navigate the car throughout the virtual environment safely. In the simulator, other vehicles travel with a driving speed between 40-60 MPH and can change a lane if needed. Prior to path planning section, the car's localization, sensor fusion data, and a sparse map list of waypoints aroudn the highway are given. The path planning algorithm must pass these criteria:

- The car is able to drive at least 4.32 miles without incident. The top right screen of the simulator shows the current/best miles driven without incident. Incidents include exceeding acceleration/jerk/speed, collision, and driving outside of the lanes. Each incident case is also listed below in more detail.

- The car drives according to the speed limit, max at 50 MPH. The car doesn't drive faster than the speed limit. Also the car isn't driving much slower than speed limit unless obstructed by traffic.

- Max acceleration and jerk are not exceeded: the car does not exceed a total acceleration of 10 m/s^2 and a jerk of 10 m/s^3.

- Car does not have collision, The car must not come into contact with any of the other cars on the road.

- The car stays in its lane, except for the time between changing lanes. The car doesn't spend more than a 3 second length out side the lane lanes during changing lanes, and every other time the car stays inside one of the 3 lanes on the right hand side of the road.

- The car is able to change lanes. The car is able to smoothly change lanes when it makes sense to do so, such as when behind a slower moving car and an adjacent lane is clear of other traffic.

The path planning algorithm is testes and passes these criteria without incident. 

### Reflection: Code Details and Explanation

The first path of this project is to determine and decide the action of the vehicle (line 264 - 326). The lane number is identified using d distance in frenet coordinate. With the s distance in frenet coodinate of 30 m ahead and behind the vehicle, the index variable to determine whehter it's safe to change lane to the right/left or stay in the lane is introduced (line 294-301). For example, with +-30 m from the vehicle to the left, if there's no car, it's safe to change lane to the left. After than, it comes to the finite state machine (line 307-326):

- If there is other car in front of our vehicle, the vehicle will change lane to the left/right it it's safe (meaning no other car within +- 30 m in the left/right lane), or stay in the lane but will decrease the speed.

- If there is no other car in front of our vehicle, the vehicle can change the lane to the middle lane if it's safe, or not changing lane but accelerate to reach the desired speed.

The second path is related the path generation. The vehicle waypoints ptsx, ptsy is introduced. Then, the heading of the vehicle is calculated, assuming that the local coordication of the vehicle starting at zero (line 334-372). Then, the next waypoint of 30, 60, 90 m ahead of the vehicle is generated using the provided "getXY" helper function, for example 
```cpp
vector<double> next_wp0 = getXY(car_s+30, (2+4*lane), map_waypoints_s, map_waypoints_x, map_waypoints_y);
vector<double> next_wp1 = getXY(car_s+60, (2+4*lane), map_waypoints_s, map_waypoints_x, map_waypoints_y);
vector<double> next_wp2 = getXY(car_s+90, (2+4*lane), map_waypoints_s, map_waypoints_x, map_waypoints_y);  
```
After than, we apply the transformation from vehicle local coordinate back to global/map coordinate by shift and rotation (line 393-414). Then, the spline library is used to created the intropolated smooth waypoint for the vehicle, next_x_vals, next_y_vals (line 417-431). In order to break up the spling points so that we travel at our desired reference speed, the details of calculation is presented in line 435-465. Particularly we only calculate and always output the desired waypoint of 50 points. The desired speed is utilized in line 445 in order to determine the number of breakout section, so that the interpolation of waypoint is guaranteed to produce the desired speed as well. After that, we apply the inverse transformation from map-->local car coordinate and store x,y point to the variable next_x_vals, and next_y_vals as desired path waypoints.


### Simulator.
The Term3 Simulator which contains the Path Planning Project from the [releases tab (https://github.com/udacity/self-driving-car-sim/releases/tag/T3_v1.2).


#### The map of the highway is in data/highway_map.txt
Each waypoint in the list contains  [x,y,s,dx,dy] values. x and y are the waypoint's map coordinate position, the s value is the distance along the road to get to that waypoint in meters, the dx and dy values define the unit normal vector pointing outward of the highway loop.

The highway's waypoints loop around so the frenet s value, distance along the road, goes from 0 to 6945.554.

## Basic Build Instructions

1. Clone this repo.
2. Make a build directory: `mkdir build && cd build`
3. Compile: `cmake .. && make`
4. Run it: `./path_planning`.

Here is the data provided from the Simulator to the C++ Program

#### Main car's localization Data (No Noise)

["x"] The car's x position in map coordinates

["y"] The car's y position in map coordinates

["s"] The car's s position in frenet coordinates

["d"] The car's d position in frenet coordinates

["yaw"] The car's yaw angle in the map

["speed"] The car's speed in MPH

#### Previous path data given to the Planner

//Note: Return the previous list but with processed points removed, can be a nice tool to show how far along
the path has processed since last time. 

["previous_path_x"] The previous list of x points previously given to the simulator

["previous_path_y"] The previous list of y points previously given to the simulator

#### Previous path's end s and d values 

["end_path_s"] The previous list's last point's frenet s value

["end_path_d"] The previous list's last point's frenet d value

#### Sensor Fusion Data, a list of all other car's attributes on the same side of the road. (No Noise)

["sensor_fusion"] A 2d vector of cars and then that car's [car's unique ID, car's x position in map coordinates, car's y position in map coordinates, car's x velocity in m/s, car's y velocity in m/s, car's s position in frenet coordinates, car's d position in frenet coordinates. 

## Details

1. The car uses a perfect controller and will visit every (x,y) point it recieves in the list every .02 seconds. The units for the (x,y) points are in meters and the spacing of the points determines the speed of the car. The vector going from a point to the next point in the list dictates the angle of the car. Acceleration both in the tangential and normal directions is measured along with the jerk, the rate of change of total Acceleration. The (x,y) point paths that the planner recieves should not have a total acceleration that goes over 10 m/s^2, also the jerk should not go over 50 m/s^3. (NOTE: As this is BETA, these requirements might change. Also currently jerk is over a .02 second interval, it would probably be better to average total acceleration over 1 second and measure jerk from that.

2. There will be some latency between the simulator running and the path planner returning a path, with optimized code usually its not very long maybe just 1-3 time steps. During this delay the simulator will continue using points that it was last given, because of this its a good idea to store the last points you have used so you can have a smooth transition. previous_path_x, and previous_path_y can be helpful for this transition since they show the last points given to the simulator controller with the processed points already removed. You would either return a path that extends this previous path or make sure to create a new path that has a smooth transition with this last path.

## Tips

A really helpful resource for doing this project and creating smooth trajectories was using http://kluge.in-chemnitz.de/opensource/spline/, the spline function is in a single hearder file is really easy to use.

---

## Dependencies

* cmake >= 3.5
  * All OSes: [click here for installation instructions](https://cmake.org/install/)
* make >= 4.1
  * Linux: make is installed by default on most Linux distros
  * Mac: [install Xcode command line tools to get make](https://developer.apple.com/xcode/features/)
  * Windows: [Click here for installation instructions](http://gnuwin32.sourceforge.net/packages/make.htm)
* gcc/g++ >= 5.4
  * Linux: gcc / g++ is installed by default on most Linux distros
  * Mac: same deal as make - [install Xcode command line tools]((https://developer.apple.com/xcode/features/)
  * Windows: recommend using [MinGW](http://www.mingw.org/)
* [uWebSockets](https://github.com/uWebSockets/uWebSockets)
  * Run either `install-mac.sh` or `install-ubuntu.sh`.
  * If you install from source, checkout to commit `e94b6e1`, i.e.
    ```
    git clone https://github.com/uWebSockets/uWebSockets 
    cd uWebSockets
    git checkout e94b6e1
    ```


