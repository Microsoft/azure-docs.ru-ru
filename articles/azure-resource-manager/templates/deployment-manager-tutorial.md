---
title: Использование диспетчера развертывания Azure для развертывания шаблонов
description: Узнайте, как использовать шаблоны Resource Manager с помощью диспетчера развертывания Azure для развертывания ресурсов Azure.
author: mumian
ms.date: 08/25/2020
ms.topic: tutorial
ms.author: jgao
ms.openlocfilehash: 95d5067eccff5c847588834061db8454f75e55d7
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99627588"
---
# <a name="tutorial-use-azure-deployment-manager-with-resource-manager-templates-public-preview"></a>Руководство по использованию диспетчера развертывания Azure с шаблонами Resource Manager (общедоступная предварительная версия)

Узнайте, как использовать [диспетчер развертывания Azure](./deployment-manager-overview.md) для развертывания приложений в нескольких регионах. Если вы предпочитаете более быстрый подход, ознакомьтесь с [кратким руководством по использованию диспетчера развертывания Azure](https://github.com/Azure-Samples/adm-quickstart), в рамках которого вы сможете создать необходимые конфигурации в своей подписке и настроить артефакты для развертывания приложения в нескольких регионах. В этом кратком руководстве выполняются те же задачи, что и в этом.

Чтобы использовать диспетчер развертывания, необходимо создать два шаблона:

* **Шаблон топологии**: описывает ресурсы Azure для создания приложений и путь их развертывания.
* **Шаблон развертывания**: описывает действия, выполняемые при развертывании приложений.

> [!IMPORTANT]
> Если ваша подписка помечена для раннего выпуска, что позволяет тестировать новые функции Azure, вы можете использовать диспетчер развертывания Azure только для развертывания в регионах, поддерживающих ранний выпуск.

В рамках этого руководства рассматриваются следующие задачи:

> [!div class="checklist"]
> * Ознакомление со сценарием
> * Скачивание учебных файлов
> * Подготовка артефактов.
> * Создание определяемого пользователем управляемого удостоверения.
> * Создание шаблона топологии службы
> * Создание шаблона развертывания
> * Развертывание шаблонов.
> * Проверка развертывания
> * Развертывание более новой версии.
> * Очистка ресурсов

Дополнительные ресурсы:

* [Справочник по REST API диспетчера развертывания Azure](/rest/api/deploymentmanager/).
* [Руководство. Use health check in Azure Deployment Manager](./deployment-manager-tutorial-health-check.md) (Учебник по использованию проверки работоспособности в диспетчере развертывания Azure).

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Предварительные условия

Для работы с этим учебником необходимы указанные ниже компоненты.

* Подписка Azure. Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.
* Опыт разработки [шаблонов Azure Resource Manager](overview.md).
* Установите Azure PowerShell. Дополнительные сведения см. в статье [Начало работы с Azure PowerShell](/powershell/azure/get-started-azureps).
* Командлеты диспетчера развертывания. Чтобы установить эти командлеты предварительной версии, вам понадобится последняя версия PowerShellGet. Сведения о получении последней версии см. в статье [Установка PowerShellGet](/powershell/scripting/gallery/installing-psget). После установки PowerShellGet закройте окно PowerShell. Откройте новое окно PowerShell с повышенными привилегиями и выполните следующую команду:

    ```powershell
    Install-Module -Name Az.DeploymentManager
    ```

## <a name="understand-the-scenario"></a>Ознакомление со сценарием

Шаблон топологии службы описывает ресурсы Azure для создания службы и путь их развертывания. Определение топологии службы имеет следующую иерархию:

* Топология службы:
  * Службы
    * Модули службы.

На следующей схеме показана топология службы, используемая в этом руководстве:

![Схема сценария управления диспетчера развертывания Azure](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-scenario-diagram.png)

В регионах "Западная часть США" и "Восточная часть США" расположены две службы. Каждая служба имеет два модуля службы — внешнее веб-приложение и серверная учетная запись хранения. Определения модулей службы содержат ссылки на файлы шаблонов и параметров для создания веб-приложений и учетных записей хранения.

## <a name="download-the-tutorial-files"></a>Скачивание учебных файлов

1. Скачайте [шаблоны и артефакты](https://github.com/Azure/azure-docs-json-samples/raw/master/tutorial-adm/ADMTutorial.zip), используемые в этом руководстве.
1. Распакуйте файлы на локальный компьютер.

В корневой папке есть две папки:

* _ADMTemplates_: содержит шаблоны диспетчера развертывания, включая такие:
  * _CreateADMServiceTopology.json;_
  * _CreateADMServiceTopology.Parameters.json;_
  * _CreateADMRollout.json;_
  * _CreateADMRollout.Parameters.json._
* _ArtifactStore_: содержит артефакты шаблонов и двоичные артефакты. Подробнее см. в разделе [Подготовка артефактов](#prepare-the-artifacts).

Существует два набора шаблонов. Один набор — это шаблоны диспетчера развертывания, которые используются для развертывания топологии службы и выпуска. Другой набор вызывается из модулей службы для создания веб-служб и учетных записей хранения.

## <a name="prepare-the-artifacts"></a>Подготовка артефактов.

Папка ArtifactStore из скачивания содержит две папки:

![Диаграмма источника артефакта диспетчера развертывания Azure](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-artifact-source-diagram.png)

* Папка _templates_: содержит артефакты шаблона. Папки _1.0.0.0_ и _1.0.0.1_ представляют две версии двоичных артефактов. Каждая версия включает папку для каждой службы: _ServiceEUS_ (служба в регионе "Восточная часть США") и _ServiceWUS_ (служба в регионе "Западная часть США"). Каждая служба имеет пару шаблонов и файлов параметров для создания учетной записи хранения и другую пару для создания веб-приложения. Шаблон веб-приложения вызывает сжатый пакет, содержащий файлы веб-приложения. Сжатый файл представляет собой двоичный артефакт, хранящийся в папке двоичных файлов.
* Папка _binaries_ содержит двоичные артефакты. Папки _1.0.0.0_ и _1.0.0.1_ представляют две версии двоичных артефактов. Каждая версия включает есть один ZIP-файл для создания веб-приложения в регионе "Западная часть США", а другой ZIP-файл — для создания веб-приложения в регионе "Восточная часть США".

Две версии (1.0.0.0 и 1.0.0.1) предназначены для [развертывания версии](#deploy-the-revision). Несмотря на то, что как артефакты шаблонов, так и двоичные артефакты имеют две версии, только двоичные артефакты различаются между двумя версиями. На практике двоичные артефакты чаще обновляются по сравнению с артефактами шаблонов.

1. Откройте _\ArtifactStore\templates\1.0.0.0\ServiceWUS\CreateStorageAccount.json_ в текстовом редакторе. Это базовый шаблон для создания учетной записи хранения.
1. Откройте _\ArtifactStore\templates\1.0.0.0\ServiceWUS\CreateWebApplication.json_.

    ![Руководство по работе с диспетчером развертывания Azure: создание шаблона веб-приложения](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-create-web-application-packageuri.png)

    Шаблон вызывает пакет развертывания, который содержит файлы веб-приложения. Для работы с этим руководством используется сжатый пакет, который содержит только файл _index.html_.
1. Откройте _\ArtifactStore\templates\1.0.0.0\ServiceWUS\CreateWebApplicationParameters.json_.

    ![Руководство по работе с диспетчером развертывания Azure: параметры containerRoot для создания шаблона веб-приложения](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-create-web-application-parameters-deploypackageuri.png)

    Значение `deployPackageUri` является путем к пакету развертывания. Параметр содержит переменную `$containerRoot`. Значение `$containerRoot` предоставляется в [шаблоне выпуска](#create-the-rollout-template) путем объединения исходного расположения артефакта SAS, корня артефакта и `deployPackageUri`.
1. Откройте _\ArtifactStore\binaries\1.0.0.0\helloWorldWebAppWUS.zip\index.html_.

    ```html
    <html>
      <head>
        <title>Azure Deployment Manager tutorial</title>
      </head>
      <body>
        <p>Hello world from west U.S.!</p>
        <p>Version 1.0.0.0</p>
      </body>
    </html>
    ```

    В HTML указано расположение и информация о версии. В двоичном файле в папке _1.0.0.1_ отображается _Version 1.0.0.1_. После развертывания службы можно перейти к этим страницам.
1. Ознакомьтесь с другими файлами артефактов. Это поможет вам лучше понять сценарий.

Артефакты шаблонов используются шаблоном топологии службы, а двоичные артефакты используются шаблоном развертывания. Как шаблон топологии, так и шаблон развертывания определяют ресурс Azure источника артефакта, который используется для указания диспетчеру ресурсов на шаблон и двоичные артефакты, применяемые в развертывании. Чтобы упростить руководство, одна учетная запись хранения используется для хранения как артефактов шаблона, так и двоичных артефактов. Оба источника артефакта указывают на одну и ту же учетную запись хранения.

Выполните следующий сценарий PowerShell, чтобы создать группу ресурсов, контейнер хранилища, контейнер больших двоичных объектов, отправить скачанные файлы, а затем создать маркер SAS.

> [!IMPORTANT]
> Параметр `projectName` в скрипте PowerShell используется, чтобы создавать имена для служб Azure, которые будут развернуты в этом руководстве. Различные службы Azure имеют различные требования к именам. Чтобы убедиться, что развертывание прошло успешно, выберите имя длиной не более 12 символов, состоящее только со строчных букв и цифр.
> Сохраните копию имени проекта. Для работы с этим руководством используется то же значение `projectName`.

```azurepowershell
$projectName = Read-Host -Prompt "Enter a project name that is used to generate Azure resource names"
$location = Read-Host -Prompt "Enter the location (i.e. centralus)"
$filePath = Read-Host -Prompt "Enter the folder that contains the downloaded files"


$resourceGroupName = "${projectName}rg"
$storageAccountName = "${projectName}store"
$containerName = "admfiles"
$filePathArtifacts = "${filePath}\ArtifactStore"

New-AzResourceGroup -Name $resourceGroupName -Location $location

$storageAccount = New-AzStorageAccount -ResourceGroupName $resourceGroupName `
  -Name $storageAccountName `
  -Location $location `
  -SkuName Standard_RAGRS `
  -Kind StorageV2

$storageContext = $storageAccount.Context

$storageContainer = New-AzStorageContainer -Name $containerName -Context $storageContext -Permission Off


$filesToUpload = Get-ChildItem $filePathArtifacts -Recurse -File

foreach ($x in $filesToUpload) {
    $targetPath = ($x.fullname.Substring($filePathArtifacts.Length + 1)).Replace("\", "/")

    Write-Verbose "Uploading $("\" + $x.fullname.Substring($filePathArtifacts.Length + 1)) to $($storageContainer.CloudBlobContainer.Uri.AbsoluteUri + "/" + $targetPath)"
    Set-AzStorageBlobContent -File $x.fullname -Container $storageContainer.Name -Blob $targetPath -Context $storageContext | Out-Null
}

$token = New-AzStorageContainerSASToken -name $containerName -Context $storageContext -Permission rl -ExpiryTime (Get-date).AddMonths(1)

$url = $storageAccount.PrimaryEndpoints.Blob + $containerName + $token

Write-Host $url
```

Создайте копию URL-адреса с маркером SAS. Этот URL-адрес необходим для заполнения полей в двух файлах параметров: файле параметров топологии и файле параметров выпуска.

Откройте контейнер на портале Azure и убедитесь, что отправлены как _двоичные файлы_, так и папки _шаблонов_ и файлы.

## <a name="create-the-user-assigned-managed-identity"></a>Создание управляемого удостоверения, назначаемого пользователем

Далее в этом руководстве развертывается выпуск. Для выполнения действий по развертыванию (например, развертывание веб-приложений и учетной записи хранения) требуется управляемое удостоверение, назначаемое пользователем. Этому удостоверению должен быть предоставлен доступ к подписке Azure, в которую развертывается служба, а также требуется достаточное разрешение для завершения развертывания артефакта.

Вам необходимо создать управляемое удостоверение, назначаемое пользователем, и настроить управление доступом для своей подписки.

1. Войдите на [портал Azure](https://portal.azure.com).
1. Создайте [управляемое удостоверение, назначаемое пользователем](../../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal.md).
1. На портале щелкните **Подписки** в меню слева и выберите свою подписку.
1. Выберите **Управление доступом (IAM)** , а затем **Добавление назначения ролей**.
1. Введите или выберите следующие значения:

    ![Руководство по использованию диспетчера развертывания Azure: управление доступом с помощью управляемого удостоверения, назначаемого пользователем](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-access-control.png)

    * **Роль**: предоставьте достаточно разрешений на выполнение развертывания артефакта (веб-приложения и учетные записи хранения). Выберите **участника**, как описано в этом руководстве. На практике следует ограничить права доступа до минимума.
    * **Назначение доступа для**: выберите **Управляемое удостоверение, назначаемое пользователем**.
    * Выберите назначаемое пользователем управляемое удостоверение, которое вы создали.
1. Щелкните **Сохранить**.

## <a name="create-the-service-topology-template"></a>Создание шаблона топологии службы

Откройте _\ADMTemplates\CreateADMServiceTopology.json_.

### <a name="the-parameters"></a>Параметры

Этот шаблон содержит следующие параметры:

* `projectName`. Это имя используется для создания имен ресурсов диспетчера развертывания. Например, если использовать **demo**, имя топологии службы будет иметь вид **demo** ServiceTopology. Имена ресурсов определяются в разделе `variables` шаблона.
* `azureResourcelocation`. Для упрощения работы с руководством все ресурсы совместно используют это расположение, если не указано иное.
* `artifactSourceSASLocation`. Универсальный код ресурса (URI) SAS для контейнера больших двоичных объектов, где файлы параметров и шаблона модулей службы хранятся для развертывания. Подробнее см. в разделе [Подготовка артефактов](#prepare-the-artifacts).
* `templateArtifactRoot`. Путь смещения из контейнера больших двоичных объектов, где хранятся шаблоны и параметры. Значение по умолчанию — _templates/1.0.0.0_. Не изменяйте это значение, если вы не хотите изменять структуру папок, как описано в разделе [Подготовка артефактов](#prepare-the-artifacts). В этом руководстве используются относительные пути. Полный путь формируется путем сцепления `artifactSourceSASLocation`, `templateArtifactRoot` и `templateArtifactSourceRelativePath` (или `parametersArtifactSourceRelativePath`).
* `targetSubscriptionID`. Идентификатор подписки, в которую будут развернуты ресурсы диспетчера развертывания и на которую будут выставляться счета. Используйте идентификатор подписки в этом руководстве.

### <a name="the-variables"></a>Переменные

Раздел переменных определяет имена ресурсов, местоположения Azure для двух служб: `ServiceWUS` и `ServiceEUS`; и пути к артефактам.

![Руководство по использованию диспетчера развертывания Azure: переменные топологии шаблона](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-topology-template-variables.png)

Сравните пути артефакта со структурой папок, которую вы скачали в учетную запись хранения. Обратите внимание, что пути артефактов являются относительными. Полный путь формируется путем сцепления `artifactSourceSASLocation`, `templateArtifactRoot` и `templateArtifactSourceRelativePath` (или `parametersArtifactSourceRelativePath`).

### <a name="the-resources"></a>Ресурсы

На корневом уровне определено два ресурса: *источник артефакта* и *топология службы*.

Определение источника артефакта:

![Руководство по использованию диспетчера развертывания Azure: источник ресурсов артефактов топологии шаблона](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-topology-template-resources-artifact-source.png)

На следующем снимке экрана отображаются только некоторые части определений топологии служб, служб и модулей служб:

![Руководство по использованию диспетчера развертывания Azure: ресурсы шаблона топологии, топология службы](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-topology-template-resources-service-topology.png)

* `artifactSourceId`. Позволяет связать артефакт исходного ресурса с ресурсом топологии службы.
* `dependsOn`. Все ресурсы топологии службы зависят от исходного ресурса артефакта.
* `artifacts`. Указывает артефакты шаблонов. Здесь используются относительные пути. Полный путь создается путем объединения `artifactSourceSASLocation` (определяется в источнике артефакта), `artifactRoot` (определяется в источнике артефакта) и `templateArtifactSourceRelativePath` (или `parametersArtifactSourceRelativePath`).

### <a name="topology-parameters-file"></a>Файл параметров топологии

Создайте файл параметров, используемый с шаблоном топологии.

1. Откройте _файл \ADMTemplates\CreateADMServiceTopology.Parameters.json_ в Visual Studio Code или любом текстовом редакторе.
1. Введите значения параметров:

    * `projectName`. Введите строку из 4–5 символов. Это имя используется для создания уникальных имен ресурсов Azure.
    * `azureResourceLocation`. Если вы не знакомы с расположениями Azure, используйте **centralus** для работы с этим руководством.
    * `artifactSourceSASLocation`. Введите URI SAS в корневой каталог (контейнер больших двоичных объектов), где хранятся файлы параметров и шаблон модулей службы для развертывания.  Подробнее см. в разделе [Подготовка артефактов](#prepare-the-artifacts).
    * `templateArtifactRoot`. Если вы не изменяли структуру папок артефактов, используйте _templates/1.0.0.0_ в этом руководстве.

> [!IMPORTANT]
> Шаблон топологии и шаблон развертывания совместно используют некоторые общие параметры. Они должны иметь общие параметры. Это такие параметры: `projectName`, `azureResourceLocation` и `artifactSourceSASLocation` (оба источника артефактов используют одну и ту же учетную запись хранения в этом учебнике).

## <a name="create-the-rollout-template"></a>Создание шаблона развертывания

Откройте _\ADMTemplates\CreateADMRollout.json_.

### <a name="the-parameters"></a>Параметры

Этот шаблон содержит следующие параметры:

![Руководство по использованию диспетчера развертывания Azure: параметры шаблона выпуска](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-rollout-template-parameters.png)

* `projectName`. Это имя используется для создания имен ресурсов диспетчера развертывания. Например, если использовать **demo**, имя выпуска будет иметь вид **demo** Rollout. Имена определяются в разделе `variables` шаблона.
* `azureResourcelocation`. Чтобы упростить работу с руководством, все ресурсы диспетчера совместно используют это расположение, если не указано иное.
* `artifactSourceSASLocation`. URI SAS для контейнера больших двоичных объектов, где файлы параметров и шаблона модулей службы хранятся для развертывания. Подробнее см. в разделе [Подготовка артефактов](#prepare-the-artifacts).
* `binaryArtifactRoot`. Значение по умолчанию — _binaries/1.0.0.0_. Не изменяйте это значение, если вы не хотите изменять структуру папок, как описано в разделе [Подготовка артефактов](#prepare-the-artifacts). В этом руководстве используются относительные пути. Полный путь создается путем объединения `artifactSourceSASLocation`, `binaryArtifactRoot` и `deployPackageUri`, указанных в файле _CreateWebApplicationParameters.json_. Подробнее см. в разделе [Подготовка артефактов](#prepare-the-artifacts).
* `managedIdentityID`. Управляемое удостоверение, назначаемое пользователем, который выполняет действия развертывания. Дополнительные сведения см. в разделе [Создание управляемого удостоверения, назначаемого пользователем](#create-the-user-assigned-managed-identity).

### <a name="the-variables"></a>Переменные

В разделе `variables` определены имена ресурсов. Убедитесь, что имя топологии службы, имена служб и имена модулей службы соответствуют именам, определенным в [шаблоне топологии](#create-the-service-topology-template).

![Руководство по использованию диспетчера развертывания Azure: переменные шаблона выпуска](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-rollout-template-variables.png)

### <a name="the-resources"></a>Ресурсы

На корневом уровне определено три ресурса: источник артефакта, топология службы и выпуск.

Определение источника артефактов идентично определению шаблона топологии. Дополнительные сведения см. в статье [Руководство. Использование диспетчера развертывания Azure с шаблонами Resource Manager (предварительная версия)](#create-the-service-topology-template).

На следующем снимке экрана показано определение шага `wait`:

![Руководство по использованию диспетчера развертывания Azure: шаг ожидания ресурса шаблона выпуска](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-rollout-template-resources-wait-step.png)

Длительность соответствует [стандарту ISO 8601](https://en.wikipedia.org/wiki/ISO_8601#Durations). **PT1M** (заглавные буквы обязательны) является примером одноминутного ожидания.

На следующем снимке экрана показаны только некоторые части определения развертывания:

![Руководство по использованию диспетчера развертывания Azure: выпуск шаблона ресурсов](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-rollout-template-resources-rollout.png)

* `dependsOn`. Ресурс развертывания зависит от исходного ресурса артефакта, а также от определенных шагов.
* `artifactSourceId`. Позволяет связать артефакт исходного ресурса с ресурсом выпуска.
* `targetServiceTopologyId`. Позволяет связать ресурс топологии службы с ресурсом выпуска.
* `deploymentTargetId`. Это идентификатор ресурса модуля службы для ресурса топологии службы.
* `preDeploymentSteps` и `postDeploymentSteps`: Содержит шаги выпуска. В шаблоне вызывается шаг `wait`.
* `dependsOnStepGroups`. Позволяет настроить зависимости между группами шагов.

### <a name="rollout-parameters-file"></a>Файл параметров выпуска

Создайте файл параметров, используемый с шаблоном выпуска.

1. Откройте _файл \ADMTemplates\CreateADMRollout.Parameters.json_ в Visual Studio Code или любом текстовом редакторе.
1. Введите значения параметров:

    * `projectName`. Введите строку из 4–5 символов. Это имя используется для создания уникальных имен ресурсов Azure.
    * `azureResourceLocation`. Укажите расположение Azure.
    * `artifactSourceSASLocation`. Введите URI SAS в корневой каталог (контейнер больших двоичных объектов), где хранятся файлы параметров и шаблон модулей службы для развертывания.  Подробнее см. в разделе [Подготовка артефактов](#prepare-the-artifacts).
    * `binaryArtifactRoot`. Если вы не изменяли структуру папок артефактов, используйте _binaries/1.0.0.0_ в этом руководстве.
    * `managedIdentityID`. Введите управляемое удостоверение, назначенное пользователем. Дополнительные сведения см. в разделе [Создание управляемого удостоверения, назначаемого пользователем](#create-the-user-assigned-managed-identity). Синтаксис:

        ```json
        "/subscriptions/<SubscriptionID>/resourcegroups/<ResourceGroupName>/providers/Microsoft.ManagedIdentity/userassignedidentities/<ManagedIdentityName>"
        ```

> [!IMPORTANT]
> Шаблон топологии и шаблон развертывания совместно используют некоторые общие параметры. Они должны иметь общие параметры. Это такие параметры: `projectName`, `azureResourceLocation` и `artifactSourceSASLocation` (оба источника артефактов используют одну и ту же учетную запись хранения в этом учебнике).

## <a name="deploy-the-templates"></a>Развертывание шаблонов.

Azure PowerShell можно использовать для развертывания шаблонов.

1. Запустите сценарий для развертывания топологии службы.

    ```azurepowershell
    # Create the service topology
    New-AzResourceGroupDeployment `
        -ResourceGroupName $resourceGroupName `
        -TemplateFile "$filePath\ADMTemplates\CreateADMServiceTopology.json" `
        -TemplateParameterFile "$filePath\ADMTemplates\CreateADMServiceTopology.Parameters.json"
    ```

    Если вы запускаете этот сценарий из сеанса PowerShell, отличного от того, в котором вы запускали сценарий [Подготовка артефактов](#prepare-the-artifacts), вам необходимо сначала заполнить переменные, которые включают `$resourceGroupName` и `$filePath`.

    > [!NOTE]
    > `New-AzResourceGroupDeployment` выполняет асинхронный вызов. Сообщение об успешном выполнении означает только то, что развертывание начато успешно. Для проверки развертывания см. шаги 2 и 4 этой процедуры.

1. Убедитесь, что топология службы и выделенные ресурсы были успешно созданы с использованием портала Azure:

    ![Руководство по использованию диспетчера развертывания Azure: развернутые ресурсы шаблона топологии](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-deployed-topology-resources.png)

    Для просмотра ресурсов необходимо выбрать **Показать скрытые типы**.

1. <a id="deploy-the-rollout-template"></a>Развертывание шаблона выпуска

    ```azurepowershell
    # Create the rollout
    New-AzResourceGroupDeployment `
        -ResourceGroupName $resourceGroupName `
        -TemplateFile "$filePath\ADMTemplates\CreateADMRollout.json" `
        -TemplateParameterFile "$filePath\ADMTemplates\CreateADMRollout.Parameters.json"
    ```

1. Проверьте выполнение развертывания с помощью следующего сценария PowerShell:

    ```azurepowershell
    # Get the rollout status
    $rolloutname = "${projectName}Rollout" # "adm0925Rollout" is the rollout name used in this tutorial
    Get-AzDeploymentManagerRollout `
        -ResourceGroupName $resourceGroupName `
        -Name $rolloutName `
        -Verbose
    ```

    Перед запуском этого командлета необходимо установить командлеты диспетчера развертывания PowerShell. См. раздел [Предварительные требования](#prerequisites). Просмотреть выходные данные полностью можно с помощью параметра `-Verbose`.

    В следующем примере показано состояние выполнения:

    ```Output
    VERBOSE:

    Status: Succeeded
    ArtifactSourceId: /subscriptions/<AzureSubscriptionID>/resourceGroups/adm0925rg/providers/Microsoft.DeploymentManager/artifactSources/adm0925ArtifactSourceRollout
    BuildVersion: 1.0.0.0

    Operation Info:
        Retry Attempt: 0
        Skip Succeeded: False
        Start Time: 03/05/2019 15:26:13
        End Time: 03/05/2019 15:31:26
        Total Duration: 00:05:12

    Service: adm0925ServiceEUS
        TargetLocation: EastUS
        TargetSubscriptionId: <AzureSubscriptionID>

        ServiceUnit: adm0925ServiceEUSStorage
            TargetResourceGroup: adm0925ServiceEUSrg

            Step: Deploy
                Status: Succeeded
                StepGroup: stepGroup3
                Operation Info:
                    DeploymentName: 2F535084871E43E7A7A4CE7B45BE06510adm0925ServiceEUSStorage
                    CorrelationId: 0b6f030d-7348-48ae-a578-bcd6bcafe78d
                    Start Time: 03/05/2019 15:26:32
                    End Time: 03/05/2019 15:27:41
                    Total Duration: 00:01:08
                Resource Operations:

                    Resource Operation 1:
                    Name: txq6iwnyq5xle
                    Type: Microsoft.Storage/storageAccounts
                    ProvisioningState: Succeeded
                    StatusCode: OK
                    OperationId: 64A6E6EFEF1F7755

    ...

    ResourceGroupName       : adm0925rg
    BuildVersion            : 1.0.0.0
    ArtifactSourceId        : /subscriptions/<SubscriptionID>/resourceGroups/adm0925rg/providers/Microsoft.DeploymentManager/artifactSources/adm0925ArtifactSourceRollout
    TargetServiceTopologyId : /subscriptions/<SubscriptionID>/resourceGroups/adm0925rg/providers/Microsoft.DeploymentManager/serviceTopologies/adm0925ServiceTopology
    Status                  : Running
    TotalRetryAttempts      : 0
    OperationInfo           : Microsoft.Azure.Commands.DeploymentManager.Models.PSRolloutOperationInfo
    Services                : {adm0925ServiceEUS, adm0925ServiceWUS}
    Name                    : adm0925Rollout
    Type                    : Microsoft.DeploymentManager/rollouts
    Location                : centralus
    Id                      : /subscriptions/<SubscriptionID>/resourcegroups/adm0925rg/providers/Microsoft.DeploymentManager/rollouts/adm0925Rollout
    Tags                    :
    ```

    После успешного развертывания выпуска вы увидите две группы ресурсов, по одной для каждой службы.

## <a name="verify-the-deployment"></a>Проверка развертывания

1. Откройте [портал Azure](https://portal.azure.com).
1. Перейдите к новым веб-приложениям в новых группах ресурсов, созданных в результате развертывания выпуска.
1. Откройте веб-приложение в веб-браузере. Проверьте расположение и версию в файле _index.html_.

## <a name="deploy-the-revision"></a>Развертывание редакции

При наличии новой версии (1.0.0.1) веб-приложения вы можете использовать следующую процедуру для повторного развертывания веб-приложения.

1. Откройте файл _CreateADMRollout.Parameters.json_.
1. Обновите `binaryArtifactRoot` до версии _binaries/1.0.0.1_.
1. Повторите развертывание, как указано в разделе [Развертывание шаблонов](#deploy-the-rollout-template).
1. Проверьте развертывание, как указано в разделе [Проверка развертывания](#verify-the-deployment). На веб-странице должна быть показана версия 1.0.0.1.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если ресурсы Azure больше не нужны, их можно удалить. Для этого необходимо удалить группу ресурсов.

1. На портале Azure в меню слева выберите **Группа ресурсов**.
1. Используйте поле **Фильтровать по имени**, чтобы сузить группы ресурсов, созданные в этом руководстве.

    * **&lt;projectName>rg**: содержит ресурсы диспетчера развертывания.
    * **&lt;projectName>ServiceWUSrg**: содержит ресурсы, определенные в ServiceWUS.
    * **&lt;projectName>ServiceEUSrg**: содержит ресурсы, определенные в ServiceEUS.
    * Группа ресурсов для определяемого пользователем управляемого удостоверения.
1. Выберите имя группы ресурсов.
1. В главном меню выберите **Удалить группу ресурсов**.
1. Повторите последние два шага для удаления других групп ресурсов, созданных в этом руководстве.

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы узнали, как использовать диспетчер развертывания Azure. Интеграция мониторинга работоспособности в диспетчер развертывания Azure описана в разделе [Tutorial: Use health check in Azure Deployment Manager](./deployment-manager-tutorial-health-check.md) (Учебник по использованию проверки работоспособности в диспетчере развертывания Azure).
