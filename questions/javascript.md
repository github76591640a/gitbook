##### 1.  `['1', '2', '3'].map(parseInt)` 会输出什么？为什么? --- [1, NaN, NaN]

​	这道面试题考的是两个函数的用法

	 + 数组的map方法

```javascript
Array.map(function(item, index, arr) {})
// 数组的map方法中传入一个构造函数，这个构造函数中有三个参数：当前项的值、当前项的下标、当前数组
```



	 + parseInt函数的用法

```
parseInt(String, 基数)
// 首先我们要明确parseInt这个方法的用处， 就是将字符串转换成为整数
// 基数的取值范围是2-36， 他的作用就是控制前面的字符串按照多少进制的数进行处理
// 如果基数超过36或者小于2输出'NaN'， 但是0比较特殊，他会按照10进制进行处理
// 例如 parseInt('1234567', '4')  输出的值是27
// 由于要将前面的参数按照4进制的数进行处理， 所有会将4567舍弃掉，然后将123转换成十进制的数
```

​	下面我们再看上面这道题，

​	当输入的值是数组的第一位的时候 parseInt('1', 0),输出的是1

​	当输入的值是第二位的时候parseInt('2', 1), 输出NaN

​	当输入的值是第三位的时候parseInt('3', 2) , 这时候会将3舍弃，然后对空字符串按照2进制进行转化，输出NaN

所以最终输出的值是[1, NaN, NaN]

#### 2. 这是一道很有意思的题， 直接上代码

```javascript
let length = 10;
function fn() {
    console.log(this.length);
}
var obj = {
    length: 5,
    method: function (fn) {
        fn();
        arguments[0]();
    }
}
obj.method(fn, 1)
```

​	首先说这道题会打印什么， 0 2.这道题考了很多的东西。

​	我觉得这种题是特别考验细心和基础的。那既然答案都出来的那么我们现在就去分析一下为什么会打印这两个东西。

​	let length = 10; 这里首先我们要明白let和var的区别，var和不带声明符号直接赋值的变量会直接挂载window对象上面作为window对象的属性存在，而let和const则不会这么做，只是在本块级作用域能够使用此变量。所以此时如果我们直接执行fn函数的话会打印0。window的length属性是页面iframe标签的个数。

​	那么现在是在obj的methods方法中执行fn方法， 这里有一点疑惑。我们平时说的谁调指谁，那么这里的fn方法中的this是不是会指向obj对象呢。答案是不会的，我们说的谁调用指向谁，因为。

​	那么arguments\[0]()执行之后为什么是2呢。其实arguments是一个伪数组，他key值为0的项指向fn，这样执行arguments其实就是相当于执行arguments对象下面的一个方法。所以this指向arguments，那么arguments的length不就是2了吗。



​	

