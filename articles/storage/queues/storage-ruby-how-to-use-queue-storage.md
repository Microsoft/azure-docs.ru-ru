---
title: Использование хранилища очередей из Ruby в службе хранилища Azure
description: Узнайте, как использовать хранилище очередей Azure для создания и удаления очередей, а также для вставки, получения и удаления сообщений. Примеры кода написаны на Ruby.
author: mhopkins-msft
ms.author: mhopkins
ms.reviewer: dineshm
ms.date: 12/08/2016
ms.topic: how-to
ms.service: storage
ms.subservice: queues
ms.openlocfilehash: 8558949e49bcf551c9276458d375fb9ac9636184
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97587668"
---
# <a name="how-to-use-queue-storage-from-ruby"></a>Использование хранилища очередей из Ruby

[!INCLUDE [storage-selector-queue-include](../../../includes/storage-selector-queue-include.md)]

[!INCLUDE [storage-try-azure-tools-queues](../../../includes/storage-try-azure-tools-queues.md)]

## <a name="overview"></a>Обзор

В этом руководстве показано, как реализовать типичные сценарии с использованием службы хранилища очередей Microsoft Azure. Примеры написаны с помощью Ruby Azure API. Здесь описаны такие сценарии, как **вставка**, **просмотр**, **получение** и **удаление** сообщений очереди, а также **создание и удаление очередей**.

[!INCLUDE [storage-queue-concepts-include](../../../includes/storage-queue-concepts-include.md)]

[!INCLUDE [storage-create-account-include](../../../includes/storage-create-account-include.md)]

## <a name="create-a-ruby-application"></a>Создание приложения Ruby

Создайте приложение Ruby. Инструкции см. [в статье Создание приложения Ruby в службе приложений на платформе Linux](../../app-service/quickstart-ruby.md).

## <a name="configure-your-application-to-access-storage"></a>Настройка приложения для доступа к хранилищу

Чтобы использовать службу хранилища Azure, необходимо скачать и использовать пакет Ruby Azure, который содержит набор удобных библиотек, взаимодействующих со службами RESTFUL хранилища.

<!-- docutune:ignore Terminal -->

### <a name="use-rubygems-to-obtain-the-package"></a>Использование RubyGems для получения пакета

1. Используйте интерфейс командной строки, например PowerShell (Windows), Terminal (Mac) или Bash (Unix).
2. Введите `gem install Azure` в командном окне, чтобы установить драгоценный камень и зависимости.

### <a name="import-the-package"></a>Импорт пакета

Используйте свой любимый текстовый редактор, чтобы добавить следующий код в начало файла Ruby, где планируется использовать хранилище.

```ruby
require "azure"
```

## <a name="setup-an-azure-storage-connection"></a>Настройка подключения к службе хранилища Azure

Модуль Azure будет считывать переменные среды `AZURE_STORAGE_ACCOUNT` и сведения, `AZURE_STORAGE_ACCESS_KEY` необходимые для подключения к учетной записи хранения Azure. Если эти переменные среды не заданы, необходимо указать сведения об учетной записи перед использованием `Azure::QueueService` со следующим кодом:

```ruby
Azure.config.storage_account_name = "<your azure storage account>"
Azure.config.storage_access_key = "<your Azure storage access key>"
```

Вот как можно получить эти значения из классический учетной записи хранения или учетной записи хранения Resource Manager на портале Azure.

