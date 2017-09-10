---
title: 给定偶数数量的序列，分为两部分使得差最小
---
> 该题类似于01背包问题
使用动态规划求解，首先对序列求和为sum，那么若要两部分差最小，即 |任一部分的和-sum/2| 最小。  
  
  
动态规划首先找出子问题，用dp[i][j]表示前i个数字中，总和为j的最优解。  
如果i+1没有加入其中，那么dp[i+1][j] = dp[i][j];  
如果i+1加入其中，那么dp[i+1][j] = dp[i][j-a[i+1]];  
动态转移方程为
dp[i+1][j] = max(dp[i][j] , dp[i][j-a[i+1]] + a[i+1]);  
代码如下：

```
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;
int main() 
{
	int n = 0;
	cin >> n;
	int num = 0;
	vector<int> s(n);
	for (int i = 0; i < n; i++) 
	{
		cin >> num;
		s.push_back(num);
	}
	diff(s);
	return 0;
}
int diff(vector<int>& vec)
{
	int len = vec.size();
	int sum = 0;
	for (int i = 0; i < len; ++i) 
	{
		sum += vec[i];
	}
	vector<vector<int>> dp;
	for (int i = 0; i <= len; i++) 
	{
		vector<int>tmp;
		for (int j = 0; j <= sum / 2; ++j) 
		{
			tmp.push_back(0);
		}
		dp.push_back(tmp);
	}
	for (int i = 1; i <= len; ++i) 
	{
		for (int j = 1; j <= sum / 2; ++j) 
		{
			if (j >= vec[i - 1])dp[i][j] = max(dp[i - 1][j], dp[i - 1][j - vec[i - 1]] + vec[i - 1]);
			else dp[i][j] = dp[i - 1][j];
		}
	}
	cout << sum - 2 * dp[len][sum / 2];
        return 0;
}
```
  
  
01背包问题：  
有一个背包，背包容量为m，有一堆物品，每个的重量为a[i],每个的价值为b[i]，物品装入背包怎样才能使拿着的背包价值最高？  
  
思路类似遇上题，用dp[i][j]表示前i个背包中，总容量为j的最优解。  
如果i+1编号的背包没有拿，那么dp[i+1][j] = dp[i][j];  
如果i+1编号的背包拿了，那么dp[i+1][j] = dp[i][j-a[i+1]];    
动态转移方程为dp[i+1][j] = max(dp[i][j] , dp[i][j-a[i+1]] + b[i+1]);（前提是背包装的下）  
代码如下：  

```
#include<iostream>  
using namespace std;  
#define  V 1500  
unsigned int f[V];//全局变量，自动初始化为0  
unsigned int weight[10];  
unsigned int value[10];  
#define  max(x,y)   (x)>(y)?(x):(y)  
int main()  
{  
      
    int N,M;  
    cin>>N;//物品个数  
    cin>>M;//背包容量  
    for (int i=1;i<=N; i++)  
    {  
        cin>>weight[i]>>value[i];  
    }  
    for (int i=1; i<=N; i++)  
        for (int j=M; j>=1; j--)  
        {  
            if (weight[i]<=j)  
            {  
                f[j]=max(f[j],f[j-weight[i]]+value[i]);  
            }             
        }  
      
    cout<<f[M]<<endl;//输出最优解  
  
}  
```

最大连续子序列和问题也是类似上述。  

完全背包问题：  
和01背包问题不同的是每种背包都不只是一个，而是无穷个。  
我们顺着0-1背包的思想，既然选择可以是多件那么dp[ i ][ v ]=max( dp[ i-1 ][ v ], dp[ i-1 ][ v- k*c[ i ] ] + k*w[ i ] );这是在0-1背包的基础上得来的，就是多了一次k的循环，时间复杂度较高，尝试对其进行改进。  
完全背包的一个很有效的优化，就是对于价值大/体积小的物品和价值小/体积大的物品，我们当然优先选择物美价廉的，因为每件可以选择n次，我们当然要尽可能地的选择价值体积比大的，还有价值体积比相等的，我们可以很大胆的直接把其中体积大的直接舍去，这样对于数据较大的题来说可以很大的优化，当然不能排除特殊情况。








