---
title: Установка пространственных привязок Azure для Unity
description: Настройка проекта Unity для использования пространственных привязок Azure
author: msftradford
manager: MehranAzimi-msft
services: azure-spatial-anchors
ms.author: parkerra
ms.date: 2/3/2021
ms.topic: how-to
ms.service: azure-spatial-anchors
ms.openlocfilehash: 56532c17c91e6c703a6acd7990bbae47cd248165
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99576571"
---
# <a name="configuring-azure-spatial-anchors-in-a-unity-project"></a>Настройка пространственных привязок Azure в проекте Unity

В этом руководство показано, как приступить к работе с пакетом SDK для пространственных привязок Azure в проекте Unity.

## <a name="project-requirements"></a>Проектные требования

[!INCLUDE [Unity Project Requirements](../../../includes/spatial-anchors-unity-project-requirements.md)]

## <a name="configuring-a-project"></a>Настройка проекта

Перед включением пакета SDK для пространственных привязок Azure в проекте Unity обязательно установите [необходимые](#project-requirements) пакеты с помощью диспетчера пакетов Unity.

### <a name="download-asa-packages"></a>Скачать пакеты ASA
[!INCLUDE [Download Unity Packages](../../../includes/spatial-anchors-unity-download-packages.md)]

### <a name="import-asa-packages"></a>Импорт пакетов ASA
[!INCLUDE [Import Unity Packages](../../../includes/spatial-anchors-unity-import-packages.md)]

### <a name="android-only-configure-the-maintemplategradle-file"></a>Только для Android: Настройка файла mainTemplate. gradle

1. Выберите **Изменить** > **Параметры проекта** > **Проигрыватель**.
2. На **панели инспектора** для **параметров проигрывателя** выберите значок **Android** .
3. В разделе **Сборка** установите флажок **настраиваемый основной шаблон Gradle** , чтобы создать пользовательский шаблон Gradle по адресу `Assets\Plugins\Android\mainTemplate.gradle` .
4. Откройте файл `mainTemplate.gradle` в текстовом редакторе.
5. В `dependencies` разделе вставьте следующие зависимости:

    ```gradle
    implementation('com.squareup.okhttp3:okhttp:[3.11.0]')
    implementation('com.microsoft.appcenter:appcenter-analytics:[1.10.0]')
    ```

Когда все будет готово, `dependencies` раздел должен выглядеть примерно так:

[!code-gradle[AzureSpatialAnchorsScript](../../../includes/spatial-anchors-unity-android-gradle-setup.md?range=9-13&highlight=3-4)]

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Как создавать и размещать привязки в Unity](./create-locate-anchors-unity.md)
