---
title: Общие сведения о параметрах пакета SDK для устройств Azure IoT
description: Узнайте, какой пакет SDK для устройств Azure IoT следует использовать в зависимости от роли и задач разработки.
author: elhorton
ms.author: elhorton
ms.service: iot-develop
ms.topic: overview
ms.date: 02/11/2021
ms.openlocfilehash: c9624e9a23d005185429c82199324ac570cbd63e
ms.sourcegitcommit: d135e9a267fe26fbb5be98d2b5fd4327d355fe97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102607736"
---
# <a name="overview-of-azure-iot-device-sdks"></a>Общие сведения о пакетах SDK для устройств Azure IoT

Пакеты SDK для устройств Azure IoT представляют собой набор клиентских библиотек устройств, руководств для разработчиков, примеров и документации. Пакеты SDK для устройств позволяют программно подключать устройства к службам Интернета вещей Azure.

:::image type="content" border="false" source="media/about-iot-sdks/iot-sdk-diagram.png" alt-text="Экран &quot;Схема различных пакетов SDK для Интернета вещей Azure&quot;":::

Как показано на схеме, существует несколько пакетов SDK для устройств, поэтому вы сможете найти подходящий пакет для вашего устройства и языка программирования. Руководство по выбору подходящего пакета SDK для устройств доступно в разделе [Какой пакет SDK следует использовать?](#which-sdk-should-i-use). Кроме того, существуют пакеты SDK для служб Azure IoT, которые позволяют подключить облачное приложение к службам Интернета вещей Azure на серверной части. Эта статья посвящена пакетам SDK для устройств, однако дополнительные сведения о пакетах SDK служб Azure можно найти [здесь](#service-sdks).

## <a name="why-should-i-use-the-azure-iot-device-sdks"></a>Почему следует использовать пакеты SDK для устройств Azure IoT?

Чтобы подключить устройства к Интернету вещей Azure, можно создать пользовательский слой подключения или использовать пакеты SDK для устройств Azure IoT. Использование пакетов SDK для устройств Azure IoT предоставляет несколько преимуществ:

| Стоимость разработки &nbsp; &nbsp; &nbsp; &nbsp; | Пользовательский слой подключения | Пакеты SDK для устройств Azure IoT |
| :-- | :------------------------------------------------ | :---------------------------------------- |
| Поддержка | Необходимо поддерживать и документировать все создаваемые ресурсы | Доступ к службе поддержки Майкрософт (GitHub, Microsoft Q&A, Документация Майкрософт, группы поддержки клиентов) |
| Новые возможности | Нужно добавлять новые функции Azure в пользовательское ПО промежуточного слоя | Может сразу использовать преимущества новых функций, которые корпорация Майкрософт постоянно добавляет в пакеты SDK для Интернета вещей |
| Investment | Нужно тратить сотни часов на разработку встраиваемых решений для проектирования, сборки, тестирования и обслуживания пользовательской версии | Может использовать преимущества бесплатных средств с открытым кодом. Единственные затраты, связанные с пакетами SDK, — это кривая обучения. |

## <a name="which-sdk-should-i-use"></a>Какой пакет SDK следует использовать?

Пакеты SDK для устройств Azure IoT доступны на популярных языках программирования, в том числе C, C#, Java, Node.js и Python. При выборе пакета SDK необходимо учитывать два основных аспекта: возможности устройства и знание языка программирования.

### <a name="device-capabilities"></a>Возможности устройства

При выборе пакета SDK необходимо учитывать ограничения устройств, которые вы используете. Устройство с ограниченными ресурсами — это устройство, которое имеет один микроконтроллер (MCU) и ограниченный объем памяти. Если вы используете устройство с ограниченными ресурсами, рекомендуется использовать [встроенный пакет SDK для C](#embedded-c-sdk). Этот пакет SDK предназначен для предоставления минимального набора возможностей для подключения к Интернету вещей Azure. Вы также можете выбрать компоненты (клиент MQTT, TLS и библиотеки сокетов), наиболее оптимизированные для встроенного устройства. Если устройство с ограниченными ресурсами также работает под управлением ОСРВ Azure, для подключения к Интернету вещей Azure можно использовать ПО промежуточного слоя ОСРВ Azure. ПО промежуточного слоя ОСРВ Azure внедряет встроенный пакет SDK для C, реализуя дополнительную функцию, чтобы упростить подключение устройства ОСРВ Azure к облаку.

Устройство с неограниченными ресурсами имеет более надежный процессор, который может работать под управлением операционной системы для поддержки языковой среды выполнения, такой как .NET или Python. При использовании устройства с неограниченными ресурсами главное — иметь опыт работы с языком.

### <a name="your-teams-familiarity-with-the-programming-language"></a>Знакомство вашей команды с языком программирования

Пакеты SDK для устройств Azure IoT реализуются на нескольких языках, поэтому вы можете выбрать язык по своему усмотрению. Пакеты SDK для устройств также интегрируются с другими привычными средствами, зависящими от языка. Возможность работать с привычным языком программирования и средствами разработки позволяет команде оптимизировать цикл разработки для исследования, создания прототипов, разработки продуктов и текущего обслуживания.

По возможности выбирайте пакет SDK, знакомый команде разработчиков. Все пакеты SDK для Интернета вещей Azure имеют открытый код и содержат несколько примеров, которые ваша команда может оценить и протестировать, прежде чем перейти к конкретному пакету SDK.

## <a name="how-can-i-get-started"></a>Как начать работу?

Начнем с изучения репозиториев GitHub, посвященных пакетам SDK для устройств Azure. Вы также можете ознакомиться с [кратким руководством](quickstart-send-telemetry-python.md), в котором показано, как быстро использовать пакет SDK для отправки данных телеметрии в Интернет вещей Azure.

Варианты начала работы зависят от типа устройства:
- для устройств с ограниченными ресурсами используйте [встроенный пакет SDK для C](#embedded-c-sdk); 
- для устройств, работающих в ОСРВ Azure, можно использовать [ПО промежуточного слоя ОСРВ Azure](#azure-rtos-middleware); 
- для устройств с неограниченными ресурсами можно [выбрать пакет SDK](#unconstrained-device-sdks) на любом языке. 

### <a name="constrained-device-sdks"></a>Пакеты SDK для устройства с ограниченными ресурсами
Эти пакеты SDK предназначены для работы на устройствах с ограниченными вычислительными ресурсами или ресурсами памяти. Дополнительные сведения о распространенных типах устройств см. в статье [Общие сведения о типах устройств Интернета вещей Azure](concepts-iot-device-types.md).

#### <a name="embedded-c-sdk"></a>Встроенный пакет SDK для C
* [Репозиторий GitHub](https://github.com/Azure/azure-sdk-for-c/tree/master/sdk/docs/iot)
* [Примеры](https://github.com/Azure/azure-sdk-for-c/blob/master/sdk/samples/iot/README.md)
* [Справочная документация](https://azure.github.io/azure-sdk-for-c/)
* [Создание встроенного пакета SDK для C](https://github.com/Azure/azure-sdk-for-c/tree/master/sdk/docs/iot#build)
* [Диаграмма размеров для устройств с ограниченными ресурсами](https://github.com/Azure/azure-sdk-for-c/tree/master/sdk/docs/iot#size-chart)

#### <a name="azure-rtos-middleware"></a>ПО промежуточного слоя ОСРВ Azure

* [Репозиторий GitHub](https://github.com/azure-rtos/netxduo/tree/master/addons/azure_iot)
* [Руководства по началу работы](https://github.com/azure-rtos/getting-started) и [другие примеры](https://github.com/azure-rtos/samples)
* [Справочная документация](/azure/rtos/threadx/)

### <a name="unconstrained-device-sdks"></a>Пакеты SDK для устройств с неограниченными ресурсами
Эти пакеты SDK могут работать на любом устройстве, поддерживающем языковую среду выполнения высшего порядка. Сюда входят такие устройства, как ПК, Raspberry Pi и смартфоны. Они различаются в первую очередь по языку, поэтому вы можете выбрать библиотеку, которая лучше всего подходит вашей команде и сценарию.

#### <a name="c-device-sdk"></a>Пакет SDK для устройств, использующих C
* [Репозиторий GitHub](https://github.com/Azure/azure-iot-sdk-c)
* [Примеры](https://github.com/Azure/azure-iot-sdk-c/tree/master/iothub_client/samples)
* [Пакеты](https://github.com/Azure/azure-iot-sdk-c/blob/master/readme.md#packages-and-libraries)
* [Справочная документация](/azure/iot-hub/iot-c-sdk-ref/)
* [Справочная документация по модулю Edge](/azure/iot-hub/iot-c-sdk-ref/iothub-module-client-h)
* [Компиляция пакета SDK для устройств C](https://github.com/Azure/azure-iot-sdk-c/blob/master/iothub_client/readme.md#compiling-the-c-device-sdk).
* [Перенос пакета SDK C для других платформ](https://github.com/Azure/azure-c-shared-utility/blob/master/devdoc/porting_guide.md)
* [Документация для разработчиков](https://github.com/Azure/azure-iot-sdk-c/tree/master/doc) содержит сведения о кроссплатформенной компиляции, а также начале работы на различных платформах
* [Сведения о потреблении ресурсов пакета SDK для Центра Интернета вещей Azure для C](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/c_sdk_resource_information.md)

#### <a name="c-device-sdk"></a>Пакет SDK для устройств, использующих C#

* [Репозиторий GitHub](https://github.com/Azure/azure-iot-sdk-csharp)
* [Примеры](https://github.com/Azure/azure-iot-sdk-csharp#samples)
* [Пакет](https://www.nuget.org/packages/Microsoft.Azure.Devices.Client/)
* [Справочная документация](/dotnet/api/microsoft.azure.devices)
* [Справочная документация по модулю Edge](/dotnet/api/microsoft.azure.devices.client.moduleclient)

#### <a name="java-device-sdk"></a>Пакет SDK для устройств, использующих Java

* [Репозиторий GitHub](https://github.com/Azure/azure-iot-sdk-java)
* [Примеры](https://github.com/Azure/azure-iot-sdk-java/tree/master/device/iot-device-samples)
* [Пакет](https://github.com/Azure/azure-iot-sdk-java/blob/master/doc/java-devbox-setup.md#for-the-device-sdk)
* [Справочная документация](/java/api/com.microsoft.azure.sdk.iot.device)
* [Справочная документация по модулю Edge](/java/api/com.microsoft.azure.sdk.iot.device.moduleclient)

#### <a name="nodejs-device-sdk"></a>Пакет SDK для устройств, использующих Node.js

* [Репозиторий GitHub](https://github.com/Azure/azure-iot-sdk-node)
* [Примеры](https://github.com/Azure/azure-iot-sdk-node/tree/master/device/samples)
* [Пакет](https://www.npmjs.com/package/azure-iot-device)
* [Справочная документация](/javascript/api/azure-iot-device/)
* [Справочная документация по модулю Edge](/javascript/api/azure-iot-device/moduleclient)

#### <a name="python-device-sdk"></a>Пакет SDK для устройств, использующих Python

* [Репозиторий GitHub](https://github.com/Azure/azure-iot-sdk-python)
* [Примеры](https://github.com/Azure/azure-iot-sdk-python/tree/master/azure-iot-device/samples)
* [Пакет](https://pypi.org/project/azure-iot-device/)
* [Справочная документация](/python/api/azure-iot-device)
* [Справочная документация по модулю Edge](/python/api/azure-iot-device/azure.iot.device.iothubmoduleclient)

### <a name="service-sdks"></a>Пакеты SDK службы
Интернет вещей Azure также предлагает пакеты SDK для служб, позволяющие создавать приложения на стороне решения для управления устройствами, получения полезных сведений, визуализации данных и многого другого. Эти пакеты SDK специфичны для каждой отдельной службы Интернета вещей Azure и доступны на языках C#, Java, JavaScript и Python, чтобы упростить процесс разработки. 

#### <a name="iot-hub"></a>Центр Интернета вещей

Пакеты SDK для службы от Центра Интернета вещей позволяют создавать приложения, которые легко взаимодействуют с Центром Интернета вещей и упрощают управление устройствами и безопасностью. Эти пакеты SDK можно использовать для отправки сообщений из облака на устройство, вызова прямых методов на устройствах, обновления свойств устройств и многого другого.

[**Дополнительные сведения о Центре Интернета вещей**](https://azure.microsoft.com/services/iot-hub/) | [**Приступить к управлению устройством**](../iot-hub/quickstart-control-device-python.md)

**Пакет SDK для службы от Центра Интернета вещей для C#** : [Репозиторий GitHub](https://github.com/Azure/azure-iot-sdk-csharp/tree/master/iothub/service) | [Пакет](https://www.nuget.org/packages/Microsoft.Azure.Devices/) | [Примеры](https://github.com/Azure/azure-iot-sdk-csharp/tree/master/iothub/service/samples) | [Справочная документация](/dotnet/api/microsoft.azure.devices)

**Пакет SDK для службы от Центра Интернета вещей для Java**: [Репозиторий GitHub](https://github.com/Azure/azure-iot-sdk-java/tree/master/service) | [Пакет](https://github.com/Azure/azure-iot-sdk-java/blob/master/doc/java-devbox-setup.md#for-the-service-sdk) | [Примеры](https://github.com/Azure/azure-iot-sdk-java/tree/master/service/iot-service-samples) | [Справочная документация](/java/api/com.microsoft.azure.sdk.iot.service)

**Пакет SDK для службы от Центра Интернета вещей для JavaScript**: [Репозиторий GitHub](https://github.com/Azure/azure-iot-sdk-node/tree/master/service) | [Пакет](https://www.npmjs.com/package/azure-iothub) | [Примеры](https://github.com/Azure/azure-iot-sdk-node/tree/master/service/samples) | [Справочная документация](/javascript/api/azure-iothub/)

**Пакет SDK для службы от Центра Интернета вещей для Python**: [Репозиторий GitHub](https://github.com/Azure/azure-iot-sdk-python/tree/master/azure-iot-hub) | [Пакет](https://pypi.python.org/pypi/azure-iot-hub/) | [Примеры](https://github.com/Azure/azure-iot-sdk-python/tree/master/azure-iot-hub/samples) | [Справочная документация](/python/api/azure-iot-hub)

#### <a name="azure-digital-twins"></a>Azure Digital Twins

Azure Digital Twins — это предложение платформы как услуги (PaaS), которое позволяет создавать графы знаний на основе цифровых моделей целых сред. Такими средами могут быть здания, фабрики, фермы, энергосистемы, железные дороги, стадионы и многое другое — даже целые города. Эти цифровые модели можно использовать для получения аналитических сведений, позволяющих улучшать продукты, оптимизировать операции, сокращать расходы и повышать эффективность работы с клиентами. Интернет вещей Azure предлагает пакеты SDK для служб, которые упрощают создание приложений, использующих возможности Azure Digital Twins.

[**Дополнительные сведения об Azure Digital Twins**](https://azure.microsoft.com/services/digital-twins/) | [**Напишите код с помощью приложения ADT**](../digital-twins/tutorial-code.md)

**Пакет SDK для службы ADT для C#** : [Репозиторий GitHub](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/digitaltwins/Azure.DigitalTwins.Core) | [Пакет](https://www.nuget.org/packages/Azure.DigitalTwins.Core) | [Примеры](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/digitaltwins/Azure.DigitalTwins.Core/samples) | [Справочная документация](/dotnet/api/overview/azure/digitaltwins/client)

**Пакет SDK для службы ADT для Java**: [Репозиторий GitHub](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/digitaltwins/azure-digitaltwins-core) | [Пакет](https://search.maven.org/artifact/com.azure/azure-digitaltwins-core/1.0.0/jar) | [Примеры](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/digitaltwins/azure-digitaltwins-core/src/samples) | [ Справочная документация](/java/api/overview/azure/digitaltwins/client)

**Пакет SDK для службы ADT для Node.js**: [Репозиторий GitHub](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/digitaltwins/digital-twins-core) | [Пакет](https://www.npmjs.com/package/@azure/digital-twins-core) | [Примеры](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/digitaltwins/digital-twins-core/samples) | [Справочная документация](/javascript/api/@azure/digital-twins-core/)

**Пакет SDK для службы ADT для Python**: [Репозиторий GitHub](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/digitaltwins/azure-digitaltwins-core) | [Пакет](https://pypi.org/project/azure-digitaltwins-core/) | [Примеры](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/digitaltwins/azure-digitaltwins-core/samples) | [Справочная документация](/python/api/azure-digitaltwins-core/azure.digitaltwins.core)

#### <a name="device-provisioning-service"></a>Device Provisioning Service (Служба подготовки устройств)

Служба подготовки устройств (DPS) Центра Интернета вещей является вспомогательной службой для Центра Интернета вещей, что позволяет быстро и полностью в автоматическом режиме подготовить необходимый Центр Интернета вещей без вмешательства пользователя. DPS позволяет безопасно и масштабируемым способом предоставлять доступ к миллионам устройств. Пакеты SDK для службы DPS позволяют создавать приложения, которые могут безопасно управлять вашими устройствами, создавая группы регистрации и выполняя массовые операции.

[**Дополнительные сведения о Службе подготовки устройств**](../iot-dps/index.yml) | [**Попробуйте создать групповую регистрацию для устройств X.509**](../iot-dps/quick-enroll-device-x509-csharp.md)

**Пакет SDK для Службы подготовки устройств для C#** : [Репозиторий GitHub](https://github.com/Azure/azure-iot-sdk-csharp/tree/master/provisioning/service) | [Пакет](https://www.nuget.org/packages/Microsoft.Azure.Devices.Provisioning.Service/) | [Примеры](https://github.com/Azure/azure-iot-sdk-csharp/tree/master/provisioning/service/samples) | [Справочная документация](/dotnet/api/microsoft.azure.devices.provisioning.service)

**Пакет SDK для Службы подготовки устройств для Java**: [Репозиторий GitHub](https://github.com/Azure/azure-iot-sdk-java/tree/master/provisioning/provisioning-service-client/src) | [Пакет](https://mvnrepository.com/artifact/com.microsoft.azure.sdk.iot.provisioning/provisioning-service-client) | [Примеры](https://github.com/Azure/azure-iot-sdk-java/tree/master/provisioning/provisioning-samples#provisioning-service-client) | [Справочная документация](/java/api/com.microsoft.azure.sdk.iot.provisioning.service)

**Пакет SDK для Службы подготовки устройств для Node.js**: [Репозиторий GitHub](https://github.com/Azure/azure-iot-sdk-node/tree/master/provisioning/service) | [Пакет](https://www.npmjs.com/package/azure-iot-provisioning-service) | [Примеры](https://github.com/Azure/azure-iot-sdk-node/tree/master/provisioning/service/samples) | [Справочная документация](/javascript/api/azure-iot-provisioning-service)

## <a name="next-steps"></a>Next Steps

* [Краткое руководство. Подключение устройства к IoT Central (Python)](quickstart-send-telemetry-python.md)
* [Краткое руководство. Подключение устройства к Центру Интернета вещей (Python)](quickstart-send-telemetry-cli-python.md)
* [Начало разработки встраиваемых устройств](quickstart-device-development.md)
* Узнайте больше о [преимуществах разработки с использованием пакетов SDK для Интернета вещей Azure](https://azure.microsoft.com/blog/benefits-of-using-the-azure-iot-sdks-in-your-azure-iot-solution/).