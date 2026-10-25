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

## Solution

> ###### Go (2026-10-25)

```go
func closedIsland(grid [][]int) int {
	rows, cols := len(grid), len(grid[0])

	var exploreIsland func(int, int) bool
	exploreIsland = func(row, col int) bool {
		if row < 0 || row >= rows || col < 0 || col >= cols {
			return false
		}
		if grid[row][col] == 1 {
			return true
		}

		grid[row][col] = 1
		isClosed := true

		isClosed = exploreIsland(row-1, col) && isClosed
		isClosed = exploreIsland(row+1, col) && isClosed
		isClosed = exploreIsland(row, col-1) && isClosed
		isClosed = exploreIsland(row, col+1) && isClosed

		return isClosed
	}

	closedIslandCount := 0
	for row := 0; row < rows; row++ {
		for col := 0; col < cols; col++ {
			if grid[row][col] == 0 && exploreIsland(row, col) {
				closedIslandCount++
			}
		}
	}

	return closedIslandCount
}
```

## How the Solution Works

### 1. Get the grid dimensions

```go
rows, cols := len(grid), len(grid[0])
```

`rows` and `cols` let us check whether a position is inside the grid. The problem constraints guarantee at least one row and one column.

### 2. Create a recursive island explorer

```go
var exploreIsland func(int, int) bool
exploreIsland = func(row, col int) bool { ... }
```

This helper explores the island connected to `(row, col)`. It returns `true` if the island is closed and `false` if any part reaches outside the grid. We declare the function variable first because the function calls itself.

### 3. Detect when the island reaches the boundary

```go
if row < 0 || row >= rows || col < 0 || col >= cols {
	return false
}
```

If exploration steps outside the grid, the island touches the boundary, so it is not closed.

### 4. Stop at water and mark land as visited

```go
if grid[row][col] == 1 {
	return true
}

grid[row][col] = 1
```

Water does not connect the island to the boundary, so it returns `true` for this path. Land is changed to water before exploring further. This marks it visited and prevents the recursive calls from revisiting it indefinitely.

### 5. Explore all four directions

```go
isClosed = exploreIsland(row-1, col) && isClosed
isClosed = exploreIsland(row+1, col) && isClosed
isClosed = exploreIsland(row, col-1) && isClosed
isClosed = exploreIsland(row, col+1) && isClosed
```

The calls visit the cells above, below, left, and right. The results are combined so that one boundary-reaching cell makes the entire island open. Keeping `exploreIsland(...)` on the left of `&&` is important: Go evaluates it on every line, even when `isClosed` is already `false`, so the rest of the island still gets visited.

### 6. Scan the grid and count closed islands

```go
if grid[row][col] == 0 && exploreIsland(row, col) {
	closedIslandCount++
}
```

The nested loops check every cell. When they find unvisited land, the helper explores and marks that whole island. The count increases only if the helper reports that it is closed. Since the island has been marked visited, it will not be counted again.

### 7. Return the count

```go
return closedIslandCount
```

After every cell has been checked, this is the total number of closed islands.


