# Hot100 面试复盘：哈希表、滑动窗口、双指针、数组 17 题

这份笔记用于自己刷题复盘。重点不是死背代码，而是看到题目时能快速判断：

```text
这题在维护什么状态？
状态怎么更新？
什么时候收缩、查找、去重？
```

## 总览表

| 模型 | 题目 | 难度 | 识别信号 | 核心动作 |
|---|---|---|---|---|
| 哈希表 | 两数之和 | 简单 | 找另一个数、查是否出现过 | `target - num` |
| 哈希表 | 字母异位词分组 | 中等 | 同一类字符串归组 | 排序后作为 key |
| 哈希集合 | 最长连续序列 | 中等 | 连续数字、要求 O(n) | 只从起点扩展 |
| 快慢指针 | 移动零 | 简单 | 原地调整数组 | 非零前移 |
| 双指针 | 盛最多水的容器 | 中等 | 两端选边界、最大面积 | 移动短板 |
| 排序 + 双指针 | 三数之和 | 中等 | 三元组、去重、和为 0 | 固定一个数转 2Sum |
| 双指针 | 接雨水 | 困难 | 左右边界、积水量 | 维护左右最大值 |
| 滑动窗口 | 无重复字符的最长子串 | 中等 | 最长、连续、不重复 | 右扩左缩 |
| 滑动窗口 | 找到所有字母异位词 | 中等 | 固定长度窗口、频率一致 | 比较计数器 |
| 前缀和 + 哈希 | 和为 K 的子数组 | 中等 | 连续子数组和、可能有负数 | 查 `prefix - k` |
| 滑动窗口 + 单调队列 | 滑动窗口最大值 | 困难 | 固定窗口、每个窗口最大值 | 队首保持最大值下标 |
| 滑动窗口 | 最小覆盖子串 | 困难 | 最短子串、覆盖目标字符 | 满足后尽量收缩 |

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

## 二、双指针模型：左右逼近、原地调整

双指针适合这类信号：

```text
数组有序或可以排序
两端向中间逼近
原地移动元素
根据左右状态决定移动哪一边
```

### 4. 移动零

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

### 5. 盛最多水的容器

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

### 6. 三数之和

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

### 7. 接雨水

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

## 三、滑动窗口模型：连续区间维护

滑动窗口适合这类信号：

```text
连续子串 / 连续子数组
最长 / 最短 / 所有满足条件的窗口
窗口右边扩张，条件不满足时左边收缩
```

### 8. 无重复字符的最长子串

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

### 9. 找到字符串中所有字母异位词

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

## 四、子串 / 子数组模型：前缀和、单调队列、覆盖窗口

这一组题都和连续区间有关，但维护方式不同：

```text
子数组和 -> 前缀和 + 哈希表
固定窗口最大值 -> 单调队列
最短覆盖子串 -> 可变滑动窗口 + 频率表
```

### 10. 和为 K 的子数组

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

### 11. 滑动窗口最大值

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

### 12. 最小覆盖子串

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

## 刷题决策树

看到题目后，可以按这个顺序判断：

```text
1. 要找某个数、某个状态是否出现过？
   -> 哈希表

2. 问连续数字序列，并要求 O(n)？
   -> set，找起点

3. 问原地移动元素、左右边界、面积、积水？
   -> 双指针，想清楚移动哪一边

4. 问连续子串 / 连续子数组，并且窗口状态可以维护？
   -> 滑动窗口

5. 问连续子数组和，并且数组可能有负数？
   -> 前缀和 + 哈希表

6. 问每个固定窗口的最大值？
   -> 单调队列
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

## 五、普通数组

这一组题继续接在前面 12 题后面，用来复盘 Hot100 普通数组。

### 普通数组总览表

| 题号 | 题目 | 难度 | 模型 | 核心 |
|---|---|---|---|---|
| 13 | 最大子数组和 | 中等 | 动态规划 / Kadane | 当前连续和变差就重新开始 |
| 14 | 合并区间 | 中等 | 排序 + 区间合并 | 按左端点排序，能合并就扩右边界 |
| 15 | 轮转数组 | 中等 | 数组切片 / 原地替换 | 后 `k` 个元素 + 前 `n-k` 个元素 |
| 16 | 除自身以外数组的乘积 | 中等 | 前缀乘积 + 后缀乘积 | 左边乘积 × 右边乘积 |
| 17 | 缺失的第一个正数 | 困难 | 原地哈希 | 数字 `x` 放到下标 `x - 1` |

### 13. 最大子数组和

题目核心：在数组里找一段连续子数组，让这段子数组的和最大。

```text
nums = [-2,1,-3,4,-1,2,1,-5,4]
最大连续子数组是 [4,-1,2,1]
最大和是 6
```

面试口述：

```text
我用 cur 表示“以当前数字结尾的最大子数组和”，用 ans 表示全局最大和。
遍历到 num 时，要么接在前面的子数组后面，要么从当前 num 重新开始。
所以 cur = max(cur + num, num)，然后用 cur 更新 ans。
```

```python
class Solution:
    def maxSubArray(self, nums: list[int]) -> int:
        cur = nums[0]
        ans = nums[0]

        for i in range(1, len(nums)):
            num = nums[i]
            cur = max(cur + num, num)
            ans = max(ans, cur)

        return ans
