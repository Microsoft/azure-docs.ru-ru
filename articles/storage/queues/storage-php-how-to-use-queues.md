---
title: Использование хранилища очередей из PHP в службе хранилища Azure
description: Узнайте, как использовать службу хранилища очередей Azure для создания и удаления очередей, а также для вставки, получения и удаления сообщений. Примеры написаны на PHP.
author: mhopkins-msft
ms.author: mhopkins
ms.reviewer: dineshm
ms.date: 01/11/2018
ms.topic: how-to
ms.service: storage
ms.subservice: queues
ms.openlocfilehash: 69369d81892a10c390aa31a2c46f79fdfa41206d
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97592030"
---
# <a name="how-to-use-queue-storage-from-php"></a>Использование хранилища очередей из PHP

[!INCLUDE [storage-selector-queue-include](../../../includes/storage-selector-queue-include.md)]

[!INCLUDE [storage-try-azure-tools-queues](../../../includes/storage-try-azure-tools-queues.md)]

В этом руководство показано, как выполнять типичные сценарии с помощью службы хранилища очередей Azure. Примеры написаны с помощью классов из [клиентской библиотеки службы хранилища Azure для PHP](https://github.com/Azure/azure-storage-php). Здесь описаны такие сценарии, как вставка, просмотр, получение и удаление сообщений очереди, а также создание и удаление очередей.

[!INCLUDE [storage-queue-concepts-include](../../../includes/storage-queue-concepts-include.md)]

[!INCLUDE [storage-create-account-include](../../../includes/storage-create-account-include.md)]

## <a name="create-a-php-application"></a>Создание приложения PHP

Единственным требованием для создания приложения PHP, обращающегося к хранилищу очередей Azure, является ссылка на классы в [клиентской библиотеке службы хранилища Azure для PHP](https://github.com/Azure/azure-storage-php) из кода. Можно использовать любые средства разработки для создания приложения, включая программу "Блокнот".

В этом разделе вы используете функции службы хранилища очередей, которые можно вызывать в приложении PHP локально или в коде, работающем в веб-приложении Azure.

## <a name="get-the-azure-client-libraries"></a>Получение клиентских библиотек Azure

### <a name="install-via-composer"></a>Установка с помощью Composer

1. Создайте файл с именем `composer.json` в корне проекта и добавьте в него следующий код:

    ```json
    {
      "require": {
        "microsoft/azure-storage-queue": "*"
      }
    }
    ```

2. Скачайте [`composer.phar`](https://getcomposer.org/composer.phar) в корневую папку проекта.

3. Откройте командную строку и выполните следующую команду в корневом каталоге проекта:

    ```console
    php composer.phar install
    ```

Также можно воспользоваться [клиентской библиотекой PHP службы хранилища Azure](https://github.com/Azure/azure-storage-php) на GitHub, чтобы клонировать исходный код.

## <a name="configure-your-application-to-access-queue-storage"></a>Настройка приложения для доступа к хранилищу очередей

Чтобы использовать API для хранилища очередей Azure, необходимо:

1. Сослаться на файл автозагрузчика с помощью [`require_once`](https://www.php.net/manual/en/function.require-once.php) инструкции.
2. Ссылка на любые классы, которые могут использоваться.

В следующем примере показано, как включить файл автозагрузчика и добавить ссылку на класс `QueueRestProxy`.

```php
require_once 'vendor/autoload.php';
use MicrosoftAzure\Storage\Queue\QueueRestProxy;
```

В следующих примерах `require_once` оператор отображается всегда, но на него ссылаются только классы, необходимые для выполнения примера.

## <a name="set-up-an-azure-storage-connection"></a>Настройка подключения к службе хранилища Azure

Чтобы создать экземпляр клиента хранилища очередей Azure, необходимо сначала создать допустимую строку подключения. Формат строки подключения к хранилищу очередей выглядит следующим образом.

Для доступа к службе в режиме реального времени:

```php
DefaultEndpointsProtocol=[http|https];AccountName=[yourAccount];AccountKey=[yourKey]
```

Для доступа к хранилищу эмулятора:

```php
UseDevelopmentStorage=true
```

Чтобы создать клиент хранилища очередей Azure, необходимо использовать `QueueRestProxy` класс. Для этого можно использовать один из следующих методов.

- Передать строку подключения напрямую или
- Используйте переменные среды в веб-приложении для хранения строки подключения. Дополнительные сведения о настройке строк подключения см. в разделе [Настройка веб-приложений в службе приложений Azure](../../app-service/configure-common.md).

В приведенных здесь примерах строка подключения передается напрямую.

```php
require_once 'vendor/autoload.php';

use MicrosoftAzure\Storage\Queue\QueueRestProxy;

$connectionString = "DefaultEndpointsProtocol=http;AccountName=<accountNameHere>;AccountKey=<accountKeyHere>";
$queueClient = QueueRestProxy::createQueueService($connectionString);
```

## <a name="create-a-queue"></a>Создание очереди

`QueueRestProxy`Объект позволяет создать очередь с помощью `CreateQueue` метода. При создании очереди можно задать ее параметры, однако это не является обязательным. В этом примере показано, как задать метаданные в очереди.

```php
require_once 'vendor/autoload.php';

use MicrosoftAzure\Storage\Queue\QueueRestProxy;
use MicrosoftAzure\Storage\Common\Exceptions\ServiceException;
use MicrosoftAzure\Storage\Queue\Models\CreateQueueOptions;

$connectionString = "DefaultEndpointsProtocol=http;AccountName=<accountNameHere>;AccountKey=<accountKeyHere>";

// Create queue REST proxy.
$queueClient = QueueRestProxy::createQueueService($connectionString);

// OPTIONAL: Set queue metadata.
$createQueueOptions = new CreateQueueOptions();
$createQueueOptions->addMetaData("key1", "value1");
$createQueueOptions->addMetaData("key2", "value2");

try    {
    // Create queue.
    $queueClient->createQueue("myqueue", $createQueueOptions);
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here:
    // https://msdn.microsoft.com/library/azure/dd179446.aspx
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

> [!NOTE]
> Не следует полагаться на учет регистра для ключей метаданных. Все параметры считываются из службы в нижнем регистре.

## <a name="add-a-message-to-a-queue"></a>Добавление сообщения в очередь

Чтобы добавить сообщение в очередь, используйте `QueueRestProxy->createMessage` . Этот метод принимает имя очереди, текст сообщения и параметры сообщения (которые не являются обязательными).

```php
require_once 'vendor/autoload.php';

use MicrosoftAzure\Storage\Queue\QueueRestProxy;
use MicrosoftAzure\Storage\Common\Exceptions\ServiceException;
use MicrosoftAzure\Storage\Queue\Models\CreateMessageOptions;

$connectionString = "DefaultEndpointsProtocol=http;AccountName=<accountNameHere>;AccountKey=<accountKeyHere>";

// Create queue REST proxy.
$queueClient = QueueRestProxy::createQueueService($connectionString);

try    {
    // Create message.
    $queueClient->createMessage("myqueue", "Hello, World");
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here:
    // https://msdn.microsoft.com/library/azure/dd179446.aspx
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

## <a name="peek-at-the-next-message"></a>Просмотр следующего сообщения

Можно просмотреть одно или несколько сообщений в начале очереди, не удаляя их из очереди путем вызова `QueueRestProxy->peekMessages` . По умолчанию `peekMessage` метод возвращает одно сообщение, но это значение можно изменить с помощью `PeekMessagesOptions->setNumberOfMessages` метода.

```php
require_once 'vendor/autoload.php';

use MicrosoftAzure\Storage\Queue\QueueRestProxy;
use MicrosoftAzure\Storage\Common\Exceptions\ServiceException;
use MicrosoftAzure\Storage\Queue\Models\PeekMessagesOptions;

$connectionString = "DefaultEndpointsProtocol=http;AccountName=<accountNameHere>;AccountKey=<accountKeyHere>";

// Create queue REST proxy.
$queueClient = QueueRestProxy::createQueueService($connectionString);

// OPTIONAL: Set peek message options.
$message_options = new PeekMessagesOptions();
$message_options->setNumberOfMessages(1); // Default value is 1.

try    {
    $peekMessagesResult = $queueClient->peekMessages("myqueue", $message_options);
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here:
    // https://msdn.microsoft.com/library/azure/dd179446.aspx
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}

$messages = $peekMessagesResult->getQueueMessages();

// View messages.
$messageCount = count($messages);
if($messageCount <= 0){
    echo "There are no messages.<br />";
}
else{
    foreach($messages as $message)    {
        echo "Peeked message:<br />";
        echo "Message Id: ".$message->getMessageId()."<br />";
        echo "Date: ".date_format($message->getInsertionDate(), 'Y-m-d')."<br />";
        echo "Message text: ".$message->getMessageText()."<br /><br />";
    }
}
```

## <a name="de-queue-the-next-message"></a>Удаление следующего сообщения из очереди

Ваш код удаляет сообщение из очереди в два этапа. Сначала вызывается метод `QueueRestProxy->listMessages` , который делает сообщение невидимым для любого другого кода, считывающего из очереди. По умолчанию это сообщение остается невидимым в течение 30 секунд. (Если сообщение не удаляется в течение этого периода времени, оно снова становится видимым в очереди.) Чтобы завершить удаление сообщения из очереди, необходимо вызвать метод `QueueRestProxy->deleteMessage` . Этот двухэтапный процесс удаления сообщения позволяет удостовериться, что если коду не удастся обработать сообщение из-за сбоя оборудования или программного обеспечения, другой экземпляр кода сможет получить то же сообщение и повторить попытку. Код вызывается `deleteMessage` сразу после обработки сообщения.

```php
require_once 'vendor/autoload.php';

use MicrosoftAzure\Storage\Queue\QueueRestProxy;
use MicrosoftAzure\Storage\Common\Exceptions\ServiceException;

$connectionString = "DefaultEndpointsProtocol=http;AccountName=<accountNameHere>;AccountKey=<accountKeyHere>";

// Create queue REST proxy.
$queueClient = QueueRestProxy::createQueueService($connectionString);

// Get message.
$listMessagesResult = $queueClient->listMessages("myqueue");
$messages = $listMessagesResult->getQueueMessages();
$message = $messages[0];

/* ---------------------
    Process message.
   --------------------- */

// Get message ID and pop receipt.
$messageId = $message->getMessageId();
$popReceipt = $message->getPopReceipt();

try    {
    // Delete message.
    $queueClient->deleteMessage("myqueue", $messageId, $popReceipt);
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here:
    // https://msdn.microsoft.com/library/azure/dd179446.aspx
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

## <a name="change-the-contents-of-a-queued-message"></a>Изменение содержимого сообщения в очереди

Вы можете изменить содержимое сообщения на месте в очереди, вызвав метод `QueueRestProxy->updateMessage` . Если сообщение представляет собой рабочую задачу, можно использовать эту функцию для обновления состояния рабочей задачи. Следующий код добавляет новое содержимое в сообщение очереди и продлевает время ожидания видимости еще на 60 секунд. Это сохраняет состояние работы, связанной с данным сообщением, и позволяет клиенту продолжить работу с сообщением на протяжении еще одной минуты. Эту методику можно использовать для контроля многошаговых рабочих процессов в сообщениях очереди без необходимости запуска с начала, если шаг обработки завершается сбоем из-за сбоя оборудования или программного обеспечения. Обычно также сохраняется счетчик повторов. Если количество повторов сообщения превысит *n* раз, его нужно удалить. Это обеспечивает защиту от сообщений, которые инициируют ошибку приложения при каждой попытке обработки.

```php
require_once 'vendor/autoload.php';

use MicrosoftAzure\Storage\Queue\QueueRestProxy;
use MicrosoftAzure\Storage\Common\Exceptions\ServiceException;

// Create queue REST proxy.
$queueClient = QueueRestProxy::createQueueService($connectionString);

$connectionString = "DefaultEndpointsProtocol=http;AccountName=<accountNameHere>;AccountKey=<accountKeyHere>";

// Get message.
$listMessagesResult = $queueClient->listMessages("myqueue");
$messages = $listMessagesResult->getQueueMessages();
$message = $messages[0];

// Define new message properties.
$new_message_text = "New message text.";
$new_visibility_timeout = 5; // Measured in seconds.

// Get message ID and pop receipt.
$messageId = $message->getMessageId();
$popReceipt = $message->getPopReceipt();

try    {
    // Update message.
    $queueClient->updateMessage("myqueue",
                                $messageId,
                                $popReceipt,
                                $new_message_text,
                                $new_visibility_timeout);
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here:
    // https://msdn.microsoft.com/library/azure/dd179446.aspx
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

## <a name="additional-options-for-dequeuing-messages"></a>Дополнительные варианты удаления сообщений из очереди

Существует два способа настройки извлечения сообщения из очереди. Во-первых, можно получить пакет сообщений (до 32 сообщений). Во-вторых, можно задать более длительное или короткое время ожидания видимости, чтобы предоставить коду больше или меньше времени на полную обработку каждого сообщения. В следующем примере кода метод используется `getMessages` для получения 16 сообщений в одном вызове. Затем он обрабатывает каждое сообщение с помощью `for` цикла. Он также задает время ожидания невидимости 5 минут для каждого сообщения.

```php
require_once 'vendor/autoload.php';

use MicrosoftAzure\Storage\Queue\QueueRestProxy;
use MicrosoftAzure\Storage\Common\Exceptions\ServiceException;
use MicrosoftAzure\Storage\Queue\Models\ListMessagesOptions;

$connectionString = "DefaultEndpointsProtocol=http;AccountName=<accountNameHere>;AccountKey=<accountKeyHere>";

// Create queue REST proxy.
$queueClient = QueueRestProxy::createQueueService($connectionString);

// Set list message options.
$message_options = new ListMessagesOptions();
$message_options->setVisibilityTimeoutInSeconds(300);
$message_options->setNumberOfMessages(16);

// Get messages.
try{
    $listMessagesResult = $queueClient->listMessages("myqueue",
                                                     $message_options);
    $messages = $listMessagesResult->getQueueMessages();

    foreach($messages as $message){

        /* ---------------------
            Process message.
        --------------------- */

        // Get message Id and pop receipt.
        $messageId = $message->getMessageId();
        $popReceipt = $message->getPopReceipt();

        // Delete message.
        $queueClient->deleteMessage("myqueue", $messageId, $popReceipt);
    }
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here:
    // https://msdn.microsoft.com/library/azure/dd179446.aspx
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

## <a name="get-queue-length"></a>Получение длины очереди

Вы можете узнать приблизительное количество сообщений в очереди. `QueueRestProxy->getQueueMetadata`Метод извлекает метаданные об очереди. Вызов `getApproximateMessageCount` метода для возвращенного объекта позволяет получить количество сообщений в очереди. Число является приблизительным, так как сообщения могут добавляться или удаляться после ответа хранилища очередей на ваш запрос.

```php
require_once 'vendor/autoload.php';

use MicrosoftAzure\Storage\Queue\QueueRestProxy;
use MicrosoftAzure\Storage\Common\Exceptions\ServiceException;

$connectionString = "DefaultEndpointsProtocol=http;AccountName=<accountNameHere>;AccountKey=<accountKeyHere>";

// Create queue REST proxy.
$queueClient = QueueRestProxy::createQueueService($connectionString);

try    {
    // Get queue metadata.
    $queue_metadata = $queueClient->getQueueMetadata("myqueue");
    $approx_msg_count = $queue_metadata->getApproximateMessageCount();
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here:
    // https://msdn.microsoft.com/library/azure/dd179446.aspx
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}

echo $approx_msg_count;
```

## <a name="delete-a-queue"></a>Удаление очереди

Чтобы удалить очередь и все сообщения в ней, вызовите `QueueRestProxy->deleteQueue` метод.

```php
require_once 'vendor/autoload.php';

use MicrosoftAzure\Storage\Queue\QueueRestProxy;
use MicrosoftAzure\Storage\Common\Exceptions\ServiceException;

$connectionString = "DefaultEndpointsProtocol=http;AccountName=<accountNameHere>;AccountKey=<accountKeyHere>";

// Create queue REST proxy.
$queueClient = QueueRestProxy::createQueueService($connectionString);

try    {
    // Delete queue.
    $queueClient->deleteQueue("myqueue");
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here:
    // https://msdn.microsoft.com/library/azure/dd179446.aspx
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы узнали основные сведения о хранилище очередей Azure, перейдите по следующим ссылкам, чтобы узнать о более сложных задачах хранилища.

- Посетите [Справочник по API для клиентской библиотеки службы хранилища Azure](https://azure.github.io/azure-storage-php/)
- См. [Пример расширенной очереди](https://github.com/Azure/azure-storage-php/blob/master/samples/QueueSamples.php).

Дополнительные сведения см. в [центре разработчиков PHP](https://azure.microsoft.com/develop/php/).
