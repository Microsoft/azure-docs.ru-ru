---
author: msftradford
ms.service: azure-spatial-anchors
ms.topic: include
ms.date: 03/18/2021
ms.author: parkerra
ms.openlocfilehash: d91c0aeda2b7ae2f133d2099cbc9d238fd19d287
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104719953"
---
Чтобы скачать требуемые пакеты, необходимо установить <a href="https://www.npmjs.com/get-npm" target="_blank">NPM</a>.

Выполните следующую команду, заменив `<version_number>` версией Пространственных привязок Azure, которую необходимо скачать в текущую папку:

```bash
npm pack com.microsoft.azure.spatial-anchors-sdk.core@<version_number> --registry https://api.bintray.com/npm/microsoft/AzureMixedReality-NPM
```

> [!NOTE]
> Чтобы получить список доступных версий пакета Пространственных привязок Azure, выполните следующую команду:
>
> ```bash
> npm view com.microsoft.azure.spatial-anchors-sdk.core --registry https://api.bintray.com/npm/microsoft/AzureMixedReality-NPM versions
> ```

> [!WARNING]
> Минимальная поддерживаемая версия SDK для Пространственных привязок Azure (ASA) — 2.7.0. Если используется Unity 2020, минимальная поддерживаемая версия этого пакета SDK — 2.9.0.

Основной пакет Пространственных привязок Azure будет скачан в папку, из которой была выполнена команда.

Повторите этот шаг, чтобы скачать пакет для каждой платформы (Android/iOS/HoloLens), поддержку которой вы хотите обеспечить в своем проекте.

| Платформа | Имя пакета                                    |
|----------|-------------------------------------------------|
| Android  | com.microsoft.azure.spatial-anchors-sdk.android@<номер_версии> |
| iOS      | com.microsoft.azure.spatial-anchors-sdk.ios@<номер_версии>     |
| HoloLens | com.microsoft.azure.spatial-anchors-sdk.windows@<номер_версии> |