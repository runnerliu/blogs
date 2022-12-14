## 题目列表

### 简单

#### 1. 两数之和

LeetCode地址: https://leetcode-cn.com/problems/two-sum/

```
class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        if not nums:
            return []
        usedNums = {}
        for index, num in enumerate(nums):
            v = target - num
            if v in usedNums:
                return [usedNums[v], index]
            else:
                usedNums[num] = index
        return []
```

#### 7. 整数反转

LeetCode地址：https://leetcode-cn.com/problems/reverse-integer/

```
class Solution:
    def reverse(self, x: int) -> int:
        if x > -10 and x < 10:
            return x

        x_s = str(x)
        if x_s[0] == '-':
            x_s = x_s[:0:-1]
            x_s = int(x_s)
            x_s = -x_s
        else:
            x_s = x_s[::-1]
            x_s = int(x_s)
        return x_s if x_s >= -2**31 and x_s <= (2**31 - 1) else 0
```

#### 20. 有效的括号

LeetCode地址: https://leetcode-cn.com/problems/valid-parentheses/

```
class Solution(object):
    def isValid(self, s):
        """
        :type s: str
        :rtype: bool
        """
        while '{}' in s or '()' in s or '[]' in s:
            s = s.replace('{}', '')
            s = s.replace('[]', '')
            s = s.replace('()', '')
        return s == ''
```

#### 21. 合并两个有序链表

LeetCode地址: https://leetcode-cn.com/problems/merge-two-sorted-lists/

```
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution:
    def mergeTwoLists(self, l1: ListNode, l2: ListNode) -> ListNode:
        if not l1: return l2
        if not l2: return l1
        if l1.val < l2.val:
            l1.next = self.mergeTwoLists(l1.next, l2)
            return l1
        else:
            l2.next = self.mergeTwoLists(l2.next, l1)
            return l2
```

#### 53. 最大子序和

LeetCode地址: https://leetcode-cn.com/problems/maximum-subarray/

```
class Solution(object):
    def maxSubArray(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        if not nums:
            return 0
        if len(nums) == 1:
            return nums[0]
        if len(nums) == 2:
            return max([nums[0], nums[1], sum(nums)])
        max_sum = nums[0]
        for i in range(len(nums)):
            tmp_max_sum = nums[i]
            if tmp_max_sum > max_sum:
                max_sum = tmp_max_sum
            for j in range(i + 1, len(nums)):
                tmp_max_sum += nums[j]
                if tmp_max_sum > max_sum:
                    max_sum = tmp_max_sum
        return max_sum

class Solution(object):
    def maxSubArray(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        if not nums:
            return 0
        if len(nums) == 1:
            return nums[0]
        if len(nums) == 2:
            return max([nums[0], nums[1], sum(nums)])
        max_sum = nums[0]
        tmp = max_sum
        for i in range(1, len(nums)):
            if tmp + nums[i] > nums[i]:
                max_sum = max(max_sum, tmp + nums[i])
                tmp += nums[i]
            else:
                max_sum = max(max_sum, tmp, nums[i])
                tmp = nums[i]
        return max_sum
```

#### 70. 爬楼梯

LeetCode地址: https://leetcode-cn.com/problems/climbing-stairs/

```
class Solution:
    def climbStairs(self, n: int) -> int:
        if n < 0:
            return 0
        if n in [0, 1]:
            return 1
        a = b = 1
        for _ in range(2, n + 1):
            a, b = b, a + b
        return b
```

#### 94. 二叉树的中序遍历

LeetCode地址: https://leetcode-cn.com/problems/binary-tree-inorder-traversal/

```
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution(object):
    def inorderTraversal(self, root):
        """
        :type root: TreeNode
        :rtype: List[int]
        """
        if not root:
            return []
        res = []

        def dfs(res, head):
            if head:
                dfs(res, head.left)
                res.append(head.val)
                dfs(res, head.right)
            return res

        dfs(res, root)
        return res
```

#### 101. 对称二叉树

LeetCode地址: https://leetcode-cn.com/problems/symmetric-tree/

```
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def isSymmetric(self, root: TreeNode) -> bool:
        if not root:
            return True
        
        def dfs(left, right):
            if left == None and right == None:
                return True
            elif left == None or right == None:
                return False
            else:
                return left.val == right.val and dfs(left.left, right.right) and dfs(left.right, right.left)


        return dfs(root.left, root.right)
```

#### 104. 二叉树的最大深度

LeetCode地址: https://leetcode-cn.com/problems/maximum-depth-of-binary-tree/

```
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution(object):
    def maxDepth(self, root):
        """
        :type root: TreeNode
        :rtype: int
        """
        if not root:
            return 0
        else:
            left_height = self.maxDepth(root.left)
            right_height = self.maxDepth(root.right)
            return max(left_height, right_height) + 1
```

#### 121. 买卖股票的最佳时机

LeetCode地址: https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock/

```
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        minprice = float('inf')
        lirui = 0
        for i in prices:
            minprice = min(i, minprice)
            lirui = max(lirui, i - minprice)
        return lirui
```

#### 136. 只出现一次的数字

LeetCode地址: https://leetcode-cn.com/problems/single-number/

