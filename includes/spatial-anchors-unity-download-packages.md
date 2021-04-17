---
author: msftradford
ms.service: azure-spatial-anchors
ms.topic: include
ms.date: 03/30/2021
ms.author: parkerra
ms.openlocfilehash: 7faab3340483d99fa276de06f3fd7787457edb9e
ms.sourcegitcommit: 3ee3045f6106175e59d1bd279130f4933456d5ff
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106076717"
---
Следующий шаг — скачать пакеты пространственных привязок Azure для Unity. 

> [!WARNING]
> Минимальная поддерживаемая версия SDK для Пространственных привязок Azure (ASA) — 2.7.0. Если используется Unity 2020, минимальная поддерживаемая версия этого пакета SDK — 2.9.0.

Чтобы использовать пространственные привязки Azure в Unity, необходимо скачать основной пакет, а также пакет для каждой платформы (Android/iOS/HoloLens), которую планируется поддерживать.

| Платформа | Имя пакета                                    |
|----------|-------------------------------------------------|
| Android/iOS/HoloLens  | com.microsoft.azure.spatial-anchors-sdk.core@<номер_версии> |
| Android  | com.microsoft.azure.spatial-anchors-sdk.android@<номер_версии> |
| iOS      | com.microsoft.azure.spatial-anchors-sdk.ios@<номер_версии>     |
| HoloLens | com.microsoft.azure.spatial-anchors-sdk.windows@<номер_версии> |

# <a name="download-with-web-browser"></a>[Скачать с помощью веб-браузера](#tab/unity-package-web-ui)

Найдите основной пакет пространственных привязок Azure для Unity [на этой странице](https://aka.ms/aoa/unity-sdk/package). Выберите нужную версию и скачайте пакет с помощью кнопки **Скачать**. Повторите это действие, чтобы скачать пакет для каждой платформы, которую планируется поддерживать.

# <a name="download-with-npm"></a>[Скачать с помощью NPM](#tab/unity-package-npm)

Для выполнения этого действия должно быть установлено и доступно решение <a href="https://www.npmjs.com/get-npm" target="_blank">NPM</a>.

Выполните следующую команду, заменив `<version_number>` версией Пространственных привязок Azure, которую необходимо скачать в текущую папку:

```bash
npm pack com.microsoft.azure.spatial-anchors-sdk.core@<version_number> --registry https://pkgs.dev.azure.com/aipmr/MixedReality-Unity-Packages/_packaging/Unity-packages/npm/registry/
```

> [!NOTE]
> Чтобы получить список доступных версий пакета Пространственных привязок Azure, выполните следующую команду:
>
> ```bash
> npm view com.microsoft.azure.spatial-anchors-sdk.core --registry https://pkgs.dev.azure.com/aipmr/MixedReality-Unity-Packages/_packaging/Unity-packages/npm/registry/ versions
> ```

Основной пакет Пространственных привязок Azure будет скачан в папку, из которой была выполнена команда. Повторите это действие, чтобы скачать пакет для каждой платформы, которую планируется поддерживать.

# <a name="install-with-mixed-reality-feature-tool-beta"></a>[Установить с помощью Mixed Reality Feature Tool (бета-версия)](#tab/unity-package-mixed-reality-feature-tool)

Перейдите к следующему шагу. Инструмент <a a href="/windows/mixed-reality/develop/unity/welcome-to-mr-feature-tool" target="_blank">Mixed Reality Feature Tool</a> понадобиться вам позднее.

---