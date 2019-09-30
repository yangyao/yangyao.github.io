---
layout: post
title: 理解yield和promise的一个过程
tags: ['php']
categories:
- php
---

# 理解yield和promise的一个过程

---

	function sayHello(){
		return Promise.resolve('hello').then(function(hello){
			return hello;
		})
	}
	
	function *genHelloWorld(){
		var hello = yield sayHello();
		console.log(hello);
		console.log('world');
	}
	
	/**
	 * 执行步骤
	 * 1.sayHello                                               返回 {}
	 * 2.yield sayHello()                                       返回 {value:Promise{<pending>},done:false}
	 * 3.var hello = sayHello() && console.log(hello) && console.log('world') 返回 { value: 'undefined', done: true }
	 */
	
	
	
	// 执行了 sayHello() 等于 执行了 resolve并传递了hello过去，执行了then并传递一个回调函数
	// 注意这里并没有赋值，仅仅是执行了sayHello函数，var hello 是 undefined
	// 此时 gen为一个空对象 {} !!! 注意这里并没有retuan但是 gen却是一个空对象,怎么来的？
	var gen = genHelloWorld()
	// 此时p为一个 Promise对象 {value:Promise{<pending>},done:false} 
	// 执行 yield {value:Promise{<pending>},done:false} 
	// !!! 注意这里也没有return 但是 p却是一个Promise对象？怎么来的？
	// 此时也没有赋值操作，var hello 是 undefined
	var p = gen.next()
	// 需要传递一个回调函数给 Promise 的then方法去获得返回的数据
	p.value.then(function(data){
		// 获得then返回的数据之后，进行第二次迭代
		// 在异步获取到值之后再调用next走下面的流程，代码写法是同步的，但却是异步执行的
		// 此时 hello 被赋值为 data ，并且执行后面的语句直到遇到下一个yield
		// 返回 { value: undefined, done: true } !!! 注意这里也是没有return的
		console.log(gen.next(data));
	
	})
