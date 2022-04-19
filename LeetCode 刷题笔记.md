# LeetCode 刷题笔记

## 4 H 寻找两个正序序列的中位数

​	用一种很暴力的思路写出来了，先拼接在排序，最后输出中位数。

​	本想用这道题试试多线程操作，可是发现多次执行程序时，只有第一次操作是正确的，原因未知。

​	错误代码：

```java
 public static double solution(int[] nums1, int[] nums2) {
        int len = nums1.length + nums2.length;
        int[] nums3 = new int[len];
        double mid = 0;
        new Thread(() -> {
            for (int i = 0; i < nums1.length; i++) {
                nums3[i] = nums1[i];
            }
        }).start();

        new Thread(() -> {
            for (int i = nums1.length; i < len; i++) {
                nums3[i] = nums2[i - nums1.length];
            }
        }).start();

        Arrays.sort(nums3);
        if (len % 2 == 0) {
            mid = (double) (nums3[len / 2] + nums3[len / 2 - 1]) / 2;
        } else {
            mid = nums3[len / 2];
        }

        return mid;
    }
```



##  18 M 四数之和

​	四数之和是三数之和的进阶题目，但是循环变多之后，暴露出很多问题。

​	1. 如何遍历数组中数字组成的4元组？（同一个位置的数字只能使用一次）

```java
for(int i=0;i<nums.length-n;i++){
    for(int j=i+1;j<nums.length-n+1;j++){
        for(int k=j+1;k<nums.length-n+2;k++){
            for(int l=k+1 ......)
        }
    }
}
```

2. 如何在循环时控制指针

```java
//对于循环控制的指针更新，可以通过continue跳出一次循环判断指针

for(int i=0;i<nums.length;i++){
    if(i>0 && nums[i] == nums[i-1]){
        continue;
    }
}

while( left < right ){
    while (right < length - 1 && nums[right] == nums[right + 1]) {
      		right--;
    }
    while (left > 0 && nums[left] == nums[left + 1]) {
      		left++;
    }
    continue;
}

```

## 13 罗马数字转整数

 - 罗马字的排列比较有趣，是驼峰式的排序，最大值左边的值要减去，最大值以及最大值右边的数字要加上。并且，左边的值可以缺省。

 - 有一个多余的储存空间，可以判断某个位置的罗马字与后面的大小关系，< 就减去，>= 才加上。

 - **有趣的字典记录方法**：

   ```java
    public int getValue(char s){
           switch(s){
               case 'I': return 1;
               case 'V': return 5;
               case 'X': return 10;
               case 'L': return 50;
               case 'C': return 100;
               case 'D': return 500;
               case 'M': return 1000;
               default : return 0;
           }
       }
   ```

## 15 M 三数之和

​	思路：排序后三重循环。

​	一种去重的思路，

```java
for(int i=0; i<nums.length-1; i++){
            if(i>0 && nums[i] == nums[i-1]){
                continue;
            }
}
```

​	bug反思：while，for循环条件，只有结束时单词循环结束时才判断一次，中间仍有可能发生越界等等错误。

## 16 M 最接近的三数之和

最初的思路，仍然是三重循环，更新最小值。效果不太好。

​	bug反思：while循环中的初始化变量不会在每次循环的时候再次更新，如果不需要更新，最好放在for循环外边，需要更新就放在for循环里边。

 	进阶，后面两重循环改为使用双指针法，将复杂度从O(n2)降低到O(n)。

```java
Arrays.sort(nums);
        int diff = 10000;
        int bestSum = 10000;
        int sum = 0;
        for (int i = 0; i < nums.length - 2; i++) {
            if (i > 0 && nums[i] == nums[i - 1]) {
                continue;
            }
            int j = i + 1;
            int k = nums.length - 1;
            while (j < k) {
                while (j > i + 1 && nums[j] == nums[j - 1] && j < k - 1) {
                    j++;
                }
                while (k < nums.length-1 && nums[k] == nums[k + 1] && j < k - 1) {
                    k--;
                }
                sum = nums[i] + nums[j] + nums[k];
                if (Math.abs(sum - target) < diff) {
                    diff = Math.abs(sum - target);
                    bestSum = sum;
                }
                if (sum > target)
                    k--;
                else if (sum < target)
                    j++;
                else if (sum == target)
                    return sum;
            }
        }
        return bestSum;
    }
```



