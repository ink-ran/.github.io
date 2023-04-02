---
title: Acwing周赛
date: 2023-03-27 23:23:08
tags:
---

# 第92场

## 4865.有效类型

建立二叉树，递归

```c#
string s;
bool suan()
{
    char a[5];
    if (scanf("%s",a)==-1) return false;//和==EOF的意思一样
    if(a[0]=='p') //!strcmp(str, "pair") ：str=="pair"返回0 strcmp函数看#include<cstring>
    {
        s+=a;       //用s=s+a;超时
                    //s+=a与s=s+a;不同 
        s+='<';
        if(suan()!=true) return false;
        s+=',';
        if(suan()!=true) return false;
        s+='>';
    }
    else s+=a;
    return true;
}

int main()
{
    int n=0;
    scanf("%d\n",&n);
    if(suan()==true&& scanf("%*s") == -1) printf(s.c_str());
      //scanf("%*s") == -1 无输入时等于-1
    else printf("Error occurred\n");
}
```

## 4866.最大数量

1.采用并查集将连通的点并为一个集合，并采用一个数组记录该集合内有多少个点。若当前两个点已连通，说明有一条边无需根据题目要求去连通当前的两个点。

2.多出来的边用于连通两个内含元素个数最多的集合，使得当前度最大。

```c
#include<stdio.h>
#include<algorithm>
using namespace std;
int n,m;
int a[1010];
int b[1010];
int c[1010];
int find(int x)
{
    if(a[x]!=x) a[x]=find(a[x]);
    return a[x];
}//并查集

int main()
{
    scanf("%d %d\n",&n,&m);
    for(int i=1;i<=n;i++)
    {
        a[i]=i;
        b[i]=1;
    }
    int s=0;
    for(int i=0;i<m;i++)
    {
        int l,r;
        scanf("%d %d\n",&l,&r);
        int x=find(l),y=find(r);
        if(x!=y)//查找当前两个点是否连通
        {
            a[x]=y;
            b[y]+=b[x];
        }
        else s++;
        int sz=0;
        for(int j=1;j<=n;j++)
        {
            if(find(j)==j)
            {
                c[sz++]=b[j];
            }
        }
        sort(c,c+sz,greater<int>());
        int sum=0;
        for(int j=0;j<sz&&j<s+1;j++) sum+=c[j];//连接集合
        printf("%d\n",sum-1);
    }
}
```

# 第93场周赛

## 4868.数字替换

dfs

```c
#include<stdio.h>
int min1=10000;
int n;
void suan(long long x,int m)
{
    int b[10]={0}; //当前x中包含的数
    int s=0; //当前x的位数
    long long x1=x;
    for(int i=0;;i++)
    {
        if(x1==0) break;
        s++;
        b[x1%10]++;
        x1=x1/10;
    }
    if(m+n-s>=min1) return; //每次乘完一位数后，s最多加1
    if(s==n)
    {
        min1=m;
        return;
    }
    for(int i=9;i>=2;i--)//向下传递的数一定要大于x
    {
        if(b[i]!=0) suan(x*i,m+1);//遍历每个x中存在的数
    }
}

int main()
{
    long long x;
    scanf("%d %lld\n",&n,&x);
    suan(x,0);
    if(min1==10000) printf("-1\n");
    else printf("%d\n",min1);
    return 0;
}
```

## 4869.异或值

1.建立一个由0和1组成的二叉树，由输入的数构建的

2.深度优先搜索找到第w层的左子树的最大值和右子树的最大值，返回左右子树的最小值。

```c
#include<stdio.h>
#include<algorithm>
#include<iostream>
using namespace std;
int a[3000010][2]; //100010*30
int sz=0;

void insert(int x) //建立一个深度为30的，只由0，1，构成的二叉树
{
    int p=0;
    for(int i=29;i>=0;i--)
    {
        int w=x>>i&1;
        if(!a[p][w]) a[p][w]=++sz; //建立一个新节点
        p=a[p][w];
    }
}

int dfs(int p,int w)
{
    if(w==-1) return 0;
    int s[2];
    for(int i=0;i<2;i++)
    {
        if(a[p][i]) s[i]=dfs(a[p][i],w-1); //当前节点存在，去找它的下一个节点，位置减一。是s[i]存的是一个十进制数，是当前节点后面异或的最小值
        else s[i]=-1; //不存在的情况，负为-1，为后续跳过。
    }
    int r=2e9;
    for(int i=0;i<2;i++)
    {
        int t=0;
        for(int j=0;j<2;j++) 
            if(s[j]!=-1) 
                t=max(t,s[j]+((i^j)<<w)); //当前节点异或的值加上是s[j]，表示当前二进制有w位的可异或得到的最大的数。
        r=min(r,t); //在这些可异或得到的最大得到的数中取最小值。
    }
    return r;
}

int main()
{
    int n=0;
    scanf("%d\n",&n);
    int x=0;
    for(int i=0;i<n;i++) scanf("%d ",&x),insert(x);
    printf("%d\n",dfs(0,29)); 
    return 0;
}
```

