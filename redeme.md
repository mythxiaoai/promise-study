# 你会得到这些收获
1. 为什么需要promise
2. promise的基础用法
3. 底层实现
4. 最佳实践
5. 未来发展

##项目准备
注：下面说到的代码有些会请求这些路径模拟数据请求，如果前端是用容器启动的话，是在同域下用get请求文件是不会报跨域的。
```
//js/a.json
[{"a":"我是a接口返回的数据"}]
//js/b.json
[{"b":"我是b接口返回的数据"}]
//js/c.json
[{"c":"我是c接口返回的数据"}]
```

## 1.为什么需要promise
>Promise的产生是为了控制异步获取方法的函数，做流程控制，那么问题来了，
1. 为什么会有异步的产生？浏览器是单线程的，在执行长任务需要时间，在于后端交互的时候，网络延迟，执行数据库语句获取数据都需要时间，所以需要异步的产生，让后面的代码执行，等长任务完毕后在运行回调函数。
2. 顺带往深处挖一下，为什么浏览器是单线程的，因为js在设计之初只是为了操作dom与页面做交互的浏览器脚本语言，如果是多线程，那么一个线程在修改dom的值，一个线程在做删除操作，那么那条线程会执行呢？所以说js是单线程的。
3. 总结就是因为单线程会照成页面堵塞所以需要异步，因为异步所以需要流程控制所以需要promise。

**回调函数**  **generator**  **promise**  **async/await**
2. **堵塞**有人说感觉不到堵塞页面测试不了，也感受不到，那么来个不要钱的for循环丢在控制台里面运行下~
``` 
      for (let index = 0; index < 99999; index++) {
        console.log(index);
      }
```
测试方法，新建个独立的窗口，别把当前的文章页面关了，，丢在控制台页面，按回车键运行，然后随意点击页面的交互功能，比如百度查询什么的，会发现，页面超级卡，浏览器卡死请关闭再开，我的卡个几秒钟就好了，原因是什么呢？1.js主线程被占用，当前点击的事件交互进入任务队列，但是主线程正在执行for循环任务，无暇顾及你呗。
3. **回调函数**  会照成地域回调的现象，没错，是现象，只是可读性非常非常非常差而已，还是能够解决实际问题的
```
      //地狱回调例子
      $.ajax({
        type:"get",
        url:"./a.json"
        success(data){
          //...业务逻辑
          $.ajax({
            type:"get",
            url:"./b.json"
            success(data){
              //...业务逻辑
              $.ajax({
                type:"get",
                url:"./c.json"
                success(data){
                  
                }
              });
            }
          });
        }
      });
      //我来拯救下地狱回调   结果将就
      $.ajax({
        type: "get",
        url: "./js/a.json",
        success: successA,
      });

      function successA(data) {
        //...业务逻辑A
        console.log(data);
        $.ajax({
          type: "get",
          url: "./js/b.json",
          success: successB,
        });
      }

      function successB(data) {
        //...业务逻辑B
        console.log(data);
        $.ajax({
          type: "get",
          url: "./js/c.json",
          success: successC,
        });
      }

      function successC(data) {
        //...业务逻辑C
        console.log(data);
      }
```
4. **generator**
```
      //generator
      function* run() {
        yield $.ajax("./js/a.json");
        yield $.ajax("./js/b.json");
        yield $.ajax("./js/c.json");
        return 666;
      }
      let runVar = run();
      console.log(runVar.next().value.then((v) => console.log(v)));
      console.log(runVar.next().value.then((v) => console.log(v)));
      console.log(runVar.next().value.then((v) => console.log(v)));
      console.log(runVar.next());
```
缺点很明显：需要自己执行next方法断点后通过then得到数据，也算是对地域回调的提升吧
5. promise 我们今天的重点~  走起  也是最终的解决方案的必需品
6. promise会自然过渡到async/await   最终解决方案