## 20 有效的括号

​	第一次写栈的题，可以以后再练习一下

## 35 搜索插入位置

### 二分查找

题目	题解	难度	推荐指数
4. 寻找两个正序数组的中位数	LeetCode 题解链接	困难	🤩🤩🤩🤩
29. 两数相除	LeetCode 题解链接	中等	🤩🤩🤩
33. 搜索旋转排序数组	LeetCode 题解链接	中等	🤩🤩🤩🤩🤩
34. 在排序数组中查找元素的第一个和最后一个位置	LeetCode 题解链接	中等	🤩🤩🤩🤩🤩
35. 搜索插入位置	LeetCode 题解链接	简单	🤩🤩🤩🤩🤩
74. 搜索二维矩阵	LeetCode 题解链接	中等	🤩🤩🤩🤩
81. 搜索旋转排序数组 II	LeetCode 题解链接	中等	🤩🤩🤩🤩
153. 寻找旋转排序数组中的最小值	LeetCode 题解链接	中等	🤩🤩🤩
154. 寻找旋转排序数组中的最小值 II	LeetCode 题解链接	困难	🤩🤩🤩
220. 存在重复元素 III	LeetCode 题解链接	中等	🤩🤩🤩
278. 第一个错误的版本	LeetCode 题解链接	简单	🤩🤩🤩🤩
354. 俄罗斯套娃信封问题	LeetCode 题解链接	困难	🤩🤩🤩
363. 矩形区域不超过 K 的最大数值和	LeetCode 题解链接	困难	🤩🤩🤩
374. 猜数字大小	LeetCode 题解链接	简单	🤩🤩🤩
778. 水位上升的泳池中游泳	LeetCode 题解链接	困难	🤩🤩🤩
852. 山脉数组的峰顶索引	LeetCode 题解链接	简单	🤩🤩🤩🤩🤩
981. 基于时间的键值存储	LeetCode 题解链接	中等	🤩🤩🤩🤩
1004. 最大连续1的个数 III	LeetCode 题解链接	中等	🤩🤩🤩
1011. 在 D 天内送达包裹的能力	LeetCode 题解链接	中等	🤩🤩🤩🤩
1208. 尽可能使字符串相等	LeetCode 题解链接	中等	🤩🤩🤩
1438. 绝对差不超过限制的最长连续子数组	LeetCode 题解链接	中等	🤩🤩🤩
1482. 制作 m 束花所需的最少天数	LeetCode 题解链接	中等	🤩🤩🤩
1707. 与数组中元素的最大异或值	LeetCode 题解链接	困难	🤩🤩🤩
1751. 最多可以参加的会议数目 II	LeetCode 题解链接	困难	🤩🤩🤩



二分查找的要素

- 指针的更新 右更**mid-1**或 左更**mid+1**。
- 控制循环的条件。

这种情况下，要求该数字必须存在于数组中，否则会返回-1。

二分的精髓是：将数据从中间分成两端，一段满足条件，另一端不满足条件来折半的放弃数据，来达到`O(logn)`的复杂度。

所以，一般的二分查找算法要求所查询的数字是唯一满足条件的数据，此时算法收敛。但如果并非如此，比如查找插入位置，原序列并不存在满足条件的那个索引，就会出现左指针越过右指针的情况。这种情况出现在，左右指针正好被更新到所求的索引附近，此时`配合right = mid -1; left = mid + 1;`的更新方法就会出现跨越所求解的情况。

```java
public static int indexOf(int[] a, int key) {
        int lo = 0;
        int hi = a.length - 1;
        while (lo <= hi) {
            // Key is in a[lo..hi] or not present.
            int mid = lo + (hi - lo) / 2;
            if      (key < a[mid]) hi = mid - 1;
            else if (key > a[mid]) lo = mid + 1;
            else return mid;
        }
        return -1;
    }

```

