---
title: Потоковая передача видеофайлов с помощью Служб мультимедиа Azure (.NET)
description: Выполните инструкции, приведенные в этом учебнике, чтобы создать учетную запись Служб мультимедиа Azure, закодировать файл и выполнить его потоковую передачу в Проигрыватель мультимедиа Azure с помощью .NET.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
keywords: azure media services, stream
ms.service: media-services
ms.workload: media
ms.topic: tutorial
ms.custom: mvc
ms.date: 08/31/2020
ms.author: inhenkel
ms.openlocfilehash: e7c35e079a6f4bf6c40992debace60e10631a8ba
ms.sourcegitcommit: edc7dc50c4f5550d9776a4c42167a872032a4151
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105962056"
---
# <a name="tutorial-encode-a-remote-file-based-on-url-and-stream-the-video---net"></a>Руководство по кодированию удаленного файла на основе URL-адреса и потоковой передачи видео с помощью .NET

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Из этого руководства вы узнаете, как без труда кодировать видео и запускать его потоковую передачу в разных браузерах и на разных устройствах с помощью Служб мультимедиа Azure. Содержимое входных данных можно указывать с помощью URL-адресов протоколов HTTP, URL-адресов SAS или путей к файлам, находящимся в хранилище BLOB-объектов Azure.
Пример в этой статье предназначен для кодирования содержимого, которое доступно через URL-адрес HTTPS. Обратите внимание, что в настоящее время AMS версии 3 не поддерживает кодирование блочной передачи по URL-адресам HTTPS.

Изучив это руководство, вы сможете выполнить потоковую передачу видео.  

![Воспроизведение видео](./media/stream-files-dotnet-quickstart/final-video.png)

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Предварительные требования

- Вы можете скачать [Visual Studio Community 2017](https://www.visualstudio.com/thank-you-downloading-visual-studio/?sku=Community&rel=15) бесплатно, если у вас нет Visual Studio.
- [Создание учетной записи Служб мультимедиа](./account-create-how-to.md).<br/>Запишите значения, которые вы использовали в качестве имени группы ресурсов и имени учетной записи Служб мультимедиа.
- Выполните действия, описанные в статье [Доступ к API Служб мультимедиа Azure с помощью Azure CLI](./access-api-howto.md), и сохраните учетные данные. Эти данные понадобятся для доступа к API.

## <a name="download-and-configure-the-sample"></a>Скачивание и настройка примера

Клонируйте репозиторий GitHub, содержащий пример потоковой передачи данных .NET, на компьютер с помощью следующей команды:  

 ```bash
 git clone https://github.com/Azure-Samples/media-services-v3-dotnet-quickstarts.git
 ```

Этот пример находится в папке [EncodeAndStreamFiles](https://github.com/Azure-Samples/media-services-v3-dotnet-quickstarts/tree/master/AMSV3Quickstarts/EncodeAndStreamFiles).

Откройте файл [appsettings.json](https://github.com/Azure-Samples/media-services-v3-dotnet-quickstarts/blob/master/AMSV3Quickstarts/EncodeAndStreamFiles/appsettings.json) в скачанном проекте. Замените значения учетными данными, которые вы получили, выполнив действия из статьи [Доступ к API Служб мультимедиа Azure с помощью Azure CLI](./access-api-howto.md).

Этот пример выполняет следующие действия:

1. Создает **преобразование** (сначала проверяет, существует ли указанное преобразование). 
2. Создает выходной **ресурс**, который используется в качестве выходных данных **задания** кодирования.
3. Создает входные данные для **задания**, основанные на URL-адресе HTTPS.
4. Отправляет **задание** кодирования с использованием входных и выходных данных, созданных ранее.
5. проверка состояния задания;
6. Создает **указатель потоковой передачи**.
7. компиляция URL-адресов потоковой передачи.

Чтобы получить сведения о назначении всех функций в образце, ознакомьтесь с кодом и просмотрите комментарии в [этом исходном файле](https://github.com/Azure-Samples/media-services-v3-dotnet-quickstarts/blob/master/AMSV3Quickstarts/EncodeAndStreamFiles/Program.cs).

## <a name="run-the-sample-app"></a>Запуск примера приложения

После запуска приложения отображаются URL-адреса, которые можно использовать для воспроизведения видео с помощью различных протоколов. 

1. Нажмите клавиши CTRL+F5, чтобы выполнить приложение *EncodeAndStreamFiles*.
2. Выберите протокол **HLS** Apple (заканчивается на *manifest(format=m3u8-aapl)* ) и скопируйте URL-адрес потоковой передачи в консоли.

![Снимок экрана: выходные данные приложения EncodeAndStreamFiles в Visual Studio с отображением трех URL-адресов потоковой передачи для использования в Проигрывателе мультимедиа Azure.](./media/stream-files-tutorial-with-api/output.png)

В [исходном коде](https://github.com/Azure-Samples/media-services-v3-dotnet-quickstarts/blob/master/AMSV3Quickstarts/EncodeAndStreamFiles/Program.cs) образца вы увидите, каким образом создан URL-адрес. Чтобы создать его, необходимо сцепить имя узла конечной точки потоковой передачи и путь указателя потоковой передачи.  

## <a name="test-with-azure-media-player"></a>Тестирование с помощью Проигрывателя мультимедиа Azure

Для тестирования потоковой передачи в этой статье используется Проигрыватель мультимедиа Azure. 

> [!NOTE]
> Если проигрыватель размещен на сайте HTTPS, обновите URL-адрес до HTTPS.

1. Откройте браузер и перейдите по ссылке [https://aka.ms/azuremediaplayer/](https://aka.ms/azuremediaplayer/).
2. В поле **URL:** (URL-адрес:) вставьте одно из значений URL-адресов потоковой передачи, полученных при работе приложения. 
 
     URL-адрес можно указать в формате HLS, Dash или Smooth, а Проигрыватель мультимедиа Azure автоматически выберет соответствующий протокол потоковой передачи для воспроизведения на устройстве.
3. Щелкните **Update Player** (Обновить проигрыватель).

Проигрыватель мультимедиа Azure можно использовать для тестирования, но его нельзя применять в рабочей среде. 

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вам больше не нужны ресурсы в группе ресурсов, включая Службы мультимедиа и учетные записи хранения, созданные в рамках этого руководства, удалите группу ресурсов.

Выполните следующую команду CLI:

```azurecli
az group delete --name amsResourceGroup
```

## <a name="examine-the-code"></a>Анализ кода

Чтобы получить сведения о назначении всех функций в образце, ознакомьтесь с кодом и просмотрите комментарии в [этом исходном файле](https://github.com/Azure-Samples/media-services-v3-dotnet-quickstarts/blob/master/AMSV3Quickstarts/EncodeAndStreamFiles/Program.cs).

В руководстве по [передаче, кодированию и потоковой передаче файлов](stream-files-tutorial-with-api.md) представлен более сложный пример потоковой передачи и подробные пояснения. 

### <a name="job-error-codes"></a>Коды ошибок задания

См. статью о [кодах ошибок](/rest/api/media/jobs/get#joberrorcode).

## <a name="multithreading"></a>Многопоточность

Пакеты SDK для Служб мультимедиа Azure версии 3 не являются потокобезопасными. При работе с многопоточным приложением необходимо создать объект AzureMediaServicesClient для каждого потока.

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Руководство. Отправка, кодирование и потоковая передача видео с помощью API](stream-files-tutorial-with-api.md)
