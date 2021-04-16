---
title: включить файл
description: включить файл
services: azure-communication-services
author: lakshmans
manager: ankita
ms.service: azure-communication-services
ms.subservice: azure-communication-services
ms.date: 03/11/2021
ms.topic: include
ms.custom: include file
ms.author: lakshmans
ms.openlocfilehash: 2b96d62fb2be27de03964212557446d2e792beb8
ms.sourcegitcommit: 5fd1f72a96f4f343543072eadd7cdec52e86511e
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/01/2021
ms.locfileid: "106113307"
---
Начало работы со Службами коммуникации Azure с помощью пакета SDK для SMS Служб коммуникации Azure для Python для отправки SMS-сообщений.

Выполнение этого краткого руководства предполагает небольшую дополнительную плату в несколько центов США в учетной записи Azure.

<!--**TODO: update all these reference links as the resources go live**

[API reference documentation](../../../references/overview.md) | [Library source code](#todo-sdk-repo) | [Package (PiPy)](#todo-nuget) | [Samples](#todo-samples)-->

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
- [Python](https://www.python.org/downloads/) 2.7, 3.6 или более поздней версии.
- Активный ресурс Служб коммуникации и строка подключения. [Создайте ресурс Служб коммуникации.](../../create-communication-resource.md)
- Номер телефона с поддержкой SMS-сообщений. [Получите номер телефона.](../get-phone-number.md)

### <a name="prerequisite-check"></a>Проверка предварительных условий

- В окне терминала или команды выполните команду `python --version`, чтобы проверить, установлен ли пакет Python.
- Чтобы просмотреть номера телефонов, связанные с ресурсом Служб коммуникации, войдите на [портал Azure](https://portal.azure.com/), перейдите к ресурсу Служб коммуникации и откройте вкладку с **номерами телефонов** в области навигации слева.

## <a name="setting-up"></a>Настройка

### <a name="create-a-new-python-application"></a>Создание приложения Python

Откройте терминал или командное окно, создайте каталог для своего приложения и перейдите к нему.

```console
mkdir sms-quickstart && cd sms-quickstart
```

Используйте текстовый редактор, чтобы создать файл с именем **send-sms.py** в корневом каталоге проекта и добавить структуру для программы, включая базовую обработку исключений. В следующих разделах показано, как добавить в этот файл весь исходный код для примера из этого краткого руководства.

```python
import os
from azure.communication.sms import SmsClient

try:
    # Quickstart code goes here
except Exception as ex:
    print('Exception:')
    print(ex)
```

### <a name="install-the-package"></a>Установка пакета

Оставаясь в каталоге приложения, установите пакет SDK для SMS Служб коммуникации для пакета Python с помощью команды `pip install`.

```console
pip install azure-communication-sms
```

## <a name="object-model"></a>Объектная модель

Следующие классы и интерфейсы реализуют некоторые основные функции пакета SDK для SMS Служб коммуникации для Python.

| Имя                                  | Описание                                                  |
| ------------------------------------- | ------------------------------------------------------------ |
| SmsClient | Этот класс требуется для реализации всех функций обмена текстовыми сообщениями. Его экземпляр можно создать на основе сведений о подписке и использовать для отправки SMS.                                                                                                                 |
| SmsSendResult               | Этот класс содержит результат, полученный от службы SMS.                                          |

## <a name="authenticate-the-client"></a>Аутентификация клиента

Создайте экземпляр **SmsClient** с использованием строки подключения. См. сведения о том, как [управлять строкой подключения ресурса](../../create-communication-resource.md#store-your-connection-string).

```python
# Create the SmsClient object which will be used to send SMS messages
sms_client = SmsClient.from_connection_string(<connection_string>)
```
Для простоты в этом кратком руководстве мы используем строки подключения. Но в рабочей среде мы рекомендуем использовать [управляемые удостоверения](../../../quickstarts/managed-identity.md), так как они являются более безопасными и управляемыми в требуемом масштабе.


## <a name="send-a-11-sms-message"></a>Отправка личного текстового сообщения

Чтобы отправить текстовое сообщение одному получателю, вызовите метод ```send``` из **SmsClient** с номером телефона этого получателя. Кроме того, вы можете передать необязательные параметры для указания того, должен ли быть включен отчет о доставке, а также задать пользовательские теги. Добавьте следующий код в конец блока `try` в **send-sms.py**:

```python

# calling send() with sms values
sms_responses = sms_client.send(
    from_="<from-phone-number>",
    to="<to-phone-number>",
    message="Hello World via SMS",
    enable_delivery_report=True, # optional property
    tag="custom-tag") # optional property

```

Необходимо изменить `<from-phone-number>` на номер телефона с поддержкой SMS, связанный с ресурсом Служб коммуникации, и `<to-phone-number>` на номер телефона, на который вы хотите отправить сообщение.

> [!WARNING]
> Обратите внимание, что номера телефонов должны быть указаны в формате международного стандарта E.164, (например, +14255550123).

## <a name="send-a-1n-sms-message"></a>Отправка группового текстового сообщения

Чтобы отправить текстовое сообщение списку получателей, вызовите метод ```send``` из **SmsClient** со списком номеров телефонов получателей. Кроме того, вы можете передать необязательные параметры для указания того, должен ли быть включен отчет о доставке, а также задать пользовательские теги. Добавьте следующий код в конец блока `try` в **send-sms.py**:

```python

# calling send() with sms values
sms_responses = sms_client.send(
    from_="<from-phone-number>",
    to=["<to-phone-number-1>", "<to-phone-number-2>"],
    message="Hello World via SMS",
    enable_delivery_report=True, # optional property
    tag="custom-tag") # optional property

```

Необходимо изменить `<from-phone-number>` на номер телефона с поддержкой SMS, связанный с ресурсом Служб коммуникации, и `<to-phone-number-1>` `<to-phone-number-2>` на номера телефона, на которые вы хотите отправить сообщение.

> [!WARNING]
> Обратите внимание, что номера телефонов должны быть указаны в формате международного стандарта E.164, (например, +14255550123).

## <a name="optional-parameters"></a>Необязательные параметры

Параметр `enable_delivery_report` является необязательным. Его можно использовать для настройки отчетов о доставке. Это полезно, если вы хотите, чтобы при доставке SMS-сообщений создавались события. Сведения о настройке отчетов о доставке SMS-сообщений см. в кратком руководстве [Обработка событий SMS-сообщений](../handle-sms-events.md).

`tag` — это необязательный параметр, который можно использовать для применения тегов к отчету о доставке.

## <a name="run-the-code"></a>Выполнение кода
Запустите приложение из каталога приложения с помощью команды `python`.

```console
python send-sms.py
```

Полный скрипт Python должен выглядеть примерно следующим образом:

```python

import os
from azure.communication.sms import SmsClient

try:
    # Create the SmsClient object which will be used to send SMS messages
    sms_client = SmsClient.from_connection_string("<connection string>")
    # calling send() with sms values
    sms_responses = sms_client.send(
       from_="<from-phone-number>",
       to="<to-phone-number>",
       message="Hello World via SMS",
       enable_delivery_report=True, # optional property
       tag="custom-tag") # optional property

except Exception as ex:
    print('Exception:')
    print(ex)
```
