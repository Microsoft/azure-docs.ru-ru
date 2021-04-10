---
author: craigktreasure
ms.service: azure-object-anchors
ms.topic: include
ms.date: 02/18/2021
ms.author: crtreasu
ms.openlocfilehash: ada83d6263ef033208200d810c53c5f045acc9a7
ms.sourcegitcommit: ed7376d919a66edcba3566efdee4bc3351c57eda
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105104308"
---
Следующий шаг — скачать пакет объектных привязок Azure для Unity.

# <a name="download-with-web-browser"></a>[Скачать с помощью веб-браузера](#tab/unity-package-web-ui)

Найдете пакет объектных привязок Azure для Unity [на этой странице](https://aka.ms/aoa/unity-sdk/package). Выберите нужную версию и скачайте пакет с помощью кнопки **Скачать**.

# <a name="download-with-npm"></a>[Скачать с помощью NPM](#tab/unity-package-npm)

Для выполнения этого действия должно быть установлено и доступно решение <a href="https://www.npmjs.com/get-npm" target="_blank">NPM</a>.

Выполните указанную ниже команду, заменив текст `<version_number>` на номер версии объектных привязок Azure, которую нужно скачать.

```bash
npm pack com.microsoft.azure.object-anchors.runtime@<version_number> --registry https://pkgs.dev.azure.com/aipmr/MixedReality-Unity-Packages/_packaging/Unity-packages/npm/registry/
```

> [!NOTE]
> Чтобы получить список доступных версий пакета объектных привязок Azure, выполните указанную ниже команду.
>
> ```bash
> npm view com.microsoft.azure.object-anchors.runtime --registry https://pkgs.dev.azure.com/aipmr/MixedReality-Unity-Packages/_packaging/Unity-packages/npm/registry/ versions
> ```

Пакет объектных привязок Azure будет скачан в папку, из которой была выполнена команда.

# <a name="install-with-mixed-reality-feature-tool-beta"></a>[Установить с помощью Mixed Reality Feature Tool (бета-версия)](#tab/unity-package-mixed-reality-feature-tool)

Перейдите к следующему шагу. Инструмент <a a href="/windows/mixed-reality/develop/unity/welcome-to-mr-feature-tool" target="_blank">Mixed Reality Feature Tool</a> понадобиться вам позднее.

---