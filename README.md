# CarND-Controls-MPC
Self-Driving Car Engineer Nanodegree Program

---

## Implementation Details

In this project I have used Model Predictive Control to drive a car around a track in Udacity provided
Simulator. A video of the output is [here](https://www.youtube.com/watch?v=FUAZ29bJllg)

## Model Description

- The model is based on the Udacity [in-class quiz](https://github.com/udacity/CarND-MPC-Quizzes/blob/master/mpc_to_line/solution/MPC.cpp)
- The state is encoded as (x, y, psi, v, cte, epsi), where (x, y) is vehicle current location,
psi is the current vehicle bearing, and v is the current speed. cte and epsi is the cross-track-error (on y axis) and bearing difference with reference waypoints.
- There are two actuators - steering and throttle, both are in the range [-1, 1]
- The vehicle model can be described as 
```c++
x_[t+1] = x[t] + v[t] * cos(psi[t]) * dt
y_[t+1] = y[t] + v[t] * sin(psi[t]) * dt
psi_[t+1] = psi[t] + v[t] / Lf * delta[t] * dt
v_[t+1] = v[t] + a[t] * dt
cte[t+1] = f(x[t]) - y[t] + v[t] * sin(epsi[t]) * dt
epsi[t+1] = psi[t] - psides[t] + v[t] * delta[t] / Lf * dt
```

## Choosing `N` and `dt` values 

- 1. If the waypoint model is accurate enough, using a big 'N' will help "foresee" the road further and thus plan driving in a smoother way. 
    However, since most of time the polynomial model of waypoints is just a local approximation, a big N might increases the computation cost and decreases the accuracy - because the model is not accurate for 'N' too far away from 0.
- 2. Using a smaller 'dt' would help find actuators parameters that are optimal to "local approximations" that are not too far away from the current position. However, if 'dt' is too small and 'N' is also small, the driving model won't be able to see "far" enough so it might have difficulties at turning. 
    Similiarly, using 'dt' should not be too small compared to latency.In practice, 'N' or/and 'dt' should be increased when the reference speed is increased - this helps look further ahead on the road, and make the driving smoother.
- 3 In practice, 'N' or/and 'dt' should be increased when the reference speed is increased - this helps look further ahead on the road, and make the driving smoother.

## How to fit polynomials to waypoints

- "ptsx" and "ptsy" contains the nearest way points (up to 6) ahead of current vehicle position (px, py). 
- So it makes sense to transform from global corrdinates to vehicle coordinates, for two reasons:
  - 1. the polynomial fit of waypoints is expected to be more accurate in the vehicle coordinates, as in most cases the road is just linear
  - 2. it helps the optimizer find optimal solutions more easily, because the optimal solutions won't be too far away from initials (all zeros).
- Switching to the vehicle coordnates makes a huge improvement in performance, although I am not sure whether it is still possible to come up with a good solution with original coordinates.
- Transform map coordinates to car coordinates
  - (1) centerize to vehicle position
  - (2) rotate -psi to align with vehichle y axis
r
## Deal with latency
- As mentioned above, picking the right value for `N` and `dt`, e.g., increasing `dt` to plan further ahead, helps with dealing with acutator latency
- I also use different weights for different objective components following the intuition
  - heaviest weights on `cte` and `epsi` to help vehicle stay on the track
  - relatively weak weight on speed v constraint with refence value, specially when driving at high speed, so acceleration can be used more freely
  - steering and throttle should be small 
  - relatively heavier weight on smooth steering to prevent driving from being wiggly
  - relatively heavier weight on throttle change so driving can be as constant as possible



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
* [uWebSockets](https://github.com/uWebSockets/uWebSockets) == 0.14, but the master branch will probably work just fine
  * Follow the instructions in the [uWebSockets README](https://github.com/uWebSockets/uWebSockets/blob/master/README.md) to get setup for your platform. You can download the zip of the appropriate version from the [releases page](https://github.com/uWebSockets/uWebSockets/releases). Here's a link to the [v0.14 zip](https://github.com/uWebSockets/uWebSockets/archive/v0.14.0.zip).
  * If you have MacOS and have [Homebrew](https://brew.sh/) installed you can just run the ./install-mac.sh script to install this.
* [Ipopt](https://projects.coin-or.org/Ipopt)
  * Mac: `brew install ipopt --with-openblas`
  * Linux
    * You will need a version of Ipopt 3.12.1 or higher. The version available through `apt-get` is 3.11.x. If you can get that version to work great but if not there's a script `install_ipopt.sh` that will install Ipopt. You just need to download the source from the Ipopt [releases page](https://www.coin-or.org/download/source/Ipopt/) or the [Github releases](https://github.com/coin-or/Ipopt/releases) page.
    * Then call `install_ipopt.sh` with the source directory as the first argument, ex: `bash install_ipopt.sh Ipopt-3.12.1`. 
  * Windows: TODO. If you can use the Linux subsystem and follow the Linux instructions.
* [CppAD](https://www.coin-or.org/CppAD/)
  * Mac: `brew install cppad`
  * Linux `sudo apt-get install cppad` or equivalent.
  * Windows: TODO. If you can use the Linux subsystem and follow the Linux instructions.
* [Eigen](http://eigen.tuxfamily.org/index.php?title=Main_Page). This is already part of the repo so you shouldn't have to worry about it.
* Simulator. You can download these from the [releases tab](https://github.com/udacity/CarND-MPC-Project/releases).



## Basic Build Instructions


1. Clone this repo.
2. Make a build directory: `mkdir build && cd build`
3. Compile: `cmake .. && make`
4. Run it: `./mpc`.

## Tips

1. It's recommended to test the MPC on basic examples to see if your implementation behaves as desired. One possible example
is the vehicle starting offset of a straight line (reference). If the MPC implementation is correct, after some number of timesteps
(not too many) it should find and track the reference line.
2. The `lake_track_waypoints.csv` file has the waypoints of the lake track. You could use this to fit polynomials and points and see of how well your model tracks curve. NOTE: This file might be not completely in sync with the simulator so your solution should NOT depend on it.
3. For visualization this C++ [matplotlib wrapper](https://github.com/lava/matplotlib-cpp) could be helpful.

## Editor Settings

We've purposefully kept editor configuration files out of this repo in order to
keep it as simple and environment agnostic as possible. However, we recommend
using the following settings:

* indent using spaces
* set tab width to 2 spaces (keeps the matrices in source code aligned)

## Code Style

Please (do your best to) stick to [Google's C++ style guide](https://google.github.io/styleguide/cppguide.html).

## Project Instructions and Rubric

Note: regardless of the changes you make, your project must be buildable using
cmake and make!

More information is only accessible by people who are already enrolled in Term 2
of CarND. If you are enrolled, see [the project page](https://classroom.udacity.com/nanodegrees/nd013/parts/40f38239-66b6-46ec-ae68-03afd8a601c8/modules/f1820894-8322-4bb3-81aa-b26b3c6dcbaf/lessons/b1ff3be0-c904-438e-aad3-2b5379f0e0c3/concepts/1a2255a0-e23c-44cf-8d41-39b8a3c8264a)
for instructions and the project rubric.

## Hints!

* You don't have to follow this directory structure, but if you do, your work
  will span all of the .cpp files here. Keep an eye out for TODOs.

## Call for IDE Profiles Pull Requests

Help your fellow students!

We decided to create Makefiles with cmake to keep this project as platform
agnostic as possible. Similarly, we omitted IDE profiles in order to we ensure
that students don't feel pressured to use one IDE or another.

However! I'd love to help people get up and running with their IDEs of choice.
If you've created a profile for an IDE that you think other students would
appreciate, we'd love to have you add the requisite profile files and
instructions to ide_profiles/. For example if you wanted to add a VS Code
profile, you'd add:

* /ide_profiles/vscode/.vscode
* /ide_profiles/vscode/README.md

The README should explain what the profile does, how to take advantage of it,
and how to install it.

Frankly, I've never been involved in a project with multiple IDE profiles
before. I believe the best way to handle this would be to keep them out of the
repo root to avoid clutter. My expectation is that most profiles will include
instructions to copy files to a new location to get picked up by the IDE, but
that's just a guess.

One last note here: regardless of the IDE used, every submitted project must
still be compilable with cmake and make./
