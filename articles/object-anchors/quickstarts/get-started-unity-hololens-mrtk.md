---
title: Краткое руководство. Создание приложения HoloLens с помощью Unity и MRTK
description: В этом кратком руководстве показано, как создать приложение в Unity для HoloLens, используя MRTK и службу "Объектные привязки".
author: craigktreasure
manager: virivera
services: azure-object-anchors
ms.author: crtreasu
ms.date: 02/02/2021
ms.topic: quickstart
ms.service: azure-object-anchors
ms.openlocfilehash: 78a8136c3f66d0c790f6fd7508ca37b55a1541fd
ms.sourcegitcommit: c27a20b278f2ac758447418ea4c8c61e27927d6a
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/03/2021
ms.locfileid: "101747825"
---
# <a name="quickstart-create-a-hololens-app-with-azure-object-anchors-in-unity-with-mrtk"></a>Краткое руководство. Создание приложения HoloLens с использованием службы "Объектные привязки Azure" и MRTK в Unity

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

В Unity откройте проект `quickstarts/apps/unity/mrtk`.

[!INCLUDE [Import Unity Package](../../../includes/object-anchors-quickstart-unity-import-package.md)]

[!INCLUDE [Unity build and run](../../../includes/object-anchors-quickstart-unity-build-run.md)]

[!INCLUDE [Unity build sample scene 1](../../../includes/object-anchors-quickstart-unity-build-sample-scene-1.md)]

Когда откроется диалоговое окно "TMP Importer" (Средство импорта TMP) с запросом на импорт ресурсов TextMesh Pro, выберите "Import TMP Essentials" (Импорт основных компонентов TMP).
:::image type="content" source="./media/textmesh-pro-importer-dialog.png" alt-text="Импорт ресурсов TextMesh Pro":::

[!INCLUDE [Unity build sample scene 2](../../../includes/object-anchors-quickstart-unity-build-sample-scene-2.md)]

[!INCLUDE [Unity build and deploy](../../../includes/object-anchors-quickstart-unity-build-deploy.md)]

### <a name="run-the-sample-app"></a>Запуск примера приложения

Включите устройство, выберите **All Apps** (Все приложения), а затем найдите и запустите приложение. После экрана-заставки Unity должен отобразиться белый ограничивающий прямоугольник. Вы можете перемещать, масштабировать и поворачивать ограничивающий прямоугольник рукой. Поместите прямоугольник на объект, который нужно обнаружить.

Откройте <a href="https://microsoft.github.io/MixedRealityToolkit-Unity/Documentation/README_HandMenu.html" target="_blank">меню руки</a> и выберите **Lock Search Area** (Зафиксировать область поиска), чтобы предотвратить дальнейшее перемещение ограничивающего прямоугольника. Выберите **Start Search** (Начать поиск), чтобы начать обнаружение объектов. Когда объект будет обнаружен, на нем отобразится сетка. На экране отобразятся подробные сведения об обнаруженном экземпляре, например обновленная метка времени и коэффициент охвата поверхности. Выберите **Stop Search** (Прервать поиск), чтобы отменить отслеживание, и все обнаруженные экземпляры будут удалены.

#### <a name="the-app-menus"></a>Меню приложения

Можно также выполнить другие действия с помощью <a href="https://microsoft.github.io/MixedRealityToolkit-Unity/Documentation/README_HandMenu.html" target="_blank">меню руки</a>.

##### <a name="primary-menu"></a>Главное меню

* **Start Search / Stop Search** (Начать поиск / Прервать поиск) — запуск или остановка обнаружения объектов.
* **Toggle Spatial Mapping** (Переключить пространственное сопоставление) — отображение или скрытие пространственного сопоставления. С помощью этого параметра можно проверить, завершено ли сканирование.
* **Tracker Settings** (Параметры отслеживания) — активация или деактивация меню параметров для средства отслеживания.
* **Search Area Settings** (Параметры области поиска) — активация или деактивация меню параметров для области поиска.
* **Start Tracing** (Начать трассировку) — сбор данных диагностики и сохранение их на устройстве. Дополнительные сведения см. в разделе **Отладка проблем с обнаружением и сбор данных диагностики**.
* **Upload Tracing** (Отправка трассировки) — отправка данных диагностики в службу "Объектные привязки". Пользователь должен указать свою учетную запись подписки в файле `subscription.json` и отправить файл в папку `LocalState`. Пример файла `subscription.json` можно найти ниже.

    :::image type="content" source="./media/mrtk-hand-menu-primary.png" alt-text="Главное меню руки в Unity":::

##### <a name="tracker-settings-menu"></a>Меню параметров средства отслеживания

* **High Accuracy** (Высокая точность) — экспериментальная функция, используемая для более точной оценки пространственного расположения. Если включить этот параметр, обнаружение объектов потребует больше системных ресурсов. В этом режиме сетка объекта будет отображаться розовым цветом. Чтобы вернуться в обычный режим отслеживания, нажмите соответствующую кнопку еще раз.
* **Relaxed Vertical Alignment** (Нестрогое выравнивание по вертикали) — позволяет выполнять обнаружение объекта не по вертикали. Полезно для обнаружения объектов на наклонной поверхности.
* **Allow Scale Change** (Разрешить изменение масштаба) — позволяет средству отслеживания изменять размер обнаруженного объекта на основе информации о среде.
* **Coverage Ratio Slider** (Ползунок коэффициента охвата) — настройка коэффициента соответствия точек поверхности для обнаружения объекта средством отслеживания.  Низкие значения позволяют средству отслеживания лучше обнаруживать объекты (например, темные объекты или объекты, отражающие свет), которые трудно обнаружить с помощью датчиков HoloLens. Более высокие значения позволяют снизить частоту ложных обнаружений.

    :::image type="content" source="./media/mrtk-hand-menu-tracker.png" alt-text="Меню руки для средства отслеживания Unity":::

##### <a name="search-area-settings-menu"></a>Меню параметров области поиска

* **Lock Search Area** (Зафиксировать область поиска) — фиксация области, охватываемой ограничивающим прямоугольником, чтобы предотвратить его случайное перемещение руками.
* **Auto-Adjust Search Area** (Автонастройка области поиска) — обеспечивает автоматическое изменение положения для области поиска при обнаружении объектов.
* **Cycle Mesh** (Циклическое переключение сеток) — переключение путем визуализации загруженных сеток в области поиска.  С помощью этого параметра пользователь может выровнять область поиска для объектов, которые трудно обнаружить.

    :::image type="content" source="./media/mrtk-hand-menu-search-area.png" alt-text="Меню руки для области поиска Unity":::

Пример `subscription.json`:

```json
{
  "AccountId": "<your account id>",
  "AccountKey": "<your account key>",
  "AccountDomain": "<your account domain>"
}
```

[!INCLUDE [Unity setup Windows Device Portal](../../../includes/object-anchors-quickstart-unity-setup-device-portal.md)]

[!INCLUDE [Unity upload your model](../../../includes/object-anchors-quickstart-unity-upload-model.md)]

[!INCLUDE [Unity troubleshooting](../../../includes/object-anchors-quickstart-unity-troubleshooting.md)]

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Общие сведения о пакете SDK среды выполнения](../concepts/sdk-overview.md)

> [!div class="nextstepaction"]
> [Часто задаваемые вопросы](../faq.md)
