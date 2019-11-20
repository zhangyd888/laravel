
### **1. 项目结构概述**
* 该项目采用了Yii2高级应用模版，由/frontend目录作为应用的主目录，其中/frontend/web/index.php作为应用的入口文件。
* 该项目舍弃了backend目录，后端服务应用不应与前端处于同一项目，在非规范非自动化健全的情况下，容易导致后端前端复用函数，类及模块，相互调用依赖的情况。会增加每次版本迭代的风险，当迭代后端服务应用时可能导致前端应用出现异常。
* 该项目舍弃并禁止使用Yii2的environments模式，该模式为非常规的环境配置模式。在该模式下，每个环境都有独立的配置文件及目录文件权限配置。但这并非最理想的方式，每个环境拥有独立的配置文件是相当危险存在隐患的方式，容易导致多个环境的配置文件配置项差异导致生产意外, 这种方式会架空各环境的测试，因各环境仅会测试该环境自身所使用的配置文件。同时，文件/目录权限配置应当极力避免由应用发起，出于生产环境安全的考虑，生产环境不应当赋予应用拥有修改系统权限的权利。
* 该项目在用了PHP ENV的方式来区分各环境不同的配置参数。同时，隔离敏感配置(git上不应存储生产环境的环境变量文件)，比如密码，私钥等。
* (可选) 该项目采用了dotenv(成熟的第三方环境变量库), 使用该库来配置环境变量，让应用保持多环境单配置文件减少生产事故发生的几率。同时，隔离敏感配置(git上不应存储生产环境的环境变量文件)，比如密码，私钥等。
* 该项目包含了static静态资源目录，用于存放静态html, css, js等。
* 该项目不包含vendor目录，部署时，项目的依赖库依靠, composer.json进行下载校验；composer.lock来锁定各依赖库的版本。本地部署可使用 composer install进行依赖库的安装。 注意： vendor及composer.phar不要提交进版本库。
* vendor依赖库在生产环境部署时，需要从 vendor 管理中心获取 composer.json 与 composer.lock 进行安装校验。 各Yii2项目所增加或升级的依赖库都需要向 vendor 管理中心提出申请，通过后最终才会在生产环境部署。 vendor 管理中心维护了Yii2项目总的 vendor 依赖库，同时维护了各项目所需要的 vendor 依赖库，在生产发布时，仅会安装各项目需要的依赖，保证 vendor 最小化。 （统一管理 vendor 是为了维持各项目间使用统一的依赖库，统一依赖库版本）

***

### **2. 项目结构说明**
#### **2.1. 项目目录摘要**
    + common                                  该目录用来存放一些公共的类库，与业务无关的工具类
    + console                                 控制台脚本目录
    - db                                      该目录用来存放一些需要执行的sql, 用于自动化部署及快速本地部署还原项目基础数据库结构及数据
          0001_create_database.sql             建库脚本
          0002_create_tables.sql               建表脚本
          0003_alter_tables.sql                修改表结构脚本
          0004_insert_datas.sql                插入基础数据脚本
    - frontend                                应用主目录
        + behaviors                            行为目录
        + biz                                  业务层目录
        + config                               配置目录
        + controllers                          控制器目录
        + filter                               过滤器目录
        + models                               模型目录
        + views                                视图目录
        + vos                                  Variable Object folder
        - web
              index.php                         网站入口
    - static                                  (后台项目需要)该目录用来存放静态资源, 比如css, js或静态html
        + css                                  CSS文件目录
        + images                               图片文件目录
        + plugs                                其它前端插件目录
                                              (为了降低维护成本，不要将前端插件中的CSS JS拆分出来，
                                              统一维持插件原有目录结构，并放在该目录下)
        + scripts                              JS脚本目录
      codeception.dist.yml                    全局自动化测试codeception公共配置
      composer.json                           依赖库描述文件(用来生成/下载/校验依赖库)
      composer.lock                           依赖库版本描述锁定文件(用来锁定依赖库的版本)
      env.dist                                环境变量模版文件(用于定义环境变量模版)
      env.php                                 dotenv启动文件
      yii                                     Yii定时任务脚本执行入口
      README.MD

