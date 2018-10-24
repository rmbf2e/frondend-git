## 以git为核心的工作流

### 前言: 
  本次分享中，我将通过介绍一些前端工程体系中的工具，通过示例代码，讲解前端项目工程化与系统化的一些经验。

### 准备: 前端环境

  常见环境部署方法，基本都是在nodejs官网下载exe，msi文件双击安装后直接可用。
  nodejs是一门更新很快的语言，我们常常要维护老项目，老项目依赖某个较早的nodejs版本，随便升级会生成莫名其妙的错误。
  而nodejs的生态环境是总在不断向前推进的，很多工具，第三方包都在前进，舍弃旧环境。
  很多时候如果环境不升级，我们需要手动安装很多老版本工具，而且连文档都查阅都会混乱，我们程序员也有需要保持最新的知识栈的需求。
  此处推荐使用编程语言版本切换工具，ruby有rvm，golang有gvm，nodejs的就是[nvm](https://github.com/creationix/nvm)，即Node Version Manager，官网有多系统安装方式。

安装完毕后，在命令行中使用
```bash
nvm ls-remote
```
查看当前可安装版本，接下来针对想要使用的版本安装即可，比如
```bash
nvm install 10.10.0
```
安装完毕后即可执行
```bash
node -v // v10.10.0
```

💡: 当同时安装多个版本之后，使用命令指定默认版本
```bash
nvm alias default 10.10.0
```

💡: 使用命令切换其他版本，版本号不必一字不差，命令会自动匹配版本。
```bash
nvm use 9.8
node -v // v9.8.0
```

💡: 当我们忘记nvm后面应该用什么命令时，直接执行`nvm`可参阅帮助文档。

💣: 如果使用nvm管理nodejs，最好把当前系统中已有的nodejs卸载，否则容易产生不必要的冲突。

下面将建立一个npm项目，用实际流程展示如何使用前端工具。
创建目录并执行，初始化项目
```bash
yarn init -y
```

💡: 在本示例项目中，全使用`yarn add ***`添加package简化操作，在实际项目中应根据第三方包的实际用途添加`-D`选项将包放到`devDependencies`中。
💡: 在本示例中，我将`./node_modules/.bin`，添加到了环境变量`PATH`中，所以只要我项目中安装了带有命令行的工具，且处于项目目录中，即可执行相应的工具命令。
💡: 在本示例中，统一使用`yarn`代替`npm`，具体实际操作中使用哪一个都可以，但不要混用，因为这两者生成的`node_modules`目录中的结构不同。

### 第一部分: 前端代码检查自动化

很多人都知道`eslint`可以检查`javascript`代码，但知道`eslint`可以自动修正代码，以及如何在工程中使用的人不多。

* 工具[eslint](https://www.npmjs.com/package/eslint)
  安装
  ```bash
  yarn add eslint
  ```

* 工具规则集[eslint-config-airbnb](https://www.npmjs.com/package/eslint-config-airbnb)
  安装
  ```bash
  yarn add eslint-config-airbnb eslint-plugin-jsx-a11y eslint-plugin-react eslint-plugin-import
  ```
  新增文件`.eslintrc.js`
  ```javascript
  module.exports = {
    extends: [
      'airbnb',
    ],
  }
  ```

  此时在命令行应该可以执行`eslint`命令对文件进行检查了。

  💡: eslint文档中的示例使用json格式的配置文件，此处使用`commonjs`规范的js模块文件。在之后的各种工具的配置文件格式，我推荐只要可以，就使用js模块，因为js模块格式可以注释，且能包含逻辑。

  此处使用vscode作为例子讲解与ide的结合。
  参考(https://segmentfault.com/a/1190000009077086)
  步骤：
    1. 在vscode中添加eslint插件。
    2. 在vscode的设置中开启`Auto Fix On Save`。
    3. 在编辑器中查看之前的js文件，应该可以有eslint对应的错误提示，且可以在保存时自动修正大部分错误。

  💡人类逻辑的错误不在修正之列，比如变量乱命名等。

* 工具[prettier](https://www.npmjs.com/package/prettier)，顾名思义，该工具的作用是美化代码。

  💡`prettier`工具有自己的一套默认规则，因此不使用任何配置文件也可直接使用。如果需要可以参照文档配置`prettier`不多的几个规则。

  因为之前已经有`eslint`帮忙修正代码，此处不用安装`prettier`本体，只要安装eslint与prettier的集成即可。
  安装:
  ```bash
  yarn add eslint-config-prettier
  ```
  安装之后，添加`prettier`规则到eslint配置文件。
  ```javascript
  module.exports = {
    extends: [
      'prettier',
      'airbnb',
    ],
  }
  ```
  💡注意扩展配置顺序，后面的扩展规则会覆盖前面的。

  此时`vscode`应该已经可以自动修正并美化js文件了🎉。

    🛠 如果遇到了问题，应按一下几点排除: 
    1. 命令行`eslint`是否已经能正确执行。
    2. IDE是否已经安装与eslint结合的插件，vim使用ale，我的同事用的vscode和sublime都有其自己对应的插件。
    如果我的IDE实在找不到对应的插件，换一个角度考虑一下，该IDE大概可以退休了💀。
    3. 正确安装`eslint`与IDE结合的插件之后，插件的配置修改有没有生效。
    4. 以上几个都没用，google。

  🐾为什么需要自动化代码检查：

    `airbnb`的验证规则，好像是js最严格的检测规则吧。
    经常听小伙伴说，一开启代码验证，报了一大堆错误🐞，根本改不过来，最后只好关掉了，干脆眼不见心不烦(不光是前端，还包括其他各种语言的coder)。
    经常有小伙伴说进步慢，要是人教就带就好了。
    代码检查工具就是最好的老师，规则错误提示是最好的即时反馈机制。
    制定这些检查规则的行业前沿的精英们，通过验证规则，与我们的一个个小项目产生了交集。
    🍵好吧，本着刻意练习的原则，针对每个人或者团队，步子不应一次迈的太大，如果之前没有在项目中严格使用验证规则的情况，
    可以先尝试`eslint-config-recommended`，`eslint-config-react-app`等一些较为宽松规则。



### 第二部分: 前端工程对接口的处理

  首先需要问一下，在座的有没有后端开发人员👷？
  前端人员大多是被动接受后端人员设计的接口规则，或者是因为我们认为这个就是后端人员的本职工作，或者是我们在接口设计上了解不多。
  只有知道什么是好的设计，才能看出问题，进而知道如何改进。
  推荐大家读一些restful设计规范的书，延伸阅读: [RESTful API 最佳实践](http://www.ruanyifeng.com/blog/2018/10/restful-api-best-practices.html)
  网上的一些速成教程太零散，如果有时间还是需要系统的学习，推荐[RESTful API实用指南](https://item.jd.com/26850610638.html)等。

  在前端的开发中，我们要接触各种各样的接口，接口的格式根据开发人员，项目的变化而多种多样。
  例如以下几种接口:
  ```text
  /ciPriceCreateRule
  /updateWorkerInstanceVO
  ```
  名称十分随意，和业务逻辑实际含义相差甚远。

  例如以下几种接口数据格式:
  ```javascript
  { code: 200, data: {...}, message: '...' }
  { code: '200', data: {...}, msg: '...' }
  { code: 'success', data: { entities: [...], page: 1, entityCount: 10 } }
  ```

  由于这种不统一性，前端人员需要在项目中到处写判断
  ```javascript
  fetchRecord () {
    ajax.get('/xxx').then(res => {
      if (res.code === 'success') {
        this.record = res.data
      }
    })
  }
  // 在处理每个需要处理分页的数据的时候，还需要挨个映射分页数据。
  // 例如在组件中，使用antd的Table组件需求的数据格式。
  <Table pagination={{current: data.pageNo, total: data.entityCount}} dataSource={data.entities} />
  ```
  大量重复代码，命名混乱没有成就感😓，而且如果接口有变化，需要改很多地方。

  庆幸的是，大多数项目，在本项目范围中，后端的接口规则还可以做到统一。
  因此我们可以做到，将混乱在我们自己的手中终结。

  我们要做的，首先是去除接口在各处的硬编码。
  将各种接口在项目中以实际业务含义命名，例如:
  创建一个`api.js`文件，统一存放各个接口地址
  ```javascript
  export order = {
    detail: '/fetchOrderInstanceDetail',
    create: '/createWorkerInstanceVO',
    update: '/updateOrderDetail',
    list: '/getWorkerInstanceMember',
  }
  ```

  然后是封装一个ajax工具，使所有请求都经过该工具的过滤，例如使用[fxios](https://github.com/superwf/fxios) (语法类似axios)，大家可以想象成axios的使用。
  ```javascript
  const ajax = {
    get(...args) {
      return fxios.get(...args).then(res => {
        // 在此处使用数据请求成功判断
        if (code === 'success') {
          return res.data
        }
        throw new Error(res.message)
      })
    },
    post ...
    put ...
  }
  export default ajax
  ```
  在`redux/mobx/vuex`或各种其他的前端的各种数据层store中使用时:
  ```javascript
  import ajax from './ajax'
  import { order as api } from './api'

  class Order {
    fetchRecord () {
      // 通过使用api对象的属性增强语义，且避免硬编码
      ajax.get(api.detail).then(record => {
        // 类似的实际获取数据的地方，不需要在每个地方硬编码判断code，直接拿到数据
        this.record = record
      })
    }
  }
  ```

  以分页功能为例，继续在`ajax`工具中添加数据加工功能。

  ```javascript
  // 例如分页数据格式为
  { code: 'success', data: { entities: [...], page: 1, entityCount: 10 } }

  const ajax = {
    get(...args) {
      fxios.get(...args).then(res => {
        if (code === 200) {
          const { data } = res
          // 判断是带有分页结构的数据
          if ('entities' in data && 'page' in data) {
            // 返回符合antd Table组件的数据结构
            return {
              dataSource: data.entityCount,
              pagination: {
                current: data.pageNo,
                total: data.entityCount,
              }
            }
          }
          return data
        }
        throw new Error(res.message)
      })
    },
  }

  // 在store中请求分页数据结构后可直接将数据赋值，不必在每个业务逻辑的store中单独处理
  ```javascript
  {
  ...
    list () {
      ajax.get(api.list).then(list => {
        this.list = list
      })
    }
  }
  // 然后在组件中获取store的数据，该数据已经是加工好符合组件要求的数据结构，直接使用即可
  <Table { ...store.order.list } />
  ```

  总结: 将系统的混乱尽量在自己这一层终结，这种思路主要是，将依赖外部环境的操作，通过一个单点(应考虑单例模式)操作工具老进行统一的过滤和加工。
  在之后的层中直接拿到工整的数据结构。
  在后端系统的开发中，也会遇到需要使用的外部接口，数据库设计混乱的情况，也可参考这种思路处理。

### 第三部分: 启用严格检查的提交格式

  * 通用提交格式规定如下
  
  ```
    type(scope): subject
    // 空行
    body
    // 空行
    footer
  ```

  scope, body和footer可选，type和subject必填。

  * 针对 type，业界通用的选项如下
  >>
    feat: 特性
    build: 构建相关的修改
    ci: 持续集成相关的修改
    fix: bug修正
    docs: 文档
    style: 样式修改
    perf: 性能优化
    refactor: 和特性修正无关的重构，例如重命名
    test: 测试
    revert: 由于上面的某个错误提交，生成恢复代码的一次提交
    chore: 不包含在上面选项中的其他情况
    参考(http://www.ruanyifeng.com/blog/2016/01/commit_message_change_log.html)
    注：不同的lint规则，可选的type可能稍有不同，比如较严格的规则不允许chore这种范围太宽泛的选项。

  * 安装工具之前需要初始化nodejs项目环境，运行：
  ```
  yarn init
  ```
  
  * 工具[commintlint](https://www.npmjs.com/package/commitlint) 检测每次提交的格式核心代码。
  * 工具[commintlint-cli](https://www.npmjs.com/package/@commitlint/prompt-cli) commintlint的命令行工具。

  按照文档，安装工具：
  ```
  yarn add commitlint @commitlint/prompt-cli
  ```
  
  * 规则集[@commintlint/config-conventional](https://www.npmjs.com/package/@commitlint/config-conventional) 检测每次提交的格式。
    该规则较宽松，angular的相对严格，取消了chore选项。
  * 配置文件`commintlint.config.js`，放到项目根目录

  ```javascript
  module.exports = { extends: ['@commitlint/config-conventional'] }
  ```

  * 工具husky/[yorkie](https://www.npmjs.com/package/yorkie)，将commitlint绑定到git的commit-msg提交钩子上，在每次生成提交前调用commitlint检测提交文字格式，不通过验证则无法生成提交。
  此处我使用yorkie作为例子
  安装：
  ```
  yarn add yorkie
  ```

    在package.json文件中添加提交消息验证

  ```javascript
  "gitHooks": {
    "commit-msg": "npx commitlint -E GIT_PARAMS"
  }
  ```

    * git hooks分为服务器hook和本地hook，此处讲的全部都是本地hook。
    * 详细的hooks说明需要看官方文档，想不起来的时候，可以快速看一下当前项目里的`.git/hooks`文件夹，里面的文件就是当前本地git支持的hook，这些文件都是见名知意的。
    * 用相同的思路，还可以在其他的git hooks中注入回调命令，例如pre-commit/pre-push自动运行测试，不通过则阻止提交/推送。

  * 工具[commitizen](https://www.npmjs.com/package/commitizen) 一个命令行下，用交互的方式生成合规的提交格式的工具，对于还不熟悉提交消息格式的人起到自动生成合规消息的作用，可有可无。
  安装过程

  ```
  yarn add commitizen
  yarn add cz-conventional-changelog
  echo '{ "path": "cz-conventional-changelog" }' > .czrc
  ```

  安装完毕之后，即可使用`git-cz`命令代替`git commit`提交。

  * 经过了上面对提交文字的规范，即可达到自动生成changelog的目的。

  * 工具[conventional-changelog-cli](https://www.npmjs.com/package/conventional-changelog-cli)，自动提取提交记录，根据当前package.json中的version版本号，生成当前发布版本的CHANGELOG.md。
    安装：

    ```
    yarn add conventional-changelog-cli
    ```

    添加脚本到package.json

    ```
    "changelog": "npx conventional-changelog -p angular -i CHANGELOG.md -s -r 10",
    ```
    
    conventional-changelog有很多可调整的参数，具体参考[文档](https://www.npmjs.com/package/conventional-changelog-cli)即可。

  * 关于版本号的讲解，一般格式为1.2.3，分为三段，为主版本号，次版本号，修正版本号。

  >>
    主版本号  当前程序经过重构，生成了与之前版本不兼容的api，则主版本号升级。
    次版本号  每次新feature的添加，即升级次版本号。
    修正版本号 每次bug修正引起的升级，即升级修正版本号。
    在首个稳定版本发布之前，会有试用版标识，例如：`2.0.0-beta.1`，`2.0.0-beta.2`等，从beta进化到正式版的第一个版本应为`2.0.0`。

  * 每次发布，需要变更版本号，才需要生成changelog，而不是经常随时生成。
  * 每次发布前需要做两件事。
    * 根据实际情况，升级package.json中的版本号。
    * 运行changelog命令，生成本次发布CHANGELOG.md日志。
    在package.json文件中添加

    ```
    "changelog": "npx conventional-changelog -p angular -i CHANGELOG.md -s -r 10"
    ```

    然后运行

    ```
    yarn changelog
    ```

### 第四部分: 介绍gitflow流程

  * git flow重要概念

  >>
		只有两个长期分支master与develop，其他都是临时分支，随需求增加，随开发完成而删除。
		feature/bugfix从develop分出
		feature/bugfix完成后会自动合并到develop
		release从develop分出
		release完成后会自动合并到develop和master
		hotfix从master分出
		hotfix完成后会自动合并到develop和master

  可结合示例图理解流程，看不懂要多看两遍，并且相信自己一定能看懂🙏。
  ![flow](//github.com/rmbf2e/frondend-git/raw/master/git-model@2x.png)
  图片来自gitflow的推荐教程文档[https://nvie.com/posts/a-successful-git-branching-model/]

  * 按照[安装教程](https://github.com/nvie/gitflow/wiki/Installation)安装完毕后，即可在命令行中使用`git flow`相关命令

  ```
  init      Initialize a new git repo with support for the branching model.
  feature   Manage your feature branches.
  bugfix    Manage your bugfix branches.
  release   Manage your release branches.
  hotfix    Manage your hotfix branches.
  support   Manage your support branches.
  version   Shows version information.
  config    Manage your git-flow configuration.
  log       Show log deviating from base branch.
  ```
    
  * 💡:
    1 在mac或*nix系统中有bash_completion支持。
    2 总是手动输入git flow很不方便，可以做个别名。
      在~/.bashrc中添加
      ```bash
      alias gf='git flow'
      ```
      之后用`gf`命令代替`git flow`


	* 首先在一个git项目中执行git flow init，会在命令行交互询问各个分支的命名，可以添加-d参数全部使用默认值。
		init完成后，会自动停留在develop分支。

	* 添加一个新特性，例如验证用户年龄，`git flow feature start validateUserAge`。
	gitflow会自动添加分支`feature/validateUserAge`，并切换到该分支。
	开发完成后执行`git flow feature finish validateUserAge`。
	gitflow会自动将该分支合并到develop，并删除该分支，表明该特性开发完成。

  * 💡: 经常我们已经对文件进行了更改，才发现并没有切换新分支，此时可以使用暂存命令。

  ```
  git stash
  ```

  将未保存的修改暂存，之后再执行git flow相关命令，待切换到新分支后，执行

  ```
  git stash pop
  ```
  将修改暂存恢复即可

  * 💡: 一个功能模块的提交中有很多次无意义的提交，可以合并为一次提交，使用reset。
  
  ```
  git reset HEAD~3
  git add .
  git commit
  ```
  撤销前三次操作，重新生成一次提交
  __IMPORTANT__🈲：该操作只能在没有`git push`到服务器之前进行，如果已经`push`了，那就保持这个提交记录，千万不要用`--force`💢，会可能覆盖其他人的工作。除非你可以确保`push`的分支，是仅仅为了保存代码的私有分支，别人肯定不会`pull`，才可以在使用`--force`参数。
  __SUMMARY__：鼓励经常`commit`，确保一个`feature`或`bugfix`完整之后再push到服务端。

  * git flow bugfix与feature使用完全相同。


  * git flow release
    运行git flow release start 1.0.0，进入发布1.0.0版本状态，gitflow会自动创建release/1.0.0分支，并切换到该分支。
    运行git flow release finish 1.0.0，自动将1.0.0添加到git的tag，添加一个版本标签，并自动将release/1.0.0分支合并到master与develop分支，
	  完成后自动删除release/1.0.0分支，并自动切换回develop分支。

    其他更多命令与流程理解，可参阅gitflow文档。
    https://nvie.com/posts/a-successful-git-branching-model/
    https://github.com/petervanderdoes/gitflow-avh

	* git flow support 在同时维护多个发行版本时使用，例如当前最新版本已经升级为2.x.x，但1.x.x版本仍然需要维护的情况。
		support的创建会针对某个主版本的tag分支来创建使用。
		一旦建立support分支，则该support分支不再使用git flow流程来管理。

  * 💡: `flow`之间切换时输入的文字是`merge`而不是`commit`信息，不会经过`commitlint`检查。

	* gitflow工具的引入，为项目带来了提交规范的管理的切实可行性，使项目的开发日志更加清晰，并将一些流程带来的分支切换自动化，避免了一些人工管理可能导致的手工错误或遗忘问题。

  * 完整的功能或修正提交，是cherry-pick或版本回滚等基于git的工作流推广的基础。

  * `git push`默认并不提交tag，可添加配置，使`git push`默认提交tag，需git2.4以上版本。
  ```
  git config --global push.followTags true
  ```
  参考https://stackoverflow.com/questions/5195859/how-to-push-a-tag-to-a-remote-repository-using-git

  💡: 可以在自己的用户目录中定义很多git操作的别名简化每次命令输入，例如我使用`git ci`命令代替`git commit`。
  __IMPORTANT__：注意自己设置的别名不要覆盖已有的git子命令。

  * 类似的改良流程有github-flow与gitlab-flow，参考http://www.ruanyifeng.com/blog/2015/12/git-workflow.html
    这两个流程没有对应的命令行工具，只是概念。

  * __没有银弹__🗝: 以上只是一些业界最佳实践介绍，在结合git或不结合git的各种开发流程中，只有经过实践才能总结出适合自己团队的工作流。
