---
title: Краткое руководство Azure. Создание большого двоичного объекта в хранилище объектов с помощью PHP | Документация Майкрософт
description: Узнайте, как передавать объекты в Хранилище BLOB-объектов Azure и обратно с помощью PHP. Вы также сможете отправлять, скачивать и выводить блочные BLOB-объекты в контейнере в Хранилище BLOB-объектов Azure.
author: twooley
ms.author: twooley
ms.date: 11/14/2018
ms.service: storage
ms.subservice: blobs
ms.topic: quickstart
ms.openlocfilehash: 400982b1cacd6ccd409559679376edcfc886ed5c
ms.sourcegitcommit: 02bc06155692213ef031f049f5dcf4c418e9f509
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/03/2021
ms.locfileid: "106280060"
---
#  <a name="transfer-objects-tofrom-azure-blob-storage-using-php"></a>Передача объектов в хранилище BLOB-объектов Azure и обратно с помощью PHP
Из этого краткого руководства вы узнаете, как использовать PHP для передачи, скачивания и перечисления блочных BLOB-объектов в контейнере в хранилище BLOB-объектов Azure. 

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [storage-quickstart-prereq-include](../../../includes/storage-quickstart-prereq-include.md)]

Прежде чем приступить к работе, убедитесь, что у вас установлены следующие дополнительные компоненты:

* [PHP](https://php.net/downloads.php)
* [пакет SDK для Службы хранилища Azure для PHP](https://github.com/Azure/azure-storage-php).

## <a name="download-the-sample-application"></a>Загрузка примера приложения
В этом кратком руководстве как [пример](https://github.com/Azure-Samples/storage-blobs-php-quickstart.git) используется простое приложение PHP.  

Используйте команду [git](https://git-scm.com/), чтобы скачать копию приложения в среду разработки. 

```bash
git clone https://github.com/Azure-Samples/storage-blobs-php-quickstart.git
```

Эта команда клонирует репозиторий в локальную папку git. Чтобы открыть пример приложения PHP, найдите папку storage-blobs-php-quickstart и откройте файл phpqs.php.  

[!INCLUDE [storage-copy-account-key-portal](../../../includes/storage-copy-account-key-portal.md)]

## <a name="configure-your-storage-connection-string"></a>Настройка строки подключения хранилища
В приложении нужно указать имя и ключ учетной записи хранения, чтобы создать экземпляр **BlobRestProxy** для вашего приложения. Рекомендуем хранить эти идентификаторы в переменной среды на локальном компьютере, где выполняется приложение. Чтобы создать переменную среды, используйте один из следующих примеров в зависимости от операционной системы. Замените строки **youraccountname** и **youraccountkey** фактическими значениями имени и ключа учетной записи.

# <a name="linux"></a>[Linux](#tab/linux)

```bash
export ACCOUNT_NAME=<youraccountname>
export ACCOUNT_KEY=<youraccountkey>
```

# <a name="windows"></a>[Windows](#tab/windows)

```cmd
setx ACCOUNT_NAME=<youraccountname>
setx ACCOUNT_KEY=<youraccountkey>
```
---

## <a name="configure-your-environment"></a>Настройка среды
Перенесите папку из локальной папки git в каталог, который обслуживается сервером PHP. Затем откройте командную строку, перейдите в тот же каталог и введите: `php composer.phar install`.

## <a name="run-the-sample"></a>Запуск примера
Этот пример создает тестовый файл в папке " ". Пример программы отправляет тестовый файл в хранилище BLOB-объектов, выводит списокBLOB-объектов в контейнере и скачивает файл с новым именем. 

Запустите образец. Ниже представлен пример выходных данных, которые возвращаются при выполнении этого приложения.
  
```
Uploading BlockBlob: HelloWorld.txt
These are the blobs present in the container: HelloWorld.txt: https://myexamplesacct.blob.core.windows.net/blockblobsleqvxd/HelloWorld.txt

This is the content of the blob uploaded: Hello Azure!
```
Когда вы нажмете появившуюся кнопку, пример программы удалит контейнер хранилища и файлы. Прежде чем продолжить, проверьте наличие двух файлов в папке сервера. Вы можете открыть их и убедиться, что они идентичны.

Для просмотра файлов в хранилище BLOB-объектов можно также воспользоваться таким средством, как [обозреватель службы хранилища Azure](https://storageexplorer.com). Обозреватель службы хранилища Azure — это бесплатное кроссплатформенное средство для доступа к данным учетной записи хранения. 

После проверки файлов нажмите любую клавишу для завершения демонстрации и удаления тестовых файлов. Теперь, когда известны функциональные возможности примера, откройте файл example.rb, чтобы просмотреть код. 

## <a name="understand-the-sample-code"></a>Разбор примера кода

Разберем пример кода, чтобы понять, как он работает.

### <a name="get-references-to-the-storage-objects"></a>Получение ссылок на объекты хранилища
Сначала необходимо создать ссылки на объекты, используемые для доступа к хранилищу BLOB-объектов и управлению им. Эти объекты зависят друг от друга, и каждый из них используется следующим в списке объектом.

* Создайте экземпляр объекта **BlobRestProxy** службы хранилища Azure, чтобы настроить учетные данные для подключения. 
* Создайте объект **BlobService**, указывающий на службу BLOB-объектов в вашей учетной записи хранения. 
* Создайте объект **Container**, представляющий контейнер, к которому осуществляется доступ. Контейнеры используются для организации BLOB-объектов аналогично папкам для упорядочения файлов на компьютере.

Теперь, когда у вас есть объект контейнер **blobClient**, вы можете создать **блочный** BLOB-объект, который указывает на конкретный интересующий вас большой двоичный объект. После этого можно выполнить такие операции как, передача, загрузка и копирование.

> [!IMPORTANT]
> Имена контейнеров должны состоять из знаков нижнего регистра. Дополнительные сведения об именовании контейнеров и больших двоичных объектов см. в статье [Naming and Referencing Containers, Blobs, and Metadata](/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata) (Именование контейнеров, больших двоичных объектов и метаданных и ссылка на них).

В этом разделе вы настроите экземпляр клиента службы хранилища Azure, создадите экземпляр объекта службы BLOB-объектов и контейнер, а затем зададите для контейнера разрешения на общий доступ к большим двоичным объектам. Контейнер называется **quickstartblobs**. 

```PHP
    # Setup a specific instance of an Azure::Storage::Client
    $connectionString = "DefaultEndpointsProtocol=https;AccountName=".getenv('account_name').";AccountKey=".getenv('account_key');
    
    // Create blob client.
    $blobClient = BlobRestProxy::createBlobService($connectionString);
    
    # Create the BlobService that represents the Blob service for the storage account
    $createContainerOptions = new CreateContainerOptions();
    
    $createContainerOptions->setPublicAccess(PublicAccessType::CONTAINER_AND_BLOBS);
    
    // Set container metadata.
    $createContainerOptions->addMetaData("key1", "value1");
    $createContainerOptions->addMetaData("key2", "value2");

    $containerName = "blockblobs".generateRandomString();

    try    {
        // Create container.
        $blobClient->createContainer($containerName, $createContainerOptions);
```

### <a name="upload-blobs-to-the-container"></a>Отправка BLOB-объектов в контейнер

Хранилище BLOB-объектов поддерживает блочные, добавочные и страничные BLOB-объекты. Чаще всего используются блочные BLOB-объекты. Именно этот тип представлен в этом кратком руководстве.  

Чтобы отправить файл в BLOB-объект, получите полный путь к файлу, соединив имя каталога и имя файла на локальном диске. Затем вы можете передать файл по указанному пути, используя метод **createBlockBlob()** . 

В примере кода используется локальный файл, который передается в Azure. В коде файл хранится под именем **myfile**, а большой двоичный объект под именем **fileToUpload**. В приведенном ниже примере файл отправляется в контейнер с именем **quickstartblobs**.

```PHP
    $myfile = fopen("HelloWorld.txt", "w") or die("Unable to open file!");
    fclose($myfile);

    # Upload file as a block blob
    echo "Uploading BlockBlob: ".PHP_EOL;
    echo $fileToUpload;
    echo "<br />";
    
    $content = fopen($fileToUpload, "r");

    //Upload blob
    $blobClient->createBlockBlob($containerName, $fileToUpload, $content);
```

Чтобы выполнить частичное обновление содержимого блочного BLOB-объекта, используйте метод **createblocklist()** . Блочные BLOB-объекты могут иметь размер 4,7 ТБ и представлять собой любые объекты, начиная от электронных таблиц Excel до больших видеофайлов. Страничные BLOB-объекты в основном используются для файлов виртуального жесткого диска, применяемых для поддержки виртуальных машин IaaS. Добавочные BLOB-объекты используются для ведения журнала, например если требуется выполнить запись в файл и затем добавлять дополнительные сведения. Добавочные BLOB-объекты должны использоваться в модели с одним средством записи. Большинство объектов, находящихся в хранилище BLOB-объектов, представляют собой блочные BLOB-объекты.

### <a name="list-the-blobs-in-a-container"></a>Перечисление BLOB-объектов в контейнере

Список файлов в контейнере можно получить с помощью метода **listBlobs()** . Следующий код извлекает список BLOB-объектов и поочередно перебирает их, отображая имена найденных в контейнере BLOB-объектов.  

```PHP
    $listBlobsOptions = new ListBlobsOptions();
    $listBlobsOptions->setPrefix("HelloWorld");
    
    echo "These are the blobs present in the container: ";
    
    do{
        $result = $blobClient->listBlobs($containerName, $listBlobsOptions);
        foreach ($result->getBlobs() as $blob)
        {
            echo $blob->getName().": ".$blob->getUrl()."<br />";
        }
        
        $listBlobsOptions->setContinuationToken($result->getContinuationToken());
    } while($result->getContinuationToken());
```

### <a name="get-the-content-of-your-blobs"></a>Получение содержимого больших двоичных объектов

Получите содержимое больших двоичных объектов с помощью метода **getBlob()**. Следующий код отображает содержимое большого двоичного объекта, который мы передали в предыдущем разделе.

```PHP
    $blob = $blobClient->getBlob($containerName, $fileToUpload);
    fpassthru($blob->getContentStream());
```

### <a name="clean-up-resources"></a>Очистка ресурсов
Если вам больше не нужны большие двоичные объекты, отправленные при работе с этим руководством, удалите весь контейнер с помощью метода **deleteContainer()** . Если вам больше не нужны созданные файлы, удалите их с помощью метода **deleteBlob()** .

```PHP
    // Delete blob.
    echo "Deleting Blob".PHP_EOL;
    echo $fileToUpload;
    echo "<br />";
    $blobClient->deleteBlob($_GET["containerName"], $fileToUpload);

    // Delete container.
    echo "Deleting Container".PHP_EOL;
    echo $_GET["containerName"].PHP_EOL;
    echo "<br />";
    $blobClient->deleteContainer($_GET["containerName"]);

    //Deleting local file
    echo "Deleting file".PHP_EOL;
    echo "<br />";
    unlink($fileToUpload);   
```

## <a name="resources-for-developing-php-applications-with-blobs"></a>Ресурсы для разработки приложений PHP с большими двоичными объектами

Ознакомьтесь со следующими дополнительными ресурсами для разработки PHP с использованием хранилища BLOB-объектов:

- Просматривайте, скачивайте и устанавливайте [исходный код клиентской библиотеки PHP](https://github.com/Azure/azure-storage-php) для службы хранилища Azure в GitHub.
- Изучите [примеры для хранилища BLOB-объектов](https://azure.microsoft.com/resources/samples/?sort=0&service=storage&platform=php&term=blob), написанные с использованием клиентской библиотеки PHP.

## <a name="next-steps"></a>Дальнейшие действия
 
В этом кратком руководстве вы узнали, как передавать файлы между локальным диском и хранилищем BLOB-объектов Azure с помощью PHP. Чтобы узнать подробнее о работе с PHP, перейдите на сайт Центра разработчиков PHP.

> [!div class="nextstepaction"]
> [Центр разработчиков PHP](https://azure.microsoft.com/develop/php/)


Дополнительные сведения об обозревателе объектов и BLOB-объектах см. в статье [Управление ресурсами хранилища BLOB-объектов Azure с помощью обозревателя хранилищ](../../vs-azure-tools-storage-explorer-blobs.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json).