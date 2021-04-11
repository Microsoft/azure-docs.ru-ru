---
title: Руководство по изменению модулей динамической видеоаналитики Azure IoT Edge
description: В этом руководстве показано, как изменять и создавать модули шлюза динамической для видеоаналитики, которые используются в шаблоне приложения видеоаналитики для обнаружения объектов и движения.
services: iot-central
ms.service: iot-central
ms.subservice: iot-central-retail
ms.topic: tutorial
ms.author: nandab
author: KishorIoT
ms.date: 07/31/2020
ms.openlocfilehash: cbff2e774fae1f71624aa3cdbd83bfb21e3a9586
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101721469"
---
# <a name="tutorial-modify-and-build-the-live-video-analytics-gateway-modules"></a>Руководство по изменению и созданию модулей шлюза для динамической видеоаналитики

В этом руководстве показано, как изменить код модуля IoT Edge для модулей динамической видеоаналитики (LVA).

В предыдущих руководствах использовались предварительно созданные образы модулей.

## <a name="prerequisites"></a>Предварительные требования

Для выполнения шагов из этого руководства требуется следующее:


> [!div class="checklist"]
> * [Node.js](https://nodejs.org/en/download/) версии 10 или более поздней;
> * [Visual Studio Code](https://code.visualstudio.com/Download) с установленным расширением [TSLint](https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-typescript-tslint-plugin);
> * подсистема [Docker](https://www.docker.com/products/docker-desktop);
> * [Реестр контейнеров Azure](../../container-registry/index.yml) для размещения версий модулей;
> * учетная запись [Служб мультимедиа Azure](../../media-services/index.yml). Если вы уже прошли предыдущие руководства, вы можете использовать созданную учетную запись.

## <a name="clone-the-repository"></a>Клонирование репозитория

Если вы еще не сделали это, с помощью следующей команды клонируйте репозиторий в подходящее расположение на локальном компьютере.

```cmd
git clone https://github.com/Azure/live-video-analytics
```

Откройте папку репозитория *live-video-analytics* в редакторе VS Code.

## <a name="edit-the-deploymentamd64json-file"></a>Редактирование файла deployment.amd64.json

1. Если вы не сделали это, создайте папку с именем *ref-apps/lva-edge-iot-central-gateway/storage* в локальной копии репозитория **lva-gateway**. Эта папка игнорируется Git, чтобы предотвратить случайную синхронизацию конфиденциальной информации.
1. Скопируйте файл *deployment.amd64.json* из папки *setup* в папку *storage*.
1. В редакторе VS Code откройте файл *storage/deployment.amd64.json*.
1. Измените раздел `registryCredentials`, добавив в него учетные данные для Реестра контейнеров Azure.
1. Измените раздел модуля `LvaEdgeGatewayModule`, добавив в `env:amsAccountName:value` имя образа и имя учетной записи AMS.
1. Измените раздел модуля `lvaYolov3`, добавив имя образа.
1. Измените раздел модуля `lvaEdge`, добавив имя образа.
1. Дополнительные сведения о том, как завершить настройку, см. в статье [Руководство. Создание приложения видеоаналитики для отслеживания объектов и движения в Azure IoT Central](tutorial-video-analytics-create-app-yolo-v3.md).

## <a name="build-the-code"></a>Сборка кода

1. Перед первой попыткой выполнить сборку кода выполните команду `npm install` в терминале VS Code. Эта команда устанавливает необходимые пакеты и запускает скрипты установки.

    > [!TIP]
    > Если вы используете более новую версию репозитория GitHub, удалите папку *node_modules* и снова запустите `npm install`.

1. Измените файл *./setup/imageConfig.json*, указав новое значение имени образа с учетом имени реестра контейнеров.

    ```json
    {
        "arch": "amd64",
        "imageName": "[Server].azurecr.io/lva-edge-gateway"
    }
    ```

1. В терминале VS Code выполните команду `docker login [your server].azurecr.io`. Используйте те же учетные данные, которые вы указали в манифесте развертывания для модулей.

1. В терминале VS Code выполните команду **npm version patch**. Этот скрипт сборки развертывает образы в реестре контейнеров. Выходные данные в окне терминала VS Code проинформируют, успешно ли выполнена сборка.

1. Версия образа **LvaEdgeGatewayModule** увеличивается при каждом завершении сборки. Эту версию необходимо правильно указать в файле манифеста развертывания.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы завершили работу с приложением, можете удалить все созданные ресурсы следующим образом:

1. В приложении IoT Central перейдите на страницу **Ваше приложение** в разделе **Администрирование**. Теперь щелкните **Удалить**.
1. На портале Azure удалите группу ресурсов **lva-rg**.

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы ознакомились с шаблоном приложения видеоаналитики для обнаружения объектов и движения, а также с модулями LVA IoT Edge, мы рекомендуем перейти к следующему руководству:

> [!div class="nextstepaction"]
> [Создание решений для розничной торговли с помощью Azure IoT Central](overview-iot-central-retail.md)