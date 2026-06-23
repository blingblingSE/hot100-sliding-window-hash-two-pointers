# Hot100 面试复盘：哈希表、滑动窗口、双指针 12 题

这份笔记用于自己刷题复盘。重点不是死背代码，而是看到题目时能快速判断：

```text
这题在维护什么状态？
状态怎么更新？
什么时候收缩、查找、去重？
```

## 总览表

| 模型 | 题目 | 识别信号 | 核心动作 |
|---|---|---|---|
| 哈希表 | 两数之和 | 找另一个数、查是否出现过 | `target - num` |
| 哈希表 | 字母异位词分组 | 同一类字符串归组 | 排序后作为 key |
| 哈希集合 | 最长连续序列 | 连续数字、要求 O(n) | 只从起点扩展 |
| 前缀和 + 哈希 | 和为 K 的子数组 | 连续子数组和、可能有负数 | 查 `prefix - k` |
| 滑动窗口 | 无重复字符的最长子串 | 最长、连续、不重复 | 右扩左缩 |
| 滑动窗口 | 找到所有字母异位词 | 固定长度窗口、频率一致 | 比较计数器 |
| 滑动窗口 + 单调队列 | 滑动窗口最大值 | 固定窗口、每个窗口最大值 | 队首保持最大值下标 |
| 滑动窗口 | 最小覆盖子串 | 最短子串、覆盖目标字符 | 满足后尽量收缩 |
| 双指针 | 盛最多水的容器 | 两端选边界、最大面积 | 移动短板 |
| 排序 + 双指针 | 三数之和 | 三元组、去重、和为 0 | 固定一个数转 2Sum |
| 快慢指针 | 移动零 | 原地调整数组 | 非零前移 |
| 双指针 | 接雨水 | 左右边界、积水量 | 维护左右最大值 |

## 一、哈希表模型：查找、计数、映射

### 1. 两数之和

题目核心：遍历当前数 `num` 时，之前是否出现过 `target - num`。

面试口述：

```text
我用哈希表记录已经遍历过的数字及其下标。
对每个 num，先查 target - num 是否在表里。
如果在，说明找到答案；如果不在，再把当前数加入表。
```

```python
class Solution:
    def twoSum(self, nums: list[int], target: int) -> list[int]:
        seen = {}

        for i, num in enumerate(nums):
            need = target - num
            if need in seen:
                return [seen[need], i]
            seen[num] = i

        return []
```

Python 知识点：

`enumerate(nums)` 是 Python 的内置函数。遍历 `nums` 时，它可以同时拿到下标 `i` 和对应的值 `num`。所以 `for i, num in enumerate(nums)` 的意思是：每次循环同时取出当前位置和当前位置的数字。

易错点：

- 先查再存，避免同一个元素被用两次。
- 哈希表存下标，不只是存数字。

### 2. 字母异位词分组

题目核心：异位词排序后完全相同。

面试口述：

```text
我把每个字符串排序，排序结果作为哈希表 key。
key 相同的字符串一定属于同一组。
最后返回哈希表中的所有 value。
```

```python
class Solution:
    def groupAnagrams(self, strs: list[str]) -> list[list[str]]:
        hashmap = {}

        for s in strs:
            key = ''.join(sorted(s))

            if key not in hashmap:
                hashmap[key] = []

            hashmap[key].append(s)

        return list(hashmap.values())
```

易错点：

- `sorted(s)` 返回列表，需要 `''.join(...)` 转成字符串 key。
- 如果字符集固定为小写字母，也可以用 26 位计数元组作为 key。

### 3. 最长连续序列

题目核心：只从连续序列的起点开始数。

面试口述：

```text
我先把所有数字放进 set。
只有当 num - 1 不存在时，num 才是一个连续序列的起点。
然后从这个起点不断向后扩展，统计长度。
```

