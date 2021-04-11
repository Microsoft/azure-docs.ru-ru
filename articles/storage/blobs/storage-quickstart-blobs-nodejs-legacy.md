---
title: Краткое руководство. Использование клиентской библиотеки хранилища BLOB-объектов Azure версии 10 для JavaScript
description: Создание, передача и удаление больших двоичных объектов и контейнеров в Node.js с помощью клиентской библиотеки службы хранилища Azure версии 10 для JavaScript.
author: twooley
ms.author: twooley
ms.date: 01/19/2021
ms.service: storage
ms.subservice: blobs
ms.topic: quickstart
ms.custom: devx-track-js
ms.openlocfilehash: ddcce3e624de7e8bed62783bdc22c5c4211e19e2
ms.sourcegitcommit: 02bc06155692213ef031f049f5dcf4c418e9f509
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/03/2021
ms.locfileid: "106280111"
---
# <a name="quickstart-manage-blobs-with-javascript-v10-sdk-in-nodejs"></a>Краткое руководство. Управление большими двоичными объектами с помощью пакета SDK для JavaScript версии 10 в Node.js

Из этого краткого руководства вы узнаете, как управлять большими двоичными объектами с использованием Node.js. Большие двоичные объекты — это объекты, которые могут содержать большие объемы текстовых или двоичных данных, включая изображения, документы, потоковое мультимедиа и архивные данные. Вы научитесь отправлять, скачивать, удалять большие двоичные объекты и получать список таких объектов, а также управлять контейнерами.

