---
title: Развертывание веб-приложения пространственного анализа
titleSuffix: Azure Cognitive Services
description: Узнайте, как использовать пространственный анализ в веб-приложении.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: conceptual
ms.date: 01/12/2021
ms.author: aahi
ms.openlocfilehash: de011fb0f827ea90efe33e237bbf1c5100dc76a7
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98183478"
---
# <a name="how-to-deploy-a-people-counting-web-application"></a>Руководство. Развертывание веб-приложения для инвентаризации людей

В этой статье содержатся сведения о том, как интегрировать пространственный анализ в веб-приложение, которое понимает перемещение людей, и отслеживает количество людей, занимающих физическое пространство. 

Из этого учебного курса вы узнаете следующее:

* Развертывание контейнера пространственного анализа
* Настройка операции и камеры
* Настройка подключения центра Интернета вещей в веб-приложении
* Развертывание и тестирование веб-приложения

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services/).
* Основные сведения о конфигурациях развертывания Azure IoT Edge и [центре Интернета вещей Azure](../../iot-hub/index.yml)
* Настроенный [главный компьютер](spatial-analysis-container.md).

## <a name="deploy-the-spatial-analysis-container"></a>Развертывание контейнера пространственного анализа

Заполните [приложение запроса](https://aka.ms/csgate) , чтобы получить доступ для запуска контейнера. 

Выполните [установку главного компьютера](./spatial-analysis-container.md) , чтобы настроить главный компьютер и подключить устройство IOT Edge к центру Интернета вещей Azure. 

### <a name="deploy-an-azure-iot-hub-service-in-your-subscription"></a>Развертывание службы центра Интернета вещей Azure в подписке

Сначала создайте экземпляр службы центра Интернета вещей Azure с ценовой категорией "Стандартный" (S1) или Free (S0). Выполните эти инструкции, чтобы создать этот экземпляр с помощью Azure CLI.

Заполните обязательные параметры:
* Подписка. имя или идентификатор подписки Azure.
* Группа ресурсов. Создайте имя для группы ресурсов.
* Имя центра Интернета вещей: Создание имени для центра Интернета вещей
* Имя IoTHub: имя созданного центра Интернета вещей. 
* Имя пограничной устройства: Создайте имя для устройства с пограничным устройством.

```azurecli
az login
az account set --subscription <name or ID of Azure Subscription>
az group create --name "<Resource Group Name>" --location "WestUS"

az iot hub create --name "<IoT Hub Name>" --sku S1 --resource-group "test-resource-group"

az iot hub device-identity create --hub-name "<IoT Hub Name>" --device-id "<Edge Device Name>" --edge-enabled
```

### <a name="deploy-the-container-on-azure-iot-edge-on-the-host-computer"></a>Развертывание контейнера на Azure IoT Edge на главном компьютере

Разверните контейнер пространственного анализа как модуль IoT на главном компьютере с помощью Azure CLI. Для процесса развертывания требуется файл манифеста развертывания, который описывает необходимые контейнеры, переменные и конфигурации для развертывания. В GitHub можно найти пример [манифеста развертывания](https://go.microsoft.com/fwlink/?linkid=2142179)для конкретного Azure Stack, не относящегося к [Azure Stack](https://go.microsoft.com/fwlink/?linkid=2152189), а также [виртуальной машины Azure с манифестом развертывания на основе GPU](https://go.microsoft.com/fwlink/?linkid=2152189) , который включает базовую конфигурацию развертывания для контейнера *пространственного анализа* . 

Кроме того, вы можете использовать расширения Azure IoT для Visual Studio Code для выполнения операций с центром Интернета вещей. Чтобы узнать больше, перейдите к разделу [Развертывание модулей Azure IOT Edge из Visual Studio Code](../../iot-edge/how-to-deploy-modules-vscode.md) .

> [!NOTE] 
> Контейнеры *spatial-Analysis-Telegraf* и *пространственный-Analysis-Diagnostics* являются необязательными. Вы можете удалить их из *DeploymentManifest.js* файла. Дополнительные сведения см. в статье [телеметрии и устранении неполадок](./spatial-analysis-logging.md) . Вы можете найти три примера *DeploymentManifest.js* файлов на сайте GitHub, для [Azure Stackных устройств](https://go.microsoft.com/fwlink/?linkid=2142179), на [настольном компьютере](https://go.microsoft.com/fwlink/?linkid=2152189)или на [виртуальной машине Azure с графическим процессором](https://go.microsoft.com/fwlink/?linkid=2152189) .

### <a name="set-environment-variables"></a>Настройка переменных среды

Большая часть **переменных среды** для модуля IOT EDGE уже заданы в образце *DeploymentManifest.jsдля* файлов, связанных выше. В файле найдите `BILLING_ENDPOINT` `API_KEY` переменные среды и, показанные ниже. Замените значения на URI конечной точки и созданный ранее ключ API. Убедитесь, что для параметра лицензионное соглашение установлено значение "принять". 

```json
"EULA": { 
    "value": "accept"
},

"BILLING_ENDPOINT":{ 
    "value": "<Use a key from your Computer Vision resource>"
},
"API_KEY":{
    "value": "<Use the endpoint from your Computer Vision resource>"
}
```

### <a name="configure-the-operation-parameters"></a>Настройка параметров операции

Теперь, когда начальная конфигурация контейнера *пространственного анализа* завершена, следующим шагом является настройка параметров операций и их добавление в развертывание. 

Первым шагом является обновление примера манифеста развертывания, связанного выше, и настройка инструкции для, `cognitiveservices.vision.spatialanalysis-personcount` как показано ниже:


```json
"personcount": {
    "operationId": "cognitiveservices.vision.spatialanalysis-personcount",
    "version": 1,
    "enabled": true,
    "parameters": {
        "VIDEO_URL": "<Replace RTSP URL here>",
        "VIDEO_SOURCE_ID": "<Replace with friendly name>",
        "VIDEO_IS_LIVE":true,
        "DETECTOR_NODE_CONFIG": "{ \"gpu_index\": 0 }",
        "SPACEANALYTICS_CONFIG": "{\"zones\":[{\"name\":\"queue\",\"polygon\":[<Replace with your values>], \"events\": [{\"type\":\"count\"}], \"threshold\":<use 0 for no threshold.}]}"
    }
},
```

После обновления манифеста развертывания следуйте инструкциям производителя камеры по установке камеры, настройке URL-адреса камеры и настройке имени пользователя и пароля. 

Затем укажите `VIDEO_URL` URL-адрес RTSP камеры и учетные данные для подключения к камере.

Если на пограничном устройстве установлено более одного GPU, выберите GPU, на котором будет выполняться эта операция. Убедитесь, что балансировка нагрузки выполняется для операций, в которых на одном GPU одновременно не более 8 операций.  

Затем настройте зону, в которой нужно подсчитать пользователей. Чтобы настроить многоугольник зоны, сначала следуйте инструкциям производителя, чтобы получить кадр с камеры. Чтобы определить каждую вершину многоугольника, выберите точку на кадре, возьмите координаты x, y точки относительно левого, верхнего угла фрейма и разделите их по соответствующим размерам рамки. Задайте результаты в виде координат x, y вершины. В поле можно задать конфигурацию многоугольников зоны `SPACEANALYTICS_CONFIG` .

Это образец видеокадра, показывающий, как вычисляется координата вершин для кадра размером 1920/1080.
![Образец видеокадра](./media/spatial-analysis/sample-video-frame.jpg)

Вы также можете выбрать пороговое значение достоверности при подсчете обнаруженных людей и создания событий. Установите пороговое значение 0, если хотите выводить все события.

### <a name="execute-the-deployment"></a>Выполнение развертывания

Теперь, когда манифест развертывания завершен, используйте эту команду в Azure CLI для развертывания контейнера на главном компьютере в качестве модуля IoT Edge.

```azurecli
az login
az extension add --name azure-iot
az iot edge set-modules --hub-name "<IoT Hub name>" --device-id "<IoT Edge device name>" --content DeploymentManifest.json -–subscription "<subscriptionId>"
```

Заполните обязательные параметры:

* Имя центра Интернета вещей: имя центра Интернета вещей Azure
* DeploymentManifest.js: имя файла развертывания
* Имя устройства IoT Edge: имя IoT Edgeного компьютера узла.
* Подписка: идентификатор или имя подписки

Эта команда начнет развертывание, и вы сможете просмотреть состояние развертывания в экземпляре центра Интернета вещей Azure в портал Azure. Состояние может отображаться как *417 — конфигурация развертывания устройства не устанавливается* до тех пор, пока устройство не закончит загрузку образов контейнеров и не начнет работать.

### <a name="validate-that-the-deployment-was-successful"></a>Проверка успешности развертывания

Нахождение *состояния среды выполнения* в параметрах модуля IOT Edge для модуля пространственного анализа в экземпляре центра Интернета вещей на портал Azure. **Требуемое значение** и **Сообщаемое значение** *состояния среды выполнения* должны быть указаны `Running` . Ниже приведены сведения о том, как это будет выглядеть на портал Azure.

![Пример проверки развертывания](./media/spatial-analysis/deployment-verification.png)

На этом этапе контейнер пространственного анализа выполняет операцию. Он выдает сведения об искусственном интеллекте для `cognitiveservices.vision.spatialanalysis-personcount` операции и направляет эти данные в качестве телеметрии в экземпляр центра Интернета вещей Azure. Чтобы настроить дополнительные камеры, можно обновить файл манифеста развертывания и выполнить развертывание еще раз.

## <a name="person-counting-web-application"></a>Веб-приложение для инвентаризации лиц

Этот пользователь подсчитывает веб-приложение, позволяющее быстро настроить пример веб-приложения и разместить его в среде Azure.

### <a name="get-the-person-counting-app-container"></a>Получение контейнера приложения для подсчета лиц

Форма контейнера этого приложения, доступная в реестре контейнеров Azure. Чтобы скачать его, используйте следующую команду docker pull. Обратитесь в корпорацию Майкрософт по адресу projectarchon@microsoft.com для получения маркера доступа.

```bash
docker login rtvsofficial.azurecr.io -u <token name> -p <password>
docker pull rtvsofficial.azurecr.io/acceleratorapp.personcount:1.0
```

Отправьте контейнер в реестр контейнеров Azure (запись контроля доступа).

```bash
az acr login --name <your ACR name>

docker tag rtvsofficial.azurecr.io/acceleratorapp.personcount:1.0 [desired local image name]

docker push [desired local image name]
```

Чтобы установить контейнер, создайте новый Веб-приложение для контейнеров Azure и введите необходимые параметры. Затем перейдите на вкладку **DOCKER** и выберите **один контейнер**, а затем **Реестр контейнеров Azure**. Используйте свой экземпляр реестра контейнеров Azure, где вы отправили приведенный выше образ.

![Введите сведения об образе](./media/spatial-analysis/solution-app-create-screen.png)

Введя указанные выше параметры, щелкните **Проверка + создать** и создайте приложение.

### <a name="configure-the-app"></a>Настройка приложения 

Дождитесь завершения установки и перейдите к ресурсу в портал Azure. Перейдите к разделу **конфигурации** и добавьте следующие два **параметра приложения**.

* `EventHubConsumerGroup` — Строковое имя группы потребителей из центра Интернета вещей Azure, можно создать новую группу потребителей в центре Интернета вещей или использовать группу по умолчанию. 
* `IotHubConnectionString` — Строка подключения к центру Интернета вещей Azure, ее можно получить из раздела ключи в ресурсе центра Интернета вещей Azure ![ Настройка параметров](./media/spatial-analysis/solution-app-config-page.png)

После добавления этих двух параметров нажмите кнопку **сохранить**. Затем в меню навигации слева щелкните **Проверка подлинности и авторизация** и обновите требуемый уровень проверки подлинности. Рекомендуем Azure Active Directory (Azure AD) Express. 

### <a name="test-the-app"></a>Тестирование приложения

Перейдите в веб-приложение Azure и убедитесь, что развертывание прошло успешно, а веб-приложение запущено. Перейдите по настроенному URL-адресу: `<yourapp>.azurewebsites.net` для просмотра работающего приложения.

![тестирование развертывания](./media/spatial-analysis/solution-app-output.png)

## <a name="get-the-personcount-source-code"></a>Получение исходного кода Персонкаунт
Если вы хотите просмотреть или изменить исходный код для этого приложения, его можно найти [на сайте GitHub](https://github.com/Azure-Samples/cognitive-services-spatial-analysis).

## <a name="next-steps"></a>Дальнейшие действия

* [Настройка операций пространственного анализа](./spatial-analysis-operations.md)
* [Ведение журнала и устранение неполадок](spatial-analysis-logging.md)
* [Путеводитель по размещению камеры](spatial-analysis-camera-placement.md)
* [Рекомендации по размещению зон и строк](spatial-analysis-zone-line-placement.md)