```
class Solution:
    def singleNumber(self, nums: List[int]) -> int:
        tmp = {}
        for i in nums:
            if i not in tmp:
                tmp[i] = 1
            else:
                del tmp[i]
        return list(tmp.keys())[0]

class Solution:
    def singleNumber(self, nums: List[int]) -> int:
        if len(nums) == 1: return nums[0]

        res = nums[0]
        for i in range(1, len(nums)):
            res ^= nums[i]
        return res
```

#### 141. 环形链表

LeetCode地址: https://leetcode-cn.com/problems/linked-list-cycle/

```
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution:
    def hasCycle(self, head: ListNode) -> bool:
        if not head or not head.next:
            return False
        slow = fast = head
        while fast and fast.next:
            slow = slow.next
            fast = fast.next.next
            if slow == fast:
                return True
        return False
```

#### 155. 最小栈

LeetCode地址: https://leetcode-cn.com/problems/min-stack/

```
class MinStack:

    def __init__(self):
        """
        initialize your data structure here.
        """
        self.stack = []

    def push(self, val: int) -> None:
        """
        进栈时同时保存当前栈的最小值
        """
        if not self.stack:
            self.stack.append((val, val))
        else:
            self.stack.append((val, min(val, self.stack[-1][1])))

    def pop(self) -> None:
        self.stack.pop()

    def top(self) -> int:
        return self.stack[-1][0]

    def getMin(self) -> int:
        return self.stack[-1][1]
```

#### 160. 相交链表

LeetCode地址: https://leetcode-cn.com/problems/intersection-of-two-linked-lists/

```
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution:
    def getIntersectionNode(self, headA: ListNode, headB: ListNode) -> ListNode:
        A, B = headA, headB
        while A != B:
            A = A.next if A else headB
            B = B.next if B else headA
        return A
```

#### 169. 多数元素

LeetCode地址: https://leetcode-cn.com/problems/majority-element/

```
class Solution:
    def majorityElement(self, nums: List[int]) -> int:
        memo = {}
        for i in nums:
            if i not in memo:
                memo[i] = 1
            else:
                memo[i] += 1
        return max(memo, key=memo.get)
```

#### 172. 阶乘后的零

LeetCode地址: https://leetcode-cn.com/problems/factorial-trailing-zeroes/

```
class Solution:
    def trailingZeroes(self, n: int) -> int:
        count = 0
        for i in range(1, n + 1):
            if not i % 5:
                while not i % 5:
                    i //= 5
                    count += 1
        return count
```

#### 206. 反转链表

LeetCode地址: https://leetcode-cn.com/problems/reverse-linked-list/

```
class Solution(object):
    def reverseList(self, head):
        """
        :type head: ListNode
        :rtype: ListNode
        """
        if not head or not head.next:
            return head
        cur = head
        prev = None
        while cur:
            tmp = cur.next
            cur.next = prev
            prev = cur
            cur = tmp
        return prev
```

#### 226. 翻转二叉树

LeetCode地址: https://leetcode-cn.com/problems/invert-binary-tree/

```
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def invertTree(self, root: TreeNode) -> TreeNode:
        if not root:
            return root
        root.left, root.right = root.right, root.left
        self.invertTree(root.left)
        self.invertTree(root.right)
        return root
```

#### 234. 回文链表

LeetCode地址: https://leetcode-cn.com/problems/palindrome-linked-list/

```
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution:
    def isPalindrome(self, head: ListNode) -> bool:
        if not head:
            return False
        if not head.next:
            return True
        tmp = []
        while head:
            tmp.append(head.val)
            head = head.next
        return tmp == tmp[::-1]
```

#### 283. 移动零

LeetCode地址: https://leetcode-cn.com/problems/move-zeroes/

```
class Solution(object):
    def moveZeroes(self, nums):
        """
        :type nums: List[int]
        :rtype: None Do not return anything, modify nums in-place instead.
        """
        if not nums:
            return []
        if 0 not in nums:
            return nums
        for i in range(len(nums)):
            if nums[i] == 0:
                for j in range(i, len(nums)):
                    if nums[j] != 0:
                        nums[i], nums[j] = nums[j], nums[i]
                        break
        return nums

class Solution(object):
    def moveZeroes(self, nums):
        """
        :type nums: List[int]
        :rtype: None Do not return anything, modify nums in-place instead.
        """
        if not nums:
            return []
        if 0 not in nums:
            return nums
        count = i = 0
        while i < len(nums):
            if nums[i] == 0:
                del nums[i]
                count += 1
            else:
                i += 1
        nums.extend([0]*count)
        return nums
```

#### 448. 找到所有数组中消失的数字

LeetCode地址: https://leetcode-cn.com/problems/find-all-numbers-disappeared-in-an-array/

```
class Solution:
    def findDisappearedNumbers(self, nums: List[int]) -> List[int]:
        if not nums:
            return []
        return list(set(range(1, len(nums) + 1)).difference(set(nums)))
```

#### 461. 汉明距离

LeetCode地址: https://leetcode-cn.com/problems/hamming-distance/

```
class Solution:
    def hammingDistance(self, x: int, y: int) -> int:
        n = x ^ y
        c = 0
        while n:
            if n & 1: c += 1
            n >>= 1
        return c
```

#### 543. 二叉树的直径

LeetCode地址: https://leetcode-cn.com/problems/diameter-of-binary-tree/

```
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def diameterOfBinaryTree(self, root: TreeNode) -> int:
        self.ans = 1

        def dfs(head):
            if not head: return 0
            l = dfs(head.left)
            r = dfs(head.right)
            self.ans = max(self.ans, l + r + 1)
            return max(l, r) + 1

        dfs(root)
        return self.ans - 1
```

