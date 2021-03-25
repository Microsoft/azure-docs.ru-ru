---
author: craigktreasure
ms.service: azure-object-anchors
ms.topic: include
ms.date: 02/18/2021
ms.author: crtreasu
ms.openlocfilehash: 4345810292896cf88de19baf419eee025ba5853f
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102044789"
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

Перейдите к следующему шагу. Инструмент <a a href="https://aka.ms/MRFeatureToolDocs" target="_blank">Mixed Reality Feature Tool</a> понадобиться вам позднее.

---
