### Part 1

#### #70. 爬楼梯

求路径数目。

```cpp
class Solution {
public:
    int climbStairs(int n) {
        if (n <= 3) return n;
        int first = 1, second = 2;
        --n;
        while (--n) std::swap(first+=second, second);
        return second;
    }
};
```

#### #120. 三角形最小路径和

```cpp
#include <algorithm>

class Solution {
public:
    int minimumTotal(vector<vector<int>>& triangle) {
        if (triangle.size() == 1) return triangle[0][0];
        for (int i = triangle.size()-1; --i >= 0;)
            for (int j = triangle[i].size(); --j >= 0;)
                triangle[i][j] += std::min(triangle[i+1][j], triangle[i+1][j+1]);
        return triangle[0][0];
    }
};
```



### Part 2

#### #322. 零钱兑换

最优子结构，记忆化。

```cpp
class Solution {
   public:
    int coinChange(std::vector<int>& coins, int amount) {
        if (!amount) return 0;
        const int N = amount + 1;
        std::vector<int> dp(amount + 1, N);
        dp[0] = 0;
        for (int i = 1; i < N; ++i) {
            for (auto itr = coins.begin(); itr != coins.cend(); ++itr) {
                if (*itr <= i) {
                    dp[i] = std::min(dp[i], dp[i - *itr] + 1);
                }
            }
        }
        return dp[amount] == N ? -1 : dp[amount];
    }
};
```

#### #152. 乘积最大子数组

至少包含一个数。

```cpp
struct Pair {
    int mini;
    int maxi;
    void keepi(int n) {
        mini *= n;
        maxi *= n;
        *this = {std::min(maxi, std::min(n, mini)),
                 std::max(mini, std::max(n, maxi))};
    }
    void keepa(Pair& p) {
        if (p.mini < mini) mini = p.mini;
        if (p.maxi > maxi) maxi = p.maxi;
    }
};

class Solution {
   public:
    int maxProduct(std::vector<int>& nums) {
        const int sz = nums.size();
        if (sz == 1) return nums[0];

        // contains nums[i] for sure
        Pair p{nums[0], nums[0]}, ans(p);
        for (size_t i = 1; i < sz; ++i) {
            p.keepi(nums[i]);
            ans.keepa(p);
        }
        return ans.maxi;
    }
};
```

#### #188. 买卖股票的最佳时机 IV

K次交易内。

```cpp
class Solution {
   public:
    int maxProfit(int k, std::vector<int>& prices) {
        const int N = prices.size();
        if (N <= 1) return 0;
        const int K = std::min(N / 2, k);
        std::vector<std::pair<int, int>> mp(K + 1);

        // the first day
        mp[0] = {0, -prices[0]};  // not buy or buy
        for (int k = 1; k <= K; ++k) mp[k] = {INT_MIN / 2, INT_MIN / 2};

        // the next day
        for (int i = 1; i < N; ++i) {
            mp[0].second = std::max(mp[0].second, mp[0].first - prices[i]);
            for (int k = 1; k <= K; ++k) {  // not hold or hold
                mp[k] = {std::max(mp[k].first, mp[k - 1].second + prices[i]),
                         std::max(mp[k].second, mp[k].first - prices[i])};
            }
        }

        int ans = mp[0].first;
        for (int k = 1; k <= K; ++k)
            if (ans < mp[k].first) ans = mp[k].first;
        return ans;
    }
};
```

