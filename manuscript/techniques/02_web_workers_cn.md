# Web Workers

[Web workers](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API) allow you to push work outside of main execution thread of JavaScript making them convenient for long running computations and background work.

[Web workers](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API) 允许你将工作推到JavaScript的主线程外以让他们更方便进行耗时计算和后台工作。

Moving data between the main thread and the worker comes with communication-related overhead. The split provides isolation that forces workers to focus on logic only as they cannot manipulate the user interface directly.

在主线程和worker之间移动数据通常会带来通信相关的开销。分割提供一种隔离方案，wokers仅能关注逻辑部分而不能直接处理用户界面。

The idea of workers is valuable on a more general level. [parallel-webpack](https://www.npmjs.com/package/parallel-webpack) uses [worker-farm](https://www.npmjs.com/package/worker-farm) underneath to parallelize webpack execution.

workers的理念在更广泛层面上是具有价值的。[parallel-webpack](https://www.npmjs.com/package/parallel-webpack) 在底层使用[worker-farm](https://www.npmjs.com/package/worker-farm) 来并发执行webpack。

As discussed in the *Build Targets* chapter, webpack allows you to build your application as a worker itself. To get the idea of web workers better, you'll learn how to build a small worker using [worker-loader](https://www.npmjs.com/package/worker-loader).

如之前在*构建目标*章节所讨论，webpack允许你通过worker自身构建应用。为了更好理解web workers这一理念，你将会了解如何使用[worker-loader](https://www.npmjs.com/package/worker-loader) 构建一个小型worker。

## 配置Worker加载器（Setting Up Worker Loader）

To get started, install *worker-loader* to the project:

首先，为项目安装*worker-loader*：

```bash
npm install worker-loader --save-dev
```

Instead of pushing the loader definition to webpack configuration, you can use inline loader definitions to keep the demonstration minimal. See the *Loader Definitions* chapter for more information about the alternatives.

除了将加载器定义推入webpack配置中，你还可以使用内联加载器定义以保持演示最小化。查看*加载器定义*一章了解更多关于该方式的信息。

## 配置Worker（Setting Up a Worker）

A worker has to do two things: listen to messages and respond. Between those two actions, it can perform a computation. In this case, you accept text data, append it to itself, and send the result:

worker必须处理两件事：监听信息和响应。在这两个行为之间，其可以进行计算。在这一例子中，你可以接收文本参数，将其内容附加给自己，而后发送结果：

**app/worker.js**

```javascript
self.onmessage = ({ data: { text } }) => {
  self.postMessage({ text: text + text });
};
```

## 配置Host（Setting Up a Host）

The host has to instantiate the worker and the communicate with it. The idea is almost the same except the host has the control:

host必须实例化worker并与其通信。原理大致相同，除了host能够进行控制：

**app/component.js**

```javascript
import Worker from 'worker-loader!./worker';

export default () => {
  const element = document.createElement('h1');
  const worker = new Worker();
  const state = { text: 'foo' };

  worker.addEventListener(
    'message',
    ({ data: { text } }) => {
      state.text = text;
      element.innerHTML = text;
    }
  );

  element.innerHTML = state.text;
  element.onclick = () => worker.postMessage({ text: state.text });

  return element;
}
```

After you have these two set up, it should work. As you click the text, it should mutate the application state as the worker completes its execution. To demonstrate the asynchronous nature of workers, you could try adding delay to the answer and see what happens.

完成这两个配置之后，其应该生效了。当你点击文本时，应用的状态随着workers执行完成而发生改变。为了演示异步状态下的worker，你可以尝试添加延迟响应并查看发生了什么。

T> [webworkify-webpack](https://www.npmjs.com/package/webworkify-webpack) is an alternative to *worker-loader*. The API allows you to use the worker as a regular JavaScript module as well given you avoid the `self` requirement visible in the example solution.

T> [webworkify-webpack](https://www.npmjs.com/package/webworkify-webpack) 是*worker-loader*的可选项。API允许你使用worker作为常规的JavaScript模块，也能让你避免在示例解决方案中可见的`self`要求。

## 结论（Conclusion）

The important thing to note is that the worker cannot access the DOM. You can perform computation and queries in a worker, but it cannot manipulate the user interface directly.

需要注意的事是worker不能访问DOM。你可以在worker中进行计算和查询，但不能直接操控用户界面。

To recap:

* Web workers allow you to push work out of the main thread of the browser. This separation is valuable especially if performance is an issue.
* Web workers cannot manipulate the DOM. Instead, it's best to use them for long running computations and requests.
* The isolation provided by web workers can be used for architectural benefit. It forces the programmers to stay within a specific sandbox.
* Communicating with web workers comes with an overhead that makes them less practical. As the specification evolves, this can change in the future.

本章回顾：

* Web workers允许你将工作推到浏览器的主线程外。这一分离十分具有价值，尤其如果性能存在问题。
* Web workers无法操作DOM。相反，最好将他们用于耗时计算和请求。
* Web workers提供的隔离被用于架构优势。其强制程序员停留在一个特定的沙盒中。
* Web workers间的通信会产生一定的损耗，使得他们不是很具有实用价值。随着规范的不断发展，这在未来将会改变。

You'll learn about internationalization in the next chapter.

你将会在下一章节了解国际化。
