# 选择排序真的和冒泡排序一样低效吗？
2017/03/14 14:10:02


## 相似，却能分高下

曾几何时，在我的脑袋里，[选择排序][wiki_selection]跟[冒泡排序][wiki_bubble]划上了等号。它们同属于时间复杂度为O(n^2)，空间复杂度为O(1)的算法。

今天偶然发现，选择排序和冒泡排序还是有一个很大的区别，那就是排序过程中元素的交换次数。

这一点在对数组排序的时候区别不大，因为交换数组元素的开销非常小，然而当需要排序的不是数组，这个交换的开销就不确定了。

所以选择排序优于冒泡排序。

下面通过例子来展现冒泡排序交换次数大于选择排序。


## 实例比较

先写一个用来交换数组元素的函数，通过在这个函数里加入一些打印，我们还可以得到交换次数的统计。

```js
function swapArray(a, i1, i2) {
	console.log('Doing a swap...');
	[ a[i1], a[i2] ] = [ a[i2], a[i1] ];
}
```

然后就是今天的主角了，选择排序：

```js
function selectionSort(a, fn) {
	for (var i = a.length - 1; i >= 1; i--) {
		var pos = i;
		for (var j = 0; j < i; j++)
			if (fn(a[j], a[pos]))
				pos = j;
		if (i != pos)
			swapArray(a, i, pos);
	}
	return a;
}
```

和冒泡排序：

```js
function bubbleSort(a, fn) {
	for (var i = a.length - 1; i >= 1; i--)
		for (var j = 0; j < i; j++)
			if (fn(a[j], a[j + 1]))
				swapArray(a, j, j + 1);
	return a;
}
```

下面通过实际调用来测试效果。

```js
bubbleSort([9,8,7,7,7,7,7,7], (a, b) => a > b)
```

在得到正确的排序结果的同时，还打印了13条"Doing a swap..."。

```js
selectionSort([9,8,7,7,7,7,7,7], (a, b) => a > b)
```

同样得到正确的排序结果，但只打印了2条"Doing a swap..."。

显而易见，选择排序的交换次数更少。


[wiki_selection]: https://en.wikipedia.org/wiki/Selection_sort
[wiki_bubble]: https://en.wikipedia.org/wiki/Bubble_sort

