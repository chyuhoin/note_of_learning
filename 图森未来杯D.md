#### 图森未来杯D	Divisor解题报告

题目大意：给定$a,b,c,d$，要求判断$B=\frac{d!}{(c-1)!}$能不能被$A=\frac{b!}{(a-1)!}$整除。$10$组数据，$a,b,c,d$的范围在$10^7$以内。

最显然的办法是通过质因数分解，求出$B$和$A$的唯一分解，就是求出每个质数在$A$和$B$之间的个数，判断$B$中每一个质数的个数是不是都比$A$中该质数的数目多。写成数学语言就是：记$F_{P}(x)=y$表示$x\equiv0(\bmod p^{y})$且$x\not\equiv0 (\bmod p^{y+1})$，也就是$x$里，因数$p$的个数，其中$p$为质数。那么$a|b$的充要条件就是$\forall p,F_{p}(a) \geq F_{p}(b)$。

对于这道题，如果枚举从$a$到$b$，从$c$到$d$，每一个整数并且分解质因数，即使使用了$pollard-rho$算法也是会严重超时的。所以$10^7$级别的数据提醒我们不能把枚举的对象放在$a,b,c,d$上。

该题求的是$B=\frac{d!}{(c-1)!}$能不能被$A=\frac{b!}{(a-1)!}$整除，稍微转化一下就是判断${d!}{(a-1)!}$能不能被${b!}{(c-1)!}$整除。代入上边的数学公式，就是判断$\forall p,F_{p}({d!}{(a-1)!}) \geq F_{p}({b!}{(c-1)!})$是否成立。而且由于对于函数$F_{p}(x)$显然有$F_{p}(x_1x_2)=F_{p}(x_1)+F_p(x_2)$，所以最后这个式子就变成了$\forall p,F_{p}({d!})+F_{p}({(a-1)!})-F_{p}({b!})-F_{p}({(c-1)!}) \geq 0$。

所以我们只需要逐个枚举$p$，并且计算这个不等式是否成立就可以了。由于$10^7$以内质数只有$60$万个左右，只要我们能够在$O(logx)$的时间内计算出$F_{p}(x!)$的值就可以解决问题了。

接下来的问题其实就是“阶乘分解”这道题。如果你做过，那么接下来的内容你可以不用看了。接下来相当于是在讲解“阶乘分解”这道题。我们现在要计算$F_p(x!)$，先算一算$(1,x]$里有多少个数字是$p$的倍数。答案很简单，一共有$x/p$个，分别是$p,2p,3p... (x/p)p$。这$x/p$个数字需要全部累计入答案；然后同理，$(1,x]$里有$x/{p^2}$个数字是$p^2$的倍数，这$x/p^2$个数字又要累计入答案。

有细心的人可能会问，这些数字里包含了两个$p$，应该累计两次才对啊？其实在统计那$x/p$个数字的时候，这些数字已经被累计过一次了，这里只需要再累计一次就可以保证不重不漏。那么以此类推，$F_{p}(x!)=x/p+x/{p^2}+x/{p^3}+...$由于$p\geq 2$，枚举$logx$次就可以求出$F_{p}(x!)$的值了。

最后贴一份$AC$代码

```C++
#include<bits/stdc++.h>
#define poi 10200000
#define inf 0x3f3f3f3f
using namespace std;
typedef long long ll;
const int mod=1e9+7;
int v[poi],prime[poi],primecnt;
inline int re()
{
	char x=getchar();
	int k=1,y=0;
	while(x<'0'||x>'9')
	{if(x=='-') k=-1;x=getchar();}
	while(x>='0'&&x<='9')
	{y=(y<<3)+(y<<1)+x-'0';x=getchar();}
	return y*k;
}
void wr(int x)
{
	if(x<0) putchar('-'),x=-x;
	if(x>9) wr(x/10);
	putchar(x%10+'0');
}
void primes(int n)
{
	primecnt=0;
	for(int i=2;i<=n;i++)
	{
		if(v[i]==0)
		v[i]=i,
		prime[++primecnt]=i;
		for(int j=1;j<=primecnt;j++)
		{
			if(prime[j]>v[i]||prime[j]>n/i) break;
			v[i*prime[j]]=prime[j];
		}
	}
}
int fenjie(int x,int p)
{
    int ans=0;
    while(x)
    {
        x/=p;
        ans+=x;
    }
    return ans;
}
void work()
{
    int l1=re(),r1=re();
    int l2=re(),r2=re();
    for(int i=1;i<=primecnt;i++)
    {
        int cnt=0;
        cnt+=fenjie(r2,prime[i]);
        cnt+=fenjie(l1-1,prime[i]);
        cnt-=fenjie(r1,prime[i]);
        cnt-=fenjie(l2-1,prime[i]);
        if(cnt<0) {printf("No\n");return;}
    }
    printf("Yes\n");
}
signed main()
{
    primes(10000000);
    int ttt=re();
    while(ttt--) work();
	return 0;
}

```

