---
title: Изучение рекомендаций на основе теста производительности CIS
description: Выполняйте базовое и расширенное расследование на основе рекомендаций базовых показателей ОС.
ms.date: 1/21/2021
ms.topic: how-to
ms.openlocfilehash: 01ca6e1fecddff9800872a3e5495a5cac578a74f
ms.sourcegitcommit: f611b3f57027a21f7b229edf8a5b4f4c75f76331
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/22/2021
ms.locfileid: "104782033"
---
# <a name="investigate-os-baseline-based-on-cis-benchmark-recommendation"></a>Рекомендации по базовым показателям ОС (на основе тестирования производительности CI) 

Выполняйте базовое и расширенное расследование на основе рекомендаций базовых показателей ОС.

## <a name="basic-os-baseline-security-recommendation-investigation"></a>Базовая базовая рекомендация по безопасности ОС — исследование  

Вы можете изучить рекомендации по базовым показателям ОС, перейдя на портал Azure для центра Интернета вещей в **центре Интернета вещей**. Дополнительные сведения см. в разделе [изучение рекомендаций по безопасности](quickstart-investigate-security-recommendations.md).

## <a name="advanced-os-baseline-security-recommendation-investigation"></a>Расширенная базовая рекомендация по безопасности ОС — исследование  

В этом разделе описывается, как лучше понять результаты базового теста ОС и запросить события в Log Analytics Azure.  

Исследование рекомендаций по безопасности для базовой ОС поддерживается только с помощью log Analytics. Перед продолжением Подключите защитник для Интернета вещей к рабочей области Log Analytics. Дополнительные сведения о дополнительных базовых рекомендациях по безопасности ОС см. в статье [Настройка защитника Azure для решения на основе агента IOT](how-to-configure-agent-based-solution.md).

Чтобы запросить события безопасности IoT в Log Analytics для оповещений:

1. Перейдите на страницу **оповещения** .

1. Выберите **исследовать рекомендации в log Analytics рабочей области**.

Чтобы запросить события безопасности IoT в Log Analytics для получения рекомендаций:

1. Перейдите на страницу **рекомендации** .

1. Выберите **исследовать рекомендации в log Analytics рабочей области**.

1. Чтобы просмотреть сведения о конкретном устройстве, выберите **Показать сведения о базовых правилах базовой системы (ОС)** на странице краткий обзор **сведений рекомендации** .

   :::image type="content" source="media/how-to-investigate-cis-benchmark/recommendation-details.png" alt-text="Просмотр сведений о конкретном устройстве."::: 

Для прямого запроса событий безопасности IoT в рабочей области Log Analytics:

1. Перейдите на страницу **журналы** .

    :::image type="content" source="media/how-to-investigate-cis-benchmark/logs.png" alt-text="Выберите журналы в левой части области.":::

1. Выберите **исследовать оповещения** или выберите параметр **исследовать оповещения в log Analytics** в любом из рекомендаций по безопасности или в поле Alert (оповещение).   

## <a name="useful-queries-to-investigate-the-os-baseline-resources"></a>Полезные запросы для изучения базовых ресурсов ОС: 

> [!Note]
> Обязательно замените имена, `<device-id>` присвоенные устройству, в каждом из следующих запросов. 


### <a name="retrieve-the-latest-information"></a>Получение последних сведений

- **Сбой парка устройства**: выполните следующий запрос, чтобы получить актуальные сведения о проверках, которые завершились сбоем на парке устройства: 

    ```azurecli
    let lastDates = SecurityIoTRawEvent | 
    
    where RawEventName == "OSBaseline" | 
    
    summarize TimeStamp=max(TimeStamp) by DeviceId; 
    
    lastDates | join kind=inner (SecurityIoTRawEvent) on TimeStamp, DeviceId  | 
    
    extend event = parse_json(EventDetails) | 
    
    where event.Result == "FAIL" | 
    
    project DeviceId, event.CceId, event.Description 
    ```
 
- **Сбой конкретного устройства** . выполните следующий запрос, чтобы получить последние сведения о проверках, завершившихся сбоем на конкретном устройстве:  

    ```azurecli
    let LastEvents = SecurityIoTRawEvent | 
    
    where RawEventName == "OSBaseline" | 
    
    where DeviceId == "<device-id>" | 
    
    top 1 by TimeStamp desc | 
    
    project IoTRawEventId; 
    
    LastEvents | join kind=leftouter SecurityIoTRawEvent on IoTRawEventId | 
    
    extend event = parse_json(EventDetails) | 
    
    where event.Result == "FAIL" | 
    
    project DeviceId, event.CceId, event.Description 
    ```

- **Ошибка конкретного устройства** . Выполните этот запрос, чтобы получить последние сведения о проверках, имеющих ошибку на определенном устройстве: 

    ```azurecli
    let LastEvents = SecurityIoTRawEvent | 
    
    where RawEventName == "OSBaseline" | 
    
    where DeviceId == "<device-id>" | 
    
    top 1 by TimeStamp desc | 
    
    project IoTRawEventId; 
    
    LastEvents | join kind=leftouter SecurityIoTRawEvent on IoTRawEventId | 
    
    extend event = parse_json(EventDetails) | 
    
    where event.Result == "ERROR" | 
    
    project DeviceId, event.CceId, event.Description 
    ```
 
- **Обновление списка устройств для парка устройства, которое не прошло определенную проверку** — выполните этот запрос, чтобы получить обновленный список устройств (через парк устройства), которые не прошли определенную проверку:  
 
    ```azurecli
    let lastDates = SecurityIoTRawEvent | 
    
    where RawEventName == "OSBaseline" | 
    
    summarize TimeStamp=max(TimeStamp) by DeviceId; 
    
    lastDates | join kind=inner (SecurityIoTRawEvent) on TimeStamp, DeviceId  | 
    
    extend event = parse_json(EventDetails) | 
    
    where event.Result == "FAIL" | 
    
    where event.CceId contains "6.2.8" | 
    
    project DeviceId; 
    ```
 
## <a name="next-steps"></a>Следующие шаги

[Изучите рекомендации по безопасности](quickstart-investigate-security-recommendations.md).
 