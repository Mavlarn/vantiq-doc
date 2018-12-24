# 使用Vantiq事件驱动平台进行微服务开发

Vantiq的Pronto是一个Dynamic Advanced Event Broker​，为构建实时企业应用，提供一个动态的分布式的事件管理、监控、权限等功能。

## Vantiq平台介绍
Vantiq作为一个PaaS平台，用于快速开发、部署和运行任务关键型实时应用。该平台包含：
1. 高生产力的可视化建模器（VANTIQ  Modelo），快速设计实时应用的功能，以复杂业务逻辑的简化规范转接界面脚本。
2. 动态的高级事件Broker（Pronto），为构建实时企业应用提供事件驱动平台。
3. 数据获取技术，获取各种企业内数据源，生成流数据，丰富上下文进行增强，过滤数据并将其转换为自动化决策引擎的事件。同时还能向数据源发送请求，进行各种自动化控制。
4. 事件和情景分析引擎，实时分析数据并根据机器学习和基于规则分析的结果来推动业务决策。
5. 协作技术，管理自动化系统和负责个人之间的协作，制定复杂情况的最优响应。

### VANTIQ Modelo
Modelo是一个可视化在线IDE，可以用来：
 * 创建数据处理流程（APP Module）
 * 创建客户端页面（Client Builder）
 * 创建和管理各种数据源，支持常见的Rest网络接口、JDBC、MQTT、Kafka、Email、短信、手机通知等等。
 * 管理事件的Topic
 * 创建人机交互的协作流程（Collaboration）
 * 进行数据的管理和数据对象结构(Schema)的管理
 * 创建各种Procedure、Rule等，用于在APP、Collaboration或其他地方进行数据处理和操作。

### Pronto
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

在这个例子当中，有1个Event事件："/domainABC/eventABC"，它有一个发布者"/serviceA/domainAbc"，ServiceA服务的一个方法，通过发送消息到这个发布队列，来发布事件。它有2个订阅者，"/service1/DomainFoo"和"/service2/DomainBar"，分别由2个Service，通过这2个订阅队列来消费事件。

>在Vantiq Pronto平台，事件是一个事件类型的定义，不是具体的事件消息。服务也不是直接使用事件，而是通过在这个事件上创建订阅者、发布者来访问事件。一般来说，这些订阅者、发布者都是本地队列(Topic)，服务通过这个本地队列来发送、获取事件消息。

接下来，我们就开始在Pronto里面创建namespace，Event，定义订阅者、发布者等。

>在切换不同的namespace的时候，一般都会提示是否保存当前的project，在Vantiq中，project是一个逻辑概念，我们用project来组织各种resources。当保存一个project时，就是保存这个project包含的resources，以及它在网页IDE中显示的状态，下次再打开，还能打开之前打开的东西。在我们的实例中，namespace中创建的Event、队列等在操作相应按钮的时候就已经创建，就保存在该namespace的resources中了，所以不需要再保存在某个项目中。如果想下次打开namespace的时候，打开之前打开的组件，那就可以创建一个项目保存下来。

### 创建namespace
我们先新建一个namespace，ms_catalog_tutorial，这需要一个拥有developer权限的用户。然后在operation Tab中创建：

![step1-namespace-create](2_vantiq_pronto_tutorial/step1-namespace-create.jpg?raw=true "create namespace")

创建好namespace以后，切换到新建的namespace：

![step2-namespace-switch.jpg](2_vantiq_pronto_tutorial/step2-namespace-switch.jpg?raw=true "switch namespace")

然后切换到新建的namespace，创建catalog。这样我们就能够在这个namespace创建和管理Event。

![step3-catalog-create.jpg](2_vantiq_pronto_tutorial/step3-catalog-create.jpg?raw=true "create catalog")


### 创建Event
下面就开始创建事件Event。我们在创建Event之前，需要先给这个事件定义个Type，来作为事件消息的schema。打开development的Tab，点击Add，添加一个Type：

![step4-type-open.jpg](2_vantiq_pronto_tutorial/step4-type-open.jpg?raw=true "Open Type")

然后点击新建以后，在Type创建/编辑页面，输入Type的名字‘EventABC’, Role类型是Schema，添加2个属性id和name：

![step5-type-create.jpg](2_vantiq_pronto_tutorial/step5-type-create.jpg?raw=true "Create Type")

下面就打开'Event Catalog'页面来创建 新的Event：

![step6-catalog-open.jpg](2_vantiq_pronto_tutorial/step6-catalog-open.jpg?raw=true "Open Catalog")

输入该事件名字、描述、schema，然后保存：

