### 2140.Solving-Questions-With-Brainpower

#### 解法1

因为n是1e5数量级，我们期望能用o(n)的时间复杂度解决。我们尝试令dp[i]表示前i个元素能取得的最大值，特别注意到，dp一定是递增的序列。

第一种“填表”思路是用已知的dp来计算当前的dp[i]。显然，如果我们不取第i个元素，那么dp[i]=dp[i-1]；如果我们取第i个元素，那么就有```dp[i] = dp[j] + val[i], where j<i```，其中要求i必须在j的冷冻期之后。但这里有个bug，假设dp[j]=dp[j-1]（对应我们取第j-1个元素，但是不取第j个元素），但j的冷冻期很短，而j-1的冷冻期很长。这样可能i没有落在j的冷冻期，但是落在了j-1的冷冻期。此时我们计算```dp[i] = dp[j] + val[i]```就出现了问题，因为dp[j]的本质是依赖于dp[j-1]的，必须考虑的是j-1的冷冻期。

第二种“刷表”思路是用当前的dp[i]来更新未来的DP，尝试构造```dp[i+skip+1] = max{dp[i+skip+1], dp[i] + val[i+skip+1}```. 这样也是有问题的。类似的，假设dp[i]=dp[i-1]（对应取第i-1个元素，但是不取第i个元素）。但i的冷冻期很短，而i-1的冷冻期很长。这样可能i+skip+1没有落在i的冷冻期，但是落在了i-1的冷冻期。此时我们用上面的式子就出现了问题，因为dp[i]的本质是依赖于dp[i-1]的，必须考虑的是i-1的冷冻期。

本题的思路非常巧妙，就是从后往前遍历。同样dp[i]表示从i到第n-1个元素能取得的最大值。如果我们不取第i个元素，那么```dp[i] = dp[i+1]```；如果我们取第i个元素，那么必然有```dp[i] = dp[i+skip+1] + val[i]```. 无论dp[i+skip+1]对应的是取还是不取第i+skip+1个元素，都不会冷冻到第i个元素的选取。最终dp[i]就是在这两种决策中取最大值，即```max(dp[i+1], dp[i+skip+1]+val[i])```. 最终的答案是dp[0].


#### 解法2
事实上本题存在传统的“填表法”DP，不过需要分别定义dp[i][0]和dp[i][1]表示第i个元素不取、取各自所能得到最大价值。

我们有状态转移方程：
```
dp[i][0] = max(dp[i-1][0], dp[i-1][1]);
dp[i][1] = max{dp[j][1]} + val[i]; 
```
其中第二项中的j是所有小于i的位置里、满足j的冷冻期不包括i、且dp[j][1]最大那个位置。我们如何不遍历全部、高效地找到那个j呢？我们提前把所有的index按照冷冻期结束（即j+skip[j]）的先后顺序排序。我们在从小到大遍历i的过程中，就可以顺序解锁那些j+skip[j]<i的位置j，然后滚动保留最大的dp[j][1]即可。这样，DP转移的过程就是o(n)的复杂度。

最终返回的答案是max{dp[n-1][0],dp[n-1][1]}。

#### 解法3：
类似地，我们也有传统的“刷表法”DP。同样定义dp[i][0]和dp[i][1]表示第i个元素不取、取各自所能得到最大价值。

在已知dp[i][0]和dp[i][1]的情况下，对于未来的状态，我们有状态转移方程：
```
dp[i+1][0] = max(dp[i+1][0], max(dp[i][0],dp[i][1]));
dp[j][1] = max(dp[j][1], dp[i][1] + val[j]);  for all j>i+skip[i]
```
第一项比较容易理解，第i+1个元素不取的话，dp[i+1]一定就完全取决于dp[i]了.

对于第二项，我们注意到，我们必须同时更新多个dp[j][1]的值。如何高效地实现这个操作呢？我们联想到差分数组的策略。第二项的意思是从j=i+skip[i]+1开始到结束，每一个j的dp[j][1]都至少被抬升到了dp[i][1]的大小。所以我们可以新建一个数组```base[i+skip[i]+1]```来表示从j=i+skip[i]+1开始到结束，所有的dp[j][1]都被统一抬升dp[i][1]。

我们在遍历i的过程中，需要滚动更新当前的抬升```diff = max(diff, base[i])```，然后更新```dp[i][1] = diff + questions[i][0]```。最后根据当前的dp[i][1]来更新未来的base[j]。

最终返回的答案是max{dp[n-1][0],dp[n-1][1]}。