<br>

#### **2.2. 公共文件/公用文件代码目录(common目录)规范说明**
* 由于不再建议将后台，API，业务前台都放在一个项目中，故该目录中不再需要公共配置、视图、模型等类
* 各工具类不建议做成Yii2的组件形式（原因：并不是每次请求都需要用到这些组件，可动态加载组件？！ 另外，写成Yii组件的方式不方便后期移植其它框架及抽取，比如一些不使用框架或其它框架的PHP项目或框架更换）

<br>

#### **2.3. 数据库脚本(db目录)规范说明**
* 该目录用来存放一些需要执行的sql, 用于自动化部署及快速本地部署还原项目基础数据库结构及数据
* 该目录中必须包含下列文件
  0001_create_database.sql          建库脚本
  0002_create_tables.sql            建表脚本
  0003_alter_tables.sql             修改表结构脚本
  0004_insert_datas.sql             插入基础数据脚本
  每个sql脚本必须保证可重复执正确执行，不报错，不产生重复数据

<br>

#### **2.4. 环境变量文件说明**
* env.dist 为环境变量模版文件，项目中使用
* env.php 为dotenv的启动文件

  ##### **2.4.1. env.php配置**
  参照下列配置
  ```php
  <?php

  $dotenv = \Dotenv\Dotenv::create(__DIR__);
  $dotenv->load();

  defined('YII_DEBUG') or define('YII_DEBUG', getenv('YII_DEBUG') === 'true');
  defined('YII_ENV') or define('YII_ENV', getenv('YII_ENV') ?: 'prod');
  defined('YII_ENABLE_EXCEPTION_HANDLER') or define('YII_ENABLE_EXCEPTION_HANDLER', getenv('YII_EXCEPTION_HANDLER') === 'true');

  ```

  ##### **2.4.2. env.dist说明**
  该文件用来配置不同环境中，项目中一些根据环境而不同的变量参数的模版。<font style="color: #f00;">(该文件需要提交进版本库)</font>
  参照下列配置
  ```dist
  # # # # # # # # # # # # # #
  # Framework
  # ---------------
  # Yii框架环境变量
  YII_DEBUG                  = true                                                         # (必须)Yii debug标记, true为开启, false为关闭
  YII_ENV                    = dev                                                          # (必须)Yii 环境变量, dev为开发环境
  YII_EXCEPTION_HANDLER      = true                                                         # (必须)
  # # # # # # # # # # # # # #

  # # # # # # # # # # # # # #
  # Web
  # ---------------
  # Web应用环境变量
  WEB_DOMAIN                 = //xxxx.zhan.com                                              # 网站域名
  WEB_STATIC_DOMAIN          = //xxxx-static.zhan.com                                       # 网站静态资源域名

  # LOG
  LOG_TRACE_LEVEL            = 3                                                            # 日志追踪级别
  LOG_LEVELS                 = error,warning,info                                           # 日志打印级别
  LOG_FILE_PATH              = /var/log/php/xxxx.zhan.com/                                  # 日志存放位置
  LOG_FILE_SIZE              = 1048576                                                      # 日志文件大小限制(单位：MB)
  # # # # # # # # # # # # # #

  # # # # # # # # # # # # # #
  # DataBase
  # ---------------
  # 数据库环境变量
  DB_DSN_MASTER              = mysql:host=192.168.41.232;port=3306;dbname=xxxx              # 数据库(主库)数据源(Data Source Name)
  DB_USER_MASTER             = dev                                                          # 数据库(主库)登录用户名
  DB_PWD_MASTER              = 3e4ziUq9qBV1                                                 # 数据库(主库)登录密码

  DB_DSN_SLAVE1              = mysql:host=192.168.41.232;port=3306;dbname=xxxx              # 数据库(从库1)数据源(Data Source Name)
  DB_USER_SLAVE1             = dev                                                          # 数据库(从库1)登录用户名
  DB_PWD_SLAVE1              = 3e4ziUq9qBV1                                                 # 数据库(从库1)登录密码
  # # # # # # # # # # # # # #
  ```
  各环境本地使用时，从env.dist中复制一份并命名为 .env 作为本地环境变量配置。 在本地环境变量配置中，修改符合本地环境的参数后，可运行应用。
  <font style="color: #f00;">生产环境的 .env 由运维管理。 运维每次在部署时，先检查 env.dist 文件是否存在修改，有修改的情况下，将对应修改项添加到运维管理的.env文件中，并完成配置。</font>
  <font style="color: #f00;">.env 本地环境变量配置，需要添加到 .gitignore 中, 不要提交进版本库。</font>
  php的配置文件中，可通过 <code>getenv('环境变量名')</code> 方式来使用。

