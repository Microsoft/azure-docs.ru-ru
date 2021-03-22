---
title: Руководство по созданию высокодоступного приложения с помощью хранилища BLOB-объектов
titleSuffix: Azure Storage
description: Использование хранилища, геоизбыточного между зонами (RA-GZRS) с доступом на чтение для обеспечения высокой доступности данных приложений.
services: storage
author: tamram
ms.service: storage
ms.topic: tutorial
ms.date: 02/18/2021
ms.author: tamram
ms.reviewer: artek
ms.custom: mvc, devx-track-python, devx-track-js, devx-track-csharp
ms.subservice: blobs
ms.openlocfilehash: 0d597f0742cfc43f1c7fb38568b2a2bbda352beb
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102049344"
---
# <a name="tutorial-build-a-highly-available-application-with-blob-storage"></a>Руководство по Создание высокодоступного приложения с помощью хранилища BLOB-объектов

Это руководство представляет первую часть цикла. В нем вы узнаете, как обеспечить высокую доступность данных приложений в Azure.

Когда вы выполните приведенные здесь инструкции, у вас будет консольное приложение, которое передает и извлекает большие двоичные объекты из учетной записи хранения [геоизбыточного между зонами хранилища с доступом на чтение](../common/storage-redundancy.md) (RA-GZRS).

Геоизбыточность между зонами означает, что служба хранилища Azure асинхронно реплицирует транзакции из основного региона в дополнительный, которые разделены несколькими сотнями километров. Репликация гарантирует, что данные в дополнительном регионе согласованы в конечном счете. Консольное приложение использует [шаблон автоматического выключения](/azure/architecture/patterns/circuit-breaker), чтобы определить, к какой конечной точке подключаться, выполняя автоматическое переключение между конечными точками в результате имитации сбоев и операций восстановления.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

В первой части цикла вы узнаете, как выполнять такие задачи:

> [!div class="checklist"]
> * Создание учетной записи хранения
> * Задание строки подключения
> * Запуск консольного приложения

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим руководством сделайте следующее:

# <a name="net-v12"></a>[.NET (версии 12)](#tab/dotnet)