#### 559. N 叉树的最大深度

LeetCode地址: https://leetcode-cn.com/problems/maximum-depth-of-n-ary-tree/

```
"""
# Definition for a Node.
class Node:
    def __init__(self, val=None, children=None):
        self.val = val
        self.children = children
"""

class Solution:
    def maxDepth(self, root: 'Node') -> int:
        if not root:
            return 0
        d = 1
        for i in root.children:
            d = max(d, self.maxDepth(i) + 1)
        return d
```

#### 589. N 叉树的前序遍历

LeetCode地址: https://leetcode-cn.com/problems/n-ary-tree-preorder-traversal/

```
"""
# Definition for a Node.
class Node:
    def __init__(self, val=None, children=None):
        self.val = val
        self.children = children
"""

class Solution:
    def preorder(self, root: 'Node') -> List[int]:
        if not root:
            return []
        res = []
        
        def dfs(head):
            res.append(head.val)
            for c in head.children:
                dfs(c)

        dfs(root)
        return res
```

#### 590. N 叉树的后序遍历

LeetCode地址: https://leetcode-cn.com/problems/n-ary-tree-postorder-traversal/

```
"""
# Definition for a Node.
class Node:
    def __init__(self, val=None, children=None):
        self.val = val
        self.children = children
"""

class Solution:
    def postorder(self, root: 'Node') -> List[int]:
        if not root:
            return []
        res = []

        def dfs(head):
            for c in head.children:
                dfs(c)
            res.append(head.val)
        
        dfs(root)
        return res
```

#### 617. 合并二叉树

LeetCode地址: https://leetcode-cn.com/problems/merge-two-binary-trees/

```
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def mergeTrees(self, root1: TreeNode, root2: TreeNode) -> TreeNode:
        if not root1:
            return root2
        if not root2:
            return root1
        newTree = TreeNode(root1.val + root2.val)
        newTree.left = self.mergeTrees(root1.left, root2.left)
        newTree.right = self.mergeTrees(root1.right, root2.right)
        return newTree
```

### 中等

#### 2. 两数相加

LeetCode地址: https://leetcode-cn.com/problems/add-two-numbers/

```
# Definition for singly-linked list.
# class ListNode(object):
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution(object):
    def addTwoNumbers(self, l1, l2):
        """
        :type l1: ListNode
        :type l2: ListNode
        :rtype: ListNode
        """
        head = ListNode(l1.val + l2.val)
        cur = head
        while l1.next or l2.next:
            l1 = l1.next if l1.next else ListNode()
            l2 = l2.next if l2.next else ListNode()
            cur.next = ListNode(l1.val + l2.val + cur.val // 10)
            cur.val = cur.val % 10
            cur = cur.next
        if cur.val >= 10:
            cur.next = ListNode(cur.val // 10)
            cur.val = cur.val % 10
        return head
```

#### 3. 无重复字符的最长子串

LeetCode地址: https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/

```
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        if not s:
            return 0
        start = max_len = 0
        chars = {}
        for index, i in enumerate(s):
            if i in chars and start <= chars[i]:
                start = chars[i] + 1
            else:
                max_len = max(max_len, index - start + 1)
            chars[i] = index
        return max_len
```

#### 5. 最长回文子串

LeetCode地址: https://leetcode-cn.com/problems/longest-palindromic-substring/

```
class Solution:
    def longestPalindrome(self, s: str) -> str:
        size = len(s)
        if size == 1:
            return s
        dp = [[False] * size for _ in range(size)]
        max_len, start = 1, 0
        for j in range(1, size):
            for i in range(j):
                if j - i <= 2:
                    if s[i] == s[j]:
                        dp[i][j] = True
                        cur_len = j - i + 1
                else:
                    if s[i] == s[j] and dp[i + 1][j - 1]:
                        dp[i][j] = True
                        cur_len = j - i + 1
                
                if dp[i][j]:
                    if cur_len > max_len:
                        max_len = cur_len
                        start = i
        return s[start:start + max_len]
```

#### 11. 盛最多水的容器

LeetCode地址: https://leetcode-cn.com/problems/container-with-most-water/

```
class Solution:
    def maxArea(self, height: List[int]) -> int:
        if not height:
            return 0
        i, j = 0, len(height) - 1
        res = 0
        while i < j:
            if height[i] >= height[j]:
                h, w = height[j], j - i
                j -= 1
            else:
                h, w = height[i], j - i
                i += 1
            res = max(res, h * w)
        return res
```

#### 15. 三数之和

LeetCode地址: https://leetcode-cn.com/problems/3sum/

```
class Solution:
    def threeSum(self, nums: List[int]) -> List[List[int]]:
        l = len(nums)
        if l <= 2:
            return []
        nums.sort()
        res, k = [], 0
        for k in range(len(nums) - 2):
            if nums[k] > 0:
                break
            if k > 0 and nums[k] == nums[k - 1]:
                continue
            i, j = k + 1, len(nums) - 1
            while i < j:
                s = nums[k] + nums[i] + nums[j]
                if s < 0:
                    i += 1
                    while i < j and nums[i] == nums[i - 1]:
                        i += 1
                elif s > 0:
                    j -= 1
                    while i < j and nums[j] == nums[j + 1]:
                        j -= 1
                else:
                    res.append([nums[k], nums[i], nums[j]])
                    i += 1
                    j -= 1
                    while i < j and nums[i] == nums[i - 1]:
                        i += 1
                    while i < j and nums[j] == nums[j + 1]:
                        j -= 1
        return res
```

