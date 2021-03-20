---
title: Задача DevOps службы Azure Image Builder
description: Задача DevOps Azure для вставки артефактов сборки в образ виртуальной машины, чтобы можно было установить и настроить приложение и ОС.
author: danielsollondon
ms.author: danis
ms.date: 01/27/2021
ms.topic: article
ms.service: virtual-machines
ms.subservice: image-builder
ms.collection: linux
ms.openlocfilehash: d02a5c6bc194009d459647721dab16be0dcade84
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101670482"
---
# <a name="azure-image-builder-service-devops-task"></a>Задача DevOps службы Azure Image Builder

В этой статье показано, как использовать задачу Azure DevOps для вставки артефактов сборки в образ виртуальной машины, чтобы можно было установить и настроить приложение и ОС.

## <a name="devops-task-versions"></a>Версии задач DevOps
Существует две задачи DevOps в построителе образов виртуальных машин Azure (AIB):

* [Задача AIB "стабильной"](https://marketplace.visualstudio.com/items?itemName=AzureImageBuilder.devOps-task-for-azure-image-builder), это последняя стабильная сборка, которая была протестирована, а телеметрии не содержит проблем. 


* [Нестабильная задача AIB](https://marketplace.visualstudio.com/items?itemName=AzureImageBuilder.devOps-task-for-azure-image-builder-canary). Это позволяет нам добавлять последние обновления и функции, позволяя клиентам тестировать их, прежде чем повысить ее до задачи «стабильной». Если проблем нет, и в данных телеметрии не отображается никаких проблем, в течение примерно 1 недели мы будем повышать код задачи до "стабильного". 

## <a name="prerequisites"></a>Предварительные условия

* Установите [стабильную задачу DevOps из Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=AzureImageBuilder.devOps-task-for-azure-image-builder).
* Необходимо иметь учетную запись VSTS DevOps и конвейер сборки, созданный
* Зарегистрируйте и включите требования к функциям построителя образов в подписке, используемой конвейерами:
    * [AZ PowerShell](../windows/image-builder-powershell.md#register-features)
    * [AZ CLI](../windows/image-builder.md#register-the-features)
    
* Создайте стандартную учетную запись хранения Azure в исходной группе ресурсов, можно использовать другую группу ресурсов или учетные записи хранения. Учетная запись хранения используется для перемещения артефактов сборки из задачи DevOps в образ.

    ```powerShell
    # Az PowerShell
    $timeInt=$(get-date -UFormat "%s")
    $storageAccName="aibstorage"+$timeInt
    $location=westus
    # create storage account and blob in resource group
    New-AzStorageAccount -ResourceGroupName $strResourceGroup -Name $storageAccName -Location $location -SkuName Standard_LRS
    ```

    ```bash
    # Az CLI
    location=westus
    scriptStorageAcc=aibstordot$(date +'%s')
    # create storage account and blob in resource group
    az storage account create -n $scriptStorageAcc -g $strResourceGroup -l $location --sku Standard_LRS
    ```

## <a name="add-task-to-release-pipeline"></a>Добавить задачу в конвейер выпуска

Выбор **изменения конвейера выпуска**  >  

В агенте пользователя выберите *+* Добавить, а затем найдите **Построитель образов**. Выберите **Добавить**.

Задайте следующие свойства задачи:

### <a name="azure-subscription"></a>Подписка Azure

В раскрывающемся меню выберите подписку, которую должен запустить построитель образов. Используйте ту же подписку, в которой находятся исходные образы и где будут распространяться образы. Необходимо авторизовать участника построителя образов доступ к подписке или группе ресурсов.

### <a name="resource-group"></a>Группа ресурсов

Используйте группу ресурсов, в которой будет храниться артефакт временного шаблона образа. При создании артефакта шаблона создается дополнительная группа ресурсов для временной построителя образов `IT_<DestinationResourceGroup>_<TemplateName>_guid` . В группе временных ресурсов хранятся метаданные образа, такие как скрипты. В конце задачи удаляется артефакт шаблона образа и группа ресурсов "временная построитель образов".
 
### <a name="location"></a>Расположение

Расположение — это регион, в котором будет выполняться построитель изображений. Поддерживается только заданное число [регионов](../image-builder-overview.md#regions) . Исходные образы должны находиться в этом расположении. Например, если вы используете коллекцию общих образов, реплика должна существовать в этом регионе.

### <a name="managed-identity-required"></a>Управляемое удостоверение (обязательно)
Для работы с Image Builder требуется управляемое удостоверение, которое используется для чтения исходных пользовательских образов, подключения к службе хранилища Azure и создания пользовательских образов. Подробнее см. [здесь](../image-builder-overview.md#permissions).

### <a name="vnet-support"></a>Поддержка виртуальной сети

В настоящее время задача DevOps не поддерживает указание существующей подсети. Это относится к плану, но если вы хотите использовать существующую виртуальную сеть, можно использовать шаблон ARM с шаблоном построителя образов, вложенный в. Дополнительные сведения см. в примерах шаблона построителя образов Windows, а также с помощью команды [AZ AIB PowerShell](../windows/image-builder-powershell.md).

### <a name="source"></a>Источник

Исходные образы должны иметь Поддерживаемые операционные системы Image Builder. Вы можете выбрать существующие пользовательские образы в том же регионе, в котором выполняется построитель образов:
* Управляемый образ — необходимо передать resourceId, например:
    ```json
    /subscriptions/<subscriptionID>/resourceGroups/<rgName>/providers/Microsoft.Compute/images/<imageName>
    ```
* Коллекция общих образов Azure — необходимо передать идентификатор resourceId версии образа, например:
    ```json
    /subscriptions/$subscriptionID/resourceGroups/$sigResourceGroup/providers/Microsoft.Compute/galleries/$sigName/images/$imageDefName/versions/<versionNumber>
    ```

    Если вам нужно получить последнюю версию коллекции общих образов, можно создать задачу AZ PowerShell или AZ CLI, чтобы получить последнюю версию и задать переменную DevOps. Используйте переменную в задаче AZ VM Image Builder DevOps. Дополнительные сведения см. в [примерах](https://github.com/danielsollondon/azvmimagebuilder/tree/master/solutions/8_Getting_Latest_SIG_Version_ResID#getting-the-latest-image-version-resourceid-from-shared-image-gallery).

* Каталоге Базовый образ есть раскрывающийся список популярных изображений. в этом случае всегда будет использоваться последняя версия поддерживаемой ОС. 

    Если базовый образ отсутствует в списке, можно указать точное изображение с помощью `Publisher:Offer:Sku` .

    Версия базового образа (необязательно). Вы можете указать версию образа, которую вы хотите использовать, по умолчанию — `latest` .

### <a name="customize"></a>Настройка

#### <a name="provisioner"></a>Средство подготовки

Изначально поддерживаются два настраиваемых настройщика — **Shell** и **PowerShell**. Поддерживается только встроенный. Если вы хотите загрузить скрипты, то для этого можно передать встроенные команды.

Для вашей ОС выберите PowerShell или Shell.

#### <a name="windows-update-task"></a>Задача Центр обновления Windows

Только для Windows задача выполняется Центр обновления Windows в конце настроек. Он обрабатывает необходимые перезагрузки.

Выполняется следующая конфигурация Центр обновления Windows:
```json
    "type": "WindowsUpdate",
    "searchCriteria": "IsInstalled=0",
    "filters": [
        "exclude:$_.Title -like '*Preview*'",
        "include:$true"
```
Он устанавливает важные и рекомендуемые обновления Windows, которые не являются предварительными.

#### <a name="handling-reboots"></a>Обработка перезагрузок
В настоящее время задача DevOps не поддерживает перезагрузку сборок Windows, если вы попытаетесь выполнить перезагрузку с помощью кода PowerShell, сборка завершится ошибкой. Однако можно использовать код для перезагрузки сборок Linux.

#### <a name="build-path"></a>Путь сборки

Задача разработана так, чтобы иметь возможность внедрять в образ артефакты выпуска сборки DevOps. Чтобы выполнить эту работу, необходимо настроить конвейер сборки. Во время установки конвейера выпуска необходимо добавить репозиторий артефактов сборки.

:::image type="content" source="./media/image-builder-devops-task/add-artifact.png" alt-text="Выберите Добавить артефакт в конвейер выпуска.":::

Нажмите кнопку **путь сборки** , чтобы выбрать папку сборки, которую нужно поместить в образ. Задача «построитель изображений» копирует все файлы и каталоги в ней. При создании образа построитель образов развертывает файлы и каталоги в разные пути в зависимости от операционной системы.

> [!IMPORTANT]
> При добавлении артефакта репозитория может оказаться, что каталог имеет префикс подчеркивания *_*. Подчеркивание может вызвать проблемы с встроенными командами. Используйте соответствующие кавычки в командах.
> 

В следующем примере показано, как это работает:

:::image type="content" source="./media/image-builder-devops-task/build-artifacts.png" alt-text="Структура каталогов, показывающая иерархию.":::


* Windows — файлы существуют в `C:\` . Создается каталог с именем, `buildArtifacts` включающий `webapp` каталог.

* Linux — файлы существуют в  `/tmp` . `webapp`Создается каталог, который включает все файлы и каталоги. Необходимо переместить файлы из этого каталога. В противном случае они будут удалены, так как они находятся во временном каталоге.

#### <a name="inline-customization-script"></a>Скрипт встроенной настройки

* Windows. можно вводить встроенные команды PowerShell, разделенные запятыми. Если вы хотите выполнить сценарий в каталоге сборки, можно использовать:

    ```PowerShell
    & 'c:\buildArtifacts\webapp\webconfig.ps1'
    ```

   Вы можете ссылаться на несколько скриптов или добавить дополнительные команды, например:

    ```PowerShell
    & 'c:\buildArtifacts\webapp\webconfig.ps1'
    & 'c:\buildArtifacts\webapp\installAgent.ps1'
    ```
* Системы Linux в Linux. артефакты сборки помещаются в `/tmp` каталог. Однако во многих ОС Linux при перезагрузке содержимое каталога/tmp удаляется. Если вы хотите, чтобы артефакты существовали в образе, необходимо создать другой каталог и скопировать его.  Пример:

    ```bash
    sudo mkdir /lib/buildArtifacts
    sudo cp -r "/tmp/_ImageBuilding/webapp" /lib/buildArtifacts/.
    ```
    
    Если вы нормально используете каталог "/tmp", можно использовать приведенный ниже код для выполнения скрипта.
    
    ```bash
    # grant execute permissions to execute scripts
    sudo chmod +x "/tmp/_ImageBuilding/webapp/coreConfig.sh"
    echo "running script"
    sudo . "/tmp/AppsAndImageBuilderLinux/_WebApp/coreConfig.sh"
    ```
    
#### <a name="what-happens-to-the-build-artifacts-after-the-image-build"></a>Что происходит с артефактами сборки после сборки образа?

> [!NOTE]
> Построитель изображений не удаляет артефакты сборки автоматически, поэтому настоятельно рекомендуется всегда иметь код для удаления артефактов сборки.
> 

* Windows-Image Builder развертывает файлы в `c:\buildArtifacts` каталоге. Каталог сохранен, необходимо удалить каталог. Его можно удалить в выполняемом скрипте. Пример:

    ```PowerShell
    # Clean up buildArtifacts directory
    Remove-Item -Path "C:\buildArtifacts\*" -Force -Recurse
    
    # Delete the buildArtifacts directory
    Remove-Item -Path "C:\buildArtifacts" -Force 
    ```
    
* Linux — артефакты сборки помещаются в `/tmp` каталог. Однако во многих ОС Linux при перезагрузке `/tmp` содержимое каталога удаляется. Рекомендуется использовать код для удаления содержимого и не полагаться на ОС для удаления содержимого. Пример:

    ```bash
    sudo rm -R "/tmp/AppsAndImageBuilderLinux"
    ```
    
#### <a name="total-length-of-image-build"></a>Общая длина сборки образа

Общая длина еще не может быть изменена в задаче конвейера DevOps. По умолчанию используется значение 240 минут. Если вы хотите увеличить [буилдтимеаутинминутес](./image-builder-json.md#properties-buildtimeoutinminutes), то можете использовать задачу AZ CLI в конвейере выпуска. Настройте задачу, чтобы скопировать шаблон и отправить его. Пример см. в описании этого [решения](https://github.com/danielsollondon/azvmimagebuilder/tree/master/solutions/4_Using_ENV_Variables#using-environment-variables-and-parameters-with-image-builder)или с помощью команды AZ PowerShell.


#### <a name="storage-account"></a>Учетная запись хранения

Выберите учетную запись хранения, созданную в предварительных требованиях. Если вы не видите его в списке, построитель изображений не имеет разрешений на доступ к нему.

При запуске сборки построитель изображений создаст контейнер с именем `imagebuilder-vststask` . Контейнер — это место, где хранятся артефакты сборки из репозитория.

> [!NOTE]
> После каждой сборки необходимо вручную удалить учетную запись хранения или контейнер.
>

### <a name="distribute"></a>Распространение

Поддерживаются три типа распространения.

#### <a name="managed-image"></a>Управляемый образ

* Идентификатора
    ```bash
    /subscriptions/<subscriptionID>/resourceGroups/<rgName>/providers/Microsoft.Compute/images/<imageName>
    ```

* Расположения

#### <a name="azure-shared-image-gallery"></a>Коллекция общих образов Azure

Коллекция общих образов уже **должна** существовать.

* Идентификатора 
    ```bash
    /subscriptions/<subscriptionID>/resourceGroups/<rgName>/providers/Microsoft.Compute/galleries/<galleryName>/images/<imageDefName>
    ```

* Регионы: список регионов, разделенных запятыми. Например, westus, eastus, centralus

#### <a name="vhd"></a>VHD 

Вы не можете передать какие-либо значения в это, построитель образов выдаст VHD во временную группу ресурсов построителя образов, `IT_<DestinationResourceGroup>_<TemplateName>` в контейнере *VHD* . При запуске сборки выпуска Image Builder порождает журналы. По завершении он выдаст URL-адрес виртуального жесткого диска.

### <a name="optional-settings"></a>Необязательные параметры

* [Размер виртуальной машины](image-builder-json.md#vmprofile) . размер виртуальной машины можно переопределить по умолчанию *Standard_D1_v2*. Вы можете переопределить, чтобы сократить общее время настройки или создать образы, зависящие от определенных размеров виртуальных машин, таких как GPU/HPC и т. д.

## <a name="how-it-works"></a>Принцип работы

При создании выпуска задача создает контейнер в учетной записи хранения с именем *имажебуилдер-встстаск*. Он zips и загружает артефакты сборки и создает маркер SAS для ZIP-файла.

Задача использует свойства, переданные задаче, для создания артефакта шаблона построителя образов. Задача выполняет следующие действия:
* Скачивает ZIP-файл артефакта сборки и все связанные с ним сценарии. Файлы сохраняются в учетной записи хранения во временной группе ресурсов построителя образов `IT_<DestinationResourceGroup>_<TemplateName>` .
* Создает *T_* шаблона с префиксом 10 цифр и монотонным целым числом. Шаблон будет сохранен в выбранной группе ресурсов. Шаблон существует на время сборки в группе ресурсов. 

Выходные данные примера:

```text
start reading task parameters...
found build at:  /home/vsts/work/r1/a/_ImageBuilding/webapp
end reading parameters
getting storage account details for aibstordot1556933914
created archive /home/vsts/work/_temp/temp_web_package_21475337782320203.zip
Source for image:  { type: 'SharedImageVersion',
  imageVersionId: '/subscriptions/<subscriptionID>/resourceGroups/<rgName>/providers/Microsoft.Compute/galleries/<galleryName>/images/<imageDefName>/versions/<imgVersionNumber>' }
template name:  t_1556938436xxx
starting put template...
```

При запуске сборки образа состояние выполнения указывается в журналах выпусков:

```text
starting run template...
```

После завершения сборки образа отобразятся выходные данные, аналогичные приведенным ниже.

```text
2019-05-06T12:49:52.0558229Z starting run template...
2019-05-06T13:36:33.8863094Z run template:  Succeeded
2019-05-06T13:36:33.8867768Z getting runOutput for  SharedImage_distribute
2019-05-06T13:36:34.6652541Z ==============================================================================
2019-05-06T13:36:34.6652925Z ## task output variables ##
2019-05-06T13:36:34.6658728Z $(imageUri) =  /subscriptions/<subscriptionID>/resourceGroups/aibwinsig/providers/Microsoft.Compute/galleries/my22stSIG/images/winWAppimages/versions/0.23760.13763
2019-05-06T13:36:34.6659989Z ==============================================================================
2019-05-06T13:36:34.6663500Z deleting template t_1557146959485...
2019-05-06T13:36:34.6673713Z deleting storage blob imagebuilder-vststask\webapp/18-1/webapp_1557146958741.zip
2019-05-06T13:36:34.9786039Z blob imagebuilder-vststask\webapp/18-1/webapp_1557146958741.zip is deleted
2019-05-06T13:38:37.4884068Z delete template:  Succeeded
```

Шаблон изображения `IT_<DestinationResourceGroup>_<TemplateName>` удален.

Вы можете взять переменную VSTS "$ (Имажеури)" и использовать ее в следующей задаче или просто использовать значение и создать виртуальную машину.

## <a name="output-devops-variables"></a>Выходные переменные DevOps

Pub/предложение/SKU/версия исходного образа Marketplace:
* $ (Пирпублишер)
* $ (Пироффер)
* $ (Пирску)
* $ (Пирверсион)

URI изображения — ResourceID распределенного образа:
* $ (Имажеури)

## <a name="faq"></a>Вопросы и ответы

### <a name="can-i-use-an-existing-image-template-i-have-already-created-outside-of-devops"></a>Можно ли использовать существующий шаблон образа, который я уже создал, за пределами DevOps?

Сейчас в настоящее время нет.

### <a name="can-i-specify-the-image-template-name"></a>Можно ли указать имя шаблона образа?

Нет. Используется уникальное имя шаблона, а затем удаляется.

### <a name="the-image-builder-failed-how-can-i-troubleshoot"></a>Не удалось выполнить построитель изображений. Как можно устранить неполадки?

В случае сбоя сборки задача DevOps не удаляет промежуточную группу ресурсов. Вы можете получить доступ к промежуточной группе ресурсов, содержащей журнал настройки сборки.

Вы увидите сообщение об ошибке в журнале DevOps для задачи "Построитель образов виртуальных машин" и просмотрите расположение настройки. log. Пример:

:::image type="content" source="./media/image-builder-devops-task/devops-task-error.png" alt-text="Пример ошибки задачи DevOps, которая показывает сбой.":::

Дополнительные сведения об устранении неполадок см. в статье [Устранение неполадок службы Azure Image Builder](image-builder-troubleshoot.md). 

После изучения сбоя можно удалить промежуточную группу ресурсов. Сначала удалите артефакт ресурса шаблона образа. Артефакт имеет префикс *T_* , и его можно найти в журнале сборки задач DevOps:

```text
...
Source for image:  { type: 'SharedImageVersion',
  imageVersionId: '/subscriptions/<subscriptionID>/resourceGroups/<rgName>/providers/Microsoft.Compute/galleries/<galleryName>/images/<imageDefName>/versions/<imgVersionNumber>' }
...
template name:  t_1556938436xxx
...

```

Артефакт ресурса шаблона образа находится в группе ресурсов, указанной изначально в задаче. По завершении устранения неполадок удалите артефакт. При удалении с помощью портал Azure в группе ресурсов выберите **Показать скрытые типы**, чтобы просмотреть артефакт.


## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в разделе [Общие сведения о построителе образов Azure](../image-builder-overview.md).
