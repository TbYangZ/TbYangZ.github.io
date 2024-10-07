---
title: 「SDOI2010」古代猪文
description: SDOI2010 古代猪文 题解
---

# 题目大意

给定 $n, G$，求
$$
G^{\sum_{d|n}\binom{d}{n}} \text{mod } {999911659}
$$

# 分析

若 $G\equiv0\pmod {999911659}$，那么答案为 $0$。否则，可以发现 $999911659$ 是一个质数，使用欧拉定理的推论：

> $\mathrm{gcd}\,(a,p)=1$，则 $a ^ b \equiv a^{b\,\text{mod}\,\varphi(p) } \pmod{p}$。

将模数转移到指数上，变为求
$$
 \sum _{ d | n } \binom{d}{n}\text{ mod }999911658
$$
求完后用快速幂就得到答案了。

然后这东西看着可以用 Lucas 定理搞，但显然 $999911658 = 2\times3\times4679\times35617$ 不是质数。使用扩展Lucas的方法，将其进行质因数分解后，分别求$\sum _{ d|n} \binom{d}{n}$ 模其质因数，设得到的结果为 $a_1,a_2,a_3,a_4 $，然后再用中国剩余定理求解以下方程组：
$$
\begin{equation*}
\begin{cases}
&x\equiv a_1\pmod{2}\\
&x\equiv a_2\pmod{3}\\
&x\equiv a_3\pmod{4679}\\
&x\equiv a_4\pmod{35617}\\
\end{cases}
\end{equation*}
$$
设该方程组的最小正整数解为 $x$，则答案为 $G^x$。

# 代码

```cpp
#include <iostream>
#include <cstdio>
#include <cmath>
using namespace std;
typedef long long LL;
const LL P=999911659;
const LL N=100005;
LL fac[N];
LL md[5]={4,2,3,4679,35617};
LL b[5],G,n;
LL ksm(LL x,LL y,LL p) {//快速幂 
	x%=p;LL res=1LL;
	for (;y;y>>=1,x=x*x%p)
		if (y&1) res=res*x%p;
	return res;
}
void initfac(LL n,LL p) {//阶乘预处理 
	fac[0]=1;
	for (int i=1;i<=n;i++)
		fac[i]=fac[i-1]*i%p;
}
LL C(LL n,LL m,LL p) {//计算C(n,m)
	if (m>n) return 0;
	return (fac[n]*ksm(fac[m],p-2,p))%p*ksm(fac[n-m],p-2,p);
}
LL Lucas(LL n,LL m,LL p) {//Lucas定理C(n,m)%p=C(n/p,m/p)*C(n%p,m%p)%p
	if (!m) return 1;
	return (Lucas(n/p,m/p,p)%p*C(n%p,m%p,p)%p)%p;
}
LL Exgcd(LL a,LL b,LL&x,LL&y) {//扩展欧几里得 
	if (!b) {
		x=1,y=0;
		return a;
	}
	LL d=Exgcd(b,a%b,x,y),t;
	t=x;
	x=y;
	y=t-(a/b)*y;
	return d;
}
LL CRT(LL *a,LL *m,LL n) {//中国剩余定理x==ai(mod mi)
	LL M=1,ans=0;
	for (int i=1;i<=n;i++) M*=m[i];
	for (int i=1;i<=n;i++) {
		LL x,y;
		Exgcd(M/m[i],m[i],x,y);
		x=(x%m[i]+m[i])%m[i];//求逆元 
		ans=(ans+a[i]*x%M*M/m[i])%M;
	}
	return (ans%M+M)%M;
}
LL Solve(LL n,LL p) {
	LL ans=0,m=sqrt(n);
	initfac(N-5,p);
	for (LL i=1;i<=m;i++)//枚举约数 
		if (n%i==0) {
			ans+=Lucas(n,i,p);
			if (i*i!=n) ans+=Lucas(n,n/i,p);//去除重复的一个 
		}
	return ans;
}
int main() {
	scanf("%lld%lld",&n,&G);
	if (G%P!=0) {
		for (int i=1;i<=4;i++)
			b[i]=Solve(n,md[i]);
		printf("%lld",ksm(G,CRT(b,md,4),P));
	} else printf("0");
    return 0;
}
```

