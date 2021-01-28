---
title: Перемещение артефактов
description: Передача коллекций образов или других артефактов из одного реестра контейнеров в другой в реестр путем создания конвейера передачи с помощью учетных записей хранения Azure
ms.topic: article
ms.date: 10/07/2020
ms.custom: ''
ms.openlocfilehash: ab6657ecd335a6de8c6c93e3c2ff392ac54c487c
ms.sourcegitcommit: 2f9f306fa5224595fa5f8ec6af498a0df4de08a8
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/28/2021
ms.locfileid: "98935355"
---
# <a name="transfer-artifacts-to-another-registry"></a>Перенос артефактов в другой реестр

В этой статье показано, как передавать коллекции образов или другие артефакты реестра из одного реестра контейнеров Azure в другой реестр. Исходные и целевые реестры могут находиться в одной или разных подписках, Active Directory клиентах, облаках Azure или физически отключенных облаках. 

Для передачи артефактов создается *конвейер передачи* , который реплицирует артефакты между двумя реестрами с помощью [хранилища BLOB-объектов](../storage/blobs/storage-blobs-introduction.md):

* Артефакты из исходного реестра экспортируются в большой двоичный объект в исходной учетной записи хранения. 
* Большой двоичный объект копируется из исходной учетной записи хранения в целевую учетную запись хранения.
* Большой двоичный объект в целевой учетной записи хранения импортируется как артефакты в целевой реестр. Вы можете настроить конвейер импорта для запуска при каждом обновлении BLOB-объекта артефакта в целевом хранилище.

Перенос идеально подходит для копирования содержимого между двумя реестрами контейнеров Azure в физически отключенных облаках, которые исправляются учетными записями хранения в каждом облаке. Если вместо этого вы хотите копировать образы из реестров контейнеров в подключенных облаках, включая DOCKER HUB и другие поставщики облачных решений, рекомендуется [импортировать изображения](container-registry-import-images.md) .

В этой статье описано, как использовать развертывания шаблонов Azure Resource Manager для создания и запуска конвейера передачи. Azure CLI используется для предоставления связанных ресурсов, таких как секреты хранилища. Рекомендуется Azure CLI версии 2.2.0 или более поздней. Если вам необходимо установить или обновить CLI, см. статью [Установка Azure CLI][azure-cli].

Эта возможность доступна на уровне **Премиум** службы реестра контейнеров. Сведения об уровнях служб реестра и ограничениях см. в статье [Уровни службы Реестра контейнеров Azure](container-registry-skus.md).

> [!IMPORTANT]
> Эта функция в настоящее время находится на стадии предварительной версии. Предварительные версии предоставляются при условии, что вы принимаете [дополнительные условия использования][terms-of-use]. Некоторые аспекты этой функции могут быть изменены до выхода общедоступной версии.

## <a name="prerequisites"></a>Предварительные требования

* Реестры **контейнеров** . для перемещения необходимо использовать существующий исходный раздел реестра с артефактами и целевой реестр. Перенос записей контроля доступа предназначен для перемещения в физически отключенных облаках. Для тестирования исходный и целевой реестры могут находиться в той же или другой подписке Azure, Active Directory клиенте или облаке. 

   Если необходимо создать реестр, см. раздел [Краткое руководство. создание закрытого реестра контейнеров с помощью Azure CLI](container-registry-get-started-azure-cli.md). 
* **Учетные записи хранения** — Создайте исходную и целевую учетные записи хранения в подписке и расположении по своему усмотрению. В целях тестирования можно использовать ту же подписку или подписки, что и в исходном и целевом реестрах. В сценариях с несколькими облаками обычно создается отдельная учетная запись хранения в каждом облаке. 

  При необходимости создайте учетные записи хранения с помощью [Azure CLI](../storage/common/storage-account-create.md?tabs=azure-cli) или других средств. 

  Создайте контейнер больших двоичных объектов для перемещения артефактов в каждой учетной записи. Например, создайте контейнер с именем *Перемещение*. Два или более конвейеров передачи могут совместно использовать одну и ту же учетную запись хранения, но должны использовать разные области контейнера хранилища.