### 进阶情况

​	原情况中，当左右指针缩短至相邻情况时，这次循环必定会由于触发```key === a[mid]```而返回。而当查找值不存在于数组中时，在这种情况下就会出现左右指针中某一个超越对方的情况。需要将循环条件改为判断指针是否相邻来结束循环，此时本该结束的循环会终止在相邻情况，再加一个判断来获取正确输出。

```java
public int searchInsert(int[] nums, int target) {
        int right = nums.length-1;
        int left = 0;
        return binarySearch(nums, left, right, target);
    }

    public int binarySearch(int[] nums, int lp, int rp, int target){
        if(target > nums[nums.length-1]){
            return nums.length;
        }else if(target < nums[0]){
            return 0;
        }
        while( rp-lp>1 ){
            if (target  > nums[(rp+lp)/2]){
                lp = (rp+lp)/2;
            }else if(target < nums[(rp+lp)/2]){
                rp = (rp+lp)/2;
            }else if(target == nums[(rp+lp)/2]){
                return (rp+lp)/2;
            }
        }
        
        if(nums[lp]!=target){
            return lp+1;
        }else{
            return lp;
        }
    }
```

### tips

 	*2 或者 /2 可以使用移位运算符>> 1、<< 1代替。>>>会忽略符号位，用零补齐。

## 37 H 序列化二叉树

关键在于如何序列化二叉树，并将二叉树还原出来。

根据图论的原理，想要靠深先遍历将树还原出来，至少需要中序以及先序或后序中的一种。

本题使用广先遍历，将null的位也存储到字符串中。

在序列化的过程中，依照普通的广先遍历来写就可以，如过遇到左右子树为空的情况也没有关系，直接入队。在出队的时候进行判断，如果为空直接打印就行，因为将这种方法本就是将null作为节点写入树中，所以不用担心会影响顺序。

在反序列化的过程中，按理说就是序列化的逆过程，依然需要依靠队列。将节点入队，每次从字符串中取两个节点作为左右子树，然后将节点入队，如果为空就不入队。

## 39 M 组合总和  I

​	递归+回溯法

​	根据题意，要找到集合中所有和为```target```的元素组合，所以必须对可能的情况进行完全的遍历。

 	回溯法的基本行为是搜索，搜索过程使用剪枝函数来为了避免无效的搜索。

​	剪枝函数包括两类：1. 使用约束函数，剪去不满足约束条件的路径；2.使用限界函数，剪去不能得到最优解的路径。（此题没有要求）

## 40 M 组合总和 II

​	与39题相比，由于不能使用重复用元素，所以在传递序号时，要传递```idx + 1```。

## 46 M 全排列

方法一	：回溯算法，多使用一个列表用于标记数据是否使用，原思路跟随回溯步骤动态删除添加数据，由于改动了数据位置一再出错。

tips:   只有原始变量数组在创建时会被赋予初始值，他们的包装类新以及其他对象数组则不会。

## 47 M 全排列 II

相比于46（数组内每个元素只能使用一次），升级了一个限制条件，数组中包含重复元素，其难度主要在于如何去重。

可以依靠树的机构去思考问题，每一层的节点的候选元素是等价的，已经用过的元素不在下一次候选元素之列。

所以去重的关键在于，让相同的元素不能出现在排列的同意位置，例如[1, 2, 1]中的“1”，如果第一个“1”占有过“0”位置，那么控制其他的1不能占有“1”位置就不会出现重复。就像是遍历组合，第二重循环只能从第一重循环所选择数字的后面下表开始选择。

应该跳过的元素的特点：

1. 与前一元素处于同一层（必然）
2. 与前一元素数值上相同
3.  前一元素没有使用

解读：前两条很明显，第三条稍难，有些反直觉。前一元素如果被使用了，说明该元素与前一元素占有不同的位置，处于不同的情况；而在前一元素没有使用时，因为它们是相邻的，说明前一元素刚刚被回溯，不应当再将同样数值的元素加入，如果此时加入会添加再同样的位置。