```python
class Solution:
    def longestConsecutive(self, nums: list[int]) -> int:
        num_set = set(nums)
        ans = 0

        for num in num_set:
            if num - 1 in num_set:
                continue

            cur = num
            length = 1

            while cur + 1 in num_set:
                cur += 1
                length += 1

            ans = max(ans, length)

        return ans
```

易错点：

- 要遍历 `num_set`，避免重复数字影响效率。
- 不要从每个数字都向后扩展，否则最坏会退化。

### 4. 和为 K 的子数组

题目核心：子数组和可以由两个前缀和相减得到。

```text
sum(i..j) = prefix[j] - prefix[i - 1]
如果 sum(i..j) = k
那么 prefix[i - 1] = prefix[j] - k
```

面试口述：

```text
我一边遍历一边维护当前前缀和 prefix。
如果之前出现过 prefix - k，说明存在若干个以当前位置结尾、和为 k 的子数组。
哈希表记录每个前缀和出现次数。
```

```python
from collections import defaultdict


class Solution:
    def subarraySum(self, nums: list[int], k: int) -> int:
        count = defaultdict(int)
        count[0] = 1

        prefix = 0
        ans = 0

        for num in nums:
            prefix += num
            ans += count[prefix - k]
            count[prefix] += 1

        return ans
```

易错点：

- `count[0] = 1` 表示空前缀，用来处理从下标 0 开始的子数组。
- 数组有负数时不能用普通滑动窗口，因为窗口和不具备单调性。

## 二、滑动窗口模型：连续区间维护

滑动窗口适合这类信号：

```text
连续子串 / 连续子数组
最长 / 最短 / 所有满足条件的窗口
窗口右边扩张，条件不满足时左边收缩
```

### 5. 无重复字符的最长子串

题目核心：窗口中不能出现重复字符。

面试口述：

```text
我用 set 维护当前窗口里的字符。
right 每次向右扩张；如果 s[right] 已经在窗口中，就不断移动 left 并删除字符，直到没有重复。
每次窗口合法时更新最大长度。
```

```python
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        window = set()
        left = 0
        ans = 0

        for right, ch in enumerate(s):
            while ch in window:
                window.remove(s[left])
                left += 1

            window.add(ch)
            ans = max(ans, right - left + 1)

        return ans
```

易错点：

- 这里是 `while`，不是 `if`，因为可能要连续移出多个字符。
- 更新答案要在窗口重新合法之后。

### 6. 找到字符串中所有字母异位词

题目核心：窗口长度固定为 `len(p)`，窗口频率等于目标频率时记录左端点。

面试口述：

```text
我维护一个长度不超过 len(p) 的滑动窗口。
每次加入右端字符，如果窗口过长就移出左端字符。
当窗口计数和 p 的计数完全相等时，当前 left 就是答案。
```

```python
from collections import Counter


class Solution:
    def findAnagrams(self, s: str, p: str) -> list[int]:
        target = Counter(p)
        window = Counter()
        left = 0
        ans = []

        for right, ch in enumerate(s):
            window[ch] += 1

            if right - left + 1 > len(p):
                left_ch = s[left]
                window[left_ch] -= 1
                if window[left_ch] == 0:
                    del window[left_ch]
                left += 1

            if window == target:
                ans.append(left)

        return ans
```

易错点：

- 频率变成 0 时要删除 key，否则 `Counter` 比较可能受影响。
- 这题窗口长度固定，和“最长子串”那种可变窗口不同。

### 7. 滑动窗口最大值

题目核心：每个长度为 `k` 的窗口都要快速拿到最大值。

面试口述：

```text
我用双端队列保存下标，并让队列对应的值保持单调递减。
每次加入新元素时，把队尾比它小的元素弹出，因为这些元素之后不可能成为最大值。
如果队首下标已经滑出窗口，就从队首弹出。
窗口形成后，队首对应的值就是当前窗口最大值。
```