* **Хранилища ключей** . хранилища ключей необходимы для хранения секретов маркеров SAS, используемых для доступа к исходным и целевым учетным записям хранения. Создайте исходное и целевое хранилище ключей в той же подписке Azure или подписки, что и исходный и целевой реестр. В демонстрационных целях в шаблонах и командах, используемых в этой статье, предполагается, что исходные и целевые хранилища ключей находятся в тех же группах ресурсов, что и исходные и целевые реестры соответственно. Использование общих групп ресурсов не является обязательным, но упрощает шаблоны и команды, используемые в этой статье.

   При необходимости создайте хранилища ключей с помощью [Azure CLI](../key-vault/secrets/quick-create-cli.md) или других средств.

* **Переменные среды** . Например, команды в этой статье устанавливают следующие переменные среды для исходной и целевой сред. Все примеры отформатированы для оболочки bash.
  ```console
  SOURCE_RG="<source-resource-group>"
  TARGET_RG="<target-resource-group>"
  SOURCE_KV="<source-key-vault>"
  TARGET_KV="<target-key-vault>"
  SOURCE_SA="<source-storage-account>"
  TARGET_SA="<target-storage-account>"
  ```

## <a name="scenario-overview"></a>Общие сведения о сценарии

Для передачи образа между реестрами необходимо создать следующие три ресурса конвейера. Все они создаются с помощью операций размещения. Эти ресурсы работают с *исходными* и *целевыми* реестрами и учетными записями хранения. 

Проверка подлинности хранилища использует маркеры SAS, управляемые как секреты в хранилищах ключей. Конвейеры используют управляемые удостоверения для чтения секретов в хранилищах.

