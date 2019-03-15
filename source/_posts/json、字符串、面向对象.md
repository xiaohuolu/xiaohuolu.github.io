---
    title: json、字符串、面向对象
    date: 2019-03-14 14:26:00
    tags: es6
    categories: 前端
---

### json

1. 如果key和value一样的话可以简写

```
    let a=5
    let b=3
    <!-- let obj={a:a,b:b} -->
    let obj={a,b}
```
2. json内的方法可以简写

```
    <!-- let obj={
        a:1,
        b:2,
        show:function(){
            alert(this.a,this.b)
        }
    } -->

    let obj={
        a:1,
        b:2,
        show(){
            alert(this.a,this.b)
        }
    }

    obj.show()
```

### 字符串

 1. 字符串模板
    植入变量，任意折行

```
    //植入变量
    let obj={name:'小明',age:18}
    <!-- alert('我叫:'+obj.name+', 我今年'+obj.age+'岁') -->

    alert(`我叫${obj.name},我今年${obj.age}岁`)

    //折行
    alert(`我叫 
    ${obj.name},
    我今年
    ${obj.age}岁`)
```

2. startsWith和endsWith

```
    //startsWith
    if(phoneNum.startsWith('135')){
        alert('移动')
    }else{
        alert('联通')
    }

    //endsWith
    if(fileName.endsWith('.txt')){
        alert('文本文件')
    }else{
        alert('图片')
    }
```

### 面向对象
```
    //之前的写法
      function Person(name,age){
          this.name=name
          this.age=age
      }

      Person.prototype.showName=function(){
          alert("我叫"+this.name)
      }
      Person.prototype.showAge=function(){
          alert("我"+this.age+"岁")
      }

      var p=new Person('小明',20)
      p.showName()
      p.showAge()
     
     //继承
      function Worker(name,age,job){
          Person.call(this,name,age)
          this.job=job
      }
      worker.prototype=new Person()
      worker.prototype.constructor=Worker
      worker.prototype.showJob=function(){
          alert("我是做"+this.job)
      }
      var w=new Worker('小明',20,'打杂')
      w.showName()
      w.showAge()
      w.showJob()
```

   ```
     //现在的写法
     class Person{
         constructor(name,age){
             this.name=name
             this.age=age
         }
         showName(){
             alert("我叫"+this.name)
         }
         showAge(){
             alert("我"+this.age+"岁")
         }
     }

     let p=new Person('小明',18)
     p.showName()
     p.showAge()

     //继承
     class Worker extends Person{
      constructor(name,age,job){
        //super   超类，父类
        super(name,age)
        this.job=job
      }
      showJob(){
          alert('我是做'+this.job)
      }
     }
     let w=new Worker('小明',18,'打杂')
     w.showName()
     w.showAge()
     w.showJob()
   ```