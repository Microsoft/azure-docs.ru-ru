---
title: Краткое руководство. Создание приложения HoloLens с помощью Unity
description: В этом кратком руководстве показано, как создать приложение HoloLens с использованием службы "Объектные привязки" в Unity.
author: craigktreasure
manager: virivera
ms.author: crtreasu
ms.date: 03/02/2021
ms.topic: quickstart
ms.service: azure-object-anchors
ms.openlocfilehash: 4f85a258042430d58690ef578db6d21a6c831d50
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102044807"
---
# <a name="quickstart-create-a-hololens-app-with-azure-object-anchors-in-unity"></a>Краткое руководство. Создание приложения HoloLens с использованием службы "Объектные привязки Azure" в Unity

В этом кратком руководстве показано, как создать приложение HoloLens в Unity с помощью службы [Объектные привязки Azure](../overview.md). "Объектные привязки Azure" — это управляемая облачная служба, которая преобразует трехмерные ресурсы в модели искусственного интеллекта, обеспечивая работу с объектами в смешанной реальности для HoloLens. По завершении работы с руководством у вас будет приложение HoloLens, созданное с помощью Unity, которое может обнаруживать объекты в реальном мире.

Вы узнаете, как:

> [!div class="checklist"]
> * настроить параметры сборки Unity;
> * экспортировать проект HoloLens в Visual Studio;
> * развернуть приложение и запустить его на устройстве HoloLens 2.

[!INCLUDE [Unity quickstart prerequisites](../../../includes/object-anchors-quickstart-unity-prerequisites.md)]

[!INCLUDE [Unity device setup](../../../includes/object-anchors-quickstart-unity-device-setup.md)]

## <a name="open-the-sample-project"></a>Открытие примера проекта

[!INCLUDE [Clone Sample Repo](../../../includes/object-anchors-clone-sample-repository.md)]

[!INCLUDE [Download Unity Package](../../../includes/object-anchors-quickstart-unity-download-package.md)]

В Unity откройте проект `quickstarts/apps/unity/basic`.

[!INCLUDE [Import Unity Package](../../../includes/object-anchors-quickstart-unity-import-package.md)]

[!INCLUDE [Unity build sample scene 1](../../../includes/object-anchors-quickstart-unity-build-sample-scene-1.md)]

[!INCLUDE [Unity build sample scene 2](../../../includes/object-anchors-quickstart-unity-build-sample-scene-2.md)]

[!INCLUDE [Unity build and deploy](../../../includes/object-anchors-quickstart-unity-build-deploy.md)]

### <a name="run-the-sample-app"></a>Запуск примера приложения

Включите устройство, выберите **All Apps** (Все приложения), а затем найдите и запустите приложение. После экрана-заставки Unity вы увидите сообщение о том, что наблюдатель объектов инициализирован. Но вам потребуется добавить модель в приложение.

[!INCLUDE [Unity setup Windows Device Portal](../../../includes/object-anchors-quickstart-unity-setup-device-portal.md)]

[!INCLUDE [Unity upload your model](../../../includes/object-anchors-quickstart-unity-upload-model.md)]

Приложение ищет объекты в текущем поле зрения, а после обнаружения отслеживает их. Экземпляр будет удален, если он находится на расстоянии 6 м от пользователя. В тексте отладки отображаются сведения об экземпляре, например идентификатор, метка времени обновления и коэффициент охвата поверхности.

[!INCLUDE [Unity troubleshooting](../../../includes/object-anchors-quickstart-unity-troubleshooting.md)]

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Unity HoloLens с MRTK](get-started-unity-hololens-mrtk.md)

> [!div class="nextstepaction"]
> [Общие сведения о пакете SDK среды выполнения](../concepts/sdk-overview.md)

> [!div class="nextstepaction"]
> [Часто задаваемые вопросы](../faq.md)
