---
title: JavaScript 实现二分搜索和快速排序
date: 2019-04-22 16:33:39
tags:
  - 算法
---

## 二分搜索

> 在计算机科学中，[二分搜索（binary search）](https://zh.wikipedia.org/wiki/%E4%BA%8C%E5%88%86%E6%90%9C%E7%B4%A2%E7%AE%97%E6%B3%95)是一种在*有序数组*中查找某一特定元素的搜索算法。搜索过程从数组的中间元素开始，如果中间元素正好是要查找的元素，则搜索过程结束；如果某一特定元素大于或者小于中间元素，则在数组大于或小于中间元素的那一半中查找，而且跟开始一样从中间元素开始比较。如果在某一步骤数组为空，则代表找不到。这种搜索算法每一次比较都使搜索范围缩小一半

利用递归实现

```javascript
/**
 * 二分查找，递归实现
 * @param {*} arr 
 * @param {*} target 
 * @param {*} low 
 * @param {*} high 
 */
function binarySearch(arr, target, low = 0, high = arr.length - 1) {
  if (low > high) {
    return -1
  }
  const mid = parseInt((low + high) / 2)

  if (target < arr[mid]) {
    return binarySearch(arr, target, low, mid - 1)
  }
  if (target > arr[mid]) {
    return binarySearch(arr, target, mid + 1, high)
  }

  return mid
}
```

非递归实现

```javascript
/**
 * 二分查找，不使用递归
 * @param {*} arr 
 * @param {*} target 
 * @param {*} low 
 * @param {*} high 
 */
function binarySearch(arr, target, low = 0, high = arr.length - 1) {
  while (low <= high) {
    console.log(low, high)
    const mid = parseInt((low + high) / 2)
    if (target < arr[mid]) {
      high = mid - 1
    } else if (target > arr[mid]) {
      low = mid + 1
    } else {
      return mid
    }
  }

  return -1
}
```

二分搜索的缺点是要求待查数组为有序数组，为保持表的有序性，在顺序结构里插入和删除都必须移动大量的结点，所以插入删除都比较困难，而优点是查找次数比较少，平均性能好。

## 快速排序

上面说过，二分搜索算法是针对*有序数组*的，那么如果给的是无序数组，那就要先把它变成有序数组。

*快速排序（Quicksort）*就是一种排序算法。

> 快速排序使用分治法（Divide and conquer）策略来把一个序列（list）分为较小和较大的2个子序列，然后递归地排序两个子序列。
>
> 步骤为：
>
> 挑选基准值：从数列中挑出一个元素，称为“基准”（pivot），
> 分割：重新排序数列，所有比基准值小的元素摆放在基准前面，所有比基准值大的元素摆在基准后面（与基准值相等的数可以到任何一边）。在这个分割结束之后，对基准值的排序就已经完成，
> 递归排序子序列：递归地将小于基准值元素的子序列和大于基准值元素的子序列排序。

实现如下

```javascript
function quickSort(arr) {
  if (arr.length < 2) return arr
  
  const basic = arr
  const left = []
  const right = []

  for (let i = 1; i < arr.length; i++) {
    const iv = arr[i]
    iv >= iv && right.push(iv)
    iv < iv && left.push(iv)
  }

  return quickSort(left).concat(basic, quickSort(right))
}
```