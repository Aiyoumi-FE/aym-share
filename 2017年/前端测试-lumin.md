## 前端测试介绍
测试，作为软件工程的一项重要环节，用来保证项目的正确性，完整性，安全性和可靠性。

前端测试是前端工程化的重要环节，根据测试的粒度可以分为单元测试，功能测试（E2E测试），集成测试。

### 前端测试框架
![](https://mmbiz.qlogo.cn/mmbiz_png/vQ9Te8pPAr8I9ib44agHTZXNGotib4cuciciau5LhqtAdQc2jhSFDZT44rtUicJvT2etW9fibOVkFibZEga65ibCwNVTbA/0?wx_fmt=png)

单元测试
- Mocha
- Jasmine
- Jest

断言库
- chai
- Jest
- expect.js
- should.js

E2E
- webdriverio
- Nightwatch
- Testcafe
- protractor
- casperjs
- dalekjs http://dalekjs.com

集成测试
- Karma

代码覆盖率测试
- Istanbul
- blanketjs
- codecov.io(服务）


自动化测试
- CasperJS
- PhantomJS 
- SlimerJS 

### 单元测试
单元测试用于测试一个模块或接口是否达到预期结果，随着项目规模的增加，函数，方法，变量，模块都在递增，为了保证新的需求或修改不影响原有的结果，同时让程序自动检测结果，我们需要对这些单元制定衡量的标准，并且可以快速检验。

如何做单元测试？
很简单，一个待检验单元要么返回成功pass，要么失败fail，检验的对象应该是接口，方法或模块。
测试的任务包括：
1 模块接口测试;
2 模块局部数据结构测试;
3 模块边界条件测试;
4 模块中所有独立执行通路测试;
5 模块的各条错误处理通路测试。

举个例子：

src/divide.js
```
function divide (x, y) {
    return x / y;
}
module.exports = divide; 
```

```
var divide = require('./src/divide.js');
var result = divide(4, 2);
if (result === 2) {
    alert('pass');
} else {
    alert('fail');
}
```

这里只是举个例子，当然实际的项目中我们不会用alert，而是用到一些测试框架，比如Mocha，Jest ，以mocha为例：

1、 首先安装mocha
```
npm i mocha -g
```

2、 编写测试脚本 test/divide.test.js
```
var divide = require('../src/divide.js');
var expect = require('chai').expect;

describe('除法测试', function() {
    it('9/3 =3', function() {
        expect(divide(9, 3)).to.be.equal(3);
    });
});
```

上面这段代码，就是测试脚本，它可以独立执行。测试脚本里面应该包括一个或多个describe块，每个describe块应该包括一个或多个it块。

describe块称为"测试套件"（test suite），表示一组相关的测试。它是一个函数，第一个参数是测试套件的名称（"除法测试"），第二个参数是一个实际执行的函数。

it块称为"测试用例"（test case），表示一个单独的测试，是测试的最小单位。它也是一个函数，第一个参数是测试用例的名称（9/3 = 3"），第二个参数是一个实际执行的函数。

下面这句是断言：
```
expect(divide(9, 3)).to.be.equal(3);
```
所谓"断言"，就是判断源码的实际执行结果与预期结果是否一致，如果不一致就抛出一个错误。上面这句断言的意思是，调用divide(9, 3)，结果应该等于3。

所有的测试用例（it块）都应该含有一句或多句的断言。它是编写测试用例的关键。断言功能由断言库来实现，Mocha本身不带断言库，所以必须先引入断言库 var expect = require('chai').expect。

3、 运行：
```
$ mocha test/divide.test.js

  除法测试
    √ 9/3 =3

  1 passing (9ms)
```


### 集成测试
集成测试，也叫组装测试或联合测试。在单元测试的基础上，将所有模块按照设计要求组装成为子系统或系统，进行集成测试。

一些模块虽然能够单独地工作，但并不能保证连接起来也能正常的工作。一些局部反映不出来的问题，在全局上很可能暴露出来。

以下两种测试技术是用于集成测试：
1. 功能性测试。使用黑盒测试技术针对被测模块的接口规格说明进行测试。
2. 非功能性测试。对模块的性能或可靠性进行测试。


### E2E测试
E2E，是end to end的缩写。E2E测试的方式是模拟用户行为，控制浏览器执行一系列的页面操作，例如点击按钮、输入文字等等，最后判断结果是否符合预期。

E2E测试属于黑盒测试，作为测试代码编写的人，可以不用关心代码的具体实现，只需关注功能。 也可以把它当成UAT测试的一种。

下面以TestCafe为例介绍E2E测试，下面的例子测试了
> https://devexpress.github.io/testcafe/
 
1、 首先安装testcafe
```
$ npm install -g testcafe
```

2、 编写测试脚本
```
import { Selector } from 'testcafe';

fixture `Getting Started`
    .page `http://devexpress.github.io/testcafe/example`;

test('My first test', async t => {
    await t
        .typeText('#developer-name', 'John Smith')
        .click('#submit-button')
        .expect(Selector('#article-header').innerText).eql('Thank you, John Smith!');
});
```
3、运行
```
$ testcafe chrome tests/*.test.js

 Running tests in:
 - Chrome 59.0.3071 / Windows 10 0.0.0

 Getting Started
 √ My first test


 1 passed (5s)
```


### 覆盖测试
即测试代码的覆盖率，有四个测量维度。
- 行覆盖率（line coverage）：是否每一行都执行了？
- 函数覆盖率（function coverage）：是否每个函数都调用了？
- 分支覆盖率（branch coverage）：是否每个if代码块都执行了？
- 语句覆盖率（statement coverage）：是否每个语句都执行了？


以Istanbul为例，介绍一下基本用法。
> https://github.com/gotwarlost/istanbul#getting-started


### 安装
```
$ npm i istanbul -g
```

divide.js增加除数为0的判断
```
function divide (x, y) {
    if (y === 0) throw new Error("除数不能为0");
    return x / y;
}
```
测试脚本 divide.test.js 在上面的例子上增加了除数为0的分支测试。

```
var divide = require('../src/divide.js');
var expect = require('chai').expect;

describe('除法测试', function() {
    it('9/3 =3', function() {
        expect(divide(9, 3)).to.be.equal(3);
    });

    it("除数为0应该报错", function() {
        expect(function() {
            divide(9, 0);
        }).to.throw("除数不能为0");
    });
});

```


执行代码覆盖测试。

```
$ istanbul cover node_modules/mocha/bin/_mocha
```

得到结果
![](https://i.imgur.com/dQUfgNc.png)

在文件夹coverage\lcov-report下可以查看html格式的覆盖测试结果
![](https://i.imgur.com/ZsECg9w.png)

```
$ istanbul help
```
查看帮助信息，比如需要调整测试覆盖结果的阀值，可以通过check-coverage调整参数。

### 持续集成测试（CI)
持续集成是指能够自动的集成已经提交的代码，直至发布到测试服务器供测试的整个过程。而每次的集成都是通过自动化的构建来验证，包括自动编译、发布和测试，从而尽快地发现集成错误，让团队能够更快的开发。

一个完整的构建系统必须包括：
一个自动构建过程，包括自动编译、分发、部署和测试等。
一个代码存储库，即需要版本控制软件来保障代码的可维护性，同时作为构建过程的源。
一个持续集成服务器。

下面以gitlab-ci为例简单介绍配置持续集成服务器
> http://docs.gitlab.com/runner/register/#gnu-linux

1、安装
可以在任意一台服务器上安装Gitlab Runner，详见http://docs.gitlab.com/runner/install/index.html

2、获取Token
在gitlab服务器上Setting->Runners->Specfic runners可以看到URL和TOKEN。

3、注册一个Runner
在安装了Gitlab Runner服务器上运行：
```
sudo gitlab-runner register
```
根据提示输入在上一步获取的Gitlab Url和Token。

在gitlab上可以看到注册好的runner：
![](https://mmbiz.qlogo.cn/mmbiz_png/vQ9Te8pPAr8I9ib44agHTZXNGotib4cucicF4FPHADiaqwSFiaKRZXDkrQhnJcdg4m3ricIM7Oq5Ytpa1CaFreZIVpWw/0?wx_fmt=png)

4、在项目的根目录添加.gitlab-ci.yml，每一次build会允许test脚本用来检查测试结果。
```
image: node:6

cache:
  paths:
  - node_modules/
  
before_script:
  - npm install --registry=https://registry.npm.taobao.org
  
test:
  script:
  - npm test
```

5、每一次提交推送代码可以看到会有build产生
![](https://mmbiz.qlogo.cn/mmbiz_png/vQ9Te8pPAr8I9ib44agHTZXNGotib4cucicAPxJCWWFYbRjicEMsBfkwSkuqSQNf9XibW9Y86WXM5sLrYUdS9khmkxQ/0?wx_fmt=png)


### 小结
测试作为一项工程，远比上面介绍的更广，更深，测试理论，测试方法也有很多，但是测试的目的都是为了保证我们的项目质量。开发人员有责任写出更健壮，质量更高的代码，所以有必要学习并在项目中使用基本的测试方法理论来避免很多问题，提高效率，减少重复bug，以最低的成本建立和维护测试用例来达到最大的效果，下一讲会结合本文的单元测试，覆盖测试，E2E测试，持续集成测试，系统介绍在前端实际项目中集成运用。