***
### **3. 自动化测试说明**
#### **3.1. 使用codeception作为自动化测试工具**
* codeception 是一个相对成熟并持续维护的全栈自动化测试框架，它可以建立自动化的单元测试，功能测试，集成测试，验收测试并产出测试报告。

<br>

#### **3.2. 使用自动化测试规范**
* 开发必须要在项目中创建单元测试，来保证长期项目迭代中自测的覆盖度，减少在迭代过程中造成对现有功能的损害。
* 开发过程中，原则上每个函数，方法都应该有其对应单元测试用例。
* 开发提测前，必须执行自动化测试，确保提测时所有单元测试用例是通过的。
* 测试期间，每发现一个BUG，在修复BUG过程中，就应该对应的修改单元测试用例及补充单元测试中对应该BUG缺失的单元测试用例，以保证长期迭代中有测试用例覆盖到该BUG。
* 每次提交测试版本时，必须执行并确保所有单元测试用例是通过的。
* 功能测试，可选。 codeception可模拟一个网络请求来进行功能测试，测试包括了接口，页面，交互等。 由于相较于单元测试，编写该测试用例相对复杂，特别是针对页面的测试用例。所以，为了保证更有效率的开发及快速提交上线，对于自动化功能测试不做要求。(另外一个原因，过于复杂的测试用例编写本身也容易出现测试用例本身的BUG，导致测试结果存在误导性，特别是复杂的，长期迭代的，会拥有悠久历史的项目，更容易造成混淆)
* 各测试用例，必须支持多环境执行，包括本机自测执行，测试环境执行，预发环境执行等。
* 各测试用例，如果需要生成测试数据，推荐使用独立的数据源进行，避免对公共库的侵扰(可使用项目中db目录下的脚本，快速自动创建库表及基础数据)。 若客观条件只能在公共库执行自动化测试时，应谨慎使用数据清理，还原执行前的数据，避免过度清理或清理行为存在BUG而对公共库数据造成损害。
* (具体如何在项目中应用codeception及如何编写用例与编写规范，请继续阅读下列内容)

<br>

#### **3.3. 在项目中安装自动化测试(codeception)**
* Yii默认的 vendor 包中已带了 codeception，所以不必自行安装，只需要根据 composer.json 在本地部署 vendor 即可。

<br>

