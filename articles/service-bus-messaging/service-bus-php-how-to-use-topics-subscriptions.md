---
title: Использование разделов Служебной шины Azure с PHP
description: В этом руководстве вы узнаете, как использовать разделы и подписки Служебной шины Azure из приложения PHP.
ms.devlang: PHP
ms.topic: quickstart
ms.date: 06/23/2020
ms.openlocfilehash: 706f523fdfb3c710bb16b048cfc68ce98875adb1
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88066208"
---
# <a name="quickstart-how-to-use-service-bus-topics-and-subscriptions-with-php"></a>Краткое руководство. Как использовать разделы и подписки Служебной шины с PHP

[!INCLUDE [service-bus-selector-topics](../../includes/service-bus-selector-topics.md)]

Из этой статьи вы узнаете, как использовать разделы и подписки служебной шины. Примеры написаны на PHP и используют [пакет Azure SDK для PHP](https://github.com/Azure/azure-sdk-for-php). К рассмотренным сценариям относятся:

- создание разделов и подписок; 
- создание фильтров подписки; 
- отправка сообщений в раздел; 
- получение сообщений из подписки;
- удаление разделов и подписок.

## <a name="prerequisites"></a>Предварительные требования
1. Подписка Azure. Для работы с этим учебником требуется учетная запись Azure. Вы можете активировать [преимущества подписчика Visual Studio или MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/?WT.mc_id=A85619ABF) или зарегистрироваться для получения [бесплатной учетной записи](https://azure.microsoft.com/free/?WT.mc_id=A85619ABF).
2. Выполните шаги из [краткого руководства по созданию раздела Служебной шины и подписок на него с помощью портала Azure](service-bus-quickstart-topics-subscriptions-portal.md), чтобы создать **пространство имен** Служебной шины и получить **строку подключения**.

    > [!NOTE]
    > В этом кратком руководстве описано, как создать **раздел** и **подписку** на этот разделу с помощью **PHP**. 

## <a name="create-a-php-application"></a>Создание приложения PHP
Единственным требованием для создания приложения PHP, которое получает доступ к службе BLOB-объектов Azure, является наличие ссылки на классы в [пакете SDK для Azure для PHP](https://github.com/Azure/azure-sdk-for-php) непосредственно из кода. Для создания приложения можно использовать любые средства разработки или Блокнот.

> [!NOTE]
> В установленном пакете PHP должно быть установлено и включено [расширение OpenSSL](https://php.net/openssl).
> 
> 

В этой статье рассказывается, как использовать компоненты службы, которые могут быть вызваны локально в приложении PHP или в коде, работающем в веб-роли, рабочей роли или на веб-сайте Azure.

## <a name="get-the-azure-client-libraries"></a>Получение клиентских библиотек Azure

### <a name="install-via-composer"></a>Установка через компоновщик
1. Создайте файл с именем **composer.json** в корневой папке проекта и добавьте в него следующий код:
   
    ```json
    {
      "require": {
        "microsoft/windowsazure": "*"
      }
    }
    ```
2. Скачайте **[composer.phar][composer-phar]** в корневой каталог проекта.
3. Откройте командную строку и выполните следующую команду в корневом каталоге проекта.
   
    ```
    php composer.phar install
    ```

## <a name="configure-your-application-to-use-service-bus"></a>Настройка приложения для использования служебной шины
Чтобы использовать интерфейсы API служебной шины, требуется следующее.

1. Создайте ссылку на файл автозагрузчика с использованием оператора [require_once][require-once].
2. Ссылка на любые классы, которые могут использоваться.

В следующем примере показано, как включить файл автозагрузчика и сослаться на класс **ServiceBusService**.

> [!NOTE]
> В этом примере (и других примерах в этой статье) предполагается, что установлены клиентские библиотеки PHP для Azure с помощью Composer. При установке библиотек вручную или в качестве пакета PEAR необходимо использовать ссылку на файл автозагрузчика **WindowsAzure.php**.
> 
> 

```php
require_once 'vendor/autoload.php';
use WindowsAzure\Common\ServicesBuilder;
```

В приведенных ниже примерах всегда отображается оператор `require_once`, однако ссылки приводятся только на классы, которые необходимы для выполнения этого примера.

## <a name="set-up-a-service-bus-connection"></a>Настройка подключения к Service Bus
Для создания клиента служебной шины необходимо сначала сформировать правильную строку подключения в следующем формате:

```
Endpoint=[yourEndpoint];SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=[Primary Key]
```

где `Endpoint` обычно имеет формат `https://[yourNamespace].servicebus.windows.net`.

Для создания клиента службы Azure необходимо использовать класс `ServicesBuilder`. Вы можете:

* Передать строку подключения напрямую или
* использовать **CloudConfigurationManager (CCM)** для проверки нескольких внешних источников на наличие строки подключения:
  * По умолчанию предоставляется поддержка одного внешнего источника — переменных среды.
  * Можно добавить новые источники, расширив класс `ConnectionStringSource`.

В приведенных здесь примерах строка подключения передается напрямую.

```php
require_once 'vendor/autoload.php';

use WindowsAzure\Common\ServicesBuilder;

$connectionString = "Endpoint=[yourEndpoint];SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=[Primary Key]";

$serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);
```

## <a name="create-a-topic"></a>Создание раздела
Операции управления разделами служебной шины можно выполнять с помощью класса `ServiceBusRestProxy`. Объект `ServiceBusRestProxy` создается посредством фабричного метода `ServicesBuilder::createServiceBusService` с соответствующей строкой подключения, инкапсулирующей в себе разрешения маркера на управление им.

В приведенном ниже примере показано, как создать экземпляр `ServiceBusRestProxy` и вызвать метод `ServiceBusRestProxy->createTopic` для создания раздела `mytopic` в пространстве имен `MySBNamespace`.

```php
require_once 'vendor/autoload.php';

use WindowsAzure\Common\ServicesBuilder;
use WindowsAzure\Common\ServiceException;
use WindowsAzure\ServiceBus\Models\TopicInfo;

// Create Service Bus REST proxy.
$serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);

try {
    // Create topic.
    $topicInfo = new TopicInfo("mytopic");
    $serviceBusRestProxy->createTopic($topicInfo);
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here: 
    // https://docs.microsoft.com/rest/api/storageservices/Common-REST-API-Error-Codes
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

> [!NOTE]
> Можно использовать метод `listTopics` для объектов `ServiceBusRestProxy`, чтобы проверить, существует ли уже раздел с указанным именем в пространстве имен службы.
> 
> 

## <a name="create-a-subscription"></a>Создание подписки
Подписки на разделы также создаются с помощью метода `ServiceBusRestProxy->createSubscription`. Подписки имеют имена и могут использовать дополнительный фильтр, ограничивающий набор сообщений, доставляемых в виртуальную очередь подписки.

### <a name="create-a-subscription-with-the-default-matchall-filter"></a>Создание подписки с фильтром по умолчанию (MatchAll)
Если при создании подписки не указан фильтр, то используется фильтр по умолчанию — **MatchAll**. Если используется фильтр **MatchAll**, то все сообщения, опубликованные в разделе, помещаются в виртуальную очередь подписки. В следующем примере создается подписка с именем `mysubscription` и используется фильтр по умолчанию **MatchAll**.

```php
require_once 'vendor/autoload.php';

use WindowsAzure\Common\ServicesBuilder;
use WindowsAzure\Common\ServiceException;
use WindowsAzure\ServiceBus\Models\SubscriptionInfo;

// Create Service Bus REST proxy.
$serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);

try {
    // Create subscription.
    $subscriptionInfo = new SubscriptionInfo("mysubscription");
    $serviceBusRestProxy->createSubscription("mytopic", $subscriptionInfo);
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here: 
    // https://docs.microsoft.com/rest/api/storageservices/Common-REST-API-Error-Codes
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

### <a name="create-subscriptions-with-filters"></a>Создание подписок с фильтрами
Кроме того, можно настраивать фильтры, позволяющие определять, какие посылаемые в раздел сообщения должны появляться в рамках конкретной подписки. Самый гибкий тип фильтра, который поддерживается подписками, — это [SqlFilter](/dotnet/api/microsoft.servicebus.messaging.sqlfilter), реализующий подмножество SQL92. Фильтры SQL работают со свойствами сообщений, которые опубликованы в разделе. См. сведения о фильтрах SqlFilter в описании [свойства SqlFilter.SqlExpression][sqlfilter].

> [!NOTE]
> Каждое правило в подписке обрабатывает входящие сообщения независимо, добавляя к подписке свои сообщения о результате. Кроме того, в состав каждой новой подписки по умолчанию входит объект **Правило** с фильтром, который добавляет все сообщения из раздела в подписку. Чтобы получать только сообщения, соответствующие условиям фильтра, необходимо удалить это правило по умолчанию. Правило по умолчанию можно удалить с помощью метода `ServiceBusRestProxy->deleteRule`.
> 
> 

В следующем примере создается подписка `HighMessages`, содержащая фильтр **SqlFilter**, который выбирает только те сообщения, значение настраиваемого свойства `MessageNumber` которых больше 3. Чтобы получить информацию о добавлении пользовательских свойств в сообщения, изучите раздел [Отправка сообщений в раздел](#send-messages-to-a-topic).

```php
$subscriptionInfo = new SubscriptionInfo("HighMessages");
$serviceBusRestProxy->createSubscription("mytopic", $subscriptionInfo);

$serviceBusRestProxy->deleteRule("mytopic", "HighMessages", '$Default');

$ruleInfo = new RuleInfo("HighMessagesRule");
$ruleInfo->withSqlFilter("MessageNumber > 3");
$ruleResult = $serviceBusRestProxy->createRule("mytopic", "HighMessages", $ruleInfo);
```

Этот код требует использования дополнительного пространства имен: `WindowsAzure\ServiceBus\Models\SubscriptionInfo`.

Аналогичным образом в следующем примере создается подписка `LowMessages` с фильтром `SqlFilter`, который выбирает только те сообщения, у которых значение свойства `MessageNumber` меньше или равно 3.

```php
$subscriptionInfo = new SubscriptionInfo("LowMessages");
$serviceBusRestProxy->createSubscription("mytopic", $subscriptionInfo);

$serviceBusRestProxy->deleteRule("mytopic", "LowMessages", '$Default');

$ruleInfo = new RuleInfo("LowMessagesRule");
$ruleInfo->withSqlFilter("MessageNumber <= 3");
$ruleResult = $serviceBusRestProxy->createRule("mytopic", "LowMessages", $ruleInfo);
```

Теперь, если сообщение отправляется в раздел `mytopic`, оно всегда будет доставляться получателям, подписанным на подписку раздела `mysubscription`, и выборочно доставляться получателям, подписанным на подписки разделов `HighMessages` и `LowMessages` (в зависимости от содержания сообщения).

## <a name="send-messages-to-a-topic"></a>Отправка сообщений в раздел
Чтобы отправить сообщение в раздел служебной шины, в приложении вызывается метод `ServiceBusRestProxy->sendTopicMessage`. В следующем примере кода показано, как отправить сообщение в раздел `mytopic`, созданный ранее в пространстве имен службы `MySBNamespace`.

```php
require_once 'vendor/autoload.php';

use WindowsAzure\Common\ServicesBuilder;
use WindowsAzure\Common\ServiceException;
use WindowsAzure\ServiceBus\Models\BrokeredMessage;

// Create Service Bus REST proxy.
$serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);

try {
    // Create message.
    $message = new BrokeredMessage();
    $message->setBody("my message");

    // Send message.
    $serviceBusRestProxy->sendTopicMessage("mytopic", $message);
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here: 
    // https://docs.microsoft.com/rest/api/storageservices/Common-REST-API-Error-Codes
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

Сообщения, отправляемые в разделы служебной шины и получаемые из них, — это экземпляры класса [BrokeredMessage][BrokeredMessage]. У объектов [BrokeredMessage][BrokeredMessage] есть набор стандартных свойств и методов, а также свойства, которые можно использовать для хранения настраиваемых свойств приложения. В следующем примере показано, как отправить пять тестовых сообщений в созданный ранее раздел `mytopic`. Метод `setProperty` используется для добавления настраиваемого свойства (`MessageNumber`) в каждое сообщение. Значение свойства `MessageNumber` меняется в каждом сообщении (с помощью этого можно определить, какие подписки его получают, как показано в разделе [Создание подписки](#create-a-subscription)).

```php
for($i = 0; $i < 5; $i++){
    // Create message.
    $message = new BrokeredMessage();
    $message->setBody("my message ".$i);

    // Set custom property.
    $message->setProperty("MessageNumber", $i);

    // Send message.
    $serviceBusRestProxy->sendTopicMessage("mytopic", $message);
}
```

Разделы служебной шины поддерживают максимальный размер сообщения 256 КБ для [уровня "Стандартный"](service-bus-premium-messaging.md) и 1 МБ для [уровня Premium](service-bus-premium-messaging.md). Максимальный размер заголовка, который содержит стандартные и настраиваемые свойства приложения, — 64 КБ. Ограничения на количество сообщений в разделе нет, но есть максимальный общий размер сообщений, содержащихся в разделе. Максимальный размер раздела ограничен 5 ГБ. Дополнительные сведения о квотах см. в статье [Квоты на служебную шину][Service Bus quotas].

## <a name="receive-messages-from-a-subscription"></a>Получение сообщений из подписки
Самым простым способом получать сообщения от подписки является использование метода `ServiceBusRestProxy->receiveSubscriptionMessage`. Сообщения можно получать в двух различных режимах: [*ReceiveAndDelete* и *PeekLock*](/dotnet/api/microsoft.servicebus.messaging.receivemode). Значение по умолчанию — **PeekLock**.

В режиме [ReceiveAndDelete](/dotnet/api/microsoft.servicebus.messaging.receivemode) получение — это одиночная операция. Когда служебная шина получает запрос на чтение для сообщения в подписке, сообщение помечается как использованное и возвращается в приложение. Режим [ReceiveAndDelete](/dotnet/api/microsoft.servicebus.messaging.receivemode)* представляет собой самую простую модель, которая лучше всего работает в ситуациях, когда приложение может не обрабатывать сообщение при сбое. Чтобы это понять, рассмотрим сценарий, в котором объект-получатель выдает запрос на получение и выходит из строя до его обработки. Так как служебная шина отмечает сообщение как использованное, перезапущенное приложение, начав снова использовать сообщения, пропустит сообщение, использованное до аварийного завершения работы.

В используемом по умолчанию режиме [PeekLock](/dotnet/api/microsoft.servicebus.messaging.receivemode) процесс получения становится двухэтапной операцией, что позволяет поддерживать приложения, не устойчивые к пропуску сообщений. Получив запрос, служебная шина находит следующее сообщение, блокирует его, чтобы предотвратить его получение другими получателями, и возвращает его приложению. Когда приложение завершает обработку сообщения (или сохраняет его для будущей обработки), оно завершает второй этап процесса получения, вызывая метод `ServiceBusRestProxy->deleteMessage` для полученного сообщения. Когда служебная шина обнаруживает вызов метода `deleteMessage`, она помечает сообщение как использованное и удаляет его из очереди.

В следующем примере показывается, как получать и обрабатывать сообщения с помощью режима [PeekLock](/dotnet/api/microsoft.servicebus.messaging.receivemode) (используется по умолчанию). 

```php
require_once 'vendor/autoload.php';

use WindowsAzure\Common\ServicesBuilder;
use WindowsAzure\Common\ServiceException;
use WindowsAzure\ServiceBus\Models\ReceiveMessageOptions;

// Create Service Bus REST proxy.
$serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);

try {
    // Set receive mode to PeekLock (default is ReceiveAndDelete)
    $options = new ReceiveMessageOptions();
    $options->setPeekLock();

    // Get message.
    $message = $serviceBusRestProxy->receiveSubscriptionMessage("mytopic", "mysubscription", $options);

    echo "Body: ".$message->getBody()."<br />";
    echo "MessageID: ".$message->getMessageId()."<br />";

    /*---------------------------
        Process message here.
    ----------------------------*/

    // Delete message. Not necessary if peek lock is not set.
    echo "Deleting message...<br />";
    $serviceBusRestProxy->deleteMessage($message);
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here:
    // https://docs.microsoft.com/rest/api/storageservices/Common-REST-API-Error-Codes
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

## <a name="how-to-handle-application-crashes-and-unreadable-messages"></a>Практическое руководство. Обработка сбоев приложения и нечитаемых сообщений
служебная шина предоставляет функции, помогающие корректно выполнить восстановление после ошибок в приложении или трудностей, возникших при обработке сообщения. Если приложение-получатель по каким-либо причинам не может обработать сообщение, то оно может вызвать метод `unlockMessage` для полученного сообщения (вместо метода `deleteMessage`). После этого служебная шина разблокирует сообщение в очереди и сделает его доступным для приема тем же или другим приложением-пользователем.

Кроме того, с сообщением, блокированным в очереди, связано время ожидания. Если приложение не сможет обработать сообщение в течение времени ожидания (например, при сбое приложения), служебная шина автоматически разблокирует сообщение и снова сделает его доступным для получения.

Если сбой приложения происходит после обработки сообщения, но перед отправкой запроса `deleteMessage`, то это сообщение повторно доставляется в приложение после его перезапуска. Часто такой тип обработки называют *обработать хотя бы один раз*, т. е. каждое сообщение будет обрабатываться по крайней мере один раз, но в некоторых случаях это же сообщение может быть доставлено повторно. Если повторная обработка недопустима, разработчики приложения должны добавить дополнительную логику для обработки повторной доставки сообщений. Часто это достигается с помощью метода `getMessageId` сообщения, который остается постоянным для различных попыток доставки.

## <a name="delete-topics-and-subscriptions"></a>Удаление разделов и подписок
Чтобы удалить раздел или подписку, используйте методы `ServiceBusRestProxy->deleteTopic` или `ServiceBusRestProxy->deleteSubscripton` соответственно. При удалении раздела также удаляются все подписки, зарегистрированные в этом разделе.

Следующий пример показывает, как удалить раздел `mytopic` и зарегистрированные в нем подписки.

```php
require_once 'vendor/autoload.php';

use WindowsAzure\ServiceBus\ServiceBusService;
use WindowsAzure\ServiceBus\ServiceBusSettings;
use WindowsAzure\Common\ServiceException;

// Create Service Bus REST proxy.
$serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);

try {
    // Delete topic.
    $serviceBusRestProxy->deleteTopic("mytopic");
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here: 
    // https://docs.microsoft.com/rest/api/storageservices/Common-REST-API-Error-Codes
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

С помощью метода `deleteSubscription` можно удалить подписку независимо.

```php
$serviceBusRestProxy->deleteSubscription("mytopic", "mysubscription");
```

> [!NOTE]
> Вы можете управлять ресурсами служебной шины с помощью [обозревателя служебной шины](https://github.com/paolosalvatori/ServiceBusExplorer/). Обозреватель служебной шины позволяет без труда подключаться к пространству имен служебной шины и управлять сущностями обмена сообщениями. Средство предоставляет дополнительные возможности, например функции импорта и экспорта или возможность проверять разделы, очереди, подписки, службы ретрансляции, центры уведомлений и концентраторы событий. 

## <a name="next-steps"></a>Дальнейшие действия
См. сведения об [очередях, разделах и подписках][Queues, topics, and subscriptions].

[BrokeredMessage]: /dotnet/api/microsoft.servicebus.messaging.brokeredmessage
[Queues, topics, and subscriptions]: service-bus-queues-topics-subscriptions.md
[sqlfilter]: /dotnet/api/microsoft.servicebus.messaging.sqlfilter
[require-once]: https://php.net/require_once
[Service Bus quotas]: service-bus-quotas.md