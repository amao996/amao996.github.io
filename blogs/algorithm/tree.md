# 二叉树

## 做题思路

**1、是否可以通过遍历一遍二叉树得到答案**？如果可以，用一个 `traverse` 函数配合外部变量来实现。

**2、是否可以定义一个递归函数，通过子问题（子树）的答案推导出原问题的答案**？如果可以，写出这个递归函数的定义，并充分利用这个函数的返回值。

**3、无论使用哪一种思维模式，你都要明白二叉树的每一个节点需要做什么，需要在什么时候（前中后序）做**。



**动归/DFS/回溯算法都可以看做二叉树问题的扩展，只是它们的关注点不同**：

- **动态规划算法属于分解问题的思路，它的关注点在整棵「子树」**。
- **回溯算法属于遍历的思路，它的关注点在节点间的「树枝」**。
- **DFS 算法属于遍历的思路，它的关注点在单个「节点」**。

## 二叉树的遍历

1.前序遍历

​	[遍历]：

```c++
vector<int> res;
//前序遍历结果
vector<int> preorderTraverse(TreeNode* root){
    traverse(root);
    return res;
}
void traverse(TreeNode* root){
    if(root == nullptr){
        return;
    }
    //前序位置
    res.push_back(root->val);
    traverse(root->left);
    traverse(root->right);
}
```

<br>[分解]：

```c++
vector<int>preOrderTraverse(TreeNode* root){
    vector<int> res;
    if(root == nullptr){
        return res;
    }
    //前序遍历结果，root->val在第一个
    res.push_back(root->val);
    //利用函数定义，后面接着左子树的前序遍历结果
    vector<int> leftRes = preOrderTraverse(root->left);
    res.insert(res.end(),leftRes.begin(),leftRes.end());
    //利用函数定义，最后接着右子树的前序遍历结果
    vector<int> rightRes = preOrderTraverse(root->right);
    res.insert(res.end(),rightRes.begin(),rightRes.end());
    return res;
}
```

<br>2.层序遍历

方法1：

```c++
void levelTraverse(TreeNode* root){
    if(root == nullptr){
        return;
    }
    queue<TreeNode*> q;
    q.push(root);
    //从上到下遍历每一层
   	while(!q.empty()){
        int sz = q.size();
        //从左到右遍历每一层的节点
        for(int i = 0; i < sz; i++){
            TreeNode* cur = q.front();
            q.pop();
            //将下一层节点放入队列
            if(cur->left != nullptr){
                q.push(cur->left);
            }
            if(cur->right != nullptr){
                q.push(cur->right);
            }
        }
    }
}
```

<br>方法2：

```c++
vector<vector<int>> levelTraverse(TreeNode* root){
    if(root == nullptr){
        return res;
    }
    //root视为第0层
    traverse(root,0);
    return res;
}
void traverse(TreeNode* root, int depth){
    if(root == nullptr){
        return;
    }
    if(res.size() <= depth){
        //第一次进入depth层
        res.push_back(vector<int>());
    }
    res[depth].push_back(root->val);
    traverse(root->left,depth+1);
    traverse(root->right,depth+1);
}
```

<br>方法3：

```c++
vector<vector<int>> res;
vector<vector<int>> levelTraverse(TreeNode* root){
    if(root == nullptr){
        return res;
    }
    //应用队列实现层序遍历
    list<TreeNode*> nodes;
    nodes.push_back(root);
    traverse(nodes);
    return res;
}
//递归函数实现层序遍历，利用双向队列实现BFS
void traverse(list<TreeNode*> curLevelNodes){
    //判断传入队列是否为空
    if(curLevelNodes.empty()){
        return;
    }
    vector<int> nodeValues;	//储存当前队列节点值
    //下一层的节点列表
    list<TreeNode*> nextLevelNodes;
    //从双向队列中取节点
    for(auto& node : curLevelNodes){
        nodeValues.push_back(node->val);
        //如果有左儿子，右儿子都插入队列
        if(node->left != nullptr){
            nextLevelNodes.push_back(node->left);
        }
        if(node->right != nullptr){
            nextLevelNodes.push_back(node->right);
        }
    }
    //将当前层的节点值添加到结果中
    res.push_back(nodeValues);
    //继续遍历下一层
    traverse(nextLevelNodes);
}
```

## <br>BFS

```c++
int BFS(Node start, Node target) {
    queue<Node> q; 
    set<Node> visited;
    
    q.push(start); 
    visited.insert(start);

    while (!q.empty()) {
        int sz = q.size();
        for (int i = 0; i < sz; i++) {
            Node cur = q.front();
            q.pop();
            if (cur == target)
                return step;
            for (Node x : cur.adj()) {
                if (visited.count(x) == 0) {
                    q.push(x);
                    visited.insert(x);
                }
            }
        }
    }
    // 如果走到这里，说明在图中没有找到目标节点
}
```

## <br>支配树

```c++
namespace dtree // 支配树模板
{
    const int MAXN = 500020;
    vector<int> E[MAXN], RE[MAXN], rdom[MAXN];

    int S[MAXN], RS[MAXN], cs;
    int par[MAXN], val[MAXN], sdom[MAXN], rp[MAXN], dom[MAXN];

    void clear(int n)
    {
        cs = 0;
        for (int i = 0; i <= n; i++)
        {
            par[i] = val[i] = sdom[i] = rp[i] = dom[i] = S[i] = RS[i] = 0;
            E[i].clear();
            RE[i].clear();
            rdom[i].clear();
        }
    }
    void add_edge(int x, int y) { E[x].push_back(y); }
    void Union(int x, int y) { par[x] = y; }
    int Find(int x, int c = 0)
    {
        if (par[x] == x)
            return c ? -1 : x;
        int p = Find(par[x], 1);
        if (p == -1)
            return c ? par[x] : val[x];
        if (sdom[val[x]] > sdom[val[par[x]]])
            val[x] = val[par[x]];
        par[x] = p;
        return c ? p : val[x];
    }
    void dfs(int x)
    {
        RS[S[x] = ++cs] = x;
        par[cs] = sdom[cs] = val[cs] = cs;
        for (int e : E[x])
        {
            if (S[e] == 0)
                dfs(e), rp[S[e]] = S[x];
            RE[S[e]].push_back(S[x]);
        }
    }
    int solve(int s, int *up)
    {
        dfs(s);
        for (int i = cs; i; i--)
        {
            for (int e : RE[i])
                sdom[i] = min(sdom[i], sdom[Find(e)]);
            if (i > 1)
                rdom[sdom[i]].push_back(i);
            for (int e : rdom[i])
            {
                int p = Find(e);
                if (sdom[p] == i)
                    dom[e] = i;
                else
                    dom[e] = p;
            }
            if (i > 1)
                Union(i, rp[i]);
        }
        for (int i = 2; i <= cs; i++)
            if (sdom[i] != dom[i])
                dom[i] = dom[dom[i]];
        for (int i = 2; i <= cs; i++)
            up[RS[i]] = RS[dom[i]];
        return cs;
    }
}
```

<br>