#### **3.4. 生成测试用例模版**
* 如果是新建项目时，请按下列步骤生成测试目录及基础配置(Linux)
  * 在项目目录下执行命令:  <code>php vendor/bin/codecept init unit</code>
  * 根据命令行出来的提示，按下列进行
      *Where tests will be stored?(测试文件目录存储的位置)*
      输入： tests  (命令执行目录下的tests目录，会自动创建)

      *Enter a default namespace for tests(测试默认的命名空间)*
      输入： tests  (默认命名空间为 tests\\ 为根)

      *Codeception provides additional features for integration tests*
      *Like accessing frameworks, ORM, Database.*
      *Do you wish to enable them? (y/n)*
      输入： y
  * 上述命令执行完后，会根据上述输入的回答在项目中初始化单元测试目录与基础配置
  * 打开项目目录下自动生成的 codeception.yml 文件, 按如下编辑
    ```yml
    namespace: tests
    suites:
        unit:
            path: unit
            actor: UnitTester
    modules:
        config:
            Yii2:
                configFile: 'config/main.php'
    settings:
        bootstrap: _bootstrap.php                            # 框架的引导程序(之后会提供改文件的模版)
        shuffle: true
        lint: true
        memory_limit: 512M
    paths:
        tests: tests                                         # 测试文件存放路径
        output: tests/_output                                # 测试报告输出路径
        support: tests/_support                              # 测试支持类路径
        data: tests
    coverage:
        enabled: true                                        # 是否包含覆盖率测试
        whitelist:                                           # 纳入覆盖率计算的代码(通常需要将除第三方库外的所有代码都加入进来)
            include:
                - common/components/*
                - frontend/biz/*
                - frontend/controllers/*
                - frontend/models/*
                - frontend/web/*
                - frontend/config/*
    ```
  * 编辑完成后，将 codeception.yml 复制一份并命名为 codeception.dist.yml 作为公共模版配置(根据codeception, 会把dist与本地yml合并，相同键以本地yml为准)
  * 在 .gitignore 中，添加忽略项 codeception.yml
  * 在 .gitignore 中, 添加忽略项 tests/\_output/

<br>

#### **3.5. 创建单元测试用例**
  * 在项目目录下执行命令:  <code>php vendor/bin/codecept g:test unit 目录/测试用例名</code>
    例如，要对项目 common/components/redis/ 目录下的Session.php创建测试用例，则使用下列命令
    <code>php vendor/bin/codecept g:test common/components/redis/Session</code>
    命令执行后，会在 tests/unit/ 目录下创建 common/components/redis/ 目录并生成 SessionTest.php 文件

<br>

#### **3.5. \_bootstrap.php文件配置**
  * 在执行测试用例前，由于我们采用了Yii2框架，所以执行用例时，就必须按Yii2框架的要求进行初始化，而这些启动信息就需要配置在\_bootstrap.php中
  我们可以在 tests/unit/ 创建 \_bootstrap.php，并按下列进行配置
  ```php
  <?php
  defined('YII_APP_BASE_PATH') or define('YII_APP_BASE_PATH', __DIR__ . '/../../');

  require(__DIR__ . '/../../vendor/autoload.php');
  require(__DIR__ . '/../../env.php');
  require(__DIR__ . '/../../vendor/yiisoft/yii2/Yii.php');
  require(__DIR__ . '/../../frontend/config/bootstrap.php');

  $config = require(__DIR__ . '/../../frontend/config/main.php');
  (new yii\web\Application($config)); 
  ```

#### **3.5. 执行测试用例**
  * 执行指定的测试用例类
    <code>php vendor/bin/codecept run unit common/components/redis/SessionTest</code>
  * 执行指定测试用例的方法
    <code>php vendor/bin/codecept run unit common/components/redis/SessionTest:testGet</code>
  * 执行全部单元测试用例
    <code>php vendor/bin/codecept run unit</code>
  * 执行全部单元测试用例，并生成报告
    <code>php vendor/bin/codecept run unit --steps --html --xml --coverage-html</code>

#### **3.6. 单元测试用例编写规范**

***

### **4. 项目代码规范**
#### **4.1. 配置文件规范**
* 因标准项目中不再把前台，API，后台都整合成一个项目中，所以common不在需要全局公共config了
* 项目的所有配置均放在frontend/config/中
  其中包含了
  bootstrap.php
  main.php
  params.php
  及根据情况自行拆分的，比如: params-route.php, params-cache.php, params-db.php, params-session.php等
  <font style="color: #f00;">配置仅维护一份，禁止出现 main-local.php main-qa.php等，不同环境用不同配置的情况。</font>