## 2.promise的基础用法
实例方法有：
.then(resolveFn,rejectFn)  下一步
.catch(rejectFn)  失败的回调
.finally(Fn)  始终会执行  没有回调参数
静态方法有：
- Prmose.all(Array[Promise])  全部完成后返回结果数组
- Prmise.race(Array[Promise])    第一个返回就返回
- Promise.resolve()
  - 参数为空，返回一个状态为fulFilled的Promise
  - 参数是一个跟Primise无关的值，同上，不过fullfilled响应函数会得到这个参数
  - 参数为Promise实例，则返回该实例不做任何修改
  - 参数为thenable，立即执行他的.then（）函数
- Promise.reject()  同上
```
     //模拟ajax
     function ajax(str,time,boolean=true){
       return new Promise((reslove,reject)=>{
         setTimeout(()=>{
          boolean?reslove(str+"返回结果成功"):reject(str+"返回结果失败")
         },time||1000);
       })
     }
     //常规调用
     ajax("a").then(function(v){
       console.log(v);
       return ajax("b")
     }).then(function(v){
       console.log(v);
       return ajax("c")
     }).then(function(v){
       console.log(v);
     });
     
     //all方法   全部ajax执行完返回
     Promise.all([ajax("a",2000),ajax("b",3000),ajax("c",4000)]).then(function(datas){
       console.log(datas)
     })
    
    //race方法  谁最先跑完就返回谁
     Promise.race([ajax("a",2000),ajax("b",3000),ajax("c",4000)]).then(function(data){
       console.log(data)
     })
     //Promise.resolve();  Promise.reject()
     let fn1 = Promise.resolve(666);
     let fn2 = new Promise((resolve)=>resolve(666))
     console.log(fn1);
     console.log(fn2);
```
解释下上面代码：
1. 为了方便测试，我们用setTimeout配合Promise模拟ajax进行网络交互的过程
2. ajax的常规调用写在then里面没毛病，注意，.then需要返回Promise对象注意那个return哈
3. all方法有个坑哦，如果中途一个ajax报错后，结果直接报异常~回调函数不会被执行
4. 最后一点Promise.resolve/reject()  一般正常情况不常用，开发插件为了返回Promise而用，现阶段理解成fn1和fn2等效就行，其他用法可以自行研究。

##3. 底层实现
```
      //发布订阅模式
      class MyPromise {
        constructor(fn) {
          this.fn = fn;
          this._resolveQueue = []; //成功队列
          this._rejectQueue = []; //失败队列
          fn(this.resolve.bind(this), this.reject.bind(this));
        }
        //成功发布
        resolve(val) {
          this._resolveQueue.forEach((cb) => cb(val));
        }
        //错误发布
        reject(val) {
          this._rejectQueue.forEach((cb) => cb(val));
        }
        //依赖收集
        then(resolve, reject) {
          resolve && this._resolveQueue.push(resolve);
          reject && this._rejectQueue.push(reject);
          return this;
        }
      }

      function ajax(v) {
        return new MyPromise((resolve, reject) => {
          setTimeout(() => {
            console.log(Object.prototype.toString.call(this));
            resolve(v);
            //reject("失败")
          }, 2000);
        });
      }
      ajax("成功")
        .then((v) => {
            console.log(v);
          },
          (error) => {
            console.log(error);
          }
        )
        .then((v) => {
            console.log(v);
          },
          (error) => {
            console.log(error);
          }
        );
```
代码解释
1. 如果要写底层实现，第一步要先会用，知道一个api的调用方法才有可能知道它的原理，api的设计也都是从调用层面依次往下深入的。
2. 我们要做的任务是做一个简易的Promise满足基本的.then方法的实现和链式调用，来简单的阐述原理
3. 根据代码执行逻辑，new Promise参数应该是一个立即执行的函数，传入两个带有一个参数的实体函数
4. 我们的目的是在.then的时候的成功函数哪里把参数传过去，但是什么时候运行这些实体函数传过来的参数呢？
5. 两连问，能否串起来呢？关键，关键，关键，既然构造器传过来了有参数的函数，.then又是需要被传参的函数调用，那么他们却一个管，把他们相互黏合。
6. 这里因为new里面是一个异步的，.then可以看做是先运行，先添加回调函数先添加到上上面队列存储着，等待，异步执行完毕setTimeout执行完，执行回调函数，传入值，那么把对应的实参传入队列就可以了。
7. 整体总结，运用发布订阅的模式，每次.then都会去订阅收集依赖，当new Promise的异步函数进入到主线程时候，进行执行发布动作，这里就会执行失败和成功的队列。


