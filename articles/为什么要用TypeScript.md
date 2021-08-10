Typescript经过这么多年的发展，已经打败了所有的竞争对手，成为了事实上的类型Javascript标准，被大量公司所采用，但还是经常有人问我，我们为什么要用ts？使用ts能给我们带来什么好处？这次我就用实际例子来讲解一下使用ts能给我们带来哪些收益。


### **1. 提供很好的的拼写检查和智能提示**
**拼写错误大部分人都会犯的常见错误。**  
然而在js这种动态语言中、排查拼写错误会很痛苦：  
```javascript
const user = {
  name: '尼古拉斯·赵四',
  pet: {
    name: '狗子'
  }
}
user.name.spit('·')
```
我想将user的名字切割成first name和last name，  
本来需要使用split方法，不小心拼成了 spit。  
不过这种的还好，在运行中运行引擎会报错，告诉我 `spit is not a function` 。 
下面这种错误就很难发现了：  
```javascript
const user = {
  name: '尼古拉斯·赵四',
  pet: {
    name: '狗子'
  },
  occupcation:'保安'
}

console.log(user.occupation)

```
user有个属性 `职业` ，
职业这个词本来应该是 `occupation`  ，但是我们的小伙伴不小心拼成了`occupcation`,多了个c。  
人都会犯错的，没什么大问题。  
但是其他使用的小伙伴并不知道这个单词被拼错了，按照正常的 `occupation`去调用,那就只会返回一个undefined了。  
而且可能由于我们做了一些防空值的处理，导致这种错误更加难以被发现。  
即使发现了，我们还面临着一个问题，修改这个词可能需要很大的成本，稍后我们会讲解rename成本。  

ts可以解决上述的这些问题吗？我们来试一试
```typescript
type User = {
  name: string,
  pet: {
    name: string
  },
  occupcation:string,
}
const user:User = {
  name: '尼古拉斯·赵四',
  pet: {
    name: '狗子'
  },
  occupcation:'保安'
}
user.name.spit('·')//  属性“spit”在类型“string”上不存在。你是否指的是“split”?

console.log('他的工作是',user.occupcation)// 属性“occupcation”在类型“IUser”上不存在。你是否指的是“occupation”?
```
当你使用ts的编译功能，或者的安装了ts相关的lint，很容易就可以得到上面那样的提示。帮助我们在开发阶段就排查出相应的错误。   