```java
if (!used[i]) {
                if (i > 0 && nums[i - 1] == nums[i] && used[i - 1] == false) {
                    continue;
                }
                path.add(nums[i]);
                used[i] = true;
                depth += 1;
                allSort(nums, depth, path, res, used);
                path.remove(path.size() - 1);
                used[i] = false;
                depth -= 1;
            }
```

## 49 字母异位词分组

本来的解法：为每个单词创建一个字典，将出现的字符和出现次数储存在哈希表中，然后通过比较两个词的哈希表是否相同确定是否为异位词。

思路方向倒是没问题，本题需要找到一个方法，将异位词映射为同一个键。

1. 可以将词转化为字符数组并排序，异位词的数组必然是等同的。

```java
class Solution {
    public List<List<String>> groupAnagrams(String[] strs) {
        Map<String, List<String>> map = new HashMap<String, List<String>>();
        for (String str : strs) {
            char[] array = str.toCharArray();
            Arrays.sort(array);
            String key = new String(array);
            List<String> list = map.getOrDefault(key, new ArrayList<String>());
            list.add(str);
            map.put(key, list);
        }
        return new ArrayList<List<String>>(map.values());
    }
}
```

2. 对异位词进行编码，如`aaabbc`编码为`a3b2c1`。

   ```java
   class Solution {
       public List<List<String>> groupAnagrams(String[] strs) {
           Map<String, List<String>> map = new HashMap<String, List<String>>();
           for (String str : strs) {
               int[] counts = new int[26];
               int length = str.length();
               for (int i = 0; i < length; i++) {
                   counts[str.charAt(i) - 'a']++;
               }
               // 将每个出现次数大于 0 的字母和出现次数按顺序拼接成字符串，作为哈希表的键
               StringBuffer sb = new StringBuffer();
               for (int i = 0; i < 26; i++) {
                   if (counts[i] != 0) {
                       sb.append((char) ('a' + i));
                       sb.append(counts[i]);
                   }
               }
               String key = sb.toString();
               List<String> list = map.getOrDefault(key, new ArrayList<String>());
               list.add(str);
               map.put(key, list);
           }
           return new ArrayList<List<String>>(map.values());
       }
   }
   ```

   但还有一点，如何完成这个两两比较的步骤，开始写的时候，选择的是遍历二元组，复杂度`O(n)`。

   但可以利用`HashMap`来完成一次遍历复杂度`O(n)`。（这才是真的要用哈希表的地方啊，OTZ）

来自甜姨的上流解法：

```java
class Solution {
    public List<List<String>> groupAnagrams(String[] strs) {
        return new ArrayList<>(Arrays.stream(strs)
            .collect(Collectors.groupingBy(str -> {
                // 返回 str 排序后的结果。
                // 按排序后的结果来grouping by，算子类似于 sql 里的 group by。
                char[] array = str.toCharArray();
                Arrays.sort(array);
                return new String(array);
            })).values());
    }
}
```



## 53 最大子序和

贪心法、动态规划	

个人觉得，动态规划难在如何确认状态。正确的确立状态之后，转移方程和初始条件，边界条件都会简单很多。

在确定状态时，一定要和转移方程地建立相适应。在本题中，同样建立一个`dp[]`数列来储存结果，如果将状态设定为，`dp[i]`用来表示，在`i`之前得到的最佳子序和，那么`dp[i]`和`dp[i-1]`之间便很难建立关系，因为仅仅是`i`之前得到的最佳子序和，最佳子序列可能与第`i`个元素相邻，也可能不相邻，这样就写不出状态转移方程。（如果这种建立状态的方法成立，最后需要返回的结果是`dp[len-1]`）

正确是方法是`dp[i]`用来表示，终点在`i`的子序列的最佳子序和，这样`dp[i]`和`dp[i-1]`之间便有简单明了的关系。这时需要用到贪心法。此时，最后返回的值应该从`dp`数组中取最大值，即以`i`结尾的数组的最大子序和中找到最大的，作为整个数组的最佳子序和。

```java
dp[i] = Math.max(dp[i - 1], 0) + num[i];
```





## 70 E 爬楼梯

错误代码