## 4. 最佳实践
```
      let page = {
        async init() {
          // this.search()
          //   .then((data) => {
          //     //得到search()返回的结果搞事情
          //     console.log(data);
          //     return this.list(data);
          //   })
          //   .then((data) => {
          //     console.log(data);
          //   })
          //   .catch((res) => {
          //     console.error(res.responseText);
          //   });
          let selectData = await this.search();
          console.log(selectData);
          let list = await this.list(selectData[0].a);
          console.log(list);
        },
        //搜索条件
        search() {
          return $.ajax({
            type: "get",
            url: "./js/a.json",
          });
        },
        //表格初始化
        list(data) {
          return $.ajax({
            type: "get",
            url: "./js/b.json",
            data:{data}
          });
        },
        info() {
          return $.ajax({
            type: "get",
            url: "./js/c.json",
          });
        },
      };
      page.init();
```
代码解释
1. 这里的业务场景很常见，在加载表格的时候，默认要加载一个指定的查询条件的表格数据，需要有前置条件
2. 可以看到async/await的代码的优势，简洁而优雅，需要注意的是await关键字必须是Promise类型，以及需要声明async异步函数，这里也可以看出点端倪，跟generator很相似，其实就是generator进化过来的，俗称async/await是generator的语法糖。

## 5.未来发展
根据上面的代码可以看出  async/await配合Promise的天下啦，但是刚也说到Promise.all有个毛病只要其中一个错误，那么所有结果全部不执行，会报错，ES2020出了新的API去解决这种错误，allSettled，值得注意的是它返回的不再是单纯的数组而是**数组对象**形式。
```
     //模拟ajax
      function ajax(str, time, boolean = true) {
        return new Promise((reslove, reject) => {
          setTimeout(() => {
            console.log("执行" + str);
            boolean ? reslove(str + "返回结果成功") : reject(str + "返回结果失败");
          }, time || 1000);
        });
      }

      Promise.all([ajax("a", 2000), ajax("b", 3000), ajax("c", 4000)]).then(function (datas) {
        //都成功  ["a返回结果成功", "b返回结果成功", "c返回结果成功"]
        //失败就会报错
        console.log(datas);
      });
      Promise.allSettled([ajax("a", 2000), ajax("b", 3000, false), ajax("c", 4000)]).then(function (datas) {
        //返回结果如下，会返回数组对象
        console.log(datas);
      });

      /*
      [
        {
          "status": "fulfilled",
          "value": "a返回结果成功"
        },
        {
          "status": "rejected",
          "reason": "b返回结果失败"
        },
        {
          "status": "fulfilled",
          "value": "c返回结果成功"
        }
      ]
      */
```
整体注意事项：
Promise的存在是为了对异步操作做控制，控制异步的执行顺序而存在的，不要为了用Promise而用Promise，要知其所以然，常规的异步获取数据才是最高效的方式,不要本末倒置。

谢谢观看~，如果对您有用点个赞再走呗，如果有什么疑问欢迎下方评论交流

代码地址：
视频地址：制作中。。。
脑图地址：[http://naotu.baidu.com/file/e53b74be05a5fea8b665f05831a450b4?token=6a85b658c9841c7a](http://naotu.baidu.com/file/e53b74be05a5fea8b665f05831a450b4?token=6a85b658c9841c7a)
