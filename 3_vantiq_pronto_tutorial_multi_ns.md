# 使用Vantiq Pronto进行多namespace管理

Vantiq的Pronto是一个Dynamic Advanced Event Broker​，为构建实时企业应用，提供一个动态的分布式的事件管理、监控、权限等功能。Pronto进行Event Catelog的权限管理，是通过namespace实现的。这篇文章我们就看看如何通过多个namespace来实现不同的服务访问Event时的权限控制。

### 场景描述
该实例的场景描述如下：

![pronto-event-pub-sub](3_vantiq_pronto_tutorial_multi_ns/pronto-event-pub-sub.jpg?raw=true "Printo-Event_Pub_Sub")

在这个例子当中，有2个Event：
 * 事件"/domainAbc/eventAA"，它有一个发布者"/serviceA/domainAbc"，ServiceA服务的一个方法，通过发送消息到这个发布队列，来发布事件。它有2个订阅者，"/service1/DomainFoo"和"/service2/DomainBar"，分别由2个Service，通过这个订阅队列来消费事件。
 * 事件"/domainEFG/eventEE"，它有一个发布者"/serviceA/domainEFG"，和一个订阅者"/service2/DomainGo"，他们分别由ServiceA发布和Service2订阅。

同时，不同的Service只能访问自己的namespace里面的订阅者和发布者，我们可以通过这个来控制服务对事件权限。

### Pronto Event
首先，我们需要在Pronto里面创建namespace，Event，定义订阅者、发布者等。

1. 创建namespace
这个例子中，我们要使用4个namespace，分别叫"ms_catalog"，"ms_serviceA"，"ms_service1"和"ms_service2"。

先创建ms_catalog，进入dev.vantiq.cn，打开"operations"进行namesp创建：

![step1-namespace](3_vantiq_pronto_tutorial_multi_ns/step1-namespace.jpg.jpg?raw=true "namespace")

打开namespace管理界面以后，点击"create"创建：

![step2-namespace-create.jpg](3_vantiq_pronto_tutorial_multi_ns/step2-namespace-create.jpg?raw=true "create namespace")

然后依次再创建其他3个namespace。然后，切换到这个新建的 ms_catalog namespace上：

![step3-namespace-switch.jpg](3_vantiq_pronto_tutorial_multi_ns/step3-namespace-switch.jpg.jpg?raw=true "switch to catalog namespace")

然后在ns列表中点击这个namespace，然后创建catalog：

![step4-namespace-create-catalog.jpg](3_vantiq_pronto_tutorial_multi_ns/step4-namespace-create-catalog.jpg?raw=true "Create catalog")

>注意，我们只需要在ms_catalog这个namespace上创建catalog，其他的几个都不需要。

2. 创建Event
下面就开始创建事件Event。我们在创建Event之前，需要先给这个事件定义个Type，来作为事件消息的schema。打开development的Tab，点击Add，添加一个Type：

![step5-catalog-create-type.jpg](3_vantiq_pronto_tutorial_multi_ns/step5-catalog-create-type.jpg?raw=true "Create Type")P

然后点击新建以后，在Type创建/编辑页面，输入Type的名字，设置属性：

![step6-catalog-create-type-properties.jpg](3_vantiq_pronto_tutorial_multi_ns/step6-catalog-create-type-properties.jpg?raw=true "Create Type 2")

然后，再创建一个EventEE，属性也是id, name。

现在就可以创建Catalog，在Show下面找到Event Catelog并打开：
![step7-catalog-create-catalog1.jpg](3_vantiq_pronto_tutorial_multi_ns/step7-catalog-create-catalog1.jpg?raw=true "Create Type 2")


输入相应的内容，保存的时候会弹出对话框设置关键字，这个关键字是用于查询过滤等。
![step8-catalog-create-catalog2.jpg](3_vantiq_pronto_tutorial_multi_ns/step8-catalog-create-catalog2.jpg?raw=true "Create Catelog")

创建完2个Type，2个Event以后，应该是这样的：
![step9-catalog-create-catalog-list.jpg](3_vantiq_pronto_tutorial_multi_ns/step9-catalog-create-catalog-list.jpg?raw=true "Catelog List")

3. 授权发布者和订阅者的namespace
下面我们就需要让我们的发布者ServiceA所在的namespace和2个订阅者Service的namespace能够访问Event Catalog的事件。确保还在"ms_catalog"的namespace中，

。。。未完待续。。。