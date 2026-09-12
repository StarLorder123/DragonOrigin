# 1.  什么是事件循环
- 事件循环是指Node.js执行非阻塞I/O操作。虽然JavaScript是单线程的，但大多数的内核是多线程的，因此Nodejs会尽可能地将操作装载到系统内核。
- 因此，他们可以处理在后台执行的多个操作。当其中一个操作完成时，内核会告知Nodejs，以便Nodejs会将相应的回调函数添加到轮询队列中以最终执行。
- # 2.  事件循环的流程
- 当Node.js启动时会初始化 **event loop **, 每一个 **event loop **都会包含按如下顺序六个循环阶段：
	- 1. **timers**** 阶段**: 这个阶段执行 setTimeout(callback) 和 setInterval(callback) 预定的 callback;
	- 2. **I/O callbacks 阶段**: 此阶段执行某些系统操作的回调，例如TCP错误的类型。 例如，如果TCP套接字在尝试连接时收到 ECONNREFUSED，则某些* nix系统希望等待报告错误。 这将操作将等待在 **I/O回调阶段** 执行;
	- 3. **idle, prepare 阶段**: 仅node内部使用;
	- 4. **poll 阶段**: 获取新的I/O事件, 例如操作读取文件等等，适当的条件下node将阻塞在这里;
	- 5. **check 阶段**: 执行 setImmediate() 设定的callbacks;
	- 6. **close callbacks 阶段**: 比如 socket.on(‘close’, callback) 的callback会在这个阶段执行;
	  
	  ![](https://cdn.nlark.com/yuque/0/2023/png/2713067/1697465395393-aa2fc50d-88fb-4011-809a-255258789990.png)
- 在Node中，同样存在宏任务和微任务，与浏览器中的事件循环相似
- 微任务对应有：
	- next tick queue：process.nextTick
	- other queue：Promise的then回调、queueMicrotask
- 宏任务对应有：
	- timer queue：setTimeout、setInterval
	- poll queue：IO事件
	- check queue：setImmediate
	- close queue：close事件
- 其执行顺序为：
	- next tick microtask queue
	- other microtask queue
	- timer queue
	- poll queue
	- check queue
	- close queue
- # 3.  循环阶段详细内容
- timer阶段：一个timer指定一个下限时间而不是准确时间，在达到这个下限时间后执行回调。在指定时间过后，timers会尽可能早地执行回调，但系统调度或者其它回调的执行可能会延迟它们。
	- 注意：技术上来说，poll 阶段控制 timers 什么时候执行。
	- 注意：这个下限时间有个范围：[1, 2147483647]，如果设定的时间不在这个范围，将被设置为1。
- I/O callbacks阶段：这个阶段执行一些系统操作的回调。比如TCP错误，如一个TCP socket在想要连接时收到ECONNREFUSED, 类unix系统会等待以报告错误，这就会放到 I/O callbacks 阶段的队列执行. 名字会让人误解为执行I/O回调处理程序, 实际上I/O回调会由poll阶段处理。
- poll阶段：poll 阶段有两个主要功能：（1）执行下限时间已经达到的timers的回调，（2）然后处理 poll 队列里的事件。 当event loop进入 poll 阶段，并且 没有设定的 timers（there are no timers scheduled），会发生下面两件事之一：
	- 如果 poll 队列不空，event loop会遍历队列并同步执行回调，直到队列清空或执行的回调数到达系统上限；
	- 如果 poll 队列为空，则发生以下两件事之一：
		- 如果代码已经被setImmediate()设定了回调, event loop将结束 poll 阶段进入 check 阶段来执行 check 队列（里面的回调 callback）。
		- 如果代码没有被setImmediate()设定回调，event loop将阻塞在该阶段等待回调被加入 poll 队列，并立即执行。
	- 但是，当event loop进入 poll 阶段，并且 有设定的timers，一旦 poll 队列为空（poll 阶段空闲状态）： event loop将检查timers,如果有1个或多个timers的下限时间已经到达，event loop将绕回 timers 阶段，并执行 timer 队列。
- check阶段：这个阶段允许在 poll 阶段结束后立即执行回调。如果 poll 阶段空闲，并且有被setImmediate()设定的回调，event loop会转到 check 阶段而不是继续等待。
	- setImmediate() 实际上是一个特殊的timer，跑在event loop中一个独立的阶段。它使用libuv的API 来设定在 poll 阶段结束后立即执行回调。
	- 通常上来讲，随着代码执行，event loop终将进入 poll 阶段，在这个阶段等待 incoming connection, request 等等。但是，只要有被setImmediate()设定了回调，一旦 poll 阶段空闲，那么程序将结束 poll 阶段并进入 check 阶段，而不是继续等待 poll 事件们 （poll events）。
- close callbacks阶段：如果一个 socket 或 handle 被突然关掉（比如 socket.destroy()），close事件将在这个阶段被触发，否则将通过process.nextTick()触发
- # 4.  示例解析
  
  ```
  async function async1() {
    console.log('async1 start')
    await async2()
    console.log('async1 end')
  }
  
  async function async2() {
    console.log('async2')
  }
  
  console.log('script start')
  
  setTimeout(function () {
    console.log('setTimeout0')
  }, 0)
  
  setTimeout(function () {
    console.log('setTimeout2')
  }, 300)
  
  setImmediate(() => console.log('setImmediate'));
  
  process.nextTick(() => console.log('nextTick1'));
  
  async1();
  
  process.nextTick(() => console.log('nextTick2'));
  
  new Promise(function (resolve) {
    console.log('promise1')
    resolve();
    console.log('promise2')
  }).then(function () {
    console.log('promise3')
  })
  
  console.log('script end')
  ```
- 分析过程：
	- 先找到同步任务，输出script start
	- 遇到第一个 setTimeout，将里面的回调函数放到 timer 队列中
	- 遇到第二个 setTimeout，300ms后将里面的回调函数放到 timer 队列中
	- 遇到第一个setImmediate，将里面的回调函数放到 check 队列中
	- 遇到第一个 nextTick，将其里面的回调函数放到本轮同步任务执行完毕后执行
	- 执行 async1函数，输出 async1 start
	- 执行 async2 函数，输出 async2，async2 后面的输出 async1 end进入微任务，等待下一轮的事件循环
	- 遇到第二个，将其里面的回调函数放到本轮同步任务执行完毕后执行
	- 遇到 new Promise，执行里面的立即执行函数，输出 promise1、promise2
	- then里面的回调函数进入微任务队列
	- 遇到同步任务，输出 script end
	- 执行下一轮回到函数，先依次输出 nextTick 的函数，分别是 nextTick1、nextTick2
	- 然后执行微任务队列，依次输出 async1 end、promise3
	- 执行timer 队列，依次输出 setTimeout0
	- 接着执行 check 队列，依次输出 setImmediate
	- 300ms后，timer 队列存在任务，执行输出 setTimeout2
- 结果
  
  ```
  script start
  async1 start
  async2
  promise1
  promise2
  script end
  nextTick1
  nextTick2
  async1 end
  promise3
  setTimeout0
  setImmediate
  setTimeout2
  ```
-