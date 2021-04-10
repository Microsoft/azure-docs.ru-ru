---
title: Руководство. Отрисовка сцены в облаке
description: Сведения о том, как отрисовать сцену Autodesk 3ds Max с Arnold с помощью пакетной службы рендеринга и интерфейса командной строки Azure.
ms.topic: tutorial
ms.date: 12/30/2020
ms.custom: mvc, devx-track-azurecli
ROBOTS: NOINDEX
ms.openlocfilehash: 5165e5feb566a4b9081f40b681b92aafa143869f
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103491748"
---
# <a name="tutorial-render-a-scene-with-azure-batch"></a>Руководство по Отрисовка сцены с помощью пакетной службы Azure

Пакетная служба Azure предоставляет возможности рендеринга в масштабах облака с оплатой по мере использования. Пакетная служба рендеринга поддерживает такие приложения для рендеринга, как Autodesk Maya, 3ds Max, Arnold и V-Ray. В этом руководстве показаны шаги по отрисовке небольшой сцены с помощью пакетной службы и интерфейса командной строки Azure. Вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> - Отправка данных в хранилище Azure.
> - Создание пула пакетной службы для рендеринга.
> - Отрисовка однокадровой сцены
> - Масштабирование пула и отрисовка сцены с несколькими кадрами.
> - Скачивание выводимых данных.

