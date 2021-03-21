---
title: Настройка функции чтения вслух
titleSuffix: Azure Cognitive Services
description: В этой статье мы покажем, как настроить различные параметры чтения вслух.
author: metanMSFT
manager: guillasi
ms.service: cognitive-services
ms.subservice: immersive-reader
ms.topic: how-to
ms.date: 06/29/2020
ms.author: metang
ms.openlocfilehash: 8b45fe07b4bd42059199197bc5b99f20f6b2a8cf
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102608722"
---
# <a name="how-to-configure-read-aloud"></a>Как настроить чтение вслух

В этой статье показано, как настроить различные параметры чтения вслух в иммерсивное средство чтения.

## <a name="automatically-start-read-aloud"></a>Автоматически начать чтение вслух

`options`Параметр содержит все флаги, которые можно использовать для настройки чтения вслух. Установите `autoplay` значение `true` , чтобы разрешить автоматический запуск чтения вслух после запуска иммерсивное средство чтения.

```typescript
const options = {
    readAloudOptions: {
        autoplay: true
    }
};

ImmersiveReader.launchAsync(YOUR_TOKEN, YOUR_SUBDOMAIN, YOUR_DATA, options);
```

> [!NOTE]
> Из-за ограничений браузера автоматическое воспроизведение не поддерживается в Safari.

## <a name="configure-the-voice"></a>Настройка голоса

Задайте значение `voice` `male` или `female` . Не все языки поддерживают оба голоса. Дополнительные сведения см. на странице [Поддержка языков](./language-support.md) .

```typescript
const options = {
    readAloudOptions: {
        voice: 'female'
    }
};
```

## <a name="configure-playback-speed"></a>Настройка скорости воспроизведения

Задайте `speed` число от `0.5` (50%) и `2.5` (250%) полным. Значения за пределами этого диапазона будут иметь один из двух значений: 0,5 или 2,5.

```typescript
const options = {
    readAloudOptions: {
        speed: 1.5
    }
};
```

## <a name="next-steps"></a>Дальнейшие действия

* Ознакомьтесь со справочной документацией по [пакету SDK для иммерсивного средства чтения](./reference.md).