#### 17. 电话号码的字母组合

LeetCode地址: https://leetcode-cn.com/problems/letter-combinations-of-a-phone-number/

```
class Solution:
    def letterCombinations(self, digits: str) -> List[str]:
        if not digits:
            return []
        chars = {'2':'abc', '3':'def', '4':'ghi', '5':'jkl', '6':'mno', '7':'pqrs', '8':'tuv', '9':'wxyz'}
        product = ['']
        for i in digits:
            product = [m + n for m in product for n in chars[i]]
        return product
```

#### 19. 删除链表的倒数第 N 个结点

LeetCode地址: https://leetcode-cn.com/problems/remove-nth-node-from-end-of-list/

```
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution:
    def removeNthFromEnd(self, head: ListNode, n: int) -> ListNode:
        if not head:
            return None
        left = right = head
        count = 0
        while count < n:
            right = right.next
            count += 1
        if not right:
            return head.next
        while right.next:
            left = left.next
            right = right.next
        if left.next:
            left.next = left.next.next
        return head
```

#### 22. 括号生成

LeetCode地址: https://leetcode-cn.com/problems/generate-parentheses/

```
class Solution:
    def generateParenthesis(self, n: int):
        dp = [[] for _ in range(n+1)]
        dp[0] = [""]
        for i in range(1, n + 1):
            for p in range(i):
                for k1 in dp[p]:
                    for k2 in dp[i - 1 - p]:
                        dp[i].append("({0}){1}".format(k1, k2))
        return dp[n]
```

#### 31. 下一个排列

LeetCode地址: https://leetcode-cn.com/problems/next-permutation/

```
class Solution:
    def nextPermutation(self, nums: List[int]) -> None:
        for i in range(len(nums)-1, 0, -1):
            if nums[i-1] < nums[i]:
                for j in range(len(nums)-1, i-1, -1):
                    if nums[j] > nums[i-1]:
                        nums[i-1], nums[j] = nums[j], nums[i-1]
                        break
                for j in range((len(nums)-i+1)//2):
                    nums[i+j], nums[len(nums)-1-j] = nums[len(nums)-1-j], nums[i+j]
                return nums
        nums.reverse()
        return nums
```

#### 33. 搜索旋转排序数组

LeetCode地址: https://leetcode-cn.com/problems/search-in-rotated-sorted-array/

```
class Solution:
    def search(self, nums: List[int], target: int) -> int:
        if not nums:
            return -1
        l, r = 0, len(nums) - 1
        while l <= r:
            mid = (l + r) // 2
            if nums[mid] == target:
                return mid
            if nums[0] <= nums[mid]:
                if nums[0] <= target < nums[mid]:
                    r = mid - 1
                else:
                    l = mid + 1
            if nums[0] > nums[mid]:
                if nums[mid] < target <= nums[len(nums) - 1]:
                    l = mid + 1
                else:
                    r = mid - 1
        return -1
```

#### 34. 在排序数组中查找元素的第一个和最后一个位置

LeetCode地址: https://leetcode-cn.com/problems/find-first-and-last-position-of-element-in-sorted-array/

```
class Solution:
    def searchRange(self, nums: List[int], target: int) -> List[int]:
        l = len(nums)
        if not l:
            return [-1, -1]
        if l == 1 and target == nums[0]:
            return [0, 0]
        left = right = -1
        for i in range(l):
            if nums[i] != target:
                continue
            if left == -1:
                left = i
                continue
            if right == -1 or right < i:
                right = i
        if left != -1 and right == -1:
            right = left
        return [left, right]
```

#### 39. 组合总和

LeetCode地址: https://leetcode-cn.com/problems/combination-sum/

```
class Solution:
    def combinationSum(self, candidates: List[int], target: int) -> List[List[int]]:
        res = []
        if not candidates:
            return res
        
        def combination(candidates,target,res_list):
            if target < 0:
                return
            if target == 0:
                res.append(res_list)
            for i,c in enumerate(candidates):
                combination(candidates[i:],target-c,res_list+[c])
        combination(candidates,target,[])
        
        return res
```

#### 46. 全排列

LeetCode地址: https://leetcode-cn.com/problems/permutations/

```
class Solution:
    def permute(self, nums: List[int]) -> List[List[int]]:
        res = []
        def backtrack(nums, tmp):
            if not nums:
                res.append(tmp)
                return 
            for i in range(len(nums)):
                backtrack(nums[:i] + nums[i+1:], tmp + [nums[i]])
        backtrack(nums, [])
        return res
```

#### 48. 旋转图像

LeetCode地址: https://leetcode-cn.com/problems/rotate-image/

```
class Solution:
    def rotate(self, matrix: List[List[int]]) -> None:
        """
        Do not return anything, modify matrix in-place instead.
        """
        pos1, pos2 = 0, len(matrix) - 1
        while pos1 < pos2:
            add = 0
            while add < (pos2 - pos1):
                temp = matrix[pos2-add][pos1]
                matrix[pos2-add][pos1] = matrix[pos2][pos2-add]
                matrix[pos2][pos2-add] = matrix[pos1+add][pos2]
                matrix[pos1+add][pos2] = matrix[pos1][pos1+add]
                matrix[pos1][pos1+add] = temp
                add += 1
            pos1 += 1
            pos2 -= 1
```

