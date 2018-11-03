You are given two **non-empty** linked lists representing two non-negative integers. The digits are stored in **reverse order** and each of their nodes contain a single digit. Add the two numbers and return it as a linked list.

You may assume the two numbers do not contain any leading zero, except the number 0 itself.

**Example:**

```
Input: (2 -> 4 -> 3) + (5 -> 6 -> 4)
Output: 7 -> 0 -> 8
Explanation: 342 + 465 = 807.
```



```javascript
/**
 * Definition for singly-linked list.
 * function ListNode(val) {
 *     this.val = val;
 *     this.next = null;
 * }
 */
/**
 * @param {ListNode} l1
 * @param {ListNode} l2
 * @return {ListNode}
 */
var addTwoNumbers = function(l1, l2) {
    
    var overflow = false;
    var result = [];
    mergeToArray (l1, l2, overflow, result);
    
    return result;
};

function mergeToArray (target1, target2, overflow, result) {
    // console.log("t1 : " + target1 + "t2 :" + target2)
    var sum = Number(target1.val) + Number(target2.val);
    
    if (overflow == true) {
        sum++;
        overflow = false;
    }
    
    if (sum >= 10) {
        overflow = true;
        sum = sum - 10;
    }
    
    result.push(sum);
    
    if (target1.next === null && target2.next === null) {
        if (overflow == true) {
            result.push(1);
        }
        return;
    }
    
    var next1 = target1.next === null ? zeroNode : target1.next;
    var next2 = target2.next === null ? zeroNode : target2.next;
    
    
    // console.log("n1 : " + next1 + "n2 :" + next2);
    mergeToArray(next1, next2, overflow, result);
}


var zeroNode = {
    val : 0,
    next : null
}

```