#### **4.2. 控制器代码编写规范**
* 必须包含一个BaseController.php 作为控制器的基类。 所有控制器必须继承该基类控制器。
* BaseController.php 包含一些与业务无关受保护的方法，方便子类使用。包含
  <code>public function runAction($id, $params = []);             // <font style="color: #f00;">(必须)</font>重写yii\base\Controller中的runAction，添加追踪日志</code>
  <code>public function actionError();                            // <font style="color: #f00;">(必须)</font>用来处理错误，比如跳转到对应的页面</code>
  <code>protected function param($key, $default = null);          // <font style="color: #f00;">(必须)</font>获取请求中的参数</code>
  <code>protected function params();                              // <font style="color: #f00;">(必须)</font>获取请求中所有参数</code>
  <code>protected function render($view, $params = []);           // <font style="color: #f00;">(必须)</font>重写Yii的render方法</code>
  <code>protected function renderPartial($view, $params = []);    // <font style="color: #f00;">(必须)</font>重写Yii的renderPartial方法</code>
  <code>protected function asJson($data);                         // <font style="color: green;">(可选)</font>重写Yii的asJson</code>
  <code>protected function asXml($data);                          // <font style="color: green;">(可选)</font>重写Yii的asXml</code>
  <font style="color: #00f;">注：上述BaseController及方法除actionError()外，标准项目中已提供，无需自己实现</font>
* 所有控制器不允许处理任何业务逻辑，仅允许获取，调用业务层接口及设置视图数据。
  控制器不允许跨层调用，不能直接调用模型, 外部接口等；一切通过业务层处理。
* Web页面控制器，一个控制器应对应一个页面，默认的actionIndex方法作为该页面的默认入口。
  该页面中的非跳转页面的行为，都应该写在同一页面控制器里。也就是说，一个控制器包含了对应页面的所有非跳转行为操作。例：
  ```php
  
  use Yii;

  /**
   * 公开课列表页控制器
   */
  class PublicClassListController extends BaseController
  {
      /**
       * 进入公开课列表页
       * 请求URL： https://xxx.zhan.com/public-class-list/index
       * SEO后URL： https://xxx.zhan.com/class/list.html
       * 请求类型： GET
       * @see params-route.php
       */
      public function actionIndex()
      {
          ...
          return $this->render(...);
      }

      /**
       * 进入公开课列表页
       * 请求URL： https://xxx.zhan.com/public-class-list/sign-up
       * SEO后URL： https://xxx.zhan.com/class/list/signup.html
       * 请求类型： AJAX/POST 
       * @see params-route.php
       */
      public function actionSignUp()
      {
          ...
          return $this->asJson(...);
      }

      ...
  }
  ```
* 默认不采用独立动作的方式来开发控制器。
* 在Web应用项目中(非接口API)的控制器中的接口API，统一采用如下书写规范
  ```php

  use Yii;
  use yii\log\Logger;
  use common\Result;
  use common\ErrorCode;

  /**
   * 公开课列表页控制器
   */
  class PublicClassListController extends BaseController
  {
      ...

      /**
       * 进入公开课列表页
       * 请求URL： https://xxx.zhan.com/public-class-list/sign-up
       * SEO后URL： https://xxx.zhan.com/class/list/signup.html
       * 请求类型： AJAX POST 
       * @see params-route.php
       */
      public function actionSignUp()
      {
          $ret = new Result();                 // Result对象为统一的接口返回对象
          try {
              ...
          } catch (\Exception $e) {
              Yii::getLogger()->log($e, Logger::LEVEL_ERROR, __METHOD__);
              $ret->code = ErrorCode::ERROR;
              $ret->message = $e->getMessage();
          }

          return $this->asJson($ret);
      }

      ...
  }
  ```
