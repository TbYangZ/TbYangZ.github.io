---
title: 「BZOJ2500」幸福的道路
description: BZOJ2500 幸福的道路 题解
---

# 题目大意

给定一棵树，树上每个点的权值为该点出发的最长路。之后选一个最长的区间 $\left [l,r\right ]$，使得点的编号在这个区间里的权值的最大值与最小值之差的小于等于 $M$。

# 分析

首先，需要求出从每个点出发的最长路经，可以用两次树型 Dp 实现。首先求出从每个点往下的最长路和次长路，记为 $f_i,g_i$，再求出每个点走它的父亲的最长路，记为 $h_i$，则从每个点的最长路为 $\max\left\lbrace f_i,g_i\right\rbrace$。记 $son_i$ 为节点 $i$ 的最长路的儿子，为 $0$ 则有多个，那么转移如下：

$$
\begin{equation*}
\begin{split}
&f_u=\max\lbrace f_v + w_i \rbrace\newline
&g_u=\max\lbrace f_v+w_i \rbrace \text{ if } f_v+w_i\le f_u\newline
&h_u=w_i+
\begin{cases}
\max\lbrace h_{prt},f_{prt} \rbrace \text{ if } son_{ptr} \neq u \text{ or } son_{ptr} =0\newline
\max\lbrace h_{prt},g_{prt} \rbrace \text{ if } son_{ptr} = u
\end{cases}
\end{split}
\end{equation*}
$$

其中，圆括号里为条件，$w_i$ 为对应边的权值，$v$ 为 $u$ 的儿子，$prt$ 为 $u$ 的父亲。

求出从每个点开始的最长路径（记为 $a_x$），然后可以用 ST 表预处理出 $Fmax,Fmin$ 数组，方便求区间最小最大值。先考录朴素做法：对于每个 $i\in \left[1,n\right]$，求出它能扩展的最大的 $p$ 满足它们之间的最大值与最小值的差小于等于 $M$。然后因为最值满足区间加法，故可以用倍增优化这个寻找的过程：令 $j=\log (n-i+1), p=i$，记录一个整个区间的最值，记为 $Max, Min$，初始值为$a_x$，查询区间$\left[p, p+2^j-1\right]$的最值，与记录的整个区间最值合并，若此时它们的差仍然小于等于 $M$，则令 $p=p+2^j$，否则不变，每次 $j-1$，一直循环下去，直到 $j=-1$ 跳出循环，此时从 $i$ 开始扩展的最长区间为 $i-p$，更新答案。

时间复杂度 $O(n\log n)$，没有单调队列 $O(n)$ 的做法快，但对于 $n\le 2\times10^5$ 来说已经足够。

# 代码

```cpp
#include <iostream>
#include <cstring>
#include <cstdio>
#include <cmath>
#include <queue>
using namespace std;
const int N=200004;
const int INF=0x3f3f3f3f;
struct Edge {
	int to,nxt,wei;
}e[N<<1];
int head[N],cnt,lg[N];
int n,m,h[N],ans;
int Fmin[N][24],Fmax[N][24];
int son[N];
int f[N],g[N];
void Add_Edge(int x,int y,int z) {
	e[++cnt]=(Edge){y,head[x],z};
	head[x]=cnt;
}
void Dfs(int x,int fa) {
	for (int i=head[x];i;i=e[i].nxt) {
		int y=e[i].to;
		if (y==fa) continue;
		Dfs(y,x);
		if (f[y]+e[i].wei>f[x]) {//更新最长路和次长路 
			g[x]=f[x];
			son[x]=y;
			f[x]=f[y]+e[i].wei;
		} else if (f[y]+e[i].wei==f[x]) {//有相等的,就说明有多个最长路,更新次长路与son 
			g[x]=f[x];
			son[x]=0;
		} else if (f[y]+e[i].wei>g[x]) g[x]=f[y]+e[i].wei;//更新次长路 
	}
}
void Dfs2(int x,int fa) {
	for (int i=head[x];i;i=e[i].nxt) {
		int y=e[i].to;
		if (y==fa) continue;
		if (!son[x]||son[x]!=y) h[y]=max(h[x],f[x])+e[i].wei;//走父亲的h[]与父亲的最长路 
		else h[y]=max(h[x],g[x])+e[i].wei;//走父亲的次长路 
		Dfs2(y,x);
	}
}
void getM(int l,int r,int &maxx,int &minn) {//ST表查询 
	int k=lg[r-l+1];
	maxx=max(Fmax[l][k],Fmax[r-(1<<k)+1][k]);
	minn=min(Fmin[l][k],Fmin[r-(1<<k)+1][k]);
}
int main() {
	scanf("%d%d",&n,&m);
	for (int i=1;i<n;i++) {
		int fa,wei;
		scanf("%d%d",&fa,&wei);
		Add_Edge(fa,i+1,wei);
		Add_Edge(i+1,fa,wei);
	}
	Dfs(1,0);
	Dfs2(1,0);
	for (int i=1;i<=n;i++) {
		Fmin[i][0]=Fmax[i][0]=h[i]=max(h[i],f[i]);
	}
	for (int j=1;(1<<j)<=n;j++)//ST表预处理 
		for (int i=1;i+(1<<j)-1<=n;i++) {
			Fmax[i][j]=max(Fmax[i][j-1],Fmax[i+(1<<(j-1))][j-1]);
			Fmin[i][j]=min(Fmin[i][j-1],Fmin[i+(1<<(j-1))][j-1]);
		}
	lg[0]=-1;
	for (int i=1;i<=n;i++)
		lg[i]=lg[i>>1]+1;
	for (int i=1;i<=n;i++) {//枚举起点 
		int p=i,maxx=h[i],minn=h[i];
		for (int j=lg[n-i+1];j>=0;j--) {//倍增找最多能扩展到的点 
			int tmax,tmin;
			getM(p,p+(1<<j)-1,tmax,tmin);
			tmax=max(tmax,maxx);
			tmin=min(tmin,minn);
			if (tmax-tmin<=m) {//满足条件,向前跳2^j步 
				maxx=tmax;
				minn=tmin;
				p=p+(1<<j);
			}
		}
		ans=max(ans,p-i);//更新答案 
	}
	printf("%d",ans);
	return 0;
}
```

