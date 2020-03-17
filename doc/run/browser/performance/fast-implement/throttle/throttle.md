## 防抖

> 每次触发事件时都取消之前的延时调用方法

```js
<input id="aa" type="text">
<script type="text/javascript">
	function debounce(fn) {
		let timeout = null; // 创建一个标记用来存放定时器的返回值
		return function () { // 这里不能用箭头函数，用了this指向window,以前指向input
			// 每当用户输入的时候把前一个 setTimeout clear 掉
			clearTimeout(timeout);
			// 然后又创建一个新的 setTimeout, 这样就能保证interval 间隔内如果时间持续触发，就不会执行 fn 函数
			timeout = setTimeout(() => {
				fn.apply(this, arguments);
			}, 500);
		};
	}
	function handle(e) {
		console.log(e.target.value);
	}
	document.getElementById('aa').addEventListener('input', debounce(handle));
</script>
```
## 节流

> 高频事件触发，但在n秒内只会执行一次，所以节流会稀释函数的执行频率

```js
<input id="aa" type="text">
<script type="text/javascript">
	function throttle(fn) {
		let canRun = true; // 通过闭包保存一个标记
		return function () {// 这里不能用箭头函数，用了this指向window,以前指向input
			// 在函数开头判断标记是否为true，不为true则return
			if (!canRun) return;
			// 立即设置为false
			canRun = false;
			// 将外部传入的函数的执行放在setTimeout中
			setTimeout(() => {
				// 最后在setTimeout执行完毕后再把标记设置为true(关键)表示可以执行下一次循环了。
				// 当定时器没有执行的时候标记永远是false，在开头被return掉
				fn.apply(this, arguments);
				canRun = true;
			}, 500);
		};
	}
	function handle(e) {
		console.log(e.target.value);
	}
	document.getElementById('aa').addEventListener('input', throttle(handle));
</script>
```