**同时，我们还可以用ts给变量或参数限定类型，用来规避js默认的隐式转换带来的一些困扰。**   
可能大家见过一些类似下面这种js的面试题：
```javascript
"" == 0;
[] == 0;
[] == ![];
[] == ![1,2,3];
```
上面的几个比较表达式的结果全部都是true。   
js有一个比较复杂的隐式转换规则，具体可以参考这篇总结 [详解一下 javascript 中的比较](https://segmentfault.com/a/1190000000650129)  

```javascript
// js中， 0、-0、null、false、NaN、undefined、或者空字符串""，在比较运算中都被认为是false
"" == 0;//true
// 比较的一端是number，则另一端执行toPrimitive(value)
// array的toPrimitive返回它的toString 
// []的toString()是一个空字符串
// 比较的一端是string，另一端是number,则将string转化为Number类型
// 空字符串会被转成0 Number('') == 0
[] == 0;//true
// 首先执行![]，上面讲了判断为false的情况，[]作为一个array对象为true，取反后为false，当比较两端有一端为boolean时，将bool值转为number，false是0，true是1，这时又变回了上一个问题，重复上一个问题的操作
[] == ![];//true
[] == ![1,2,3];//true
[0]==![0]//true
```
建议大家在生产环境中尽量不要使用这些复杂的转换特性，防止写错或记错造成bug。  
比较判断尽量写的含义清晰一些，比如 ` [].length==0 ` 或者` ![].length `这样，简洁易懂。  
或者在书写中使用三等号 ` [].length===0 ` ,三等号会首先验证等号两侧的类型，类型不同会直接返回false。

隐式转换在代码中也会造成一些隐患:  
```javascript
function incTen(n) {
  if (n > 1) {
    return n + 10
  }
  return n
}
```
当给这个方法传入一个正数时，返回加10，否则返回原值 , 现在传进去一个字符串"10" ,会发生什么结果呢?  
答案是会返回 `"1010"`。  
由于js中没有类型的限制，因此为了防止传参错误，我们需要在代码中增加很多对参数类型的判断。   
```javascript
function incTen(n) {
  if(typeof n !== 'number'){
    throw Error('type is not number')
  }
  if (n > 1) {
    return n + 10
  }
  return n
}
```
编写类型判断语句会耗费时间不说，即使增加了检查代码，也并不能在第一时间发现调用错误，必须在运行的时候才能发现问题。**因此我们可以使用ts来给变量或参数限定类型，第一时间提示参数类型的错误：** 

```typescript
function incTen(n: number) {
  if (n > 1) {
    return n + 10
  }
  return n
}

incTen('10')// error：类型“"10"”的参数不能赋给类型“number”的参数。
```

ts给我们提供了非常好用的智能提示，同时也可以将很大一部分错误在编写或者编译阶段就提示出来，让我们加以修改。  

    注意！当你将变量的类型设置为any的时候，会绕过类型检查。所以我们要谨慎使用any类型，防止typescript变成"anyscript"
```typescript
// 例：
const num:any = '10'
incTen(num) //'1010'
```
### **2. 方便快捷的代码重构**
在刚刚的例子中讲到了，**在js中，即使是一个简单的属性rename，也有可能需要高昂的成本。**  
举个简单的例子   
```javascript
// a.js
export const user = {
  name: '尼古拉斯·赵四',
}

// b.js
import { user } from './a'

function callName(user) {
  console.log(user.name)
}

callName(user)

// c.js
import { user } from './a'

function callName(user) {
  console.log(user.name)
}

callName(user)
```
我们在a文件中获取了一个从接口返回的user，user中有一个属性叫name，在b,c文件中分别import了这个user并且输出它的name

这时需求有了变化，接口返回的user中没有了name，变成了realName和nickName  

我们现在怎么进行修改呢？  
有人可能会说全局搜索name，全部替换成nickName就好了。  

那么稍微等一下，我们修改一下代码
```javascript
// a.js
export const user = {
  name: '尼古拉斯·赵四',
  pet: {
    name: '狗子'
  }
}

// b.js
import { user } from './a'

function callName(user) {
  console.log(user.name)
}

function hisPet(user) {
  console.log('他的宠物叫：', user.pet.name)
}

callName(user)
hisPet(user)

```
现在user中有个pet，pet也有name，
在全局替换的时候会把pet的name也替换成nickName。  
这下出错了，pet的name变成了undefined。  

所以很明显，在实际开发中经常不能简单的全局替换，只能全局搜索之后，排查每一个name是否是a文件里user的name，之后再一个一个的替换。  
如果这是一个有几百个文件的大型项目的话，那一个简单的rename都需要半天的时间。   

现在我们看看ts怎么帮我们解决rename问题。   
```typescript
// 首先我们定义一个User类型。  
// a.ts
export type IUser = {
  name: string
  pet: {
    name: string
  }
}
// 将user设置为User类型
export const user:User = {
  name: '尼古拉斯·赵四',
  pet: {
    name: '狗子'
  }
}

// b.ts
import { user,IUser } from './a'

function callName(user:IUser) {
  console.log(user.name)
}

function hisPet(user) {
  console.log('他的宠物叫：', user.pet.name)
}

callName(user)
hisPet(user)
// c.ts
import { user,IUser } from './a'

function callName(user:IUser) {
  console.log(user.name)
}

callName(user)
```
使用vscode的重命名功能 `F2`，  输入新名称并回车。rename结束。     

### **3. 明确定义函数的参数类型和函数重载**

axios是目前大家常用的前端HTTP库，它提供了多种请求方式
```javascript
  const url='http://www.baidu.com'
  // 1
  axios.get(url)
  // 2
  axios.request({
    url,
    method:'get'
  })
}
```
如果我请求时这样操作，具体会发送什么请求呢？
```javascript
  const url='http://www.baidu.com'
  // 1
  axios.get(url,{
    url:'http://www.google.com',
    method:'post'
  })
  // 结果是什么呢？自己去试一试吧
}
```
这种情况只能看作者对方法定义，是方法定义描述优先级更高，还是用户的参数描述优先级更高，不同的作者可能会有不同的定义。
但是在ts中我们可以使用参数定义来规避这种情况的发生。
```typescript
type AxiosRequestConfig = {
  url: string;
  method: Method;
  headers?: any;
  params?: any;
  data?: any;
}
function request(config:AxiosRequestConfig){
  //do request
}
type AxiosGetConfig = {
  headers?: any;
  params?: any;
  data?: any;
}
function get(url:string,config:AxiosGetConfig){
  // do get
}
// 也可以借用ts高级一些的特性减少重复定义
function get(url:string,config:Omit<AxiosRequestConfig,'url'|'method'>){
  // do get
}
```
```javascript
function logPeople() {
  if (typeof arguments[0] == 'object') {
    console.log('name', arguments[0].name)
    console.log('age', arguments[0].age)
  }else {
    console.log('name', arguments[0])
    console.log('age', arguments[1])
  }
}
```
现在这个例子实现了一个简单的输出people个人信息的方法  
你可以传入一个people对象 
```javascript
logPeople({name:'赵四',age:18}) //print name 赵四 age 18
```  
也可以将个人属性依次传入 ``  
```javascript
logPeople('赵四',18)//print name 赵四 age 18
```
如果理解错了用法。就会产生问题 
```javascript
logPeople({name:'赵四'},18)//print name { name: '赵四' } age 18
```
这样的问题还算是比较容易发现的，在运行时就可以发现  
但是一些比较有责任心的作者为了方便调用者，会对提供一些默认参数，当你没有传递部分参数时他会使用默认参数来代替  
```javascript
function logPeople() {
  if (typeof arguments[0] == 'object') {
    const p = {
      adulted:true,
      ...arguments[0],
    }
    console.log('name', p.name)
    console.log('age', p.age)
    console.log('adulted', p.adulted)
  }
  else {
    console.log(2)
    console.log('name', arguments[0])
    console.log('age', arguments[1])
    console.log('adulted', arguments[2].adulted||true)
  }
}
```
以上方法提供了一个默认值，是否已经成年，默认为true  
如果你写错了也可以正常运行,输出也是正常，只有遇到一些特殊情况时，才会产生错误  
如果我们在测试时的用例覆盖范围不够全面，会导致这种错误更难以发现，以至于会在生产环境中发生不可预知的错误  
```javascript
logPeople({name:'赵四',age:18},true)
```  

## 7 明确定义函数重载

```typescript
function logPeople(people: { name: string, age: number, adulted?: boolean }): void;
function logPeople(name: string, age: number, adulted?: boolean): void;
function logPeople(): void {
  if (typeof arguments[0] == 'object') {
    const p = {
      adulted: true,
      ...arguments[0],
    }
    console.log('name', p.name)
    console.log('age', p.age)
    console.log('adulted', p.adulted)
  }
  else {
    console.log(2)
    console.log('name', arguments[0])
    console.log('age', arguments[1])
    console.log('adulted', arguments[2].adulted || true)
  }
}

logPeople({
  name: 'zhaosi',
  age: 18,
})
```





### **ts与c#、java都很像**
    ts的领导人是一位顶级大佬，安德斯海尔斯伯格，微软的c#就是这位大佬领导设计的，而c#又与java几乎一致，因此ts与c#、java用起来都很像，这就方便了java和c#程序员使用ts参与到项目的开发中，尤其是c#。
    例如，在游戏领域unity3D已是占有率最高的游戏引擎，而unity3D使用c#作为主要语言。随着web端渲染的日益成熟，大量的相关从业人员加入到web端3D渲染的开发中，使用ts可以减少他们重新学习的负担。比如白鹭引擎就使用了ts作为默认开发语言。
