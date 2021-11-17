---
layout: post
author: shopkeeper
categories: nodejs
---
&emsp;&emsp;在CardiacKeeper这个项目中，NLPServer部署了一个Koa2的应用服务器，用于接收前端的QA请求或解析文章关键字请求，需要运行python代码。本篇博客用于记录我在nodejs中执行python的解决方案。

&emsp;&emsp;我使用了nodejs的`child_process`模块，在Typescript中引用之的方式为
```typescript
import * as child from "child_process";
```
&emsp;&emsp;[child_process](https://nodejs.org/api/child_process.html)有四种创建子进程的方式。我使用的是`spawn`方法，该方法的第一个参数是指令类型，可以填入"nodejs""python"等等；第二个参数是需要执行指令的入参，类型是字符串数组，且如果是文件的话第一个元素必须指明执行文件的位置；第三个参数是一些选项设置。

```typescript
const childProcess = child.spawn("python", [
  "src/test/echo.py",
  question,
  ...contexts
]);
```
&emsp;&emsp;该子进程通过回调的方式捕获stdout、stderr与进程结束：
```typescript
childProcess.stdout.on('data', (data) => {
  console.log(`stdout: ${data}`);
});

childProcess.stderr.on('data', (data) => {
  console.error(`stderr: ${data}`);
});

childProcess.on('close', (code) => {
  console.log(`child process exited with code ${code}`);
});
```
&emsp;&emsp;由于执行python的函数需要在koa2的中间件中同步执行，所以上述函数需要改造成async/await的
```typescript
import * as child from "child_process";

async function spawnChildQa(question: string, contexts: string[]) {
  const childProcess = child.spawn("python", [
    "src/test/echo.py",
    question,
    ...contexts
  ]);

  let data = "";
  for await (const chunk of childProcess.stdout) {
    data += chunk;
    data = data.trim();
  }
  let error = "";
  for await (const chunk of childProcess.stderr) {
    error += chunk;
  }
  const exitCode = await new Promise((resolve) => {
    childProcess.on("close", resolve);
  });

  if (exitCode) {
    throw new Error(`subprocess error exit ${exitCode}, ${error}`);
  }
  return data;
}

export { spawnChildQa };
```
&emsp;&emsp;在koa2的中间件中使用：
```typescript
await spawnChildQa(question, contexts).then(
  (data) => {
    ctx.body = {
      answer: data,
      status: 0
    };
  },
  (err) => {
    ctx.body = {
      answer: err,
      status: 1
    };
  }
);
```