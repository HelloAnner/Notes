Leetcode 303
```java
class NumArray {

    private int[] preSums;

    public NumArray(int[] nums) {
        preSums = new int[nums.length + 1];
        for (int i = 1; i < preSums.length; i++) {
            preSums[i] = preSums[i - 1] + nums[i - 1];
        }
    }

    public int sumRange(int left, int right) {
        return preSums[right + 1] - preSums[left];
    }
}
```

Leetcode 304 : part Two-dimensional array sum
```java
class NumMatrix {

    private int[][] preSums;

    public NumMatrix(int[][] matrix) {
        int m = matrix.length, n = matrix[0].length;
        if (m == 0 || n == 0) {
            return;
        }
        preSums = new int[m + 1][n + 1];
        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                preSums[i][j] = preSums[i - 1][j] + preSums[i][j - 1] + matrix[i - 1][j - 1] - preSums[i - 1][j - 1];
            }
        }
    }

    public int sumRegion(int row1, int col1, int row2, int col2) {
        return preSums[row2 + 1][col2 + 1] - preSums[row1][col2 + 1] - preSums[row2 + 1][col1] + preSums[row1][col1];
    }
}
```

