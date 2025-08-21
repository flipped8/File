### [LeetCode42 接雨水](https://leetcode.cn/problems/trapping-rain-water/description/?envType=study-plan-v2&envId=top-100-liked)

#### 解法一

​	动态规划：使用两个数组分别存储某个下标左右的最大值，并取最小值即可找到当前下标可存储的的水的最大容量

#### 解法二

​	双指针：维护两个指针 left 和 right，以及两个变量 leftMax 和 rightMax，初始时 left=0,right=n−1,leftMax=0,rightMax=0。指针 left 只会向右移动，指针 right 只会向左移动，在移动指针的过程中维护两个变量 leftMax 和 rightMax 的值。
当两个指针没有相遇时，进行如下操作：
​	使用height[left]和height[right]的值更新leftMax和rightMax的值。
​	如果leftMax<rightMax，则下标left处能接的雨水量为leftMax-height[left],然后left+1。
​	如果leftMax>=rightMax，则下标right处能接到的雨水量为rightMax-height[right]，然后right-1。

算法原理：首先leftMax<rightMax时，left下标处能容纳的雨水数量为leftMax-height[left]这个比较容易理解。核心在于双指针的移动，