#### 49. 字母异位词分组

LeetCode地址: https://leetcode-cn.com/problems/group-anagrams/

```
class Solution:
    def groupAnagrams(self, strs: List[str]) -> List[List[str]]:
        r = {}
        for i in strs:
            s = ''.join(sorted(i))
            if s not in r:
                r[s] = [i]
            else:
                r[s].append(i)
        return [v for v in r.values()]
```

#### 50. Pow(x, n)

LeetCode地址: https://leetcode-cn.com/problems/powx-n/

```
class Solution:
    def myPow(self, x: float, n: int) -> float:
        if x == 0.0: return 0.0
        res = 1
        if n < 0: x, n = 1 / x, -n
        while n:
            if n & 1: res *= x
            x *= x
            n >>= 1
        return res

class Solution:
    def myPow(self, x: float, n: int) -> float:
        if n >= 0:
            return self.quick_pow(x, n)
        else:
            return 1.0 / self.quick_pow(x, -n)

    def quick_pow(self, a: float, n: int) -> float:
        res = 1.0
        while n > 0:
            if n & 1 == 1:
                res *= a
            a *= a
            n //= 2
        return res
```

#### 55. 跳跃游戏

LeetCode地址: https://leetcode-cn.com/problems/jump-game/

```
class Solution:
    def canJump(self, nums: List[int]) -> bool:
        max_i = 0
        for i, jump in enumerate(nums):
            if max_i >= i and i + jump > max_i:  
                max_i = i + jump
        return max_i >= i
```

#### 56. 合并区间

LeetCode地址: https://leetcode-cn.com/problems/merge-intervals/

```
class Solution:
    def merge(self, intervals: List[List[int]]) -> List[List[int]]:
        if not intervals:
            return []
        intervals.sort()
        res = [intervals[0]]
        for x, y in intervals[1:]:
            if res[-1][1] < x:
                res.append([x, y])
            else:
                res[-1][1] = max(y, res[-1][1])
        return res
```

#### 62. 不同路径

LeetCode地址: https://leetcode-cn.com/problems/unique-paths/

#### 64. 最小路径和

LeetCode地址: https://leetcode-cn.com/problems/minimum-path-sum/

#### 75. 颜色分类

LeetCode地址: https://leetcode-cn.com/problems/sort-colors/

#### 78. 子集

LeetCode地址: https://leetcode-cn.com/problems/subsets/

#### 79. 单词搜索

LeetCode地址: https://leetcode-cn.com/problems/word-search/

#### 96. 不同的二叉搜索树

LeetCode地址: https://leetcode-cn.com/problems/unique-binary-search-trees/

#### 98. 验证二叉搜索树

LeetCode地址: https://leetcode-cn.com/problems/validate-binary-search-tree/

#### 102. 二叉树的层序遍历

LeetCode地址: https://leetcode-cn.com/problems/binary-tree-level-order-traversal/

```
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def levelOrder(self, root: TreeNode) -> List[List[int]]:
        if not root:
            return []
        
        def dfs(res, head, cur):
            if head:
                cur += 1
                if cur not in res:
                    res[cur] = []
                dfs(res, head.left, cur)
                res[cur].append(head.val)
                dfs(res, head.right, cur)
                
            return res
        
        res = {}
        dfs(res, root, 0)
        return list(res.values())
```

#### 103. 二叉树的锯齿形层序遍历

LeetCode地址: https://leetcode-cn.com/problems/Binary-Tree-Zigzag-Level-Order-Traversal/

```
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def zigzagLevelOrder(self, root: TreeNode) -> List[List[int]]:
        if not root:
            return []
        
        def dfs(head, res, cur):
            if head:
                cur += 1
                if cur not in res:
                    res[cur] = []
                dfs(head.left, res, cur)
                res[cur].append(head.val)
                dfs(head.right, res, cur)
            return res

        res = {}
        dfs(root, res, 0)
        for k, v in res.items():
            if k % 2 == 0:
                res[k] = v[::-1]
        return list(res.values())
        
```

#### 105. 从前序与中序遍历序列构造二叉树

LeetCode地址: https://leetcode-cn.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/

#### 113. 路径总和 II

LeetCode地址：https://leetcode-cn.com/problems/path-sum-ii/

```
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def pathSum(self, root: TreeNode, targetSum: int) -> List[List[int]]:
        def DFS(root, targetSum):
            if not root:
                return 
            res.append(root.val)
            targetSum -= root.val
            if not root.left and not root.right:
                if targetSum == 0:
                    ress.append(res[:])
            DFS(root.left, targetSum)
            DFS(root.right, targetSum)
            res.pop()

        ress = []
        res = []
        DFS(root, targetSum)
        return ress
```

#### 114. 二叉树展开为链表

LeetCode地址: https://leetcode-cn.com/problems/flatten-binary-tree-to-linked-list/

#### 128. 最长连续序列

LeetCode地址: https://leetcode-cn.com/problems/longest-consecutive-sequence/

#### 139. 单词拆分

LeetCode地址: https://leetcode-cn.com/problems/word-break/

#### 142. 环形链表 II

