---
title: 粒子滤波器 - 姿态跟踪（Particle Filter -­ Pose Tracking）
date: 2019-04-11 12:00:27
tags:
---

<img src="https://paper-attachments.dropbox.com/s_B11EF229FFC176E08FB7E677870E42312ADC86B9923359B137487FF1777CDD72_1554936111933_image.png" alt="venv" style="height:500px;" align="middle">

## 概述

这次作业是要求用粒子滤波器（Particle Filter）的方法在2维平面跟踪姿态。

想象一个机器人想要知道它自己的位置，并以此决定之后往哪里探索。下面这张图描绘了一个机器人定位的例子，用激光雷达测量投射到一张地图上。

<!--more-->

## 指导

设计粒子滤波器模型


1. 动力学模型（系统模型）


  a. 为了随时间移动粒子，一个里程计更新的分布用来建立机器人的运动模型。粒子滤波器每次从这个分布中采样，再更新各个粒子的位置和方向。


  b. 为了简化，编码器测量这里没有提供。不过，你可以使用一个期望为0但具有高协方差的分布。（或者，你可以使用卡尔曼滤波器（Kalman Filter）追踪位置和时间的变化，以此更新期望值）。


  c. 现在考虑在粒子中的随机噪声，通过系统噪声协方差Sigma_m。最简单的形式是一个用4个变量表示的对角矩阵：Sigma_m = diag([sigma_x^2, sigma_y^2, sigma_theta^2])


    i. 大体上，你不一定要限制于这个形式，但一个对角矩阵是一个很好的开始，如果你没有特别强的理由不这么做。

    ii. 你应该随机地从这个多变量标准分布中采样，以此加入噪声。注意：使用对角矩阵，你可以对各维分别采样，用randn。

    iii. [高级] 或者，你可以使用从卡尔曼滤波器得到的估计的机器人速度，当速度增加和减少来相应增大或减小噪声。


2. 地图标记


  a. 测量数据由一组各角度的激光雷达距离读取值提供，在机器人的本地坐标上。


    i. 这个数据被画在 exmaple.m

    ii. 在这个点，你应该测试方法，转换本地激光雷达数据到全局地图坐标下。