在递归的调用函数的时候，虽然初始类型`int res`形参的写法相同，但是并非是一个参数（引用对象应该是一样的）。所以在更深的节点中，`res`事实上+1了，但结果返回调用的上级函数时，返回的并非被修改的`res`，而是原函数中没有被更改的`res`。

```java
class Solution {
    public static void main(String[] args) {
        
        System.out.println(climb(0, 2));
    }

    public static int climb(int res, int n){
        if(n == 0){
            res += 1;
            return 0;
        }else if( n < 0){
            return 0;
        }

        climb(res, n - 1);
        climb(res, n - 2);
        return res;
    }
}
```

正确代码：普通递归

通过每个节点，返回数值的方式解决了上述问题。凡是由于遍历所有节点，复杂度呈`2^n`的趋势上升，很快便超过了时间限制。

```java
 public static int climb( int n){
        if(n == 0){
            return 1;
        }else if( n < 0){
            return 0;
        }

        return climb(n - 1)  + climb(n - 2);
    }
```

尾递归

什么叫尾调用优化，即在函数结束时调用其他函数，这样调用栈中仅需存放一个需要执行的信息，可以减少内存消耗。

```java
 public static int climbStairs(int n) {
        return Fibonacci(n, 1, 1);
    }

    public static int Fibonacci(int n, int a, int b) {
        if (n <= 1)
            return b;
        return Fibonacci(n - 1, b, a + b);
    }
```

斐波那契数列的通项公式