```python
from collections import deque


class Solution:
    def maxSlidingWindow(self, nums: list[int], k: int) -> list[int]:
        queue = deque()
        ans = []

        for right, num in enumerate(nums):
            while queue and nums[queue[-1]] <= num:
                queue.pop()

            queue.append(right)

            if queue[0] <= right - k:
                queue.popleft()

            if right >= k - 1:
                ans.append(nums[queue[0]])

        return ans
```

易错点：

- 队列里存下标，不直接存值，这样才能判断元素是否滑出窗口。
- 队列单调递减，队首永远是当前窗口最大值。
- `right >= k - 1` 时窗口才真正形成。

### 8. 最小覆盖子串

题目核心：找到 `s` 中最短的子串，使它覆盖 `t` 中所有字符及出现次数。

面试口述：

```text
我用 need 统计 t 需要的字符频率，用 window 统计当前窗口频率。
right 负责扩张窗口，当某个字符数量刚好满足 need 时，valid 加 1。
当 valid 等于 need 的字符种类数，说明当前窗口已经覆盖 t，此时不断移动 left 缩小窗口并更新最短答案。
```

```python
from collections import Counter, defaultdict


class Solution:
    def minWindow(self, s: str, t: str) -> str:
        need = Counter(t)
        window = defaultdict(int)
        left = 0
        valid = 0
        start = 0
        min_len = float('inf')

        for right, ch in enumerate(s):
            if ch in need:
                window[ch] += 1
                if window[ch] == need[ch]:
                    valid += 1

            while valid == len(need):
                if right - left + 1 < min_len:
                    start = left
                    min_len = right - left + 1

                left_ch = s[left]
                if left_ch in need:
                    if window[left_ch] == need[left_ch]:
                        valid -= 1
                    window[left_ch] -= 1

                left += 1

        if min_len == float('inf'):
            return ''

        return s[start:start + min_len]
```

易错点：

- `valid` 记录的是“满足要求的字符种类数”，不是窗口长度。
- 收缩窗口前先更新答案，因为当前窗口已经合法。
- `t` 里可能有重复字符，所以必须比较频率，不能只用 set。

## 三、双指针模型：左右逼近、原地调整

双指针适合这类信号：

```text
数组有序或可以排序
两端向中间逼近
原地移动元素
根据左右状态决定移动哪一边
```

### 9. 盛最多水的容器

题目核心：面积由短板决定。

面试口述：

```text
我用 left 和 right 指向两端。
当前面积由较短的一边决定。
为了可能得到更大面积，只移动短板，因为移动长板不会提高当前高度上限。
```

```python
class Solution:
    def maxArea(self, height: list[int]) -> int:
        left, right = 0, len(height) - 1
        ans = 0

        while left < right:
            width = right - left
            area = min(height[left], height[right]) * width
            ans = max(ans, area)

            if height[left] < height[right]:
                left += 1
            else:
                right -= 1

        return ans
```

易错点：

- 移动短板，不是移动长板。
- 宽度每次都会变小，所以只能寄希望于高度变大。

### 10. 三数之和

题目核心：排序后固定一个数，剩下部分用双指针找两数之和。

面试口述：

```text
我先排序，枚举第一个数 nums[i]。
对 i 之后的区间使用 left 和 right 找两数之和为 -nums[i]。
为了避免重复，i 要去重，找到答案后 left 和 right 也要跳过重复值。
```

```python
class Solution:
    def threeSum(self, nums: list[int]) -> list[list[int]]:
        nums.sort()
        ans = []

        for i in range(len(nums)):
            if i > 0 and nums[i] == nums[i - 1]:
                continue

            left, right = i + 1, len(nums) - 1

            while left < right:
                total = nums[i] + nums[left] + nums[right]

                if total == 0:
                    ans.append([nums[i], nums[left], nums[right]])
                    left += 1
                    right -= 1

                    while left < right and nums[left] == nums[left - 1]:
                        left += 1
                    while left < right and nums[right] == nums[right + 1]:
                        right -= 1
                elif total < 0:
                    left += 1
                else:
                    right -= 1

        return ans
```

