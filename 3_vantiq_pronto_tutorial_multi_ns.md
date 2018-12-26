# 使用Vantiq Pronto进行多namespace管理

Vantiq的Pronto是一个Dynamic Advanced Event Broker​，为构建实时企业应用，提供一个动态的分布式的事件管理、监控、权限等功能。Pronto进行Event Catalog的权限管理，是通过namespace实现的。这篇文章我们就看看如何通过多个namespace来实现不同的服务访问Event时的权限控制。

### 场景描述
该实例的场景描述如下：

![pronto-event-pub-sub](3_vantiq_pronto_tutorial_multi_ns/pronto-event-pub-sub.jpg?raw=true "Printo-Event_Pub_Sub")

在这个例子当中，有2个Event：
 * 事件"`/domainAbc/eventAA`"，它有一个发布者"`/serviceA/domainAbc`"，`ServiceA`服务的一个方法，通过发送消息到这个发布队列，来发布事件。它有2个订阅者，"`/service1/DomainFoo`"和"`/service2/DomainBar`"，分别由2个Service，通过这个订阅队列来消费事件。
 * 事件"`/domainEFG/eventEE`"，它有一个发布者"/`serviceA/domainEFG`"，和一个订阅者"`/service2/DomainGo`"，他们分别由`ServiceA`发布和`Service2`订阅。

同时，不同的Service只能访问自己的namespace里面的订阅者和发布者，我们可以通过这个来控制服务对事件权限。

在下面的步骤当中，我们需要创建namespace，然后在pronto里面创建Event，定义订阅者、发布者，再进行授权。

>在切换不同的namespace的时候，一般都会提示是否保存当前的project，在Vantiq中，project是一个逻辑概念，我们用project来组织各种resources。当保存一个project时，就是保存这个project包含的resources，以及它在网页IDE中显示的状态，下次再打开，还能打开之前打开的东西。在我们的实例中，namespace中创建的Event、队列等在操作相应按钮的时候就已经创建，就保存在该namespace的resources中了，所以不需要再保存在某个项目中。如果想下次打开namespace的时候，打开之前打开的组件，那就可以创建一个项目保存下来。

### 创建namespace

这个例子中，我们要使用4个namespace，分别叫"`ms_catalog`"，"`ms_serviceA`"，"`ms_service1`"和"`ms_service2`"。

先创建`ms_catalog`，进入dev.vantiq.cn，打开"operations"进行namesp创建：

![step1-namespace](3_vantiq_pronto_tutorial_multi_ns/step1-namespace.jpg?raw=true "namespace")

打开namespace管理界面以后，点击"create"创建：

![step2-namespace-create.jpg](3_vantiq_pronto_tutorial_multi_ns/step2-namespace-create.jpg?raw=true "create namespace")

然后依次再创建其他3个namespace。然后，切换到这个新建的 `ms_catalog` namespace上：

![step3-namespace-switch.jpg](3_vantiq_pronto_tutorial_multi_ns/step3-namespace-switch.jpg?raw=true "switch to catalog namespace")

### Pronto Event Catalog

创建好namespace以后，进入`ms_catalog`这个namespace，我们将要在这个namespace里面创建Catalog。在Administer里面打开namespace列表，从里面打开`ms_catalog`这个namespace：

![step4-namespace-create-catalog.jpg](3_vantiq_pronto_tutorial_multi_ns/step4-namespace-create-catalog.jpg?raw=true "Create catalog")

>注意，我们只需要在`ms_catalog`这个namespace上创建catalog，其他的几个都不需要。其他的namespace会通过access token的方式授权访问这个catalog namespace，并使用这个里面的事件定义。

下面就开始创建事件Event。我们在创建Event之前，需要先给这个事件定义个Type，来作为事件消息的schema。打开development的Tab，点击Add，添加一个Type：

![step5-catalog-create-type.jpg](3_vantiq_pronto_tutorial_multi_ns/step5-catalog-create-type.jpg?raw=true "Create Type")

然后点击新建以后，在Type创建/编辑页面，输入Type的名字，设置属性：

![step6-catalog-create-type-properties.jpg](3_vantiq_pronto_tutorial_multi_ns/step6-catalog-create-type-properties.jpg?raw=true "Create Type 2")

然后，再创建一个`EventEE`，属性也是id, name。

现在就可以创建Event了，在Show下面找到Event Catalog并打开，点新建来创建：

![step7-catalog-create-catalog1.jpg](3_vantiq_pronto_tutorial_multi_ns/step7-catalog-create-catalog1.jpg?raw=true "Create Type 2")


输入相应的内容，保存的时候会弹出对话框设置关键字，这个关键字是用于查询过滤等。

![step8-catalog-create-catalog2.jpg](3_vantiq_pronto_tutorial_multi_ns/step8-catalog-create-catalog2.jpg?raw=true "Create Catalog")

创建完2个Type，2个Event以后，应该是这样的，我们可以在Event Catalog界面的输入框输入事件名、字段、关键字来进行事件的查询。这是为了方便之后打开这个namespace的时候能打开之前打开的东西，可以选择保存项目。

![step9-catalog-create-catalog-list.jpg](3_vantiq_pronto_tutorial_multi_ns/step9-catalog-create-catalog-list.jpg?raw=true "Catalog List")



### 授权发布者和订阅者的namespace
下面我们就需要让我们的发布者`ServiceA`所在的namespace和2个订阅者Service的namespace能够访问Event Catalog的事件。确保还在"`ms_catalog`"的namespace中，打开access tokens界面：

