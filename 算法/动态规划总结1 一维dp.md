动态规划的核心就是将问题分解成子问题,然后推导出公式,根据公式利用以前计算的结果进行计算.它需要枚举出所有的可能,和贪心算法不一样.
但是问题的难点在于,如何将问题分解并推导出公式呢.
比如可以枚举当前元素的状态,从而获得所有的可能.
先从`Leetcode`中简单的题目开始练习吧.

### [Leetcode 53. Maximum Subarray](https://leetcode.com/problems/maximum-subarray/)
需要计算连续子序列最大的和.
**Example**:
```
Input: nums = [-2,1,-3,4,-1,2,1,-5,4]
Output: 6
Explanation: [4,-1,2,1] has the largest sum = 6.
```
通常能想到的解法是这样的:
如果前面累加的和已经小于0,那么与当前的值累加,只会比当前值小,可以丢弃.起始点就从当前值开始继续累加;
如果前面累加的和大于0,那么就可以继续累加当前值:
```
class Solution(object):
    def maxSubArray(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        total = nums[0]
        res = total
        for i in range(1, len(nums)):
            if total <= 0:
                total = nums[i]
            else:
                total += nums[i]
            res = max(res, total)
        return res
```
乍一看,好像和动态规划有那么一点关系,但是却不是那么明朗的关系.所以每次我隔很久做的时候,都会因为这个逻辑想上一会.如果尝试用动态规划的思路来描述并解决它呢?
假设对于`nums`中的每一个位置`i`的元素,可以有两种情况:
1. 出现在了累加的子序列中,即这次累加包含了`nums[i]`
2. 没有出现,即这次我们还是采纳以前的累加`nums[k:m]`,其中`m<i`
那么,据此我们可以有两个数组记录累加和:
```
# 包含了nums[i]的累加和
dp = [float('-inf')] * len(nums) # dp[0] = nums[0]
# 不包含nums[i]
fun = [float('-inf')] * len(nums)
```
于是`dp`的值可以有两种取值:
1. 跟上`i-1`结尾的序列后
2. 自己为序列的开始
```
dp[i] = max(dp[i-1]+nums[i], nums[i])
```
而`fun`的取值呢,也有两种:
1. 采用以`i-1`为结尾的累加和
2. 采用不以`i-1`为结尾的累加和
```
fun[i] = max(dp[i-1], fun[i-1])
```
所以可以写出第一个版本的答案如下:
```
class Solution(object):
    def maxSubArray(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        dp = [float('-inf')] * len(nums)
        fun = [float('-inf')] * len(nums)
        dp[0] = nums[0]
        res = nums[0]
        for i in range(1, len(nums)):
            dp[i] = max(nums[i], nums[i]+dp[i-1])
            fun[i] = max(dp[i-1], fun[i-1])
            res = max(res, dp[i], fun[i])
        return res
```
可以发现,无论是`dp`还是`fun`,每次利用的都是前一个值,所以并不需要保存所有的中间值,因此将数组去掉,于是就有了第二个版本:
```
class Solution(object):
    def maxSubArray(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        dp = nums[0]
        fun = float('-inf')
        res = nums[0]
        for i in range(1, len(nums)):
            prev_d, prev_f = dp, fun
            dp = max(nums[i], nums[i]+prev_d)
            fun = max(prev_d, prev_f)
            res = max(res, dp, fun)
        return res
```
在很多时候,这种方式已经够了,但是是否可以再精简一下呢?
注意到这句:
```
fun = max(prev_d, prev_f)
```
也就是说`fun`的值取决于`dp`的历史值以及它自身的历史值,但是`fun`的初始值是`float(-'inf')`,也就是实际上它是依赖于`dp`的值而变化的,那么对它的记录与比较就是冗余的了,将它去掉,也就有了第三个版本:
```
class Solution(object):
    def maxSubArray(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        dp = nums[0]
        res = nums[0]
        for i in range(1, len(nums)):
            dp = max(nums[i], nums[i]+dp)
            res = max(res, dp)
        return res
```
其实最后就说明了一件事:**要么就加上`nums[i]`,要么,就从`nums[i]`开始**.而最开始的解法,只是更详细的描述了,在什么情况下,我们应该加上它,而什么时候又应该从它开始.