> [!NOTE]
> В этом кратком руководстве используется устаревшая версия клиентской библиотеки Хранилища BLOB-объектов Azure. Сведения о том, как начать работу с последней версией, см. в статье [Краткое руководство. Управление большими двоичными объектами с помощью пакета SDK для JavaScript версии 12 в Node.js](storage-quickstart-blobs-nodejs.md)

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) бесплатно.
- Учетная запись хранения Azure. [Создание учетной записи хранения](../common/storage-account-create.md).
- [Node.js](https://nodejs.org/en/download/).

## <a name="download-the-sample-application"></a>Загрузка примера приложения

[Пример приложения](https://github.com/Azure-Samples/azure-storage-js-v10-quickstart.git), используемый в этом кратком руководстве, является простым консольным приложением Node.js. Чтобы начать работу, клонируйте репозиторий на компьютер, используя следующую команду.

```bash
git clone https://github.com/Azure-Samples/azure-storage-js-v10-quickstart.git
```

Затем измените папки для приложения.

```bash
cd azure-storage-js-v10-quickstart
```

Теперь откройте папку в избранной среде редактирования кода.

## <a name="configure-your-storage-credentials"></a>Настройка учетных данных хранилища

Перед запуском приложения укажите учетные данные безопасности для учетной записи хранения. Образец репозитория включает в себя файл с именем *.env.example*. Переименуйте этот файл, удалив расширение *.example*, после чего в названии файла будет расширение *.env*. Внутри файла *.env* добавьте имя своей учетной записи и значение ключа доступа после ключей *AZURE_STORAGE_ACCOUNT_NAME* и *AZURE_STORAGE_ACCOUNT_ACCESS_KEY*.

## <a name="install-required-packages"></a>Установка необходимых пакетов

В каталоге приложения выполните команду *npm install*, чтобы установить необходимые пакеты для приложения.

```bash
npm install
```

## <a name="run-the-sample"></a>Запуск примера

После установки зависимостей вы можете запустить пример, выполнив следующую команду.

```bash
npm start
```

Результат этого приложения будет аналогичен следующему примеру.

```bash
Container "demo" is created
Containers:
 - container-one
 - container-two
 - demo
Blob "quickstart.txt" is uploaded
Local file "./readme.md" is uploaded
Blobs in "demo" container:
 - quickstart.txt
 - readme-stream.md
 - readme.md
Blob downloaded blob content: "hello!"
Blob "quickstart.txt" is deleted
Container "demo" is deleted
Done
```

Если вы используете новую учетную запись хранения для работы с этим кратким руководством, вам может отображаться только контейнер *demo* с меткой *Контейнеры:* .

## <a name="understanding-the-code"></a>Основные сведения о коде

Пример начинается с импорта классов и функций из пространства имен хранилища BLOB-объектов Azure. Каждый из импортированных элементов рассматривается в контексте использования в примере.

```javascript
const {
    Aborter,
    BlobURL,
    BlockBlobURL,
    ContainerURL,
    ServiceURL,
    SharedKeyCredential,
    StorageURL,
    uploadStreamToBlockBlob
} = require('@azure/storage-blob');
```

Учетные данные считываются из переменных среды на основе соответствующего контекста.

```javascript
if (process.env.NODE_ENV !== 'production') {
    require('dotenv').config();
}
```

Если приложение запущено в локальной среде с целью отладки, модуль *dotenv* загружает переменные среды. Значения определяются в файле с именем *.env* и загружаются в текущий контекст выполнения. Конфигурация сервера предоставляет эти значения в рабочей среде, поэтому этот код запускается только тогда, когда сценарий *не* выполняется в "рабочей" среде.

Следующий блок модулей импортируется для поддержки интерфейса с помощью файловой системы.

```javascript
const fs = require('fs');
const path = require('path');
```

Цель этих модулей указана ниже: 

- *fs* — это собственный модуль Node.js, который используется для работы с файловой системой;

- *path* требуется, чтобы определить абсолютный путь к файлу, который используется для передачи файла в хранилище BLOB-объектов.

Затем значения переменных среды считываются и записываются в константах.

```javascript
const STORAGE_ACCOUNT_NAME = process.env.AZURE_STORAGE_ACCOUNT_NAME;
const ACCOUNT_ACCESS_KEY = process.env.AZURE_STORAGE_ACCOUNT_ACCESS_KEY;
```
Следующий набор констант помогает отобразить намерение вычислений размера файла во время операций передачи.

```javascript
const ONE_MEGABYTE = 1024 * 1024;
const FOUR_MEGABYTES = 4 * ONE_MEGABYTE;
```

Время ожидания запросов, отправляемых API, может истечь после указанного интервала. Класс[Aborter](/javascript/api/%40azure/storage-blob/aborter?view=azure-node-legacy&preserve-view=true) отвечает за управление тем, как истекает время ожидания запросов. Следующие константы в этом примере используются для определения времени ожидания.

```javascript
const ONE_MINUTE = 60 * 1000;
```

### <a name="calling-code"></a>Код вызова

Для поддержки синтаксиса *async/await* JavaScript весь код вызова упакован в функцию с именем *execute*. Затем *execute* вызывается и обрабатывается как обещание.

```javascript
async function execute() {
    // commands... 
}

execute().then(() => console.log("Done")).catch((e) => console.log(e));
```

Весь следующий код выполняется внутри функции "execute", где размещается комментарий `// commands...`.

Сперва объявляются соответствующие переменные для назначения имен, примера содержимого и указания локального файла для передачи в хранилище BLOB-объектов.

```javascript
const containerName = "demo";
const blobName = "quickstart.txt";
const content = "hello!";
const localFilePath = "./readme.md";
```

Учетные данные учетной записи используются, чтобы создать конвейер, который отвечает за управление отправками запросов в REST API. Конвейеры являются потокобезопасными и задают логику для политик повтора, ведения журналов, правил десериализации ответов HTTP и многое другое.

```javascript
const credentials = new SharedKeyCredential(STORAGE_ACCOUNT_NAME, ACCOUNT_ACCESS_KEY);
const pipeline = StorageURL.newPipeline(credentials);
const serviceURL = new ServiceURL(`https://${STORAGE_ACCOUNT_NAME}.blob.core.windows.net`, pipeline);
```

В этом блоке кода используются следующие классы.

- Класс [SharedKeyCredential](/javascript/api/%40azure/storage-blob/sharedkeycredential?view=azure-node-legacy&preserve-view=true) отвечает за создание программы-оболочки учетных данных учетной записи хранения для предоставления их в конвейере запросов.

- Класс [StorageURL](/javascript/api/%40azure/storage-blob/storageurl?view=azure-node-legacy&preserve-view=true) отвечает за создание нового конвейера.

- [ServiceURL](/javascript/api/%40azure/storage-blob/serviceurl?view=azure-node-legacy&preserve-view=true) моделирует URL-адрес, используемый в REST API. Экземпляры этого класса позволяют выполнять такие действия, как перечисление контейнеров и предоставление информации контекста для формирования URL-адресов контейнеров.

Экземпляр класса *ServiceURL* используется с экземплярами классов [ContainerURL](/javascript/api/%40azure/storage-blob/containerurl?view=azure-node-legacy&preserve-view=true) и [BlockBlobURL](/javascript/api/%40azure/storage-blob/blockbloburl?view=azure-node-legacy&preserve-view=true) для управления контейнерами и большими двоичными объектами в учетной записи хранения.

```javascript
const containerURL = ContainerURL.fromServiceURL(serviceURL, containerName);
const blockBlobURL = BlockBlobURL.fromContainerURL(containerURL, blobName);
```

Переменные *containerURL* и *blockBlobURL* используются повторно на протяжении всего примера, чтобы действовать в соответствии с учетной записью хранения. 

На этом этапе в учетной записи хранения контейнер не существует. Экземпляр класса *ContainerURL* представляет URL-адрес, на основании которого вы можете действовать. С помощью этого экземпляра можно создать и удалить контейнер. Расположение этого контейнера соответствует следующему расположению.

```bash
https://<ACCOUNT_NAME>.blob.core.windows.net/demo
```

Класс *blockBlobURL* используется для управления отдельными большими двоичными объектами, позволяя передавать, загружать и удалять содержимое больших двоичных объектов. Представленный здесь URL-адрес аналогичен следующему расположению.

```bash
https://<ACCOUNT_NAME>.blob.core.windows.net/demo/quickstart.txt
```

Как и в случае с контейнером, блочный BLOB-объект еще не существует. Переменная *blockBlobURL* будет использоваться позже для создания большого двоичного объекта путем передачи содержимого.

### <a name="using-the-aborter-class"></a>Применение класса Aborter

Время ожидания запросов, отправляемых API, может истечь после указанного интервала. Класс *Aborter* отвечает за управление тем, как истекает время ожидания запросов. Следующий код создает контекст, где у набора запросов есть 30 минут для выполнения.

```javascript
const aborter = Aborter.timeout(30 * ONE_MINUTE);
```

Классы Aborter позволяют контролировать запросы, предоставляя возможность:

- определять время для пакета запросов;
- определять, как долго должен выполняться отдельный запрос в пакете;
- отменять запросы;
- использовать статический элемент *Aborter.none* для остановки истечения времени ожидания всех запросов.

### <a name="create-a-container"></a>Создание контейнера

Чтобы создать контейнер, используется метод *create* класса *ContainerURL*.

```javascript
await containerURL.create(aborter);
console.log(`Container: "${containerName}" is created`);
```

Поскольку имя контейнера определено при вызове *ContainerURL.fromServiceURL (serviceURL, containerName)* , вызов метода *create* — это все, что требуется для создания контейнера.

### <a name="show-container-names"></a>Отображение имен контейнеров

Учетные записи могут хранить большое число контейнеров. Следующий код демонстрирует, как перечислять контейнеры сегментированным образом, что позволяет циклически просматривать большое количество контейнеров. Функция *showContainerNames* — это экземпляры классов *ServiceURL* и *Aborter*.

```javascript
console.log("Containers:");
await showContainerNames(serviceURL, aborter);
```

Функция *showContainerNames* использует метод *listContainersSegment* для запроса пакетов имен контейнеров из учетной записи хранения.

```javascript
async function showContainerNames(aborter, serviceURL) {
    let marker = undefined;

    do {
        const listContainersResponse = await serviceURL.listContainersSegment(aborter, marker);
        marker = listContainersResponse.nextMarker;
        for(let container of listContainersResponse.containerItems) {
            console.log(` - ${ container.name }`);
        }
    } while (marker);
}
```

Когда возвращается ответ, то *containerItems* осуществляет итерацию для записи имени на консоль. 

### <a name="upload-text"></a>Передача текста

Чтобы загрузить текст в большой двоичный объект, используйте метод *upload*.

```javascript
await blockBlobURL.upload(aborter, content, content.length);
console.log(`Blob "${blobName}" is uploaded`);
```

Здесь текст и его длина передаются в метод.

### <a name="upload-a-local-file"></a>Передача локального файла

Чтобы передать локальный файл в контейнер, требуется URL-адрес контейнера и путь к файлу.

```javascript
await uploadLocalFile(aborter, containerURL, localFilePath);
console.log(`Local file "${localFilePath}" is uploaded`);
```

Функция *uploadLocalFile* вызывает функцию *uploadFileToBlockBlob*, которая принимает путь к файлу и имя экземпляра назначения блочного BLOB-объекта в качестве аргументов.

```javascript
async function uploadLocalFile(aborter, containerURL, filePath) {

    filePath = path.resolve(filePath);

    const fileName = path.basename(filePath);
    const blockBlobURL = BlockBlobURL.fromContainerURL(containerURL, fileName);

    return await uploadFileToBlockBlob(aborter, filePath, blockBlobURL);
}
```

### <a name="upload-a-stream"></a>Отправка потока

Также поддерживается отправка потоков. Этот пример открывает локальный файл как поток для передачи в метод отправки.

```javascript
await uploadStream(containerURL, localFilePath, aborter);
console.log(`Local file "${localFilePath}" is uploaded as a stream`);
```

Функция *uploadStream* вызывает функцию *uploadStreamToBlockBlob* для передачи потока в контейнер хранилища.

```javascript
async function uploadStream(aborter, containerURL, filePath) {
    filePath = path.resolve(filePath);

    const fileName = path.basename(filePath).replace('.md', '-stream.md');
    const blockBlobURL = BlockBlobURL.fromContainerURL(containerURL, fileName);

    const stream = fs.createReadStream(filePath, {
      highWaterMark: FOUR_MEGABYTES,
    });

    const uploadOptions = {
        bufferSize: FOUR_MEGABYTES,
        maxBuffers: 5,
    };

    return await uploadStreamToBlockBlob(
                    aborter, 
                    stream, 
                    blockBlobURL, 
                    uploadOptions.bufferSize, 
                    uploadOptions.maxBuffers);
}
```

Во время передачи *uploadStreamToBlockBlob* выделяет буферы для кэширования данных из потока, в случае если будет необходима повторная попытка. Значение *maxBuffers* определяет максимальное количество буферов, поскольку каждый буфер создает отдельный запрос на загрузку. В идеальном случае буферы приравниваются к более высокой скорости, но за счет увеличения использования памяти. Скорость загрузки достигает пика, когда количество буферов довольно большое, и узкое место переходит в сеть или диск, а не в клиент.

### <a name="show-blob-names"></a>Отображение имен BLOB-объектов

Так же, как учетные записи могут содержать много контейнеров, каждый контейнер может потенциально содержать огромное количество больших двоичных объектов. Доступ к каждому большому двоичному объекту в контейнере возможен через экземпляр класса *ContainerURL*.

```javascript
console.log(`Blobs in "${containerName}" container:`);
await showBlobNames(aborter, containerURL);
```

Функция *showBlobNames* вызывает функцию *listBlobFlatSegment* для запроса пакетов больших двоичных объектов из контейнера.

```javascript
async function showBlobNames(aborter, containerURL) {
    let marker = undefined;

    do {
        const listBlobsResponse = await containerURL.listBlobFlatSegment(Aborter.none, marker);
        marker = listBlobsResponse.nextMarker;
        for (const blob of listBlobsResponse.segment.blobItems) {
            console.log(` - ${ blob.name }`);
        }
    } while (marker);
}
```

### <a name="download-a-blob"></a>Загрузка BLOB-объектов

После создания большого двоичного объекта можно загрузить содержимое с помощью метода *download*.

```javascript
const downloadResponse = await blockBlobURL.download(aborter, 0);
const downloadedContent = await streamToString(downloadResponse.readableStreamBody);
console.log(`Downloaded blob content: "${downloadedContent}"`);
```

Ответ возвратится в виде потока. В этом примере поток преобразуется в строку с помощью следующей вспомогательной функции *streamToString*.

```javascript
// A helper method used to read a Node.js readable stream into a string
async function streamToString(readableStream) {
    return new Promise((resolve, reject) => {
      const chunks = [];
      readableStream.on("data", data => {
        chunks.push(data.toString());
      });
      readableStream.on("end", () => {
        resolve(chunks.join(""));
      });
      readableStream.on("error", reject);
    });
}
```

### <a name="delete-a-blob"></a>Удаление большого двоичного объекта

Метод *delete* экземпляра класса *BlockBlobURL* удаляет большой двоичный объект из контейнера.

```javascript
await blockBlobURL.delete(aborter)
console.log(`Block blob "${blobName}" is deleted`);
```

### <a name="delete-a-container"></a>Удаление контейнера

Метод *delete* экземпляра класса *ContainerURL* удаляет контейнер из учетной записи хранения.

```javascript
await containerURL.delete(aborter);
console.log(`Container "${containerName}" is deleted`);
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Все данные, записанные в учетную запись хранения, автоматически удаляются в конце примера кода. 

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве показано, как управлять BLOB-объектами и контейнерами в хранилище BLOB-объектов Azure с помощью Node.js. Дополнительные сведения о работе с этим пакетом SDK см. в соответствующем репозитории GitHub.

> [!div class="nextstepaction"]
> [Репозиторий пакета SDK версии 10 службы хранилища Azure для JavaScript](https://github.com/Azure/azure-storage-js)
> [Справочник по API JavaScript для службы хранилища Azure](/javascript/api/overview/azure/storage-overview)