# 第94场周赛

## 4871.最早时刻

#### Dijikstra

1.建图

2.用dijikstra找到1到各点的最短距离

3.判断当前时间点是否能经过当前节点，不能时间加1，加到能通行为止。

```c
#include<stdio.h>
#include <iostream>
#include <cstring>
#include <algorithm>
#include <queue>
using namespace std;
const int N=100010, M=2*N,INF = 0x3f3f3f3f;//INF表示无穷大
int h[N],e[M],w[M],ne[M];//作用：建立一个图
int idx=0;
int dist[N];//1到每个点的最短距离
int st[N];
vector<int> delay[N];//每个点不可通行的时间节点
void add(int a,int b,int c)//建立图
{
    e[idx]=b;
    w[idx]=c;
    ne[idx]=h[a];
    h[a]=idx++;
}
int add_delay(int ver,int distance)//当前时间点是否可以同行该点
{
    for (int t: delay[ver])
        if (t == distance) distance ++ ;//不能，时间+1
        else if (t > distance) break;
    return distance;//返回能通行该点的最小时间点
}
void dijkstra()
{
    priority_queue<tuple<int,int>, vector<tuple<int,int>>, greater<>> q;//小根堆
    q.push({0,1});//t=0时刻从1出发
    memset(dist, 0x3f, sizeof dist);
    dist[1]=0;
    while(q.size())
    {
        auto [x,y]=q.top();
        q.pop();
        if(st[y]==1) continue;//走过跳过
        st[y]=1;
        int distance=add_delay(y,x);//从1走到y点需要的最短时间
        for(int i=h[y];i!=-1;i=ne[i])
        {
            int d=e[i];//d：y直接连接的点
            if(dist[d]>w[i]+distance)//若1到d经过点y所需要的时间更短，则更新1到d的最短距离
            {
                dist[d]=w[i]+distance;
                q.push({dist[d],d});
            }
        }
    }
}
int main()
{
    int n,m;
    scanf("%d %d\n",&n,&m);
    memset(h, -1, sizeof h);
    for(int i=0;i<m;i++)
    {
        int a,b,c;
        scanf("%d %d %d\n",&a,&b,&c);
        add(a,b,c);
        add(b,a,c);
    }
    for(int i=1;i<=n;i++)
    {
        int s=0;
        scanf("%d ",&s);
        delay[i].resize(s);//设置delay[i]的大小，或者说把delay[i]当做一个变量d，将变量d变为一个一维数组。
        for(int j=0;j<s;j++) scanf("%d", &delay[i][j]);
    }
    dijkstra();
    if (dist[n] == INF) printf("-1\n");
    else printf("%d\n",dist[n]);
}
```



## 4871.最短路径

#### Floyd

删除节点转变为添加节点，每次添加检查i到j距离是否因为经过k节点而变得更小，变小则更新i到j的最短距离。

```c
#include<iostream>
#include<algorithm>
using namespace std;
const int N=510;
int a[N][N]; //i到j的距离——>i到j的最短距离
int b[N];
long long d[N];
bool st[N];
int main()
{
    int n=0;
    scanf("%d\n",&n);
    for(int i=1;i<=n;i++)
        for(int j=1;j<=n;j++)
            scanf("%d\n",&a[i][j]);
    for(int i=1;i<=n;i++) scanf("%d\n",&b[i]);
    for(int x=n;x>0;x--)
    {
        int k=b[x];
        st[k]=true;
        for(int i=1;i<=n;i++)
            for(int j=1;j<=n;j++)
                a[i][j]=min(a[i][j],a[i][k]+a[k][j]);//更新为最短距离
        for(int i=1;i<=n;i++)
            for(int j=1;j<=n;j++)
                if(st[i]&&st[j])
                    d[x]+=a[i][j];//加入当前节点的最短路径和
    }
    for(int i=1;i<=n;i++) printf("%lld ",d[i]);
    return 0;
}
```

![](D:\笔记\QQ图片20230314144911.jpg)

# 第95场周赛

## 4875.整数游戏

#### 博弈论

双方都采取最优策略,就是说双方在选调换的a[i]时会选最小的那个，以保证a[0]尽可能快的到达0来结束游戏；先把0换到a[0]的人获胜，而就是说当轮到某个人时（动手前）a[0]=0了，这场游戏就结束了，这个人就输了。

