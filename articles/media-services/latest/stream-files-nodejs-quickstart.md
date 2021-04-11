---
title: Как выполнять кодирование и потоковую передачу видеофайлов с помощью Node.JS
description: Как выполнять потоковую передачу видеофайлов с помощью Node.JS Выполните инструкции, приведенные в этом руководстве, чтобы создать учетную запись Служб мультимедиа Azure, закодировать файл и выполнить его потоковую передачу в Проигрыватель мультимедиа Azure.
services: media-services
author: IngridAtMicrosoft
manager: femila
editor: ''
keywords: службы мультимедиа azure, потоковая передача, Node.JS
ms.service: media-services
ms.workload: media
ms.topic: tutorial
ms.date: 02/17/2021
ms.author: inhenkel
ms.openlocfilehash: 5bb061af37f6f6d7e6e27cf25f0faa63bca7353c
ms.sourcegitcommit: 5fd1f72a96f4f343543072eadd7cdec52e86511e
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/01/2021
ms.locfileid: "106109192"
---
# <a name="how-to-encode-and-stream-video-files-with-nodejs"></a>Как выполнять кодирование и потоковую передачу видеофайлов с помощью Node.JS

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Это краткое руководство показывает, как можно легко кодировать и начинать потоковую передачу видео в разных браузерах и на разных устройствах с помощью Служб мультимедиа Azure. Входной видеофайл можно указывать с помощью URL-адресов протоколов HTTP, URL-адресов SAS или путей к файлам, находящимся в хранилище BLOB-объектов Azure.

Изучив это краткое руководство, вы будете знать:

- как выполнять кодирование с помощью Node.JS;
- как выполнять потоковую передачу с помощью Node.JS;
- как передавать файл из URL-адреса HTTPS с помощью Node.JS;
- как использовать клиентский проигрыватель HLS или DASH с помощью Node.JS.

Пример в этой статье предназначен для кодирования содержимого, которое доступно через URL-адрес HTTPS. Обратите внимание, что в настоящее время AMS версии 3 не поддерживает кодирование блочной передачи по URL-адресам HTTPS.

![Воспроизведение видео](./media/stream-files-nodejs-quickstart/final-video.png)

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Предварительные требования

