## 反转链表
public ListNode ReverseList(ListNode head) {
        ListNode pre = null;
        ListNode cur = head;
        ListNode nex = null; // 这里可以指向nullptr，循环里面要重新指向
        while (cur != null) {
            nex = cur.next;
            cur.next = pre;
            pre = cur;
            cur = nex;
        }
        return pre;
    }

## 快排
public int[] MySort(int[] arr) {
    quickSort(arr, 0, arr.length - 1);
    return arr;
}
 
private void quickSort(int[] array, int start, int end) {
    if (start < end) {
        int key = array[start];//用待排数组的第一个作为中枢
        int i = start;
        for (int j = start + 1; j <= end; j++) {
            if (key > array[j]) {
                swap(array, j, ++i);
            }
        }
        array[start] = array[i];//先挪，然后再把中枢放到指定位置
        array[i] = key;
        quickSort(array, start, i - 1);
        quickSort(array, i + 1, end);
    }
}
 
//交换两个数的值
public void swap(int[] A, int i, int j) {
    if (i != j) {
        A[i] ^= A[j];
        A[j] ^= A[i];
        A[i] ^= A[j];
    }
}

## 树遍历
public void preorder(TreeNode root, List<Integer> list) {
    if (root != null) {
        list.add(root.val);
        preorder(root.left, list);
        preorder(root.right, list);
    }
}
public void inorder(TreeNode root, List<Integer> list) {
    if (root != null) {
        inorder(root.left, list);
        list.add(root.val);
        inorder(root.right, list);
    }
}
public void postorder(TreeNode root, List<Integer> list) {
    if (root != null) {
        postorder(root.left, list);
        postorder(root.right, list);
        list.add(root.val);
    }
}
## 层序遍历
 ArrayList<ArrayList<Integer>> res = new ArrayList<ArrayList<Integer>>();
    public ArrayList<ArrayList<Integer>> levelOrder (TreeNode root) {
        // write code here
        if(root == null){
            return res;
        }
        count(root,0);
        return res;
    }
 
    public void count(TreeNode node, int level){
        if(level == res.size()){
            res.add(new ArrayList<Integer>());
        }
 
       ArrayList<Integer> list = res.get(level);
       list.add(node.val);
 
       if(node.left != null){
           count(node.left, level+1);
       }
 
       if(node.right != null){
           count(node.right, level+1);
       }
 
    }
## 两数之和
 public int[] twoSum (int[] numbers, int target) {
        int[] result = new int[2];
        Map<Integer, Integer> map = new HashMap();
        for(int i = 0; i < numbers.length; i++) {
            if(map.get(target - numbers[i]) != null) {
                result[0] = map.get(target - numbers[i]) + 1;
                result[1] = i + 1;
                return result;
            }
            map.put(numbers[i], i);
        }
        return result;
    }
## 栈实现队列
 public void push(int node) {
        stack1.push(node);
    }
 
    public int pop() {
        if (stack2.size() <= 0) {
            while (stack1.size() != 0) {
                stack2.push(stack1.pop());
            }
        }
        return stack2.pop();
    }
## 跳台阶
 public int JumpFloor(int target) {
        if (target <= 1) {
            return 1;
        }
        // a 表示第 f[i-2] 项，b 表示第 f[i-1] 项
        int a = 1, b = 1, c = 0;
        for (int i = 2; i <= target; i++) {
            c = a + b; // f[i] = f[i - 1] + f[i - 2];
            // 为下一次循环求 f[i + 1] 做准备
            a = b; // f[i - 2] = f[i - 1]
            b = c; // f[i - 1] = f[i]
        }
        return c;
    }
## 反转K个节点
public ListNode reverseKGroup (ListNode head, int k) {
        if(head==null||head.next==null||k==1) return head;
        ListNode res = new ListNode(0);
        res.next = head;
        int length = 0;
        ListNode pre = res,
                 cur = head,
                 temp = null;
        while(head!=null){
            length++;
            head = head.next;
        }
        //分段使用头插法将链表反序
        for(int i=0; i<length/k; i++){
            //pre作为每一小段链表的头节点，负责衔接
            for(int j=1; j<k; j++){
                temp = cur.next;
                cur.next = temp.next;
                //相当于头插法，注意：
                //temp.next = cur是错误的，temp需要连接的不是前一节点，而是子序列的头节点
                temp.next = pre.next;
                pre.next = temp;
            }
            //每个子序列反序完成后，pre，cur需要更新至下一子序列的头部
            pre = cur;
            cur = cur.next;
        }
        return res.next;
    }

## 最长回文
public static int getLongestPalindrome(String A, int n) {
       // write code here
       char[] aa =A.toCharArray();
       int max=1;
       boolean[][] dp = new boolean[n][n];
       for(int i=0;i<n;i++){
           dp[i][i] = true;
       }
       for(int i=1;i<n;i++)//i指向的是字符的最后一位
           for(int j=i-1;j>=0;j--){//j指向的是字符的前部。
               if(i-j==1){//当两个指针靠近时，直接判断
                   dp[j][i]=(aa[i]==aa[j]);
                   if(max<i-j+1)
                       max = i-j+1;
               }
 
               else{
                   if(dp[j+1][i-1]&&aa[i]==aa[j]){
                       dp[j][i]=true;
                       if(max<i-j+1)
                           max = i-j+1;
                   }
                   else
                       dp[j][i]=false;
               }
           }
       return max;
   }
## 最长公共字串
 public static String LCS1(String str1, String str2) {
        int maxLength = 0;
        int index = 0;
        for(int i = 0; i < str2.length(); i++){
            for(int j = i+1; j <= str2.length(); j++){
                if(str1.contains(str2.substring(i, j)) ){
                    if(maxLength < j-i){
                        maxLength = j-i;
                        index = i;
                    }
                } else break;
            }
        }
        if( maxLength == 0 ){ //没有相同的字符串
            return "-1";
        }
        return str2.substring(index,index + maxLength);
    }