Сейчас мы работаем над созданием фрагментов кода для версии 12.x клиентских библиотек службы хранилища Azure. Дополнительные сведения см. в [объявлении о клиентских библиотеках службы хранилища Azure версии 12](https://techcommunity.microsoft.com/t5/azure-storage/announcing-the-azure-storage-v12-client-libraries/ba-p/1482394).

# <a name="net-v11"></a>[.NET (версии 11)](#tab/dotnet11)

* Установите [Visual Studio 2019](https://www.visualstudio.com/downloads/) с рабочей нагрузкой **разработки в Azure**.

  ![Разработка Azure (в разделе Web & Cloud (Сеть и облако))](media/storage-create-geo-redundant-storage/workloads.png)

# <a name="python-v12"></a>[Python v12](#tab/python)

Сейчас мы работаем над созданием фрагментов кода для версии 12.x клиентских библиотек службы хранилища Azure. Дополнительные сведения см. в [объявлении о клиентских библиотеках службы хранилища Azure версии 12](https://techcommunity.microsoft.com/t5/azure-storage/announcing-the-azure-storage-v12-client-libraries/ba-p/1482394).

# <a name="python-v21"></a>[Python v2.1](#tab/python2)

* установите [Python](https://www.python.org/downloads/);
* Загрузите и установите [пакет SDK службы хранилища Azure для Python](https://github.com/Azure/azure-storage-python).

# <a name="nodejs-v12"></a>[Node.js v12](#tab/nodejs)

Сейчас мы работаем над созданием фрагментов кода для версии 12.x клиентских библиотек службы хранилища Azure. Дополнительные сведения см. в [объявлении о клиентских библиотеках службы хранилища Azure версии 12](https://techcommunity.microsoft.com/t5/azure-storage/announcing-the-azure-storage-v12-client-libraries/ba-p/1482394).

# <a name="nodejs-v11"></a>[Node.js v11](#tab/nodejs11)

* Установите [Node.js](https://nodejs.org).

---

## <a name="sign-in-to-the-azure-portal"></a>Вход на портал Azure

Войдите на [портал Azure](https://portal.azure.com/).

## <a name="create-a-storage-account"></a>Создание учетной записи хранения

Учетная запись хранения предоставляет уникальное пространство имен для хранения ваших объектов данных в службе хранилища Azure и доступа к ним.

Выполните следующие действия, чтобы создать учетную запись хранения геоизбыточного между зонами хранилища (RA-GZRS) с доступом на чтение.

1. Нажмите кнопку **Создать ресурс** на портале Azure.
2. Выберите **Учетная запись хранения — BLOB-объект, файл, таблица, очередь** на странице **Новый**.
4. Заполните форму учетной записи хранения следующей информацией, как показано в изображении ниже, и нажмите кнопку **Создать**:

   | Параметр       | Образец значения | Описание |
   | ------------ | ------------------ | ------------------------------------------------- |
   | **Подписка** | *Моя подписка* | Дополнительные сведения о подписках см. [здесь](https://account.azure.com/Subscriptions). |
   | **ResourceGroup** | *myResourceGroup* | Допустимые имена групп ресурсов см. в статье о [правилах и ограничениях именования](/azure/architecture/best-practices/resource-naming). |
   | **имя**; | *mystorageaccount* | Уникальное имя учетной записи хранения. |
   | **Расположение** | *Восточная часть США* | Выберите расположение. |
   | **Производительность** | *Standard Edition* | Производительность уровня "Стандартный" хорошо подходит для работы с примерами. |
   | **Account kind** (Тип учетной записи) | *StorageV2* | Мы рекомендуем использовать учетную запись хранения общего назначения версии 2. Дополнительные сведения об учетных записях хранения Azure см. в [этой обзорной статье](../common/storage-account-overview.md). |
   | **Репликация**| *Хранилище, геоизбыточное между зонами, с доступом на чтение (RA-GZRS)* | Основной регион является избыточным в пределах зоны и реплицируется в дополнительный регион, для которого включен доступ на чтение. |
   | **Уровень доступа**| *Горячий уровень хранилища* | Выберите горячий уровень для часто используемых данных. |

    ![создание учетной записи хранения](media/storage-create-geo-redundant-storage/createragrsstracct.png)

## <a name="download-the-sample"></a>Скачивание примера приложения

# <a name="net-v12"></a>[.NET (версии 12)](#tab/dotnet)

Сейчас мы работаем над созданием фрагментов кода для версии 12.x клиентских библиотек службы хранилища Azure. Дополнительные сведения см. в [объявлении о клиентских библиотеках службы хранилища Azure версии 12](https://techcommunity.microsoft.com/t5/azure-storage/announcing-the-azure-storage-v12-client-libraries/ba-p/1482394).

# <a name="net-v11"></a>[.NET (версии 11)](#tab/dotnet11)

[Загрузите пример проекта](https://github.com/Azure-Samples/storage-dotnet-circuit-breaker-pattern-ha-apps-using-ra-grs/archive/master.zip) и извлеките (распакуйте) файл storage-dotnet-circuit-breaker-pattern-ha-apps-using-ra-grs.zip. Вы также можете использовать команду [git](https://git-scm.com/), чтобы загрузить копию приложения в среду разработки. Пример проекта содержит консольное приложение.

```bash
git clone https://github.com/Azure-Samples/storage-dotnet-circuit-breaker-pattern-ha-apps-using-ra-grs.git
```

# <a name="python-v12"></a>[Python v12](#tab/python)

Сейчас мы работаем над созданием фрагментов кода для версии 12.x клиентских библиотек службы хранилища Azure. Дополнительные сведения см. в [объявлении о клиентских библиотеках службы хранилища Azure версии 12](https://techcommunity.microsoft.com/t5/azure-storage/announcing-the-azure-storage-v12-client-libraries/ba-p/1482394).

# <a name="python-v21"></a>[Python v2.1](#tab/python2)

[Загрузите пример проекта](https://github.com/Azure-Samples/storage-python-circuit-breaker-pattern-ha-apps-using-ra-grs/archive/master.zip) и извлеките (распакуйте) файл storage-python-circuit-breaker-pattern-ha-apps-using-ra-grs.zip. Вы также можете использовать команду [git](https://git-scm.com/), чтобы загрузить копию приложения в среду разработки. Пример проекта содержит основное приложение Python.

```bash
git clone https://github.com/Azure-Samples/storage-python-circuit-breaker-pattern-ha-apps-using-ra-grs.git
```

# <a name="nodejs-v12"></a>[Node.js v12](#tab/nodejs)

Сейчас мы работаем над созданием фрагментов кода для версии 12.x клиентских библиотек службы хранилища Azure. Дополнительные сведения см. в [объявлении о клиентских библиотеках службы хранилища Azure версии 12](https://techcommunity.microsoft.com/t5/azure-storage/announcing-the-azure-storage-v12-client-libraries/ba-p/1482394).

# <a name="nodejs-v11"></a>[Node.js v11](#tab/nodejs11)

[Скачайте пример проекта](https://github.com/Azure-Samples/storage-node-v10-ha-ra-grs) и распакуйте файл. Вы также можете использовать команду [git](https://git-scm.com/), чтобы загрузить копию приложения в среду разработки. Пример проекта содержит простое приложение Node.js.

```bash
git clone https://github.com/Azure-Samples/storage-node-v10-ha-ra-grs
```

---

## <a name="configure-the-sample"></a>Настройка примера

# <a name="net-v12"></a>[.NET (версии 12)](#tab/dotnet)

Сейчас мы работаем над созданием фрагментов кода для версии 12.x клиентских библиотек службы хранилища Azure. Дополнительные сведения см. в [объявлении о клиентских библиотеках службы хранилища Azure версии 12](https://techcommunity.microsoft.com/t5/azure-storage/announcing-the-azure-storage-v12-client-libraries/ba-p/1482394).

# <a name="net-v11"></a>[.NET (версии 11)](#tab/dotnet11)

В приложении необходимо указать строку подключения для учетной записи хранения. Вы можете хранить эту строку подключения в переменной среде на локальном компьютере, где выполняется приложение. Чтобы создать переменную среды, выполните один из приведенных ниже примеров в зависимости от операционной системы.

Войдите в свою учетную запись хранения на портале Azure. В меню учетной записи хранения в разделе **Параметры** выберите **Ключи доступа**. Скопируйте **строку подключения** из основного или вторичного ключа. Выполните одну из следующих команд в зависимости от операционной системы, заменив \<yourconnectionstring\> фактической строкой подключения. Эта команда сохраняет переменную среды на локальном компьютере. В Windows переменная среды доступна только после перезагрузки **командной строки** или используемой оболочки.

### <a name="linux"></a>Linux

```
export storageconnectionstring=<yourconnectionstring>
```

### <a name="windows"></a>Windows

```powershell
setx storageconnectionstring "<yourconnectionstring>"
```

# <a name="python-v12"></a>[Python v12](#tab/python)

Сейчас мы работаем над созданием фрагментов кода для версии 12.x клиентских библиотек службы хранилища Azure. Дополнительные сведения см. в [объявлении о клиентских библиотеках службы хранилища Azure версии 12](https://techcommunity.microsoft.com/t5/azure-storage/announcing-the-azure-storage-v12-client-libraries/ba-p/1482394).

# <a name="python-v21"></a>[Python v2.1](#tab/python2)

В приложении необходимо указать учетные данные учетной записи хранения. Вы можете хранить эту информацию в переменных среды на локальном компьютере, где выполняется приложение. Чтобы создать переменные среды, выполните один из приведенных ниже примеров в зависимости от операционной системы.

Войдите в свою учетную запись хранения на портале Azure. В меню учетной записи хранения в разделе **Параметры** выберите **Ключи доступа**. Вставьте значения для параметров **Имя учетной записи хранения** и **Ключ** в следующие команды, заменив заполнители \<youraccountname\> и \<youraccountkey\>. Эта команда сохраняет переменные среды на локальном компьютере. В Windows переменная среды доступна только после перезагрузки **командной строки** или используемой оболочки.

### <a name="linux"></a>Linux

```
export accountname=<youraccountname>
export accountkey=<youraccountkey>
```

### <a name="windows"></a>Windows

```powershell
setx accountname "<youraccountname>"
setx accountkey "<youraccountkey>"
```

# <a name="nodejs-v12"></a>[Node.js v12](#tab/nodejs)

Сейчас мы работаем над созданием фрагментов кода для версии 12.x клиентских библиотек службы хранилища Azure. Дополнительные сведения см. в [объявлении о клиентских библиотеках службы хранилища Azure версии 12](https://techcommunity.microsoft.com/t5/azure-storage/announcing-the-azure-storage-v12-client-libraries/ba-p/1482394).

# <a name="nodejs-v11"></a>[Node.js v11](#tab/nodejs11)

Чтобы запустить этот пример, необходимо добавить учетные данные учетной записи хранения в файл `.env.example`, а затем переименовать его на `.env`.

```
AZURE_STORAGE_ACCOUNT_NAME=<replace with your storage account name>
AZURE_STORAGE_ACCOUNT_ACCESS_KEY=<replace with your storage account access key>
```

Чтобы найти эту информацию на портале Azure, перейдите к учетной записи хранения и выберите **Ключи доступа** в разделе **Параметры**.

Установите необходимые зависимости. Для этого откройте командную строку, перейдите к папке с примерами и введите `npm install`.

---

## <a name="run-the-console-application"></a>Запуск консольного приложения

# <a name="net-v12"></a>[.NET (версии 12)](#tab/dotnet)

Сейчас мы работаем над созданием фрагментов кода для версии 12.x клиентских библиотек службы хранилища Azure. Дополнительные сведения см. в [объявлении о клиентских библиотеках службы хранилища Azure версии 12](https://techcommunity.microsoft.com/t5/azure-storage/announcing-the-azure-storage-v12-client-libraries/ba-p/1482394).

# <a name="net-v11"></a>[.NET (версии 11)](#tab/dotnet11)

В Visual Studio нажмите клавишу **F5** или щелкните **Запустить**, чтобы начать отладку приложения. Visual Studio автоматически восстанавливает отсутствующие пакеты NuGet, если это настроено. Дополнительные сведения см. в разделе [Обзор восстановления пакетов](/nuget/consume-packages/package-restore#package-restore-overview).

Окно консоли запускается, и приложение начинает работать. Приложение передает изображение **HelloWorld.png** из решения в учетную запись хранения. Приложение проверяет, было ли изображение реплицировано во вторичную конечную точку RA-GZRS. Затем оно начинает скачивать изображение (до 999 раз). Каждая операция чтения помечается буквой **P** или **S**. **P** представляет основную конечную точку, а **S** — вторичную конечную точку.

![Консольное приложение выполняется](media/storage-create-geo-redundant-storage/figure3.png)

В примере кода задача `RunCircuitBreakerAsync` в файле `Program.cs` используется для скачивания изображения из учетной записи хранения с помощью метода [DownloadToFileAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblob.downloadtofileasync). Перед скачиванием определяется [OperationContext](/dotnet/api/microsoft.azure.cosmos.table.operationcontext). Контекст операции определяет обработчики событий, которые срабатывают при успешном завершении скачивания или при сбое и повторной попытке.

# <a name="python-v12"></a>[Python v12](#tab/python)

Сейчас мы работаем над созданием фрагментов кода для версии 12.x клиентских библиотек службы хранилища Azure. Дополнительные сведения см. в [объявлении о клиентских библиотеках службы хранилища Azure версии 12](https://techcommunity.microsoft.com/t5/azure-storage/announcing-the-azure-storage-v12-client-libraries/ba-p/1482394).

# <a name="python-v21"></a>[Python v2.1](#tab/python2)

Чтобы запустить приложение в терминале или в командной строке, перейдите к каталогу **circuitbreaker.py**, а затем введите `python circuitbreaker.py`. Приложение передает изображение **HelloWorld.png** из решения в учетную запись хранения. Приложение проверяет, было ли изображение реплицировано во вторичную конечную точку RA-GZRS. Затем оно начинает скачивать изображение (до 999 раз). Каждая операция чтения помечается буквой **P** или **S**. **P** представляет основную конечную точку, а **S** — вторичную конечную точку.

![Консольное приложение выполняется](media/storage-create-geo-redundant-storage/figure3.png)

В примере кода метод `run_circuit_breaker` в файле `circuitbreaker.py` используется для скачивания изображения из учетной записи хранения с помощью метода [get_blob_to_path](/python/api/azure-storage-blob/azure.storage.blob.baseblobservice.baseblobservice#get-blob-to-path-container-name--blob-name--file-path--open-mode--wb---snapshot-none--start-range-none--end-range-none--validate-content-false--progress-callback-none--max-connections-2--lease-id-none--if-modified-since-none--if-unmodified-since-none--if-match-none--if-none-match-none--timeout-none-).

Для функции повтора объекта хранилища задана линейная политика повтора. Функция повтора определяет, следует ли повторять запрос, и указывает время ожидания (в секундах) перед повторным выполнением запроса. Задайте для **retry\_to\_secondary** значение true, если запрос следует повторять в случае сбоя исходного запроса к основной конечной точке. В примере приложения пользовательская политика повтора определяется в функции `retry_callback` объекта хранилища.

Перед скачиванием определяется функция [retry_callback](/python/api/azure-storage-common/azure.storage.common.storageclient.storageclient) и [response_callback](/python/api/azure-storage-common/azure.storage.common.storageclient.storageclient) для объекта службы. Эти функции определяют обработчики событий, которые срабатывают при успешном завершении скачивания или при сбое и повторной попытке.

# <a name="nodejs-v12"></a>[Node.js v12](#tab/nodejs)

Сейчас мы работаем над созданием фрагментов кода для версии 12.x клиентских библиотек службы хранилища Azure. Дополнительные сведения см. в [объявлении о клиентских библиотеках службы хранилища Azure версии 12](https://techcommunity.microsoft.com/t5/azure-storage/announcing-the-azure-storage-v12-client-libraries/ba-p/1482394).

# <a name="nodejs-v11"></a>[Node.js v11](#tab/nodejs11)

Чтобы выполнить пример, откройте командную строку, перейдите к папке с примерами и введите `node index.js`.

В этом примере создается контейнер в учетной записи хранения больших двоичных объектов, **HelloWorld.png** передается в контейнер. После этого выполняется повторная проверка того, реплицировались ли контейнер и образ в дополнительный регион. После репликации вам будет предложено ввести **D** или **Q** (после ENTER), чтобы скачать или выйти. Результат должен выглядеть примерно следующим образом:

```
Created container successfully: newcontainer1550799840726
Uploaded blob: HelloWorld.png
Checking to see if container and blob have replicated to secondary region.
[0] Container has not replicated to secondary region yet: newcontainer1550799840726 : ContainerNotFound
[1] Container has not replicated to secondary region yet: newcontainer1550799840726 : ContainerNotFound
...
[31] Container has not replicated to secondary region yet: newcontainer1550799840726 : ContainerNotFound
[32] Container found, but blob has not replicated to secondary region yet.
...
[67] Container found, but blob has not replicated to secondary region yet.
[68] Blob has replicated to secondary region.
Ready for blob download. Enter (D) to download or (Q) to quit, followed by ENTER.
> D
Attempting to download blob...
Blob downloaded from primary endpoint.
> Q
Exiting...
Deleted container newcontainer1550799840726
```

---

## <a name="understand-the-sample-code"></a>Разбор примера кода

# <a name="net-v12"></a>[.NET (версии 12)](#tab/dotnet)

Сейчас мы работаем над созданием фрагментов кода для версии 12.x клиентских библиотек службы хранилища Azure. Дополнительные сведения см. в [объявлении о клиентских библиотеках службы хранилища Azure версии 12](https://techcommunity.microsoft.com/t5/azure-storage/announcing-the-azure-storage-v12-client-libraries/ba-p/1482394).

# <a name="net-v11"></a>[.NET (версии 11)](#tab/dotnet11)

### <a name="retry-event-handler"></a>Обработчик события при повторном запуске

Обработчик событий `OperationContextRetrying` вызывается, когда скачивание завершается с ошибкой и настроено на повторный запуск. Если достигнуто максимальное количество попыток, определенных для приложения, значение параметра [LocationMode](/dotnet/api/microsoft.azure.storage.blob.blobrequestoptions.locationmode) запроса изменяется на `SecondaryOnly`. В таком случае приложение принудительно повторно пытается скачать изображение из вторичной конечной точки. Эта конфигурация уменьшает время, затрачиваемое на запрос изображения, так как основная конечная точка не запрашивается бесконечно.

```csharp
private static void OperationContextRetrying(object sender, RequestEventArgs e)
{
    retryCount++;
    Console.WriteLine("Retrying event because of failure reading the primary. RetryCount = " + retryCount);

    // Check if we have had more than n retries in which case switch to secondary.
    if (retryCount >= retryThreshold)
    {

        // Check to see if we can fail over to secondary.
        if (blobClient.DefaultRequestOptions.LocationMode != LocationMode.SecondaryOnly)
        {
            blobClient.DefaultRequestOptions.LocationMode = LocationMode.SecondaryOnly;
            retryCount = 0;
        }
        else
        {
            throw new ApplicationException("Both primary and secondary are unreachable. Check your application's network connection. ");
        }
    }
}
```

### <a name="request-completed-event-handler"></a>Обработчик события при успешном выполнении запроса

Обработчик события `OperationContextRequestCompleted` вызывается при успешном выполнении запроса. Если приложение использует вторичную конечную точку, приложение запрашивает эту конечную точку до 20 раз. После 20 раза приложение обратно устанавливает для параметра [LocationMode](/dotnet/api/microsoft.azure.storage.blob.blobrequestoptions.locationmode) значение `PrimaryThenSecondary` и повторно запрашивает основную конечную точку. При успешном выполнении запроса приложение продолжает выполнять чтение из основной конечной точки.

```csharp
private static void OperationContextRequestCompleted(object sender, RequestEventArgs e)
{
    if (blobClient.DefaultRequestOptions.LocationMode == LocationMode.SecondaryOnly)
    {
        // You're reading the secondary. Let it read the secondary [secondaryThreshold] times,
        //    then switch back to the primary and see if it's available now.
        secondaryReadCount++;
        if (secondaryReadCount >= secondaryThreshold)
        {
            blobClient.DefaultRequestOptions.LocationMode = LocationMode.PrimaryThenSecondary;
            secondaryReadCount = 0;
        }
    }
}
```

# <a name="python-v12"></a>[Python v12](#tab/python)

Сейчас мы работаем над созданием фрагментов кода для версии 12.x клиентских библиотек службы хранилища Azure. Дополнительные сведения см. в [объявлении о клиентских библиотеках службы хранилища Azure версии 12](https://techcommunity.microsoft.com/t5/azure-storage/announcing-the-azure-storage-v12-client-libraries/ba-p/1482394).

# <a name="python-v21"></a>[Python v2.1](#tab/python2)

### <a name="retry-event-handler"></a>Обработчик события при повторном запуске

Обработчик событий `retry_callback` вызывается, когда скачивание завершается с ошибкой и настроено на повторный запуск. Если достигнуто максимальное количество попыток, определенных для приложения, значение параметра [LocationMode](/python/api/azure-storage-common/azure.storage.common.models.locationmode) запроса изменяется на `SECONDARY`. В таком случае приложение принудительно повторно пытается скачать изображение из вторичной конечной точки. Эта конфигурация уменьшает время, затрачиваемое на запрос изображения, так как основная конечная точка не запрашивается бесконечно.

```python
def retry_callback(retry_context):
    global retry_count
    retry_count = retry_context.count
    sys.stdout.write(
        "\nRetrying event because of failure reading the primary. RetryCount= {0}".format(retry_count))
    sys.stdout.flush()

    # Check if we have more than n-retries in which case switch to secondary
    if retry_count >= retry_threshold:

        # Check to see if we can fail over to secondary.
        if blob_client.location_mode != LocationMode.SECONDARY:
            blob_client.location_mode = LocationMode.SECONDARY
            retry_count = 0
        else:
            raise Exception("Both primary and secondary are unreachable. "
                            "Check your application's network connection.")
```

### <a name="request-completed-event-handler"></a>Обработчик события при успешном выполнении запроса

Обработчик события `response_callback` вызывается при успешном выполнении запроса. Если приложение использует вторичную конечную точку, приложение запрашивает эту конечную точку до 20 раз. После 20 раза приложение обратно устанавливает для параметра [LocationMode](/python/api/azure-storage-common/azure.storage.common.models.locationmode) значение `PRIMARY` и повторно запрашивает основную конечную точку. При успешном выполнении запроса приложение продолжает выполнять чтение из основной конечной точки.

```python
def response_callback(response):
    global secondary_read_count
    if blob_client.location_mode == LocationMode.SECONDARY:

        # You're reading the secondary. Let it read the secondary [secondaryThreshold] times,
        # then switch back to the primary and see if it is available now.
        secondary_read_count += 1
        if secondary_read_count >= secondary_threshold:
            blob_client.location_mode = LocationMode.PRIMARY
            secondary_read_count = 0
```

# <a name="nodejs-v12"></a>[Node.js v12](#tab/nodejs)

Сейчас мы работаем над созданием фрагментов кода для версии 12.x клиентских библиотек службы хранилища Azure. Дополнительные сведения см. в [объявлении о клиентских библиотеках службы хранилища Azure версии 12](https://techcommunity.microsoft.com/t5/azure-storage/announcing-the-azure-storage-v12-client-libraries/ba-p/1482394).

# <a name="nodejs-v11"></a>[Node.js v11](#tab/nodejs11)

При использовании пакета SDK для Node.js версии 10 обработчики обратных вызовов не нужны. Вместо этого в примере создается конвейер, настроенный с использованием параметров повторных попыток и вторичной конечной точки. Это позволяет приложению автоматически переключаться на дополнительный конвейер, если ему не удается получить доступ к данным через основной.

```javascript
const accountName = process.env.AZURE_STORAGE_ACCOUNT_NAME;
const storageAccessKey = process.env.AZURE_STORAGE_ACCOUNT_ACCESS_KEY;
const sharedKeyCredential = new SharedKeyCredential(accountName, storageAccessKey);

const primaryAccountURL = `https://${accountName}.blob.core.windows.net`;
const secondaryAccountURL = `https://${accountName}-secondary.blob.core.windows.net`;

const pipeline = StorageURL.newPipeline(sharedKeyCredential, {
  retryOptions: {
    maxTries: 3,
    tryTimeoutInMs: 10000,
    retryDelayInMs: 500,
    maxRetryDelayInMs: 1000,
    secondaryHost: secondaryAccountURL
  }
});
```

---

## <a name="next-steps"></a>Дальнейшие действия

В первой части этого цикла вы узнали о том, как сделать приложение доступным с помощью учетных записей хранения RA-GZRS.

Перейдите ко второй части серии, чтобы узнать, как имитировать сбой и заставить приложение использовать вторичную конечную точку RA-GZRS.

> [!div class="nextstepaction"]
> [Моделирование сбоя при чтении из основного региона](simulate-primary-region-failure.md)