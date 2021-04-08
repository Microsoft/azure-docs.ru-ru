---
title: Воспроизведение с помощью служб Azure записей, сделанных в течение нескольких дней
description: В этом учебнике описано, как использовать API служб мультимедиа Azure для воспроизведения непрерывной записи, сделанной в течение нескольких дней.
ms.topic: tutorial
ms.date: 05/27/2020
ms.openlocfilehash: 81a778b40649c1318b3738a289f0db37fd35376a
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99492794"
---
# <a name="tutorial-playback-of-multi-day-recordings"></a>Руководство по воспроизведению записей, сделанных в течение нескольких дней  

Этот учебник выступает как продолжение учебника [Continuous video recording](continuous-video-recording-concept.md) (Непрерывная запись видео). В этом учебнике описано, как использовать API служб мультимедиа Azure для воспроизведения из облака непрерывной записи, сделанной в течение нескольких дней. 

Один из сценариев использования — обеспечение безопасности населения, при котором требуется хранить записи с видеокамер в течение нескольких дней или недель и иногда необходимо пересматривать определенные отрывки этих записей.

В этом руководстве описаны следующие процедуры.

> [!div class="checklist"]
> * Запуск примера приложения, который демонстрирует, как можно просматривать содержимое записи длительностью в несколько дней.
> * Просмотр определенных частей записи.

## <a name="suggested-pre-reading"></a>Рекомендуемые материалы для предварительного ознакомления  

Рекомендуем вам ознакомиться с перечисленными ниже справочными статьями.

* [Аналитика видеотрансляций в IoT Edge: обзор](overview.md)
* [Аналитика видеотрансляций в IoT Edge: терминология](terminology.md)
* [Понятие графа мультимедиа](media-graph-concept.md)
* [Непрерывная запись видео](continuous-video-recording-concept.md) 
* [Руководство. Воспроизведение записей](playback-recordings-how-to.md)
* [Руководство. Непрерывная запись видео](continuous-video-recording-tutorial.md)

## <a name="prerequisites"></a>Предварительные требования

* Прочитайте учебник по [непрерывной записи видео](continuous-video-recording-tutorial.md) и выполните содержащиеся в нем указания. Хотя этот учебник и API, описанные в [учебнике по непрерывной записи видео](continuous-video-recording-tutorial.md), предназначены для записей длительностью от 5 минут, рекомендуется записать видео длительностью не менее 5 часов. Использование API для навигации по записям проще всего продемонстрировать на примере длинных записей.
* Рекомендуется выполнять указания в этом учебнике одновременно с указаниями [учебника по непрерывной записи видео](continuous-video-recording-tutorial.md): так, чтобы продолжалась запись видео в облако.

## <a name="run-the-sample"></a>Запуск примера 

В рамках работы с [учебником по непрерывной записи видео](continuous-video-recording-tutorial.md) вы создали учетную запись службы мультимедиа Azure. Для работы с этим учебником вам потребуется полный доступ к API для этой учетной записи. Чтобы создать субъект-службу, вы можете выполнить действия, описанные в разделе [Получение учетных данных для доступа к API служб мультимедиа](../latest/access-api-howto.md?tabs=portal). Вы получите от портала Microsoft Azure блок JSON, который выглядит следующим образом:

```
{
    "AadClientId": "<<INSERT_AZURE_AD_APP_ID_HERE>>",
    "AadSecret": "<<INSERT_AZURE_AD_APP_SECRET_HERE>>",
    "AadTenantDomain": "<<YOUR_TENANT_DOMAIN>>",
    "AadTenantId": "<<YOUR_TENANT_ID>>",
    "AccountName": "<<YOUR_ACCOUNT_NAME>>",
    "Location": "<<YOUR_REGION_NAME>>",
    "ResourceGroup": "<<YOUR_RESOURCE_GROUP>>",
    "SubscriptionId": "<<YOUR_SUBSCRIPTION_ID>>",
    "ArmAadAudience": "https://management.core.windows.net",
    "ArmEndpoint": "https://management.azure.com"
}
```

Затем в Visual Studio Code откройте src/ams-asset-player. Эта папка содержит файлы, необходимые для работы с этим учебником. Откройте файл appsettings.json и скопируйте его содержимое в новый файл под названием appsettings.development.json. Внесите следующие изменения в только что созданный файл appsettings.development.json:

```
  "AMS" : {
    "subscriptionId" : "Use value of SubscriptionId above",
    "resourceGroup" : "Use value of ResourceGroup above",
    "accountId" : "Use value of AccountName above",
    "aadTenantId" : "Use value of AadTenantId above",
    "aadClientId" : "Use value of AadClientId above",
    "aadSecret" : "Use value of AadSecret above"
} 
```