- Установите [Node.js](https://nodejs.org/en/download/)
- [Создание учетной записи Служб мультимедиа](./create-account-howto.md).<br/>Запишите значения, которые вы использовали в качестве имени группы ресурсов и имени учетной записи Служб мультимедиа.
- Выполните действия, описанные в статье [Доступ к API Служб мультимедиа Azure с помощью Azure CLI](./access-api-howto.md), и сохраните учетные данные. Эти данные понадобятся для доступа к API.
- Сначала изучите практическое руководство по [выполнению подключения и настройки с помощью Node.js](./configure-connect-nodejs-howto.md), чтобы понять, как использовать клиентский пакет SDK для Node.js.

## <a name="download-and-configure-the-sample"></a>Скачивание и настройка примера

Клонируйте репозиторий GitHub, содержащий пример потоковой передачи данных Node.js, на компьютер с помощью следующей команды:  

 ```bash
 git clone https://github.com/Azure-Samples/media-services-v3-node-tutorials.git
 ```

Этот пример находится в папке [StreamFilesSample](https://github.com/Azure-Samples/media-services-v3-node-tutorials/tree/master/AMSv3Samples/StreamFilesSample).

Откройте файл [index.ts](https://github.com/Azure-Samples/media-services-v3-node-tutorials/blob/master/AMSv3Samples/StreamFilesSample/index.ts) в скачанном проекте. Обновите файл *sample.env* в корневой папке с помощью значений и учетных данных, которые вы получили, выполнив действия из руководства по получению [доступа к API](./access-api-howto.md). Переименуйте файл *sample.env* в *.env* (да, только расширение).

Этот пример выполняет следующие действия:

1. Создает **преобразование** с [предустановкой кодирования с учетом содержимого](./encode-content-aware-concept.md). Сначала он проверяет, существует ли указанное преобразование.
1. Создает выходной **ресурс**, который используется **заданием** кодирования для размещения входных данных.
1. При необходимости отправляет локальный файл с помощью пакета SDK хранилища BLOB-объектов.
1. Создает входные данные для **задания**, основанные на URL-адресе HTTPS или отправленном файле.
1. Отправляет **задание** кодирования с использованием входных и выходных данных, созданных ранее.
1. Проверяет состояние задания.
1. Загружает выходные данные задания кодирования в локальную папку.
1. Создает **указатель потоковой передачи** для использования в проигрывателе.
1. Компилирует URL-адреса потоковой передачи для HLS и DASH.
1. Воспроизводит содержимое в приложении проигрывателя (Проигрыватель мультимедиа Azure).

## <a name="run-the-sample"></a>Запуск примера

1. Приложение загружает закодированные файлы. Создайте папку для выходных файлов и измените значение переменной **outputFolder** в файле [index.js](https://github.com/Azure-Samples/media-services-v3-node-tutorials/blob/main/AMSv3Samples/StreamFilesSample/index.ts#L59). По умолчанию установлено значение Temp.
1. Откройте **командную строку** и перейдите в каталог с примером.
1. Перейдите в папку AMSv3Samples.

    ```bash
    cd AMSv3Samples
    ```

1. Установите пакеты, используемые в файле *packages.js*.

    ```bash
    npm install 
    ```

1. Перейдите в папку *StreamFilesSample*.

    ```bash
    cd StreamFilesSample
    ```

1. Запустите Visual Studio Code из папки *AMSv3Samples*. (Это необходимо для запуска из папки, в которой находятся папка *.vscode* и файлы *tsconfig.js*.)

    ```bash
    cd ..
    code .
    ```

Откройте папку *StreamFilesSample*, а затем откройте файл *index.ts* в редакторе Visual Studio Code.
В файле *index.ts* нажмите клавишу F5, чтобы запустить отладчик.

## <a name="test-with-azure-media-player"></a>Тестирование с помощью Проигрывателя мультимедиа Azure

Для тестирования потоковой передачи используйте Проигрыватель мультимедиа Azure. Вы также можете использовать любой проигрыватель, совместимый с HLS или DASH, например проигрыватель Shaka, HLS.js, Dash.js и другие.

Вы должны щелкнуть ссылку, созданную в примере, и запустить проигрыватель AMP с уже загруженным манифестом DASH.

> [!NOTE]
> Если проигрыватель размещен на сайте HTTPS, обновите URL-адрес до HTTPS.

1. Откройте браузер и перейдите по ссылке [https://aka.ms/azuremediaplayer/](https://aka.ms/azuremediaplayer/).
2. В поле **URL:** (URL-адрес:) вставьте одно из значений URL-адресов потоковой передачи, полученных при работе приложения. URL-адрес можно указать в формате HLS, Dash или Smooth, а Проигрыватель мультимедиа Azure автоматически выберет соответствующий протокол потоковой передачи для воспроизведения на устройстве.
3. Щелкните **Update Player** (Обновить проигрыватель).

Проигрыватель мультимедиа Azure можно использовать для тестирования, но его нельзя применять в рабочей среде.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вам больше не нужны ресурсы в группе ресурсов, включая Службы мультимедиа и учетные записи хранения, созданные в рамках этого руководства, удалите группу ресурсов.

Выполните следующую команду CLI:

```azurecli
az group delete --name amsResourceGroup
```

## <a name="more-developer-documentation-for-nodejs-on-azure"></a>Дополнительная документация для разработчиков по Node.js в Azure

- [Azure для разработчиков JavaScript и Node.js](/azure/developer/javascript/)
- [Исходный код Служб мультимедиа в репозитории GitHub @azure/azure-sdk-for-js](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/mediaservices/arm-mediaservices)
- [Документация по пакетам Azure для разработчиков Node.js](/javascript/api/overview/azure/)

## <a name="see-also"></a>См. также раздел

- [Коды ошибок задания](/rest/api/media/jobs/get#joberrorcode).
- [Установка с помощью npm @azure/arm-mediaservices](https://www.npmjs.com/package/@azure/arm-mediaservices)
- [Azure для разработчиков JavaScript и Node.js](/azure/developer/javascript/)
- [Исходный код Служб мультимедиа в репозитории @azure/azure-sdk-for-js](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/mediaservices/arm-mediaservices)

## <a name="next-steps"></a>Дальнейшие действия

> [Основные понятия служб мультимедиа Azure](concepts-overview.md)
