---
title: Поддержка платформы пакетов SDK для устройств Azure IoT | Документация Майкрософт
description: Пакеты SDK для устройств с открытым исходным кодом доступны на сайте GitHub на языках C, .NET (C#), Java, Node.js и Python для подключения устройств к центру Интернета вещей Azure и службе подготовки устройств (DPS).
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 10/08/2019
ms.author: robinsh
ms.openlocfilehash: b8f8403617a5fc8b442e8a3f74ab9226fe1479fe
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97588766"
---
# <a name="azure-iot-device-sdks-platform-support"></a>Поддержка платформ SDK для устройств Azure IoT

Корпорация Майкрософт стремится постоянно расширять мир устройств с поддержкой центра Интернета вещей Azure. Корпорация Майкрософт публикует пакеты SDK для устройств с открытым исходным кодом на сайте GitHub, чтобы помочь в подключении устройств к центру Интернета вещей Azure и службе подготовки устройств. Пакеты SDK для устройств доступны для C, .NET (C#), Java, Node.js и Python. Корпорация Майкрософт тестирует каждый пакет SDK, чтобы убедиться, что он выполняется в поддерживаемых конфигурациях, описанных в разделе [пакеты SDK и поддержка платформ устройств](#microsoft-sdks-and-device-platform-support) .

Помимо пакетов SDK для устройств Корпорация Майкрософт предоставляет несколько других способов, позволяющих клиентам и разработчикам подключать свои устройства к Azure IoT.

* Корпорация Майкрософт совместно взаимодействуют с несколькими компаниями-партнерами, помогая публиковать пакеты средств разработки на основе пакета SDK для Azure IoT C для своих аппаратных платформ.

* Корпорация Майкрософт сотрудничает с надежными партнерами Майкрософт, чтобы предоставить постоянно расширяемый набор устройств, проверенных и сертифицированных для Интернета вещей Azure. Текущий список этих устройств см. в [каталоге устройств "сертифицировано для Интернета вещей Azure](https://catalog.azureiotsolutions.com/)".

* Корпорация Майкрософт предоставляет уровень абстракции платформы (PAL) в пакете SDK для устройств центра Интернета вещей Azure, который помогает разработчикам легко перенести пакет SDK на платформу. Дополнительные сведения см. в [руководстве по переносу пакетов SDK для C](https://github.com/Azure/azure-c-shared-utility/blob/master/devdoc/porting_guide.md).

В этом разделе содержатся сведения о пакетах SDK для Microsoft и поддерживаемых ими конфигурациях платформы, а также о каждом из перечисленных выше вариантов.

## <a name="microsoft-sdks-and-device-platform-support"></a>Пакеты SDK и поддержка платформ устройств Microsoft

Корпорация Майкрософт публикует пакеты SDK с открытым кодом на GitHub для следующих языков: C, .NET (C#), Node.js, Java и Python. В этом разделе перечислены пакеты SDK и их зависимости. Пакеты SDK поддерживаются на всех платформах устройств, удовлетворяющих этим зависимостям.

Для каждого из перечисленных пакетов SDK Корпорация Майкрософт:

* Непрерывно создает и выполняет комплексные тесты для основной ветви соответствующего пакета SDK в GitHub на нескольких популярных платформах.  Чтобы обеспечить покрытие тестов в разных версиях компилятора, мы обычно тестируем последнюю версию LTS и самую популярную версию.

* Содержит руководство по установке или пакеты установки, если это применимо.

* Полностью поддерживает пакеты SDK на GitHub с открытым кодом, путь для вклада клиентов и сотрудничество группы разработки продукта с проблемами GitHub.

### <a name="c-sdk"></a>Пакет C SDK

[Пакет SDK для устройства C центра Интернета вещей Azure](https://github.com/Azure/azure-iot-sdk-c) тестируется в и поддерживает следующие конфигурации.

| OS                  | Библиотека TLS                  | Дополнительные требования                                                                     |
|---------------------|------------------------------|---------------------------------------------------------------------------------------------|
| Linux               | OpenSSL, Волфссл или Беарссл | Протоколы Berkeley</br></br>Интерфейс переносимой операционной системы (POSIX)                       |
| iOS 12,2            | OpenSSL.                      | XCode, имитируемый в OSX 10.13.4                                                               |
| Семейство Windows 10   | SChannel                     |                                                                                             |
| Mbed OS 5,4         | Mbed TLS 2                   | [Комплект разработчика IoT MXChip](https://microsoft.github.io/azure-iot-developer-kit/)                  |
| Azure Sphere ОС     | волфссл                      | [Azure Sphere MT3620](https://azure.microsoft.com/services/azure-sphere/get-started/) |
| Arduino             | беарссл                      | [ESP32 или ESP8266](https://github.com/Azure/azure-iot-arduino#simple-sample-instructions) 

### <a name="python-sdk"></a>Пакет SDK для Python

[Пакет SDK для устройств Python центра Интернета вещей Azure](https://github.com/Azure/azure-iot-sdk-python) тестируется в и поддерживает следующие конфигурации.

| OS                  | Компилятор                          |
|---------------------|-----------------------------------|
| Linux               | Python 2,7. *, 3,5 или более поздней версии |
| macOS High Sierra;   | Python 2,7. *, 3,5 или более поздней версии |
| Семейство Windows 10   | Python 2,7. *, 3,5 или более поздней версии |

Только Python версии 3.5.3 или более поздней поддерживают асинхронные API-интерфейсы, рекомендуется использовать версию 3,7 или более позднюю.

### <a name="net-sdk"></a>Пакет SDK для .NET

[Пакет SDK для устройств .NET для центра Интернета вещей (C#)](https://github.com/Azure/azure-iot-sdk-csharp) проверяется с поддержкой следующих конфигураций.

| OS                                   | Стандартный                                                   |
|--------------------------------------|------------------------------------------------------------|
| Linux                                | .NET Core 2.1                                              |
| SKU для настольных компьютеров и серверов Windows 10   | .NET Core 2,1, платформа .NET Framework 4.5.1 или платформа .NET Framework 4,7 |

Пакет SDK для .NET также можно использовать с Windows IoT Core с [агентом устройства Azure](https://github.com/ms-iot/azure-client-tools/blob/master/docs/device-agent/device-agent.md) или [с пользовательским служба NTService, который может использовать RPC для взаимодействия с приложениями UWP](/samples/microsoft/windows-iotcore-samples/ntservice-rpc/).

### <a name="nodejs-sdk"></a>Пакет SDK для Node.js

[Центр Интернета вещей Azure Node.js пакет SDK для устройств](https://github.com/Azure/azure-iot-sdk-node) и поддерживает следующие конфигурации.

| OS                  | Версия узла    |
|---------------------|-----------------|
| Linux               | LTS и текущий |
| Семейство Windows 10   | LTS и текущий |

### <a name="java-sdk"></a>Пакет SDK для Java

[Пакет SDK для устройства Java центра Интернета вещей Azure](https://github.com/Azure/azure-iot-sdk-java) тестируется в и поддерживает следующие конфигурации.

| OS                     | Версия Java |
|------------------------|--------------|
| API 28 для Android         | Java 8       |
| Linux x64             | Java 8       |
| Семейство Windows 10 x64  | Java 8       |

## <a name="partner-supported-development-kits"></a>Поддерживаемые партнерами пакеты средств разработки

Корпорация Майкрософт работает с различными партнерами, предоставляющими наборы средств разработки для нескольких архитектур микропроцессора. Эти партнеры перестроили пакет SDK для Azure IoT C на свою платформу. Партнеры создают и поддерживают уровень абстракции платформы (PAL) пакета SDK. Корпорация Майкрософт работает с этими партнерами для предоставления расширенной поддержки.

| Партнер             | Устройства                            | Ссылка                     | Поддержка |
|---------------------|------------------------------------|--------------------------|---------|
| Espressif           | ESP32 <br/> ESP8266                              | [ESP-azure](https://github.com/espressif/esp-azure)                | [GitHub](https://github.com/espressif/esp-azure)  
| Qualcomm            | Модем Qualcomm MDM9206 LTE IoT     | [Qualcomm LTE для пакета SDK IoT](https://developer.qualcomm.com/software/lte-iot-sdk) | [Форум](https://developer.qualcomm.com/forums/software/lte-iot-sdk)   |
| ST Microelectronics | Серии STM32L4 <br/> Серии STM32F4 <br/>  Серии STM32F7 <br/>  Пакет обнаружения STM32L4 для узла Интернета вещей    | [X-CUBE-AZURE](https://www.st.com/en/embedded-software/x-cube-azure.html) <br/>  <br/> [P-NUCLEO AZURE](https://www.st.com/content/st_com/en/products/evaluation-tools/solution-evaluation-tools/communication-and-connectivity-solution-eval-boards/p-nucleo-azure1.html) <br/> [FP-CLD AZURE](https://www.st.com/content/st_com/en/products/embedded-software/mcus-embedded-software/stm32-embedded-software/stm32-ode-function-pack-sw/fp-cld-azure1.html)            | [Поддержка](https://www.st.com/content/st_com/en/support/support-home.html)
| Texas Instruments   | Панель запуска CC3220SF </br> Панель запуска CC3220S </br> Панель запуска CC3235SF </br> Панель запуска CC3235S </br> Панель запуска MSP432E4 | [Run a simple C sample on a SimpleLink device](https://github.com/TexasInstruments/azure-iot-pal-simplelink) (Запуск простого примера C на устройстве SimpleLink) | [Форум TI E2E](https://e2e.ti.com) <br/> [Форум TI E2E для CC3220](https://e2e.ti.com/support/wireless_connectivity/simplelink_wifi_cc31xx_cc32xx/) <br/> [Форум TI E2E для MSP432E4](https://e2e.ti.com/support/microcontrollers/msp430/) |

## <a name="porting-the-microsoft-azure-iot-c-sdk"></a>Перенос пакета SDK для Microsoft Azure IoT C

Если ваша платформа устройства не охватывается одним из предыдущих разделов, можно рассмотреть возможность переноса пакета SDK для Azure IoT C. Перенос пакета SDK C в основном включает реализацию уровня абстракции платформы (PAL) пакета SDK. Список доступа к публикации определяет примитивы, которые обеспечивают привязывание между устройством и функциями более высокого уровня в пакете SDK. Дополнительные сведения см. в разделе [руководство по переносу](https://github.com/Azure/azure-c-shared-utility/blob/master/devdoc/porting_guide.md).

## <a name="microsoft-partners-and-certified-azure-iot-devices"></a>Партнеры Майкрософт и сертифицированные устройства Интернета вещей Azure

Корпорация Майкрософт работает с несколькими участниками, чтобы постоянно расширять мир Интернета вещей Azure с проверенными и сертифицированными устройствами Azure IoT.

* Чтобы просмотреть сертифицированные устройства Azure IoT, см. статью [Microsoft Azure Certified for IOT каталога устройств](https://catalog.azureiotsolutions.com/).

* Дополнительные сведения о сертификации Azure для экосистемы IoT см. в статье [Присоединяйтесь к экосистеме "сертифицировано для Интернета вещей"](https://catalog.azureiotsolutions.com/register).

## <a name="connecting-to-iot-hub-without-an-sdk"></a>Подключение к центру Интернета вещей без пакета SDK

Если вы не можете использовать один из пакетов SDK для устройств центра Интернета вещей, вы можете подключиться непосредственно к центру Интернета вещей с помощью поддерживаемого протокола связи. Дополнительные сведения см. в статье [Разработка без пакета SDK для Azure IOT](iot-hub-devguide-no-sdk.md).

## <a name="support-and-other-resources"></a>Поддержка и другие ресурсы

При возникновении проблем при использовании пакетов SDK для устройств Azure IoT существует несколько способов поиска и поддержки, приведенных ниже. Дополнительные сведения обо всех вариантах поддержки см. в статье [Поддержка Azure IOT и параметры справки](../iot-fundamentals/iot-support-help.md). 

**Сообщения об ошибках** . ошибки в пакетах SDK для устройств можно сообщить на странице проблем в соответствующем проекте GitHub. Внесенные исправления очень быстро попадают из проекта в обновление продукта.

* [Проблемы с пакетом SDK для центра Интернета вещей Azure](https://github.com/Azure/azure-iot-sdk-c/issues)

* [Проблемы с пакетом SDK .NET для центра Интернета вещей Azure (C#)](https://github.com/Azure/azure-iot-sdk-csharp/issues)

* [Проблемы с пакетом SDK для Java в центре Интернета вещей Azure](https://github.com/Azure/azure-iot-sdk-java/issues)

* [Проблемы с пакетом SDK Node.js центра Интернета вещей Azure](https://github.com/Azure/azure-iot-sdk-node/issues)

* [Проблемы с пакетом SDK для Python центра Интернета вещей Azure](https://github.com/Azure/azure-iot-sdk-python/issues)

**Технические вопросы** : вы можете задавать технические вопросы как в [Microsoft Q&A](/answers/topics/azure-iot-sdk.html) , так [Stack overflow](https://stackoverflow.com/questions/tagged/azure-iot-sdk) с помощью тега *Azure-IOT-SDK*.

**Группа поддержки пользователей Майкрософт** — пользователи, у которых есть [план поддержки](https://azure.microsoft.com/support/plans/) , могут обратиться в службу поддержки пользователей Майкрософт, создав новый запрос в службу поддержки непосредственно из [портал Azure](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview).

**Запросы функций** . запросы к функциям Интернета вещей Azure отправляются с помощью [веб-страницы пользователя](https://feedback.azure.com/forums/321918-azure-iot)продукта.

## <a name="next-steps"></a>Дальнейшие действия

* [Пакеты SDK для устройств и служб](iot-hub-devguide-sdks.md)
* [Руководство по переносу](https://github.com/Azure/azure-c-shared-utility/blob/master/devdoc/porting_guide.md)