1. Войдите на [портал Azure](https://portal.azure.com).
2. Перейдите к учетной записи хранения, которая будет использоваться.
3. В колонке **Параметры** справа щелкните **ключи доступа**.
4. В появившейся колонке **ключи доступа** вы увидите ключ доступа 1 и ключ доступа 2. Можно использовать любой из них.
5. Щелкните значок копирования, чтобы скопировать ключ в буфер обмена.

## <a name="how-to-create-a-queue"></a>Практическое руководство. Создание очереди

Следующий код создает `Azure::QueueService` объект, который позволяет работать с очередями.

```ruby
azure_queue_service = Azure::QueueService.new
```

Используйте `create_queue()` метод, чтобы создать очередь с указанным именем.

```ruby
begin
  azure_queue_service.create_queue("test-queue")
rescue
  puts $!
end
```

## <a name="how-to-insert-a-message-into-a-queue"></a>Практическое руководство. Вставка сообщения в очередь

Чтобы вставить сообщение в очередь, используйте `create_message()` метод, чтобы создать новое сообщение и добавить его в очередь.

```ruby
azure_queue_service.create_message("test-queue", "test message")
```

## <a name="how-to-peek-at-the-next-message"></a>Практическое руководство. Просмотр следующего сообщения

Можно просмотреть сообщение в начале очереди, не удаляя его из очереди, вызвав `peek_messages()` метод. По умолчанию `peek_messages()` просматривает одно сообщение. Вы также можете указать количество сообщений для просмотра.

```ruby
result = azure_queue_service.peek_messages("test-queue",
  {:number_of_messages => 10})
```

## <a name="how-to-dequeue-the-next-message"></a>Практическое руководство. Удаление следующего сообщения из очереди

Вы можете удалить сообщение из очереди в два этапа.

1. При вызове `list_messages()` вы получаете следующее сообщение в очереди по умолчанию. Вы также можете указать количество сообщений для получения. Сообщения, возвращенные из, `list_messages()` становятся невидимыми для любого другого кода, считывающего сообщения из этой очереди. Время ожидания видимости (в секундах) передается в качестве параметра.
2. Чтобы завершить удаление сообщения из очереди, необходимо также вызвать метод `delete_message()` .

Этот двухэтапный процесс удаления сообщения позволяет удостовериться, что если коду не удастся обработать сообщение из-за сбоя оборудования или программного обеспечения, другой экземпляр кода сможет получить то же сообщение и повторить попытку. Код вызывается `delete_message()` сразу после обработки сообщения.

```ruby
messages = azure_queue_service.list_messages("test-queue", 30)
azure_queue_service.delete_message("test-queue",
  messages[0].id, messages[0].pop_receipt)
```

## <a name="how-to-change-the-contents-of-a-queued-message"></a>Практическое руководство. Изменение содержимого сообщения в очереди

Вы можете изменить содержимое сообщения непосредственно в очереди. В следующем коде метод используется `update_message()` для обновления сообщения. Метод возвращает кортеж, содержащий уведомление об открытии сообщения очереди, и значение времени в формате UTC `DateTime` , которое представляет, когда сообщение будет видно в очереди.

```ruby
message = azure_queue_service.list_messages("test-queue", 30)
pop_receipt, time_next_visible = azure_queue_service.update_message(
  "test-queue", message.id, message.pop_receipt, "updated test message",
  30)
```

## <a name="how-to-additional-options-for-dequeuing-messages"></a>Поочередно: дополнительные параметры для удаления сообщений из очереди

Способ извлечения сообщения из очереди можно настроить двумя способами.

1. Можно получить пакет сообщения.
2. Можно задать более длительное или короткое время ожидания невидимости, чтобы предоставить коду больше или меньше времени на полную обработку каждого сообщения.

В следующем примере кода метод используется `list_messages()` для получения 15 сообщений в одном вызове. Затем код выводит и удаляет все сообщения. Он также задает время ожидания невидимости 5 минут для каждого сообщения.

```ruby
azure_queue_service.list_messages("test-queue", 300
  {:number_of_messages => 15}).each do |m|
  puts m.message_text
  azure_queue_service.delete_message("test-queue", m.id, m.pop_receipt)
end
```

## <a name="how-to-get-the-queue-length"></a>Практическое руководство. Получение длины очереди

Вы можете получить приблизительное количество сообщений в очереди. `get_queue_metadata()`Метод возвращает приблизительное число сообщений и другие метаданные очереди.

```ruby
message_count, metadata = azure_queue_service.get_queue_metadata(
  "test-queue")
```

## <a name="how-to-delete-a-queue"></a>Практическое руководство. Удаление очереди

Чтобы удалить очередь и все сообщения, содержащиеся в ней, вызовите `delete_queue()` метод для объекта очереди.

```ruby
azure_queue_service.delete_queue("test-queue")
```

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы узнали основные сведения о хранилище очередей, перейдите по следующим ссылкам, чтобы узнать о более сложных задачах хранилища.

- Посетите [блог команды разработчиков службы хранилища Azure](/archive/blogs/windowsazurestorage/)
- Посетите репозиторий [Azure SDK для Ruby](https://github.com/WindowsAzure/azure-sdk-for-ruby) в GitHub

Сравнение хранилища очередей Azure, рассмотренного в этой статье, и очередей служебной шины Azure, которые обсуждались в разделе [Использование очередей служебной шины](https://azure.microsoft.com/develop/ruby/how-to-guides/service-bus-queues/), см. в статьях [хранилище очередей Azure и очереди служебной шины: сравнение и отличие](../../service-bus-messaging/service-bus-azure-and-service-bus-queues-compared-contrasted.md)
