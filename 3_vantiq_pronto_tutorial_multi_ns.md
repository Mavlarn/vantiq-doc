# 使用Vantiq事件驱动平台进行微服务开发

Vantiq的Pronto是一个Dynamic Advanced Event Broker​，为构建实时企业应用，提供一个动态的分布式的事件管理、监控、权限等功能。

## Vantiq平台介绍
Vantiq作为一个PaaS平台，用于快速开发、部署和运行任务关键型实时应用。该平台包含：
1. 高生产力的可视化建模器（VANTIQ  Modelo），快速设计实时应用的功能，以复杂业务逻辑的简化规范转接界面脚本。
2. 动态的高级事件Broker（Pronto），为构建实时企业应用提供事件驱动平台。
3. 数据获取技术，获取各种企业内数据源，生成流数据，丰富上下文进行增强，过滤数据并将其转换为自动化决策引擎的事件。同时还能向数据源发送请求，进行各种自动化控制。
4. 事件和情景分析引擎，实时分析数据并根据机器学习和基于规则分析的结果来推动业务决策。
5. 协作技术，管理自动化系统和负责个人之间的协作，制定复杂情况的最优响应。

 介绍，以及一些名词说明，如namespace、project，以及user的权限等
### VANTIQ Modelo
Modelo是一个可视化在线IDE，可以用来：
 * 创建数据处理流程（APP Module）
 * 创建客户端页面（Client Builder）
 * 创建和管理各种数据源，支持常见的Rest网络接口、JDBC、MQTT、Kafka、Email、短信、手机通知等等。
 * 管理事件的Topic
 * 创建人机交互的协作流程（Collaboration）
 * 进行数据的管理和数据对象结构(Schema)的管理
 * 创建各种Procedure、Rule等，用于在APP、Collaboration或其他地方进行数据处理和操作。

## Pronto介绍
Pronto作为一个Event Broker​，相比其他开源的消息队列产品，或商用的Event Broker​产品，提供了很多独特或便捷的功能，包括：
 * 事件目录（Catalog），我们可以用它来查看所有的事件，通过事件属性和其他信息进行事件查找、过滤等，给事件定义schema。
 * 事件管理器（Manager），我们可以管理事件的订阅和发布，设置访问权限。
 * 事件访问日志（Ledger），用于对所有的事件访问进行日志记录、权限控制。
 * 企业连接器（Enterprise Connector），我们还可以使用Vantiq平台的Source、Procedure、Rule等进行事件的自动处理、验证、设置规则等。

使用Pronto作为事件驱动平台的vent Broker​，大致的流程如下：
1. 定义事件的schema。在Vantiq里，几乎所有的数据都是以Json格式进行传输，Pronto里面的事件，也需要定义一个schema，这样我们就能通过schema属性查找事件。
2. 定义Event，Pronto里面的Event相当于一个事件的定义，而不是具体发生的事件。所以，我们可以理解成这个Event实际上相当于一个事件队列，因为通常我们会把一种事件消息统一发到一个队列里。
3. 定义该事件的发布者。一个事件的发布者就是一个队列，发布到这个队列的消息，就会发布到该事件上。该队列对这个发布者来说，应该是一个本地队列，其他的程序不应该针对这个队列有读写权限。
4. 定义该事件的订阅者。事件的消费者也是一个队列，消费者从这个消费队列获得事件消息，改消费队列的事件是从这个事件来的。

## Pronto Event Brokers使用实例
下面，就通过一个完整的实例来看一下如果使用Pronto进行事件驱动开发。

### 场景描述
该实例的场景描述如下：

![pronto-event-pub-sub](2_vantiq_pronto_tutorial/pronto-event-pub-sub.jpg?raw=true "Printo-Event_Pub_Sub")

在这个例子当中，有2个Event：
 * 事件"/domainAbc/eventAA"，它有一个发布者"/serviceA/domainAbc"，ServiceA服务的一个方法，通过发送消息到这个发布队列，来发布事件。它有2个订阅者，"/service1/DomainFoo"和"/service2/DomainBar"，分别由2个Service，通过这个订阅队列来消费事件。
 * 事件"/domainEFG/eventEE"，它有一个发布者"/serviceA/domainEFG"，和一个订阅者"/service2/DomainGo"，他们分别由ServiceA发布和Service2订阅。

同时，不同的Service只能访问自己的namespace里面的订阅者和发布者，我们可以通过这个来控制服务对事件权限。

### Pronto Event
首先，我们需要在Pronto里面创建namespace，Event，定义订阅者、发布者等。

1. 创建namespace
这个例子中，我们要使用4个namespace，分别叫"ms_catalog"，"ms_serviceA"，"ms_service1"和"ms_service2"。

先创建ms_catalog，进入dev.vantiq.cn，打开"operations"进行namesp创建：

![step1-namespace](2_vantiq_pronto_tutorial/step1-namespace.jpg.jpg?raw=true "namespace")

打开namespace管理界面以后，点击"create"创建：

![step2-namespace-create.jpg](2_vantiq_pronto_tutorial/step2-namespace-create.jpg?raw=true "create namespace")

然后依次再创建其他3个namespace。然后，切换到这个新建的 ms_catalog namespace上：

![step3-namespace-switch.jpg](2_vantiq_pronto_tutorial/step3-namespace-switch.jpg.jpg?raw=true "switch to catalog namespace")

然后在ns列表中点击这个namespace，然后创建catalog：

![step4-namespace-create-catalog.jpg](2_vantiq_pronto_tutorial/step4-namespace-create-catalog.jpg?raw=true "Create catalog")

>注意，我们只需要在ms_catalog这个namespace上创建catalog，其他的几个都不需要。

2. 创建Event
下面就开始创建事件Event。我们在创建Event之前，需要先给这个事件定义个Type，来作为事件消息的schema。打开development的Tab，点击Add，添加一个Type：

![step5-catalog-create-type.jpg](2_vantiq_pronto_tutorial/step5-catalog-create-type.jpg?raw=true "Create Type")P

然后点击新建以后，在Type创建/编辑页面，输入Type的名字，设置属性：

![step6-catalog-create-type-properties.jpg](2_vantiq_pronto_tutorial/step6-catalog-create-type-properties.jpg?raw=true "Create Type 2")

然后，再创建一个EventEE，属性也是id, name。

现在就可以创建Catalog，在Show下面找到Event Catelog并打开：
![step7-catalog-create-catalog1.jpg](2_vantiq_pronto_tutorial/step7-catalog-create-catalog1.jpg?raw=true "Create Type 2")


输入相应的内容，保存的时候会弹出对话框设置关键字，这个关键字是用于查询过滤等。
![step8-catalog-create-catalog2.jpg](2_vantiq_pronto_tutorial/step8-catalog-create-catalog2.jpg?raw=true "Create Catelog")

创建完2个Type，2个Event以后，应该是这样的：
![step9-catalog-create-catalog-list.jpg](2_vantiq_pronto_tutorial/step9-catalog-create-catalog-list.jpg?raw=true "Catelog List")

3. 授权发布者和订阅者的namespace
下面我们就需要让我们的发布者ServiceA所在的namespace和2个订阅者Service的namespace能够访问Event Catalog的事件。确保还在"ms_catalog"的namespace中，

3. 创建发布者
4. 创建订阅者
5. 测试

下面，就通过一个完整的实例来看一下如果使用Pronto进行事件驱动开发。该实例的场景描述如下：
 * 

场景：统一的catelog，2个服务，