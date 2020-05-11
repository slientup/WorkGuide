# 数据结构与算法

[用动画的形式呈现解LeetCode题目的思路](https://github.com/MisterBooo/LeetCodeAnimation)

一 算法
---
### 递归算法
递归算法 必须满足两个条件：
- 自己调用自己
- 必须要有终止条件  list为null 叶子节点等
```
void foo()
{
    // 返回部分 
    if (condition)
    {
        return;
    }
    
    // 递归部分
    foo();
}
```
递归算法适用场景是`分而治之`，即要解决当前问题，需要先解决无数个子问题，当这些子问题都解决了，该问题自然解决,那就是下一个数的值跟前面的值有关系
参考链接：https://www.jianshu.com/p/0e4753ac9ffb   https://blog.csdn.net/sakura__lu/article/details/95486397  
**总结：** 递归并不是一个好的算法，非常耗时，能不用递归就不要用递归来处理问题(如空间换时间),直接先生成对应的list 或者动态规划

