---
title: Базовые и пользовательские проверки
description: Узнайте о концепции Azure Defender для базовых показателей IoT.
services: defender-for-iot
ms.service: defender-for-iot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/07/2019
ms.author: mlottner
ms.openlocfilehash: bced45474a3a851bc5785f662c0b2e50ae3a380c
ms.sourcegitcommit: 4bda786435578ec7d6d94c72ca8642ce47ac628a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/16/2021
ms.locfileid: "103491084"
---
# <a name="azure-defender-for-iot-baseline-and-custom-checks"></a>Azure Defender для базового плана IoT и пользовательские проверки

В этой статье описывается защитник для базовых показателей IoT и приводятся сводные сведения обо всех связанных свойствах базовых пользовательских проверок.

## <a name="baseline"></a>Базовый

Базовый план устанавливает стандартное поведение для каждого устройства и упрощает установку необычного поведения или отклонений от ожидаемых норм.

## <a name="baseline-custom-checks"></a>Пользовательские проверки базовой конфигурации

Базовые пользовательские проверки. Создание пользовательского списка проверок для каждого базового уровня устройства с помощью **удостоверения модуля двойника** устройства.

## <a name="setting-baseline-properties"></a>Задание свойств базового плана

1. В центре Интернета вещей перейдите к устройству, которое вы хотите изменить, и выберите его.

1. Щелкните устройство, а затем щелкните модуль **азуреиотсекурити** .

1. Щелкните **модуль удостоверение двойника**.

1. Передайте на устройство файл **базовых пользовательских проверок** .

1. Добавьте свойства базового плана в защитник-IoT-Micro-Agent и нажмите кнопку **сохранить**.

### <a name="baseline-custom-check-file-example"></a>Пример файла пользовательской проверки базового уровня

Чтобы настроить базовые пользовательские проверки:

   ```json
    "desired": {
      "ms_iotn:urn_azureiot_Security_SecurityAgentConfiguration": {
        "baselineCustomChecksEnabled": {
          "value" : true
        },
        "baselineCustomChecksFilePath": {
          "value" : "/home/user/full_path.xml"
        },
        "baselineCustomChecksFileHash": {
          "value" : "#hashexample!"
        }
      }
    },
   ```

## <a name="baseline-custom-check-properties"></a>Свойства базовой пользовательской проверки

| Имя| Состояние | Допустимые значения| Значения по умолчанию| Описание |
|------|-----|------|-----|-----|
|баселинекустомчекксенаблед|Обязательный: true |Допустимые значения: **Boolean** |Значение по умолчанию: **false** |Максимальный интервал времени до отправки сообщений с высоким приоритетом.|
|баселинекустомчекксфилепас |Обязательный: true|Допустимые значения: **String**, **null** |Значение по умолчанию: **null** |Полный путь к базовой конфигурации XML|
|баселинекустомчекксфилехаш |Обязательный: true|Допустимые значения: **String**, **null** |Значение по умолчанию: **null** |`sha256sum` XML-файла конфигурации. Дополнительные сведения см. в [справочнике по sha256sum](https://linux.die.net/man/1/sha256sum) . |

Дополнительные примеры базовых показателей см. в разделе [Пользовательский базовый пример 1](https://ascforiot.blob.core.windows.net/public/custom_baseline_example_hyperv_ubuntu1804.xml) и [Пользовательский базовый пример — 2](https://ascforiot.blob.core.windows.net/public/oms_audits.xml).

## <a name="next-steps"></a>Дальнейшие действия

- Доступ к [необработанным данным безопасности](how-to-security-data-access.md)
- [Исследование устройства](how-to-investigate-device.md)
- Ознакомьтесь с [рекомендациями по безопасности](concept-recommendations.md)
- Изучение и изучение [оповещений системы безопасности](concept-security-alerts.md)
