---
title: Управление подключением Центра Интернета вещей & надежным обменом сообщениями с помощью пакетов SDK для устройств
description: Узнайте, как улучшить связь с устройствами и обмен сообщениями при использовании пакетов SDK для устройств центра Интернета вещей Azure.
services: iot-hub
author: robinsh
ms.author: robinsh
ms.date: 07/07/2018
ms.topic: article
ms.service: iot-hub
ms.custom:
- amqp
- mqtt
- devx-track-csharp
ms.openlocfilehash: 4135f1c29ce868c04c23b3ce97529037b123c9fa
ms.sourcegitcommit: 3ee3045f6106175e59d1bd279130f4933456d5ff
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106077431"
---
# <a name="manage-connectivity-and-reliable-messaging-by-using-azure-iot-hub-device-sdks"></a>Управление возможностью подключения и надежным обменом сообщениями с помощью пакетов SDK для устройств Центра Интернета вещей Azure

В этой статье описано, как повысить устойчивость при разработке приложений для устройств. Также вы узнаете, как использовать возможность подключения и надежного обмена сообщениями в пакетах SDK для устройств Azure IoT В этом руководстве описаны следующие сценарии:

* восстановление потерянного сетевого подключения;

* переключение между разными сетевыми подключениями;

* повторное подключение после временных ошибок с подключением к службе.

Реализация будет зависеть от конкретного языка. Дополнительные сведения см. в документации по API для конкретного пакета SDK.

