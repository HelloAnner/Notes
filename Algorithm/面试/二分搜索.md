Templates
```java
/*  
 * @lc app=leetcode.cn id=704 lang=java * * [704] 二分查找 */  
// @lc code=start  
class Solution {  
    public int search(int[] nums, int target) {  
        int left = 0, right = nums.length - 1;  
        while (left <= right) {  
            int mid = left + (right - left) / 2;  
            if (nums[mid] == target) {  
                return mid;  
            } else if (nums[mid] < target) {  
                left = mid + 1;  
            } else if (nums[mid] > target) {  
                right = mid - 1;  
            }  
        }  
        return -1;  
    }  
}  
// @lc code=end
```


Left Search:
```java
class Solution {
    public int shipWithinDays(int[] weights, int days) {
        if (weights.length == 0) {
            return -1;
        }

        int left = 0, right = weights.length;
        while (left < right) {
            int mid = left + (right - left) / 2;
            if (weights[mid] == days) {
                right = mid;
            } else if (weights[mid] < days) {
                left = mid + 1;
            } else if (weights[mid] > days) {
                right = mid;
            }

        }
        return left;
    }
}
```


Right Search:
```java
class Solution {
    public int shipWithinDays(int[] weights, int days) {
        if (weights.length == 0) {
            return -1;
        }

        int left = 0, right = weights.length;
        while (left < right) {
            int mid = left + (right - left) / 2;
            if (weights[mid] == days) {
                left = mid+1;
            } else if (weights[mid] < days) {
                left = mid + 1;
            } else if (weights[mid] > days) {
                right = mid;
            }

        }
        return left-1;
    }
}
```