![step8-catalog-popup.jpg](2_vantiq_pronto_tutorial/step8-catalog-popup.jpg?raw=true "Save Catalog")

点击保存以后，会弹出一个对话框，来设置该事件的关键字，用于进行事件的查询、过滤。

![step8-catalog-popup.jpg](2_vantiq_pronto_tutorial/step8-catalog-popup.jpg?raw=true "Save Catalog")

保存成功后就可以在Event Catalog界面看到新建的事件：

![step9-catalog-list.jpg](2_vantiq_pronto_tutorial/step9-catalog-list.jpg?raw=true "Catalog Result")

点击新建的Event，就可以打开Event的界面，在这里，我们可以针对这个Event创建订阅者和发布者：

![step10-catalog-event.jpg](2_vantiq_pronto_tutorial/step10-catalog-event.jpg?raw=true "Catalog Event")

点击publisher旁边的'click to view'，打开发布者，输入发布队列的名字，点‘become publisher’:

![step11-catalog-event-publisher.jpg](2_vantiq_pronto_tutorial/step11-catalog-event-publisher.jpg?raw=true "Catalog Event Publisher")

创建完成以后，就可以在下面的列表中看到发布者。我们的ServiceA会通过这个队列来发布事件。

同样，创建2个订阅者，两个Service会通过这个订阅者订阅来订阅消息：

![step12-catalog-event-subscriber.jpg](2_vantiq_pronto_tutorial/step12-catalog-event-subscriber.jpg?raw=true "Catalog Event Subscriber")

至此，我们在Vantiq上需要的Event、队列等已经创建完成，我们可以通过 show - 'resource list' 来查看现在所有的资源：

![step13-resource-list.jpg](2_vantiq_pronto_tutorial/step13-resource-list.jpg?raw=true "Resource List")

### 测试事件的发布与订阅
下面我们就通过发布者队列来发布一个事件消息，然后通过订阅者队列来查看这个事件。这样我们就能验证这个Event Catelog，和发布者、订阅者之间的关系。所以，我们需要先查看这个订阅者队列上的消息，这需要我们新建一个订阅器：

![step14-subscription-open.jpg](2_vantiq_pronto_tutorial/step14-subscription-open.jpg?raw=true "Open Subscription")

我们在刚才创建的订阅队列上创建订阅器，这样就可以在页面上看到队列上的消息。

![step15-subscription-create.jpg](2_vantiq_pronto_tutorial/step15-subscription-create.jpg?raw=true "Open Subscription")

下面从Resource List中分别打开发布者队列"/serviceA/domainAbc"，和两个订阅器："service1_DomainFoo_Sub"和"service2_DomainBar_Sub"。我们要在发布者队列的界面上发布一个消息，检查下面两个订阅器上是否有刚才创建的消息。

![step16-test-1.jpg](2_vantiq_pronto_tutorial/step16-test-1.jpg?raw=true "Open Subscription")

当我们在发布队列的界面里，输入发布的消息的内容以后，点击”Publish“，这个消息会被发送到发送至队列，并同步到两个订阅者队列。

![step16-test-2.jpg](2_vantiq_pronto_tutorial/step16-test-2.jpg?raw=true "Open Subscription")

## 微服务开发
至此就完成了Vantiq平台的设置，下面开始服务的开发。Vantiq提供了多种语言的SDK，包括Java、JavaScript、iOS等，也可以直接通过Vantiq的Rest API接口，来访问Vantiq服务。

Vantiq Java SDK地址：  
https://github.com/Vantiq/vantiq-sdk-java

Rest API地址：  
https://dev.vantiq.cn/docs/system/api/index.html

### Java中使用Vantiq Java SDK
在Java项目中，最简单的访问Vantiq的方法就是使用Vantiq提供的Java SDK，我们只需在Maven中引入依赖：
```xml
<repositories>
    <repository>
        <id>Vantiq Maven Repo</id>
        <url>https://dl.bintray.com/vantiq/maven</url>
    </repository>
</repositories>

<dependencies>
    <dependency>
        <groupId>io.vantiq</groupId>
        <artifactId>vantiq-sdk</artifactId>
        <version>1.0.17</version>
        <scope>compile</scope>
    </dependency>
</dependencies>    
```

如果使用gradle，则是：
```
repositories {
    maven {
        url "https://dl.bintray.com/vantiq/maven"
    }
}

dependencies {
    compile 'io.vantiq:vantiq-sdk:1.0.17'
}
```

