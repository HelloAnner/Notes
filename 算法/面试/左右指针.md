关于虚拟头节点: 当你需要创造一条新链表的时候,可以使用虚拟头结点简化边界情况的处理  


### 单遍历获取倒数K节点的位置
这里记录一个只遍历一遍链表，可以获取到倒数第K个链表的技巧，难点在于不知道整个链表的长度如何获取倒数第K个位置：
指针A从Header走K步停下，指针B从Header走，直到A走到尾部，那么AB一起走了 N-K步，B走到了 N-K+1的位置  

```java
//返回链表的倒数第k个节点
ListNode findFromEnd(ListNode head,int k){
     ListNode p1 head; 
     //p1先走k步
     for (int i=0;i<k;i++){ 
        p1 = pl.next; 
    } 
    ListNode p2 head; 
    //p1和p2同时走n-k步
    while (p1 !null){ 
        p2 = p2.next; 
        p1 = p1.next; 
    }
     //p2现在指向第n-k+1个节点，即倒数第k个节点
     return p2; 
}
```

### 快慢指针
快慢指针的经典场景就是判断链表有没有存在环，二者相遇即存在，但是如何判断环的起点呢？
```java
ListNode detectCycle(ListNode head) {
    ListNode slow,fast;
    fast = slow = head;
    while (fast != null && fast,next != null){
        fast = fast.next.next;
        slow = slow.next;
        if (fast == slow) {
            break;
        }
    }
    if (fast == null || fast.next == null){
        return null;
    }
    slow = head;
    while (slow!=fast){
        slow = slow.next;
        fast = fast.next;
    }
    return slow;
}
```


Leetcode 21 combine two sorted linked list
```java
class Solution {
    public ListNode mergeTwoLists(ListNode list1, ListNode list2) {
        ListNode dummy = new ListNode(-1), p = dummy;
        ListNode p1 = list1, p2 = list2;
        while (p1 != null && p2 != null) {
            if (p1.val > p2.val) {
                p.next = p2;
                p2 = p2.next;
            } else {
                p.next = p1;
                p1 = p1.next;
            }
            p = p.next;
        }
        if (p1 != null) {
            p.next = p1;
        }
        if (p2 != null) {
            p.next = p2;
        }
        return dummy.next;
    }
}
```

Leetcode 876 : find mid link list node

```java
class Solution {
    public ListNode middleNode(ListNode head) {
        ListNode slow = head, fast = head;
        // 快指针走到末尾时停止
        while (fast != null && fast.next != null) {
            // 慢指针走一步,快指针走两步
            slow = slow.next;
            fast = fast.next.next;
        }
        // 慢指针指向中点
        return slow;
    }
}
```


Leetcode 26: Remove Sort Array Repeat Element
```java
class Solution {
    public int removeDuplicates(int[] nums) {
        if (nums.length == 0) {
            return 0;
        }
        int slow = 0, fast = 0;
        while (fast < nums.length) {
            if (nums[fast] != nums[slow]) {
                slow++;
                nums[slow] = nums[fast];
            }
            fast++;
        }
        return slow + 1;
    }
}
```

Leetcode 83: Remove sorted link list repeat node
```java
class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        if (head == null) {
            return null;
        }
        ListNode slow = head, fast = head;
        while (fast != null) {
            if (fast.val != slow.val) {
                slow.next = fast;
                slow = slow.next;
            }
            fast = fast.next;
        }
        slow.next = null;
        return head;
    }
}
```