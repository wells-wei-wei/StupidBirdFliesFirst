<!--
 * @Author: your name
 * @Date: 2020-06-14 19:55:52
 * @LastEditTime: 2020-06-14 20:16:33
 * @LastEditors: Please set LastEditors
 * @Description: In User Settings Edit
 * @FilePath: \undefinedc:\Users\conan\Desktop\LongTime\StupidBirdFliesFirst\Algorithm\DynamicProgramming.md
--> 
# 动态规划
## 基本原则
动态规划最基本的原则是找到**最优子结构**和**状态转移方程**

## 原理解读
动态规划与递归是一种演进的关系：**递归的暴力解法**->**带备忘录的递归**->**非递归的动态规划**

拿最简单的斐波那契数列来说，**递归的暴力解法**如下所述：
```
int fib(int N) {
    if (N == 1 || N == 2) return 1;
    return fib(N - 1) + fib(N - 2);
}
```
这种最基本的递归是非常费时的，算法复杂度为 O(2^n)，指数级别。

但是经过观察就能知道其中其实有很多你数字是被重复计算的，比如计算N=20时，需要计算N=19和N=18的情况，而计算N=19就要计算N=18和N=17的情况，这里N=18就被计算了两回，因此可以将其记录下来，如果下次再用的时候就能直接拿来用，也就是**带备忘录的递归**：
```
int fib(int N) {
    if (N < 1) return 0;
    // 备忘录全初始化为 0
    vector<int> memo(N + 1, 0);
    return helper(memo, N);
}
int helper(vector<int>& memo, int n) {
    if (n == 1 || n == 2) return 1;
    if (memo[n] != 0) return memo[n];
    // 未被计算过
    memo[n] = helper(memo, n - 1) + helper(memo, n - 2);
    return memo[n];
}
```
这就是带记录的算法。这里的主体思想跟递归其实是一样的，但是在操作上则是动态规划的思想，因为这种思想是**自顶向下**的，所谓自顶向下就是由一个规模很大的原问题（例如N=20）分解为规模更小的问题（N=19和N=18），而通过记录减少计算则是动态规划的思想。如果要继续发挥“记录”的效果的话需要转变思想，将自顶向下改为**自底向上**，也就是从N=1和N=2开始，逐渐扩展到N=20，此即为**非递归的动态规划**：
```
int fib(int N) {
    vector<int> dp(N + 1, 0);
    dp[1] = dp[2] = 1;
    for (int i = 3; i <= N; i++)
        dp[i] = dp[i - 1] + dp[i - 2];
    return dp[N];
}
```
这里就突出了**最优子结构**和**动态转移方程**的概念。需要将原问题分解为一个一个最优子结构，然后由状态转移方程将他们连接起来，最终得到当前问题的解。