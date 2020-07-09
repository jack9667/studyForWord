递归，非递归
非递归用栈正常循环就行，按计数器返回
```java
int cnt = 0;
    TreeNode KthNode(TreeNode pRoot, int k) {
        if(pRoot != null) {
            TreeNode tmp = KthNode(pRoot.left, k);
            if(tmp != null) {
                return tmp;
            }
            cnt++;
            if(cnt == k) {
                return pRoot;
            }
            tmp = KthNode(pRoot.right, k);
            if(tmp != null) {
                return tmp;
            }
          }
        return null;
    }
```