# CarND-Path-Planning-Project
Self-Driving Car Engineer Nanodegree Program

#### William Rifenburgh

### Simulator.
You can download the Term3 Simulator which contains the Path Planning Project from the [releases tab (https://github.com/udacity/self-driving-car-sim/releases).

### Goals
In this project your goal is to safely navigate around a virtual highway with other traffic that is driving +-10 MPH of the 50 MPH speed limit. You will be provided the car's localization and sensor fusion data, there is also a sparse map list of waypoints around the highway. The car should try to go as close as possible to the 50 MPH speed limit, which means passing slower traffic when possible, note that other cars will try to change lanes too. The car should avoid hitting other cars at all cost as well as driving inside of the marked road lanes at all times, unless going from one lane to another. The car should be able to make one complete loop around the 6946m highway. Since the car is trying to go 50 MPH, it should take a little over 5 minutes to complete 1 loop. Also the car should not experience total acceleration over 10 m/s^2 and jerk that is greater than 50 m/s^3.

## Basic Build Instructions

1. Clone this repo.
2. Make a build directory: `mkdir build && cd build`
3. Compile: `cmake .. && make`
4. Run it: `./path_planning`.

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
## Reflection

The path planning pipeline of this project first collects information on the state of the vehicle and the vehicles around it and then calculates a smooth spline trajectory based on speed and lane objectives from a simple algorithm. The algorithm can be summed up as follows:

```
If a car less than 30[m] infront of me is going slower than me, try to change lanes. If the lanes next to me are blocked by other cars, slow down. Otherwise, try to go the speed limit.
```

In code, this algorithm appears as follows

```cpp

// If the car in front of me is less than 30 meters away, its too
// close
if (opponent_lane == my_lane && (opponent_s > car_s) &&
    ((opponent_s - car_s) < 30)) {
  too_close = true;
}

// If the opponent car is going faster than me or blocking me, the
// lane is not empty
if (opponent_speed > car_speed ||
    ((opponent_s - car_s > -10) && (opponent_s - car_s < 30))) {
  lane_is_empty[opponent_lane] = false;
}
}

if (too_close) {
// If the car in front of me is too close, try to change to an empty
// lane - otherwise, slow down.
if (my_lane - 1 >= 0 && lane_is_empty[my_lane - 1]) {
  my_lane -= 1;
} else if (my_lane + 1 <= 2 && lane_is_empty[my_lane + 1]) {
  my_lane += 1;
} else {
  ref_vel -= 0.244;
}
}
// If the car in front is not too close, try to go the speed limit.
else if (ref_vel < 49.5) {
ref_vel += 0.224;
}

```

After a desired velocity and desired lane are calculated, waypoints 30, 60, and 90 meters ahead are selected and a trajectory is calculated using spline methods as described in the unlisted path planning walkthrough video (See project lesson chapter 5).