* [Пакет SDK для C/iOS](https://github.com/azure/azure-iot-sdk-c)

* [Пакет SDK для .NET](https://github.com/Azure/azure-iot-sdk-csharp/blob/master/iothub/device/devdoc/retrypolicy.md)

* [пакет SDK для Java](https://github.com/Azure/azure-iot-sdk-java/blob/master/device/iot-device-client/devdoc/requirement_docs/com/microsoft/azure/iothub/retryPolicy.md)

* [Пакет SDK для Node](https://github.com/Azure/azure-iot-sdk-node/wiki/Connectivity-and-Retries#types-of-errors-and-how-to-detect-them)

* [Пакет SDK для Python](https://github.com/Azure/azure-iot-sdk-python) (еще не обеспечена надежность)

## <a name="designing-for-resiliency"></a>Проектирование для обеспечения устойчивости

Устройства IoT часто используют непостоянные или нестабильные сетевые подключения, например сети GSM или спутниковую связь. Это означает, что могут возникать ошибки при взаимодействии устройств с облачными службами из-за временных сбоев на уровне доступности службы, инфраструктуры и т. д. Работающее на устройстве приложение должно управлять механизмами подключения, восстановления подключения и повторных попыток при отправке и получении сообщений. Кроме того, требования к стратегии повторных попыток очень сильно меняются в зависимости от сценария, контекста и возможностей устройства Интернета вещей.

Пакеты SDK для устройств Центра Интернета вещей Azure помогают упростить подключение и обмен данными в сценариях "из облака на устройство" и "с устройства в облако". Эти пакеты SDK предоставляют надежный способ подключения к Центру Интернета вещей Azure и широкий набор средств для отправки и получения сообщений. Разработчики также могут изменить существующую реализацию, чтобы внедрить стратегию повторных попыток в соответствии со сценарием.

Соответствующие возможности пакета SDK, которые поддерживают подключения и надежный обмен сообщениями, рассматриваются в следующих разделах.

## <a name="connection-and-retry"></a>Подключение и повторные попытки

В этом разделе приводится обзор шаблонов повторного подключения и повторных попыток, доступных для управления подключениями. Также приведены рекомендации по реализации разных политик повторных попыток в приложении устройства, а также перечислены соответствующие API из пакетов SDK для устройств.

### <a name="error-patterns"></a>Шаблоны ошибок

Сбои подключения могут происходить на разных уровнях:

* ошибки сети, например отключенные сокеты и ошибки разрешения имен;

* ошибки уровня транспортных протоколов HTTP, AMQP и MQTT, например отключение каналов или истечение срока действия сеансов;

* ошибки уровня приложения, которые происходят из-за локальных ошибок, например недопустимых учетных данных или особенностей поведения службы (превышение квоты, регулирование и т. п.).

Пакеты SDK для устройств обнаруживают ошибки на всех трех уровнях. С помощью пакетов SDK для устройств невозможно обнаружить и обработать ошибки, связанные с ОС и оборудованием. Пакеты SDK спроектированы на основе [рекомендаций по обработке временных сбоев](/azure/architecture/best-practices/transient-faults#general-guidelines) Центра архитектуры Azure.

### <a name="retry-patterns"></a>Шаблоны повтора

Следующие шаги описывают процесс повторных попыток при обнаружении ошибок подключения.

1. Пакет SDK перехватывает ошибку и соответствующую ошибку сети, протокола или приложения.

2. Пакет SDK применяет фильтрацию ошибок для определения типа ошибки и принимает решение о необходимости повторных попыток.

3. Если пакет SDK обнаруживает **неустранимую ошибку**, он останавливает все операции подключения, отправки и получения. Пакет SDK уведомляет пользователя. К неустранимым ошибкам относятся, например, ошибка аутентификации и неправильные параметры конечной точки.

4. Если пакет SDK обнаруживает **устранимую ошибку**, он выполняет повторные попытки в соответствии с настроенной политикой повтора, пока не будет превышено указанное время ожидания.  Обратите внимание, что в пакете SDK используется политика повтора по умолчанию **Экспоненциальная задержка с дрожанием**.
5. По истечении указанного времени ожидания пакет SDK прекращает попытки подключения или отправки. Затем он уведомляет пользователя.

6. Пакет SDK позволяет пользователю подключить обратный вызов для получения изменений состояния подключения.

Пакеты SDK поддерживают три политики повтора.

* **Экспоненциальная задержка с дрожанием**. Эта политика повтора применяется по умолчанию и действует довольно агрессивно в начале периода с постепенным замедлением вплоть до максимального периода задержки. Она спроектирована на основе [рекомендаций по логике повтора из центра архитектуры Azure](/azure/architecture/best-practices/retry-service-specific). 

* **Пользовательская политика повтора**. Для некоторых языков пакет SDK позволяет создать пользовательскую политику повтора, которая лучше соответствует конкретному сценарию работы, и поместить ее в объект RetryPolicy. Настраиваемые повторные попытки недоступны в пакете SDK для C и в настоящее время не поддерживаются в пакете SDK для Python. Пакет SDK для Python выполняет повторные подключения по мере необходимости.

* **Без повтора**. Вы может выбрать политику повтора "Без повтора", то есть отключить логику повторных попыток. Пакет SDK один раз пытается подключиться и, если подключение установлено, один раз пытается отправить сообщение. Эта политика обычно используется в сценариях, чувствительных к пропускной способности или затратам. Если вы выберете этот вариант, то неотправленные сообщения сразу теряются без возможности восстановления.

### <a name="retry-policy-apis"></a>Интерфейсы API политики повтора

   | SDK | Метод SetRetryPolicy | Реализации политики | Рекомендации по реализации |
   |-----|----------------------|--|--|
   |  C/iOS  | [IOTHUB_CLIENT_RESULT IoTHubClient_SetRetryPolicy](https://github.com/Azure/azure-iot-sdk-c/blob/2018-05-04/iothub_client/inc/iothub_client.h#L188)        | **По умолчанию**: [IOTHUB_CLIENT_RETRY_EXPONENTIAL_BACKOFF](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/connection_and_messaging_reliability.md#connection-retry-policies).<BR>**Пользовательская политика:** используйте доступную политику [retryPolicy](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/connection_and_messaging_reliability.md#connection-retry-policies).<BR>**Без повтора:** [IOTHUB_CLIENT_RETRY_NONE](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/connection_and_messaging_reliability.md#connection-retry-policies).  | [Реализация для C/iOS](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/connection_and_messaging_reliability.md#)  |
   | Java| [SetRetryPolicy](/java/api/com.microsoft.azure.sdk.iot.device.deviceclientconfig.setretrypolicy)        | **По умолчанию**: [класс ExponentialBackoffWithJitter](https://github.com/Azure/azure-iot-sdk-java/blob/master/device/iot-device-client/src/main/java/com/microsoft/azure/sdk/iot/device/transport/NoRetry.java).<BR>**Пользовательская**: реализуйте [интерфейс RetryPolicy](https://github.com/Azure/azure-iot-sdk-java/blob/master/device/iot-device-client/src/main/java/com/microsoft/azure/sdk/iot/device/transport/RetryPolicy.java).<BR>**Без повтора:** [класс NoRetry](https://github.com/Azure/azure-iot-sdk-java/blob/master/device/iot-device-client/src/main/java/com/microsoft/azure/sdk/iot/device/transport/NoRetry.java).  | [Реализация для Java](https://github.com/Azure/azure-iot-sdk-java/blob/master/device/iot-device-client/devdoc/requirement_docs/com/microsoft/azure/iothub/retryPolicy.md) |
   | .NET| [DeviceClient.SetRetryPolicy](/dotnet/api/microsoft.azure.devices.client.deviceclient.setretrypolicy) | **По умолчанию**: [класс ExponentialBackoff](/dotnet/api/microsoft.azure.devices.client.exponentialbackoff).<BR>**Пользовательская**: реализуйте [интерфейс IRetryPolicy](/dotnet/api/microsoft.azure.devices.client.iretrypolicy).<BR>**Без повтора:** [класс NoRetry](/dotnet/api/microsoft.azure.devices.client.noretry). | [Реализация для C#](https://github.com/Azure/azure-iot-sdk-csharp) |
   | Узел| [setRetryPolicy](/javascript/api/azure-iot-device/client) | **По умолчанию**: [класс ExponentialBackoffWithJitter](https://github.com/Azure/azure-iot-sdk-java/blob/master/device/iot-device-client/src/main/java/com/microsoft/azure/sdk/iot/device/transport/NoRetry.java). | [Реализация для Node](https://github.com/Azure/azure-iot-sdk-node/wiki/Connectivity-and-Retries#types-of-errors-and-how-to-detect-them) |
   | Python| Сейчас не поддерживается | Сейчас не поддерживается | Сейчас не поддерживается |

Следующие примеры кода иллюстрируют этот поток.

#### <a name="net-implementation-guidance"></a>Рекомендации по реализации для .NET

Следующий пример кода демонстрирует, как правильно определить и настроить политику повтора по умолчанию.

   ```csharp
   // define/set default retry policy
   IRetryPolicy retryPolicy = new ExponentialBackoff(int.MaxValue, TimeSpan.FromMilliseconds(100), TimeSpan.FromSeconds(10), TimeSpan.FromMilliseconds(100));
   SetRetryPolicy(retryPolicy);
   ```

Чтобы избежать высокой загрузки ЦП, повторные попытки регулируются, если в коде происходит мгновенный сбой. Например, если нет сетевого подключения или маршрута к назначению. Минимальное время до следующей повторной попытки составляет 1 секунду.

Если служба возвращает ошибку регулирования, значит, политика повтора отличается и ее невозможно изменить с помощью общедоступного API.

   ```csharp
   // throttled retry policy
   IRetryPolicy retryPolicy = new ExponentialBackoff(RetryCount, TimeSpan.FromSeconds(10), 
     TimeSpan.FromSeconds(60), TimeSpan.FromSeconds(5)); SetRetryPolicy(retryPolicy);
   ```

Механизм повтора останавливается по истечении интервала `DefaultOperationTimeoutInMilliseconds`, который сейчас составляет 4 минуты.

#### <a name="other-languages-implementation-guidance"></a>Рекомендации по реализации для других языков

Примеры кода на других языках вы можете найти в следующих документах по реализации. Репозиторий содержит примеры использования API для политик повтора.

* [Пакет SDK для C/iOS](https://github.com/azure/azure-iot-sdk-c)

* [Пакет SDK для .NET](https://github.com/Azure/azure-iot-sdk-csharp/blob/master/iothub/device/devdoc/retrypolicy.md)

* [пакет SDK для Java](https://github.com/Azure/azure-iot-sdk-java/blob/master/device/iot-device-client/devdoc/requirement_docs/com/microsoft/azure/iothub/retryPolicy.md)

* [Пакет SDK для Node](https://github.com/Azure/azure-iot-sdk-node/wiki/Connectivity-and-Retries#types-of-errors-and-how-to-detect-them)

* [Пакет SDK для Python](https://github.com/Azure/azure-iot-sdk-python) (еще не обеспечена надежность)

## <a name="next-steps"></a>Дальнейшие действия

* [Использование пакетов SDK для устройств и служб](./iot-hub-devguide-sdks.md)

* [Использование пакета SDK для устройств Azure IoT для C](./iot-hub-device-sdk-c-intro.md)

* [Разработка приложений для устройств с ограниченным ресурсами](./iot-hub-devguide-develop-for-constrained-devices.md)

* [Разработка приложений для мобильных устройств](./iot-hub-how-to-develop-for-mobile-devices.md)

* [Detect and troubleshoot disconnects with Azure IoT Hub](iot-hub-troubleshoot-connectivity.md) (Обнаружение и устранение неполадок отключения Центра Интернета вещей Azure)