手里拿着锤子,看见什么都想钉一钉,再试试下面这题.
#### [Leetcode 152. Maximum Product Subarray](https://leetcode.com/problems/maximum-product-subarray/)
计算连续子序列的最大积.
**Example**:
```
Input: nums = [2,3,-2,4]
Output: 6
Explanation: [2,3] has the largest product 6.
```
如果还继续用上一题的思路最开始的解法,想着根据前序序列的正负,来决定是否要乘上当前值,情况就有点复杂了.因为最大值有可能来源于以前序列的最大值与当前值相乘,也有可能是最小值与当前值相乘.
因此,将问题扩展一下,不止记录是否乘上了当前值,还要记录乘上的最大值和最小值:
```
c = [float('-inf')] * len(nums)  #contains i max
mc = [float('inf')] * len(nums)  #contains i min
n = [float('-inf')] * len(nums)  #not contains i max
mn = [float('inf')] * len(nums)  #not contains i min
```
那么,可以写出第一个版本的答案:
```
class Solution(object):
    def maxProduct(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        c = [float('-inf')] * len(nums)  #contains i max
        mc = [float('inf')] * len(nums)  #contains i min
        n = [float('-inf')] * len(nums)  #not contains i max
        mn = [float('inf')] * len(nums)  #not contains i min


        c[0] = mc[0] = nums[0]
        res = nums[0]
        for i in range(1, len(nums)):
            c[i] = max(nums[i], c[i-1]*nums[i], mc[i-1]*nums[i])
            mc[i] = min(nums[i], mc[i-1]*nums[i], c[i-1]*nums[i])
            n[i] = max(c[i-1], n[i-1])
            mn[i] = min(mc[i-1], mn[i-1])
            res = max(res, n[i], c[i], mc[i], mn[i])
        return res
```
可以看出,只是在取最大值最小值的时候不放心,将以前的最大值和最小值都拿出来计算得出了结果.保证`c[i]`一定是乘上了`nums[i]`最大的值,因此`n[i]`一定是不包含`nums[i]`的最大的值.
既然是这样,那么在确定`res`的时候,就不需要和`mc[i]`以及`mn[i]`进行比较了.而`mn`的值也只有自己用上了,也可以去掉了:
```
class Solution(object):
    def maxProduct(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        c = [float('-inf')] * len(nums)  #contains i max
        mc = [float('inf')] * len(nums)  #contains i min
        n = [float('-inf')] * len(nums)  #not contains i max


        c[0] = mc[0] = nums[0]
        res = nums[0]
        for i in range(1, len(nums)):
            c[i] = max(nums[i], c[i-1]*nums[i], mc[i-1]*nums[i])
            mc[i] = min(nums[i], mc[i-1]*nums[i], c[i-1]*nums[i])
            n[i] = max(c[i-1], n[i-1])
            res = max(res, n[i], c[i])
        return res
```
再次发现,可以不用数组存储中间的状态,于是再精简一次:
```
class Solution(object):
    def maxProduct(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        n = float('-inf')
        c = mc = nums[0]
        res = nums[0]
        for i in range(1, len(nums)):
            prev_c, prev_mc, prev_n = c, mc, n
            c = max(nums[i], prev_c*nums[i], prev_mc*nums[i])
            mc = min(nums[i], prev_mc*nums[i], prev_c*nums[i])
            n = max(prev_c, prev_n)
            res = max(res, n, c)
        return res
```
然后又能发现和上题一样的问题,`n`的值取决于`c`的值,所以最终的版本也不再需要`n`了:
```
class Solution(object):
    def maxProduct(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        c = mc = nums[0]
        res = nums[0]
        for i in range(1, len(nums)):
            prev_c, prev_mc = c, mc
            c = max(nums[i], prev_c*nums[i], prev_mc*nums[i])
            mc = min(nums[i], prev_mc*nums[i], prev_c*nums[i])
            res = max(res, c)
        return res
```
虽然比起大神的解答还有差距,但是至少不会再忘记了~
其实总结起来也就是枚举了所有的可能,只不过采用了`max`,过滤掉了一些不符合的情况,也就是进行了**剪枝**.
类似的问题还有(待更新):
经典的爬楼梯:[Leetcode 70. Climbing Stairs](https://leetcode.com/problems/climbing-stairs/)