LeetCode地址: https://leetcode-cn.com/problems/linked-list-cycle-ii/

#### 146. LRU 缓存机制

LeetCode地址: https://leetcode-cn.com/problems/lru-cache/

```
class LRUCache(object):

    def __init__(self, cap):
        self.cap = cap
        self.cache = {}
        self.keys = []

    def get(self, key):
        if key in self.cache:
            value = self.cache[key]
            self.keys.remove(key)
            self.keys.insert(0, key)
        else:
            value = -1
        return value

    def put(self, key, value):
        if key in self.cache:
            self.keys.remove(key)
        elif len(self.keys) == self.cap:
            old_key = self.keys.pop()
            del self.cache[old_key]
        self.keys.insert(0, key)
        self.cache[key] = value


class LinkedNode:
    def __init__(self, key, val):
        self.key = key
        self.val = val
        self.prev = None
        self.next = None

class LRUCache:
    def __init__(self, capacity: int):
        self.cache = {}
        self.head = LinkedNode(0, 0)
        self.tail = LinkedNode(0, 0)
        self.head.next = self.tail
        self.tail.prev = self.head
        self.capacity = capacity
        self.size = 0

    def get(self, key: int) -> int:
        if key not in self.cache: return -1
        node = self.cache[key]
        self.move_to_head(node)
        return node.val

    def put(self, key: int, value: int) -> None:
        if key not in self.cache:
            node = LinkedNode(key, value)
            self.cache[key] = node
            self.add_head(node)
            self.size += 1
            if self.size > self.capacity:
                cur = self.remove_tail()
                self.cache.pop(cur.key)
                self.size -= 1
        else:
            node = self.cache[key]
            node.val = value
            self.move_to_head(node)

    def add_head(self, node):
        node.prev = self.head
        node.next = self.head.next
        self.head.next.prev = node
        self.head.next = node
    
    def remove(self, node):
        node.prev.next = node.next
        node.next.prev = node.prev
    
    def move_to_head(self, node):
        self.remove(node)
        self.add_head(node)
    
    def remove_tail(self):
        node = self.tail.prev
        self.remove(node)
        return node
```

#### 148. 排序链表

LeetCode地址: https://leetcode-cn.com/problems/sort-list/

#### 152. 乘积最大子数组

LeetCode地址: https://leetcode-cn.com/problems/maximum-product-subarray/

#### 198. 打家劫舍

LeetCode地址: https://leetcode-cn.com/problems/house-robber/

#### 200. 岛屿数量

LeetCode地址: https://leetcode-cn.com/problems/number-of-islands/

#### 207. 课程表

LeetCode地址: https://leetcode-cn.com/problems/course-schedule/

#### 208. 实现 Trie (前缀树)

LeetCode地址: https://leetcode-cn.com/problems/implement-trie-prefix-tree/

#### 215. 数组中的第K个最大元素

LeetCode地址: https://leetcode-cn.com/problems/kth-largest-element-in-an-array/

```
def partition(nums, left, right):
    key = nums[left]
    i, j = left, right
    while i < j:
        while i < j and nums[j] >= key:
            j -= 1
        nums[i] = nums[j]
        while i < j and nums[i] < key:
            i += 1
        nums[j] = nums[i]
    nums[i] = key
    return i


def quick_sort(nums, left, right):
    if left < right:
        index = partition(nums, left, right)
        quick_sort(nums, left, index - 1)
        quick_sort(nums, index + 1, right)

def topk_split(nums, k, left, right):
    if left < right:
        index = partition(nums, left, right)
        if index == k:
            return
        elif index < k:
            topk_split(nums, k, index + 1, right)
        else:
            topk_split(nums, k, left, index - 1)

# 第k大
def k_max(nums, k):
    topk_split(nums, len(nums) - k, 0, len(nums) - 1)
    return nums[len(nums) - k]

# topk
def topk(nums, k):
    res = nums[:k]
    quick_sort(res, 0, len(res) - 1)
    for i in range(k, len(nums)):
        if nums[i] > res[0]:
            for j in range(len(res)):
                if j == len(res) - 1:
                    res.append(nums[i])
                    res.pop(0)
                elif res[j] < nums[i] <= res[j + 1]:
                    res.insert(j + 1, nums[i])
                    res.pop(0)
                    break
    return res[::-1]

# 最大值
def max_nums(nums):
    if not nums:
        return -1
    m = nums[0]
    for i in nums:
        if i > m:
            m = i
    return m
```

#### 221. 最大正方形

LeetCode地址: https://leetcode-cn.com/problems/maximal-square/

#### 236. 二叉树的最近公共祖先

LeetCode地址: https://leetcode-cn.com/problems/lowest-common-ancestor-of-a-binary-tree/

#### 238. 除自身以外数组的乘积

LeetCode地址: https://leetcode-cn.com/problems/product-of-array-except-self/

#### 240. 搜索二维矩阵 II

LeetCode地址: https://leetcode-cn.com/problems/search-a-2d-matrix-ii/

#### 279. 完全平方数

LeetCode地址: https://leetcode-cn.com/problems/perfect-squares/

#### 287. 寻找重复数

LeetCode地址: https://leetcode-cn.com/problems/find-the-duplicate-number/

#### 300. 最长递增子序列

LeetCode地址: https://leetcode-cn.com/problems/longest-increasing-subsequence/

#### 309. 最佳买卖股票时机含冷冻期

