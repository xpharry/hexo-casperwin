---
title: 代码赏析 | Stanley Path Tracking Controller
date: 2019-07-31 12:00:27
tags:
---

# Stanley Path Tracking Controller

## 参考论文：
- [Stanley: The robot that won the DARPA grand challenge](http://isl.ecst.csuchico.edu/DOCS/darpa2005/DARPA%202005%20Stanley.pdf)
- [Autonomous Automobile Path Tracking](https://www.ri.cmu.edu/pub_files/2009/2/Automatic_Steering_Methods_for_Autonomous_Automobile_Path_Tracking.pdf)

## 源代码：

https://github.com/AtsushiSakai/PythonRobotics/blob/master/PathTracking/stanley_controller/stanley_controller.py

<!--more-->

### 节选代码1：

```python
def pid_control(target, current):
    """
    Proportional control for the speed.

    :param target: (float)
    :param current: (float)
    :return: (float)
    """
    return Kp * (target - current)
```

Stanley Controller对速度的控制是简单的PID控制，在这段代码中，其实也就是P控制，因为只有比例增益P。

### 节选代码2：

```python
def stanley_control(state, cx, cy, cyaw, last_target_idx):
    """
    Stanley steering control.

    :param state: (State object)
    :param cx: ([float])
    :param cy: ([float])
    :param cyaw: ([float])
    :param last_target_idx: (int)
    :return: (float, int)
    """
    current_target_idx, error_front_axle = calc_target_index(state, cx, cy)

    if last_target_idx >= current_target_idx:
        current_target_idx = last_target_idx

    # theta_e corrects the heading error
    theta_e = normalize_angle(cyaw[current_target_idx] - state.yaw)
    # theta_d corrects the cross track error
    theta_d = np.arctan2(k * error_front_axle, state.v)
    # Steering control
    delta = theta_e + theta_d

    return delta, current_target_idx
```

控制器相对复杂的部分是对方向角的控制，公式的运用主要体现在'stanley_control()'这段代码中。

{% asset_img stanley-geometry.png Stanley Geometry %}

{% asset_img stanley-equation.png Stanley Equation %}
