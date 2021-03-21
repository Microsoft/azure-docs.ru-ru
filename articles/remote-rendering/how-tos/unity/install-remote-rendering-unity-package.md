---
title: Установка пакета Удаленной отрисовки для Unity
description: Объясняет, как установить клиентские библиотеки DLL удаленной подготовки к просмотру для Unity.
author: florianborn71
ms.author: flborn
ms.date: 02/26/2020
ms.topic: how-to
ms.openlocfilehash: 9454bef52798650fc431f8df994e1a964662b453
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101720834"
---
# <a name="install-the-remote-rendering-package-for-unity"></a>Установка пакета Удаленной отрисовки для Unity

Удаленная визуализация Azure использует пакет Unity для инкапсуляции интеграции в Unity.
Этот пакет содержит весь API C#, а также все двоичные файлы подключаемого модуля, необходимые для использования удаленной отрисовки Azure с Unity.
В следующей схеме именования Unity для пакетов пакет называется **com. Microsoft. Azure. Remote-Rendering**.

Можно выбрать один из следующих вариантов для установки пакета Unity.

## <a name="install-remote-rendering-package-using-the-mixed-reality-feature-tool"></a>Установка удаленного пакета подготовки к просмотру с помощью средства "функция смешанной реальности"

[Средство для функций смешанной реальности](https://aka.ms/MRFeatureToolDocs) ([download](https://aka.ms/mrfeaturetool)) — это средство, используемое для интеграции пакетов функций смешанной реальности в проекты Unity. Пакет не является частью [репозитория примеров arr](https://github.com/Azure/azure-remote-rendering)и недоступен из внутреннего реестра пакетов Unity.

Чтобы добавить пакет в проект, необходимо выполнить следующие действия.
1. [Загрузка средства "функция смешанной реальности"](https://aka.ms/mrfeaturetool)
1. Следуйте [полным инструкциям](https://aka.ms/MRFeatureToolDocs) по использованию этого средства.
1. На странице **Обнаружение компонентов** установите флажок для пакета **удаленного рендеринга Microsoft Azure** и выберите версию пакета, которую нужно добавить в проект.

![Mixed_Reality_feature_tool_package](media/mixed-reality-feature-tool-package.png)

Чтобы обновить локальный пакет, просто выберите более новую версию в средстве "функция смешанной реальности" и установите ее. Обновление пакета может иногда приводить к ошибкам консоли. Если это происходит, попробуйте закрыть и снова открыть проект.

## <a name="install-remote-rendering-package-manually"></a>Установить удаленный пакет подготовки отчетов вручную

Для установки пакета удаленной подготовки вручную необходимо выполнить следующие действия.

1. Скачайте пакет из канала NPM для смешанной реальности по адресу `https://pkgs.dev.azure.com/aipmr/MixedReality-Unity-Packages/_packaging/Unity-packages/npm/registry` .
    * Можно использовать [NPM](https://www.npmjs.com/get-npm) и выполнить следующую команду, чтобы скачать пакет в текущую папку.
      ```
      npm pack com.microsoft.azure.remote-rendering --registry https://pkgs.dev.azure.com/aipmr/MixedReality-Unity-Packages/_packaging/Unity-packages/npm/registry
      ```

    * Вы также можете использовать сценарий PowerShell `Scripts/DownloadUnityPackages.ps1` из [репозитория GitHub Azure-Remote-Rendering](https://github.com/Azure/azure-remote-rendering).
        * Изменить содержимое `Scripts/unity_sample_dependencies.json` на
          ```json
          {
            "packages": [
              {
                "name": "com.microsoft.azure.remote-rendering", 
                "version": "latest", 
                "registry": "https://pkgs.dev.azure.com/aipmr/MixedReality-Unity-Packages/_packaging/Unity-packages/npm/registry"
              }
            ]
          }
          ```

        * Выполните следующую команду в PowerShell, чтобы скачать пакет в указанный каталог назначения.
          ```
          DownloadUnityPackages.ps1 -DownloadDestDir <destination directory>
          ```

1. [Установите скачанный пакет](https://docs.unity3d.com/Manual/upm-ui-tarball.html) с помощью диспетчера пакетов Unity.

Чтобы обновить локальный пакет, перезапустите соответствующую команду и повторно импортируйте пакет. Обновление пакета может иногда приводить к ошибкам консоли. Если это происходит, попробуйте закрыть и снова открыть проект.

## <a name="unity-render-pipelines"></a>Конвейеры отрисовки Unity

Удаленная визуализация работает как с, так **:::no-loc text="Universal render pipeline":::** и с **:::no-loc text="Standard render pipeline":::** . По соображениям производительности рекомендуется использовать универсальный конвейер отрисовки.

Чтобы использовать **:::no-loc text="Universal render pipeline":::** , его пакет должен быть установлен в Unity. Это можно сделать в пользовательском интерфейсе **диспетчера пакетов** ( **Universal RP**, версии 7.3.1 или более новой) или с помощью `Packages/manifest.json` файла, как описано в [руководстве по установке проекта Unity](../../tutorials/unity/view-remote-models/view-remote-models.md#include-the-azure-remote-rendering-package).

## <a name="next-steps"></a>Дальнейшие действия

* [Игровые объекты и компоненты Unity](objects-components.md)
* [Учебник. Просмотр удаленных моделей](../../tutorials/unity/view-remote-models/view-remote-models.md)
