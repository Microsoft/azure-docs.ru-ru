---
title: Устранение неполадок службы Azure Image Builder
description: Устранение распространенных проблем и ошибок при использовании службы "Построитель образов виртуальных машин Azure"
author: cynthn
ms.author: danis
ms.date: 10/02/2020
ms.topic: troubleshooting
ms.service: virtual-machines
ms.subservice: image-builder
ms.collection: linux
ms.openlocfilehash: f76c3e6c739ae4dd13355d350a01b878e4d4f360
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101666215"
---
# <a name="troubleshoot-azure-image-builder-service"></a>Устранение неполадок службы Azure Image Builder

Эта статья поможет вам устранить и устранить распространенные проблемы, которые могут возникнуть при использовании службы Azure Image Builder.

Ошибки AIB могут возникать в двух областях:
- Отправка шаблона изображения
- Сборка образа

## <a name="troubleshoot-image-template-submission-errors"></a>Устранение ошибок отправки шаблона образа

Ошибки отправки шаблона образа возвращаются только при отправке. Не существует журнала ошибок отправки шаблона образа. Если во время отправки произошла ошибка, можно возвратить ошибку, проверив состояние шаблона, в частности просмотрев `ProvisioningState` и `ProvisioningErrorMessage` / `provisioningError` .

```azurecli
az image builder show --name $imageTemplateName  --resource-group $imageResourceGroup
```

