logN需要维护一个数组保存a中出现得位置，遍历b进行不断横跳

```java
public static boolean fun(String a, String b) {
ArrayList<Integer>[] lists = new ArrayList[256];
int m = a.length();
int n = b.length();
for (int i = 0;i < n;i++) {
char c = b.charAt(i);
if (lists[c] == null) {
lists[c] = new ArrayList<>();
}
lists[c].add(i);
}
int j = 0;
for (int i = 0;i < m;i++) {
char c = a.charAt(i);
if (lists[c] == null) {
return false;
}
int p = find(lists[c], j);
if (p >= lists[c].size()) {
return false;
}
j = lists[c].get(p) + 1;
}
return true;
}

public static int find(ArrayList<Integer> nums, int j) {
int left = 0;
int right = nums.size() - 1;
while (left <= right) {
int mid = left + (right - left) / 2;
if (nums.get(mid) == j) {
return mid;
} else if (nums.get(mid) > j) {
right = mid;
} else {
left = mid + 1;
}
}
return left;
}
```