LeetCode地址: https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/

#### 322. 零钱兑换

LeetCode地址: https://leetcode-cn.com/problems/coin-change/

#### 337. 打家劫舍 III

LeetCode地址: https://leetcode-cn.com/problems/house-robber-iii/

#### 347. 前 K 个高频元素

LeetCode地址: https://leetcode-cn.com/problems/top-k-frequent-elements/

#### 394. 字符串解码

LeetCode地址: https://leetcode-cn.com/problems/decode-string/

#### 399. 除法求值

LeetCode地址: https://leetcode-cn.com/problems/evaluate-division/

#### 406. 根据身高重建队列

LeetCode地址: https://leetcode-cn.com/problems/queue-reconstruction-by-height/

#### 416. 分割等和子集

LeetCode地址: https://leetcode-cn.com/problems/partition-equal-subset-sum/

#### 429. N 叉树的层序遍历

LeetCode地址: https://leetcode-cn.com/problems/n-ary-tree-level-order-traversal/

```
"""
# Definition for a Node.
class Node:
    def __init__(self, val=None, children=None):
        self.val = val
        self.children = children
"""

class Solution:
    def levelOrder(self, root: 'Node') -> List[List[int]]:
        res = []
        if not root:
            return res
        q = [root]
        while q:
            l = len(q)
            sub = []
            for i in range(l):
                node = q.pop(0)
                sub.append(node.val)
                if node.children:
                    q.extend(node.children)
            res.append(sub)
        return res
```

#### 437. 路径总和 III

LeetCode地址: https://leetcode-cn.com/problems/path-sum-iii/

#### 438. 找到字符串中所有字母异位词

LeetCode地址: https://leetcode-cn.com/problems/find-all-anagrams-in-a-string/

#### 470. 用 Rand7() 实现 Rand10()

LeetCode地址: https://leetcode-cn.com/problems/implement-rand10-using-rand7/

```
# The rand7() API is already defined for you.
# def rand7():
# @return a random integer in the range 1 to 7

class Solution:
    def rand10(self):
        """
        :rtype: int
        """
        num = (rand7() - 1) * 7 + rand7()
        if num > 40: return self.rand10()
        return num % 10 + 1

利用(randX() - 1) * X + randX() 可以等概率的生成[1, X * X]范围的随机数
```

#### 494. 目标和

LeetCode地址: https://leetcode-cn.com/problems/target-sum/

#### 538. 把二叉搜索树转换为累加树

LeetCode地址: https://leetcode-cn.com/problems/convert-bst-to-greater-tree/

#### 560. 和为K的子数组

LeetCode地址: https://leetcode-cn.com/problems/subarray-sum-equals-k/

#### 581. 最短无序连续子数组

LeetCode地址: https://leetcode-cn.com/problems/shortest-unsorted-continuous-subarray/

#### 621. 任务调度器

LeetCode地址: https://leetcode-cn.com/problems/task-scheduler/

#### 647. 回文子串

LeetCode地址: https://leetcode-cn.com/problems/palindromic-substrings/

```
class Solution:
    def countSubstrings(self, s: str) -> int:
        ans = 0
        n = len(s)
        dp = [[False] * n for _ in range(n)]
        ans = 0
        for k in range(n):
            for i in range(n - k):
                j = i + k
                if k == 0: 
                    dp[i][j] = True
                elif k == 1:
                    dp[i][j] = s[i] == s[j]
                else:
                    dp[i][j] = dp[i + 1][j - 1] and s[i] == s[j]
                
                if dp[i][j]: 
                    ans+=1
        return ans
```

#### 739. 每日温度

LeetCode地址: https://leetcode-cn.com/problems/daily-temperatures/

#### 1143. 最长公共子序列

LeetCode地址: https://leetcode-cn.com/problems/longest-common-subsequence/

```
class Solution:
    def longestCommonSubsequence(self, text1: str, text2: str) -> int:
        if not text1 or not text2:
            return 0
        m, n = len(text1), len(text2)
        dp = [[0] * (n + 1) for _ in range(m + 1)]
        for i in range(1, m + 1):
            for j in range(1, n + 1):
                if text1[i - 1] == text2[j - 1]:
                    dp[i][j] = dp[i - 1][j - 1] + 1
                else:
                    dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])
        return dp[m][n]
```

最长公共子串

```
def longestCommonSubsequence(text1, text2) -> (str, int):
    if not text1 or not text2:
        return None, 0
    m, n = len(text1), len(text2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    end = max_len = 0
    for i in range(len(text1)):
        for j in range(len(text2)):
            if text1[i] == text2[j]:
                dp[i + 1][j + 1] = dp[i][j] + 1
                if dp[i + 1][j + 1] > max_len:
                    max_len = dp[i + 1][j + 1]
                    end = i + 1
    return text1[end - max_len:end], max_len
```

#### 1254. 统计封闭岛屿的数目

LeetCode地址: https://leetcode-cn.com/problems/number-of-closed-islands/

```
class Solution:
    def closedIsland(self, grid: List[List[int]]) -> int:
        if len(grid) == 0:
            return 0
        rows, cols = len(grid), len(grid[0])

        def dfs(x, y):
            if grid[x][y] == 1:
                return
            grid[x][y] = 1
            for xx, yy in [(x-1, y), (x, y+1), (x+1, y), (x, y-1)]:
                if 0 <= xx < rows and 0 <= yy < cols:
                    dfs(xx, yy)

        for i in range(rows):
            dfs(i, 0)
            dfs(i, cols - 1)
        
        for j in range(cols):
            dfs(0, j)
            dfs(rows - 1, j)

        count = 0
        for i in range(rows):
            for j in range(cols):
                if grid[i][j] == 0:
                    dfs(i, j)
                    count += 1
        
        return count
```

