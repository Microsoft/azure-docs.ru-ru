---
title: Интеграция службы хранилища Azure с уведомлениями и резервным копированием модели
titleSuffix: Azure Cognitive Services
description: Узнайте, как интегрировать службу хранилища Azure для получения push-уведомлений при обучении или экспорте Пользовательское визуальное распознавание моделей. Также можно сохранить резервную копию экспортированных моделей.
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: custom-vision
ms.topic: how-to
ms.date: 09/11/2020
ms.author: pafarley
ms.openlocfilehash: fd4ad1443f466a78abfc569d5f52f6bdeff2d5be
ms.sourcegitcommit: ed7376d919a66edcba3566efdee4bc3351c57eda
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105048890"
---
# <a name="integrate-azure-storage-for-notifications-and-backup"></a>Интеграция службы хранилища Azure с уведомлениями и резервным копированием

Проект "Пользовательское визуальное распознавание" можно интегрировать с очередью Хранилища BLOB-объектов Azure, чтобы получать push-уведомления об обучении или экспорте проекта и о резервных копиях опубликованных моделей. Эта функция позволяет избежать постоянного опроса службы на предмет результатов, когда выполняются длительные операции. Вместо этого уведомления очереди хранилища можно интегрировать в рабочий процесс.

В этом руководство показано, как использовать эти API-интерфейсы RESTFUL с фигурными скобками. Можно также использовать службу HTTP-запросов, например POST, чтобы выдать запросы.

> [!NOTE]
> Push-уведомления зависят от необязательного параметра _нотификатионкуеуеури_ в API **CreateProject** , и для резервных копий модели необходимо также использовать необязательный параметр _експортмоделконтаинерури_ . В этом руководством будут использоваться оба варианта для полного набора функций.

## <a name="prerequisites"></a>Предварительные требования