* 在API应用项目中(非WEB应用)的控制器中，实现接口采用如下规范
  ```php

  use Yii;
  use yii\log\Logger;
  use common\Result;
  use common\ErrorCode;

  /**
   * 接口控制器
   */
  class ApiController extends BaseController
  {

      /**
       * 获取用户名称接口
       * 请求URL： https://xxx.zhan.com/external/v1/api/get-user-name
       * 请求类型： POST | GET
       * 请求参数:
       *     ...
       * 返回参数：
       *     ...
       */
      public function actionGetUserName()
      {
          $ret = new Result();
          try {
              ...
          } catch (\Exception $e) {
              Yii::getLogger()->log($e, Logger::LEVEL_ERROR, __METHOD__);
              $ret->code = ErrorCode::ERROR;
              $ret->message = $e->getMessage();
          }

          return $ret;  // 因接口项目已在配置文件中统一配置了 请求格式，所以可以直接支持返回对象并转化成JSON 如果配置了的话。
      }
  }

  ```

#### **4.3. 业务层代码编写规范**
* 业务层统一使用 biz 作为表示，所有业务层代码文件都因在 frontend/biz 下
* 业务层代码文件的命名方式 必须以 XxxxxBiz.php 且类名也必须是 XxxxxBiz 的格式。 禁止直接用Xxxxx的方式命名，可读性不加，无法直观得知所声明或实例化的对象是什么
* 必须包含一个BaseBiz.php 作为业务层的基类。 所有业务层逻辑类必须继承该基类。
* 业务层分成2种类型，一种是基于前台业务与控制器配套的，比如我们有 HomeworkController 那对应会有 HomeworkBiz 来处理 homework页面相关的业务逻辑。
  另外一种是抽象的业务层，轻业务，通常是可公用的逻辑，比如 UserBiz 来处理获取用户信息，存取用户会话等，基于前台业务的业务类可以通过UserBiz来存取用户信息。
* 业务层统一采用静态 static 方法；

#### **4.4. 模型(AR Model)代码编写规范**
* 模型的命名方式，必须以 AR 开头，如 ARUser.php 类名ARUser ;  禁止直接使用Xxxx的命名方式，比如直接 User；
* 模型中，禁止实现任何业务逻辑
* 模型中，因手动 在方法 attributes 中返回数据库所有字段。 减少使用模型时，额外的查询表结构的开销。
* 模型中可以定义常量，比如一些状态字段等，避免在代码里产生魔法数字。 