В этом руководстве вы выполните рендеринг сцены 3ds Max с помощью пакетной службы Azure и отрисовщика трассировки лучей [Arnold](https://www.autodesk.com/products/arnold/overview). Пул пакетной службы использует образ Azure Marketplace с предварительно установленными графиками и визуализацией приложений, предоставляющие лицензирование с оплатой по мере использования.

## <a name="prerequisites"></a>Предварительные требования

- Вам потребуется подписка с оплатой по мере использования или другой вариант приобретения Azure для использования приложениями для рендеринга в пакетной службе Azure по принципу оплаты по мере использования. **Лицензирование с оплатой за использование не поддерживается, если использовать бесплатное предложение Azure, которое предоставляет денежной кредит.**

- Примеры сцены 3ds Max и скрипта Bash, а также файлы конфигурации JSON для этого руководства находятся на сайте [GitHub](https://github.com/Azure/azure-docs-cli-python-samples/tree/master/batch/render-scene). Сцена 3ds Max получена из [примера файлов Autodesk 3ds Max](https://download.autodesk.com/us/support/files/3dsmax_sample_files/2017/Autodesk_3ds_Max_2017_English_Win_Samples_Files.exe). (Примеры файлов Autodesk 3ds Max доступны в соответствии с лицензией Creative Commons Attribution-NonCommercial-Share Alike. Copyright &copy; Autodesk, Inc.)

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

- Для работы с этим учебником требуется Azure CLI версии 2.0.20 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

> [!TIP]
> Вы можете просмотреть [шаблоны заданий Arnold](https://github.com/Azure/batch-extension-templates/tree/master/templates/arnold/render-windows-frames) в репозитории шаблонов расширений для пакетной службы Azure на GitHub.

## <a name="create-a-batch-account"></a>Создание учетной записи Пакетной службы

Создайте в своей подписке группу ресурсов, учетную запись пакетной службы и связанную учетную запись хранения.

Создайте группу ресурсов с помощью команды [az group create](/cli/azure/group#az-group-create). В следующем примере создается группа ресурсов с именем *myResourceGroup* в расположении *eastus2*.

```azurecli-interactive
az group create \
    --name myResourceGroup \
    --location eastus2
```

В группе ресурсов создайте учетную запись службы хранилища Azure с помощью команды [az storage account create](/cli/azure/storage/account#az-storage-account-create). Для этого руководства вы используете учетную запись хранения, в которой будут находиться входная сцена 3ds Max и выводимые данные.

```azurecli-interactive
az storage account create \
    --resource-group myResourceGroup \
    --name mystorageaccount \
    --location eastus2 \
    --sku Standard_LRS
```

Создайте учетную запись пакетной службы с помощью команды [az batch account create](/cli/azure/batch/account#az-batch-account-create). В следующем примере создается учетная запись пакетной службы с именем *mybatchaccount* в группе ресурсов *myResourceGroup*. Она связывается с созданной учетной записью хранения.  

```azurecli-interactive
az batch account create \
    --name mybatchaccount \
    --storage-account mystorageaccount \
    --resource-group myResourceGroup \
    --location eastus2
```

Для создания пулов вычислений и заданий, а также управления ими необходимо выполнить проверку подлинности с помощью пакетной службы. Войдите в учетную запись с помощью команды [az batch account login](/cli/azure/batch/account#az-batch-account-login). После входа ваши команды `az batch` используют контекст учетной записи. В следующем примере используется проверка подлинности с общим ключом на основе имени и ключа учетной записи пакетной службы. Пакетная служба Azure также поддерживает проверку подлинности отдельных пользователей или приложений с помощью [Azure Active Directory](batch-aad-auth.md).

```azurecli-interactive
az batch account login \
    --name mybatchaccount \
    --resource-group myResourceGroup \
    --shared-key-auth
```

## <a name="upload-a-scene-to-storage"></a>Передача сцены в хранилище

Чтобы отправить входную сцену в хранилище, сначала нужно получить доступ к учетной записи хранения и создать контейнер назначения для больших двоичных объектов. Чтобы получить доступ к учетной записи хранения Azure, экспортируйте переменные среды `AZURE_STORAGE_KEY` и `AZURE_STORAGE_ACCOUNT`. Первая команда оболочки Bash получает ключ с помощью команды [az storage account keys list](/cli/azure/storage/account/keys#az-storage-account-keys-list). После установки переменных среды ваши команды хранилища используют контекст учетной записи.

```azurecli-interactive
export AZURE_STORAGE_KEY=$(az storage account keys list --account-name mystorageaccount --resource-group myResourceGroup -o tsv --query [0].value)

export AZURE_STORAGE_ACCOUNT=mystorageaccount
```

Теперь создайте контейнер больших двоичных объектов в учетной записи хранения для файлов сцены. В следующем примере используется команда [az storage container create](/cli/azure/storage/container#az-storage-container-create) для создания контейнера больших двоичных объектов с именем *scenefiles*, который разрешает доступ на чтение.

```azurecli-interactive
az storage container create \
    --public-access blob \
    --name scenefiles
```

Скачайте сцену `MotionBlur-Dragon-Flying.max` с сайта [GitHub](https://github.com/Azure/azure-docs-cli-python-samples/raw/master/batch/render-scene/MotionBlur-DragonFlying.max) в локальный рабочий каталог. Пример:

```azurecli-interactive
wget -O MotionBlur-DragonFlying.max https://github.com/Azure/azure-docs-cli-python-samples/raw/master/batch/render-scene/MotionBlur-DragonFlying.max
```

Передайте файл сцены из локального рабочего каталога в контейнер больших двоичных объектов. В следующем примере используется команда [az storage blob upload-batch](/cli/azure/storage/blob#az-storage-blob-upload-batch). С ее помощью можно передавать несколько файлов.

```azurecli-interactive
az storage blob upload-batch \
    --destination scenefiles \
    --source ./
```

## <a name="create-a-rendering-pool"></a>Создание пула рендеринга

Создайте пул пакетной службы для рендеринга с помощью команды [az batch pool create](/cli/azure/batch/pool#az-batch-pool-create). В этом примере вы указываете параметры пула в файле JSON. В текущей оболочке создайте файл *mypool.json* и скопируйте в него следующее содержимое. Убедитесь, что весь текст скопирован правильно. (Вы можете скачать файл с сайта [GitHub](https://raw.githubusercontent.com/Azure/azure-docs-cli-python-samples/master/batch/render-scene/json/mypool.json).)


```json
{
  "id": "myrenderpool",
  "vmSize": "standard_d2_v2",
  "virtualMachineConfiguration": {
    "imageReference": {
      "publisher": "batch",
      "offer": "rendering-windows2016",
      "sku": "rendering",
      "version": "1.3.8"
    },
    "nodeAgentSKUId": "batch.node.windows amd64"
  },
  "targetDedicatedNodes": 0,
  "targetLowPriorityNodes": 1,
  "enableAutoScale": false,
  "applicationLicenses":[
         "3dsmax",
         "arnold"
      ],
  "enableInterNodeCommunication": false 
}
```

Пакетная служба Azure поддерживает выделенные узлы и узлы [с низким приоритетом](batch-low-pri-vms.md). Вы можете использовать их в своих пулах. Выделенные узлы зарезервированы для пула. Низкоприоритетные узлы предлагаются по сниженной цене с учетом избыточных ресурсов виртуальной машины в Azure. Эти узлы становятся недоступны, если в Azure недостаточно ресурсов.

Указанный пул содержит один низкоприоритетный узел с образом Windows Server с программным обеспечением для пакетной службы рендеринга. Этот пул лицензирован для отрисовки с помощью 3ds Max и Arnold. Позже вы измените размер пула для большего количества узлов.

Если вы еще не вошли в учетную запись пакетной службы, воспользуйтесь командой [az batch account login](/cli/azure/batch/account#az-batch-account-login). Затем создайте пул, передав JSON-файл команде `az batch pool create`:

```azurecli-interactive
az batch pool create \
    --json-file mypool.json
```

На подготовку пула может потребоваться несколько минут. Чтобы увидеть состояние пула, выполните команду [az batch pool show](/cli/azure/batch/pool#az-batch-pool-show). С помощью следующей команды можно получить сведения о состоянии распределения пула:

```azurecli-interactive
az batch pool show \
    --pool-id myrenderpool \
    --query "allocationState"
```

Для создания задания и задач при изменении состояния пула выполните следующие шаги. Пул полностью подготовлен, если состояние распределения — `steady`, а узлы работают.  

## <a name="create-a-blob-container-for-output"></a>Создание контейнера больших двоичных объектов для выходных данных

В примерах этого руководства каждая задача в задании рендеринга создает выходной файл. Перед планированием задания создайте в учетной записи хранения контейнер больших двоичных объектов в качестве места назначения для выходных файлов. В следующем примере с помощью команды [az storage container create](/cli/azure/storage/container#az-storage-container-create) создается контейнер *job-myrenderjob* с открытым доступом на чтение.

```azurecli-interactive
az storage container create \
    --public-access blob \
    --name job-myrenderjob
```

Для записи выходных файлов в контейнер пакетной службе требуется токен подписанного URL-адреса (SAS). Создайте токен с помощью команды [az storage account-sas](/cli/azure/storage/account#az-storage-account-generate-sas). В этом примере создается токен для записи в любой контейнер больших двоичных объектов в учетной записи. Срок действия токена истекает 15 ноября 2021 года:

```azurecli-interactive
az storage account generate-sas \
    --permissions w \
    --resource-types co \
    --services b \
    --expiry 2021-11-15
```

Обратите внимание на токен, возвращенный следующей командой: Он понадобится вам позже.

`se=2021-11-15&sp=rw&sv=2019-09-24&ss=b&srt=co&sig=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`

## <a name="render-a-single-frame-scene"></a>Отрисовка однокадровой сцены

### <a name="create-a-job"></a>Создание задания

Создайте задание рендеринга для запуска в пуле с помощью команды [az batch job create](/cli/azure/batch/job#az-batch-job-create). Изначально у задания нет задач.

```azurecli-interactive
az batch job create \
    --id myrenderjob \
    --pool-id myrenderpool
```

### <a name="create-a-task"></a>Создание задачи

С помощью команды [az batch task create](/cli/azure/batch/task#az-batch-task-create) создайте задачи рендеринга в задании. В этом примере вы задаете параметры задачи в файле JSON. В текущей оболочке создайте файл *myrendertask.json* и скопируйте в него следующее содержимое. Убедитесь, что весь текст скопирован правильно. (Вы можете скачать файл с сайта [GitHub](https://raw.githubusercontent.com/Azure/azure-docs-cli-python-samples/master/batch/render-scene/json/myrendertask.json).)

Задача определяет команду 3ds Max для рендеринга сцены с одним кадром *MotionBlur-DragonFlying.max*.

Измените элементы `blobSource` и `containerURL` в файле JSON так, чтобы они включали имя вашей учетной записи хранения и токен SAS. 

> [!TIP]
> `containerURL` заканчивается токеном SAS и аналогичен следующему: `https://mystorageaccount.blob.core.windows.net/job-myrenderjob/$TaskOutput?se=2018-11-15&sp=rw&sv=2017-04-17&ss=b&srt=co&sig=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`

```json
{
  "id": "myrendertask",
  "commandLine": "cmd /c \"%3DSMAX_2018%3dsmaxcmdio.exe -secure off -v:5 -rfw:0 -start:1 -end:1 -outputName:\"dragon.jpg\" -w 400 -h 300 MotionBlur-DragonFlying.max\"",
  "resourceFiles": [
    {
        "httpUrl": "https://mystorageaccount.blob.core.windows.net/scenefiles/MotionBlur-DragonFlying.max",
        "filePath": "MotionBlur-DragonFlying.max"
    }
  ],
    "outputFiles": [
        {
            "filePattern": "dragon*.jpg",
            "destination": {
                "container": {
                    "containerUrl": "https://mystorageaccount.blob.core.windows.net/job-myrenderjob/myrendertask/$TaskOutput?Add_Your_SAS_Token_Here"
                }
            },
            "uploadOptions": {
                "uploadCondition": "TaskSuccess"
            }
        }
    ],
  "userIdentity": {
    "autoUser": {
      "scope": "task",
      "elevationLevel": "nonAdmin"
    }
  }
}
```

Добавьте задачу в задание с помощью следующей команды:

```azurecli-interactive
az batch task create \
    --job-id myrenderjob \
    --json-file myrendertask.json
```

Пакетная служба Azure планирует задачу. Она запускается, как только узел в пуле станет доступен.

### <a name="view-task-output"></a>Просмотр выходных данных задачи

На запуск задания может потребоваться несколько минут. С помощью команды [az batch task show](/cli/azure/batch/task#az-batch-task-show) можно просмотреть сведения о задаче.

```azurecli-interactive
az batch task show \
    --job-id myrenderjob \
    --task-id myrendertask
```

Задача создает файл *dragon0001.jpg* на вычислительном узле и загружает его в контейнер *job-myrenderjob* в вашей учетной записи хранения. Чтобы просмотреть выходные данные, загрузите файл из хранилища на локальный компьютер с помощью команды [az storage blob download](/cli/azure/storage/blob#az-storage-blob-download).

```azurecli-interactive
az storage blob download \
    --container-name job-myrenderjob \
    --file dragon.jpg \
    --name dragon0001.jpg

```

Откройте файл *dragon.jpg* на компьютере. Отрисованное изображение должно выглядеть примерно так:

![Отрисованный кадр с драконом 1](./media/tutorial-rendering-cli/dragon-frame.png) 

## <a name="scale-the-pool"></a>Масштабирование пула

Теперь измените пул, чтобы подготовиться к большему заданию отрисовки с несколькими кадрами. Пакетная служба Azure предоставляет несколько способов масштабирования вычислительных ресурсов, включая [автомасштабирование](batch-automatic-scaling.md). При этом узлы добавляются или удаляются по мере изменения требований к задаче. В этом примере с помощью команды [az batch pool resize](/cli/azure/batch/pool#az-batch-pool-resize) количество низкоприоритетных узлов в пуле увеличивается до *6*:

```azurecli-interactive
az batch pool resize --pool-id myrenderpool --target-dedicated-nodes 0 --target-low-priority-nodes 6
```

Для изменения размера пула может потребоваться несколько минут. Пока этот процесс выполняется, настройте следующие задачи для запуска в имеющемся задании рендеринга.

## <a name="render-a-multiframe-scene"></a>Отрисовка многокадровой сцены

Как и в примере с одним кадром, с помощью команды [az batch task create](/cli/azure/batch/task#az-batch-task-create) создайте задачи рендеринга в задании с именем *myrenderjob*. Задайте параметры задачи в JSON-файле с именем *myrendertask_multi.json*. (Вы можете скачать файл с сайта [GitHub](https://raw.githubusercontent.com/Azure/azure-docs-cli-python-samples/master/batch/render-scene/json/myrendertask_multi.json).) Каждая из шести задач указывает командную строку Arnold для рендеринга одного кадра сцены 3ds Max *MotionBlur-DragonFlying.max* .

Создайте файл в текущей оболочке с именем *myrendertask_multi.json* и скопируйте содержимое из скачанного файла. Измените элементы `blobSource` и `containerURL` в файле JSON, указав имя своей учетной записи хранения и токен SAS. Обязательно измените настройки для каждой из шести задач. Сохраните файл и создайте очередь задач, выполнив следующую команду:

```azurecli-interactive
az batch task create --job-id myrenderjob --json-file myrendertask_multi.json
```

### <a name="view-task-output"></a>Просмотр выходных данных задачи

На запуск задания может потребоваться несколько минут. Просмотреть состояние задач можно с помощью команды [az batch task list](/cli/azure/batch/task#az-batch-task-list). Пример:

```azurecli-interactive
az batch task list \
    --job-id myrenderjob \
    --output table
```

С помощью команды [az batch task show](/cli/azure/batch/task#az-batch-task-show) просмотрите сведения об отдельных задачах. Пример:

```azurecli-interactive
az batch task show \
    --job-id myrenderjob \
    --task-id mymultitask1
```

Задачи создают выходные файлы *dragon0002.jpg* - *dragon0007.jpg* на вычислительных узлах и передают их в контейнер *job-myrenderjob* в вашей учетной записи хранения. Чтобы просмотреть выходные данные, скачайте файлы в папку на локальном компьютере с помощью команды [az storage blob download-batch](/cli/azure/storage/blob). Пример:

```azurecli-interactive
az storage blob download-batch \
    --source job-myrenderjob \
    --destination .
```

Откройте один из файлов на своем компьютере. Отрисованный кадр 6 будет выглядеть примерно так:

![Отрисованный кадр с драконом 6](./media/tutorial-rendering-cli/dragon-frame6.png) 

## <a name="clean-up-resources"></a>Очистка ресурсов

Вы можете удалить ставшие ненужными группу ресурсов, учетную запись пакетной службы, пулы и все связанные с ними ресурсы, выполнив команду [az group delete](/cli/azure/group#az-group-delete). Удалите ресурсы следующим образом:

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>Дальнейшие действия

Из этого руководства вы узнали, как выполнять такие задачи:

> [!div class="checklist"]
> - Отправка сцен в хранилище Azure.
> - Создание пула пакетной службы для рендеринга.
> - Отрисовка однокадровой сцены с помощью Arnold.
> - Масштабирование пула и отрисовка сцены с несколькими кадрами.
> - Скачивание выводимых данных.

Дополнительные сведения о рендеринге облачных вычислений см. в документации о возможностях рендеринга в пакетной службе.

> [!div class="nextstepaction"]
> [Пакетная служба рендеринга](batch-rendering-service.md)
