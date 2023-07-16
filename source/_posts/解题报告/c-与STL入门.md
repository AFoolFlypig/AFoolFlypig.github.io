---
title: c++与STL入门
date: 2018-02-18 11:43:06
categories: 解题报告
tags:
- c/c++
- 竞赛
---
这几天过年过的真是，没心思学习。从今天开始收收心，接着学习。感觉我这学习进度够慢的，但是没办法，快了的话根本学不扎实。
<!--more-->

1. 大理石在哪（Where is the Marble?,Uva 10474）
------
这个题蛮简单的，就是排序之后再检索。通过这个题，我学会了一个新的函数：lower_bound()。我这个题多次wa，最后我发现只是输出的时候少打了一个冒号，还是得细心啊。
```cpp
#include<cstdio>
#include<algorithm>

using namespace std;

const int maxn = 10005;

int main(){
    int N,Q,n = 0;
    int number[maxn],q;
    while(scanf("%d%d",&N,&Q) == 2 && N){
        printf("CASE# %d:\n",++n);
        for(int i = 0;i < N;i++){
            scanf("%d",&number[i]);
        }
        sort(number,number + N);
        while(Q--){
            scanf("%d",&q);
            int pos = lower_bound(number,number + N,q) - number;
            if(number[pos] == q)
                printf("%d found at %d\n",q,pos + 1);
            else
                printf("%d not found\n",q);
        }
    }
    return 0;
}
```

2. 丑数（Ugly Number，UVa136）
------
解题思路：假设x是丑数，那么2x，3x以及5x都是丑数。1是丑数，则初始x值为1。
利用集合set来存储丑数，去重复且从小到大自动排序。这种方法的缺点可能就是效率比较低。书上的方法其实也蛮好的，采用优先队列，效率应该比直接用集合高。
```cpp
#include<cstdio>
#include<set>
#include<algorithm>
#define N 1500

using namespace std;

typedef long long ll;

int main(){
    set<ll>ugly;
    ugly.insert(1);
    set<ll>::iterator it = ugly.begin();
    for(int i = 0;i < N - 1;i++){
        ugly.insert(2 * (*it));
        ugly.insert(3 * (*it));
        ugly.insert(5 * (*it));
        it++;
    }
    printf("The 1500'th ugly number is %lld.\n",*it);
    return 0;
}
```

3. 卡片游戏（Throwing cards away I ，uva10935）
------
解题思路：这个题蛮简单的，直接用队列模拟就可以。
```cpp
#include<cstdio>
#include<algorithm>
#include<queue>

using namespace std;

int main(){
    int n;
    while(scanf("%d",&n) == 1 && n){
        printf("Discarded cards:");
        queue<int> q;
        for(int i = 1;i <= n;i++)
            q.push(i);
        while(q.size() - 1){
            printf(" %d",q.front());
            if(q.size() > 2)
                printf(",");
            q.pop();
            int t = q.front();
            q.push(t);
            q.pop();
        }
        printf("\nRemaining card: %d\n",q.front());
    }
    return 0;
}
```

4. 复合词（Compound Words，uva10391）
------
解题思路：这个题有两种解题思路，先将所有的词放在集合里。第一种思路是将所有的词组合，查看集合中是否有该复合词;第二种是将所有的词拆分，查看集合中是否有这两个拆分词，如果有，则该词就是复合词。

第一种思路超时，第二种思路较好。另外，还有一个注意点，当判断出该词是复合词之后，应当退出循环，否则会wa。
```cpp
#include<cstdio>
#include<iostream>
#include<vector>
#include<set>

using namespace std;

vector<string> v;
set<string> word;

int main(){
    string s;
    while(cin >> s){
        v.push_back(s);
        word.insert(s);
    }
    string s1,s2;
    for(int i = 0;i < v.size();i++){
        for(int j = 1;j < v[i].length();j++){
            s1 = v[i].substr(0,j);
            s2 = v[i].substr(j,v[i].length() - j);
            if(word.count(s1) && word.count(s2)){
                cout << v[i] << endl;
                break;
            }
        }
    }
    return 0;
}
```

5. 打印队列（Printer Queue,uva12100）
------
解题思路：这个题能看出来就是队列模拟，但是怎么模拟，我刚开始一点思路没有。模拟的话有两个问题需要解决，一个是如何判断当前队列最前方元素的优先级;另一个是如何判断当前出队元素是否是指定的元素，若是指定元素，结束循环，打印结果。

我用了pair型的队列jobs来存储元素，其中first是元素起始位置，second为元素优先级。优先队列priority存储的是各元素的优先级。将jobs队首元素优先级与priority队首优先级比较，来决定是否出队，然后判断出队元素是否是指定元素。
```cpp
#include<cstdio>
#include<queue>

using namespace std;

int main(){
    int t;
    scanf("%d",&t);
    while(t--){
        int n,pos;
        scanf("%d%d",&n,&pos);
        queue<pair<int,int> > jobs;
        priority_queue<int> priority;
        for(int i = 0;i < n;i++){
            int j;
            scanf("%d",&j);
            jobs.push(make_pair(i,j));
            priority.push(j);
        }
        int times = 0;
        while(1){
            int p = priority.top();
            pair<int,int> job = jobs.front();
            if(job.second == p){
                priority.pop();
                jobs.pop();
                times++;
                if(pos == job.first) break;
            }
            else{
                jobs.pop();
                jobs.push(job);
            }
        }
        printf("%d\n",times);
    }
    return 0;
}
```