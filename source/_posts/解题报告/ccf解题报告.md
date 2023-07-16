---
title: ccf解题报告
date: 2020-02-05 11:01:30
categories: 解题报告
mathjax: true
tags: 
- ccf
- c/c++
---
为了考研复试，先准备一下ccf吧。我会把一些做着不太顺手的题整理到这里，至于那些比较简单的题就不往这写了。
<!--more-->

201312-4 有趣的数
======
[题目描述](http://118.190.20.162/view.page?gpid=T2) 
这个题对现在的我来说难度有点大，没做出来。当时做的时候能想到用动态规划，但是并不知道该怎么用。根据网上的一些资料才做了出来。
**思路分析**：本题用暴力搜索肯定是没有办法做的，这种情况下想到用**动态规划**。本题使用动态规划最主要的一点就是要找齐所有的**状态**。

针对本题，数（字符串）可以分为以下6种状态：

1. 只包含数字2，记为$S_1$
2. 只包含数字2和0（0开始的数0个，以此数为前缀的数均不是以0开始），记为$S_2$
3. 只包含数字2和3，且满足所有的2都出现在所有的3之前，记为$S_3$
4. 只包含数字2、0和1，且满足所有的0都出现在所有的1之前，记为$S_4$
5. 只包含数字2、0和3，且满足所有的2都出现在所有的3之前，记为$S_5$
6. 包含0、1、2和3，且满足所有的0都出现在所有的1之前，所有的2都出现在所有的3之前，记为$S_6$

考虑递推式（状态转移方程）：

1. 对于$S_1$，考虑其长度n，定义$f(n,S_1)$为长度为n的$S_1$的数量，则$f(n,S_1) = 1$。也就是说长度为n的只包含2的数只有1个。
2. 对于$S_2$，考虑其长度n，定义$f(n,S_2)$为长度为n的$S_2$的数量。
当n = 1时，$f(n,S_2) = 0$；当n > 1时，$f(n,S_2) = 2f(n-1,S_2) + f(n-1,S_1)$。
这是因为，长度为n的$S_2$数可以是由长度为n-1的$S_2$数加上2或0构成，例如20为长度为2的$S_2$数，那么202和200为长度为3的$S_2$数；
另外，长度为n的$S_2$数可以是由长度为n-1的$S_1$数加上0构成，例如22为长度为2的$S_1$数，那么220为长度为3的$S_2$数。后面递推式的分析方法与此类似。
3. 对于$S_3$，考虑其长度n，定义$f(n,S_3)$为长度为n的$S_3$的数量。
当n = 1时，$f(n,S_3) = 0$；当n > 1时，$f(n,S_3) = f(n-1,S_3) + f(n-1,S_1)$。
4. 对于$S_4$，考虑其长度n，定义$f(n,S_4)$为长度为n的$S_4$的数量。
当n = 1时，$f(n,S_4) = 0$；当n > 1时，$f(n,S_4) = 2f(n-1,S_4) + f(n-1,S_2)$。
5. 对于$S_5$，考虑其长度n，定义$f(n,S_5)$为长度为n的$S_5$的数量。
当n = 1时，$f(n,S_5) = 0$；当n > 1时，$f(n,S_5) = 2f(n-1,S_5) + f(n-1,S_3) + f(n-1,S_2)$。
6. 对于$S_6$，考虑其长度n，定义$f(n,S_6)$为长度为n的$S_6$的数量。
当n = 1时，$f(n,S_6) = 0$；当n > 1时，$f(n,S_6) = 2f(n-1,S_6) + f(n-1,S_5) + f(n-1,S_4)$。

代码：
```cpp
#include <iostream>
#include <cstring>

using namespace std;

const long long MOD = 1000000007;
const int MAXN = 1005;
long long status[MAXN][6];  //由于数值很大，定义为long long类型

int main()
{
    int n;
    cin >> n;
    memset(status,0,sizeof(status));    //初始化为0

    status[1][0] = 1;   // 赋初始值，长度为1的S1有1种。这里的长度不是从零开始的，而是从1开始的，这样比较直观
    for(int i = 2;i <= n;i++) { 
        status[i][0] = 1;
        status[i][1] = (2 * status[i-1][1] + status[i-1][0]) % MOD; //别忘了每次运算后取余数，不要放在最后，会溢出
        status[i][2] = (status[i-1][2] + status[i-1][0]) % MOD;
        status[i][3] = (2 * status[i-1][3] + status[i-1][1]) % MOD;
        status[i][4] = (2 * status[i-1][4] + status[i-1][2] + status[i-1][1]) % MOD;
        status[i][5] = (2 * status[i-1][5] + status[i-1][4] + status[i-1][3]) % MOD;
    }

    cout << status[n][5];   //输出结果
    return 0;
}

```

201403-3 命令行选项
======
[题目描述](http://118.190.20.162/view.page?gpid=T8) 
这个题确实是有点恶心，难倒不是太难，但是比较麻烦。不过做完这个题后确实学到了一些东西。下面说一下我的思路。

**思路**：

+ 对得到的格式字符串进行分析，得到各合法选项以及该选项是否有参数，存入到map中。
+ 依次对输入命令的各选项进行分析，进行模拟。

**注意点**：

+ 读入一行字符串要使用**getline()函数**
+ 在读入n后，要调用**getchar()**将其后的回车吃掉，否则会被getline函数读取为空字符串。
+ 使用**string流**提取命令中的选项比较方便

代码如下：
```cpp
#include <iostream>
#include <sstream>
#include <map>
#include <cstdio>
#include <utility>

using namespace std;

int main()
{
    string format_string;   // 格式字符串
    int n;
    cin >> format_string >> n;
    getchar();  // 吃掉n后的回车
    string commands[21];    // 存储命令的数组
    for(int i = 0; i< n;i++) {
        getline(cin,commands[i]);
    }

    map<string,bool> have_parameter;    // 记录各选项是否有参数
    for(int i = 0;i < format_string.length();i++) {
        if(format_string[i] != ':') {
            have_parameter.insert(pair<string,bool>(format_string.substr(i,1),false));
        }
        else {
            have_parameter[format_string.substr(i-1,1)] = true;
        }
    }

    map<string,string> parameter;   // 存储各选项的参数
    for(int i = 0;i < n;i++) {
        cout << "Case " << i + 1 << ":";
        istringstream command(commands[i]); // string流
        string s;
        command >> s;
        while(command >> s) {
            if(s[0] == '-') {   // 合法选项以“-”打头
                s.erase(0,1);   // 删除“-”
                if(s.length() == 1 && have_parameter.count(s)) {    // 检查选项是否合法
                    parameter.insert(pair<string,string>(s,""));
                    map<string,bool>::iterator it = have_parameter.find(s);
                    if(it != have_parameter.end() && it->second) {  // 若选项后有参数
                        string temp;
                        if(command >> temp) parameter[s] = " " + temp;  // 这里注意要加一个if语句，因为可加参数的选项其后不一定有参数
                        else break;
                    }
                }
                else break;
            }
            else break;
        }

        map<string,string>::iterator it = parameter.begin();    // map是有序的，默认按键升序，直接输出即可
        while(it != parameter.end()) {
            cout << " -" + it->first + it->second;
            it++;
        }
        cout << endl;
        parameter.clear();  // 清空map
    }
    return 0;
}
```
上面代码中第44行的if语句一定要加，否则提交上去只会得90。这是因为可带参数的选项其后不一定有参数。我第一次就是想当然了，结果出错了。
我总感觉这个题有点问题。看下面这个测试用例。
```cpp
albw:x 
2
ls -w -w 
ls -w
```
运行结果：
```py
Case 1: -w -w
Case 2: -w
```
根据常识来说，第二个“-w”应该是被当成一个选项来处理的，但是这个题是将其当成参数来处理的。不过这玩意毕竟是人家说了算，题里面咋说，咱就咋做吧。

201403-4 无线网络
======
[题目描述](http://118.190.20.162/view.page?gpid=T7) 
这个题我也没做出来，主要是图的一些算法都忘干净了，就连bfs求无权图的最短路问题都想不起来了，还得加强复习啊。

**思路分析**：

+ 路由器构成了一个图，如果来两个路由器的距离小于r则存在边
+ 求解的是最短路径。图所有边的权重均为1，可用**bfs**来解决。使用广度优先搜索的层次遍历，可以方便地得到第1个路由器与第2个路由器之间的距离
+ 搜索的时候要记录路径上所经过的增设路由器的个数，确保其不会超过k

**注意点**：

+ 坐标以及r均应定义为**long long**类型的变量，否则会在运算时溢出。用int提交只能得80分（血的教训）
+ 当增设路由器为k时仍应继续搜索，只有当大于k时才停止搜索此路径
+ 栈和队列的**pop()函数**返回值均为**void**，千万不要以为该函数将元素删除后再返回

代码：
```cpp
#include <iostream>
#include <queue>

using namespace std;

const int MAXN = 205;

bool visited[MAXN] = {};    // 定义visited数组并将其初始化为false，也可用memset进行初始化
int n,m,k;
long long r;    

typedef struct router {
    long long x,y;
    int counter,step;   // counter为增设路由器个数，step为步数
    router() {counter = step = 0;}  // 默认构造函数，初始化counter和step为0
}Router;
Router routers[MAXN];   // 存放所有路由器信息的数组，包括增设路由器

bool has_edge(Router rou1,Router rou2);
int bfs(int st,int ed);

int main()
{
    cin >> n >> m >> k >> r;
    for(int i = 0;i < n + m;i++) {
        cin >> routers[i].x >> routers[i].y;
    }

    int step = bfs(0,1);
    cout << step;
    return 0;
}

bool has_edge(Router rou1,Router rou2) {    // 判断两个路由器之间是否有边连通
    return (rou1.x - rou2.x)*(rou1.x - rou2.x) + (rou1.y - rou2.y)*(rou1.y - rou2.y) <= r*r;
}

int bfs(int st,int ed) {    
    queue<Router> que;
    que.push(routers[st]);
    visited[st] = true;
    while(!que.empty()) {
        Router cur_router = que.front();    // front获得队首元素，但不删除
        que.pop();  // pop函数删除队首元素，返回void
        if(cur_router.x == routers[ed].x && cur_router.y == routers[ed].y)  // 若找到终点
            return cur_router.step - 1; // 由于题目要的是中间经过的路由器个数，所以这里要减去1
        if(cur_router.counter > k) continue;    // 如果增设路由器个数超过k，则放弃搜索此条路径。注意这里是大于，而不是大于等于
        for(int i = 0;i < n + m;i++) {
            if(!visited[i] && has_edge(cur_router,routers[i])) {
                routers[i].step = cur_router.step + 1;  // 步数加1
                if(i >= n) routers[i].counter = cur_router.counter + 1; // 若为增设路由器，counter加1
                que.push(routers[i]);
                visited[i] = true;
            }
        }
    }
}

```

201409-4 最优配餐
======
[题目描述](http://118.190.20.162/view.page?gpid=T13) 
有了上面那道题的基础，这道题倒是做出来了，但是超时，就得了10分。
我对于bfs有点形成思维定式了。给定起点和终点，找到二者之间的最短路径，返回路径长度。这道题这么做的话铁定超时。

**正确思路**：

+ 这道题主要是求哪家分店离顾客最近，最近一定是所有分店中最近的，所以考虑从**所有分店同时开始BFS**。
+ 为了“同时”开始BFS，在开始时将所有的分店加入到队列中去就行了。由于BFS的特性，虽然各分店开始的顺序与加入队列的顺序有关，但是每一轮（同一层）的搜索都是同步的，所以虽然不是严格意义上的“同时”，但是这是没问题的。
+ 在所有分店进行BFS的过程中，并没有一个固定的终点，而是**到达一个顾客点就完成这个点的需求**，直到遍历完整张图为止。这是一个非常关键的点，而且这样做是非常快的。

**注意点**：

+ 计算成本时要用long long类型，并且将与计算相关的变量也定义为long long类型，避免在默认类型转换时发生错误
+ 将障碍物对应位置的visited数组置为true可以简化逻辑与代码哦

代码：
```cpp
#include <iostream>
#include <queue>
#include <cstring>

using namespace std;

const int MAXN = 1005;
long long orders[MAXN][MAXN];   //地图上对应点的订单总数
bool visited[MAXN][MAXN];
int n,m,k,d;
long long cost = 0; // 总花费

struct Node {
    int x,y;
    long long step; // 步数
    Node() {step = 0;}
    Node(int x,int y,long long step): x(x),y(y),step(step) {}
};
queue<Node> que;   

struct {
    int x,y;
} dir[4] = {{0,1},{0,-1},{-1,0},{1,0}}; // 四个移动方向

bool judge(int x,int y);
void bfs();

int main()
{
    cin >> n >> m >> k >> d;
    memset(visited,0,sizeof(visited));  // 可用memset初始化bool类型的数组
    memset(orders,0,sizeof(orders));
    int x,y;
    long long order;
    for(int i = 0;i < m;i++) {
        cin >> x >> y;
        que.push(Node(x,y,0));  // 初始将所有的分店都置于队列中
    }
    for(int i = 0;i < k;i++) {
        cin >> x >> y >> order;
        orders[x][y] += order;  // 由于一个位置可能有多位顾客，统计每一位置的订单总数
    }
    for(int i = 0;i < d;i++) {
        cin >> x >> y;
        visited[x][y] = true;   // 将障碍物对应的visited数组置为true
    }

    bfs();  // 对整张图进行一次大的bfs
    cout << cost;
    return 0;
}

bool judge(int x,int y) {   // 移动位置的合法性判断
    if(x > 0 && y > 0 && x <= n && y <= n && !visited[x][y])
        return true;
    else
        return false;
}

void bfs() {
    while(!que.empty()) {
        Node cur_node = que.front();
        que.pop();
        for(int i = 0;i < 4;i++) {
            int x = cur_node.x + dir[i].x;
            int y = cur_node.y + dir[i].y;
            if(judge(x,y)) {
                long long step = cur_node.step + 1;
                que.push(Node(x,y,step));
                visited[x][y] = true;
                if(orders[x][y] != 0) { // 若到达顾客所在位置
                    cost += orders[x][y] * step;    // 对其进行处理，计算花费
                }
            }
        }
    }
}

```

201412-2 Z字形扫描
======
[题目描述](http://118.190.20.162/view.page?gpid=T20) 

解法一：直接模拟
------
**思路**：

+ 题目中有右、下、左下、右上四种走法
+ 根据题目中的走法直接进行模拟

**小技巧**：

+ 定义一个结构体数组来存放每一种走法的坐标变化量

代码：
```cpp
#include <iostream>
#define RIGHT 0     //右
#define DOWN 1      //下
#define LEFTDOWN 2  //左下
#define RIGHTUP 3   //右上

using namespace std;

const int maxn = 505;

struct {    //定义每种走法的坐标变化量
    int x;
    int y;
}coor_change[4] = {{0,1},{1,0},{1,-1},{-1,1}};

int main()
{
    int n,matrix[maxn][maxn];
    cin >> n;
    for(int i = 0;i < n;i++) { 
        for(int j = 0;j < n;j++) {
            cin >> matrix[i][j];
        }
    }

    int x = 0,y = 0;
    int next = RIGHT;   //next为下一步的运动方向，起始向右运动
    while(x < n - 1 || y < n - 1) {     //当x，y位于矩阵的最后一个元素位置时，循环结束
        cout << matrix[x][y] << " ";    //输出当前值
        x += coor_change[next].x;       //进行下一步运动，更新呢坐标
        y += coor_change[next].y;

        if(next == RIGHT && x == 0) {   //向右运动，且在第一行，则下一步为左下
            next = LEFTDOWN;
        }
        else if(next == RIGHT && x == n - 1) {      //向右运动，且在最后一行，则下一步为右上
            next = RIGHTUP;
        }
        else if(next == DOWN && y == 0) {   //向下运动，且在第一列，则下一步为右上
            next = RIGHTUP;
        }
        else if(next == DOWN && y == n - 1) {   //向下运动，且在最后一列，则下一步为左下
            next = LEFTDOWN;
        }
        else if(next == LEFTDOWN && (x == n - 1 || y == 0)) {   //左下运动遇到边界
            if(x < n - 1) {     //优先向下运动
                next = DOWN;
            }
            else {
                next = RIGHT;   //若无法向下运动，则向右运动
            }
        }
        else if(next == RIGHTUP && (x == 0 || y == n - 1)) {    //右上运动遇到边界
            if(y < n - 1) {     //优先向右运动
                next = RIGHT;
            }
            else next = DOWN;   //若无法向右运动，则向下运动
        }
    }
    cout << matrix[x][y] << " ";    //输出最后一个元素
    return 0;
}
```

解法二：找规律
------
**思路**：

+ 把矩阵分为2n-1条斜线，从0开始编号，第i条斜线编号是偶数 ，则左下打印到右上，第i条斜线编号是奇数 ，从右上打印到左下

代码：
```cpp
#include <iostream>

using namespace std;

const int maxn = 505;

int main()
{
    int n,matrix[maxn][maxn];
    cin >> n;
    for(int i = 0;i < n;i++) {
        for(int j = 0;j < n;j++) {
            cin >> matrix[i][j];
        }
    }

    for(int i = 0;i < 2*n-1;i++) {  //共有2n-1条斜线
        int st = i<n ? 0 : (i-n+1); //下面循环起始值的选取，当i<n时为0，i>=n时为i-n+1
        int ed = i<n ? i : (n-1);   //下面循环终止值的选取，当i<n时为i，i>=n时为n-1

        if(i % 2 == 0) {    //当i为偶数，左下到右上
            for(int j = st;j <= ed;j++){        //当i<n时，从左边界开始，当i>=n时，从下边界开始
                cout << matrix[i-j][j] << " ";  //并且x坐标依次减少1,y坐标依次增加1
            }
        }
        else {  //当i为奇数，右上到左下
            for(int j = st;j <= ed;j++) {
                cout << matrix[j][i-j] << " ";  //逆序的话，只需要将x坐标和y坐标互换就可以了
            }                                   //这是由方阵的特性所决定的
        }
    }
    return 0;
}
```
这种做法最主要的难点就在于下标的处理，上面的代码要理解也要花一番功夫。

201412-4 最优灌溉
======
[题目描述](http://118.190.20.162/view.page?gpid=T18) 
这道题其实挺直白的，直接就能看出来用最小生成树做。但是由于之前学习最小生成树的时候并没有学习代码，只学了个思想，所以这道题是我根据算法思想自己写的。由于考研的时候考察prim算法比较多，所以我就用的prim算法。
第一遍的时候由于对prim算法理解的不太到位，直接写错了。prim算法应该是从已有生成树的**所有结点**出发，来找到到其他未加入生成树的结点最短的边，我写的时候是从最新加入的结点出发，很自然就错了。第二遍改正过来后，发现代码超时，得了70分。第三遍优化了一下后，只得了80。
我发现是我太死板了。代码完全是按照书上的定义写的。先是用邻接矩阵表示图的信息，又定义了两个集合。集合S1代表生成树结点集合，初始化为空；集合S2代表未加入生成树的结点集合，初始化为所有结点。起始将结点1加入S1，并从将其从S2中删除。然后遍历邻接矩阵，找到从S1中结点到S2中结点最短的边，将其上的权值累加起来，并且将目标结点从S2中删除，加入到S1中。循环，直到S2为空。
这种做法最大的缺陷在于每次找最短的边时，都要遍历邻接矩阵，很是浪费时间。而且对集合的一些操作也会花费不少时间。
上面的这些你可以都当成是废话，主要是为了说明一下情况，下面正式介绍这个题的比较好的做法。

解法一：prim算法
------
**思路**：

+ 定义一个vis数组，初始化为false，当结点i对应的vis为true时，表示其在生成树的结点集合内，否则不在。这种做法要比建立两个集合，然后再进行插入删除操作省事的多
+ 每当将一个数组元素置为true时，结点数n自减，当其减少为0时，最小生成树的结点集合已包括图中的全部结点
+ 不用邻接矩阵或者邻接表存储图的信息，而是定义一个边的结构体，包含两个结点和权值等信息。然后建立一个边的数组来存储图的信息
+ 将该数组从小到大排序，然后找到两结点一个对应的vis数组一个为正，一个为负的第一条边，该边就是要找的最短边。然后再进行其他操作就可以了

代码：
```cpp
#include <iostream>
#include <algorithm>
#include <cstring>

using namespace std;

const int MAXN = 1005;
const int MAXM = 100005;
bool vis[MAXN]; // 定义访问数组

struct Edge {
    int v1,v2,w;
    Edge() {}
    Edge(int _v1,int _v2,int _w): v1(_v1),v2(_v2),w(_w) {}
};
Edge edges[MAXM];   // 定义边数组

bool cmp(Edge e1,Edge e2);
int prim(int n,int m);

int main()
{
    int n,m;
    cin >> n >> m;
    for(int i = 0;i < m;i++) {
        cin >> edges[i].v1 >> edges[i].v2 >> edges[i].w;
    }

    memset(vis,0,sizeof(vis));  // 将vis数组初始化为false
    sort(edges,edges + m,cmp);  // 将边按照权值大小由小到大排序
    int cost = prim(n,m);
    cout << cost;
    return 0;
}

bool cmp(Edge e1,Edge e2) {
    return e1.w < e2.w;
}

int prim(int n,int m) { // prim算法
    int cost = 0;   // 总花费
    vis[1] = true;
    n--;
    while(n > 0) {  // 当n为0时，循环结束
        int i = 0;
        while(i < m) {
            int v1 = edges[i].v1,v2 = edges[i].v2,w = edges[i].w;
            if(vis[v1] && !vis[v2]) {   // 当一个结点已访问，另一个未访问时，该边就是我们要找的最短边
                cost += w;  // 计算总花费
                vis[v2] = true;
                n--;
                break;
            }
            else if(vis[v2] && !vis[v1]) {  // 注意该图为无向图，也可能由v2到v1
                cost += w;
                vis[v1] = true;
                n--;
                break;
            }
            else i++;
        }
    }
    return cost;    // 返回总花费
}
```
虽然这种做法效果还不错，只运行的234ms，但是总感觉怪怪的。以后遇到最小生成树还是用kruskal比较好，实现起来较简单，而且效率也比较高。

解法二：kruskal算法
------
算法的思想就不多说了，就是先将边由小到大排序，然后从小到大依次考察每条边(u,v)。若结点u和结点v在不同的连通分量中，那么就选择该边，最后就会形成最小生成树。
这里重点要说的是用**并查集（Union Find Set）**来表示连通分量。并查集的精妙之处在于**用树表示集合**。
假设把x的父结点保存在ufs[x]中（如果x没有父结点，则ufs[x]=x），那么当结点u和v的根结点相同时，就说明u和v在同一连通分量中。
查找根结点的程序用递归也很容易写出：
```py
int ufs_find(int v) {
    if(ufs[v] == v) {
        return v;
    }
    else {
        return ufs_find(ufs[v]);
    }
}
```
但是在特殊情况下，这个树可能是长长的一条链，这样的话遍历的效率就会十分低下。其实改进的方法也很简单，因为每棵树表示的只是一个集合，所以树的形态是无关紧要的，在每次查找后并不需要保持树的形态不变。
要解决这个问题，只需要顺便**把遍历过的结点都改成树根的子结点**就可以了。
修改后的代码如下：
```cpp
int ufs_find(int v) {
    if(ufs[v] == v) {
        return v;
    }
    else {
        return ufs[v] = ufs_find(ufs[v]);
    }
}
```
合并连通分量时，只需要**将一个根结点作为另一个根结点的子结点**就可以了。具体看完整代码。

本题完整代码：
```cpp
#include <iostream>
#include <algorithm>
#include <cstring>

using namespace std;

const int MAXN = 1005;
const int MAXM = 100005;

struct Edge {
    int v1,v2,w;
    Edge() {}
    Edge(int _v1,int _v2,int _w): v1(_v1),v2(_v2),w(_w) {}
};
Edge edges[MAXM];   // 边数组
int ufs[MAXN];  // 并查集

bool cmp(Edge e1,Edge e2);
int ufs_find(int v);
int kruskal(int n,int m);

int main()
{
    int n,m;
    cin >> n >> m;
    for(int i = 0;i < m;i++) {
        cin >> edges[i].v1 >> edges[i].v2 >> edges[i].w;
    }
    for(int i = 0;i <= n;i++) {
        ufs[i] = i;     // 初始化并查集，每个结点都是一个连通分量
    }

    sort(edges,edges + m,cmp);      // 将所有边按照权值进行排序
    int cost = kruskal(n,m);
    cout << cost;
    return 0;
}

bool cmp(Edge e1,Edge e2) {
    return e1.w < e2.w;
}

int ufs_find(int v) {       // 查找v所在树的根结点
    if(ufs[v] == v) {
        return v;
    }
    else {
        return ufs[v] = ufs_find(ufs[v]);       // 在查找的同时，将遍历过的结点的父结点置为根结点
    }
}

int kruskal(int n,int m) {
    int cost = 0;
    for(int i = 0;i < m;i++) {
        // 查找v1和v2所在树的根结点x，y
        int x = ufs_find(edges[i].v1),y = ufs_find(edges[i].v2);    
        if(x != y) {        // 若根结点不同，则v1和v2属于不同的连通分量
            cost += edges[i].w;     // 计算总花费
            ufs[x] = ufs[y];        // 合并连通分量，将x作为y的子结点
            n--;
            if(n <= 1) break;   // 若找到了n-1条边，直接结束循环
        }
    }
    return cost;
}

```