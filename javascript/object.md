#### 对象的属性

#####    对象属性的描述

​	在js当中，对象的属性分为两种： 数据属性、访问器属性。那他们各是什么东西呢

​	数据属性：数据属性包含一个保存数据值的位置。也就是有value值的对象属性就是数据属性，没有就不是喽。数据属性提供四个配置值来对属性进行描述。

​	[[Configurable]]： 用来控制该属性是否可以通过delete删除，是否可以修改其配置值。若设置为false，使用Object.definedProperty来修改配置的值都会报错。

​	[[Enumberable]]：用来控制该属性是否可以通过循环找到，包括for..in、Object.keys...

​	[[Writeable]]：用来控制该属性是否是可以被重写的。如果为false，那么该属性将不能被修改。

​	[[value]]：用来给该属性添加值

​	当我们使用Object.defineProperty来给对象添加属性的时候，如果不指定某项配置的值，那么他默认将是false。当我们直接给属性赋值的时候，默认值都是true，value为赋予的值。使用

```javascript
Object.defineProperty(obj, key, {
    // 描述
})
```



​	访问器属性：访问器属性不包含数据值。相反，它们包含一个获取（getter）函数和一个设置（setter）函数，不过这两个函数不是必需的。访问器属性也提供四个配置值来描述属性的行为。

​	[[Configurable]]、[[Enumberable]] 这两个属性和值属性是一样的。除了这两个属性，还另外存在两个属性setter和getter函数。

​	[[set]]： 他的值是一个函数。用来描述当前的值被设置的时候执行哪些操作，此函数接收一个参数，表示当前接收到的值。

​	[[get]]：他的值同样是一个函数，但是不接受任何的参数。用来描述当获取当前该属性的时候执行哪些操作。

​	如果想要一次设置多个属性Object.defineProperties将是最好的选择。他接收两个参数，分别是需要操作的对象和一个描述对象。

```javascript
Object.defineProperties(obj, {
	key1: {
		// 描述
	}
})
```



##### 获取对象属性的描述

​	获取对象属性的描述经常用到的是两个方法。 **Object.getOwnPropertyDescriptor**和**Object.getOwnPropertyDescriptors**，通过方法的名字不难看出来。前者是获取对象上单个属性的描述，而后者是获取对象上全部属性的描述。

​	他们的使用方法是一样的，不同的是前者传入两个参数：对象和key值。后者传入一个参数：需要获取的对象。



​	

​	