---
title: 1254. Number of Closed Islands
description: Count islands fully surrounded by water.
tags:
  - leetcode
  - medium
  - graph
  - dfs
---

# 1254. Number of Closed Islands

## Problem

Given a 2D grid of `0`s (land) and `1`s (water). An island is a maximal 4-directionally connected group of `0`s. A closed island is an island totally surrounded by `1`s on all sides (left, top, right, bottom). Return the number of closed islands.

## Examples

**Example 1**

```text
Input: grid = [[1,1,1,1,1,1,1,0],[1,0,0,0,0,1,1,0],[1,0,1,0,1,1,1,0],[1,0,0,0,0,1,0,1],[1,1,1,1,1,1,1,0]]

Output: 2
```

Islands in gray are closed because they are completely surrounded by water (group of 1s).

**Example 2**

```text
Input: grid = [[0,0,1,0,0],[0,1,0,1,0],[0,1,1,1,0]]

Output: 1
```

**Example 3**

```text
Input: grid = [[1,1,1,1,1,1,1],
               [1,0,0,0,0,0,1],
               [1,0,1,1,1,0,1],
               [1,0,1,0,1,0,1],
               [1,0,1,1,1,0,1],
               [1,0,0,0,0,0,1],
               [1,1,1,1,1,1,1]]

Output: 2
```

## Constraints

- `1 <= grid.length, grid[0].length <= 100`
- `0 <= grid[i][j] <= 1`
