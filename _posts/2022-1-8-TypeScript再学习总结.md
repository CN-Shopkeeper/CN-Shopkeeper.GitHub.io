---
layout: post
author: shopkeeper
categories: typescript
---

最近参照视频教程重新学习了一遍TypeScript，现在将之前没有注意到的细节记录下来。

unknown 类型可以视作为安全的any，即any可以赋值给确定类型的变量，但是unknown不可以。

***

声明一个变量为对象类型，且指定必须含有某一个字段，但不限制其它的字段
```typescript
let a: {name: string, [propName: string]: any}
```

***

声明一个变量为函数类型
```typescript
let f: (a:number, b:number) => number
```

***

枚举的用法
```typescript
enum Gender {
  male = 1,
  female = 2
}
```
使用时
```typescript
let p: {name: string, gender: Gender};
p={
  name: "abc",
  gender: Gender.Male
}
```

***

类型别名
```typescript
type myType = 1 | 2 | 3 | 4;
let m : myType;
```

***

tsconfig.json详解
```json
{
/*
  tsconfig.json是ts编译器的配置文件，ts编译器可以根据它的信息来对代码进行编译
  可以写注释：JSON with comments
*/ 
/*
"include" 用来指定哪些ts文件需要被编译
一个*表示任意文件，两个*表示任意目录
*/
  "include":[
    "./src/**/*"
  ],
/*
"exclude" 表示不编译哪些文件
默认值：["node_modules", "bower_components", "jspm_packages"]
*/
  "exclude":[
    ".src/hello/**/*"
  ],
/*
"extends" 定义被继承的文件
"files" 指定被编译文件的列表，只有需要编译的文件少时才会用到
*/

  "compilerOptions": {
    // target 用来指定ts被编译为的ES版本
    // es3, es5, es6, es2015, es2016, es2017, es2018, es2019, es2020, esnext
    "target": "ES3",
    // module 指定要使用的模块化的规范
    // none, commonjs, amd, system, umd, es6, es2015, es2020, esnext
    "module": "es2015",
    // lib 用来指定项目中要使用的库
    // "lib": [],
    // outDir 用来指定编译后文件所在目录
    "outDir": "./dist",
    // 将代码合并为一个文件。用的不多，只有module为amd和system时可以使用
    "outFile": "./dist/app.js",
	// 是否对js文件进行编译，默认是false
    "allowJs": false,
    // 是否检查js代码是否符合语法规范，默认是false
    "checkJs": false,
    // 是否移除注释
    "removeComnents": false,
    // 不生成编译后的文件，默认为false
	"noEmit": true,
	// 当有错误时不生成编译后文件，默认为false
	"noEmitOnError": true,
	// 用来设置编译后的文件是否使用严格模式，默认false
	//所有严格检查的总开关
	"strict": true,
	"alwaysStrict": false,
	// 不允许隐式any
	"noImplicitAny": false,
	// 不允许不明确类型的this
	"noImplicitThis": false,
	// 严格检查null
	"strictNullChecks": false	
  }
}
```

***

TS类

- readonly开头的属性表示一个只读的属性，无法修改
- get方法也可以写成`get fieldName(){}`， 使用时`obj.fieldName`；同理，set方法也可以写成`set fieldName(type value){}`，使用时`obj.fieldName=value`
- 构造器语法糖
```typescript
class C{
	constructor(public name:string){}
}
```