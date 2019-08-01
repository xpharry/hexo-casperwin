---
title: 代码赏析 | Rapidly-exploring Random Trees (RRTs)
date: 2019-07-26 12:00:27
tags:
---

Rapidly-exploring Random Trees (RRTs)

源论文：Steven M. LaVlle, Rapicly-Exploring Random Trees: A New Tool for Path Planning, 1998.

<!--more-->

From Wikipedia:

A rapidly exploring random tree (RRT) is an algorithm designed to efficiently search nonconvex, high-dimensional spaces by randomly building a space-filling tree. The tree is constructed incrementally from samples drawn randomly from the search space and is inherently biased to grow towards large unsearched areas of the problem. RRTs were developed by Steven M. LaValle and James J. Kuffner Jr. They easily handle problems with obstacles and differential constraints (nonholonomic and kinodynamic) and have been widely used in autonomous robotic motion planning.

RRTs can be viewed as a technique to generate open-loop trajectories for nonlinear systems with state constraints. An RRT can also be considered as a Monte-Carlo method to bias search into the largest Voronoi regions of a graph in a configuration space. Some variations can even be considered stochastic fractals.

Why RRT ?

- High dimensional planning w/ complex dynamic model
- Smooth maneuvering

For a general configuration space C, the algorithm in pseudocode is as follows:

```
Algorithm BuildRRT
  Input: Initial configuration qinit, number of vertices in RRT K, incremental distance Δq)
  Output: RRT graph G

  G.init(qinit)
  for k = 1 to K
    qrand ← RAND_CONF()
    qnear ← NEAREST_VERTEX(qrand, G)
    qnew ← NEW_CONF(qnear, qrand, Δq)
    G.add_vertex(qnew)
    G.add_edge(qnear, qnew)
  return G
```

代码源：https://github.com/AtsushiSakai/PythonRobotics

simple_rrt.py

```python
def Planning(self, animation=True):
    """
    Pathplanning

    animation: flag for animation on or off
    """

    self.nodeList = [self.start]
    while True:
        # Random Sampling
        if random.randint(0, 100) > self.goalSampleRate:
            rnd = [random.uniform(self.minrand, self.maxrand), random.uniform(
                self.minrand, self.maxrand)]
        else:
            rnd = [self.end.x, self.end.y]

        # Find nearest node
        nind = self.GetNearestListIndex(self.nodeList, rnd)
        # print(nind)

        # expand tree
        nearestNode = self.nodeList[nind]
        theta = math.atan2(rnd[1] - nearestNode.y, rnd[0] - nearestNode.x)

        newNode = copy.deepcopy(nearestNode)
        newNode.x += self.expandDis * math.cos(theta)
        newNode.y += self.expandDis * math.sin(theta)
        newNode.parent = nind

        if not self.__CollisionCheck(newNode, self.obstacleList):
            continue

        self.nodeList.append(newNode)
        print("nNodelist:", len(self.nodeList))

        # check goal
        dx = newNode.x - self.end.x
        dy = newNode.y - self.end.y
        d = math.sqrt(dx * dx + dy * dy)
        if d <= self.expandDis:
            print("Goal!!")
            break

        if animation:
            self.DrawGraph(rnd)

    path = [[self.end.x, self.end.y]]
    lastIndex = len(self.nodeList) - 1
    while self.nodeList[lastIndex].parent is not None:
        node = self.nodeList[lastIndex]
        path.append([node.x, node.y])
        lastIndex = node.parent
    path.append([self.start.x, self.start.y])

    return path
```
