---
title: Устранение неполадок центра Интернета вещей Azure 403006 Девицемаксимумактивефилеуплоадлимитексцеедед
description: Сведения об исправлении ошибки 403006 Девицемаксимумактивефилеуплоадлимитексцеедед
author: jlian
manager: briz
ms.service: iot-hub
services: iot-hub
ms.topic: troubleshooting
ms.date: 01/30/2020
ms.author: jlian
ms.openlocfilehash: 1e3c05e4cc3ccf34573b55d3729aded16e26d66e
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "76960845"
---
# <a name="403006-devicemaximumactivefileuploadlimitexceeded"></a>403006 DeviceMaximumActiveFileUploadLimitExceeded

В этой статье описываются причины и решения для ошибок **403006 девицемаксимумактивефилеуплоадлимитексцеедед** .

## <a name="symptoms"></a>Симптомы

Запрос на отправку файла завершается ошибкой с кодом **403006** и сообщением "число активных запросов на отправку файлов не может превышать 10".

## <a name="cause"></a>Причина

Каждый клиент устройства ограничен [10 параллельными передачами файлов](./iot-hub-devguide-quotas-throttling.md#other-limits). 

Вы можете легко превысить это ограничение, если устройство не уведомляет центр Интернета вещей о завершении передачи файлов. Эта проблема обычно вызвана ненадежной сетью на стороне устройства.

## <a name="solution"></a>Решение

Убедитесь, что устройство может немедленно [уведомить о завершении отправки файла центра Интернета вещей](./iot-hub-devguide-file-upload.md#notify-iot-hub-of-a-completed-file-upload). Затем попробуйте [уменьшить срок жизни маркера SAS для конфигурации отправки файла](iot-hub-configure-file-upload.md).

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о передаче файлов см. в разделе [Отправка файлов с помощью центра Интернета вещей](./iot-hub-devguide-file-upload.md) и [Настройка отправки файлов центра Интернета вещей с помощью портал Azure](./iot-hub-configure-file-upload.md).