![step10-access-tokens-open.jpg](3_vantiq_pronto_tutorial_multi_ns/step10-access-tokens-open.jpg?raw=true "Open Access Tokens")

这里面列出的是先有所有的access token。access token的用处就是授权访问某一个namespace，所以我们要创建一个token给其他几个namespace，以让他们能访问这个namespace里面的Event Catalog。点击新建：

![step11-access-tokens-create.jpg](3_vantiq_pronto_tutorial_multi_ns/step11-access-tokens-create.jpg?raw=true "Create Access Token")

创建好以后，在列表中找到这个token并复制：

![step12-access-tokens-copy.jpg](3_vantiq_pronto_tutorial_multi_ns/step12-access-tokens-copy.jpg?raw=true "Copy Access Token")

然后，我们进入`ms_serviceA`这个namespace，我们要在这个namespace中连接到`ms_catalog`中定义的Event Catalog。进入`ms_serviceA`这个namespace后，打开namespace列表，点击自己的namespace，设置Event Catalog链接：

![step13-link-catalog-namespace.jpg](3_vantiq_pronto_tutorial_multi_ns/step13-link-catalog-namespace.jpg?raw=true "Link Event Catalog")

然后在弹出的对话框里复制刚才拷贝的token，然后点击"Add Catalog"来添加链接。添加成功后，会在下面显示链接到的namespace名字，确保

![step14-link-catalog-namespace-result.jpg](3_vantiq_pronto_tutorial_multi_ns/step14-link-catalog-namespace-result.jpg?raw=true "Link Event Catalog Result")

为`Service1`和`Service2`进行同样的操作，这样，我们的3个服务使用的namespace对Catalog所在的namespace的授权就进行完了。

### 创建事件发布者
完成了所有的service所在的命名空间对Catalog命名空间的授权以后，我们就来创建事件的发布者。在这个例子中，`ServiceA`这个服务能够发布这两个事件，所以我们需要在'`ms_serviceA`'这个命名空间中针对这两个事件创建发布者队列。这样`ServiceA`这个服务就能够空间它的发布者队列，发送消息到事件上。

所以，进入'`ms_serviceA`'命名空间，打开Event Catalog，点击一个事件，查看事件的发布者：

![step15-event-publisher-view.jpg](3_vantiq_pronto_tutorial_multi_ns/step15-event-publisher-view.jpg?raw=true "Event Publisher View")

在弹出的对话框中，我们可以看到目前这个事件的所有发布者，目前是没有任何发布者的，它可能弹出一个窗口警告说没有找到发布者。我们可以通过一个Source来发布事件，也可以通过一个Topic队列来发布。我们选择Topic。数据我们的之前定义好的队列名'`/serviceA/domainAbc`'。这个队列名的命名是说'`serviceA'的'domainAbc`'领域对象发生变化，产生的事件。

![step16-event-publisher-add.jpg](3_vantiq_pronto_tutorial_multi_ns/step16-event-publisher-add.jpg?raw=true "Event Publisher View")

我们输入队列名并创建以后，新建的队列就会显示到下面：

![step17-event-publisher-result.jpg](3_vantiq_pronto_tutorial_multi_ns/step17-event-publisher-result.jpg?raw=true "Event Publisher <rp></rp>esult")

>注意这里，'Local Event Path'意思就是在ms_serviceA这个namespace里面，发布事件使用的队列，这个队列的名字前面自动加上了'/topics'的前缀。

同样，我们也给'`/domainEFG/eventEE`'这个事件加上发布者队列'`/serviceA/domainEFG`'。

### 创建事件订阅者
跟创建发布者队列类似，我们现在进入'`ms_service1`'这个命名空间，由于'`Service1`'只需要订阅事件'`/domainAbc/eventAA`'，所以只是在这个事件上创建一个订阅者。打开Event Catalog，点击这个事件，查看事件的订阅者：

![step18-event-subscription-view.jpg](3_vantiq_pronto_tutorial_multi_ns/step18-event-subscription-view.jpg?raw=true "Event Publisher <rp></rp>esult")

在弹出的对话框中输入队列名'`/service1/DomainFoo`'创建，可以看到下面的结果：

![step19-event-subscription-view.jpg](3_vantiq_pronto_tutorial_multi_ns/step19-event-subscription-view.jpg?raw=true "Event Publisher <rp></rp>esult")

>这里的'Local Event'就是针对'`ms_service1`'这个命名空间本地队列，创建完以后，它的名字前不会添加前缀。如果需要命名统一，可以自行添加。

同样，进入'`ms_service2`'这个命名空间，`Serice2`在两个事件上都需要订阅，所以要创建2个订阅者队列，也就是对事件'`/domainAbc/eventAA`'，订阅者是'`/service2/DomainBar`'，对事件'`/domainEFG/eventEE`'订阅者是'`/service2/DomainGo`'。

### 在Catalog中查看发布、订阅
我们创建完了发布者、订阅者，下面就能在'`ms_catalog`'命名空间中查看我们的事件，以及这些事件上的发布者、订阅者。如果有一些服务不应该对某些事件进行订阅，就可以在这里看到并删除。

这样，我们在企业范围内的所有事件都可以在一个地方统一的管理，并通过图形界面的方式查看、搜索，查看数据，查看发布者、订阅者等。

### 发布订阅事件
至此，就完成了事件在多个namespace之间的组织，Event的创建，发布者和订阅者的创建，接下来，就可以在Java客户端或其他客户端里，发布、订阅事件。具体的代码跟上一篇类似，就不再赘述了。