- Ресурс Пользовательское визуальное распознавание в Azure. Если у вас ее нет, перейдите к портал Azure и [Создайте новый ресурс пользовательское визуальное распознавание](https://portal.azure.com/?microsoft_azure_marketplace_ItemHideKey=microsoft_azure_cognitiveservices_customvision#create/Microsoft.CognitiveServicesCustomVision?azure-portal=true). В настоящее время эта функция не поддерживает ресурс службы "непонятный" (все в одном ключе).
- Учетная запись хранения Azure с контейнером больших двоичных объектов. Выполните [упражнения 1 лаборатории службы хранилища Azure](https://github.com/Microsoft/computerscience/blob/master/Labs/Azure%20Services/Azure%20Storage/Azure%20Storage%20and%20Cognitive%20Services%20(MVC).md#Exercise1) , если вам нужна помощь с этим шагом.
* [PowerShell версии 6.0 и выше](/powershell/scripting/install/installing-powershell-core-on-windows) или аналогичное приложение командной строки.

## <a name="set-up-azure-storage-integration"></a>Настройка интеграции службы хранилища Azure

Перейдите к учебному ресурсу Пользовательское визуальное распознавание на портал Azure, выберите страницу **удостоверение** и включите управляемое удостоверение, назначенное системой.

Затем перейдите к ресурсу хранилища в портал Azure. Перейдите на страницу **управления доступом (IAM)** и добавьте назначение ролей для каждой функции интеграции:
* Выберите ресурс обучения Пользовательское визуальное распознавание и назначьте роль **участника данных BLOB-объекта хранилища** , если планируется использовать функцию резервного копирования модели. 
* Затем выберите ресурс обучения Пользовательское визуальное распознавание и назначьте **участника данных очереди хранилища** , если планируется использовать функцию очереди уведомлений.

> [!div class="mx-imgBorder"]
> ![Страница "Добавление назначения ролей" для учетной записи хранения](./media/storage-integration/storage-access.png)

### <a name="get-integration-urls"></a>Получение URL-адресов интеграции

Далее вы получите URL-адреса, позволяющие Пользовательское визуальное распознаваниеому ресурсу получать доступ к этим конечным точкам.

В поле URL-адрес интеграции очереди уведомлений перейдите на страницу **очереди** учетной записи хранения, добавьте новую очередь и сохраните ее URL-адрес во временном расположении.

> [!div class="mx-imgBorder"]
> ![Страница очереди службы хранилища Azure](./media/storage-integration/queue-url.png) 

Для URL-адреса интеграции резервного копирования модели перейдите на страницу **контейнеры** вашей учетной записи хранения и создайте новый контейнер. Затем выберите его и перейдите на страницу **свойств** . Скопируйте URL-адрес во временное расположение.
 
> [!div class="mx-imgBorder"]
> ![Страница свойств контейнера службы хранилища Azure](./media/storage-integration/container-url.png) 


## <a name="integrate-custom-vision-project"></a>Интеграция Пользовательское визуальное распознавание проекта

Теперь, когда у вас есть URL-адреса интеграции, можно создать новый проект Пользовательское визуальное распознавание, который интегрирует функции службы хранилища Azure. Можно также обновить существующий проект, чтобы добавить функции.

### <a name="create-new-project"></a>Создание нового проекта

При вызове API [CreateProject](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Training_3.3/operations/5eb0bcc6548b571998fddeae) добавьте необязательные параметры _експортмоделконтаинерури_ и _нотификатионкуеуеури_. Назначьте значения URL-адресов, которые были получены в предыдущем разделе. 

```curl
curl -v -X POST "{endpoint}/customvision/v3.3/Training/projects?exportModelContainerUri={inputUri}&notificationQueueUri={inputUri}&name={inputName}"
-H "Training-key: {subscription key}"
```

Если получен ответ, то это означает, что `200/OK` URL-адреса успешно настроены. Значения URL-адреса должны отображаться также в ответе JSON:

```json
{
  "id": "00000000-0000-0000-0000-000000000000",
  "name": "string",
  "description": "string",
  "settings": {
    "domainId": "00000000-0000-0000-0000-000000000000",
    "classificationType": "Multiclass",
    "targetExportPlatforms": [
      "CoreML"
    ],
    "useNegativeSet": true,
    "detectionParameters": "string",
    "imageProcessingSettings": {
      "augmentationMethods": {}
},
"exportModelContainerUri": {url}
"notificationQueueUri": {url}
  },
  "created": "string",
  "lastModified": "string",
  "thumbnailUri": "string",
  "drModeEnabled": true,
  "status": "Succeeded"
}
```

### <a name="update-existing-project"></a>Обновить существующий проект

Чтобы обновить существующий проект с помощью интеграции функций хранилища Azure, вызовите API [упдатепрожект](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Training_3.3/operations/5eb0bcc6548b571998fddeb1) , используя идентификатор проекта, который требуется обновить. 

```curl
curl -v -X PATCH "{endpoint}/customvision/v3.3/Training/projects/{projectId}"
-H "Content-Type: application/json"
-H "Training-key: {subscription key}"

--data-ascii "{body}" 
```

Задайте текст запроса ( `body` ) в следующем формате JSON, заполнив соответствующие значения для _Експортмоделконтаинерури_ и _нотификатионкуеуеури_:

```json
{
  "name": "string",
  "description": "string",
  "settings": {
    "domainId": "00000000-0000-0000-0000-000000000000",
    "classificationType": "Multiclass",
    "targetExportPlatforms": [
      "CoreML"
    ],
    "imageProcessingSettings": {
      "augmentationMethods": {}
},
"exportModelContainerUri": {url}
"notificationQueueUri": {url}

  },
  "status": "Succeeded"
}
```

Если получен ответ, то это означает, что `200/OK` URL-адреса успешно настроены.

## <a name="verify-the-connection"></a>Проверка подключения 

Вызов API в предыдущем разделе должен уже активировать новые сведения в учетной записи хранения Azure. 

В указанном контейнере должен быть тестовый большой двоичный объект в папке **кустомвисион-тестпермиссион** . Этот большой двоичный объект будет существовать только временно.

В очереди уведомлений должно отобразиться тестовое уведомление в следующем формате:
 
```json
{
"version": "1.0" ,
"type": "ConnectionTest",
"Content":
    {
    "projectId": "00000000-0000-0000-0000-000000000000"
    }
}
```

## <a name="get-event-notifications"></a>Получение уведомлений о событиях

Когда вы будете готовы, вызовите API [траинпрожект](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Training_3.3/operations/5eb0bcc7548b571998fddee1) для своего проекта, чтобы выполнить обычные операции обучения.

В очереди уведомлений хранилища вы получите уведомление после завершения обучения:

```json
{
"version": "1.0" ,
"type": "Training",
"Content":
    {
    "projectId": "00000000-0000-0000-0000-000000000000",
    "iterationId": "00000000-0000-0000-0000-000000000000",
    "trainingStatus": "TrainingCompleted"
    }
}
```

`"trainingStatus"`Поле может иметь значение `"TrainingCompleted"` или `"TrainingFailed"` . `"iterationId"`Поле является идентификатором обученной модели.

## <a name="get-model-export-backups"></a>Получение резервных копий экспорта модели

Когда вы будете готовы, вызовите API [експортитератион](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Training_3.3/operations/5eb0bcc6548b571998fddece) , чтобы экспортировать обученную модель в указанную платформу.

В назначенном контейнере хранилища появится резервная копия экспортированной модели. Имя большого двоичного объекта будет иметь формат:

```
{projectId} - {iterationId}.{platformType}
```

Кроме того, при завершении экспорта вы получите уведомление в своей очереди. 

```json
{
"version": "1.0" ,
"type": "Export",
"Content":
    {
    "projectId": "00000000-0000-0000-0000-000000000000",
    "iterationId": "00000000-0000-0000-0000-000000000000",
    "exportStatus": "ExportCompleted",
    "modelUri": {url}
    }
}
```

`"exportStatus"`Поле может иметь значение `"ExportCompleted"` или `"ExportFailed"` . `"modelUri"`Поле будет содержать URL-адрес модели резервного копирования, хранящейся в контейнере, при условии, что в начале были интегрированы уведомления об очередях. Если это не так, в `"modelUri"` поле отобразится URL-адрес SAS для пользовательское визуальное распознавание модели BLOB.

## <a name="next-steps"></a>Дальнейшие действия

В этом руководство вы узнали, как копировать и перемещать проект между Пользовательское визуальное распознаваниеными ресурсами. Затем изучите документацию по API, чтобы узнать, что еще можно сделать с помощью Пользовательское визуальное распознавание.
* [Справочная документация по REST API (обучение)](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Training_3.3/operations/5eb0bcc6548b571998fddeb3)
* [Справочная документация по REST API (прогноз)](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Prediction_3.1/operations/5eb37d24548b571998fde5f3)