然后，为了能够访问Vantiq，我们需要权限，我们可以使用用户名密码授权，也可以使用access token授权。如果在代码中暴露用户名和密码，肯定是不合适的，所以最好还是使用token。我们可以为每个namespace创建一个access token，这个token就能访问这个namespace的资源。所以就先创建一个token，还是打开vantiq的Operation的界面，这个界面显示的是当前用户拥有的所有的access token，可以看到，即使是创建者自己，在每个namespace都需要一个access token才能访问。

![step17-access-token-open.jpg](2_vantiq_pronto_tutorial/step17-access-token-open.jpg?raw=true "Open access token")

点击创建按钮，为微服务创建一个专用的access token：

![step18-access-token-create.jpg](2_vantiq_pronto_tutorial/step18-access-token-create.jpg?raw=true "Create access token")

创建完成以后，就可以看到新建的token，将这个token复制下来在java中使用。

![step19-access-token-copy.jpg](2_vantiq_pronto_tutorial/step19-access-token-copy.jpg?raw=true "Create access token")

### Java代码
在Java中访问Vantiq的队列很简单，我们先来看看发布：

```java
    String TOKEN = "MXLzkzJE7P0f6whhTeKWn9qcuFRAuqIeqivLl8j7rl0="; // 上面创建的token
    String VANTIQ_URL = "https://dev.vantiq.cn"; // 我们的测试服务器地址
    String TOPIC_PUB = "/serviceA/domainAbc"; // 发布者队列

    Vantiq vantiq = new Vantiq(VANTIQ_URL);
    vantiq.setAccessToken(TOKEN);
    Map event = new HashMap();
    event.put("id", "112");
    event.put("name", "name java 1");

    vantiq.publish(Vantiq.SystemResources.TOPICS.value(), TOPIC_PUB, event, new BaseResponseHandler()
    );
```

下面是订阅的代码：
```java
    String TOKEN = "MXLzkzJE7P0f6whhTeKWn9qcuFRAuqIeqivLl8j7rl0=";
    String VANTIQ_URL = "https://dev.vantiq.cn"; // 我们的测试服务器地址
    String TOPIC_SUB_1 = "/service1/DomainFoo"; // 订阅者队列1
    String TOPIC_SUB_2 = "/service2/DomainBar"; // 订阅者队列2

    Vantiq vantiq = new Vantiq(TestRestApi.VANTIQ_URL);
    vantiq.setAccessToken(TestRestApi.TOKEN);
    vantiq.subscribe(Vantiq.SystemResources.TOPICS.value(),
            TestRestApi.TOPIC_SUB_1,
            null,
            new StandardOutputCallback(TestRestApi.TOPIC_SUB_1)
    );
```

### 使用Rest API发布事件
除了使用Vantiq SDK以外，我们也可以直接使用Rest API，它是一个Rest风格的WEB接口，我们可以在一些不方便添加java库的地方，或者在其他的一些系统中，使用这种方式访问Vantiq。下面是使用Java通过Rest接口提交事件的代码。

```java

    String TOKEN = "MXLzkzJE7P0f6whhTeKWn9qcuFRAuqIeqivLl8j7rl0=";
    String VANTIQ_URL = "https://dev.vantiq.cn";
    String TOPIC_PUB = "/serviceA/domainAbc";
    String TOPIC_URL = VANTIQ_URL + "/api/v1/resources/topics/" + TOPIC_PUB + "?token=" + TOKEN;
    JSONObject postJSON = new JSONObject();
    postJSON.put("name", "Brett 1");
    postJSON.put("id", "23456789");

    String json = postJSON.toString();
    try {
        URL url = new URL(TOPIC_URL);
        HttpURLConnection conn = (HttpURLConnection) url.openConnection();
        conn.setConnectTimeout(5000);
        conn.setRequestProperty("Content-Type", "application/json; charset=UTF-8");
        conn.setDoOutput(true);
        conn.setDoInput(true);
        conn.setRequestMethod("POST");
        OutputStream os = conn.getOutputStream();
        os.write(json.getBytes("UTF-8"));
        os.close();
        // read the response
        InputStream in = new BufferedInputStream(conn.getInputStream());

        String result = IOUtils.toString(in, "UTF-8");
        System.out.println(result);
        System.out.println("Message Published");
        in.close();
        conn.disconnect();
    } catch (Exception e) {
        System.out.println(e);
    }
```

有关Rest接口的详细文档，请参考官方文档（还未翻译成中文，会尽快翻译）：
https://dev.vantiq.cn/docs/system/api/index.html

### 在SAP中访问Rest API
如果需要在SAP等系统中访问Vantiq，可以使用Vantiq是Rest接口进行，具体方法可以参考如下文章：
https://blogs.sap.com/2013/01/24/developing-a-rest-api-in-abap/

