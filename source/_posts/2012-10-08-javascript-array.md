---
layout: post  
title: "Array in Javascript"  
description: "Array in Javascript"  
category: Javascript
tags: [Javascript, array]  
---

- Javascript数组是无类型的：数组元素可以是任意类型，并且同一个数组中的不同元素也可能有不同的类型。  
- 可以使用非负整数来索引数组。在这种情况下，数值装换为字符串，字符串作为属性名来用。  
- 数组类型索引仅仅是对象属性名的一种特殊类型，这意味着Javascript数组没有越界的概念。
- 通常，数组的length属性值代表数组中元素的个数。如果数组是稀疏的，length属性值大于元素的个数。
- 当在数组直接量中省略时不会创建稀疏数组。省略的元素在数组中是存在的，其值为undefined.这和数组元素根本不存在是有一些微妙的区别的。可以用in操作符检测连着之间的区别。
>var a1=[,,,]			//数组是[undefined,undefined,undefined]  
>var a2=new Array(3)	//该数组根本没有元素  
>0 in a1				//=> true:a1 在索引0处有一个元素  
>0 in a2				//=> false:a2在元素0处一个值为undefined的元素  

    当使用for/in循环时，a1和a2之间的区别也很明显。  
- 在嵌套循环或其他性能非常重要的上下文中，优化代码
>for(var i=0,len=keys.length;i< len;i++)

- Array.join()方法是String.split()方法的逆向操作，后者是将字符串分割成若干块来创建一个数组。
- 数组对象的有一些特性是其他对象所没有的
	- 当有新的元素添加到列表中时，自动更新length属性
	- 设置length为一个较小值将截断数组
	- 从Array.prototype中继承一些有用的方法
	- 其类属性为“Array”
    