* **[Експортпипелине](#create-exportpipeline-with-resource-manager)** — долгосрочный ресурс, содержащий высокоуровневые сведения о *исходном* реестре и учетной записи хранения. Эти сведения включают универсальный код ресурса (URI) контейнера BLOB-объектов хранилища и хранилище ключей, управляющее исходным маркером SAS. 
* **[Импортпипелине](#create-importpipeline-with-resource-manager)** — долгосрочный ресурс, содержащий высокоуровневые сведения о *целевом* реестре и учетной записи хранения. Эти сведения включают в себя URI контейнера BLOB-объектов хранилища и хранилище ключей, управляющее целевым маркером SAS. Триггер импорта включен по умолчанию, поэтому конвейер запускается автоматически при создании BLOB-объекта артефакта в целевом контейнере хранилища. 
* **[Пипелинерун](#create-pipelinerun-for-export-with-resource-manager)** — ресурс, используемый для вызова ресурса Експортпипелине или импортпипелине.  
  * Чтобы запустить Експортпипелине вручную, создайте ресурс Пипелинерун и укажите артефакты для экспорта.  
  * Если включен триггер импорта, Импортпипелине запускается автоматически. Его также можно запустить вручную с помощью Пипелинерун. 
  * В настоящее время для каждого Пипелинерун можно передать не более **50 артефактов** .

### <a name="things-to-know"></a>Полезная информация
* Експортпипелине и Импортпипелине обычно находятся в разных клиентах Active Directory, связанных с исходным и целевым облаками. В этом сценарии для ресурсов экспорта и импорта требуются отдельные управляемые удостоверения и хранилища ключей. В целях тестирования эти ресурсы можно разместить в том же облаке, совместно использовать удостоверения.
* По умолчанию шаблоны Експортпипелине и Импортпипелине позволяют управляемому системному удостоверению получать доступ к секреты хранилища ключей. Шаблоны Експортпипелине и Импортпипелине также поддерживают предоставленное пользователем удостоверение. 

## <a name="create-and-store-sas-keys"></a>Создание и хранение ключей SAS

При переносе используются маркеры подписанного URL-адрес (SAS) для доступа к учетным записям хранения в исходной и целевой средах. Создайте и сохраните токены, как описано в следующих разделах.  

### <a name="generate-sas-token-for-export"></a>Создать маркер SAS для экспорта

Выполните команду [AZ Storage Container Generate-SAS][az-storage-container-generate-sas] , чтобы создать маркер SAS для контейнера в исходной учетной записи хранения, используемый для экспорта артефактов.

*Рекомендуемые разрешения маркера*: чтение, запись, список, добавить. 

В следующем примере выходные данные команды присваиваются переменной среды EXPORT_SAS с префиксом "?". Обновите `--expiry` значение для своей среды:

```azurecli
EXPORT_SAS=?$(az storage container generate-sas \
  --name transfer \
  --account-name $SOURCE_SA \
  --expiry 2021-01-01 \
  --permissions alrw \
  --https-only \
  --output tsv)
```

### <a name="store-sas-token-for-export"></a>Токен SAS хранилища для экспорта

Сохраните маркер SAS в исходном хранилище ключей Azure с помощью команды [AZ keyvault Secret Set][az-keyvault-secret-set]:

```azurecli
az keyvault secret set \
  --name acrexportsas \
  --value $EXPORT_SAS \
  --vault-name $SOURCE_KV 
```

### <a name="generate-sas-token-for-import"></a>Создать маркер SAS для импорта

Выполните команду [AZ Storage Container Generate-SAS][az-storage-container-generate-sas] , чтобы создать маркер SAS для контейнера в целевой учетной записи хранения, используемый для импорта артефактов.

*Рекомендуемые разрешения маркера*: чтение, удаление, список

В следующем примере выходные данные команды присваиваются переменной среды IMPORT_SAS с префиксом "?". Обновите `--expiry` значение для своей среды:

```azurecli
IMPORT_SAS=?$(az storage container generate-sas \
  --name transfer \
  --account-name $TARGET_SA \
  --expiry 2021-01-01 \
  --permissions dlr \
  --https-only \
  --output tsv)
```

### <a name="store-sas-token-for-import"></a>Токен SAS хранилища для импорта

Сохраните маркер SAS в целевом хранилище ключей Azure с помощью команды [AZ keyvault Secret Set][az-keyvault-secret-set]:

```azurecli
az keyvault secret set \
  --name acrimportsas \
  --value $IMPORT_SAS \
  --vault-name $TARGET_KV 
```

## <a name="create-exportpipeline-with-resource-manager"></a>Создание Експортпипелине с помощью диспетчер ресурсов

Создайте ресурс Експортпипелине для исходного реестра контейнеров с помощью развертывания шаблона Azure Resource Manager.

Скопируйте [файлы шаблонов](https://github.com/Azure/acr/tree/master/docs/image-transfer/ExportPipelines) експортпипелине диспетчер ресурсов в локальную папку.

Введите следующие значения параметров в файл `azuredeploy.parameters.json` :

|Параметр  |Значение  |
|---------|---------|
|регистринаме     | Имя исходного реестра контейнеров      |
|експортпипелиненаме     |  Имя, выбранное для конвейера экспорта       |
|targetUri     |  URI контейнера хранилища в исходной среде (целевой объект конвейера экспорта).<br/>Например, `https://sourcestorage.blob.core.windows.net/transfer`.       |
|keyVaultName     |  Имя исходного хранилища ключей  |
|састокенсекретнаме  | Имя секрета маркера SAS в исходном хранилище ключей <br/>Пример: акрекспортсас

### <a name="export-options"></a>Параметры экспорта

`options`Свойство для конвейеров экспорта поддерживает необязательные логические значения. Рекомендуется использовать следующие значения:

|Параметр  |Значение  |
|---------|---------|
|options | Овервритеблобс — перезаписать существующие целевые BLOB-объекты<br/>Континуеонеррорс — продолжайте экспорт оставшихся артефактов в исходном реестре в случае сбоя экспорта одного артефакта.

### <a name="create-the-resource"></a>Создание ресурса

Выполните команду [AZ Deployment Group Create][az-deployment-group-create] , чтобы создать ресурс с именем *експортпипелине* , как показано в следующих примерах. По умолчанию с первым параметром пример шаблона включает в ресурсе Експортпипелине удостоверение, назначенное системой. 

Второй вариант позволяет предоставить ресурсу удостоверение, назначенное пользователем. (Не показано создание назначенного пользователю удостоверения.)

При использовании любого из этих параметров шаблон настраивает удостоверение для доступа к маркеру SAS в хранилище ключей экспорта. 

#### <a name="option-1-create-resource-and-enable-system-assigned-identity"></a>Вариант 1. Создание ресурса и включение назначенного системой удостоверения

```azurecli
az deployment group create \
  --resource-group $SOURCE_RG \
  --template-file azuredeploy.json \
  --name exportPipeline \
  --parameters azuredeploy.parameters.json
```

#### <a name="option-2-create-resource-and-provide-user-assigned-identity"></a>Вариант 2. Создание ресурса и указание назначенного пользователю удостоверения

В этой команде укажите идентификатор ресурса назначенного пользователем удостоверения в качестве дополнительного параметра.

```azurecli
az deployment group create \
  --resource-group $SOURCE_RG \
  --template-file azuredeploy.json \
  --name exportPipeline \
  --parameters azuredeploy.parameters.json \
  --parameters userAssignedIdentity="/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myUserAssignedIdentity"
```

В выходных данных команды запишите идентификатор ресурса ( `id` ) конвейера. Это значение можно сохранить в переменной среды для последующего использования, выполнив команду [AZ Deployment Group][az-deployment-group-show]. Пример.

```azurecli
EXPORT_RES_ID=$(az deployment group show \
  --resource-group $SOURCE_RG \
  --name exportPipeline \
  --query 'properties.outputResources[1].id' \
  --output tsv)
```

## <a name="create-importpipeline-with-resource-manager"></a>Создание Импортпипелине с помощью диспетчер ресурсов 

Создайте ресурс Импортпипелине в целевом реестре контейнеров, используя развертывание шаблона Azure Resource Manager. По умолчанию конвейер включен для автоматического импорта, если учетная запись хранения в целевой среде имеет большой двоичный объект артефакта.

Скопируйте [файлы шаблонов](https://github.com/Azure/acr/tree/master/docs/image-transfer/ImportPipelines) импортпипелине диспетчер ресурсов в локальную папку.

Введите следующие значения параметров в файл `azuredeploy.parameters.json` :

Параметр  |Значение  |
|---------|---------|
|регистринаме     | Имя целевого реестра контейнеров      |
|импортпипелиненаме     |  Имя, выбранное для конвейера импорта       |
|sourceUri     |  URI контейнера хранилища в целевой среде (источник для конвейера импорта).<br/>Например, `https://targetstorage.blob.core.windows.net/transfer/`.|
|keyVaultName     |  Имя целевого хранилища ключей |
|састокенсекретнаме     |  Имя секрета маркера SAS в целевом хранилище ключей<br/>Пример: запись контроля доступа импортсас |

### <a name="import-options"></a>Способы импорта

`options`Свойство конвейера импорта поддерживает необязательные логические значения. Рекомендуется использовать следующие значения:

|Параметр  |Значение  |
|---------|---------|
|options | Овервритетагс — перезаписать существующие целевые Теги<br/>Делетесаурцеблобонсукцесс — удаляется исходный BLOB-объект хранилища после успешного импорта в целевой реестр.<br/>Континуеонеррорс — продолжить импорт оставшихся артефактов в целевом реестре, если импорт одного артефакта завершается неудачно.

### <a name="create-the-resource"></a>Создание ресурса

Выполните команду [AZ Deployment Group Create][az-deployment-group-create] , чтобы создать ресурс с именем *импортпипелине* , как показано в следующих примерах. По умолчанию с первым параметром пример шаблона включает в ресурсе Импортпипелине удостоверение, назначенное системой. 

Второй вариант позволяет предоставить ресурсу удостоверение, назначенное пользователем. (Не показано создание назначенного пользователю удостоверения.)

При использовании любого из этих параметров шаблон настраивает удостоверение для доступа к маркеру SAS в хранилище ключей импорта. 

#### <a name="option-1-create-resource-and-enable-system-assigned-identity"></a>Вариант 1. Создание ресурса и включение назначенного системой удостоверения

```azurecli
az deployment group create \
  --resource-group $TARGET_RG \
  --template-file azuredeploy.json \
  --name importPipeline \
  --parameters azuredeploy.parameters.json 
```

#### <a name="option-2-create-resource-and-provide-user-assigned-identity"></a>Вариант 2. Создание ресурса и указание назначенного пользователю удостоверения

В этой команде укажите идентификатор ресурса назначенного пользователем удостоверения в качестве дополнительного параметра.

```azurecli
az deployment group create \
  --resource-group $TARGET_RG \
  --template-file azuredeploy.json \
  --name importPipeline \
  --parameters azuredeploy.parameters.json \
  --parameters userAssignedIdentity="/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myUserAssignedIdentity"
```

Если вы планируете выполнить импорт вручную, запишите идентификатор ресурса ( `id` ) конвейера. Это значение можно сохранить в переменной среды для последующего использования, выполнив команду [AZ Deployment Group показ][az-deployment-group-show] . Пример.

```azurecli
IMPORT_RES_ID=$(az deployment group show \
  --resource-group $TARGET_RG \
  --name importPipeline \
  --query 'properties.outputResources[1].id' \
  --output tsv)
```

## <a name="create-pipelinerun-for-export-with-resource-manager"></a>Создание Пипелинерун для экспорта с помощью диспетчер ресурсов 

Создайте ресурс Пипелинерун для исходного реестра контейнеров с помощью развертывания шаблона Azure Resource Manager. Этот ресурс запускает ранее созданный ресурс Експортпипелине и экспортирует указанные артефакты из реестра контейнеров в качестве BLOB-объекта в исходную учетную запись хранения.

Скопируйте [файлы шаблонов](https://github.com/Azure/acr/tree/master/docs/image-transfer/PipelineRun/PipelineRun-Export) пипелинерун диспетчер ресурсов в локальную папку.

Введите следующие значения параметров в файл `azuredeploy.parameters.json` :

|Параметр  |Значение  |
|---------|---------|
|регистринаме     | Имя исходного реестра контейнеров      |
|пипелинеруннаме     |  Имя, выбранное для запуска       |
|пипелинересаурцеид     |  Идентификатор ресурса для конвейера экспорта.<br/>Например, `/subscriptions/<subscriptionID>/resourceGroups/<resourceGroupName>/providers/Microsoft.ContainerRegistry/registries/<sourceRegistryName>/exportPipelines/myExportPipeline`.|
|targetName     |  Имя, выбранное для BLOB-объекта артефактов, экспортированного в исходную учетную запись хранения, например *myblob*
|артефакты | Массив исходных артефактов для перемещения в виде тегов или дайджестов манифеста<br/>Например, `[samples/hello-world:v1", "samples/nginx:v1" , "myrepository@sha256:0a2e01852872..."]`. |

При повторном развертывании ресурса Пипелинерун с идентичными свойствами необходимо также использовать свойство [forceUpdateTag](#redeploy-pipelinerun-resource) .

Выполните команду [AZ Deployment Group Create][az-deployment-group-create] , чтобы создать ресурс пипелинерун. В следующем примере называется *експортпипелинерун* развертывания.

```azurecli
az deployment group create \
  --resource-group $SOURCE_RG \
  --template-file azuredeploy.json \
  --name exportPipelineRun \
  --parameters azuredeploy.parameters.json
```

Для последующего использования Сохраните идентификатор ресурса для запуска конвейера в переменной среды:

```azurecli
EXPORT_RUN_RES_ID=$(az deployment group show \
  --resource-group $SOURCE_RG \
  --name exportPipelineRun \
  --query 'properties.outputResources[0].id' \
  --output tsv)
```

Экспорт артефактов может занять несколько минут. После успешного завершения развертывания проверьте экспорт артефактов, выполнив список экспортированного большого двоичного объекта в контейнере *перемещения* исходной учетной записи хранения. Например, выполните команду [AZ Storage BLOB List][az-storage-blob-list] :

```azurecli
az storage blob list \
  --account-name $SOURCE_SA \
  --container transfer \
  --output table
```

## <a name="transfer-blob-optional"></a>Перенос большого двоичного объекта (необязательно) 

Используйте средство AzCopy или другие методы для [перемещения данных большого двоичного объекта](../storage/common/storage-use-azcopy-v10.md#transfer-data) из исходной учетной записи хранения в целевую учетную запись хранения.

Например, следующая [`azcopy copy`](../storage/common/storage-ref-azcopy-copy.md) команда копирует myblob из контейнера *перемещения* в исходной учетной записи в контейнер *перемещения* в целевой учетной записи. Если большой двоичный объект существует в целевой учетной записи, он перезаписывается. Для проверки подлинности используются маркеры SAS с соответствующими разрешениями для исходного и целевого контейнеров. (Шаги по созданию маркеров не показаны.)

```console
azcopy copy \
  'https://<source-storage-account-name>.blob.core.windows.net/transfer/myblob'$SOURCE_SAS \
  'https://<destination-storage-account-name>.blob.core.windows.net/transfer/myblob'$TARGET_SAS \
  --overwrite true
```

## <a name="trigger-importpipeline-resource"></a>Запустить ресурс Импортпипелине

Если вы включили `sourceTriggerStatus` параметр импортпипелине (значение по умолчанию), конвейер активируется после копирования большого двоичного объекта в целевую учетную запись хранения. Импорт артефактов может занять несколько минут. После успешного завершения импорта проверьте импорт артефактов, выполнив список репозиториев в целевом реестре контейнеров. Например, выполните команду [AZ контроля доступа к репозиторию][az-acr-repository-list]:

```azurecli
az acr repository list --name <target-registry-name>
```

Если вы не включили `sourceTriggerStatus` параметр конвейера импорта, запустите ресурс импортпипелине вручную, как показано в следующем разделе. 

## <a name="create-pipelinerun-for-import-with-resource-manager-optional"></a>Создание Пипелинерун для импорта с диспетчер ресурсов (необязательно) 
 
Вы также можете использовать ресурс Пипелинерун, чтобы активировать Импортпипелине для импорта артефактов в целевой реестр контейнеров.

Скопируйте [файлы шаблонов](https://github.com/Azure/acr/tree/master/docs/image-transfer/PipelineRun/PipelineRun-Import) пипелинерун диспетчер ресурсов в локальную папку.

Введите следующие значения параметров в файл `azuredeploy.parameters.json` :

|Параметр  |Значение  |
|---------|---------|
|регистринаме     | Имя целевого реестра контейнеров      |
|пипелинеруннаме     |  Имя, выбранное для запуска       |
|пипелинересаурцеид     |  Идентификатор ресурса конвейера импорта.<br/>Например, `/subscriptions/<subscriptionID>/resourceGroups/<resourceGroupName>/providers/Microsoft.ContainerRegistry/registries/<sourceRegistryName>/importPipelines/myImportPipeline`.       |
|sourceName     |  Имя существующего большого двоичного объекта для экспортированных артефактов в учетной записи хранения, например *myblob*

При повторном развертывании ресурса Пипелинерун с идентичными свойствами необходимо также использовать свойство [forceUpdateTag](#redeploy-pipelinerun-resource) .

Выполните команду [AZ Deployment Group Create][az-deployment-group-create] , чтобы запустить ресурс.

```azurecli
az deployment group create \
  --resource-group $TARGET_RG \
  --name importPipelineRun \
  --template-file azuredeploy.json \
  --parameters azuredeploy.parameters.json
```

Для последующего использования Сохраните идентификатор ресурса для запуска конвейера в переменной среды:

```azurecli
IMPORT_RUN_RES_ID=$(az deployment group show \
  --resource-group $TARGET_RG \
  --name importPipelineRun \
  --query 'properties.outputResources[0].id' \
  --output tsv)

When deployment completes successfully, verify artifact import by listing the repositories in the target container registry. For example, run [az acr repository list][az-acr-repository-list]:

```azurecli
az acr repository list --name <target-registry-name>
```

## <a name="redeploy-pipelinerun-resource"></a>Повторное развертывание ресурса Пипелинерун

При повторном развертывании ресурса Пипелинерун с *одинаковыми свойствами* необходимо использовать свойство **forceUpdateTag** . Это свойство указывает, что ресурс Пипелинерун должен быть создан повторно, даже если конфигурация не изменилась. При каждом повторном развертывании ресурса Пипелинерун убедитесь, что forceUpdateTag различаются. В приведенном ниже примере повторно создается Пипелинерун для экспорта. Текущая дата и время используются для установки forceUpdateTag, тем самым гарантируя, что это свойство всегда является уникальным.

```console
CURRENT_DATETIME=`date +"%Y-%m-%d:%T"`
```

```azurecli
az deployment group create \
  --resource-group $SOURCE_RG \
  --template-file azuredeploy.json \
  --name exportPipelineRun \
  --parameters azuredeploy.parameters.json \
  --parameters forceUpdateTag=$CURRENT_DATETIME
```

## <a name="delete-pipeline-resources"></a>Удаление ресурсов конвейера

В следующих примерах команд Команда [AZ Resource Delete][az-resource-delete] используется для удаления ресурсов конвейера, созданных в этой статье. Идентификаторы ресурсов ранее хранились в переменных среды.

```
# Delete export resources
az resource delete \
--resource-group $SOURCE_RG \
--ids $EXPORT_RES_ID $EXPORT_RUN_RES_ID \
--api-version 2019-12-01-preview

# Delete import resources
az resource delete \
--resource-group $TARGET_RG \
--ids $IMPORT_RES_ID $IMPORT_RUN_RES_ID \
--api-version 2019-12-01-preview
```

## <a name="troubleshooting"></a>Устранение неполадок

* **Сбои или ошибки Шаблоны развертывания**
  * В случае сбоя выполнения конвейера просмотрите `pipelineRunErrorMessage` свойство ресурса Run.
  * Общие сведения об ошибках развертывания шаблонов см. в статье [Устранение неполадок при развертывании шаблонов ARM](../azure-resource-manager/templates/template-tutorial-troubleshoot.md) .
* **Проблемы с экспортом или импортом больших двоичных объектов хранилища**
  * Возможно, истек срок действия маркера SAS или у вас недостаточно разрешений для указанного выполнения экспорта или импорта
  * Существующий большой двоичный объект хранилища в исходной учетной записи хранения может быть не перезаписан во время нескольких запусков экспорта. Убедитесь, что в ходе выполнения экспорта задан параметр Овервритеблоб, а маркер SAS имеет достаточные разрешения.
  * Возможно, BLOB-объект хранилища в целевой учетной записи хранения не будет удален после успешного выполнения импорта. Убедитесь, что параметр Делетеблобонсукцесс установлен в выполнении импорта, а маркер SAS имеет достаточные разрешения.
  * Большой двоичный объект хранилища не создан или удален. Убедитесь, что контейнер, указанный при выполнении экспорта или импорта, существует, или существует указанный большой двоичный объект хранилища для выполнения импорта вручную. 
* **Проблемы AzCopy**
  * См. раздел [Устранение неполадок AzCopy](../storage/common/storage-use-azcopy-configure.md#troubleshoot-issues).  
* **Проблемы с перенаправлением артефактов**
  * Передаются не все артефакты или нет. Подтвердите написание артефактов при выполнении экспорта и имя большого двоичного объекта при экспорте и импорте. Убедитесь, что вы передаете не более 50 артефактов.
  * Выполнение конвейера может быть не завершено. Выполнение экспорта или импорта может занять некоторое время. 
  * Для других проблем конвейера укажите [идентификатор корреляции](../azure-resource-manager/templates/deployment-history.md) развертывания для выполнения экспорта или импорта в группу реестра контейнеров Azure.


## <a name="next-steps"></a>Дальнейшие действия

Сведения об импорте одного образа контейнера в реестр контейнеров Azure из общедоступного реестра или другого частного реестра см. в справочнике по команде [AZ запись контроля][az-acr-import] доступа.

<!-- LINKS - External -->
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/



<!-- LINKS - Internal -->
[azure-cli]: /cli/azure/install-azure-cli
[az-login]: /cli/azure/reference-index#az-login
[az-keyvault-secret-set]: /cli/azure/keyvault/secret#az-keyvault-secret-set
[az-keyvault-secret-show]: /cli/azure/keyvault/secret#az-keyvault-secret-show
[az-keyvault-set-policy]: /cli/azure/keyvault#az-keyvault-set-policy
[az-storage-container-generate-sas]: /cli/azure/storage/container#az-storage-container-generate-sas
[az-storage-blob-list]: /cli/azure/storage/blob#az-storage-blob-list
[az-deployment-group-create]: /cli/azure/deployment/group#az-deployment-group-create
[az-deployment-group-delete]: /cli/azure/deployment/group#az-deployment-group-delete
[az-deployment-group-show]: /cli/azure/deployment/group#az-deployment-group-show
[az-acr-repository-list]: /cli/azure/acr/repository#az-acr-repository-list
[az-acr-import]: /cli/azure/acr#az-acr-import
[az-resource-delete]: /cli/azure/resource#az-resource-delete
