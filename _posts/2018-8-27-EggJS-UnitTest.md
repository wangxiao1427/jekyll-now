---
layout: post
title: EggJS中整合UnitTest!
---

在已有的EggJS项目中整合单元测试

EggJS约定根目录下的test目录为单元测试文件，并且测试文件名称格式为${fileName}.test.js  

* 安装`egg-mock/bootstrap`模块  
  按照官方介绍的示例，在test目录中新增material.test.js文件。   
  (需要注意的一点是，官方示例中说可以在test目录下创建多层子目录，结果本地尝试之后，运行测试，总是报找不到测试脚本。遂看了一下node_modules/egg-bin/node_modules/mocha/bin/_mocha代码发现解析测试脚本不会递归寻找，难道是我安装的模块版本太低...也不是什么大的问题，不影响主流程的可以暂时忽略) 
  <pre>
  <code>
  // test/material.test.js
  it('should echo hello', async () => {
    const ctx = app.mockContext();
    const result = ctx.service.schedule.unitTestGet();
    assert(result === 'hello')
    // assert(result === 'hello1')
  });

  // app/service/schedule.ts
  unitTestGet() {
    return 'hello';
  }
  </code>
  </pre>
  运行`npm test`，可以看到断言成功和失败的结果显示  
  ![_config.yml]({{ site.baseurl }}/images/unittest_schedule.png)
  ![_config.yml]({{ site.baseurl }}/images/unittest_schedule_faild.png)  
  看到失败的结果后有非常想改的欲望。
* controller中的单元测试  
  <pre>
  <code>
  describe('test/material.test.js', () => {
    // test cases
    it('should get a ctx', () => {
      const ctx = app.mockContext();
      assert(ctx.method === 'GET');
      assert(ctx.url === '/');
    });
    it('should say hello', async () => {
      await app.httpRequest()
        .get('/service/api/hello')
        .expect(200);
    });
    it('should post hello', async () => {
      // app.mockCsrf();
      await app.httpRequest()
        .post('/service/api/hello')
        .type('form')
        .send({ content: 'ABC' })
        .expect(200)
        .expect({ content: 'ABC' });
    });
  });
  </code>
  </pre>
  需要通过MockApplication创建一个app对象,然后用这个app对象创建一个模拟的上下文对象。这个上下文对象中httpRequest方法返回一个request对象，通过这个req对象就可以实现接口调用。  
  最终测试结果为：
  ![_config.yml]({{ site.baseurl }}/images/unittest_controller.png)
* Mock一个文件对象  
  <pre>
  <code>
  const fs = require('fs')
  describe('test/service/schedule.test.js', () => {
    // test cases
    it('should read file', async () => {
      mock(fs, 'readFileSync', filename => {
        return 'file content';
      });
      assert(fs.readFileSync('foo.txt') === 'file content');
    });
  });
  </code>
  </pre>
  通过mock模拟fs模块的readFileSync方法，模拟的时候会临时覆盖掉fs的原生readFileSync方法，所以应当在测试完成之后恢复。  
  但是引入 egg-mock/bootstrap 时，会自动在 afterEach 钩子中还原所有的 mock，不需要在测试文件中再次编写恢复代码。  
  测试结果：  
  ![_config.yml]({{ site.baseurl }}/images/unittest_schedule_mock_file.png)  
* 最终使用`npm run cov`统计覆盖率，如图：  
  ![_config.yml]({{ site.baseurl }}/images/unittest_cov.png)  