易错点：

- 只写固定 `i` 的去重还不够，找到三元组后左右指针也要去重。
- 排序是这题能用双指针的前提。

### 11. 移动零

题目核心：把非零元素按原顺序放到前面，剩余位置补 0。

面试口述：

```text
我用 slow 表示下一个非零元素应该放的位置。
fast 遍历数组，遇到非零就放到 slow，并让 slow 前进。
最后 slow 后面的所有位置补 0。
```

```python
class Solution:
    def moveZeroes(self, nums: list[int]) -> None:
        slow = 0

        for fast in range(len(nums)):
            if nums[fast] != 0:
                nums[slow] = nums[fast]
                slow += 1

        for i in range(slow, len(nums)):
            nums[i] = 0
```

易错点：

- 题目要求原地修改，不返回新数组。
- 非零元素的相对顺序不能变。

### 12. 接雨水

题目核心：当前位置能接多少水，由左右两侧最大高度的较小值决定。

面试口述：

```text
我用 left 和 right 从两端向中间走，并维护 left_max 和 right_max。
如果 height[left] < height[right]，左侧能接的水只取决于 left_max，因为右边一定有更高边界。
反之处理右侧。
```

```python
class Solution:
    def trap(self, height: list[int]) -> int:
        left, right = 0, len(height) - 1
        left_max = 0
        right_max = 0
        ans = 0

        while left < right:
            if height[left] < height[right]:
                left_max = max(left_max, height[left])
                ans += left_max - height[left]
                left += 1
            else:
                right_max = max(right_max, height[right])
                ans += right_max - height[right]
                right -= 1

        return ans
```

易错点：

- 先更新 `left_max/right_max`，再累加水量。
- 双指针写法本质是在选择“较低一侧”结算。

## 刷题决策树

看到题目后，可以按这个顺序判断：

```text
1. 要找某个数、某个状态是否出现过？
   -> 哈希表

2. 问连续子数组 / 连续子串，并且窗口状态可以维护？
   -> 滑动窗口

3. 问连续子数组和，并且数组可能有负数？
   -> 前缀和 + 哈希表

4. 数组可以排序，问两数 / 三数 / 多数之和？
   -> 排序 + 双指针

5. 问左右边界、面积、积水？
   -> 双指针，想清楚移动哪一边

6. 问连续数字序列，并要求 O(n)？
   -> set，找起点
```

## 高频模板

### 哈希查找模板

```python
seen = {}

for i, x in enumerate(nums):
    need = target - x
    if need in seen:
        return [seen[need], i]
    seen[x] = i
```

### 前缀和计数模板

```python
count = defaultdict(int)
count[0] = 1
prefix = 0
ans = 0

for x in nums:
    prefix += x
    ans += count[prefix - k]
    count[prefix] += 1
```

### 可变滑动窗口模板

```python
left = 0

for right in range(len(nums)):
    add(nums[right])

    while window_invalid():
        remove(nums[left])
        left += 1

    update_answer()
```

### 固定滑动窗口模板

```python
left = 0

for right in range(len(nums)):
    add(nums[right])

    if right - left + 1 > window_size:
        remove(nums[left])
        left += 1

    if window_valid():
        update_answer()
```

### 单调队列模板

```python
queue = deque()

for right, x in enumerate(nums):
    while queue and nums[queue[-1]] <= x:
        queue.pop()

    queue.append(right)

    if queue[0] <= right - k:
        queue.popleft()

    if right >= k - 1:
        ans.append(nums[queue[0]])
```

### 左右双指针模板

```python
left, right = 0, len(nums) - 1

while left < right:
    update_answer()

    if should_move_left():
        left += 1
    else:
        right -= 1
```

## 一句话总结

```text
Hot100 这组题的底层能力：
哈希表解决“查找和计数”，滑动窗口解决“连续区间状态”，双指针解决“左右逼近”，前缀和解决“子数组和”。
```
