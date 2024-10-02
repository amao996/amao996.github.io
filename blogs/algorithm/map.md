# unordered

## unordered_set

1.头文件

```c++
#include<unordered_set>
```

2.特征：

1）直接存储数据的值，不是键值对的形式

2）容器内部存储的元素值互不相等且不能修改

3）不会对内部存储数据进行排序

4）**不能有重复元素**

5）迭代器至少是向前迭代器

3.常用函数

```c++
empty();	//判断是否为空
find();		//找到返回迭代器，失败返回end()
count(2);	//返回2出现的次数
clear();	//清空容器
swap();		//交换两个容器中的数据
insert();	//传入一个参数（待插入元素）	两个参数（迭代器（指定元素的被插入位置）+待插入元素）
emplace();		//插入元素（转移构造），效率比insert高
erase();		//删除元素
```

## unordered_map

1.头文件

```c++
#include<unordered_map>
```

2.特征

1）允许通过key值快速索引到与其对应的value

2）键值通常用于唯一标识元素，而映射值是一个对象，其内容与此键关联，键和映射值的类型可能不同

3）允许使用key作为参数直接访问value

4）迭代器至少是向前迭代器

3.常用函数

```c++
unordered_map<int, double> um1; //构造一个key为int类型，value为double类型的空容器
insert();		//um.insert(pair<int, string>(1, "one"));um.insert(make_pair(4, "four"));
erase();
find();
size();
empty();
clear();
swap();
count();
```

### map

与unordered_map的不同点：

1.map基于红黑树，元素**有序**存储，unordered_map基于散列表，元素**无序**存储

## 优先队列

1.头文件：

```c++
#include<queue>
```

2.常用大顶堆和小顶堆

```c++
priority_queue<int,vector<int>,greater<int>> pq;//小顶堆
//不写后两个参数，容器默认为vector，大顶堆
priority_queue<int,vector<int>,less<int>> pq;	//大顶堆

# 自定义比较器
struct Comparator{
    bool operator()(ListNode* a, ListNode *b){
        return a->val > b->val;
    }
};
```

3.常用函数

```C++
push(element)	//插入到队尾
pop()
top()
empty()
size()    
emplace(element)	//原地构造一个元素并插入队列
```

4.pair数据类型比较

先比较第一个元素然后比较第二个元素

eg：

```c++
#include <iostream>
#include <queue>
#include <vector>
using namespace std;
int main() 
{
    priority_queue<pair<int, int> > a;	--默认是大顶堆
    pair<int, int> b(1, 2);
    pair<int, int> c(1, 3);
    pair<int, int> d(2, 5);
    a.push(d);
    a.push(c);
    a.push(b);
    while (!a.empty()) 
    {
        cout << a.top().first << ' ' << a.top().second << '\n';
        a.pop();
    }
}
```

## multiset

1.头文件

```C++
#include<set>
```

2.作用：它可以看成一个序列，插入或删除一个数都能在O(logn)的时间内完成，而且它能时刻保证序列中的数是有序的，而且序列中可以存在重复的数。

3.常用函数总结

```c++
//构造，拷贝，析构
set c;		//产生一个空的set/multiset,不含任何元素	eg: multiset<int> s;
set c(op);	//以op为排序准则，产生一个空的set/multiset
set c1(c2)	//产生某个set/multiset的副本，所有元素都被拷贝
set c(beg,end)	//以区间[beg,end)内的所有元素产生一个set/multiset
set c(beg,end, op)	//以op为排序准则，区间[beg,end)内的元素产生一个set/multiset
c.~set()		//销毁所有元素，释放内存
set<Elem>		//产生一个set，以(operator <)为排序准则
set<Elem,0p>	//产生一个set，以op为排序准则
c.size()
c.empty()
c.max_size()
//特殊的搜寻函数
count (elem)	//返回元素值为elem的个数
find(elem)		//返回元素值为elem的第一个元素，如果没有返回end()
lower_bound(elem)	//返回元素值为elem的第一个可安插位置，也就是元素值 >= elem的第一个元素位置
upper_bound (elem)	//返回元素值为elem的最后一个可安插位置，也就是元素值 > elem 的第一个元素位置
equal_range (elem)	//返回elem可安插的第一个位置和最后一个位置，也就是元素值==elem的区间
//迭代器相关函数
c.begin()	//返回一个随机存取迭代器，指向第一个元素
c.end()		//返回一个随机存取迭代器，指向最后一个元素的下一个位置
c.rbegin()	//返回一个逆向迭代器，指向逆向迭代的第一个元素
c.rend()	//返回一个逆向迭代器，指向逆向迭代的最后一个元素的下一个位置
//插入和删除
c.insert(elem)	//插入一个elem副本，返回新元素位置，无论插入成功与否。
c.insert(pos, elem)	//安插一个elem元素副本，返回新元素位置，pos为收索起点，提升插入速度。
c.insert(beg,end)	//将区间[beg,end)所有的元素安插到c，无返回值。
c.erase(elem)	//删除与elem相等的所有元素，返回被移除的元素个数。
c.erase(pos)	//移除迭代器pos所指位置元素，无返回值。
c.erase(beg,end)	//移除区间[beg,end)所有元素，无返回值。
c.clear()		//移除所有元素，将容器清空
```

4.注意：multiset中元素不可修改，元素可以重复

### Prim

```c++
class Prim{
private:
	//核心数据结构，存储[横切边]的优先级队列
	//三元组{from,to,weight}表示一条边
	priority_queue<vector<int>,vector<vector<int>>, greater<vector<int>>> pq;
	//类似visited数组的作用，记录哪些节点已经成为最小生成树的一部分
	vector<bool> inMST;
	int weightSum = 0;
	vector<vector<int>>* graph;
public:
	Prim(vector<vector<int>>* graph){
		this->graph = graph;
		int n  =graph.size();
		this->inMST.resize(n);

		//随便从一个点开始切分都可以，这里从节点0开始
		inMST[0] = true;
		cut(0);
		while(!pq.empty()){
			vector<int> edge = pq.top();
			pq.pop();
			int to = edge[1];
			int weight = edge[2];
			if(inMST[to]){
				continue;
			}
			weightSum += weight;
			inMST[to] = true;
			cut(to);
		}
	}
	void cut(int s){
		for(vector<int>& edge : (*graph)[s]){
			int to = edge[1];
			if(inMST[to]){
				//相邻节点to已经在最小生成树中，skip
				//否则这条边会产生环
				continue;
			}
			pq.push(edge);
		}
	}
	int weightSum(){
		return weightSum;
	}
	bool allConnected(){
		for(bool connected : inMST){
			if(!connected)
				return false;
		}
		return true;
	}
};
```

### 并查集Union-Find

```c++
class UF{
private:
	int count;	//连通分量数目
    int *parent;	//存储每个节点的父节点
public:
    UF(int n){
        this->count = n;
        parent = new int[n];
        for(int i = 0; i < n; i++){
            parent[i] = i;
        }
    }
    void union_(int p, int q){
        int rootP = find(p);
        int rootQ = find(q);
        if(rootP == rootQ)
            return;
        parent[rootQ] = rootP;	//两个连通分量合并成一个
        count--;
    }
    bool connected(int p, int q){
        int rootP = find(p);
        int rootQ = find(Q);
        return rootP == rootQ;
    }
    //状态压缩
    int find(int x){
        if(parent[x] != x){
            parent[x] = find(parent[x]);
        }
        return parent[x];
    }
    int count_(){
        return count;
    }
}
```