```azurepowershell-interactive
Get-AzImageBuilderTemplate -ImageTemplateName  <imageTemplateName> -ResourceGroupName <imageTemplateResourceGroup> | Select-Object ProvisioningState, ProvisioningErrorMessage
```
> [!NOTE]
> Для PowerShell необходимо установить [модули PowerShell для Azure Image Builder](../windows/image-builder-powershell.md#prerequisites).

В следующих разделах приведены рекомендации по устранению проблем для распространенных ошибок отправки шаблона образа.

### <a name="updateupgrade-of-image-templates-is-currently-not-supported"></a>Обновление или обновление шаблонов образов сейчас не поддерживается

#### <a name="error"></a>Ошибка

```text
'Conflict'. Details: Update/Upgrade of image templates is currently not supported
```

#### <a name="cause"></a>Причина

Шаблон уже существует.

#### <a name="solution"></a>Решение

Если вы отправите шаблон конфигурации образа и отправка завершается неудачей, неудачный артефакт шаблона по-прежнему существует. Удалите шаблон, который не удалось выполнить.

### <a name="the-resource-operation-completed-with-terminal-provisioning-state-failed"></a>Операция с ресурсом завершена с состоянием подготовки терминала "сбой"

#### <a name="error"></a>Ошибка

```text
Microsoft.VirtualMachineImages/imageTemplates 'helloImageTemplateforSIG01' failed with message '{
  "status": "Failed",
  "error": {
    "code": "ResourceDeploymentFailure",
    "message": "The resource operation completed with terminal provisioning state 'Failed'.",
    "details": [
      {
        "code": "InternalOperationError",
        "message": "Internal error occurred."
```
#### <a name="cause"></a>Причина

В большинстве случаев ошибка при развертывании ресурса возникает из-за отсутствия разрешений.

#### <a name="solution"></a>Решение

В зависимости от вашего сценария для построителя образов Azure могут потребоваться следующие разрешения:
- Исходный образ или группа ресурсов коллекции общих образов
- Образ распространения или общий ресурс коллекции образов
- Учетная запись хранения, контейнер или большой двоичный объект, к которому обращается настраиваемый файл. 

Дополнительные сведения о настройке разрешений см. в статье [Настройка разрешений службы Azure Image Builder с помощью Azure CLI](image-builder-permissions-cli.md) или [Настройка разрешений службы Azure Image Builder с помощью PowerShell](image-builder-permissions-powershell.md) .

### <a name="error-getting-managed-image"></a>Ошибка при получении управляемого образа

#### <a name="error"></a>Ошибка

```text
Build (Managed Image) step failed: Error getting Managed Image '/subscriptions/.../providers/Microsoft.Compute/images/mymanagedmg1': Error getting managed image (...): compute.
ImagesClient#Get: Failure responding to request: StatusCode=403 -- Original Error: autorest/azure: Service returned an error.
Status=403 Code="AuthorizationFailed" Message="The client '......' with object id '......' does not have authorization to perform action 'Microsoft.Compute/images/read' over scope 
```
#### <a name="cause"></a>Причина

Отсутствуют разрешения.

#### <a name="solution"></a>Решение

В зависимости от вашего сценария для построителя образов Azure могут потребоваться следующие разрешения:
* Исходный образ или группа ресурсов коллекции общих образов
* Образ распространения или общий ресурс коллекции образов
* Учетная запись хранения, контейнер или большой двоичный объект, к которому обращается настраиваемый файл. 

Дополнительные сведения о настройке разрешений см. в статье [Настройка разрешений службы Azure Image Builder с помощью Azure CLI](image-builder-permissions-cli.md) или [Настройка разрешений службы Azure Image Builder с помощью PowerShell](image-builder-permissions-powershell.md) .

### <a name="build--step-failed-for-image-version"></a>Не удалось выполнить этап сборки для версии образа

#### <a name="error"></a>Ошибка
```text
Build (Shared Image Version) step failed for Image Version '/subscriptions/.../providers/Microsoft.Compute/galleries/.../images/... /versions/0.23768.4001': Error getting Image Version '/subscriptions/.../resourceGroups/<rgName>/providers/Microsoft.Compute/galleries/.../images/.../versions/0.23768.4001': Error getting image version '... :0.23768.4001': compute.GalleryImageVersionsClient#Get: Failure responding to request: StatusCode=404 -- Original Error: autorest/azure: Service returned an error. 
Status=404 Code="ResourceNotFound" Message="The Resource 'Microsoft.Compute/galleries/.../images/.../versions/0.23768.4001' under resource group '<rgName>' was not found."
```
#### <a name="cause"></a>Причина

Построитель образов Azure не может нахождение исходного образа.

#### <a name="solution"></a>Решение

Убедитесь, что исходное изображение указано правильно и существует в расположении службы "Построитель образов Azure".

### <a name="downloading-external-file-to-local-file"></a>Загрузка внешнего файла в локальный файл

#### <a name="error"></a>Ошибка

```text
Downloading external file (<myFile>) to local file (xxxxx.0.customizer.fp) [attempt 1 of 10] failed: Error downloading '<myFile>' to 'xxxxx.0.customizer.fp'..
```

#### <a name="cause"></a>Причина

Неверное имя или расположение файла, или расположение недоступно.

#### <a name="solution"></a>Решение

Убедитесь, что файл доступен. Проверьте правильность имени и расположения.

## <a name="troubleshoot-build-failures"></a>Устранение ошибок сборки

Для сбоев сборки образа можно получить ошибку из `lastrunstatus` , а затем просмотреть сведения в журнале настройки. log.


```azurecli
az image builder show --name $imageTemplateName  --resource-group $imageResourceGroup
```

```azurepowershell-interactive
Get-AzImageBuilderTemplate -ImageTemplateName  <imageTemplateName> -ResourceGroupName <imageTemplateResourceGroup> | Select-Object LastRunStatus, LastRunStatusMessage
```

### <a name="customization-log"></a>Журнал настройки

При выполнении сборки образа журналы создаются и сохраняются в учетной записи хранения. Построитель образов Azure создает учетную запись хранения во временной группе ресурсов при создании артефакта шаблона образа.

Имя учетной записи хранения использует следующий шаблон: **IT_ \<ImageResourceGroupName\> _\<TemplateName\>_ \<GUID\>**

Например, *IT_aibmdi_helloImageTemplateLinux01*.

Вы можете просмотреть настройку. Войдите в учетную запись хранения в группе ресурсов, выбрав BLOB-объекты **учетной записи хранения**  >    >  `packerlogs` .  Затем выберите **каталог > Настройка. log**.


### <a name="understanding-the-customization-log"></a>Основные сведения о журнале настройки

Журнал является подробным. В нем рассматривается сборка образа, включающая все проблемы, связанные с распространением образа, например репликация общей коллекции образов. Эти ошибки отображаются в сообщении об ошибке состояния шаблона образа.

В журнал настройки. log входят следующие этапы.

1. Разверните виртуальную машину сборки и зависимости, используя шаблоны ARM, на этапе промежуточной группы ресурсов IT_. Этот этап включает несколько записей в поставщик ресурсов построителя образов Azure:
    ```text
    Azure request method="POST" request="https://management.azure.com/subscriptions/<subID>/resourceGroups/IT_aibImageRG200_window2019VnetTemplate01_dec33089-1cc3-cccc-cccc-ccccccc/providers/Microsoft.Storage/storageAccounts
    ..
    PACKER OUT ==> azure-arm: Deploying deployment template ...
    ..
    ```

2. Состояние этапа развертываний. Этот этап включает в себя состояние развертывания каждого ресурса:
    ```text
    PACKER ERR 2020/04/30 23:28:50 packer: 2020/04/30 23:28:50 Azure request method="GET" request="https://management.azure.com/subscriptions/<subID>/resourcegroups/IT_aibImageRG200_window2019VnetTemplate01_dec33089-1cc3-4505-ae28-6661e43fac48/providers/Microsoft.Resources/deployments/pkrdp51lc0339jg/operationStatuses/08586133176207523519?[REDACTED]" body=""
    ```

3. Подключитесь к этапу сборки виртуальной машины.

    Если Windows, служба Azure Image Builder подключается с помощью WinRM:
    ```text
    PACKER ERR 2020/04/30 23:30:50 packer: 2020/04/30 23:30:50 Waiting for WinRM, up to timeout: 10m0s
    ..
    PACKER OUT     azure-arm: WinRM connected.
    ```

    Если вы используете Linux, служба Azure Image Builder будет подключаться по протоколу SSH:
    ```text
    PACKER OUT ==> azure-arm: Waiting for SSH to become available...
    PACKER ERR 2019/12/10 17:20:51 packer: 2020/04/10 17:20:51 [INFO] Waiting for SSH, up to timeout: 20m0s
    PACKER OUT ==> azure-arm: Connected to SSH!
    ```

4. Этап настройки запуска. При выполнении настроек их можно найти, просмотрев настройки. log. Выполните поиск по запросу *(телеметрии)*.
    ```text
    (telemetry) Starting provisioner windows-update
    (telemetry) ending windows-update
    (telemetry) Starting provisioner powershell
    (telemetry) ending powershell
    (telemetry) Starting provisioner file
    (telemetry) ending file
    (telemetry) Starting provisioner windows-restart
    (telemetry) ending windows-restart
    
    (telemetry) Finalizing. - This means the build hasfinished
    ```
5. Стадия отмены подготовки. В построителе образов Azure добавлен скрытый элемент настройки. Этот этап отмены подготовки отвечает за подготовку виртуальной машины для отмены подготовки. Он запускает Windows Sysprep (с помощью c:\DeprovisioningScript.ps1) или в Linux waagent unготовить (с помощью/ТМП/депровисионингскрипт.ш). 

    Пример:
    ```text
    PACKER ERR 2020/03/04 23:05:04 [INFO] (telemetry) Starting provisioner powershell
    PACKER ERR 2020/03/04 23:05:04 packer: 2020/03/04 23:05:04 Found command: if( TEST-PATH c:\DeprovisioningScript.ps1 ){cat c:\DeprovisioningScript.ps1} else {echo "Deprovisioning script [c:\DeprovisioningScript.ps1] could not be found. Image build may fail or the VM created from the Image may not boot. Please make sure the deprovisioning script is not accidentally deleted by a Customizer in the Template."}
    ```

6. Стадия очистки. После завершения сборки ресурсы Azure Image Builder будут удалены.
    ```text
    PACKER ERR 2020/02/04 02:04:23 packer: 2020/02/04 02:04:23 Azure request method="DELETE" request="https://management.azure.com/subscriptions/<subId>/resourceGroups/IT_aibDevOpsImg_t_vvvvvvv_yyyyyy-de5f-4f7c-92f2-xxxxxxxx/providers/Microsoft.Network/networkInterfaces/pkrnijamvpo08eo?[REDACTED]" body=""
    ```
## <a name="tips-for-troubleshooting-scriptinline-customization"></a>Советы по устранению неполадок сценария или встроенной настройки
- Протестируйте код перед его предоставлением в построитель изображений
- Убедитесь, что политики и брандмауэры Azure разрешают подключение к удаленным ресурсам.
- Выводить комментарии на консоль, например, с помощью `Write-Host` или `echo` , это позволит выполнять поиск в файл настройки. log.

## <a name="troubleshoot-common-build-errors"></a>Устранение распространенных ошибок сборки

### <a name="packer-build-command-failure"></a>Сбой команды сборки пакета

#### <a name="error"></a>Ошибка

```text
  "provisioningState": "Succeeded",
  "lastRunStatus": {
   "startTime": "2020-04-30T23:24:06.756985789Z",
   "endTime": "2020-04-30T23:39:14.268729811Z",
   "runState": "Failed",
   "message": "Failed while waiting for packerizer: Microservice has failed: Failed while processing request: Error when executing packerizer: Packer build command has failed: exit status 1. During the image build, a failure has occurred, please review the build log to identify which build/customization step failed. For more troubleshooting steps go to https://aka.ms/azvmimagebuilderts. Image Build log location: https://xxxxxxxxxx.blob.core.windows.net/packerlogs/xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxx/customization.log. OperationId: xxxxxx-5a8c-4379-xxxx-8d85493bc791. Use this operationId to search packer logs."
```

#### <a name="cause"></a>Причина

Сбой настройки.

#### <a name="solution"></a>Решение

Проверьте журнал на обнаружение сбоев настраиваемых типов. Выполните поиск по запросу *(телеметрии)*. 

Пример:
```text
(telemetry) Starting provisioner windows-update
(telemetry) ending windows-update
(telemetry) Starting provisioner powershell
(telemetry) ending powershell
(telemetry) Starting provisioner file
(telemetry) ending file
(telemetry) Starting provisioner windows-restart
(telemetry) ending windows-restart

(telemetry) Finalizing. - This means the build has finished
```

### <a name="timeout-exceeded"></a>Превышено время ожидания

#### <a name="error"></a>Ошибка

```text
Deployment failed. Correlation ID: xxxxx-xxxx-xxxx-xxxx-xxxxxxxxx. Failed in building/customizing image: Failed while waiting for packerizer: Timeout waiting for microservice to complete: 'context deadline exceeded'
```

#### <a name="cause"></a>Причина

Сборка превысила время ожидания сборки. Эта ошибка отображается в "ластрунстатус".

#### <a name="solution"></a>Решение

1. Ознакомьтесь с настройкой. log. Найдите последнего настроенного для запуска. Поиск `(telemetry)` начинается с конца журнала. 

2. Проверьте настройки скрипта. Настройки не могут подавить взаимодействие с пользователем для команд, таких как `quiet` Параметры. Например, `apt-get install -y` в результате выполнения скрипта ожидается вмешательство пользователя.

3. Если вы используете `File` настройку для загрузки артефактов, превышающих 20 МБ, см. раздел обходные пути.

4. Проверьте ошибки и зависимости в скрипте, которые могут привести к ожиданию сценария.

5. Если предполагается, что настройки требуют больше времени, увеличьте [буилдтимеаутинминутес](image-builder-json.md). Значение по умолчанию — четыре часа.

6. При наличии ресурсоемких действий, таких как загрузка гигабайтов файлов, учитывайте размер базовой виртуальной машины сборки. Служба использует Standard_D1_v2ную виртуальную машину. Виртуальная машина имеет, 1 виртуальных ЦП и 3,5 ГБ памяти. Если вы скачиваете 50 ГБ, это, скорее всего, приведет к исчерпанию ресурсов виртуальной машины и вызовет сбои связи между службой Azure Image Builder и виртуальной машиной сборки. Повторите сборку, используя виртуальную машину с большим объемом памяти, задав [VM_Size](image-builder-json.md#vmprofile).

### <a name="long-file-download-time"></a>Длительное время загрузки файла

#### <a name="error"></a>Ошибка
```text
[086cf9c4-0457-4e8f-bfd4-908cfe3fe43c] PACKER OUT 
myBigFile.zip 826 B / 826000 B  1.00%
[086cf9c4-0457-4e8f-bfd4-908cfe3fe43c] PACKER OUT 
myBigFile.zip 1652 B / 826000 B  2.00%
[086cf9c4-0457-4e8f-bfd4-908cfe3fe43c] PACKER OUT 
..
hours later...
..
myBigFile.zip 826000 B / 826000 B  100.00%
[086cf9c4-0457-4e8f-bfd4-908cfe3fe43c] PACKER OUT 
```
#### <a name="cause"></a>Причина 

Настройка файлов скачивает большой файл.

#### <a name="solution"></a>Решение

Настройка файлов подходит только для небольших Скачиваний файлов размером менее 20 МБ. Для загрузки больших файлов используйте сценарий или встроенную команду. Например, в Linux можно использовать `wget` или `curl` . В Windows можно использовать `Invoke-WebRequest` .

### <a name="error-waiting-on-shared-image-gallery"></a>Ошибка при ожидании в коллекции общих образов

#### <a name="error"></a>Ошибка

```text
Deployment failed. Correlation ID: XXXXXX-XXXX-XXXXXX-XXXX-XXXXXX. Failed in distributing 1 images out of total 1: {[Error 0] [Distribute 0] Error publishing MDI to shared image gallery:/subscriptions/<subId>/resourceGroups/xxxxxx/providers/Microsoft.Compute/galleries/xxxxx/images/xxxxxx, Location:eastus. Error: Error returned from SIG client while publishing MDI to shared image gallery for dstImageLocation: eastus, dstSubscription: <subId>, dstResourceGroupName: XXXXXX, dstGalleryName: XXXXXX, dstGalleryImageName: XXXXXX. Error: Error waiting on shared image gallery future for resource group: XXXXXX, gallery name: XXXXXX, gallery image name: XXXXXX.Error: Future#WaitForCompletion: context has been cancelled: StatusCode=200 -- Original Error: context deadline exceeded}
```

#### <a name="cause"></a>Причина

Истекло время ожидания создания образа и его репликация в общую галерею образов (SIG) в построителе образов. Если изображение внедряется в подпись, может быть, предполагается, что сборка образа выполнена успешно. Однако общий процесс завершился ошибкой, так как для завершения репликации в построителе образов Ожидалась коллекция общих образов. Несмотря на то, что сборка завершилась неудачно, Репликация продолжится. Свойства версии образа можно получить, проверив *рунаутпут* распространения.

```bash
$runOutputName=<distributionRunOutput>
az resource show \
    --ids "/subscriptions/$subscriptionID/resourcegroups/$imageResourceGroup/providers/Microsoft.VirtualMachineImages/imageTemplates/$imageTemplateName/runOutputs/$runOutputName"  \
    --api-version=2019-05-01-preview
```

#### <a name="solution"></a>Решение

Увеличьте **буилдтимеаутинминутес**.
 
### <a name="low-windows-resource-information-events"></a>События с низкими сведениями о ресурсах Windows

#### <a name="error"></a>Ошибка

```text
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER OUT     azure-arm: Waiting for operation to complete (system performance: 1% cpu; 37% memory)...
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER OUT     azure-arm: Waiting for operation to complete (system performance: 51% cpu; 35% memory)...
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER OUT     azure-arm: Waiting for operation to complete (system performance: 21% cpu; 37% memory)...
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER OUT     azure-arm: Waiting for operation to complete (system performance: 21% cpu; 36% memory)...
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER OUT     azure-arm: Waiting for operation to complete (system performance: 90% cpu; 32% memory)...
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER OUT     azure-arm: Waiting for the Windows Modules Installer to exit...
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 packer: 2020/04/30 23:38:58 [INFO] command 'PowerShell -ExecutionPolicy Bypass -OutputFormat Text -File C:/Windows/Temp/packer-windows-update-elevated.ps1' exited with code: 101
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER OUT ==> azure-arm: Restarting the machine...
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 packer: 2020/04/30 23:38:58 [INFO] RPC endpoint: Communicator ended with: 101
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 [INFO] 1672 bytes written for 'stdout'
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 [INFO] 0 bytes written for 'stderr'
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 [INFO] RPC client: Communicator ended with: 101
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 [INFO] RPC endpoint: Communicator ended with: 101
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER OUT ==> azure-arm: Waiting for machine to become available...
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 packer-provisioner-windows-update: 2020/04/30 23:38:58 [INFO] 1672 bytes written for 'stdout'
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 packer-provisioner-windows-update: 2020/04/30 23:38:58 [INFO] 0 bytes written for 'stderr'
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 packer-provisioner-windows-update: 2020/04/30 23:38:58 [INFO] RPC client: Communicator ended with: 101
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 packer: 2020/04/30 23:38:58 [INFO] starting remote command: shutdown.exe -f -r -t 0 -c "packer restart"
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 packer: 2020/04/30 23:38:58 [INFO] command 'shutdown.exe -f -r -t 0 -c "packer restart"' exited with code: 0
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 packer: 2020/04/30 23:38:58 [INFO] RPC endpoint: Communicator ended with: 0
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 [INFO] 0 bytes written for 'stderr'
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 [INFO] 0 bytes written for 'stdout'
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER OUT ==> azure-arm: A system shutdown is in progress.(1115)
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 [INFO] RPC client: Communicator ended with: 0
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 [INFO] RPC endpoint: Communicator ended with: 0
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 packer-provisioner-windows-update: 2020/04/30 23:38:58 [INFO] 0 bytes written for 'stdout'
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 packer-provisioner-windows-update: 2020/04/30 23:38:58 [INFO] 0 bytes written for 'stderr'
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 packer-provisioner-windows-update: 2020/04/30 23:38:58 [INFO] RPC client: Communicator ended with: 0
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:59 packer: 2020/04/30 23:38:59 [INFO] starting remote command: shutdown.exe -f -r -t 60 -c "packer restart test"
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:59 packer: 2020/04/30 23:38:59 [INFO] command 'shutdown.exe -f -r -t 60 -c "packer restart test"' exited with code: 1115
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:59 packer: 2020/04/30 23:38:59 [INFO] RPC endpoint: Communicator ended with: 1115
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:59 [INFO] 0 bytes written for 'stdout'
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:59 [INFO] 40 bytes written for 'stderr'
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:59 [INFO] RPC client: Communicator ended with: 1115
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:59 [INFO] RPC endpoint: Communicator ended with: 1115
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:59 packer-provisioner-windows-update: 2020/04/30 23:38:59 [INFO] 40 bytes written for 'stderr'
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:59 packer-provisioner-windows-update: 2020/04/30 23:38:59 [INFO] 0 bytes written for 'stdout'
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:59 packer-provisioner-windows-update: 2020/04/30 23:38:59 [INFO] RPC client: Communicator ended with: 1115
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:59 packer-provisioner-windows-update: 2020/04/30 23:38:59 Retryable error: Machine not yet available (exit status 1115)
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER OUT Build 'azure-arm' errored: unexpected EOF
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER OUT 
```
#### <a name="cause"></a>Причина

Нехватка ресурсов. Эта проблема обычно возникает при работе Центр обновления Windows с использованием размера виртуальной машины сборки по умолчанию D1_V2.

#### <a name="solution"></a>Решение

Увеличьте размер виртуальной машины сборки.

### <a name="builds-finished-but-no-artifacts-were-created"></a>Сборки завершены, но артефакты не созданы

#### <a name="error"></a>Ошибка

```text
[a170b40d-2d77-4ac3-8719-72cdc35cf889] PACKER OUT Build 'azure-arm' errored: Future#WaitForCompletion: context has been cancelled: StatusCode=200 -- Original Error: context deadline exceeded
[a170b40d-2d77-4ac3-8719-72cdc35cf889] PACKER ERR ==> Some builds didn't complete successfully and had errors:
[a170b40d-2d77-4ac3-8719-72cdc35cf889] PACKER OUT 
[a170b40d-2d77-4ac3-8719-72cdc35cf889] PACKER ERR 2020/04/30 22:29:23 machine readable: azure-arm,error []string{"Future#WaitForCompletion: context has been cancelled: StatusCode=200 -- Original Error: context deadline exceeded"}
[a170b40d-2d77-4ac3-8719-72cdc35cf889] PACKER OUT ==> Some builds didn't complete successfully and had errors:
[a170b40d-2d77-4ac3-8719-72cdc35cf889] PACKER ERR ==> Builds finished but no artifacts were created.
[a170b40d-2d77-4ac3-8719-72cdc35cf889] PACKER OUT --> azure-arm: Future#WaitForCompletion: context has been cancelled: StatusCode=200 -- Original Error: context deadline exceeded
[a170b40d-2d77-4ac3-8719-72cdc35cf889] PACKER ERR 2020/04/30 22:29:23 Cancelling builder after context cancellation context canceled
[a170b40d-2d77-4ac3-8719-72cdc35cf889] PACKER OUT 
[a170b40d-2d77-4ac3-8719-72cdc35cf889] PACKER ERR 2020/04/30 22:29:23 [INFO] (telemetry) Finalizing.
[a170b40d-2d77-4ac3-8719-72cdc35cf889] PACKER OUT ==> Builds finished but no artifacts were created.
[a170b40d-2d77-4ac3-8719-72cdc35cf889] PACKER ERR 2020/04/30 22:29:24 waiting for all plugin processes to complete...
Done exporting Packer logs to Azure for Packer prefix: [a170b40d-2d77-4ac3-8719-72cdc35cf889] PACKER OUT
```
#### <a name="cause"></a>Причина

Время ожидания истекло из – за ожидания создания необходимых ресурсов Azure.

#### <a name="solution"></a>Решение

Перезапустите сборку, чтобы повторить попытку.

### <a name="resource-not-found"></a>Ресурс не найден

#### <a name="error"></a>Ошибка

```text
  "provisioningState": "Succeeded",
  "lastRunStatus": {
   "startTime": "2020-05-01T00:13:52.599326198Z",
   "endTime": "2020-05-01T00:15:13.62366898Z",
   "runState": "Failed",
   "message": "network.InterfacesClient#UpdateTags: Failure responding to request: StatusCode=404 -- Original Error: autorest/azure: Service returned an error. Status=404 Code=\"ResourceNotFound\" Message=\"The Resource 'Microsoft.Network/networkInterfaces/aibpls7lz2e.nic.4609d697-be0a-4cb0-86af-49b6fe877fe1' under resource group 'IT_aibImageRG200_window2019VnetTemplate01_9988723b-af56-413a-9006-84130af0e9df' was not found.\""
  },
```
#### <a name="cause"></a>Причина

Отсутствуют разрешения.

#### <a name="solution"></a>Решение

Повторная проверка построителя образов Azure имеет все необходимые разрешения. 

Дополнительные сведения о настройке разрешений см. в статье [Настройка разрешений службы Azure Image Builder с помощью Azure CLI](image-builder-permissions-cli.md) или [Настройка разрешений службы Azure Image Builder с помощью PowerShell](image-builder-permissions-powershell.md) .

### <a name="sysprep-timing"></a>Время работы Sysprep

#### <a name="error"></a>Ошибка

```text
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: Write-Output '>>> Waiting for GA Service (RdAgent) to start ...'
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: while ((Get-Service RdAgent) -and ((Get-Service RdAgent).Status -ne 'Running')) { Start-Sleep -s 5 }
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: Write-Output '>>> Waiting for GA Service (WindowsAzureTelemetryService) to start ...'
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: while ((Get-Service WindowsAzureTelemetryService) -and ((Get-Service WindowsAzureTelemetryService).Status -ne 'Running')) { Start-Sleep -s 5 }
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: Write-Output '>>> Waiting for GA Service (WindowsAzureGuestAgent) to start ...'
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: while ((Get-Service WindowsAzureGuestAgent) -and ((Get-Service WindowsAzureGuestAgent).Status -ne 'Running')) { Start-Sleep -s 5 }
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: Write-Output '>>> Sysprepping VM ...'
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: if( Test-Path $Env:SystemRoot\system32\Sysprep\unattend.xml ) {
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm:   Remove-Item $Env:SystemRoot\system32\Sysprep\unattend.xml -Force
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: }
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: & $Env:SystemRoot\System32\Sysprep\Sysprep.exe /oobe /generalize /quiet /quit
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: while($true) {
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm:   $imageState = (Get-ItemProperty HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Setup\State).ImageState
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm:   Write-Output $imageState
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm:   if ($imageState -eq 'IMAGE_STATE_GENERALIZE_RESEAL_TO_OOBE') { break }
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm:   Start-Sleep -s 5
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: }
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: Write-Output '>>> Sysprep complete ...'
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: >>> Waiting for GA Service (RdAgent) to start ...
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: >>> Waiting for GA Service (WindowsAzureTelemetryService) to start ...
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: >>> Waiting for GA Service (WindowsAzureGuestAgent) to start ...
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: >>> Sysprepping VM ...
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_COMPLETE
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT ==> azure-arm: Get-Service : Cannot find any service with service name 'WindowsAzureGuestAgent'.
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT ==> azure-arm: At C:\DeprovisioningScript.ps1:6 char:9
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT ==> azure-arm: + while ((Get-Service WindowsAzureGuestAgent) -and ((Get-Service Window ...
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT ==> azure-arm: +         ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT ==> azure-arm:     + CategoryInfo          : ObjectNotFound: (WindowsAzureGuestAgent:String) [Get-Service], ServiceCommandException
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT ==> azure-arm:     + FullyQualifiedErrorId : NoServiceFoundForGivenName,Microsoft.PowerShell.Commands.GetServiceCommand
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT ==> azure-arm:
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_UNDEPLOYABLE
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_UNDEPLOYABLE
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_UNDEPLOYABLE
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_UNDEPLOYABLE
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_UNDEPLOYABLE
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_UNDEPLOYABLE
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_UNDEPLOYABLE
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_UNDEPLOYABLE
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_UNDEPLOYABLE
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_UNDEPLOYABLE
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_UNDEPLOYABLE
...
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_UNDEPLOYABLE
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER ERR 2020/05/05 22:26:17 Cancelling builder after context cancellation context canceled
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT Cancelling build after receiving terminated
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER ERR 2020/05/05 22:26:17 packer: 2020/05/05 22:26:17 Cancelling provisioning due to context cancellation: context canceled
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT ==> azure-arm: 
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER ERR 2020/05/05 22:26:17 packer: 2020/05/05 22:26:17 Cancelling hook after context cancellation context canceled
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT ==> azure-arm: The resource group was not created by Packer, deleting individual resources ...
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER ERR ==> azure-arm: The resource group was not created by Packer, deleting individual resources ...
```
#### <a name="cause"></a>Причина

Причиной может быть ошибка времени, обусловленная D1_V2 размером виртуальной машины. Если настройки ограничены и выполняются менее трех секунд, команды Sysprep запускаются построителем образов Azure для отмены инициализации. При отмене подготовки построителя образов Azure команда Sysprep проверяет наличие *виндовсазурегуестажент*, которые могут быть не полностью установлены, что приводит к возникновению проблемы с синхронизацией. 

#### <a name="solution"></a>Решение

Увеличьте размер виртуальной машины. Также можно добавить 60-секундную настройку спящего режима PowerShell, чтобы избежать проблем с синхронизацией.

### <a name="cancelling-builder-after-context-cancellation-context-canceled"></a>Отмена построителя после отмены контекста отмены контекста

#### <a name="error"></a>Ошибка
```text
PACKER ERR 2020/03/26 22:11:23 Cancelling builder after context cancellation context canceled
PACKER OUT Cancelling build after receiving terminated
PACKER ERR 2020/03/26 22:11:23 packer-builder-azure-arm plugin: Cancelling hook after context cancellation context canceled
..
PACKER ERR 2020/03/26 22:11:23 packer-builder-azure-arm plugin: Cancelling provisioning due to context cancellation: context canceled
PACKER ERR 2020/03/26 22:11:25 packer-builder-azure-arm plugin: [ERROR] Remote command exited without exit status or exit signal.
PACKER ERR 2020/03/26 22:11:25 packer-builder-azure-arm plugin: [INFO] RPC endpoint: Communicator ended with: 2300218
PACKER ERR 2020/03/26 22:11:25 [INFO] 148974 bytes written for 'stdout'
PACKER ERR 2020/03/26 22:11:25 [INFO] 0 bytes written for 'stderr'
PACKER ERR 2020/03/26 22:11:25 [INFO] RPC client: Communicator ended with: 2300218
PACKER ERR 2020/03/26 22:11:25 [INFO] RPC endpoint: Communicator ended with: 2300218
```
#### <a name="cause"></a>Причина
Служба Image Builder использует порт 22 (Linux) или 5986 (Windows) для подключения к виртуальной машине сборки. это происходит, когда служба отключается от виртуальной машины сборки во время сборки образа. Причины отключения могут различаться, но включение или Настройка брандмауэров в сценарии может заблокировать указанные выше порты.

#### <a name="solution"></a>Решение
Проверьте свои сценарии на наличие изменений или включения брандмауэра, а также измените протокол SSH или WinRM и убедитесь, что все изменения обеспечивают постоянное подключение между службой и создание виртуальной машины на указанных выше портах. Дополнительные сведения о сетях с помощью Image Builder см. в [требованиях](./image-builder-networking.md).

## <a name="devops-task"></a>Задание DevOps 

### <a name="troubleshooting-the-task"></a>Устранение неполадок в задаче
Задача завершится сбоем, только если во время настройки возникает ошибка, и в этом случае задача сообщит о сбое и оставит промежуточную группу ресурсов с журналами, чтобы можно было найти проблему. 

Чтобы найти журнал, необходимо знать имя шаблона, перейти в конвейер > сбой сборки > детализации задачи AIB DevOps, после чего вы увидите журнал и имя шаблона:
```text
start reading task parameters...
found build at:  /home/vsts/work/r1/a/_ImageBuilding/webapp
end reading parameters
getting storage account details for aibstordot1556933914
created archive /home/vsts/work/_temp/temp_web_package_21475337782320203.zip
Source for image:  { type: 'SharedImageVersion',
  imageVersionId: '/subscriptions/<subscriptionID>/resourceGroups/<rgName>/providers/Microsoft.Compute/galleries/<galleryName>/images/<imageDefName>/versions/<imgVersionNumber>' }
template name:  t_1556938436xxx
``` 

Перейдите на портал, найдите имя шаблона в группе ресурсов, а затем найдите группу ресурсов с IT_ *.
Перейдите в учетную запись хранения > большие двоичные объекты > контейнеры > журналы.

### <a name="troubleshooting-successful-builds"></a>Устранение неполадок сборок
Возможно, в некоторых случаях требуется исследовать успешные сборки и просмотреть журнал. Как уже упоминалось, при успешном завершении сборки образа промежуточная группа ресурсов, содержащая журналы, будет удалена в процессе очистки. Однако, что можно сделать, представляет собой спящий режим после встроенной команды, а затем получает журналы по мере приостановки сборки. Для этого выполните следующие действия.
 
1. Обновите встроенную команду и добавьте: Write-Host/echo "Sleep" — это позволит выполнять поиск в журнале.
2. Добавьте спящий режим по крайней мере для 10mins. можно использовать команду [запуска в спящем режиме](/powershell/module/microsoft.powershell.utility/start-sleep)или `Sleep` Linux.
3. Используйте приведенный выше метод, чтобы указать расположение журнала, а затем сохраните и проверяя журнал, пока он не перейдет в спящий режим.


### <a name="operation-was-canceled"></a>Операция отменена

#### <a name="error"></a>Ошибка

```text
2020-05-05T18:28:24.9280196Z ##[section]Starting: Azure VM Image Builder Task
2020-05-05T18:28:24.9609966Z ==============================================================================
2020-05-05T18:28:24.9610739Z Task         : Azure VM Image Builder Test(Preview)
2020-05-05T18:28:24.9611277Z Description  : Build images using Azure Image Builder resource provider.
2020-05-05T18:28:24.9611608Z Version      : 1.0.18
2020-05-05T18:28:24.9612003Z Author       : Microsoft Corporation
2020-05-05T18:28:24.9612718Z Help         : For documentation, and end to end example, please visit: http://aka.ms/azvmimagebuilderdevops
2020-05-05T18:28:24.9613390Z ==============================================================================
2020-05-05T18:28:26.0651512Z start reading task parameters...
2020-05-05T18:28:26.0673377Z found build at:  d:\a\r1\a\_AppsAndImageBuilder\webApp
2020-05-05T18:28:26.0708785Z end reading parameters
2020-05-05T18:28:26.0745447Z getting storage account details for aibstagstor1565047758
2020-05-05T18:28:29.8812270Z created archive d:\a\_temp\temp_web_package_09737279437949953.zip
2020-05-05T18:28:33.1568013Z Source for image:  { type: 'PlatformImage',
2020-05-05T18:28:33.1584131Z   publisher: 'MicrosoftWindowsServer',
2020-05-05T18:28:33.1585965Z   offer: 'WindowsServer',
2020-05-05T18:28:33.1592768Z   sku: '2016-Datacenter',
2020-05-05T18:28:33.1594191Z   version: '14393.3630.2004101604' }
2020-05-05T18:28:33.1595387Z template name:  t_1588703313152
2020-05-05T18:28:33.1597453Z starting put template...
2020-05-05T18:28:52.9278603Z put template:  Succeeded
2020-05-05T18:28:52.9281282Z starting run template...
2020-05-05T19:33:14.3923479Z ##[error]The operation was canceled.
2020-05-05T19:33:14.3939721Z ##[section]Finishing: Azure VM Image Builder Task
```
#### <a name="cause"></a>Причина

Если сборка не была отменена пользователем, она была отменена агентом пользователя Azure DevOps. Скорее всего, время ожидания в 1 час истекло из-за возможностей Azure DevOps. Если вы используете частный проект и агент, вы получаете 60 минут времени сборки. Если сборка превышает время ожидания, DevOps отменяет выполняемую задачу.

Дополнительные сведения о возможностях и ограничениях Azure DevOps см. в разделе [агенты, размещенные в Майкрософт](/azure/devops/pipelines/agents/hosted#capabilities-and-limitations)
 
#### <a name="solution"></a>Решение

Вы можете разместить собственные агенты DevOps или попытаться сократить время сборки. Например, если вы распространяете в коллекцию общих образов, то реплицируете ее в один регион. Значение, если требуется выполнить репликацию асинхронно. 

### <a name="slow-windows-logon-please-wait-for-the-windows-modules-installer"></a>Задержка входа в Windows: "Подождите, пока установщик модулей Windows

#### <a name="error"></a>Ошибка
После создания образа Windows 10 с помощью программы Image Builder создайте виртуальную машину из образа, вы можете использовать протокол RDP и подождите, пока первый вход повидит синий экран с сообщением:
```text
Please wait for the Windows Modules Installer
```

#### <a name="solution"></a>Решение
Во-первых, в сборке образа проверяется отсутствие необработанных перезагрузок, необходимых для добавления средства настройки перезапуска Windows в качестве последней настройки и завершения установки всех программ. Наконец, добавьте параметр [/mode: VM](/windows-hardware/manufacture/desktop/sysprep-command-line-options) в Sysprep по умолчанию, который AIB использует, см. ниже, "виртуальные машины, созданные из образов AIB", не создаются успешно, > переопределение команд  

 
## <a name="vms-created-from-aib-images-do-not-create-successfully"></a>Виртуальные машины, созданные из образов AIB, не создаются успешно

По умолчанию построитель образов Azure запускает код *отмены подготовки* в конце каждого этапа настройки образа, чтобы *обобщать* образ. Generalize — это процесс, в котором образ настроен для повторного использования при создании нескольких виртуальных машин. Вы можете передать параметры виртуальной машины, такие как имя узла, имя пользователя и т. д. Для Windows Azure Image Builder выполняет *Sysprep*, а для запуска построителя образов Azure для Linux `waagent -deprovision` . 

Для Windows в построителе образов Azure используется универсальная команда Sysprep. Однако это может оказаться непригодным для всех успешных обобщения Windows. Построитель образов Azure позволяет настроить команду Sysprep. Примечание. построитель образов Azure — это средство автоматизации образов. Она отвечает за успешное выполнение команды Sysprep. Однако для повторного использования образа могут потребоваться разные команды Sysprep. Для Linux в Azure Image Builder используется универсальная `waagent -deprovision+user` команда. Дополнительные сведения см. в разделе [Документация по Microsoft Azure Linux Agent](https://github.com/Azure/WALinuxAgent#command-line-options).

Если выполняется миграция существующей настройки и используются разные команды Sysprep и waagent, можно попробовать выполнить универсальные команды Image Builder. В случае сбоя создания виртуальной машины используйте предыдущие команды Sysprep/waagent.

> [!NOTE]
> Если AIB создает пользовательский образ Windows и создает на его основе виртуальную машину, то после этого вы обнаружите, что ВИРТУАЛЬная машина не будет успешно создана (например, команда создания виртуальной машины не завершается и не истечет время ожидания), необходимо ознакомиться с документацией по Windows Server Sysprep. Вы также можете отправить запрос в службу поддержки с помощью команды службы поддержки пользователей Windows Server Sysprep, которая может устранить неполадки и дать рекомендации по правильной команде Sysprep.

### <a name="command-locations-and-filenames"></a>Расположение команд и имена файлов

Windows:

```
c:\DeprovisioningScript.ps1
```

Linux:
```bash
/tmp/DeprovisioningScript.sh
```

### <a name="sysprep-command-windows"></a>Команда Sysprep: Windows

```PowerShell
Write-Output '>>> Waiting for GA Service (RdAgent) to start ...'
while ((Get-Service RdAgent).Status -ne 'Running') { Start-Sleep -s 5 }
Write-Output '>>> Waiting for GA Service (WindowsAzureTelemetryService) to start ...'
while ((Get-Service WindowsAzureTelemetryService) -and ((Get-Service WindowsAzureTelemetryService).Status -ne 'Running')) { Start-Sleep -s 5 }
Write-Output '>>> Waiting for GA Service (WindowsAzureGuestAgent) to start ...'
while ((Get-Service WindowsAzureGuestAgent).Status -ne 'Running') { Start-Sleep -s 5 }
Write-Output '>>> Sysprepping VM ...'
if( Test-Path $Env:SystemRoot\system32\Sysprep\unattend.xml ) {
  Remove-Item $Env:SystemRoot\system32\Sysprep\unattend.xml -Force
}
& $Env:SystemRoot\System32\Sysprep\Sysprep.exe /oobe /generalize /quiet /quit
while($true) {
  $imageState = (Get-ItemProperty HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Setup\State).ImageState
  Write-Output $imageState
  if ($imageState -eq 'IMAGE_STATE_GENERALIZE_RESEAL_TO_OOBE') { break }
  Start-Sleep -s 5
}
Write-Output '>>> Sysprep complete ...'
```

### <a name="deprovision-command-linux"></a>Команда отмены инициализации: Linux

```bash
/usr/sbin/waagent -force -deprovision+user && export HISTSIZE=0 && sync
```

### <a name="overriding-the-commands"></a>Переопределение команд

Чтобы переопределить эти команды, используйте PowerShell или профилировщики скриптов оболочки, чтобы создать командные файлы с точным именем файла и разместить их в указанных ранее каталогах. Построитель образов Azure считывает эти команды и выходные данные записываются в файл *настройки. log*.

## <a name="getting-support"></a>Получение поддержки
Если вы ссылались на руководство и по-прежнему не можете устранить неполадки, можно открыть обращение в службу поддержки. При этом выберите раздел "правильный продукт и поддержка", чтобы сделать это, применяя службу поддержки сборщика образов виртуальных машин Azure.

Выбор варианта продукта:
```bash
Product Family: Azure
Product: Virtual Machine Running (Window\Linux)
Support Topic: Azure Features
Support Subtopic: Azure Image Builder
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в разделе [Общие сведения о построителе образов Azure](../image-builder-overview.md).
