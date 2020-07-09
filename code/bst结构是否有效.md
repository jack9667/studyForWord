
```java
public static boolean fun(TreeNode root) {
if (root == null) {
return true;
}
return core(root, null, null);
}

public static boolean core(TreeNode root, TreeNode min, TreeNode max) {
if (root == null) {
return true;
}
if (min != null && root.val <= min.val) {
return false;
}
if (max != null && root.val >= max.val) {
return false;
}
return core(root.left, min, root) && core(root.right, root, max);
}
```