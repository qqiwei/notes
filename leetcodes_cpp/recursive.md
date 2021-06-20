### 二叉树

#### #236. 二叉树的最近公共祖先

```cpp
class Solution {
    std::pair<TreeNode*, TreeNode*> any;
    TreeNode* find_from(TreeNode* root) {
        // found nothing or something
        if (root == nullptr || root == any.first || root == any.second) return root;
        auto l = find_from(root->left), r = find_from(root->right);
        return l ? (r ? root : l) : r;
    }

   public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        any = {p, q};
        return find_from(root);
    }
};
```

#### #111. 二叉树的最小深度

```cpp
class Solution {
public:
    int maxDepth(TreeNode *root)  // #104
    {
        if (root == nullptr) return 0;
        return max(maxDepth(root->left), maxDepth(root->right)) + 1;
    }
    int minDepth(TreeNode *root)
    {
        if (root == nullptr) return 0;
        // if (root->left == nullptr && root->right == nullptr) return 1;
        if (root->left == nullptr) return minDepth(root->right) + 1;
        if (root->right == nullptr) return minDepth(root->left) + 1;
        return min(minDepth(root->left), minDepth(root->right)) + 1;
    }
};
```

#### #98. 验证二叉搜索树

中序遍历结果。

```cpp
class Solution {
    bool go(TreeNode* root, long min, long max) {
        if (root == nullptr) return true;

        int temp = root->val;
        if (temp <= min || temp >= max) return false;
        return go(root->left, min, temp) && go(root->right, temp, max);
    }
    void go2(TreeNode* root, std::vector<int>& order) {
        if (root == nullptr) return;

        go2(root->left, order);
        order.push_back(root->val);
        go2(root->right, order);
    }

   public:
    bool isValidBST(TreeNode* root) {
        return go(root, LONG_MIN, LONG_MAX);

        // std::vector<int> out;
        // go2(root, out);
        // for (size_t i = 1, sz = out.size(); i < sz; ++i)
        //     if (out[i - 1] >= out[i]) return false;
        // return true;
    }
};
```





### 数据结构与单元测试


#### `struct TreeNode`、`main`

```cpp
#include <iostream>

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(NULL), right(NULL) {}
    TreeNode(int x, TreeNode* l, TreeNode* r) : val(x), left(l), right(r) {}
    ~TreeNode() {
        if (left) {
            delete left;
        }
        if (right) {
            delete right;
        }
        std::cout << "gc: " << val << std::endl;
    }
};

// leetcode #236
int main() {
    // root = [3,5,1,6,2,0,8,null,null,7,4], p = 5, q = 1
    //                 3
    //          5             1
    //      6      2         0   8
    //           7   4
    auto node4 = new TreeNode(4);
    auto root = new TreeNode(
        3, new TreeNode(5, new TreeNode(6), new TreeNode(2, new TreeNode(7), node4)),
        new TreeNode(1, new TreeNode(0), new TreeNode(8)));

    std::cout << Solution().lowestCommonAncestor(root, root->left, root->right)->val
              << std::endl;
    std::cout << Solution().lowestCommonAncestor(root, root->left, node4)->val
              << std::endl;

    delete root;
    std::cout << "end." << std::endl;
}
```

