---
layout:     post
title:      "Python AStar Demo"
subtitle:   "A*寻路演示"
date:       2019-03-01
author:     "leehp258"
header-img: "img/home-bg.jpg"
tags:
    - Python
    - astar
    - 算法
---

# A星寻路算法演示

> 源自一次技术分享，废话不多说，先看效果三连拍

![astar_demo1.png](https://leehp258.github.io/img/astar_demo1.png) ![astar_demo2.png](https://leehp258.github.io/img/astar_demo1.png) ![astar_demo3.png](https://leehp258.github.io/img/astar_demo1.png)

## 直接上代码
```python
# -*- coding: utf-8 -*-

import sys
import math
import time
import random
import curses

__author__ = "leehp258"

TILE = '.'
NPC = 'o'
HILL = '*'
GOLD = '$'

HILL_NUM = 100
START = (3, 3)
GOAL = (19, 18)
GRID = 20

class ANode():
    def __init__(self, point, g=float('inf'), f=float('inf')):
        self.point = point
        self.g = g
        self.f = f
        self.closed = False
        self.prev = None

    def __lt__(self, b):
        return self.f < b.f

class AStar():

    def __init__(self, grid):
        self.grid = grid
        self.width = len(grid[0])
        self.height = len(grid)
        self.nodes = self.init_nodes(grid)

    def init_nodes(self, grid):
        nodes = {}
        for y, row in enumerate(grid):
            for x, v in enumerate(row):
                point = (x, y)
                node = ANode(point)
                if v == HILL:
                    node.closed = True
                nodes[point] = node
        return nodes

    def estimate(self, node, goal):
        x1, y1 = node
        x2, y2 = goal
        return abs(x2 - x1)+abs(y2 - y1)

    def reverse_path(self, last):
        def _gen():
            current = last
            while current:
                yield current.point
                current = current.prev
        return reversed(list(_gen()))

    def get_neighbors(self, node):
        x, y = node.point
        neighbor_points = [(x, y - 1), (x, y + 1), (x - 1, y), (x + 1, y)]
        neighbors = []
        for i, j in neighbor_points:
            if 0 <= i < self.width and 0 <=j < self.height and self.grid[j][i] == TILE:
                neighbors.append(self.nodes[(i, j)])
        return neighbors

    def __call__(self, start, goal):
        if start == goal:
            return [start]
        start_node = ANode(start, 0, self.estimate(start, goal))

        open_set = set([start_node])
        while open_set:
            curr_node = min(open_set)
            open_set.remove(curr_node)
            if curr_node.point == goal:
                return self.reverse_path(curr_node)
            curr_node.closed = True
            for neighbor_node in self.get_neighbors(curr_node):
                if neighbor_node.closed:
                    continue
                g = curr_node.g + 1
                if neighbor_node.g <= g:
                    continue
                neighbor_node.prev = curr_node
                neighbor_node.g = g
                neighbor_node.f = g + self.estimate(neighbor_node.point, goal)
                if neighbor_node not in open_set:
                    open_set.add(neighbor_node)

class AMap():
    def __init__(self):
        self.grid = []
        for i in range(GRID):
            self.grid.append([TILE]*GRID)

    def demo_ramdom(self):
        for i in range(HILL_NUM):
            x = random.randint(1,GRID-1)
            y = random.randint(1,GRID-1)
            if (x, y) == START or (x, y) == GOAL:
                continue
            self.grid[y][x] = HILL

    def demo1(self):
        self.range_x(10, [8])

    def demo2(self):
        self.range_y(9, [13])

    def demo3(self):
        self.range_x(8, [2])
        self.range_y(12, [10, 18])
        self.range_x(15, [7,8,9,15])

    def demo4(self):
        self.range_x(6, [2])
        self.range_y(12, [6, 18])
        self.range_x(10, [9,10,15])
        self.range_x(16, [1,2,15,16])

    def range_y(self, y, gaps=[0]):
        for i in range(GRID):
            self.grid[i][y] = HILL
        for gap in gaps:
            self.grid[gap][y] = TILE

    def range_x(self, x, gaps=[0]):
        for i in range(0,GRID):
            self.grid[x][i] = HILL
        for gap in gaps:
            self.grid[x][gap] = TILE

def path_render(grid, path):
    grid[START[1]][START[0]] = NPC
    grid[GOAL[1]][GOAL[0]] = GOLD
    if not path:
        yield grid
        return
    for p in path:
        for y, row in enumerate(grid):
            for x, v in enumerate(row):
                if p == (x, y):
                    row[x] = NPC
        yield grid

_map = AMap()
# _map.demo1()
# _map.demo2()
# _map.demo3()
# _map.demo4()
_map.demo_ramdom()

a = AStar(_map.grid)
path = a(START, GOAL)

def grid2str(grid):
    rtn = []
    for row in grid:
        rtn.append(''.join(map(str, row)))
        rtn.append('\n')
    return ''.join(rtn)

render = path_render(_map.grid, path)
def loop(window):
    i = 0
    for frame in render:
        window.addstr(0, 0, grid2str(frame))
        window.refresh()
        i+=1
        time.sleep(0.5)
    time.sleep(30)
curses.wrapper(loop)

```
