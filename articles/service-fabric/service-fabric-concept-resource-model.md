---
title: Модель ресурсов приложения Service Fabric Azure
description: В этой статье содержатся общие сведения об управлении приложением Service Fabric Azure с помощью Azure Resource Manager.
ms.topic: conceptual
ms.date: 10/21/2019
ms.openlocfilehash: 0019f154f301d2b688d4c16c9adb36ec386adef2
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98790729"
---
# <a name="service-fabric-application-resource-model"></a>Модель ресурсов приложения Service Fabric

Существует несколько вариантов развертывания приложений Azure Service Fabric в кластере Service Fabric. Рекомендуется использовать Azure Resource Manager. При использовании диспетчер ресурсов можно описать приложения и службы в JSON, а затем развернуть их в том же шаблоне диспетчер ресурсов, что и кластер. В отличие от использования PowerShell или Azure CLI для развертывания приложений и управления ими, при использовании диспетчер ресурсов вам не нужно ждать, пока кластер будет готов. Регистрация приложений, подготовка и развертывание могут выполняться за один шаг. Использование диспетчер ресурсов — лучший способ управления жизненным циклом приложения в кластере. Дополнительные сведения см. в разделе рекомендации [: инфраструктура как код](service-fabric-best-practices-infrastructure-as-code.md#azure-service-fabric-resources).

Управление приложениями в качестве ресурсов в диспетчер ресурсов может помочь в улучшении этих областей:

* Журнал аудита. диспетчер ресурсов выполняет аудит каждой операции и сохраняет подробный журнала действий. Журнал действий может помочь в трассировке любых изменений, внесенных в приложения и в кластер.
* Управление доступом на основе ролей. Вы можете управлять доступом к кластерам и приложениям, развернутым в кластере, с помощью того же шаблона диспетчер ресурсов.
* Эффективность управления. Использование диспетчер ресурсов предоставляет единое расположение (портал Azure) для управления кластером и критическими развертываниями приложений.

В этом документе вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
>
> * Развертывание ресурсов приложения с помощью диспетчер ресурсов.
> * Обновление ресурсов приложения с помощью диспетчер ресурсов.
> * Удаление ресурсов приложения.

## <a name="deploy-application-resources"></a>Развертывание ресурсов приложения

Высокоуровневые действия, которые необходимо выполнить для развертывания приложения и его служб с помощью модели ресурсов диспетчер ресурсов приложения:
1. Упакуйте код приложения.
1. Отправьте пакет.
1. Ссылка на расположение пакета в шаблоне диспетчер ресурсов в качестве ресурса приложения. 

Дополнительные сведения см. в [пакете приложения](service-fabric-package-apps.md#create-an-sfpkg).

Затем создайте шаблон диспетчер ресурсов, обновите файл параметров, указав сведения о приложении, и разверните шаблон в кластере Service Fabric. [Ознакомьтесь с примерами](https://github.com/Azure-Samples/service-fabric-dotnet-quickstart/tree/master/ARM).

### <a name="create-a-storage-account"></a>Создание учетной записи хранения

Чтобы развернуть приложение из шаблона диспетчер ресурсов, необходимо иметь учетную запись хранения. Учетная запись хранения используется для размещения образа приложения. 

Вы можете повторно использовать существующую учетную запись хранения или создать новую учетную запись хранения для промежуточных приложений. Если вы используете существующую учетную запись хранения, этот шаг можно пропустить. 

![Создание учетной записи хранения][CreateStorageAccount]

### <a name="configure-your-storage-account"></a>Настройка учетной записи хранения

После создания учетной записи хранения создайте контейнер больших двоичных объектов, в котором можно разместить приложения. В портал Azure перейдите в учетную запись хранения Azure, в которой нужно сохранить приложения. Выберите **BLOB-объекты**  >  **добавить контейнер**. 

Ресурсы в кластере можно защитить, задав для общего уровня доступа значение **частный**. Предоставить доступ можно несколькими способами:

* Авторизуйте доступ к BLOB-объектам и очередям с помощью [Azure Active Directory](../storage/common/storage-auth-aad-app.md).
* Предоставьте доступ к данным большого двоичного объекта и очереди Azure с помощью [Azure RBAC в портал Azure](../storage/common/storage-auth-aad-rbac-portal.md).
* Делегирование доступа с помощью [подписанного](/rest/api/storageservices/delegate-access-with-shared-access-signature)URL-доступа.

Пример на следующем снимке экрана использует анонимный доступ на чтение для больших двоичных объектов.

![Создать BLOB-объект][CreateBlob]

### <a name="stage-the-application-in-your-storage-account"></a>Размещение приложения в учетной записи хранения

Перед развертыванием приложения необходимо разместить приложение в хранилище BLOB-объектов. В этом руководстве мы создадим пакет приложения вручную. Помните, что этот шаг можно автоматизировать. Дополнительные сведения см. в разделе [Упаковка приложения](service-fabric-package-apps.md#create-an-sfpkg). 

В этом руководстве мы будем использовать [пример приложения для голосования](https://github.com/Azure-Samples/service-fabric-dotnet-quickstart).

1. В Visual Studio щелкните правой кнопкой мыши проект **голосования** и выберите **пакет**.

   ![Приложение пакета][PackageApplication]  
1. Перейдите в каталог *.\сервице-фабрик-дотнет-куиккстарт\вотинг\пкг\дебуг* . Заархивировать содержимое в файл с именем *Voting.zip*. Файл *ApplicationManifest.xml* должен находиться в корневом каталоге ZIP-файла.

   ![ZIP-приложение][ZipApplication]  
1. Переименуйте файл, чтобы изменить расширение ZIP на *. sfpkg*.

1. В портал Azure в контейнере **приложения** для учетной записи хранения выберите **Отправить**, а затем отправьте **голосование. sfpkg**. 

   ![Отправить пакет приложения][UploadAppPkg]

Теперь приложение подготовлено к поэтапному развертыванию, и вы можете создать шаблон диспетчер ресурсов для развертывания приложения.

### <a name="create-the-resource-manager-template"></a>Создание шаблона Resource Manager

Пример приложения содержит [шаблоны Azure Resource Manager](https://github.com/Azure-Samples/service-fabric-dotnet-quickstart/tree/master/ARM) , которые можно использовать для развертывания приложения. Имена файлов шаблонов *UserApp.jsв* и *UserApp.Parameters.js*.

> [!NOTE]
> *UserApp.Parameters.jsв* файле необходимо обновить именем кластера.
>
>


| Параметр              | Описание                                 | Пример                                                      | Комментарии                                                     |
| ---------------------- | ------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| clusterName            | Имя кластера, в который выполняется развертывание | SF-cluster123                                                |                                                              |
| application            | Имя приложения                 | Голосовани                                                       |
| applicationTypeName    | Имя типа приложения.           | VotingType                                                   | Должно соответствовать ApplicationManifest.xml                 |
| applicationTypeVersion | Версия типа приложения         | 1.0.0                                                        | Должно соответствовать ApplicationManifest.xml                 |
| serviceName            | Имя службы         | Голосование ~ VotingWeb                                             | Должен быть в формате ApplicationName ~ ServiceType            |
| serviceTypeName        | Имя типа службы                | VotingWeb                                                    | Должно соответствовать ServiceManifest.xml                 |
| апппаккажеурл          | URL-адрес хранилища больших двоичных объектов приложения     | HTTPS: \/ /servicefabricapps.BLOB.Core.Windows.NET/Apps/Voting.sfpkg | URL-адрес пакета приложения в хранилище BLOB-объектов (процедура для задания URL-адреса описывается далее в этой статье). |

```json
{
    "apiVersion": "2019-03-01",
    "type": "Microsoft.ServiceFabric/clusters/applications",
    "name": "[concat(parameters('clusterName'), '/', parameters('applicationName'))]",
    "location": "[variables('clusterLocation')]",
},
{
    "apiVersion": "2019-03-01",
    "type": "Microsoft.ServiceFabric/clusters/applicationTypes",
    "name": "[concat(parameters('clusterName'), '/', parameters('applicationTypeName'))]",
    "location": "[variables('clusterLocation')]",
},
{
    "apiVersion": "2019-03-01",
    "type": "Microsoft.ServiceFabric/clusters/applicationTypes/versions",
    "name": "[concat(parameters('clusterName'), '/', parameters('applicationTypeName'), '/', parameters('applicationTypeVersion'))]",
    "location": "[variables('clusterLocation')]",
},
{
    "apiVersion": "2019-03-01",
    "type": "Microsoft.ServiceFabric/clusters/applications/services",
    "name": "[concat(parameters('clusterName'), '/', parameters('applicationName'), '/', parameters('serviceName'))]",
    "location": "[variables('clusterLocation')]"
}
```

### <a name="deploy-the-application"></a>Развертывание приложения

Выполните командлет **New-азресаурцеграупдеплоймент** , чтобы развернуть приложение в группе ресурсов, содержащей кластер:

```powershell
New-AzResourceGroupDeployment -ResourceGroupName "sf-cluster-rg" -TemplateParameterFile ".\UserApp.Parameters.json" -TemplateFile ".\UserApp.json" -Verbose
```

## <a name="upgrade-the-service-fabric-application-by-using-resource-manager"></a>Обновление Service Fabric приложения с помощью диспетчер ресурсов


> [!IMPORTANT]
> Любая служба, развернутая с помощью определения JSON в ARM, должна быть удалена из раздела Дефаултсервицес соответствующего файла ApplicationManifest.xml.


Вы можете обновить приложение, которое уже развернуто в кластере Service Fabric, по одной из следующих причин:

* В приложение добавляется новая служба. Определение службы необходимо добавить в *service-manifest.xml* и *application-manifest.xml* файлы при добавлении службы в приложение. Чтобы отразить новую версию приложения, необходимо также изменить версию типа приложения с 1.0.0 на 1.0.1 в [UserApp.Parameters.js](https://github.com/Azure-Samples/service-fabric-dotnet-quickstart/blob/master/ARM/UserApp.Parameters.json):

    ```json
    "applicationTypeVersion": {
        "value": "1.0.1"
    },
    "serviceName2": {
        "value": "Voting~VotingData"
    },
    "serviceTypeName2": {
        "value": "VotingDataType"
    }
    ```

* В приложение добавляется новая версия существующей службы. Примеры включают изменения кода приложения и обновления для версии и имени типа приложения. Для этого обновления обновите UserApp.Parameters.jsследующим образом:

    ```json
     "applicationTypeVersion": {
        "value": "1.0.1"
    },
    ```

## <a name="delete-application-resources"></a>Удаление ресурсов приложения

Чтобы удалить приложение, которое было развернуто с помощью модели ресурсов приложения, в диспетчер ресурсов:

1. Используйте командлет [Get-азресаурце](/powershell/module/az.resources/get-azresource) , чтобы получить идентификатор ресурса для приложения:

    ```powershell
    Get-AzResource  -Name <String> | f1
    ```

1. Удалите ресурсы приложения с помощью командлета [Remove-азресаурце](/powershell/module/az.resources/remove-azresource) :

    ```powershell
    Remove-AzResource  -ResourceId <String> [-Force] [-ApiVersion <String>]
    ```

## <a name="next-steps"></a>Дальнейшие действия

Получение сведений о модели ресурсов приложения:

* [Моделирование приложения в Service Fabric](service-fabric-application-model.md)
* [Манифесты службы и приложения Service Fabric](service-fabric-application-and-service-manifests.md)
* [Рекомендации: инфраструктура как код](service-fabric-best-practices-infrastructure-as-code.md#azure-service-fabric-resources)
* [Управление приложениями и службами как ресурсами Azure](service-fabric-best-practices-infrastructure-as-code.md)


<!--Image references-->
[CreateStorageAccount]: ./media/service-fabric-application-model/create-storage-account.png
[CreateBlob]: ./media/service-fabric-application-model/create-blob.png
[PackageApplication]: ./media/service-fabric-application-model/package-application.png
[ZipApplication]: ./media/service-fabric-application-model/zip-application.png
[UploadAppPkg]: ./media/service-fabric-application-model/upload-app-pkg.png