必败态：当前a[0]<=a[i]时，必败。

必胜态：当前a[0]>a[i]时，必胜。

设当前a[0],a[1],......,a[n]中a[0]最小，通过与a[k]变换,数组变为a[k],a[1],......,a[0]-1,a[n],必有a[k]>a[0]-1,说明a[0]无论与哪个值变换，都会从必败转为必胜

```c
#include<iostream>
#include<algorithm>
using namespace std;
int a[100010];
int main()
{
    int n=0;
    cin>>n;
    for(int x=0;x<n;x++)
    {
        int m=0;
        cin>>m;
        for(int i=0;i<m;i++) scanf("%d ",&a[i]);
        int sum=a[0];
        sort(a,a+m);
        if(sum==a[0]) printf("Bob\n");
        else printf("Alice\n");
    }
}
```

# 第96场周赛

## 4878.维护数组

树状数组

1.利用树状数组存储某个区间的和：C₈₈=a[81~88],因为lowbit(88)=8,所以存储了(往前推)8个数的和。

2.查找

```c
#include<iostream>
#include<algorithm>
using namespace std;
const int N=200010;
int n,k,a,b,q;
int d[N];
int l[N],r[N];//树状数组
int lowbit(int x)//划分区间的作用
{
    return x & -x;//返回二进制的最低位，3:11 -> 01:1   6:110 -> 010:2
}
void add(int tr[],int x,int v)
{
    for(int i=x;i<=n;i+=lowbit(i))//从叶子节点出发，找到包含该节点的所有区间
    {
        tr[i]+=v;//在包含这个节点的区间内加入这个节点的变换的值，因为这个区间就是i-lowbit(i)+1~i区间的前缀和
    }
}
int query(int tr[],int x)//给定右端点，寻找1~x的和
{
    int sum=0;
    for(int i=x;i;i-=lowbit(i))//找到1~x包含的每个区间，加上这些区间的和就是1~x的和,例如l[6](5~6的前缀和)=a[5]+a[6],l[4](1~4的前缀和)=a[1]+~+a[4];当x=6时，就会找到l[6]和l[4]两个区间，就可以求出1~6的和，就不需要进行O(n)次寻找
    {
        sum+=tr[i];
    }
    return sum;
}
int main()
{
    cin>>n>>k>>a>>b>>q;
    for(int i=0;i<q;i++)
    {
        int t=0;
        cin>>t;
        if(t==1)
        {
            int x,y;
            cin>>x>>y;
            add(l,x,min(d[x]+y,b)-min(d[x],b));
            add(r,x,min(d[x]+y,a)-min(d[x],a));
            d[x]+=y;
        }
        if(t==2)
        {
            int p;
            cin>>p;
            printf("%d\n",query(l,p-1)+query(r,n)-query(r,p+k-1));//jiu'shi前缀和求值得样子
        }
    }
}
```

# 第97场周赛

## 4946.叶子节点

搜索就行了，重点在题目理解，无向边！！！生气

```c
#include<iostream>
#include<algorithm>
#include<queue>
#include<cstring>
using namespace std;
const int N=1000010,M=2000010;
int h[M],e[M],ne[M];
int sz=0;
int a[N];
int d[N];
bool st[N];
int n,k;
int zui[N];//记录叶子节点
int zs=0;
void add(int x,int y)
{
    ne[sz]=h[x]; e[sz]=y; h[x]=sz++;
}
void bfs(int x)
{
    d[1]=a[1];
    queue<int> q;
    q.push(x);
    st[x]=true;
    while(q.size())
    {
        int t=q.front();
        q.pop();
        int flag=0;
        for(int i=h[t];i!=-1;i=ne[i])
        {
            int j=e[i];
            if(!st[j])
            {
                flag=1;
                q.push(j);
                st[j]=true;
                if(d[t]>k) d[j]=d[t];
                else
                {
                    if(a[j]==1) d[j]=d[t]+1;
                    else d[j]=0;
                }
            }
        }
        if(flag==0) zui[zs++]=t;
    }
}
int main()
{
    scanf("%d %d\n",&n,&k);
    for(int i=1;i<=n;i++) scanf("%d ",&a[i]);
    memset(h, -1, sizeof h);
    for(int i=0;i<n-1;i++)
    {
        int x,y;
        scanf("%d %d\n",&x,&y);
        add(x,y);
        add(y,x);
    }
    bfs(1);
    int sum=0;
    for(int i=0;i<zs;i++) if(d[zui[i]]<=k) sum++;//该叶子节点是否满足条件
    printf("%d\n",sum);
}
```

