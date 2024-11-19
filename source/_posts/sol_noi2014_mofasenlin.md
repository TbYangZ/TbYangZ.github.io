---
title: 「NOI2014」魔法森林
tags:
- 题解
- OI
categories:
- 学习
description: NOI2014 魔法森林 题解
---

# 题目大意

给定一张图,每条边有两个边权 $a_i,b_i$，问 $1$ 到 $n$ 的所有路径中，$\max\lbrace a_i\rbrace+\max\lbrace b_i\rbrace$ 的最小值。

# 分析

如果没有 $b_i$，那么就是求 $1\sim N$ 的所有路径中最大边权的最小值。很容易想到用最小生成树解决。现在一条边有两个权值，同样按照 Kruskal 算法的思路，固定边的一种权值 $a_i$，将其按从小到大排序，然后就可以发现，答案中 $b_i$ 就一定在以 $b_i$ 为权值的最小生成树上，于是就可以用 LCT 维护 $b$ 权值的最小生成树。当 $1$ 和 $n$ 连通时就更新答案。对于自环与重边，可以不处理，LCT 会自动过滤。似乎还有一种用 Spfa 的方法，供大家思考。（LCT 也没那么长，压行后百行都不到！）

# 代码

```cpp
#include <algorithm>
#include <climits>
#include <cstdio>
using namespace std;
const int N=500005,M=100005;
const int V=N+M,INF=INT_MAX>>1;
typedef long long LL;
struct Edge {int u,v,ca,cb;}e[M];
int n,m,fa[N],ans;
bool cmp(Edge a,Edge b) {return a.ca<b.ca;}
int getf(int x) {return fa[x]==x?x:fa[x]=getf(fa[x]);}
int c[V][2],f[V];
int mx[V],v[V],rv[V],st[V];
void init(int x,int p) {mx[x]=v[x]=p;}
bool nroot(int x) {return c[f[x]][0]==x||c[f[x]][1]==x;}
void makerv(int x) {rv[x]^=1;swap(c[x][0],c[x][1]);}
void pushup(int x) {
	mx[x]=v[x];
	if (e[mx[c[x][0]]].cb>e[mx[x]].cb) mx[x]=mx[c[x][0]];
	if (e[mx[c[x][1]]].cb>e[mx[x]].cb) mx[x]=mx[c[x][1]];
}
void pushdown(int x) {
	if (!x||!rv[x]) return;
	if (c[x][0]) makerv(c[x][0]);
	if (c[x][1]) makerv(c[x][1]);
	rv[x]=0;
}
void pushtg(int x) {
	int top=0;
	st[++top]=x;
	while (f[x]) st[++top]=(x=f[x]);
	while (top) pushdown(st[top--]);
}
void rotate(int x) {
	int y=f[x],z=f[y];
	int k=c[y][1]==x;
	if (nroot(y)) c[z][c[z][1]==y]=x;
	f[x]=z;f[y]=x;
	if (c[x][k^1]) f[c[x][k^1]]=y;
	c[y][k]=c[x][k^1];c[x][k^1]=y;
	pushup(y);pushup(x);
}
void splay(int x) {
	pushtg(x);
	for (int y,z;y=f[x],z=f[y],nroot(x);rotate(x))
		if (nroot(y)) rotate((c[y][0]==x)^(c[z][0]==y)?x:y);
	pushup(x);
}
void access(int x) {for (int y=0;x;splay(x),c[x][1]=y,pushup(x),x=f[y=x]);}
void makert(int x) {access(x);splay(x);makerv(x);}
void split(int x,int y) {makert(x);access(y);splay(y);}
void link(int x,int y) {makert(x);f[x]=y;pushup(y);}
void cut(int x,int y) {split(x,y);f[x]=c[y][0]=0;pushup(y);}
int main() {
	scanf("%d%d",&n,&m);
	for (int i=1;i<=m;i++) {
		int u,v,a,b;
		scanf("%d%d%d%d",&u,&v,&a,&b);
		e[i]=(Edge){u,v,a,b};
	}
	sort(e+1,e+m+1,cmp);
	for (int i=1;i<=n;i++) init(i,0),fa[i]=i;
	for (int i=n+1;i<=n+m;i++) init(i,i-n);
	ans=INF;
	for (int i=1;i<=m;i++) {
		int u=e[i].u,v=e[i].v;
		int fu=getf(u),fv=getf(v);
		if (fu!=fv) link(u,i+n),link(i+n,v),fa[fu]=fv;
		else {
			split(u,v);
			int num=mx[v];
			if (e[num].cb>e[i].cb) {
				cut(e[num].u,num+n);cut(num+n,e[num].v);
				link(u,i+n);link(i+n,v);
			}
		}
		if (getf(1)==getf(n)) {
			split(1,n);
			ans=min(ans,e[i].ca+e[mx[n]].cb);
		}
	}
	if (ans==INF) printf("-1");
	else printf("%d",ans);
	return 0;
}
```

