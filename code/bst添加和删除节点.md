
```java
//删除
public static TreeNode fun1(TreeNode root, int k) {
if (root == null) {
return root;
}
if (root.val == k) {
if (root.right == null) {
return root.left;
}
if (root.left == null) {
return root.right;
}
TreeNode tmp = getMin(root.right);
root.val = tmp.val;
root.right = fun1(root.right, tmp.val);
} else if (root.val < k) {
root.right = fun1(root.right, k);
} else if (root.val > k) {
root.left = fun1(root.left, k);
}
return root;
}
public static TreeNode getMin(TreeNode root) {
if (root == null) {
return root;
}
while (root.left != null) {
root = root.left;
}
return root;
}
// 添加
public static TreeNode fun(TreeNode root, int x) {
if (root == null) {
return new TreeNode(x);
}
if (root.val < x) {
root.right = fun(root.right, x);
} else if (root.val > x){
root.left = fun(root.left, x);
}
return root;
}
```