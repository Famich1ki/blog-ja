---
title: Leetcode Review
date: 2024-10-15 22:53:51
tags:
  - Data Structure and Algorithm
  - Leetcode
categories:
  - Data Structure and Algorithm
  - Leetcode
cover: https://pics.findfuns.org/leetcode.jpg
---
# [399. Evaluate Division](https://leetcode.com/problems/evaluate-division/)

Problem Description:

You are given an array of variable pairs `equations` and an array of real numbers `values`, where `equations[i] = [Ai, Bi]` and `values[i]` represent the equation `Ai / Bi = values[i]`. Each `Ai` or `Bi` is a string that represents a single variable.

You are also given some `queries`, where `queries[j] = [Cj, Dj]` represents the `jth` query where you must find the answer for `Cj / Dj = ?`.

Return *the answers to all queries*. If a single answer cannot be determined, return `-1.0`.

**Note:** The input is always valid. You may assume that evaluating the queries will not result in division by zero and that there is no contradiction.

**Note:** The variables that do not occur in the list of equations are undefined, so the answer cannot be determined for them.

**Example 1:**

```
Input: equations = [["a","b"],["b","c"]], values = [2.0,3.0], queries = [["a","c"],["b","a"],["a","e"],["a","a"],["x","x"]]
Output: [6.00000,0.50000,-1.00000,1.00000,-1.00000]
Explanation: 
Given: a / b = 2.0, b / c = 3.0
queries are: a / c = ?, b / a = ?, a / e = ?, a / a = ?, x / x = ? 
return: [6.0, 0.5, -1.0, 1.0, -1.0 ]
note: x is undefined => -1.0
```

**Example 2:**

```
Input: equations = [["a","b"],["b","c"],["bc","cd"]], values = [1.5,2.5,5.0], queries = [["a","c"],["c","b"],["bc","cd"],["cd","bc"]]
Output: [3.75000,0.40000,5.00000,0.20000]
```

**Example 3:**

```
Input: equations = [["a","b"]], values = [0.5], queries = [["a","b"],["b","a"],["a","c"],["x","y"]]
Output: [0.50000,2.00000,-1.00000,-1.00000]
```

<img src="https://pics.findfuns.org/Problem 399.png" style="zoom:50%;" />

This is a typical graph-related problem. The idea is to construct an adjacency matrix `matrix`, where `matrix[i][j]` represents the value of `i / j`. If there is no such value, it is set to 0.

After constructing the matrix, we can use DFS to recursively search for the relationship. For example, in Example 1, to compute `a / c`, we first find `a / b` through DFS, then continue to find `b / c`, and multiply them to obtain `a / c`.

The detailed implementation is as follows:

```java
class Solution {
    Map<String, Integer> map;
    double[][] matrix;
    int cnt = 0;
    double[] res;
    public double[] calcEquation(List<List<String>> equations, double[] values, List<List<String>> queries) {
        map = new HashMap<>();
        for(List<String> list: equations) {
            for(String str: list) {
                if(!map.containsKey(str)) {
                    map.put(str, cnt ++);
                }
            }
        }
        int num = map.size();
        matrix = new double[num][num];
        for(int i = 0; i < num; i ++) {
            matrix[i][i] = 1.0;
        }
        for(int i = 0; i < equations.size(); i ++) {
            String from = equations.get(i).get(0);
            String to = equations.get(i).get(1);
            matrix[map.get(from)][map.get(to)] = values[i];
            matrix[map.get(to)][map.get(from)] = 1.0 / values[i];
        }
        res = new double[queries.size()];
        cnt = 0;
        for(List<String> list: queries) {
            if(!map.containsKey(list.get(0)) || !map.containsKey(list.get(1))) {
                res[cnt] = -1.0;
            } else {
                int from = map.get(list.get(0));
                int to = map.get(list.get(1));
                dfs(from, to, 1.0, new HashSet<>());
                if(res[cnt] == 0) {
                    res[cnt] = -1.0;
                }
            }   
            cnt ++;
        }
        return res;
    }
    private void dfs(int from, int to, double multiple, Set<Integer> set) {
        if(set.contains(from)) {
            return;
        }
        for(int i = 0; i < matrix[from].length; i ++) {
            if(i == to && matrix[from][to] != 0.0) {
                res[cnt] = multiple * matrix[from][to];
                return;
            }
            if(matrix[from][i] != 0.0) {
                set.add(from);
                dfs(i, to, multiple * matrix[from][i], set);
                set.remove(from);
            }
        }
    }
}
```

Time Complexity Analysis:

- Building the map (mapping strings to matrix indices) takes `O(E)`, where `E` is the number of equations.

- Constructing the adjacency matrix takes `O(E)`.

- In the worst case, the DFS method may traverse every value in the adjacency matrix. Suppose there are `n` distinct variables, so the matrix contains `n^2` values. The time complexity of DFS is `O(n^2)`.

- When iterating through `queries`, suppose there are `Q` queries. Each query calls DFS once, so the time complexity is `O(Q * n^2)`.

Overall time complexity is `O(E + Q * n^2)`.

Space Complexity Analysis:

- The matrix contains `n^2` entries, so the complexity is `O(n^2)`.

- The map stores `n` entries, so the complexity is `O(n)`.

- The recursion depth of DFS may reach all nodes, so the complexity is `O(n)`.

- The set used in DFS may contain all nodes in the worst case, so the complexity is `O(n)`.

- The result array `res` contains `Q` entries, so the complexity is `O(Q)`.

Overall space complexity is `O(Q + n^2)`.



# [994. Rotting Oranges](https://leetcode.com/problems/rotting-oranges/)

Problem Description:

