- [排序算法](#排序算法)
  - [快排](#快排)
  - [归并排序](#归并排序)
- [二分查找](#二分查找)
- [二叉树](#二叉树)
- [双指针](#双指针)
- [hash算法](#hash算法)
- [动态贪心回溯适用场景](#动态贪心回溯适用场景)
- [动态规划](#动态规划)
- [回溯算法](#回溯算法)
- [贪心算法](#贪心算法)


### 排序算法
#### 快排
##### 思想
分治思想，将待排数组中随机选择一个元素作为`pivot`(分区点),然后再将数组遍历一遍，将小于分区点`pivot`的值放在分区点的左边，将大于分区点值放在分区点的右边，将`pivot`放在中间。将其分成三部分，然后再将左右边分别进行排序，直到最终排序成功。
##### 程序逻辑
1. 分区`partition`
2. 遍历待排序的数组，将小于值放到分区点左边，大于值放在分区点右边，交换
3. 再对左边数组排序
4. 再对右边数组排序
##### 代码
```python
def quick_sort(nums, left, right):
    # 1.结束条件 为什么会有大于情况，因为可能分区点的值在遍历后位于最后一位
    if left >= right:
        return
    # 2.选择分区 以最后一个值为分区
    partition_index = right
    # 3.遍历数组 将小于分区值的放在左边，大于的放在右边
    for i in range(left, right):
        if i < partition_index and nums[i] > nums[partition_index]:
            nums[i], nums[partition_index] = nums[partition_index], nums[i]
            partition_index = i
        elif i > partition_index and nums[i] < nums[partition_index]:
            nums[i], nums[partition_index] = nums[partition_index], nums[i]
            partition_index = i
    # 4.分别对分区点两边进行排序
    quick_sort(nums, left, partition_index - 1)
    quick_sort(nums, partition_index + 1, right)
    return nums

a = [1, 2, 12, 4, 7, 3, 10]
print(quick_sort(a, 0, len(a) - 1))
```
##### 复杂度
时间复杂度：最坏的情况`O(n2)`,比如数组本身就有序了，平均是`n*logn`

空间复杂度：
`O(h)`,其中 h为快速排序递归调用的层数

#### 归并排序
##### 思想
分治思想，将待排序的数组从中间进行分解，形成两个长度为`n/2`(二分)的子数组，然后将其排成一个有序数组，然后再将这两个有序数组进行合并
##### 程序逻辑
1. 递归结束条件 长度为1 返回数组
2. 中间值
3. 左边子数组排序
4. 右边子数组排序
5. 合并两个有序子序列
##### 代码
```python
def merge_sort(nums):
    # 1. 结束条件，这里与快排不一样,这里的值跟mid的值是否越界有关系
    if len(nums) <= 1:
        return nums
    # 2. 中间index
    mid = len(nums) // 2
    # 3. 分别对两部分数组进行排序
    left = merge_sort(nums[:mid])
    right = merge_sort(nums[mid:])
    # 4. 进行合并 采用双指针
    temp = []
    l_index = 0
    r_index = 0
    while l_index < len(left) and r_index < len(right):
        if left[l_index] >= right[r_index]:
            temp.append(right[r_index])
            r_index = r_index + 1
        else:
            temp.append(left[l_index])
            l_index = l_index + 1
    if l_index == len(left):
        temp.extend(right[r_index:])
    else:
        temp.extend(left[l_index:])
    return temp
a = [2,1,3,8,7]
print(merge_sort(a))
```
##### 复杂度
时间复杂度：`nlogn`

空间复杂度：需要额外的`O(n)`

#### 二分查找
##### 思想
对半查找，取有序数组的中间值跟目标值进行比较，如果小于目标值，则在目标值一定在右边数组，如果大于目标值，则目标值则一定在左边数组，依次取中间值，直到找到目标值。
##### 程序逻辑
1. 给左右指针赋予初始值
2. 求中间值
3. 进行比较
##### 代码
```python
def find(nums, target):
    left = 0
    right = len(nums) - 1
    while left <= right:
        mid = (left + right) // 2
        if nums[mid] == target:
            return mid
        elif nums[mid] > target:
            right = mid - 1
        else:
            left = mid + 1
    return
```
##### 复杂度
时间复杂度：`logn`

#### 二叉树
```python
def traverse(TreeNode root) {
    # 1. 递归结束条件
    if root==null:
       return
    # 2. 取值的位置   
    // 前序遍历
    traverse(root.left)
    // 中序遍历
    traverse(root.right)
    // 后序遍历
}
```

#### 动态贪心回溯适用场景
`动态规划`，`回溯算法`，`贪心算法`
- 回溯算法 本质是暴力穷举，解决的是求所有解的问题，时间复杂度最高
- 动态规划 求最优解的问题，往往可以通过dp或者备忘录的方式降低时间复杂度
- 贪心算法 只是动态规划的一种特例，即每次获取到的都是局部最优解，那最后一定是最优解，即该类问题满足这种情形就可以用贪心算法

#### 动态规划
动态规划的核心是找到状态转移方程，要解状态转移方程，需要弄清楚三个概念，`状态`，`选择`，`base case`
- 状态：即定义好函数对应的含义
- 选择：最优解，就一定存在多个选择，弄清楚有哪些选择项
- base_case: 是在最大或者最小的情况下该值是多少
- 状态转移方程：当前状态的值，一定跟之前的状态值有关联
##### 模板代码


#### 回溯算法
回溯算法，本质是一颗决策树，解题的核心框架：
0. 找到递归的结束条件
1. track数组记录已选择
2. 选择有根据的数据
3. 加入到track数组中
4. 递归继续选择合理的数据，直到满足递归结束条件
5. 然后再取消刚刚的选择
##### 模板代码
```python
result = []
def backtrack(路径, 选择列表):
    if 满足结束条件:
        result.add(路径)
        return

    for 选择 in 选择列表:
        做选择
        backtrack(路径, 选择列表)
        撤销选择
```
##### 实例代码
```python
// 路径：记录在 track 中
// 选择列表：nums 中不存在于 track 的那些元素
// 结束条件：nums 中的元素全都在 track 中出现
void backtrack(int[] nums, LinkedList<Integer> track) {
    // 触发结束条件
    if (track.size() == nums.length) {
        res.add(new LinkedList(track));
        return;
    }

    for (int i = 0; i < nums.length; i++) {
        // 排除不合法的选择
        if (track.contains(nums[i]))
            continue;
        // 做选择
        track.add(nums[i]);
        // 进入下一层决策树
        backtrack(nums, track);
        // 取消选择
        track.removeLast();
    }
}
```

