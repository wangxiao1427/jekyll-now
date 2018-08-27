---
layout: post
title: 初步探秘UnitTest!
---

mocha在js中的应用

- 安装模块  
  npm install mocha  
- 测试脚本
  目录结构:
  >mocha_test  
  > &nbsp;&nbsp;|- test  
  > &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|- add.js  
  > &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|- add.test.js  
  
  <pre>
    <code>
      // add.js
      function add(x,y){  
        return x + y;  
      }  
      module.exports = add;  
    </code>
  </pre>

  <pre>
    <code>
      // add.test.js
      var add = require('./add.js');
      var expect = require('chai').expect;
      
      describe('加法函数的测试', function() {
        it('1 加 1 应该等于 2', function() {
          expect(add(1, 1)).to.be.equal(2);
        });
      });
    </code>
  </pre>
  
  `describe`块称为“测试套件”，表示一组相关的测试。  
  `it`块称为“测试用例”，表示一个单独的测试，是测试的最小单位  
  `*.test.js`文件是测试用例代码文件，mocha会从*.test.js文件中执行测试用例代码  
- 运行  
  `cd ./mocha_test`  
  `$mocha add.test.js`
  * 如果 $mocha命令没有在系统环境变量path中配置路径时会报错，此时应在环境变量path中添加路径`C:\CODE\mocha-demos\node_modules\.bin`，bin是安装的mocha模块文件可执行路径。  
  * 默认的测试用例脚本放在test目录下，如果有大量脚本需要同时执行，可以执行脚本：
  `mocha`或`npm test`(此时package.json中应配置scripts.test = "mocha")
  * 还支持运行通配符以及各种参数  
  * 异步测试
  * 测试用例钩子  
  before/after/beforeEach/afterEach  

  before:在本区块的所有测试用例之前执行  
  beforeEach:在本区块的每个测试用例之前执行   
  其他类似。
  * 浏览器测试  
  1\.初始化目录  `mocha init test-dir`    
  2\.在tests.js中添加单元测试脚本  
  3\.在index.html中假如断言库chai.js、引用需要测试的代码文件  
  4\.在浏览器打开index.html，会执行测试脚本。
  * 生成格式化文档  
  1\.markdown  
  `mocha --recursive -R markdown > spec.md`  
  2\.html  
  `mocha --recursive -R doc > spec.html`
