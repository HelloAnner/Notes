Leetcode 206 : reverse all linked list
```java
class Solution {
    public ListNode reverseList(ListNode head) {
        if (head == null || head.next == null) {
            return head;
        }
        ListNode last = reverseList(head.next);
        head.next.next = head;
        head.next = null;
        return last;
    }
}
```

Leetcode 92 : reverse linked list betweek left and right
```java
class Solution {

    ListNode finalNode = null;

    public ListNode reverseBetween(ListNode head, int left, int right) {
        if (left == 1) {
            return this.reverseN(head, right);
        }
        head.next = this.reverseBetween(head.next, left - 1, right - 1);
        return head;
    }

    // 翻转前N个
    ListNode reverseN(ListNode head, int n) {
        if (n == 1) {
            finalNode = head.next;
            return head;
        }
        ListNode last = this.reverseN(head.next, n - 1);
        head.next.next = head;
        head.next = finalNode;
        return last;
    }
}
```