### 困难

#### 4. 寻找两个正序数组的中位数

LeetCode地址: https://leetcode-cn.com/problems/median-of-two-sorted-arrays/

#### 10. 正则表达式匹配

LeetCode地址: https://leetcode-cn.com/problems/regular-expression-matching/

#### 23. 合并K个升序链表

LeetCode地址: https://leetcode-cn.com/problems/merge-k-sorted-lists/

#### 32. 最长有效括号

LeetCode地址: https://leetcode-cn.com/problems/longest-valid-parentheses/

```
class Solution:
    def longestValidParentheses(self, s: str) -> int:
        if not s: return 0
        stack, l = [], 0
        for i in range(len(s)):
            if not stack or s[i] == '(' or s[stack[-1]] == ')':
                stack.append(i)
            else:
                stack.pop()
                l = max(l, i - (stack[-1] if stack else -1))
        return l
```

#### 42. 接雨水

LeetCode地址: https://leetcode-cn.com/problems/trapping-rain-water/

#### 72. 编辑距离

LeetCode地址: https://leetcode-cn.com/problems/edit-distance/

#### 76. 最小覆盖子串

LeetCode地址: https://leetcode-cn.com/problems/minimum-window-substring/

#### 84. 柱状图中最大的矩形

LeetCode地址: https://leetcode-cn.com/problems/largest-rectangle-in-histogram/

#### 85. 最大矩形

LeetCode地址: https://leetcode-cn.com/problems/maximal-rectangle/

#### 124. 二叉树中的最大路径和

LeetCode地址: https://leetcode-cn.com/problems/binary-tree-maximum-path-sum/

#### 128. 最长连续序列

LeetCode地址: https://leetcode-cn.com/problems/longest-consecutive-sequence/

#### 239. 滑动窗口最大值

LeetCode地址: https://leetcode-cn.com/problems/sliding-window-maximum/

#### 297. 二叉树的序列化与反序列化

LeetCode地址: https://leetcode-cn.com/problems/serialize-and-deserialize-binary-tree/

#### 301. 删除无效的括号

LeetCode地址: https://leetcode-cn.com/problems/remove-invalid-parentheses/

#### 312. 戳气球

LeetCode地址: https://leetcode-cn.com/problems/burst-balloons/


### 其他

#### 1. 逆序字符串

str_1 = "helloworld"，str_2 = "oworldhell"，func(str_1, str_2)  ->  true / false

```
def reverse(str1, str2) -> bool:
    if not str2 or not str2:
        return False
    key = str2[0]
    l1 = len(str1)
    l2 = len(str2)
    for i in range(l1):
        if str1[i] == key:
            if str1[i:] == str2[:(l1 - i)] and str1[:i] == str2[(l1 - i):]:
                return True
    return False
```

#### 2. 合并N个有序数组

```
import heapq
from collections import deque

def list_merge(*lists):
    #入参判断, 这里直接pass
    #将所有链表转化为deque,方便使用popleft获取链表的最左元素及根据索引返回该索引对应的剩余链表
    queues = [queue for queue in map(deque, lists)]
    heap = []
    #初始化链表,该链表中的元素为元组, 各个链表的第一个元素及链表所在索引
    for i, lst in enumerate(queues):
        heap.append((lst.popleft(), i))
    #将链表转换成最小堆
    heapq.heapify(heap)
    #链表: 用于存放每次获取的堆顶层元素
    result = []
    
    while heap:
        #将堆顶层元素出堆
        value, index = heapq.heappop(heap)
        #将顶层元素追加
        result.append(value)
        #根据索引获取对应链表的剩余元素
        if queues[index]:
             #如果存在下一个元素,则将该元素及索引入堆
            heapq.heappush(heap, (queues[index].popleft(), index))
    return result
```

#### 3. json字符串解析

```
{
    "@id": "xxx", 
    "@type": ["Person"], 
    "name": "刘德华", 
    "works": [
        {
            "@value": "无间道", 
            "@id": "xxx"
        }, 
        {
            "@value": "无间道2", 
            "@id": "xxx"
        }
    ], 
    "award": {
        "awardCeremony": "xxx颁奖典礼", 
        "awardName": "最佳演员"
    }
}

输出：
@id, xxx
@type, Person
name, 刘德华
works.@value, 无间道
works.@id, xxx
works.@value, 无间道2
works.@id, xxx
award.awardCeremony, xxx颁奖典礼
award.awardName, 最佳演员

def print_json(data, parent=None):
    if isinstance(data, dict):
        for k in data.keys():
            v = data[k]
            if isinstance(v, dict):
                print_json(v, k)
            elif isinstance(v, list):
                for v_data in v:
                    print_json(v_data, k)
            else:
                if not parent:
                    print('{}, {}'.format(k, v))
                else:
                    print('{}.{}, {}'.format(parent, k, v))
    elif isinstance(data, list):
        for i in data:
            print_json(i)
    elif isinstance(data, str):
        print('{}, {}'.format(parent, data))
```