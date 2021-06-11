```cpp
#include <iostream>

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(NULL), right(NULL) {}
    TreeNode(int x, TreeNode* l, TreeNode* r) : val(x), left(l), right(r) {}
    ~TreeNode() {
        delTreeNode(left);
        delTreeNode(right);
        std::cout << "gc: " << val << std::endl;
    }
    void delTreeNode(TreeNode* son) {
        if (!son) return;
        son->~TreeNode();
    }
};

// 236. 二叉树的最近公共祖先
class Solution {
    std::pair<TreeNode*, TreeNode*> any;
    TreeNode* find_from(TreeNode* root) {
        // found something
        if (root == nullptr || root == any.first || root == any.second) return root;
        // found nothing
        auto l = find_from(root->left), r = find_from(root->right);
        return l ? (r ? root : l) : r;
    }

   public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        any = {p, q};
        return find_from(root);
    }
};

int main() {
    // root = [3,5,1,6,2,0,8,null,null,7,4], p = 5, q = 1
    auto node4 = new TreeNode(4);
    auto root =
        new TreeNode(3, new TreeNode(5, new TreeNode(6), new TreeNode(2, new TreeNode(7), node4)),
                     new TreeNode(1, new TreeNode(0), new TreeNode(8)));
    std::cout << Solution().lowestCommonAncestor(root, root->left, root->right)->val << std::endl;
    std::cout << Solution().lowestCommonAncestor(root, root->left, node4)->val << std::endl;
    delete root;
}
```



```cpp

```