```

易错点：

- `cur` 不是全局最大值，而是“以当前数字结尾”的最大和。
- 如果前面的连续和是负数，它只会拖累当前数字，所以可以从当前数字重新开始。
- 初始化要用 `nums[0]`，不能用 0，因为数组可能全是负数。

一句话记忆：

```text
当前连续和如果变差了，就从当前数字重新开始；每一步维护全局最大值。
```

### 14. 合并区间

题目核心：把有重叠的区间合并成一个更大的区间。

```text
intervals = [[1,3],[2,6],[8,10],[15,18]]
输出 [[1,6],[8,10],[15,18]]
```

面试口述：

```text
我先按区间左端点排序。
然后遍历每个区间，如果当前区间的左端点小于等于结果中最后一个区间的右端点，说明可以合并。
合并时更新最后一个区间的右端点为两者右端点最大值。
如果不能合并，就把当前区间加入结果。
```

```python
class Solution:
    def merge(self, intervals: list[list[int]]) -> list[list[int]]:
        intervals.sort(key=lambda x: x[0])
        ans = []

        for start, end in intervals:
            if not ans or start > ans[-1][1]:
                ans.append([start, end])
            else:
                ans[-1][1] = max(ans[-1][1], end)

        return ans
```

易错点：

- 必须先按左端点排序，否则无法只看结果中的最后一个区间。
- 判断能否合并用 `start <= ans[-1][1]`。
- 合并时右端点取 `max(ans[-1][1], end)`，不能直接覆盖。

一句话记忆：

```text
先排序，再看当前区间能不能接到上一个合并区间后面。
```

### 15. 轮转数组

题目核心：把数组向右轮转 `k` 步。

```text
nums = [1,2,3,4,5,6,7], k = 3
结果是 [5,6,7,1,2,3,4]
```

面试口述：

```text
向右轮转 k 步，本质是把后 k 个元素移动到前面。
所以新数组等于 nums[-k:] + nums[:-k]。
题目要求原地修改时，用 nums[:] = ... 替换原列表内容。
```

```python
class Solution:
    def rotate(self, nums: list[int], k: int) -> None:
        n = len(nums)
        k %= n
        nums[:] = nums[-k:] + nums[:-k]
```

Python 知识点：

`nums[-k:]` 表示取后 `k` 个元素，`nums[:-k]` 表示取前 `n-k` 个元素，`+` 表示列表拼接，`nums[:] = new_list` 表示原地替换整个列表内容。

易错点：

- 要先写 `k %= n`，因为 `k` 可能大于数组长度。
- `nums[:] = ...` 是原地修改原列表；`nums = ...` 只是让局部变量指向新列表。
- 如果 `k == 0`，切片写法仍然能处理，但理解时要注意 `nums[-0:]` 等价于 `nums[0:]`。

一句话记忆：

```text
右转 k 步 = 后 k 个元素 + 前 n-k 个元素。
```

### 16. 除自身以外数组的乘积

题目核心：对每个位置 `i`，答案等于 `i` 左边所有数的乘积乘以 `i` 右边所有数的乘积。

```text
nums = [1,2,3,4]
answer = [24,12,8,6]
```

面试口述：

```text
不能用除法，所以我把答案拆成左边乘积和右边乘积。
第一遍从左到右，让 answer[i] 先存 nums[i] 左边所有数的乘积。
第二遍从右到左，用 right 表示 nums[i] 右边所有数的乘积，再乘到 answer[i] 上。
```

```python
class Solution:
    def productExceptSelf(self, nums: list[int]) -> list[int]:
        n = len(nums)
        answer = [1] * n

        left = 1
        for i in range(n):
            answer[i] = left
            left *= nums[i]

        right = 1
        for i in range(n - 1, -1, -1):
            answer[i] *= right
            right *= nums[i]

        return answer
