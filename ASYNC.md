# ASYNC



todo: 1. async.series源码解析

[TOC]

 Async是一个流程控制工具包，提供了直接而强大的异步功能。基于Javascript为Node.js设计，同时也可以直接在浏览器中使用。 

## async.series

所有方法会按顺序从上往下执行，所有结果会传递到最终的results里

格式：async(tasks, callback)

tasks里的方法都会运行一遍，如果某个方法出错，error会被传递到callback，后续函数不能执行。task既可以是array，也可以是object。

当task为array：

```javascript
var async = require('async')
async.series(
  [
    function(callback) {
        /**
        * 当callback第一个参数err为null时，继续执行下面的函数；
        * 当为false时，不执行最后的callback；当为err不为空时，
        * 不执行后续的函数，并传递err到最后callback中的err;
        * 营销系统中参数为false时，效果同参数为null时
        */
      callback(null, 'this one', 'this one more')
    },
    function(callback) {
      callback(null, 'this two')
    }
  ],
    //最后的函数
  function(err, results) {
    console.log(results)
  }
)
```

当task为object：

```javascript
var async = require('async')
async.series(
  {
    one: function(callback) {
      callback(null, 'this one')
    },
    two: function(callback) {
      callback(null, 'this two')
    }
  },
  function(err, results) {
    console.log(results)
    console.timeEnd('series')
  }
)

```



## async.auto

方法模板

```javascript
async.auto({
    function_name1: function(callback) {
        ...
    },
    function_name2: function(callback) {
        ...
    },
    function_name3: [
        'function_name1',
        'function_name2',
        function(results, callback) {
            ...
        }
    ]
})
```

function_name1和function_name2之间无依赖，运行顺序不确定；

function_name3的依赖为function_name1和function_name2，故在运行完这两个方法之后才会运行function_name3

参数存键值对，形式为 

``` javascript
function_name: function(callback) {
    /**
    * arg1: true结束async；false向下传递
    * arg2, arg2: 多余的参数被当做result传入下一个function
    */
    callback(arg1, arg2, arg3);
}
```

传入的result形式为

```javascript
{
    function_name: [arg2, arg3]//传入值为array
}
```

如果只有arg2，则result形式为

```javascript
{
    function_name: arg2//传入值为string
}
```



### auto例子

```javascript
// import async from './node_modules/async'
//
var async = require('async')
async.auto(
  {
    get_data: function(callback) {
      //当task是function只能有一个参数就是回调函数
      console.log('in get_data')

      callback(null, 'data', 'converted to array') //第一个参数：true就会直接结束async，false向下传递。剩下参数，会被当做result中“get_data”的value，如果一个值就是string类型，多个值就是array
    },
    make_folder: function(callback) {
      //这个函数会和上个函数是同步执行（因为当前没有耗时操作，因此执行会从上到下）
      callback(null, 'folder')
    },
    write_file: [
      'get_data',
      'make_folder',
      function(results, callback) {
        //当函数的task是数组，那么function必须在前两个task执行完毕（就是第二个task调用callback的时候执行）再执行，也只有这种时候，这个function才有result这个参数
        console.log('in write_file', JSON.stringify(results))
        // once there is some data and the directory exists,
        // write the data to a file in the directory
        callback(null, 'filename')
      }
    ],
    email_link: [
      'write_file',
      function(results, callback) {
        console.log('in email_link', JSON.stringify(results))
        // once the file is written let's email a link to it...
        // results.write_file contains the filename returned by write_file.
        callback(null, { file: results.write_file, email: 'user@example.com' })
      }
    ]
  },
  function(err, results) {
    //跟所有task同步执行，之前有耗时操作，必然先执行这个函数，但是之前如果有函数回调函数参数为false，那么会直接执行此函数
    console.log('err = ', err)
    console.log('results = ', results)
  }
)
```