![](https://pics.findfuns.org/Problem 994.png)



The idea is to first traverse the grid. If we find a rotten orange, we add it to a `Queue`. After the traversal is complete, we perform BFS on all rotten oranges in the queue and record the time for each BFS step, dynamically updating the maximum time. Finally, we scan the grid again. If there are still fresh oranges left, return -1; otherwise, return the maximum time.

The code is as follows:

```java
class Solution {
    public int orangesRotting(int[][] grid) {
        Queue<int[]> queue = new LinkedList<>();
        int res = 0;
        for(int i = 0; i < grid.length; i ++) {
            for(int j = 0; j < grid[i].length; j ++) {
                if(grid[i][j] == 2) {
                    queue.offer(new int[] {i, j, 0});
                }
            }
        }
        int[][] directions = new int[][] {{0, 1}, {0, -1}, {1, 0}, {-1, 0}};
        while(!queue.isEmpty()) {
            int[] cur = queue.poll();
            for(int[] direction: directions) {
                int x = cur[0] + direction[0];
                int y = cur[1] + direction[1];
                if(x < 0 || x >= grid.length || y < 0 || y >= grid[x].length || grid[x][y] == 2 || grid[x][y] == 0) {
                    res = Math.max(res, cur[2]);
                } else {
                    grid[x][y] = 2;
                    queue.offer(new int[] {x, y, cur[2] + 1});
                }
            }
        }
        for(int i = 0; i < grid.length; i ++) {
            for(int j = 0; j < grid[i].length; j ++) {
                if(grid[i][j] == 1) {
                    return -1;
                }
            }
        }
        return res;
    }
}
```

Time Complexity Analysis:

- The two nested `for` loops take `O(n*m)`, where `n` and `m` are the number of rows and columns of the grid.
- In the worst case, BFS traverses every cell once, so the time complexity is `O(n*m)`.

Overall time complexity: `O(n*m)`

Space Complexity Analysis:

- In the worst case, the queue may contain all cells, so the space complexity is `O(n*m)`.

Overall space complexity: `O(n*m)`



# [875. Koko Eating Bananas](https://leetcode.com/problems/koko-eating-bananas/)

Koko loves to eat bananas. There are `n` piles of bananas, and the `ith` pile has `piles[i]` bananas. The guards have gone and will come back in `h` hours.

Koko can decide her bananas-per-hour eating speed `k`. Each hour, she chooses one pile of bananas and eats `k` bananas from that pile. If the pile has fewer than `k` bananas, she eats all of them and will not eat any more bananas during that hour.

Koko likes to eat slowly but still wants to finish all the bananas before the guards return.

Return *the minimum integer* `k` *such that she can eat all the bananas within* `h` *hours*.



**Example 1:**

```
Input: piles = [3,6,7,11], h = 8
Output: 4
```

**Example 2:**

```
Input: piles = [30,11,23,4,20], h = 5
Output: 30
```

**Example 3:**

```
Input: piles = [30,11,23,4,20], h = 6
Output: 23
```

We can use binary search to continuously narrow down the minimum valid eating speed until we find the smallest possible speed.

The code is as follows:

```java
class Solution {
    public int minEatingSpeed(int[] piles, int h) {
        int slow = 1;
        int fast = 1000000000;
        int result = -1;
        while(slow <= fast) {
            int mid = slow + (fast - slow) / 2;
            if(check(mid, piles, h)) {
                result = mid;
                fast = mid - 1;
            } else {
                slow = mid + 1;
            }
        }
        return result;
    }

    private boolean check(int speed, int[] piles, int h) {
        long hours = 0;
        for(Integer pile: piles) {
            hours += pile % speed == 0 ? pile / speed : pile / speed + 1;
        }
        if(hours <= h) {
            return true;
        }
        return false;
    }
}
```

Since the maximum number of bananas in a single pile is `10^9`, we can set the maximum possible speed to `10^9` and the minimum speed to `1`. Then we apply binary search to continuously narrow the range.

It is important to note that in the `check()` method, the variable `hours` must be declared as `long` to avoid overflow.

Time Complexity Analysis:

- Let `n` be the length of `piles`. Each binary search step requires traversing the `piles` array once, which takes `O(n)`.
- The search range is from `1` to `10^9`, and binary search runs in `O(log 10^9)` time.
- Therefore, the overall time complexity is `O(n log 10^9)`.

Space Complexity:

- `O(1)`





# [962. Maximum Width Ramp](https://leetcode.com/problems/maximum-width-ramp/)

A **ramp** in an integer array `nums` is a pair `(i, j)` for which `i < j` and `nums[i] <= nums[j]`. The **width** of such a ramp is `j - i`.

Given an integer array `nums`, return *the maximum width of a **ramp** in* `nums`. If there is no **ramp** in `nums`, return `0`.

**Example 1:**

```
Input: nums = [6,0,8,2,1,5]
Output: 4
Explanation: The maximum width ramp is achieved at (i, j) = (1, 5): nums[1] = 0 and nums[5] = 5.
```

**Example 2:**

```
Input: nums = [9,8,1,0,1,9,4,0,4,1]
Output: 7
Explanation: The maximum width ramp is achieved at (i, j) = (2, 9): nums[2] = 1 and nums[9] = 1.
```

In simple terms, this problem asks us to find, for each element, the farthest element to its right that is greater than or equal to it, and then compute the maximum possible distance.

The specific approach is to maintain a **monotonically decreasing stack** while traversing from left to right. Then, traverse from right to left. If the value at the top index of the stack is less than or equal to the current element, pop it from the stack and update the maximum width. Continue this process until the stack becomes empty.

The code is as follows:

```java
class Solution {
    public int maxWidthRamp(int[] nums) {
        Deque<Integer> stack = new LinkedList<>();
        for(int i = 0; i < nums.length; i ++) {
            if(stack.isEmpty() || nums[stack.peekLast()] > nums[i]) {
                stack.offerLast(i);
            }
        }

        int res = 0;
        for(int i = nums.length - 1; i >= 0; i --) {
            while(!stack.isEmpty() && nums[stack.peekLast()] <= nums[i]) {
                res = Math.max(res, i - stack.pollLast());
            }
        } 
        return res;
    }
}
```

## Time Complexity Analysis

- Maintaining the monotonic stack requires traversing each element once: `O(n)`.
- The second traversal from right to left processes each element at most once in the worst case: `O(n)`.

Therefore, the overall time complexity is `O(n)`.

## Space Complexity Analysis

- In the worst case, the `stack` stores all indices: `O(n)`.

Therefore, the space complexity is `O(n)`.

# [2406. Divide Intervals Into Minimum Number of Groups](https://leetcode.com/problems/divide-intervals-into-minimum-number-of-groups/)

You are given a 2D integer array `intervals` where `intervals[i] = [lefti, righti]` represents the **inclusive** interval `[lefti, righti]`.

You have to divide the intervals into one or more **groups** such that each interval is in **exactly** one group, and no two intervals that are in the same group **intersect** each other.

Return *the **minimum** number of groups you need to make*.

Two intervals **intersect** if there is at least one common number between them. For example, the intervals `[1, 5]` and `[5, 8]` intersect.

**Example 1:**

```
Input: intervals = [[5,10],[6,8],[1,5],[2,3],[1,10]]
Output: 3
Explanation: We can divide the intervals into the following groups:
- Group 1: [1, 5], [6, 8].
- Group 2: [2, 3], [5, 10].
- Group 3: [1, 10].
It can be proven that it is not possible to divide the intervals into fewer than 3 groups.
```

**Example 2:**

```
Input: intervals = [[1,3],[5,6],[8,10],[11,13]]
Output: 1
Explanation: None of the intervals overlap, so we can put all of them in one group.
```

这道题可以用时间线和事件的思路去解决，不必拘泥于时间对，而是把每一个时间对拆开，将其视为开始时间和结束时间。

同时使用一个最小堆对时间进行排序，当存在开始时间和结束时间相同时**先处理开始时间**。维护一个变量记录同时存在的事件的最大数量，即为答案。

代码如下：

```java
class Solution {
    public int minGroups(int[][] intervals) {
        Queue<int[]> events = new PriorityQueue<>((a, b) -> {
            if(a[0] == b[0]) {
                return b[1] - a[1];
            }
            return a[0] - b[0];
        });
        for(int[] interval: intervals) {
            events.offer(new int[] {interval[0], 1}); // 1 is starting
            events.offer(new int[] {interval[1], -1}); // 0 is ending
        }

        int cur = 0;
        int res = 0;
        while(!events.isEmpty()) {
            int[] curEvent = events.poll();
            cur += curEvent[1];
            res = Math.max(cur, res);
        }
        return res;
    }
}
```

时间复杂度分析：

- 假设intervals有n个事件，那么一共有2n个时间点，插入堆的复杂度为`O(log2n)`,综合复杂度为`O(2nlog(2n)) = O(nlogn)`。

空间复杂度分析：

- 堆需要`O(2n) = O(n)`的空间

# [632. Smallest Range Covering Elements from K Lists](https://leetcode.com/problems/smallest-range-covering-elements-from-k-lists/)

You have `k` lists of sorted integers in **non-decreasing order**. Find the **smallest** range that includes at least one number from each of the `k` lists.

We define the range `[a, b]` is smaller than range `[c, d]` if `b - a < d - c` **or** `a < c` if `b - a == d - c`.

---

**Example 1:**

```
Input: nums = [[4,10,15,24,26],[0,9,12,20],[5,18,22,30]]
Output: [20,24]
Explanation: 
List 1: [4, 10, 15, 24, 26], 24 is in range [20,24].
List 2: [0, 9, 12, 20], 20 is in range [20,24].
List 3: [5, 18, 22, 30], 22 is in range [20,24].
```

**Example 2:**

```
Input: nums = [[1,2,3],[1,2,3],[1,2,3]]
Output: [1,1]
```

When a problem involves finding a “maximum” or “minimum”, it is often a good idea to consider using a **heap**, because we frequently need to retrieve the smallest or largest value efficiently. A heap allows us to get the minimum (or maximum) in `O(1)` time and update it in `O(log k)` time, which helps reduce the overall time complexity.

At each step, we put one element from each list into the heap. We remove the minimum element, dynamically maintain the current maximum value, compute the current range, and update the result if necessary. Then we push the next element from the same list as the removed element into the heap. We repeat this process until the heap size becomes smaller than `k`. The smallest recorded range is the final answer.

```java
class Solution {
    public int[] smallestRange(List<List<Integer>> nums) {
        Queue<int[]> pq = new PriorityQueue<>((a, b) -> {
            return a[0] - b[0];
        });
        int max = Integer.MIN_VALUE;
        for(int i = 0; i < nums.size(); i ++) {
            pq.offer(new int[] {nums.get(i).get(0), i, 0});
            max = Math.max(max, nums.get(i).get(0));
        }
        int[] res = new int[2];
        int range = Integer.MAX_VALUE;
        while(pq.size() == nums.size()) {
            int[] cur = pq.poll();
            int min = cur[0];
            if(range > max - min) {
                range = max - min;
                res[0] = min;
                res[1] = max;
            }

            if(cur[2] < nums.get(cur[1]).size() - 1) {
                pq.offer(new int[] {nums.get(cur[1]).get(cur[2] + 1), cur[1], cur[2] + 1});
                max = Math.max(max, nums.get(cur[1]).get(cur[2] + 1));
            }
        }
        return res;
    }
}
```

```
The step-by-step process for Example 1 is as follows:

heap: 0 4 5
max: 5
min: 0
range: 5

heap: 4 5 9
max: 9
min: 4
range: 5

heap: 5 9 10
max: 10
min: 5
range: 5

heap: 9 10 18
max: 18
min: 9
range: 9

heap: 10 12 18
max: 18
min: 10
range: 8

heap: 12 15 18
max: 18
min: 12 
range: 6

heap: 15 18 20
min: 15
max: 20
range: 5

heap: 18 20 24
min: 18
max: 24
range: 6

heap: 20 22 24
min: 20
max: 24
range: 4
```

## Time Complexity Analysis

- Suppose there are `k` lists and a total of `N` elements.
- In the worst case, every element is pushed into and popped from the heap once.
- Each heap operation costs `O(log k)`.

Therefore, the total time complexity is `O(N log k)`.

## Space Complexity Analysis

- The heap always contains at most `k` elements.

Therefore, the space complexity is `O(k)`.

# [1942. The Number of the Smallest Unoccupied Chair](https://leetcode.com/problems/the-number-of-the-smallest-unoccupied-chair/)

There is a party where `n` friends numbered from `0` to `n - 1` are attending. There is an **infinite** number of chairs in this party that are numbered from `0` to `infinity`. When a friend arrives at the party, they sit on the unoccupied chair with the **smallest number**.

- For example, if chairs `0`, `1`, and `5` are occupied when a friend comes, they will sit on chair number `2`.

When a friend leaves the party, their chair becomes unoccupied at the moment they leave. If another friend arrives at that same moment, they can sit in that chair.

You are given a **0-indexed** 2D integer array `times` where `times[i] = [arrivali, leavingi]`, indicating the arrival and leaving times of the `ith` friend respectively, and an integer `targetFriend`. All arrival times are **distinct**.

Return *the **chair number** that the friend numbered* `targetFriend` *will sit on*.

---

**Example 1:**

```
Input: times = [[1,4],[2,3],[4,6]], targetFriend = 1
Output: 1
Explanation: 
- Friend 0 arrives at time 1 and sits on chair 0.
- Friend 1 arrives at time 2 and sits on chair 1.
- Friend 1 leaves at time 3 and chair 1 becomes empty.
- Friend 0 leaves at time 4 and chair 0 becomes empty.
- Friend 2 arrives at time 4 and sits on chair 0.
Since friend 1 sat on chair 1, we return 1.
```

**Example 2:**

```
Input: times = [[3,10],[1,5],[2,6]], targetFriend = 0
Output: 2
Explanation: 
- Friend 1 arrives at time 1 and sits on chair 0.
- Friend 2 arrives at time 2 and sits on chair 1.
- Friend 0 arrives at time 3 and sits on chair 2.
- Friend 1 leaves at time 5 and chair 0 becomes empty.
- Friend 2 leaves at time 6 and chair 1 becomes empty.
- Friend 0 leaves at time 10 and chair 2 becomes empty.
Since friend 0 sat on chair 2, we return 2.
```

## Approach

- We can split each person's time into two events: an arrival event and a leaving event.  
  Store them in a min-heap as arrays:
    - `arr[0]` → time
    - `arr[1]` → friend number
    - `arr[2]` → event type (1 = arrival, 0 = leaving)  
      Sort primarily by time.

- Use another priority queue `availableChairs` to keep track of currently available chairs, so we can always obtain the smallest available chair.

- Use a `Map` called `occupiedChairs` to record which chair each friend is currently using.
    - Key → friend number
    - Value → chair number

### Process

Traverse all events in chronological order:

- If it is an **arrival event**:
    - Take the smallest chair from `availableChairs`
    - If the friend is `targetFriend`, return the chair immediately
    - Otherwise, record it in `occupiedChairs`

- If it is a **leaving event**:
    - Retrieve the chair from `occupiedChairs`
    - Put the chair back into `availableChairs`

---

```java
class Solution {
    public int smallestChair(int[][] times, int targetFriend) {
      	
        Queue<int[]> events = new PriorityQueue<>((a, b) -> {
            if(a[0] == b[0]) {
                return a[2] - b[2];
            }
            return a[0] - b[0];
        });

        Queue<Integer> avaliableChairs = new PriorityQueue<>();
        Map<Integer, Integer> occupiedChairs = new HashMap<>();

        for(int i = 0; i < times.length; i ++) {
            events.offer(new int[] {times[i][0], i, 1}); // arrival
            events.offer(new int[] {times[i][1], i, 0}); // leaving
        }

        for(int i = 0; i < times.length; i ++) {
            avaliableChairs.offer(i);
        }

        while(!events.isEmpty()) {
            int[] cur = events.poll();
            int time = cur[0];
            int number = cur[1];
            int event = cur[2];

            if(event == 1) { // arrival
                int chair = avaliableChairs.poll();
                if(number == targetFriend) {
                    return chair;
                }
                occupiedChairs.put(number, chair);
            } else { // leaving
                int chair = occupiedChairs.get(number);
                avaliableChairs.offer(chair);
            }
        }
        return -1;
    }
}
```

## Time Complexity Analysis

- Suppose there are `n` friends.
- We insert `2n` events into the heap. Each insertion costs `O(log n)`, so this part costs `O(n log n)`.
- Adding all chairs into `availableChairs` costs `O(n log n)`.
- Processing each event:
    - Polling from `availableChairs` costs `O(log n)`
    - Inserting into `occupiedChairs` costs `O(1)`
    - Overall worst-case cost is `O(n log n)`

Therefore, the total time complexity is:

```
O(n log n)
```

## Space Complexity Analysis

- `events` requires `O(2n)`
- `availableChairs` requires up to `O(n)`
- `occupiedChairs` requires up to `O(n)`

Therefore, the overall space complexity is:

```
O(n)
```



# [**632. Smallest Range Covering Elements from K Lists**](https://leetcode.com/problems/smallest-range-covering-elements-from-k-lists/)

You have `k` lists of sorted integers in **non-decreasing order**. Find the **smallest** range that includes at least one number from each of the `k` lists.

We define the range `[a, b]` is smaller than range `[c, d]` if `b - a < d - c` **or** `a < c` if `b - a == d - c`.

---

**Example 1:**

```plain
Input: nums = [[4,10,15,24,26],[0,9,12,20],[5,18,22,30]]
Output: [20,24]
Explanation: 
List 1: [4, 10, 15, 24,26], 24 is in range [20,24].
List 2: [0, 9, 12, 20], 20 is in range [20,24].
List 3: [5, 18, 22, 30], 22 is in range [20,24].
```

**Example 2:**

```plain
Input: nums = [[1,2,3],[1,2,3],[1,2,3]]
Output: [1,1]
```

---

**Constraints:**

- `nums.length == k`
- `1 <= k <= 3500`
- `1 <= nums[i].length <= 50`
- `-10^5 <= nums[i][j] <= 10^5`
- `nums[i]` is sorted in **non-decreasing** order.

---

## Approach

The core of this problem is to ensure that the selected range always contains **at least one element from each list**.

To achieve this, we maintain a container that always holds exactly **k** elements (where `k` is the number of lists). During traversal, we must repeatedly obtain the current minimum and maximum values to determine the range, so using a **min-heap** is the natural choice.

- **Initialization**:  
  Push the first element of each list into the heap and record the current maximum value.

- **Traversal**:  
  Use a `while` loop. Each time:
    - Remove the smallest element from the heap.
    - Update the current minimum.
    - Compare and update the smallest range if needed.

- After removing the smallest element, insert the next element from the same list (if it exists) into the heap, and update the maximum value.

- When the loop ends, the recorded range is the answer.

---

```java
class Solution {
    public int[] smallestRange(List<List<Integer>> nums) {
        Queue<int[]> q = new PriorityQueue<>((a, b) -> {
            return a[0] - b[0];
        }); 
        int max = Integer.MIN_VALUE;
        for(int i = 0; i < nums.size(); i ++) {
            q.offer(new int[] {nums.get(i).get(0), i, 0});
            // arr[0] for value, arr[1] for index i of nums[i], arr[2] for index j of nums[i][j]
            max = Math.max(max, nums.get(i).get(0));
        }
        int len = Integer.MAX_VALUE;
        int[] res = new int[2];
        while(q.size() == nums.size()) {
            int[] cur = q.poll();
            int min = cur[0];
            int i = cur[1];
            int j = cur[2];
            if(len > max - min) {
                res[0] = min;
                res[1] = max;
                len = max - min;
            }
            if(j != nums.get(i).size() - 1) {
                q.offer(new int[] {nums.get(i).get(j + 1), i, j + 1});
                max = Math.max(max, nums.get(i).get(j + 1));
            }
        }
        return res;
    }
}
```

---

## Time Complexity Analysis

- Suppose there are `N` total elements across all lists.
- In the worst case, we traverse all elements.
- Each element is pushed into and popped from the heap at most once.
- The heap size is always `k`.
- Insertion and heap maintenance cost `O(log k)`.

Therefore, the total time complexity is:

```
O(N log k)
```

---

## Space Complexity Analysis

- The heap contains at most `k` elements.

Therefore, the space complexity is:

```
O(k)
```

# [2406. Divide Intervals Into Minimum Number of Groups](https://leetcode.com/problems/divide-intervals-into-minimum-number-of-groups/)

You are given a 2D integer array `intervals` where `intervals[i] = [lefti, righti]` represents the **inclusive** interval `[lefti, righti]`.

You have to divide the intervals into one or more **groups** such that each interval is in **exactly** one group, and no two intervals that are in the same group **intersect** each other.

Return *the* ***minimum*** *number of groups you need to make*.

Two intervals **intersect** if there is at least one common number between them. For example, the intervals `[1, 5]` and `[5, 8]` intersect.

---

**Example 1:**

```plain
Input: intervals = [[5,10],[6,8],[1,5],[2,3],[1,10]]
Output: 3
Explanation: We can divide the intervals into the following groups:
- Group 1: [1, 5], [6, 8].
- Group 2: [2, 3], [5, 10].
- Group 3: [1, 10].
It can be proven that it is not possible to divide the intervals into fewer than 3 groups.
```

**Example 2:**

```plain
Input: intervals = [[1,3],[5,6],[8,10],[11,13]]
Output: 1
Explanation: None of the intervals overlap, so we can put all of them in one group.
```

---

**Constraints:**

- `1 <= intervals.length <= 10^5`
- `intervals[i].length == 2`
- `1 <= lefti <= righti <= 10^6`

---

## Approach

We can visualize a timeline and draw each interval as a segment on it.  
The time point with the maximum number of overlapping intervals corresponds to the **minimum number of groups** required.

- Split each interval into a **start event** and an **end event**, and push them into a min-heap.
- Traverse all events in chronological order.
- While traversing, maintain the number of currently active intervals.
- The maximum number of simultaneously active intervals is the answer.

---

```java
class Solution {
    public int minGroups(int[][] intervals) {
        Queue<int[]> q = new PriorityQueue<>((a, b) -> {
            return a[0] - b[0];
        });
        for(int[] arr: intervals) {
            q.offer(new int[] {arr[0], 0}); // start
            q.offer(new int[] {arr[1], 1}); // end
        }
        int overlapped = 0;
        int mostOverlapped = 0;
        while(!q.isEmpty()) {
            int[] cur = q.poll();
            if(cur[1] == 0) {
                mostOverlapped = Math.max(mostOverlapped, ++overlapped);
            } else {
                overlapped--;
            }
        }
        return mostOverlapped;
    }
}
```

---

## Time Complexity Analysis

- Suppose there are `N` intervals.
- We insert `2N` elements into the heap and remove `2N` elements.
- Each heap operation costs `O(log(2N))`.

Therefore, the total time complexity is:

```
O(N log N)
```

---

## Space Complexity Analysis

- The heap contains at most `2N` elements.

Therefore, the space complexity is:

```
O(N)
```



# [962. Maximum Width Ramp](https://leetcode.com/problems/maximum-width-ramp/)

A **ramp** in an integer array `nums` is a pair `(i, j)` such that `i < j` and `nums[i] <= nums[j]`. The **width** of the ramp is `j - i`.

Given an integer array `nums`, return *the maximum width of a* ***ramp*** *in* `nums`. If there is no ramp in `nums`, return `0`.

---

## Example 1

```plain
Input: nums = [6,0,8,2,1,5]
Output: 4
Explanation: The maximum width ramp is achieved at (i, j) = (1, 5): 
nums[1] = 0 and nums[5] = 5.
```

## Example 2

```plain
Input: nums = [9,8,1,0,1,9,4,0,4,1]
Output: 7
Explanation: The maximum width ramp is achieved at (i, j) = (2, 9): 
nums[2] = 1 and nums[9] = 1.
```

---

## Constraints

- `2 <= nums.length <= 5 * 10^4`
- `0 <= nums[i] <= 5 * 10^4`

---

# Method 1: Sorting

## Idea

Bind each element with its index, then sort them in ascending order by value.

After sorting:

- If `nums[i] <= nums[j]`, then `(nums[i], i)` must appear before `(nums[j], j)` in the sorted array.
- While traversing the sorted array, maintain the smallest index seen so far (`minIndex`).
- For each element:
    - Update the maximum width using `index - minIndex`.
    - Update `minIndex`.

---

```java
class Solution {
    public int maxWidthRamp(int[] nums) {
        List<int[]> arr = new ArrayList<>();
        for(int i = 0; i < nums.length; i++) {
            arr.add(new int[] {nums[i], i});
        }

        Collections.sort(arr, (a, b) -> {
            return a[0] - b[0];
        });

        int min = arr.get(0)[1];
        int res = 0;

        for(int[] num : arr) {
            min = Math.min(min, num[1]);
            res = Math.max(num[1] - min, res);
        }

        return res;
    }
}
```

---

## Time Complexity Analysis

- Sorting takes `O(N log N)`.
- One traversal takes `O(N)`.

Overall time complexity:

```
O(N log N)
```

---

# Method 2: Monotonic Stack (More Efficient)

## Idea

Use a **monotonic decreasing stack** to store candidate left indices.

### First Pass (Left to Right)

- If the current element is smaller than the element at the top of the stack, push its index.
- This ensures the stack maintains strictly decreasing values.
- If there are duplicate values, only the leftmost index is kept.

### Second Pass (Right to Left)

- If `nums[i] >= nums[stack.peek()]`:
    - A valid ramp is found.
    - Update the maximum width.
    - Pop the stack.
- Continue until the stack is empty or the condition no longer holds.

The stack effectively behaves like a reversed sorted “set” of candidate left endpoints, ensuring we always try the leftmost valid index to maximize width.

---

```java
class Solution {
    public int maxWidthRamp(int[] nums) {
        Deque<Integer> stack = new LinkedList<>();

        for(int i = 0; i < nums.length; i++) {
            if(stack.isEmpty() || nums[stack.peekLast()] > nums[i]) {
                stack.offerLast(i);
            }
        }

        int res = 0;

        for(int i = nums.length - 1; i >= 0; i--) {
            while(!stack.isEmpty() && nums[stack.peekLast()] <= nums[i]) {
                res = Math.max(res, i - stack.pollLast());
            }
        }

        return res;
    }
}
```

---

## Time Complexity Analysis

- Building the monotonic stack: `O(N)`
- Reverse traversal: `O(N)`
- Each index is pushed and popped at most once.

Overall time complexity:

```
O(N)
```




# [3152. Special Array II](https://leetcode.com/problems/special-array-ii/)

An array is considered **special** if every pair of adjacent elements contains two numbers with different parity.

You are given an integer array `nums` and a 2D integer matrix `queries`, where `queries[i] = [fromi, toi]`.  
Your task is to determine whether the subarray `nums[fromi..toi]` is **special**.

Return a boolean array `answer` such that `answer[i]` is `true` if `nums[fromi..toi]` is special.

---

## Example 1

**Input:**  
nums = [3,4,1,2,6], queries = [[0,4]]

**Output:**  
[false]

**Explanation:**  
The subarray is `[3,4,1,2,6]`.  
2 and 6 are both even, so it is not special.

---

## Example 2

**Input:**  
nums = [4,3,1,6], queries = [[0,2],[2,3]]

**Output:**  
[false,true]

**Explanation:**

1. The subarray `[4,3,1]`: 3 and 1 are both odd → not special.
2. The subarray `[1,6]`: the pair (1,6) has different parity → special.

---

## Constraints

- `1 <= nums.length <= 10^5`
- `1 <= nums[i] <= 10^5`
- `1 <= queries.length <= 10^5`
- `queries[i].length == 2`
- `0 <= queries[i][0] <= queries[i][1] <= nums.length - 1`

---

# Method 1: Binary Search on Invalid Adjacent Pairs

## Idea

1. Traverse the array once.
2. If two adjacent elements have the **same parity**, store the smaller index `i` in a list.
3. For each query `[left, right]`:
   - We need to check whether there exists an index `i` in the range `[left, right - 1]`
   - Use binary search to check if such an index exists in the list.
   - If found → the subarray is NOT special.
   - Otherwise → it is special.

---

```java
class Solution {
    public boolean[] isArraySpecial(int[] nums, int[][] queries) {
        List<Integer> list = new ArrayList<>();
        
        for(int i = 0; i < nums.length - 1; i++) {
            if(((nums[i] & 1) ^ (nums[i + 1] & 1)) == 0) {
                list.add(i);
            }
        }

        boolean[] res = new boolean[queries.length];

        for(int i = 0; i < queries.length; i++) {
            int left = queries[i][0];
            int right = queries[i][1] - 1;

            int l = 0, r = list.size() - 1;
            boolean has = false;

            while(l <= r) {
                int mid = l + (r - l) / 2;
                int idx = list.get(mid);

                if(left <= idx && idx <= right) {
                    has = true;
                    break;
                } else if(idx > right) {
                    r = mid - 1;
                } else {
                    l = mid + 1;
                }
            }

            res[i] = !has;
        }

        return res;
    }
}
```

---

## Time Complexity

- Traverse array: `O(N)`
- Binary search for each query: `O(Q log N)`

Overall:

```
O(N + Q log N)
```

## Space Complexity

- Store invalid adjacent indices: `O(N)`

---

# Method 2: Prefix Sum (More Efficient)

## Idea

Instead of binary search, we can preprocess prefix sums.

Define:

- `prefixSum[i]` = number of adjacent same-parity pairs in range `[0, i]`

If two adjacent elements have the same parity:

```
prefixSum[i] = prefixSum[i - 1] + 1
```

Otherwise:

```
prefixSum[i] = prefixSum[i - 1]
```

For a query `[from, to]`:

- If `prefixSum[to] - prefixSum[from] == 0`
  → no same-parity adjacent pair exists
  → subarray is special.

---

```java
class Solution {
    public boolean[] isArraySpecial(int[] nums, int[][] queries) {
        boolean[] ans = new boolean[queries.length];
        int[] prefixSum = new int[nums.length];

        for(int i = 1; i < nums.length; i++) {
            prefixSum[i] = prefixSum[i - 1];
            if(nums[i] % 2 == nums[i - 1] % 2) {
                prefixSum[i] += 1;
            }
        }

        int count = 0;

        for(int[] q : queries) {
            int from = q[0];
            int to = q[1];
            ans[count] = (prefixSum[to] - prefixSum[from] == 0);
            count++;
        }

        return ans;
    }
}
```

---

## Time Complexity

- Build prefix sum: `O(N)`
- Process queries: `O(Q)`

Overall:

```
O(N + Q)
```

## Space Complexity

```
O(N)
```

---

## Summary

| Method | Time Complexity | Space Complexity | Recommended |
|--------|------------------|------------------|-------------|
| Binary Search | O(N + Q log N) | O(N) | ⭐⭐ |
| Prefix Sum | O(N + Q) | O(N) | ⭐⭐⭐⭐ |

The prefix sum solution is cleaner and optimal for large inputs.




# [2779. Maximum Beauty of an Array After Applying Operation](https://leetcode.com/problems/maximum-beauty-of-an-array-after-applying-operation/)

You are given a **0-indexed** array `nums` and a **non-negative** integer `k`.

In one operation, you can:

- Choose an index `i` that **has not been chosen before**.
- Replace `nums[i]` with any integer in the range `[nums[i] - k, nums[i] + k]`.

The **beauty** of the array is defined as the length of the longest subsequence consisting of equal elements.

Return the ***maximum*** possible beauty of the array after applying the operation any number of times.

Each index can be chosen **at most once**.

A **subsequence** is formed by deleting some elements (possibly none) without changing the order of the remaining elements.

---

## Example 1

```plain
Input: nums = [4,6,1,2], k = 2
Output: 3
Explanation:
- Choose index 1, replace it with 4 → nums = [4,4,1,2]
- Choose index 3, replace it with 4 → nums = [4,4,1,4]
The longest equal subsequence has length 3.
```

## Example 2

```plain
Input: nums = [1,1,1,1], k = 10
Output: 4
Explanation:
No operation is needed. The whole array is already equal.
```

---

## Constraints

- `1 <= nums.length <= 10^5`
- `0 <= nums[i], k <= 10^5`

---

# Key Insight

Each number `nums[i]` can be transformed into any value in the interval:

```
[nums[i] - k, nums[i] + k]
```

So the problem becomes:

> Find the maximum number of intervals that overlap at some point.

This is essentially an **interval overlap** problem.

Although the problem mentions subsequences, the actual goal is to find the largest group of numbers whose intervals intersect at some common value.

---

# Approach: Sorting + Binary Search

## Idea

1. Sort the array.
2. Fix a left index `i`.
3. Use binary search to find the furthest index `mid` such that:

```
nums[mid] - k <= nums[i] + k
```

This ensures that the intervals overlap.

4. Update the maximum length.

---

```java
class Solution {
    public int maximumBeauty(int[] nums, int k) {
        int n = nums.length;
        Arrays.sort(nums);

        int res = 1;

        for(int i = 0; i < n; i++) {
            int left = i + 1;
            int right = n - 1;
            int farthest = i;

            while(left <= right) {
                int mid = left + (right - left) / 2;

                if(nums[mid] - k <= nums[i] + k) {
                    farthest = mid;
                    left = mid + 1;
                } else {
                    right = mid - 1;
                }
            }

            res = Math.max(res, farthest - i + 1);
        }

        return res;
    }
}
```

---

## Time Complexity

- Sorting: `O(N log N)`
- Outer loop: `O(N)`
- Binary search inside: `O(log N)`

Overall:

```
O(N log N)
```

---

## Space Complexity

```
O(1)
```

(ignoring sorting's internal stack space)

---

## Intuition Summary

Even though the problem mentions subsequences, the core idea is:

- Each element defines a transformable interval.
- We want the maximum number of intervals that overlap.
- Sorting enables efficient binary search to compute overlap size.

This is fundamentally an interval overlap maximization problem.



# [2981. Find Longest Special Substring That Occurs Thrice I](https://leetcode.com/problems/find-longest-special-substring-that-occurs-thrice-i/)

You are given a string `s` consisting of lowercase English letters.

A string is called **special** if it contains only a single repeated character.  
For example:

- `"abc"` is NOT special
- `"ddd"`, `"zz"`, and `"f"` are special

Return the length of the **longest special substring** that occurs **at least three times**, or `-1` if no such substring exists.

A **substring** is a contiguous non-empty sequence of characters.

---

## Example 1

```plain
Input: s = "aaaa"
Output: 2
Explanation:
The longest special substring occurring at least three times is "aa".
```

## Example 2

```plain
Input: s = "abcdef"
Output: -1
Explanation:
No special substring occurs at least three times.
```

## Example 3

```plain
Input: s = "abcaba"
Output: 1
Explanation:
The longest special substring occurring at least three times is "a".
```

---

## Constraints

- `3 <= s.length <= 50`
- `s` consists only of lowercase English letters.

---

# Key Idea

A brute-force approach would try all substrings of all possible lengths for each character — but that would be inefficient.

Instead, observe:

For each character, we only need to track:

- The **longest** consecutive segment
- The **second longest** consecutive segment
- Their counts

For a character:

Let:

- `longest`
- `countLongest`
- `secondLongest`
- `countSecondLongest`

Then:

1. If `countLongest >= 3`  
   → answer can be `longest`

2. If `countLongest == 2`  
   → answer can be `longest - 1`

3. If `countLongest == 1`:
    - If `secondLongest == longest - 1`  
      → answer can be `longest - 1`
    - Otherwise  
      → answer can be `longest - 2`

Additionally, if total occurrences across segments ≥ 3, length 1 is always possible.

---

# Implementation

```java
class Solution {
    public int maximumLength(String s) {
        int n = s.length();
        
        // [secondLongest, countSecondLongest, longest, countLongest]
        int[][] map = new int[26][4];
        
        int start = 0;
        
        for(int end = 0; end < n; end++) {
            if(s.charAt(start) == s.charAt(end)) {
                if(end != n - 1) continue;
                update(map, s, start, end);
            } else {
                update(map, s, start, end - 1);
                start = end;
            }
        }

        if(s.charAt(n - 1) != s.charAt(n - 2)) {
            update(map, s, start, start);
        }

        int res = 0;

        for(int[] arr : map) {

            // If total segments allow at least three single chars
            if(arr[0] + arr[2] >= 3) {
                res = Math.max(res, 1);
            }

            if(arr[3] >= 3) {
                res = Math.max(res, arr[2]);
            } else if(arr[3] == 2) {
                res = Math.max(res, arr[2] - 1);
            } else if(arr[3] == 1) {
                if(arr[0] == arr[2] - 1 && arr[1] >= 1) {
                    res = Math.max(res, arr[2] - 1);
                } else {
                    res = Math.max(res, arr[2] - 2);
                }
            }
        }

        return res == 0 ? -1 : res;
    }

    private void update(int[][] map, String s, int start, int end) {
        int cur = s.charAt(start) - 'a';
        int len = end - start + 1;

        int second = map[cur][0];
        int secondCount = map[cur][1];
        int longest = map[cur][2];
        int longestCount = map[cur][3];

        if(longest == 0 && second == 0) {
            map[cur][2] = len;
            map[cur][3] = 1;
        } 
        else if(len > longest) {
            map[cur][0] = longest;
            map[cur][1] = longestCount;
            map[cur][2] = len;
            map[cur][3] = 1;
        } 
        else if(len == longest) {
            map[cur][3]++;
        } 
        else if(len > second) {
            map[cur][0] = len;
            map[cur][1] = 1;
        } 
        else if(len == second) {
            map[cur][1]++;
        }
    }
}
```

---

# Time Complexity

- Traverse string: `O(N)`
- Update operation: `O(1)`
- Traverse 26 letters: `O(1)`

Overall:

```
O(N)
```

---

# Space Complexity

```
O(1)
```

(Only 26 × 4 fixed-size storage)

---

## Final Insight

The trick is recognizing that:

- Only the top two segment lengths per character matter.
- We don't need to examine all substrings.
- Careful case analysis reduces the problem to simple arithmetic comparisons.

This makes the solution linear and efficient.



# [3381. Maximum Subarray Sum With Length Divisible by K](https://leetcode.com/problems/maximum-subarray-sum-with-length-divisible-by-k/)

You are given an integer array `nums` and an integer `k`.

Return the **maximum** sum of a subarray whose length is **divisible by `k`**.

---

## Example 1

**Input:**  
nums = [1,2], k = 1

**Output:**  
3

**Explanation:**  
The subarray `[1, 2]` has length 2, which is divisible by 1.

---

## Example 2

**Input:**  
nums = [-1,-2,-3,-4,-5], k = 4

**Output:**  
-10

**Explanation:**  
The subarray `[-1, -2, -3, -4]` has length 4, which is divisible by 4.

---

## Example 3

**Input:**  
nums = [-5,1,2,-3,4], k = 2

**Output:**  
4

**Explanation:**  
The subarray `[1, 2, -3, 4]` has length 4, which is divisible by 2.

---

## Constraints

- `1 <= k <= nums.length <= 2 * 10^5`
- `-10^9 <= nums[i] <= 10^9`

---

# Key Idea

If `k = 1`, the problem reduces to the classic **maximum subarray sum**, which can be solved using **Kadane’s algorithm** in `O(N)` time.

If `k > 1`, observe:

A valid subarray must have length:

```
k, 2k, 3k, ...
```

For each possible starting remainder `i` in `[0, k - 1]`, we can:

- Group elements into blocks of size `k`:

  ```
  [i, i+k), [i+k, i+2k), ...
  ```

- Treat each block sum as a single element.
- Then apply Kadane’s algorithm on that transformed array.

This converts the problem into multiple standard maximum subarray problems.

---

# Implementation

```java
class Solution {
    public long maxSubarraySum(int[] nums, int k) {
        if (k == 1) {
            return kadane(nums);
        }

        long res = Long.MIN_VALUE;
        long[] prefix = new long[nums.length + 1];

        // Build prefix sum
        for (int i = 1; i <= nums.length; i++) {
            prefix[i] = prefix[i - 1] + nums[i - 1];
        }

        // Try each possible starting offset
        for (int i = 0; i < k; i++) {
            int len = (nums.length - i) / k;
            long[] tmp = new long[len];

            for (int j = 0; j < len; j++) {
                int start = i + j * k;
                int end = start + k;
                tmp[j] = prefix[end] - prefix[start];
            }

            res = Math.max(res, kadane(tmp));
        }

        return res;
    }

    static long kadane(long[] nums) {
        long res = Long.MIN_VALUE;
        long curr = 0;

        for (int i = 0; i < nums.length; i++) {
            if (curr + nums[i] > nums[i]) {
                curr += nums[i];
            } else {
                curr = nums[i];
            }
            res = Math.max(res, curr);
        }

        return res;
    }
}
```

---

# Time Complexity

- Prefix sum computation: `O(N)`
- Outer loop: `O(K)`
- Inner grouping: `O(N / K)`
- Kadane per group: `O(N / K)`

Total:

```
O(K × (N/K + N/K)) = O(N)
```

---

# Space Complexity

- Prefix array: `O(N)`
- Temporary block array: up to `O(N/K)`

Overall:

```
O(N)
```

---

## Final Insight

This problem cleverly reduces a “length divisible by k” constraint into grouping elements into blocks of size `k`, allowing reuse of Kadane’s algorithm.

The key realization is:

> Any valid subarray can be decomposed into consecutive full blocks of size `k`.

Once that’s recognized, the problem becomes linear-time solvable.



# [3376. Minimum Time to Break Locks I](https://leetcode.com/problems/evaluate-division/)

Bob needs to break `n` locks, each requiring a certain amount of energy given by the array `strength`, where `strength[i]` represents the energy needed to break the `i`-th lock.

The rules are:

- The sword's initial energy is `0`.
- The initial growth factor `X = 1`.
- Every minute, the sword's energy increases by the current factor `X`.
- To break the `i`-th lock, the energy must be **at least** `strength[i]`.
- After breaking a lock:
    - The energy resets to `0`.
    - The factor increases by `K` (i.e., `X += K`).

Your task is to compute the **minimum time (in minutes)** required to break all locks.

---

## Approach

Since `n ≤ 8`, we can use **backtracking (DFS)** to try all possible orders of breaking the locks.

At each step:

- Choose an unvisited lock.
- Compute the time needed to accumulate enough energy:

  ```
  time = ceil(strength[i] / X)
       = (strength[i] + X - 1) / X
  ```

- Recurse with:
    - Updated factor `X + K`
    - Increased total time
    - Marked lock as visited

Keep track of the minimum total time across all permutations.

---

## Code

```java
class Solution {
    int res = Integer.MAX_VALUE;

    public int findMinimumTime(List<Integer> strength, int K) {
        dfs(strength, 1, K, 0, new boolean[strength.size()], 0);
        return res;
    }

    private void dfs(List<Integer> list, int x, int k, int finished,
                     boolean[] visited, int sum) {

        if (finished == list.size()) {
            res = Math.min(res, sum);
            return;
        }

        for (int i = 0; i < list.size(); i++) {
            if (!visited[i]) {
                visited[i] = true;

                int time = (list.get(i) + x - 1) / x;

                dfs(list,
                    x + k,
                    k,
                    finished + 1,
                    visited,
                    sum + time);

                visited[i] = false;
            }
        }
    }
}
```

---

## Time Complexity

Since we try all possible orders of breaking the locks:

```
O(N!)
```

Given that `N ≤ 8`, this is acceptable.

---

## Space Complexity

- Recursion stack depth: `O(N)`
- Visited array: `O(N)`

Overall:

```
O(N)
```



# [2762. Continuous Subarrays](https://leetcode.com/problems/evaluate-division/)

You are given a **0-indexed** integer array `nums`. A subarray of `nums` is called **continuous** if:

For indices `i` to `j` in the subarray, for every pair `i ≤ i1, i2 ≤ j`:

```
0 ≤ |nums[i1] - nums[i2]| ≤ 2
```

Return the total number of **continuous subarrays**.

A subarray is a contiguous non-empty sequence of elements within an array.

---

## Key Insight

For a subarray to be valid:

```
max(subarray) - min(subarray) ≤ 2
```

So this becomes a classic **sliding window** problem where we must maintain:

- The **maximum** value in the window
- The **minimum** value in the window

Whenever:

```
max - min > 2
```

we shrink the window from the left.

To efficiently maintain max and min, we can use:

- Two heaps (min heap + max heap) → `O(N log N)`
- Two monotonic deques → `O(N)` (optimal)

---

# Heap Version

```java
class Solution {
    public long continuousSubarrays(int[] nums) {
        long res = 0;

        Queue<int[]> min = new PriorityQueue<>((a, b) -> a[0] - b[0]);
        Queue<int[]> max = new PriorityQueue<>((a, b) -> b[0] - a[0]);

        int start = 0;

        for (int end = 0; end < nums.length; end++) {

            while (!min.isEmpty() && min.peek()[0] + 2 < nums[end]) {
                int[] cur = min.poll();
                if (start <= cur[1]) {
                    start = cur[1] + 1;
                }
            }
            min.offer(new int[]{nums[end], end});

            while (!max.isEmpty() && max.peek()[0] > nums[end] + 2) {
                int[] cur = max.poll();
                if (start <= cur[1]) {
                    start = cur[1] + 1;
                }
            }
            max.offer(new int[]{nums[end], end});

            res += end - start + 1;
        }

        return res;
    }
}
```

### Time Complexity

- Each element enters and leaves the heap at most once
- Heap operations cost `O(log N)`

```
O(N log N)
```

### Space Complexity

```
O(N)
```

---

# Monotonic Deque Version (Optimal)

```java
class Solution {

    public long continuousSubarrays(int[] nums) {

        Deque<Integer> maxQ = new ArrayDeque<>();
        Deque<Integer> minQ = new ArrayDeque<>();

        int left = 0;
        long count = 0;

        for (int right = 0; right < nums.length; right++) {

            // Maintain decreasing deque for max
            while (!maxQ.isEmpty() && nums[maxQ.peekLast()] < nums[right]) {
                maxQ.pollLast();
            }
            maxQ.offerLast(right);

            // Maintain increasing deque for min
            while (!minQ.isEmpty() && nums[minQ.peekLast()] > nums[right]) {
                minQ.pollLast();
            }
            minQ.offerLast(right);

            // Shrink window if invalid
            while (!maxQ.isEmpty() &&
                   !minQ.isEmpty() &&
                   nums[maxQ.peekFirst()] - nums[minQ.peekFirst()] > 2) {

                if (maxQ.peekFirst() < minQ.peekFirst()) {
                    left = maxQ.peekFirst() + 1;
                    maxQ.pollFirst();
                } else {
                    left = minQ.peekFirst() + 1;
                    minQ.pollFirst();
                }
            }

            count += right - left + 1;
        }

        return count;
    }
}
```

### Time Complexity

Each element:

- Enters deque once
- Leaves deque once

```
O(N)
```

### Space Complexity

```
O(N)
```

---

## Final Takeaway

This is a standard sliding window problem with a bounded range constraint:

```
max - min ≤ 2
```

Maintaining both the maximum and minimum dynamically is the key.

Using two monotonic deques reduces the complexity from `O(N log N)` to `O(N)`.

# 1734. [Decode XORed Permutation](https://leetcode.com/problems/evaluate-division/description/)

There is an integer array `perm` that is a permutation of the first `n` positive integers, where `n` is always **odd**.

It was encoded into another array `encoded` of length `n - 1`, such that:

```
encoded[i] = perm[i] XOR perm[i + 1]
```

Given `encoded`, return the original array `perm`.

It is guaranteed that the answer exists and is unique.

---

## Key Insight

Since:

```
encoded[i] = perm[i] XOR perm[i+1]
```

If we know **any one value** in `perm`, we can recover the entire permutation using XOR.

The idea is to compute `perm[0]`.

---

## Derivation

Because `perm` is a permutation of `[1, 2, ..., n]`:

```
perm[0] ^ perm[1] ^ ... ^ perm[n-1] = 1 ^ 2 ^ ... ^ n
```

Let:

```
total = 1 ^ 2 ^ ... ^ n
```

Now observe:

```
encoded[1] = perm[1] ^ perm[2]
encoded[3] = perm[3] ^ perm[4]
...
```

If we XOR all encoded elements at **odd indices**:

```
encoded[1] ^ encoded[3] ^ ... ^ encoded[n-2]
```

We get:

```
perm[1] ^ perm[2] ^
perm[3] ^ perm[4] ^
...
perm[n-2] ^ perm[n-1]
```

Since `n` is odd, this covers:

```
perm[1] ^ perm[2] ^ ... ^ perm[n-1]
```

Call this value `partial`.

Then:

```
perm[0] = total ^ partial
```

Once `perm[0]` is known, the rest can be reconstructed using:

```
perm[i] = perm[i-1] ^ encoded[i-1]
```

---

## Code

```java
class Solution {
    public int[] decode(int[] encoded) {
        int n = encoded.length;
        
        int a0 = 0;

        // XOR of 1 to n+1
        for (int i = 1; i <= n + 1; i++) {
            a0 ^= i;
        }

        // XOR encoded elements at odd indices
        for (int i = 1; i < n; i += 2) {
            a0 ^= encoded[i];
        }

        int[] res = new int[n + 1];
        res[0] = a0;

        for (int i = 1; i < res.length; i++) {
            res[i] = res[i - 1] ^ encoded[i - 1];
        }

        return res;
    }
}
```

---

## Time Complexity

```
O(n)
```

- One pass to compute total XOR
- One pass over half of `encoded`
- One pass to reconstruct the permutation

---

## Space Complexity

```
O(1)
```

Only constant extra variables are used (excluding output array).

---

## Final Insight

The key trick is using the property that:

```
a ^ a = 0
a ^ 0 = a
```

and leveraging the fact that `n` is **odd**, which guarantees we can isolate `perm[0]`.

Once the first element is determined, the entire permutation follows naturally.



# 1930. [Unique Length-3 Palindromic Subsequences](https://leetcode.com/problems/unique-length-3-palindromic-subsequences/description/)

Given a string `s`, return the number of **unique palindromes of length 3** that are a **subsequence** of `s`.

A palindrome reads the same forward and backward.  
A subsequence is formed by deleting some (possibly zero) characters without changing the relative order.

---

## Key Idea

Since the required palindrome length is **3**, its structure must be:

```
a b a
```

So for every character `a`, we:

1. Find its **first occurrence**
2. Find its **last occurrence**
3. Count how many **distinct characters** appear between them

Each distinct middle character `b` forms a unique palindrome:

```
a b a
```

The main challenge is avoiding duplicates.

---

## Approach

We use:

- `first[26]` to record the first occurrence of each character
- A prefix frequency array `map[i][26]`:
  - `map[i][c]` stores how many times character `c` appears in `s[0..i]`
- A boolean matrix `exists[26][26]` to prevent duplicate counting

When processing index `i` as a potential **right boundary**:

- Let `cur` be the character
- Check if the distance from its first occurrence is at least 2
- For every character `k`:
  - If it appears between `first[cur]` and `i`
  - And hasn't been counted before
  - Then increment result

---

## Code

```java
class Solution {
    public int countPalindromicSubsequence(String s) {
        int n = s.length();
        int[][] map = new int[n][26];
        int[] first = new int[26]; 
        Arrays.fill(first, -1);

        for(int i = 0; i < n; i++) {
            int cur = s.charAt(i) - 'a';

            if(first[cur] == -1) {
                first[cur] = i;
            }

            if(i != 0) {
                for(int j = 0; j < 26; j++) {
                    map[i][j] = map[i - 1][j];
                }
            }

            map[i][cur]++;
        }

        boolean[][] exists = new boolean[26][26];
        int res = 0;

        for(int i = 2; i < n; i++) {
            int cur = s.charAt(i) - 'a';

            if(first[cur] == -1 || i - first[cur] < 2) {
                continue;
            }

            for(int k = 0; k < 26; k++) {
                if(map[i - 1][k] - map[first[cur]][k] > 0 
                   && !exists[cur][k]) {
                    res++;
                    exists[cur][k] = true;
                }
            }
        }

        return res;
    }
}
```

---

## Time Complexity

```
O(n)
```

- One pass to build prefix frequency
- One pass to count valid palindromes
- Inner loop runs over 26 characters (constant)

---

## Space Complexity

```
O(n)
```

- Prefix frequency array requires `O(n * 26)`
- Auxiliary arrays are constant size

---

## Core Insight

Because the palindrome length is fixed at 3, we reduce the problem to:

> For each character `a`, count distinct characters between its first and last occurrence.

The constraint of lowercase English letters (26 characters) makes this efficient and manageable.






# 2381. [Shifting Letters II](https://leetcode.com/problems/shifting-letters-ii/description/)

You are given a string `s` and a list of shift operations `shifts`, where:

```
shifts[i] = [start, end, direction]
```

- If `direction == 1`, shift characters in `[start, end]` **forward**
- If `direction == 0`, shift characters in `[start, end]` **backward**

Shifting forward:
```
'z' → 'a'
```

Shifting backward:
```
'a' → 'z'
```

Return the final string after applying all shifts.

---

## Key Idea

If we simulate each shift directly, the worst-case time complexity becomes:

```
O(n * m)
```

where `n` is the string length and `m` is the number of shifts — this will TLE.

Instead, we use a **difference array (prefix sum technique)**.

### Why it works

For each shift operation:

- If forward (`direction == 1`):
  ```
  diff[start] += 1
  diff[end + 1] -= 1
  ```
- If backward (`direction == 0`):
  ```
  diff[start] -= 1
  diff[end + 1] += 1
  ```

After processing all operations, we compute the prefix sum of `diff`.

Then `diff[i]` tells us how many times index `i` should be shifted.

Finally, apply the shift with modulo 26.

---

## Code

```java
class Solution {
    public String shiftingLetters(String s, int[][] shifts) {
        int n = s.length();
        int[] diff = new int[n + 1];

        for (int[] arr : shifts) {
            if (arr[2] == 1) {
                diff[arr[0]]++;
                diff[arr[1] + 1]--;
            } else {
                diff[arr[0]]--;
                diff[arr[1] + 1]++;
            }
        }

        for (int i = 1; i < n; i++) {
            diff[i] += diff[i - 1];
        }

        char[] str = s.toCharArray();

        for (int i = 0; i < n; i++) {
            str[i] = (char) (
                ((str[i] - 'a' + diff[i]) % 26 + 26) % 26 + 'a'
            );
        }

        return new String(str);
    }
}
```

---

## Time Complexity

```
O(n + m)
```

- Process all shift operations
- One prefix sum pass
- One pass to build result

---

## Space Complexity

```
O(n)
```

- Difference array of size `n + 1`

---

## Core Insight

This is a classic **range update problem**.

Instead of applying each shift directly, we:

1. Convert range updates into point updates using a difference array
2. Use prefix sum to recover final shifts
3. Apply modulo arithmetic for circular alphabet shifts

This reduces the problem to linear time.





# 983. [Minimum Cost For Tickets](https://leetcode.com/problems/minimum-cost-for-tickets/description/)

You are given a list of travel days `days` and ticket costs:

- 1-day pass → `costs[0]`
- 7-day pass → `costs[1]`
- 30-day pass → `costs[2]`

Each pass covers consecutive days starting from the purchase day.

Return the minimum total cost to cover all travel days.

---

## Key Idea — Dynamic Programming

This is a classic DP problem.

Let:

```
dp[d] = minimum cost to cover travel starting from day d
```

We compute it **backwards** from the last travel day.

For a travel day `d`, we have three choices:

```
dp[d] = min(
    cost[0] + dp[d + 1],
    cost[1] + dp[d + 7],
    cost[2] + dp[d + 30]
)
```

If a day is **not** a travel day, its cost equals the next day’s cost.

To simplify indexing beyond the last day, we extend the DP array by 30 extra days.

---

## Code

```java
class Solution {
    public int mincostTickets(int[] days, int[] costs) {
        int n = days.length;
        int[] dp = new int[days[n - 1] + 1 + 30];
        Arrays.fill(dp, Integer.MAX_VALUE);

        int[] len = new int[]{1, 7, 30};

        // After last travel day, cost is 0
        for (int i = days[n - 1] + 1; i < dp.length; i++) {
            dp[i] = 0;
        }

        int last = -1;

        for (int i = n - 1; i >= 0; i--) {

            // Fill non-travel days between two travel days
            while (last != -1 && last > days[i]) {
                dp[last--] = dp[days[i + 1]];
            }

            last = days[i];

            for (int j = 0; j < 3; j++) {
                dp[days[i]] = Math.min(
                    dp[days[i]],
                    costs[j] + dp[days[i] + len[j]]
                );
            }
        }

        return dp[days[0]];
    }
}
```

---

## Time Complexity

```
O(n)
```

Each travel day is processed once, and non-travel days are filled in once.

---

## Space Complexity

```
O(365)
```

The DP array size is bounded by the maximum possible day (≤ 365 + 30).

---

## Core Insight

At every travel day, we decide:

> Should we buy a 1-day, 7-day, or 30-day pass?

Since each decision only depends on future days, backward DP works naturally.

This is essentially a **minimum cost coverage problem with fixed interval options**.