#### **4.5. 外部服务对接规范**
* 外部服务要有统一的SDK进行操作，简单的实现方式可以编写成工具类提供开发调用。 复杂的方式是根据对应外部服务，创建对应的SDK项目；PHP项目中以第三方库的方式存在。
* 禁止维护多份外部SDK封装库，禁止以YII组件的方式实现。
* 封装库SDK代码规范：必须有详细的注释说明用途，使用场景，使用方式；调用举例，同任何工具类，组件(不限语言)都应有完善的注释。下列是某JS库的注释参考
  
  ```javascript

  /** 
   * Quick Modal
   * ================
   * 基于Bootstrap的模态框封装库
   * 该封装库可提高开发速度，无需在页面中硬编码模态框代码。
   * 同时该封装库还提供 modal() 方法，用于获取模态框创建后的DOM元素，满足自定义修改DOM元素，使得使用者可以向操作原生模态框一样随意修改。
   * 使用说明:
   * 1. 页面引入本JS
   * 2. 创建本组件对象实例，如: var myModal = new QuickModal("MyFirstModal", "This is Modal Title", "Hello world!");
   * 3. 调用 show()方法显示模态框，如： myModal.show();
   *
   * API：
   *    QuickModal(alias, title, bodyHTML, opts)                        构造方法
   *    参数
   *        alias                必须    模态框别名，可用来通过 QuickModal.modal.get("别名") 来获取已创建的对象实例(不用额外手动声明全局变量)
   *        title                必须    模态框标题
   *        bodyHTML             非必须  模态框内容HTML(如果参数opts中配置了 bodyHtmlUrl 时，bodyHtmlUrl会生效，从该URL来获取模态框内容)
   *        opts                 非必须  模态框参数, js object类型，opts = {...}，其中的可选参数如下
   *                                     debug                是否开启DEBUG模式，默认为false;  值 true|false
   *                                     destoryOnClose       关闭模态框时是否销毁DOM及对象，默认为true;  值 true|false
   *                                     bodyHtmlUrl          模态框内容获取URL，当配置了该参数时，模态框会使用该URL的返回内容进行填充; 默认为空
   *                                     bodyOverflowX        模态框内容超出模态框宽度时，显示行为，默认为空； 值 同css的overflow-x;
   *                                     bodyOverflowX        模态框内容超出模态框高度时，显示行为，默认为空； 值 同css的overflow-y
   *                                     width                模态框宽度，默认为空(默认大小)；
   *                                     height               模态框高度，默认为空(自动扩充)；
   *                                     reloadBodyHandler    重新加载模态框内容时调用的函数；通过注册该函数，当我们调用 myModal.reload(arg1, arg2...)
   *                                                          时，会用所注册的函数进行处理。(该函数及用法为试作版本，根据实际易用性及使用场景后续版本保留
   *                                                          或移除)
   *                                     btnDone              【确认】按钮参数
   *                                                          text          【确认】按钮显示文本
   *                                                          callback      【确认】按钮回调函数
   *                                     btnCancel            【取消】按钮参数
   *                                                          text          【取消】按钮显示文本
   *                                                          callback      【取消】按钮回调函数
   *    返回
   *        QuickModal的对象实例
   *    ----
   *    on(event, callback)                                             注册事件(所注册的事件参考bootstrap模态框事件)
   *    参数
   *        event                       事件
   *        callback                    事件回调处理函数
   *    ----
   *    modal()                         所有模态框实例的集合
   *    返回
   *        Map<alias, QuickModal>
   *    ----
   *    show()                                                          显示模态框
   *    ----
   *    hide()                                                          关闭模态框，当参数 destoryOnClose 为 true时，该方法关闭模态框后会销毁DOM元素及对象实例
   *    ----
   *    destory()                                                       销毁模态框的DOM元素及对象实例
   *    ----
   *    load(url)                                                       手动加载模态框内容
   *    参数
   *        url                         根据该URL的返回内容，填充进模态框
   *    ----
   *    reload()                                                        重新加载模态框内容，当注册了 reloadBodyHandler 时有效。
   *    ----
   *    empty()                                                         清空模态框内容
   *    ----
   *    setDoneData(data)                                               设置【确认】按钮返回数据，该数据会传递给所注册的【确认】按钮回调函数。
   *    参数
   *        data                        返回数据(任何类型)
   *    setCancelData(data)                                             设置【取消】按钮返回数据，该数据会传递给所注册的【取消】按钮回调函数。
   *    参数
   *        data                        返回数据(任何类型)
   *
   * 使用案例1： 通过配置【确认】按钮回调，实现获取数据
   *     var opts = {
   *         btnDone: {
   *             text: "提交",
   *             callback: mySubmit
   *         }
   *     };
   *     var myModal = new QuickModal("Example1", "This is Example1 Title", "Hello world!");
   *     myModal.show();
   *     QuickModal.modal.get("Example1").setDoneData("hi");
   *     ...
   *     // 【确认】按钮回调
   *     function mySubmit(data) {
   *         console.log(data);    // 当点击模态框的【确认】按钮后，控制台中会打印出  hi
   *     }
   *
   * @Author  <xinwei.he@zhan.com>
   * @Email   <xinwei.he@zhan.com>
   * @version 0.1.0
   * @license MIT <http://opensource.org/licenses/MIT>
   */
   "use strict";function ...
  ```
 * 封装库SDK不应与框架耦合，不应加入框架特有的代码。


***
[TOC]
