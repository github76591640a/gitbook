#### 1. 谈谈什么是防抖和节流， 并书写防抖和节流函数

​	防抖： 就是在一定时间范围内只执行一次动作， 若这个范围内这个动作又被触发了， 那么这个动作不会执行且重新计时。就像坐电梯一样， 进来一个人等待几秒然后工作，若在等待的这个过程中又进来人了，那个就再等待几秒之后再升降。下面看代码：

```javascript
// 工作思路：
// 创建一个可以传入函数和限制时间作为参数的函数， 并且不能妨碍执行环境的作用域

function debounce(fn, wait) {
    let timer
    return function() {
        let context = this;
        let args = arguments;

        if (timer) clearTimeout(timer);
        timer = setTimeout(function () {
            fun.apply(context, args)
        }, wait);
    }
}
```



​	节流： 节流就是在一定时间内只执行一次动作，若在这个时间范围内此动作再次触发则不会执行。下面看代码：

 ```javascript
// 工作思路：
// 创建一个可以传入函数和限制时间作为参数的函数， 并且不能妨碍执行环境的作用域
function throttle(fn, wait) {
	let prev = new Date()
    return function() {
        let now = new Date()
        if (now - prev > wait) {
            fn.apply(this, arguments)
            prev = new Date()
         }
    }
}
 ```

