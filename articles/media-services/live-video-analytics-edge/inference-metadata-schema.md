---
title: Схема метаданных выводов — Azure
description: Из этой статьи вы узнаете о схеме метаданных выводов.
ms.topic: overview
ms.date: 09/14/2020
ms.openlocfilehash: 2de437577dc00692fb98c46fec32bfaa6612dc99
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92019500"
---
# <a name="inference-metadata-schema"></a>Схема метаданных вывода 

Независимо от используемого контракта (на основе HTTP или gRPC), каждый объект вывода соответствует объектной модели, описанной в этом разделе.

## <a name="object-model"></a>Объектная модель

![Объектная модель](./media/inference-metadata-schema/object-model.png)
 
|Определение типа|Описание|
|---|---|
|Тег|Тег или метка, связанная с результатом. Добавляя тег, вы также получаете связанное с ним значение достоверности.|
|Атрибут|Дополнительные атрибуты, связанные с результатом. Вы можете добавить новые атрибуты, полученные от подсистемы формирования выводов со значением достоверности.|
|Список атрибутов|Список необязательных атрибутов.|
|Прямоугольник|Прямоугольная область, построенная относительно верхнего левого угла изображения. Обязательными свойствами будут "длина", "ширина", "высота" и "расстояние верхнего края от источника".|
|Классификация|Метка классификатора часто применяется ко всему образцу. С помощью "тега" можно классифицировать результат.|
|Сущность|Сущность, обнаруженная или определенная в образце. При обнаружении подсистемой формирования выводов сущности она получает "тег", дополнительные выведенные атрибуты, а также координаты прямоугольника вокруг нее.|
|Событие|Событие, обнаруженное в образце. При обнаружении в образце события возвращаются характерные для него свойства и его имя.|
|Движение|Движение, обнаруженное в образце. Если в образце обнаружено движение, то возвращаются координаты ограничивающего прямоугольника, в котором оно выявлено.|
|text|Возвращается связанный с образцом текст вместе с метками времени для его начала и окончания.|
|Другое|Возвращает другие полезные данные общего характера.|

Приведенный ниже пример содержит одно событие с несколькими поддерживаемыми типами выводов.

```
[ 
  // Light Detection 
  { 
    "type": "classification", 
    "subtype": "lightDetection", 
    "classification": { 
      "tag": { "value": "daylight", "confidence": 0.86 }, 
      "attributes": [ 
          { "name": "isBlackAndWhite", "value": "false", "confidence": 0.71 } 
      ] 
    } 
  }, 
 
  // Motion Detection 
  { 
    "type": "motion", 
    "subtype": "motionDetection", 
    "motion": 
    { 
      "box": { "l": 0.0, "t": 0.0, "w": 0.0, "h": 0.0 } 
    } 
  }, 
 
  // Yolo V3 
  { 
    "type": "entity", 
    "subtype": "objectDetection",     
    "entity": 
    { 
      "tag": { "value": "dog", "confidence": 0.97 }, 
      "box": { "l": 0.0, "t": 0.0, "w": 0.0, "h": 0.0 } 
    } 
  }, 
 
  // Vehicle Identification 
  { 
    "type": "entity", 
    "subtype": "vehicleIdentification",     
    "entity": 
    { 
      "tag": { "value": "007-SPY", "confidence": 0.82 }, 
      "attributes":[   
        { "name": "color", "value": "black", "confidence": 0.90 }, 
        { "name": "body", "value": "coupe", "confidence": 0.87 }, 
        { "name": "make", "value": "Aston Martin", "confidence": 0.35 }, 
        { "name": "model", "value": "DBS V12", "confidence": 0.33 } 
      ], 
      "box": { "l": 0.0, "t": 0.0, "w": 0.0, "h": 0.0 } 
    } 
  }, 
 
  // People Identification 
  { 
    "type": "entity", 
    "subtype": "peopleIdentification",     
    "entity": 
    { 
      "tag": { "value":"Erwin Schrödinger", "confidence": 0.50 }, 
      "attributes":[   
        { "name": "age", "value": "73", "confidence": 0.87 }, 
        { "name": "glasses", "value": "yes", "confidence": 0.94 } 
      ], 
      "box": { "l": 0.0, "t": 0.0, "w": 0.0, "h": 0.0 } 
    }, 
 
    // Open type coming from the gRPC Map 
    "extensions":  
    { 
      "vector": "e1xkaXNwbGF5c3R5bGUgaVxoYmFyIHtcZnJhYyB7ZH17ZHR9fVx2ZXJ0IFxQc2kgKHQpXHJhbmdsZSA9e1xoYXQge0h9fVx2ZXJ0IFxQc2kgKHQpXHJhbmdsZSB9KQ==", 
      "skeleton": "p1,p2,p3,p4" 
    } 
  }, 
 
  // Captions 
  {     
    "type": "text", 
    "subtype": "speechToText",   
    "text": 
    { 
      "value": "Humor 75%. Confirmed. Self-destruct sequence in T minus 10… 9…", 
      "language": "en-US", 
      "startTimestamp": 12000, 
      "endTimestamp": 13000 
    } 
]
```

## <a name="next-steps"></a>Дальнейшие шаги

- [Контракт данных gRPC](./grpc-extension-protocol.md)
- [Контракт данных HTTP](./http-extension-protocol.md)