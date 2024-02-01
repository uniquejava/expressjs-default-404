## Getting started
```shell
pnpm install -g express-generator 
pnpm install nodemon -g
express -e --git expressjs-default-404
cd expressjs-default-404 && pnpm install

pnpm start
```

## How express.js handle 404(NOT Found Request) and why our custom header is overridden

如果没有合适的路由中间件处理用户请求, express 会使用 `node_modules/.pnpm/finalhandler@1.1.1/node_modules/finalhandler/index.js` 的 send 方法处理.

它会生成一个默认的 error 页面, 然后设置一些 header, 其中包括两个有关 security的 header:

```js
// security headers
res.setHeader('Content-Security-Policy', "default-src 'self'")
res.setHeader('X-Content-Type-Options', 'nosniff')
```

他们会覆盖我们的 header 设置. 解决办法, 就是加一个 catchAll 的路由中间件, 如下:

```js
// catch 404 and forward to error handler
app.use(function(req, res, next) {
  next({status: 404, message: "Oops, not found."});
});
```

这段代码做了两件事:
1. catchAll route, 也可以写成 `app.use("*", function(req, res, next))`, 当前面middleware无法匹配request path或是处理后没有结束/终止请求, 则会调用这里的 catchAll route.
2. 只要给 next 传递一个非空参数, 请求就会 forward to error handler, 其中给 next 传递的参数即错误对象
3. 一旦 next 有参数, 就会跳转到带 err 参数的 middleware: `app.use(function(err, req, res, next)`

expressjs(finalhandler) 默认的 Error 页面 代码如下: 

```js
function createHtmlDocument (message) {
  var body = escapeHtml(message)
    .replace(NEWLINE_REGEXP, '<br>')
    .replace(DOUBLE_SPACE_REGEXP, ' &nbsp;')

  return '<!DOCTYPE html>\n' +
    '<html lang="en">\n' +
    '<head>\n' +
    '<meta charset="utf-8">\n' +
    '<title>Error</title>\n' +
    '</head>\n' +
    '<body>\n' +
    '<pre>' + body + '</pre>\n' +
    '</body>\n' +
    '</html>\n'
}
```



