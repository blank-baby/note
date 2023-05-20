#  javascript

## 1.对象

 ###  1.直接定义

```javascript
var car = {
	 name:"汽车",
	 run: function() {
		console.log("汽车的eat")
	}
}
//这就相当于直接new出来一个叫car1的对象
```

### 2.set和get

```javascript
var person = {
    firstName: "Bill",
    lastName : "Gates",
    language : "en",
    //给langage属性加上了set和get  当对象给属性赋值是直接用set和get方法
    //并且用属性的方式调用，不用加()
    get lang() {
        console.log("get")
        return this.language;
    },
    set lang(language){
        console.log("set")
        this.language = language
    } 
};

person.firstName = "7877887"
console.log(person.firstName)
person.lang = "12121"
```





### 3.通过new创建

```javascript
function car(name) {
	this.name = name
	this.run = function(){
		console.log("汽车的run")
	}
}

var car = new car()
//function是相当于定义的一个类(class),通过new car2把这个类给创建出来
```

- 通过class创建一个对象

```javascript
//定义的class
function vue(argument) {
    console.log(argument)
    this.name=argument.name
    this.run=argument.run
}

//创建一个对象并且初始化
var vue1 = new vue({
    name: 'jiruixin',
    run:{
        run1:function(){
            console.log("1111")
        },
        run2:function(){
            console.log("2222")
        }
    }
})

console.log(vue1.name)
vue1.run.run1()
```



## 2.函数

用function定义的就是函数

```javascript
function car(name) {
	this.name = name
	this.run = function(){
		console.log("汽车的run")
	}
	console.log(22222222222)
}
//function既可以当普通的函数也可以用来定义class
```

## 3. ES6模块

- 使用export default 可以不用在导出和导入的变量加{}

```javascript
var person = {
    firstName: "Bill",
    lastName : "Gates",
    language : "en",
    get lang() {
        console.log("get")
        return this.language;
    },

    set lang(language){
        console.log("set")
        this.language = language
    } 
};

export default person
```

```javascript
import person from './java2.mjs'

person.firstName = "7877887"
console.log(person.firstName)
person.lang = "12121"
```

- 直接使用export导出变量，必须在导出和导入后面加{}

```javascript
var person = {
    firstName: "Bill",
    lastName : "Gates",
    language : "en",
    get lang() {
        console.log("get")
        return this.language;
    },

    set lang(language){
        console.log("set")
        this.language = language
    } 
};

export {person}
```

```javascript
import {person} from './java2.mjs'


person.firstName = "7877887"
console.log(person.firstName)
person.lang = "12121"
```