![图2： 本地坐标系下的激光雷达数据，单位米。你需要将其转换为离散的地图晶格坐标，会在(b)中讨论。](https://paper-attachments.dropbox.com/s_B11EF229FFC176E08FB7E677870E42312ADC86B9923359B137487FF1777CDD72_1554939460449_image.png)

  b. 可以进行对地图的扫描匹配，这里全局地图坐标系基于地图比例尺被离散化。举例来说，x = 1.5m, y = 2.2m 可以是 x = 15, y = 22 如果每米表示成10个晶格。


    i.  考虑哟个激光扫描点的子集来节约计算时间。


    ii. [高级]自由空间坐标可以也在此时考虑。使用提供的 bresenham 函数。这将为每个射线提供一组晶格，但可能变得计算繁重。参见第三周的 example_bresenham.m 获取更多细节。


  c. 确定你的相关性打分函数。当激光雷达的自由空间测量和地图自由空间吻合，相关性分数会怎样提高？或者降低？你需要微调这些值。下面这个表格展示了可能的相关性更新的例子，其中的列显示了地图上晶格的状态，行显示了激光雷达读取的晶格状态。

|                | Map Free | Map Occupied |
| -------------- | -------- | ------------ |
| LIDAR Free     | +1       | -5           |
| LIDAR Occupied | -5       | +10          |



3. 粒子重新给予权重


  a. 现在，有了更新粒子和测量其与地图相关性的能力，你可以使用一组粒子来跟踪机器人的状态。一个粒子是一个位置和方向猜想的列向量(x, y, theta)，并被赋予一个权重。


  b. 从10个粒子开始微调，看对于这个特别的定位作业的最佳数字，也许100个粒子是最好的。请小心，更多的粒子意味着更多的计算时间！


  c. 对每一次激光雷达测量，你将计算每个粒子的相关性分数。在一开始，每个粒子是相等权重的，那么对于10个粒子，每个粒子将会有0.1的权重。新的权重将会是一个现有权重和相关性分数的乘积。请小心权重的比例，这些数字可能变得非常小！ 在每一轮重新标准化权重，确保权重总和等于1。


  d. 计算粒子的有效数目，如4.3节所讨论的。重新采样粒子，如果这个数目太低。在重新采样后，你将会复制（权重和位姿）高权重的粒子，但噪声将会基于运动模型加入，所以没有复制的采样会留下，而且下一个地图相关性步会重新计算这些粒子的权重。


## 执行


1. 完成下面这个函数，其接收一组参数：传感器数据，已知地图，并返回机器人的姿态和方向。

```matlab
    function [ myPose ] = particleLocalization(ranges, scanAngles, map, param)
```

2. 参数变量包含地图精度（param.resol），其形式为每米的晶格数，地图尺寸（param.size），形式为地图的晶格数，和机器人的起始位置（param.origin），形式为机器人在地图上的起始的晶格坐标。


3. 最终函数需要返回机器人的整个路径，是一个3xn的矩阵形式，其中n是激光雷达u观测的数目。在每n个观测中，你将追踪位置坐标x和y和方向角theta。


4. example_test.m提供来帮助你图象化输出你的结果。


## 解析


1. 首先来理解提供的变量意义。

在 example_test.m 一开始，先载入训练数据，

```matlab
    load practice.mat
    % This will load four variables: ranges, scanAngles, t, pose
    % [1] t is K-by-1 array containing time in second. (K=3701)
    %     You may not need to use time info for implementation.
    % [2] ranges is 1081-by-K lidar sensor readings.
    %     e.g. ranges(:,k) is the lidar measurement at time index k.
    % [3] scanAngles is 1081-by-1 array containing at what angles the 1081-by-1 lidar
    %     values ranges(:,k) were measured. This holds for any time index k. The
    %     angles are with respect to the body coordinate frame.
    % [4] M is a 2D array containing the occupancy grid map of the location
    %     e.g. map(x,y) is a log odds ratio of occupancy probability at (x,y)
```

t: 时间变量，Kx1维，其中K = 3701。

ranges: 1081xK的矩阵，表示激光雷达测量数据。那么1081显然是指各个方向的测距值。举例来说：ranges(:, k) 就是在时间k的激光雷达测量。

scanAngles: 激光雷达的1081个测量角度，显然这是一个1081x1矩阵。测量角度随时间是不变的。并且这些角度是以本地坐标系为参考坐标系。

M: 这是一个表示地图像素位置的二维矩阵。


2. 参数选择

```matlab
    %% Set parameters
    param = {};
    % 1. Decide map resolution, i.e., the number of grids for 1 meter.
    param.resol = 25;

    % 3. Indicate where you will put the origin in pixels
    param.origin = [685,572]';

    param.init_pose = -init_pose;

    %% Plot LIDAR data
    lidar_local = [ranges(:,1).*cos(scanAngles) -ranges(:,1).*sin(scanAngles)];
```

这里继续采用默认值。


3. 激光雷达数据示意图


![图3： 激光雷达测量，在本地坐标系下](https://paper-attachments.dropbox.com/s_B11EF229FFC176E08FB7E677870E42312ADC86B9923359B137487FF1777CDD72_1554943538608_lida_measurement.png)

4. particleLocalization()

```matlab
    N = size(ranges, 2);
    myPose = zeros(3, N); % Output format is [x1 x2, ...; y1, y2, ...; z1, z2, ...]
    % The initial pose is given
    myPose(:,1) = param.init_pose;
```

机器人位姿初始化，N是时间轴的分布值。其中初始位置是不需要更新的，可以认为是真实值。

```matlab
    M = 1000;                      % Please decide a reasonable number of M,
                                   % based on your experiment using the practice data.
```

粒子数选择，比如这里我选择了 M = 1000。可以考虑较小的值，以节约计算时间。

```matlab
    % Create M number of particles
    P = repmat(myPose(:,1), [1, M]);
    W = ones(1,M) * (1/M);
```

那么将粒子群围绕i机器人的位置初始化，将得到一个 3xM 的矩阵。相应的，每个粒子的初始权重是1/M。

```matlab
    ids = round(linspace(1, size(ranges, 1), 100));
```

从所有扫描射线角中平均地选取100个角度，以节约计算时间。

```matlab
    robotGrid = [myOrigin]; % for visualization

    figure
```

初始化一个变量记录机器人在地图上的轨迹，用于图像化显示。

```matlab
    for j = 2:N % You will start estimating myPose from j=2 using ranges(:,2).
        % 1) Propagate the particles
        P(3, :) = P(3,:) + randn(1, M) * 0.08;
        dist = 0.01 + randn(1, M) * 0.08;
        P(1, :) = P(1, :) + dist .* cos(P(3, :));
        P(2, :) = P(2, :) - dist .* sin(P(3, :));
```

对于每一个粒子，分别加入角度和距离的噪声。

```matlab
            % 2) Measurement Update
            % 2-1) Find grid cells hit by the rays (in the grid map coordinate frame)
            phi = P(3, m) + scanAngles(ids);

            x_occ = ranges(ids, j) .* cos(phi) + P(1, m);
            y_occ = -ranges(ids, j) .* sin(phi) + P(2, m);

            i_x_occ = ceil(x_occ * myResolution) + myOrigin(1);
            i_y_occ = ceil(y_occ * myResolution) + myOrigin(2);
```

对于每一个粒子，转换本地坐标系到全局坐标系，然后再到地图的像素坐标。

```matlab
            % 2-2) For each particle, calculate the correlation scores of the particles
            N_occ = size(i_x_occ, 1);
            corr = 0;
            for k = 1:N_occ
                if (i_x_occ(k) > size(map, 2)) || (i_x_occ(k) < 1)
                    continue;
                end
                if (i_y_occ(k) > size(map, 1)) || (i_y_occ(k) < 1)
                    continue;
                end

                if map(i_y_occ(k), i_x_occ(k)) < 0.4
                    corr = corr - 5 * map(i_y_occ(k), i_x_occ(k)); % for testing.mat
                    % corr = corr + map(i_y_occ(k), i_x_occ(k)) + 0.5; % for practice.mat
                end
                if map(i_y_occ(k), i_x_occ(k)) > 0.6
                    corr = corr + 10 * map(i_y_occ(k), i_x_occ(k)); % for testing.mat
                    % corr = corr + map(i_y_occ(k), i_x_occ(k)) + 0.5; % for practice.mat
                end

            end
```

这里遇到了一个陷阱，但不知是不是有意为之，你应该注意到了，在计算训练数据和测试数据时，我用了两种计算相关性分数的方法。被注释的方法，对训练数据有效，但无法通过测试数据。经过改进，现在采用的方法能够较好的匹配测量和地图数据。

这是因为训练数据和测试数据有所不同，训练数据的地图数据是在-0.5 - 0.5之间，而测试数据是在0 - 1之间。

```matlab
            % 2-3) Update the particle weights
            W(1,m) = W(1,m) * corr;
```

更新权重。

```matlab
        % 2-4) Choose the best particle to update the pose
        W_norm = W / sum(W);
        N_eff = floor(sum(W_norm)*sum(W_norm) / sum(W_norm .^ 2));
        [W_sorted, sorted_index] = sort(W_norm, 'descend');
        W_sample = W_sorted(1:N_eff);
        P_sorted = P(:, sorted_index);
        P_sample = P_sorted(:, 1:N_eff);
        myPose(:, j) = mean(P(:, 1:10), 2);
```

用最好的粒子来更新机器人位姿。这里其实是采用的是最好的10个粒子的平均值。用如下公式计算有效数目。


![图 3： 计算有效数目的公式](https://paper-attachments.dropbox.com/s_B11EF229FFC176E08FB7E677870E42312ADC86B9923359B137487FF1777CDD72_1555007982592_image.png)


```matlab
        % 3) Resample if the effective number of particles is smaller than a threshold
        p_ids = randsample(N_eff, M, 'true', W_sample); % 80
        W = W_sample(p_ids');
        P = P_sample(:, p_ids');
```

如果有效粒子不足，需要进行重采样。

```matlab
        % 4) Visualize the pose on the map as needed
        imagesc(map);
        hold on;
        axis equal;
        set(gca,'YDir','reverse');

        robot_grid = ceil([myPose(1,j); myPose(2,j)]*myResolution) + myOrigin;
        robotGrid = [robotGrid, robot_grid]; % dynamically show the moving trajectory
        plot(robotGrid(1,:),robotGrid(2,:),'r.-','LineWidth',2.5); % indicate start point

        lidar_global(:,1) =  (ranges(:,j).*cos(scanAngles + myPose(3,j)) + myPose(1,j))*myResolution + myOrigin(1);
        lidar_global(:,2) = (-ranges(:,j).*sin(scanAngles + myPose(3,j)) + myPose(2,j))*myResolution + myOrigin(2);
        plot(lidar_global(:,1), lidar_global(:,2), 'g.');

        hold off;
        drawnow;
```

这一部分是将每一步机器人的轨迹记录下来，并显示于图上，记住这里仍要进行如之前所做的左边转化。因为本函数计算得到的机器人位置和方向是基于全局坐标，需要先转换为地图像素坐标。

对任何问题，能够进行图像化显示，都将很有帮助。


https://www.dropbox.com/s/oixd00ppwaj36ph/particle-filter-pose-tracking.mp4?dl=0



https://www.youtube.com/watch?v=JKqo4VsC7RE&



## 总结

这次作业应该说考察了对粒子滤波器的深度理解，之前对重采样，只知道一个轮盘式的概率采样，这里最大的不同是重采样的方法不一样了。

另一个比较难的地方是，如何进行测量和地图的扫描匹配，一开始的方法，只能通过训练数据，不能通过测试数据，后来也是看了论坛才找到通关的方法。还需要更进一步理解。


## 参考