```

关键理解：

```python
answer[i] = left
left *= nums[i]
```

这里必须先赋值，再更新 `left`，因为 `answer[i]` 要的是“不包含自己”的左边乘积。

易错点：

- 最左边没有左边元素，左边乘积当作 1。
- 最右边没有右边元素，右边乘积当作 1。
- 不能用除法，因为题目明确要求不要使用除法。

一句话记忆：

```text
先从左到右存左边乘积，再从右到左乘右边乘积。
```

### 17. 缺失的第一个正数

题目核心：找数组里没有出现的最小正整数，要求 `O(n)` 时间和 `O(1)` 额外空间。

为什么答案只可能在 `1` 到 `n + 1`？

```text
如果长度为 n 的数组刚好包含 1 到 n，那么答案是 n + 1。
如果 1 到 n 中有任何一个缺失，答案就在 1 到 n 之间。
```

面试口述：

```text
我把数组本身当作哈希表。
数字 x 如果在 1 到 n 之间，就应该放到下标 x - 1 的位置。
遍历数组时不断交换，把能归位的数字放回正确位置。
最后再扫描一遍，第一个 nums[i] != i + 1 的位置，答案就是 i + 1。
```

```python
class Solution:
    def firstMissingPositive(self, nums: list[int]) -> int:
        n = len(nums)

        for i in range(n):
            while 1 <= nums[i] <= n and nums[nums[i] - 1] != nums[i]:
                correct_index = nums[i] - 1
                nums[i], nums[correct_index] = nums[correct_index], nums[i]

        for i in range(n):
            if nums[i] != i + 1:
                return i + 1

        return n + 1
```

关键条件：

```python
while 1 <= nums[i] <= n and nums[nums[i] - 1] != nums[i]:
```

含义：

- `1 <= nums[i] <= n`：只处理可能影响答案的数字。
- `nums[nums[i] - 1] != nums[i]`：目标位置还没有这个数字，才交换，避免重复数字导致死循环。
- 用 `while` 不用 `if`：一次交换后，当前位置可能换来一个新的数字，这个新数字也可能需要继续归位。

易错点：

- 不能排序，因为排序通常不是 `O(n)`。
- 不能用哈希表，因为要求常数级额外空间。
- 数字 `x` 的正确下标是 `x - 1`。

一句话记忆：

```text
把每个数字 x 放到下标 x-1，最后第一个位置不对的地方，就是缺失的最小正数。
```

## 普通数组知识点总结

### 1. `nums[:] = nums[-k:] + nums[:-k]`

这句用于轮转数组：

```text
nums[-k:]  -> 后 k 个元素
nums[:-k]  -> 前 n-k 个元素
+          -> 拼接列表
nums[:] =  -> 原地替换原列表内容
```

注意：`nums[:] = ...` 会修改原列表本身；`nums = ...` 只是重新绑定变量，在 LeetCode 原地修改题里通常不符合要求。

### 2. 为什么 `answer[i] = left` 要写在 `left *= nums[i]` 前面？

因为 `answer[i]` 要的是 `nums[i]` 左边所有元素的乘积，不包含 `nums[i]` 自己。先保存当前 `left`，再把 `nums[i]` 乘进去，给下一个位置使用。

### 3. 为什么缺失的第一个正数用 `while`，不是 `if`？

因为交换一次后，当前位置可能换来另一个也需要归位的数字。用 `while` 可以一直处理当前位置，直到它不需要交换为止。

### 4. 为什么缺失的第一个正数只关心 `1 <= num <= n`？

长度为 `n` 的数组，缺失的第一个正数只可能在 `1` 到 `n + 1`。负数、0、大于 `n` 的数都不影响 `1..n` 这些位置是否完整。

### 普通数组一句话总结

```text
普通数组的核心是：连续状态、区间边界、原地修改、前后缀信息、把数组当哈希表。
```