1. В Visual Studio Code откройте вкладку **Расширения** (или нажмите клавиши CTRL+SHIFT+X) и найдите центр Интернета вещей Azure.
1. Щелкните правой кнопкой мыши и выберите **Параметры расширения**.

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/run-program/extensions-tab.png" alt-text="Параметры расширения":::
1. Найдите и включите параметр "Show Verbose Message" (Показывать подробное сообщение).

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/run-program/show-verbose-message.png" alt-text="Show Verbose Message"::: (Показывать подробное сообщение)
1. <!--In Visual Studio Code, you can click-->Щелкните значок "Выполнить" слева или нажмите клавиши CTRL+SHIFT+D, чтобы открыть приложения, которые можно запустить.

    ![Снимок экрана: меню в Visual Studio Code с выбранным элементом выполнения.](./media/playback-multi-day-recordings-tutorial/run.png)
1. Выберите приложение AMS Asset Player из раскрывающегося списка, как показано ниже, и нажмите клавишу F5, чтобы начать отладку.

    ![Снимок экрана: меню в Visual Studio Code с выбранным AMS Asset Player.](./media/playback-multi-day-recordings-tutorial/debug.png)

Пример приложения скомпилируется и запустит браузер по умолчанию, в котором автоматически откроется страница AMS Asset Player.

> [!NOTE]
> В зависимости от настроек безопасности браузера может появиться предупреждение. Так как веб-страница открыта локально, его можно игнорировать.

AMS Asset Player предложит ввести имя ресурса службы мультимедиа. Укажите имя ресурса, который вы использовали для записи видео при выполнении заданий [учебника по непрерывной записи видео](continuous-video-recording-tutorial.md).

После того, как вы введете имя ресурса и нажмете "Отправить", код проигрывателя загрузит URL-адрес потоковой передачи. Дополнительные сведения см. в [руководстве по воспроизведению записей](playback-recordings-how-to.md). Если вы следуете рекомендациям и продолжаете запись в ресурс, проигрыватель обнаружит это и предпримет попытку начать воспроизведение с последнего момента записи. Метка времени (в формате UTC) отображается в левом верхнем углу проигрывателя. Обратите внимание на то, как выбрана кнопка Live на снимке экрана ниже.

![STREAM](./media/playback-multi-day-recordings-tutorial/assetplayer1.png)
 
В правой части проигрывателя находятся элементы управления для просмотра архива. Годы, месяцы и даты в этом элементе управления заполняются с помощью API availableMedia, описанного в [руководстве по воспроизведению записей](playback-recordings-how-to.md).
Если запись видео длится несколько часов, то, если вы развернете день, вы увидите следующее:

![Обзор архива](./media/playback-multi-day-recordings-tutorial/results.png)

Источником видеоканала в учебнике является файл MKV. Когда симулятор RSTP (см. [Live555 Media Server](https://github.com/Azure/live-video-analytics/tree/master/utilities/rtspsim-live555)) достигает конца файла, он завершает потоковую передачу. Исходный узел RTSP в графе мультимедиа обнаруживает это и повторно устанавливает подключение. Воспроизведение видео возобновляется. Промежутки времени между завершением передачи и повторным подключением отображаются как новые записи в результатах вызова availableMedia.

![Результаты](./media/playback-multi-day-recordings-tutorial/assetplayer2.png)
 
Если щелкнуть любую из записей в списке, приложение создаст URL-адрес потоковой передачи с соответствующим фильтром, например https://{hostname}/{locatorId}/content.ism/manifest(format=mpd-time-csf,startTime=YYYY-MM-DDTHH:MM:SS). Проигрыватель загрузит этот URL-адрес, и воспроизведение начнется с момента, указанного в startTime.

Также можно указать определенные значения времени начала и окончания с помощью элементов управления в нижней части страницы. При этом вы можете ориентироваться на результаты вызова availableMedia, перечисленные в правой части страницы. Подробные ограничения для фильтров startTime и endTime описаны в [инструкции по воспроизведению записей](playback-recordings-how-to.md). После выбора значений времени при нажатии кнопки "Отправить" приложение загрузит проигрыватель с URL-адресом потоковой передачи, например https://{hostname}/{locatorId}/content.ism/manifest(format=mpd-time-csf,startTime=YYYY-MM-DDTHH:MM:SS,endTime= YYYY-MM-DDTHH:MM:SS). Воспроизведение начнется с момента, указанного в startTime.

## <a name="next-steps"></a>Дальнейшие действия

[Запись видео в облако и его воспроизведение оттуда на основе событий](event-based-video-recording-tutorial.md)
