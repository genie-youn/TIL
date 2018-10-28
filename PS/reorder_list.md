Given a singly linked list *L*: *L*0→*L*1→…→*L**n*-1→*L*n,
reorder it to: *L*0→*L**n*→*L*1→*L**n*-1→*L*2→*L**n*-2→…

You may **not** modify the values in the list's nodes, only nodes itself may be changed.



```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public void reorderList(ListNode head) {
        List<Integer> nodes = convertToList(head);
        if (nodes.size() < 3) {
            return; 
        } 
        
        int middleIndex = (nodes.size() / 2) + (nodes.size() % 2);
        List<Integer> heads = nodes.subList(0, middleIndex);
        List<Integer> tails = nodes.subList(middleIndex, nodes.size());
        Collections.reverse(tails);
        
        changeValue(head, heads, tails, 0);
    }
    
    private List<Integer> convertToList(ListNode first) {
        if (first == null) return new ArrayList<>();
        List<Integer> nodes = new ArrayList<>();
        recursiveAdd(first, nodes);
        return nodes;
        
    }
    
    private void recursiveAdd(ListNode targetNode, List<Integer> bucket) {
        bucket.add(targetNode.val);
        if (targetNode.next != null) {
            recursiveAdd(targetNode.next, bucket);
        }
    }
    
    private void changeValue(ListNode node, List<Integer> heads, List<Integer> tails, int index) {    
        node.val = heads.get(index);
        if (node.next != null) {
            node.next.val = tails.get(index);
            index++;
            if (node.next.next != null) changeValue(node.next.next, heads, tails, index);
        }  
    }
}
```

