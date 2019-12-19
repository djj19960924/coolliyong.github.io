# 排序算法

## 冒泡排序（线性查找）

冒泡排序就是重复“从序列右边开始比较相邻两个数字的大小，再根据结果交换两个数字
的位置”这一操作的算法。在这个过程中，数字会像泡泡一样，慢慢从右往左“浮”到序列的
顶端，所以这个算法才被称为“冒泡排序”。

```js
function lineSort(list) {
  for (let i = 0; i < list.length; i++) {
    for (let j = i; j < list.length - 1; j++) {
      if (list[j] > list[j + 1]) {
        list[j] = list[j + 1] + list[j]
        list[j + 1] = list[j] - list[j + 1]
        list[j] = list[j] - list[j + 1]
      }
    }
  }
}
```

### 过程

比如: 3 1 5 2 4

1.  第一次遍历结果 13245
2.  第二次遍历结果 12345

### 弊端
已经出来了，尽管只需要遍历10次就可以，但是还是要遍历更多次



## 选择排序 （线性查找最小值）

选择排序就是重复“从待排序的数据中寻找最小值，将其与序列最左边的数字进行交换”
这一操作的算法。在序列中寻找最小值时使用的是线性查找。

```js
function selectSory(list) {
  // 记录最小值
  let minIdx
  for (let i = 0; i < list.length; i++) {
    minIdx = i
    for (let j = i; j < list.length; j++) {
      // 如果当前小于 需要比对的值 且 小于最小值
      if (list[j] < list[minIdx]) {
        minIdx = j
      }
    }
    if (i !== minIdx) {
      list[i] = list[minIdx] + list[i]
      list[minIdx] = list[i] - list[minIdx]
      list[i] = list[i] - list[minIdx]
    }
  }
}


```

### 过程

1. 找出最小值，换到第一个位置上
2. 找出最小值，换到第二个位置上
...


### 弊端
查找次数过多，和冒泡排序差不多





## 堆排序

## 快速排序