[356，青蛙跳台阶相关问题](https://mp.weixin.qq.com/s/hLpHLUfXsABzUNjuNflWzQ)

非递归

```java
public int climbStairs(int n) {
    if (n <= 1)
        return 1;
    int[] dp = new int[n + 1];
    dp[1] = 1;
    dp[2] = 2;
    for (int i = 3; i <= n; i++) {
        dp[i] = dp[i - 1] + dp[i - 2];
    }
    return dp[n];
}
```

非递归优化

```java
public int climbStairs(int n) {
    if (n <= 2)
        return n;
    int first = 1, second = 2, sum = 0;
    while (n-- > 2) {
        sum = first + second;
        first = second;
        second = sum;
    }
    return sum;
}
```

## 88 E 合并两个有序数组

if else 分区嵌套



## 108 E 将有序数组转换为二叉搜索树

二分递归（如何优雅地进行二分并边界健壮的结束？） + 将节点挂在树上。

每次递归的时候，将`mid`的值更近一步，不仅是为了加快收敛，也是为了结束条件`start > end`。否则在两指针相邻的时候，由于Java对`int`除法的舍入问题，将导致一致无法更新左指针，导致死循环内存溢出。

```java
if(start > end){
    return null;
}
int mid = (start + end) >> 1
function(start, mid-1, nums[]);
function(mid+1, end, num s[]);
```

## 121 E  买卖股票的最佳时机

动态规划确定状态很重要，一般最终状态就是求解目标。

我们来定义一个二维数组`dp[length][2]`，其中`dp[i][0]`表示第`i+1`天（`i`是从0开始的）结束的时候没持有股票的最大利润，`dp[i][1]`表示第`i+1`天结束的时候持有股票的最大利润。

第一种情况就是第`i+1`天我们即没买也没卖，那么最大利润就是第`i`天没持有股票的最大利润`dp[i-1][0]`。

第二种情况就是第`i+1`天我们卖了一支股票，那么最大利润就是第i天持有股票的最大利润（这个是负的，并且也不一定是第i天开始持有的，有可能在第`i`天之前就已经持有了）加上第`i+1`天卖出股票的最大利润，`dp[i-1][1]+prices[i]`。

## 149 H 直线上最多的点数

HashMap不能用数组作为键，补一补HashMap的原理

## 168  E  Excel表列名称

这道题是一道进制转换的变种。n进制数中，并不会出现n。每一位上共会有 [0,n-1]共n个数字，第n个数是10。本题中，每一位上总共可能有26个符号，所以是26进制无疑。

本题中，每一位的累加，是从“A”到“Z”的,26进制数是从“0”到“25”.
也就是说这道题字母改变的规律和一个26进制数的累加规律没有任何区别。

这也是本题能够进行逐步求余的基础。

但平常的进制中并不会出现首位为0的情况，所以A作为第一个字母，并不能直接对应0。
如果A对应1，那么Z便对应到了26进制的10。此时需要进行判断，向前一位借位，便出现了第一种解法。

```java
class Solution {
    public String convertToTitle(int columnNumber) {
        StringBuilder sb = new StringBuilder();
        while(columnNumber != 0){
            char c =  (char)(columnNumber%26  + '@');
            columnNumber /= 26;
            if(c == '@') {
                c = 'Z';
                columnNumber -= 1;
            }
            sb.append(String.valueOf(c));
        }
        return sb.reverse().toString();
    }
}
```

第二种解法，就是将数字减一，相当于将A对应到了0，Z对应到25，A-Z就可以正常的排列了。

```java
class Solution {
    public String convertToTitle(int cn) {
        StringBuilder sb = new StringBuilder();
        while (cn > 0) {
            cn--;
            sb.append((char)(cn % 26 + 'A'));
            cn /= 26;
        }
        sb.reverse();
        return sb.toString();
    }
}
```

## 198 M 打家劫舍  1

个人认为动态规划，确定状态大于一切。我就常常找不准状态而导致思路卡壳。 

该题的状态看似比较容易确定，即`dp[i]`代表打劫到`i`的位置时，所能获得的最大收益，但这时转移方程就不太好确定了。

直觉上， 可以现根据打劫房屋不能相邻的规则写出了下式：

```java
dp[i] = Math.max(dp[i-2] + nums[i], dp[i-1])
```

但是这样定义，`dp[i-1]`中`nums[i-1]`可能被打劫到，也可能没有，思路就断了。（其实，如果`nums[i-1]`没有被打劫到的话，`dp[i-2] `一定是和`dp[i-1]`相等的，并不影响结果。）

与第 [53 最大子序和](# 53 最大子序和) 题十分类似，如果将`dp[i]`代表打劫到`i`的位置便很容易出错，但是如果用`dp[i]`表示最后一家抢劫索引`i`位置的房屋的最大收益，再看上式，就简单清楚多了。`dp[i]`根据抢还是不抢第`i+1`栋房子，轻易的就写出了转移方程。

## 204 计数质数

埃式筛选法，将素数的倍数标记为合数，是列出最小素数最有效的方法之一。

```java
class Solution {
    public int countPrimes(int n) {
        boolean[] arr = new boolean[n];
        int cnt = 0;
        for(int i = 2; i < n; i++) {
            if(arr[i]) continue;
            cnt++;
            for(int j = i; j < n; j+=i) {
                arr[j] = true;
            }
        }
        return cnt;
    }
}
```

## 274 M H指数

很有趣的一道题，这道题排序后很简单，用二分法会更快。

二分的不是给定的数组，而是H值。

充分反映了二分法灵活性。

## 275 M H指数II

二分法的指针跟新也很有讲究，如果只更新到左右指针都只更新到mid，那么就会出现无限循环的情况。但是只要有一边更新的时候加一或者减一，就不会出现这种情况。`r = mid -1`还是`r = mid`，只需看r是否满足条件，可以作为输出。

## 384 M 打乱数组

相对于成熟的洗牌算法，我遍历数组中每个数，并将它与0到n-1中的随机数字交换顺序，这样的得到的数组也是符合条件的随机数列。

那，我的这种方法和现有的随机算法有什么区别呢？

区别大了，随机将数字与0到n-1位置的数字交换，会导致知道所有数字遍历完之前，数组都所有位置都有可能会改变。

这道题目没有很好的考出的一点是，如果问题是要求从数组中随机取出k个数，我的算法都需要遍历数组才行，而kd算法等shuffle算法，操作一次产生的就是一个满足平均分布的随机数！

## 401 E 二进制手表

血泪教训，不重复遍历回溯法，一定要传`i+1`作为接下来的`index`, 而不是`index+1`，因为遍历回来后`index`还是初始值，`index`并没有更新。

```java
 for(int i=index; i<bit.length;i++){
                bit[i] = 1;
                System.out.println(index);
                possibleBit(count-1, i+1, bit);
                bit[i] = 0;
            }
```

## 852 E 山峰数组的封顶索引

二分的精髓是：将数据从中间分成两端，一段满足条件，另一端不分组条件来折半的放弃数据，来达到`O(logn)`的复杂度。

所以，一般的二分查找算法要求所查询的数字是唯一满足条件的数据，此时算法收敛。但如果并非如此，比如查找插入位置，原序列并不存在满足条件的那个索引，就会出现左指针越过右指针的情况。这种情况出现在，左右指针正好被更新到所求的索引附近，此时`配合right = mid -1; left = mid + 1;`的更新方法就会出现跨越所求解的情况。

## 1600 M 皇位继承

纪念以下第一次能使用 Map + 多叉树了。

```java
class ThroneInheritance {
    Map<String, TreeNode> mapTree;
    TreeNode king;
    List<String> inheritanceOrder;


    public ThroneInheritance(String kingName) {
        this.king = new TreeNode(kingName);
        this.mapTree = new HashMap<>();
        mapTree.put(kingName, king);
    }
    
    public void birth(String parentName, String childName) {
        TreeNode father = mapTree.get(parentName);
        TreeNode child = new TreeNode(childName);
        father.addChild(child);
        mapTree.put(childName, child);
    }
    
    public void death(String name) {
        TreeNode deathPeople = mapTree.get(name);
        deathPeople.death();
    }
    
    public List<String> getInheritanceOrder() {
        this.inheritanceOrder = new ArrayList<>();
        getInheritor(king);
        return inheritanceOrder;
    }

    public void getInheritor(TreeNode node){
        if(!node.death){
            inheritanceOrder.add(node.name);
        }
        for(int i = 0; i< node.childList.size();i++){
            getInheritor(node.childList.get(i));
        }
    }

    class TreeNode{
        String name;
        List<TreeNode> childList;
        boolean death = false;

        public TreeNode(String name){
            this.childList = new ArrayList<>();
            this.name = name;
        }
        public void addChild(TreeNode child){
            childList.add(child);
        }
        public void death(){
            this.death = true;
        }
    }
}

/**
 * Your ThroneInheritance object will be instantiated and called as such:
 * ThroneInheritance obj = new ThroneInheritance(kingName);
 * obj.birth(parentName,childName);
 * obj.death(name);
 * List<String> param_3 = obj.getInheritanceOrder();
 */
```

```java
其他「链表」题解

2. 两数相加	LeetCode 题解链接	中等	🤩🤩🤩
3. 删除链表的倒数第 N 个结点	LeetCode 题解链接	中等	🤩🤩🤩🤩🤩
4. 合并两个有序链表	LeetCode 题解链接	简单	🤩🤩🤩🤩🤩
5. 合并K个升序链表	LeetCode 题解链接	困难	🤩🤩🤩
6. 两两交换链表中的节点	LeetCode 题解链接	中等	🤩🤩🤩🤩
7. K 个一组翻转链表	LeetCode 题解链接	困难	🤩🤩
8. 旋转链表	LeetCode 题解链接	中等	🤩🤩🤩
9. 删除排序链表中的重复元素	LeetCode 题解链接	简单	🤩🤩🤩🤩🤩
10. 删除排序链表中的重复元素 II	LeetCode 题解链接	简单	🤩🤩🤩🤩🤩
11. 反转链表 II	LeetCode 题解链接	中等	🤩🤩🤩
12. 相交链表	LeetCode 题解链接	简单	🤩🤩🤩🤩🤩
13. LRU 缓存机制	LeetCode 题解链接	中等	🤩🤩🤩🤩🤩
14. 移除链表元素	LeetCode 题解链接	简单	🤩🤩🤩
15. LFU 缓存	LeetCode 题解链接	困难	🤩🤩🤩🤩🤩
```

#### [面试题 17.10. 主要元素](https://leetcode-cn.com/problems/find-majority-element-lcci/)

