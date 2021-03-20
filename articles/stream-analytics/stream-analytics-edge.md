---
title: Azure Stream Analytics в IoT Edge
description: Создание заданий Edge в Azure Stream Analytics и развертывание их на устройствах, где работает Azure IoT Edge.
ms.service: stream-analytics
author: an-emma
ms.author: raan
ms.topic: conceptual
ms.date: 12/18/2020
ms.custom: contperf-fy21q2
ms.openlocfilehash: c2a062b75caa84a0e3c342ca1ce45ccce12f2bb7
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98012620"
---
# <a name="azure-stream-analytics-on-iot-edge"></a>Azure Stream Analytics в IoT Edge
 
Azure Stream Analytics на IoT Edge позволяет разработчикам развертывать аналитическую аналитику практически в реальном времени ближе к устройствам IoT, чтобы они могли разблокировать все значения данных, созданных на устройстве. Azure Stream Analytics предоставляет такие преимущества, как минимальная задержка, отказоустойчивость, эффективное использование пропускной способности и соответствие требованиям. Предприятия могут развертывать логику управления ближе к промышленным операциям и дополнять аналитику больших данных, выполняемую в облаке.

Azure Stream Analytics в IoT Edge работает на платформе [Azure IoT Edge](https://azure.microsoft.com/campaigns/iot-edge/). После создания задания в Stream Analytics можно развернуть его и управлять им с помощью центра Интернета вещей.

## <a name="common-scenarios"></a>Распространенные сценарии

В этом разделе описаны распространенные сценарии для Stream Analytics IoT Edge. На следующей схеме показан поток данных между устройствами IoT и облаком Azure.

:::image type="content" source="media/stream-analytics-edge/edge-high-level-diagram.png" alt-text="Схема высокого уровня IoT Edge":::

### <a name="low-latency-command-and-control"></a>Команда и управление с низкой задержкой

Системы безопасности производства должны реагировать на операционные данные с Ultra-Low задержками. С Stream Analytics на IoT Edge можно анализировать данные датчиков практически в реальном времени, а также выполнять команды при обнаружении аномалий для того, чтобы отключить компьютер или инициировать оповещения.

### <a name="limited-connectivity-to-the-cloud"></a>Ограниченное подключение к облаку

Критически важные системы, такие как оборудование для удаленного интеллектуального анализа данных, подключенные резервуары или оборудование для детализации данных внешней разработки, должны анализировать данные и реагировать на них, даже если подключение к облаку периодически прерывается. При использовании Stream Analytics логика потоковой передачи выполняется независимо от сетевого подключения, и вы можете выбрать, что вы отправляете в облако для дальнейшей обработки или хранения.

### <a name="limited-bandwidth"></a>Ограниченная пропускная способность

Объем данных, создаваемых с помощью реактивных двигателей или подключенных автомобилей, может быть настолько большим, что данные необходимо отфильтровывать или предварительно обрабатывать, прежде чем отправить их в облако. С помощью Stream Analytics можно фильтровать или объединять данные, которые необходимо отправить в облако.

### <a name="compliance"></a>Соответствие нормативным требованиям

Для обеспечения соответствия нормативным требованиям может потребоваться локально анонимизировать или статистически обработать некоторые данные перед их отправкой в облако.

## <a name="edge-jobs-in-azure-stream-analytics"></a>Задания Edge в службе Azure Stream Analytics

Stream Analytics пограничных заданий выполняются в контейнерах, развернутых на [Azure IOT Edge устройствах](../iot-edge/about-iot-edge.md). Задания пограничных заданий состоят из двух частей:

* Облачная часть, отвечающая за определение задания: пользователи определяют входные данные, выходные данные, запросы и другие параметры, такие как неупорядоченные события, в облаке.

* Модуль, выполняющийся на устройствах IoT Edge. Модуль содержит Stream Analytics подсистему и получает определение задания из облака. 

Stream Analytics использует центр Интернета вещей для развертывания заданий пограничных устройств на устройствах. Дополнительные сведения см. в разделе [IOT Edge Deployment](../iot-edge/module-deployment-monitoring.md).

:::image type="content" source="media/stream-analytics-edge/stream-analytics-edge-job.png" alt-text="Задание Edge Azure Stream Analytics":::

## <a name="edge-job-limitations"></a>Ограничения для задания пограничных заданий

Целью является обеспечение соответствия между заданиями IoT Edge и облачными заданиями. Большинство функций языка запросов SQL поддерживаются как для пограничных, так и для облака. Однако задания пограничных заданий не поддерживают следующие функции:
* Определяемые пользователем функции (UDF) в JavaScript. Определяемые пользователем функции доступны в [C# для заданий IoT Edge](./stream-analytics-edge-csharp-udf.md) (предварительная версия).
* Определяемые пользователем статистические функции (UDA).
* Функции машинного обучения Azure.
* Формат AVRO для входных и выходных данных. На данный момент поддерживаются только форматы CSV и JSON.
* Следующие операторы SQL:
    * PARTITION BY
    * GetMetadataPropertyValue
* Политика для событий, сведения о которых поступают с задержкой

### <a name="runtime-and-hardware-requirements"></a>Требования к оборудованию и среде выполнения
Для запуска Stream Analytics на IoT Edge необходимы устройства, которые могут выполнять [Azure IOT Edge](https://azure.microsoft.com/campaigns/iot-edge/). 

Stream Analytics и Azure IoT Edge используйте контейнеры **DOCKER** , чтобы предоставить переносимое решение, работающее на нескольких операционных системах (Windows, Linux).

Stream Analytics на IoT Edge доступны в виде образов Windows и Linux, работающих на архитектурах x86-64 или ARM (с расширенными RISC-компьютерами). 


## <a name="input-and-output"></a>Ввод и вывод

Задания Stream Analytics пограничных устройств могут получать входные и выходные данные из других модулей, работающих на IoT Edge устройствах. Чтобы подключиться к конкретным модулям и из них, можно задать конфигурацию маршрутизации во время развертывания. Дополнительные сведения см. в [документации по композиции модуля IoT Edge](../iot-edge/module-composition.md).

Для входных и выходных данных поддерживаются форматы CSV и JSON.

Для каждого входного и выходного потока, создаваемого в задании Stream Analytics, в развернутом модуле создается соответствующая конечная точка. Эти конечные точки можно использовать в маршрутах развертывания.

Поддерживаемые входные типы потока:
* Центр Edge
* Концентратор событий
* Центр Интернета вещей

Поддерживаются следующие типы потокового вывода:
* Центр Edge
* База данных SQL
* Концентратор событий
* Хранилище BLOB-объектов/ADLS 2-го поколения

Эталонные входные данные поддерживают тип эталонного файла. Другие выходные данные можно получить с использованием подчиненного облачного задания. Например, задание Stream Analytics, размещенное в Edge, отправляет выходные данные в концентратор Edge, который затем может отправлять их в Центр Интернета вещей. Вы можете использовать второе размещенное в облаке задание Azure Stream Analytics с входными данными из центра Интернета вещей и выходом для Power BI или другого типа выходных данных.

## <a name="license-and-third-party-notices"></a>Уведомления о лицензии и сторонних производителях
* [Лицензия Azure Stream Analytics на IoT Edge](https://go.microsoft.com/fwlink/?linkid=862827). 
* [Уведомление сторонних производителей для Azure Stream Analytics на IoT Edge](https://go.microsoft.com/fwlink/?linkid=862828).

## <a name="azure-stream-analytics-module-image-information"></a>Сведения об образе модуля Azure Stream Analytics 

Сведения о версии были обновлены в последний раз 2020-09-21:

- Образ: `mcr.microsoft.com/azure-stream-analytics/azureiotedge:1.0.9-linux-amd64`
   - базовый образ: mcr.microsoft.com/dotnet/core/runtime:2.1.13-alpine
   - платформа:
      - архитектура: amd64
      - ОС: Linux
 
- Образ: `mcr.microsoft.com/azure-stream-analytics/azureiotedge:1.0.9-linux-arm32v7`
   - базовый образ: mcr.microsoft.com/dotnet/core/runtime:2.1.13-bionic-arm32v7
   - платформа:
      - архитектура: arm
      - ОС: Linux
 
- Образ: `mcr.microsoft.com/azure-stream-analytics/azureiotedge:1.0.9-linux-arm64`
   - базовый образ: mcr.microsoft.com/dotnet/core/runtime:3.0-bionic-arm64v8
   - платформа:
      - Архитектура: arm64
      - ОС: Linux
      
      
## <a name="get-help"></a>Получить справку
Для получения дополнительной помощи воспользуйтесь [страницей вопросов и ответов об Azure Stream Analytics](/answers/topics/azure-stream-analytics.html).

## <a name="next-steps"></a>Дальнейшие действия

* [Дополнительные сведения о Azure IoT Edge](../iot-edge/about-iot-edge.md)
* [Руководство по работе с IoT Edge Stream Analytics](../iot-edge/tutorial-deploy-stream-analytics.md)
* [Разработка заданий Edge Stream Analytics с помощью средств Visual Studio](./stream-analytics-tools-for-visual-studio-edge-jobs.md)
* [Сведения о реализации CI/CD для Stream Analytics с использованием API-интерфейсов](stream-analytics-cicd-api.md)

<!--Link references-->
[stream.analytics.developer.guide]: ../stream-analytics-developer-guide.md
[stream.analytics.scale.jobs]: stream-analytics-scale-jobs.md
[stream.analytics.introduction]: stream-analytics-introduction.md
[stream.analytics.get.started]: stream-analytics-real-time-fraud-detection.md
[stream.analytics.query.language.reference]: /stream-analytics-query/stream-analytics-query-language-reference
[stream.analytics.rest.api.reference]: /rest/api/streamanalytics/
