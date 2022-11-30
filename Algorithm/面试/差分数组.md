```java
int[] diff = new int[nums.length]; // 构造差分数组 
diff[0] = nums[0]; 
for (int i = 1; i < nums.length; i++) {    
     diff[i] = nums[i] - nums[i - 1]; 
}
```

```java
int[] res = new int[diff.length]; // 根据差分数组构造结果数组 
res[0] = diff[0]; 
for (int i = 1; i < diff.length; i++) {     
    res[i] = res[i - 1] + diff[i]; 
}
```

Leetcode 1109:

```java
class Difference {
    private int[] diff;

    public Difference(int[] nums) {
        assert nums.length > 0;
        diff = new int[nums.length];
        diff[0] = nums[0];
        for (int i = 1; i < nums.length; i++) {
            diff[i] = nums[i] - nums[i - 1];
        }
    }

    public void increase(int i, int j, int k) {
        diff[i] += k;
        if (j + 1 < diff.length) {
            diff[j + 1] -= k; // 抵消后面的数组的影响
        }
    }

    public int[] result() {
        int[] res = new int[diff.length];
        res[0] = diff[0];
        for (int i = 1; i < diff.length; i++) {
            res[i] = diff[i] + res[i - 1];
        }
        return res;
    }
}
```