---
title: 「NOI2018」归程
tags:
- 题解
- OI
categories:
- 学习
description: NOI2018 归程 题解
---

# 题目大意

给定一张 $n$ 个点 $m$ 条边的无向连通图，每条边带两个权值 $l,a$，每次询问给出 $v,p$，要求从 $v$ 点开始，可以走边 $a>p$ 的边，路程为 $0$，不能走后，走其他边，路程为 $l$，求从 $v$ 开始到 $1$ 的最短路程。部分数据强制在线。

# 分析

对于每次询问给定的 $v,p$，若点 $x,y$ 之间最大边的最小值大于 $p$，则 $x,y$ 可以互相到达。于是可以想到构建 Kruskal 重构树，边权从大到小排序建树。由于这样建树后，对于 $v$ 能到达的节点，肯定在以 $v$ 的一个权值大于 $p$ 且层数最低的祖先的子树中。于是我们可以先跑一遍从 $1$ 开始的最短路，计算每个点到 $1$ 的最短路径，然后对于 Kruskal 重构树进行一次 dfs，求出每个点的子树中叶子节点的距离最小值。至于找这个祖先，可以用倍增去找。因为这棵树呈一棵小根堆，满足儿子节点的点权一定大于等于父节点的点权（叶子节点除外），故可以用倍增。这样可以做到在线回答。

对于离线的子问题，可以将边按 $a$ 值排序，询问按照 $p$ 值排序，两个指针扫过去，用并查集维护连通以及连通块内到 $1$ 的距离最小值。基于这种思想，就有人想到用可持久化并查集代替普通的并查集做到在线回答。

还有这道题卡 SPFA，只能用 Djikstra。

代码只有 Kruskal 重构树。

# 代码

```cpp
#include <algorithm>
#include <iostream>
#include <cstring>
#include <cstdio>
#include <queue>
using namespace std;
const int N=200050*2;
const int M=400050;
const int INF=0x3f3f3f3f;
typedef long long LL;
struct Edge {int to,nxt;LL wei;};
struct Array {int x,y;LL w;};
Edge e[M<<1];
Array p[M];
int n,m,f[N][31];
int h[N],cnt,fa[N];
int n_,q,T;
int L[N],R[N],k;
LL v[N],lastans,val[N],len;
bool cmp(Array a,Array b) {return a.w>b.w;}
void Add_Edge(int x,int y,int z) {e[++cnt]=(Edge){y,h[x],z};h[x]=cnt;}
int getf(int x) {return fa[x]==x?x:fa[x]=getf(fa[x]);}
void Clear() {//清空 
	memset(f,0,sizeof(f));
	memset(L,0,sizeof(L));
	memset(R,0,sizeof(R));
	memset(val,0x3f,sizeof(val));
	memset(v,0,sizeof(v));
	memset(h,0,sizeof(h));
	cnt=lastans=0;
}
int Ex_Kruskal() {//Kruskal重构树 
	sort(p+1,p+m+1,cmp);
	int ind=n_,lim=n_*2;
	for (int i=1;i<=lim;i++) fa[i]=i;
	for (int i=1;i<=m;i++) {
		int x=getf(p[i].x),y=getf(p[i].y);
		if (x!=y) {
			fa[x]=fa[y]=++ind;
			L[ind]=x,R[ind]=y;//由于是个二叉树,记录左右儿子就行了 
			val[ind]=p[i].w;
			if (ind==n_*2-1) break;
		}
	}
	return ind;
}
struct Node {//Dijkstra 
	LL d;
	int x;
	Node(LL d=0,int x=0):d(d),x(x){}
};
bool operator<(const Node&a,const Node&b) {return a.d>b.d;}
LL dis[N];
int vis[N];
void Dijkstra(int s) {
	fill(dis,dis+N,1e17);dis[s]=0;
	memset(vis,0,sizeof(vis));
	priority_queue<Node> q;
	q.push(Node(0,s));
	while (!q.empty()) {
		int x=q.top().x;
		q.pop();
		if (vis[x]) continue;
		vis[x]=1;
		for (int i=h[x];i;i=e[i].nxt) {
			int y=e[i].to;
			if (dis[y]>dis[x]+e[i].wei&&!vis[y]) {
				dis[y]=dis[x]+e[i].wei;
				q.push(Node(dis[y],y));
			}
		}
	}
}
void Dfs(int x) {//初始化子树叶子节点到1的最小值 
	if (!L[x]&&!R[x]) {
		v[x]=dis[x];
		return;
	}
	f[L[x]][0]=x,f[R[x]][0]=x;
	Dfs(L[x]),Dfs(R[x]);
	v[x]=min(v[L[x]],v[R[x]]);
}
int Find(int x,int d) {//倍增找点 
	for (int i=30;~i;i--)
		if (f[x][i]&&val[f[x][i]]>d) x=f[x][i];
	return x;
}
void Read() {//数据读入 
	scanf("%d%d",&n_,&m);
	for (int i=1;i<=n_;i++) val[i]=INF;
	for (int i=1;i<=m;i++) {
		int u,v,w,l;
		scanf("%d%d%d%d",&u,&v,&l,&w);
		p[i]=(Array){u,v,w};
		Add_Edge(u,v,l);
		Add_Edge(v,u,l);
	}
	scanf("%d%d%lld",&q,&k,&len);
}
void Init() {
	n=Ex_Kruskal(); 
	Dijkstra(1);
	Dfs(n);
	for (int j=1;(1<<j)<=n;j++)//倍增初始化 
		for (int i=1;i<=n;i++)
			f[i][j]=f[f[i][j-1]][j-1];
}
void Solve() {//回答 
	while (q--) {
		int u;
		LL d;
		scanf("%d%lld",&u,&d);
		u=(u+k*lastans-1)%n_+1;
		d=(d+k*lastans)%(len+1);
		printf("%lld\n",lastans=v[Find(u,d)]);
	}
}
int main() {
	scanf("%d",&T);
	while (T--) {
		Clear();
		Read();
		Init();
		Solve();
	}
	return 0;
}
```

