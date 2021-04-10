---
title: Руководство. Использование последовательностей runbook службы автоматизации Azure для создания кластеров (Azure HDInsight)
description: Узнайте, как создавать и удалять кластеры Azure HDInsight с помощью выполняемых в облаке скриптов, оформленных в виде runbook службы автоматизации Azure.
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: tutorial
ms.date: 12/27/2019
ms.openlocfilehash: 5eb0f353579233041bb5ccba46de2549ada7e9b7
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104864794"
---
# <a name="tutorial-create-azure-hdinsight-clusters-with-azure-automation"></a>Руководство. Создание кластеров Azure HDInsight с помощью службы автоматизации Azure

Служба автоматизации Azure позволяет создавать выполняемые в облаке скрипты, которые управляют ресурсами Azure по требованию или по расписанию. В этой статье описывается создание runbook с инструкциями PowerShell для создания и удаления кластеров Azure HDInsight.

В этом руководстве описано следующее.

> [!div class="checklist"]
> * Установка модулей, необходимых для взаимодействия с HDInsight.
> * Создание и сохранение учетных данных, которые потребуются при создании кластера.
> * Создание runbook службы автоматизации Azure для создания кластера HDInsight.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

* Существующая [учетная запись службы автоматизации Azure](../automation/automation-quickstart-create-account.md).
* Существующая [учетная запись хранения Azure](../storage/common/storage-account-create.md), которая будет использоваться в качестве хранилища кластера.

## <a name="install-hdinsight-modules"></a>Установка модулей HDInsight

1. Войдите на [портал Azure](https://portal.azure.com).
1. Выберите учетные записи службы автоматизации Azure.
1. Щелкните **Коллекция модулей** в разделе **Общие ресурсы**.
1. Введите в поле поиска **AzureRM.Profile** и нажмите клавишу ВВОД. Выберите полученный результат поиска.
1. На экране **AzureRM.profile** щелкните **Импортировать**. Установите флажок, отвечающий за обновление модулей Azure, а затем щелкните **ОК**.

    :::image type="content" source="./media/manage-clusters-runbooks/import-azurermprofile-module.png" alt-text="Импорт модуля AzureRM.profile" border="false":::

1. Вернитесь в коллекцию модулей, выбрав элемент **Коллекция модулей** в разделе **Общие ресурсы**.
1. Введите **HDInsight**. Щелкните **AzureRM.HDInsight**.

    :::image type="content" source="./media/manage-clusters-runbooks/browse-modules-hdinsight.png" alt-text="Просмотр модулей HDInsight" border="true":::

1. На панели **AzureRM.HDInsight** щелкните **Импортировать**, а затем **ОК**.

    :::image type="content" source="./media/manage-clusters-runbooks/import-azurermhdinsight-module.png" alt-text="Импорт модуля AzureRM.HDInsight" border="true":::

## <a name="create-credentials"></a>Создание учетных данных

1. Выберите **Учетные данные** в разделе **Общие ресурсы**.
1. Щелкните **Добавить учетные данные**.
1. На панели **Новые учетные данные** введите необходимые сведения. Эти учетные данные нужны для хранения пароля кластера, который позволит войти в Ambari.

    | Свойство | Значение |
    | --- | --- |
    | Имя | `cluster-password` |
    | Имя пользователя | `admin` |
    | Пароль | `SECURE_PASSWORD` |
    | Подтверждение пароля | `SECURE_PASSWORD` |

1. Нажмите кнопку **создания**.
1. Повторите ту же процедуру для новых учетных данных `ssh-password` с именем пользователя `sshuser` и выбранным вами паролем. Нажмите кнопку **создания**. Эти учетные данные нужны для хранения пароля SSH для кластера.

    :::image type="content" source="./media/manage-clusters-runbooks/create-credentials.png" alt-text="Создание учетных данных" border="true":::

## <a name="create-a-runbook-to-create-a-cluster"></a>Создание runbook, который создает кластер

1. Выберите **Модули Runbook** в разделе **Автоматизация процессов**.
1. Нажмите кнопку **Создать Runbook**.
1. На панели **Создание Runbook** введите имя runbook, например `hdinsight-cluster-create`. Выберите значение **PowerShell** в раскрывающемся списке **Тип runbook**.
1. Нажмите кнопку **создания**.

    :::image type="content" source="./media/manage-clusters-runbooks/create-runbook.png" alt-text="Создание runbook" border="true":::

1. На экране **Редактировать runbook PowerShell** введите следующий код и щелкните **Опубликовать**.

    :::image type="content" source="./media/manage-clusters-runbooks/publish-runbook.png" alt-text="Публикация runbook" border="true":::

    ```powershell
    Param
    (
      [Parameter (Mandatory= $true)]
      [String] $subscriptionID,
    
      [Parameter (Mandatory= $true)]
      [String] $resourceGroup,
    
      [Parameter (Mandatory= $true)]
      [String] $storageAccount,
    
      [Parameter (Mandatory= $true)]
      [String] $containerName,
    
      [Parameter (Mandatory= $true)]
      [String] $clusterName
    )
    ### Authenticate to Azure 
    $Conn = Get-AutomationConnection -Name 'AzureRunAsConnection'
    Add-AzureRMAccount -ServicePrincipal -Tenant $Conn.TenantID -ApplicationId $Conn.ApplicationID -CertificateThumbprint $Conn.CertificateThumbprint
    
    # Set cluster variables
    $storageAccountKey = (Get-AzureRmStorageAccountKey –Name $storageAccount –ResourceGroupName $resourceGroup)[0].value 
    
    # Setting cluster credentials
    
    #Automation credential for Cluster Admin
    $clusterCreds = Get-AutomationPSCredential –Name 'cluster-password'
    
    #Automation credential for user to SSH into cluster
    $sshCreds = Get-AutomationPSCredential –Name 'ssh-password' 
    
    $clusterType = "Hadoop" #Use any supported cluster type (Hadoop, HBase, Storm, etc.)
    $clusterOS = "Linux"
    $clusterWorkerNodes = 3
    $clusterNodeSize = "Standard_D3_v2"
    $location = Get-AzureRmStorageAccount –StorageAccountName $storageAccount –ResourceGroupName $resourceGroup | %{$_.Location}
    
    ### Provision HDInsight cluster
    New-AzureRmHDInsightCluster –ClusterName $clusterName –ResourceGroupName $resourceGroup –Location $location –DefaultStorageAccountName "$storageAccount.blob.core.windows.net" –DefaultStorageAccountKey $storageAccountKey -DefaultStorageContainer $containerName –ClusterType $clusterType –OSType $clusterOS –Version “3.6” –HttpCredential $clusterCreds –SshCredential $sshCreds –ClusterSizeInNodes $clusterWorkerNodes –HeadNodeSize $clusterNodeSize –WorkerNodeSize $clusterNodeSize
    ```

## <a name="create-a-runbook-to-delete-a-cluster"></a>Создание runbook, который удаляет кластер

1. Выберите **Модули Runbook** в разделе **Автоматизация процессов**.
1. Нажмите кнопку **Создать Runbook**.
1. На панели **Создание Runbook** введите имя runbook, например `hdinsight-cluster-delete`. Выберите значение **PowerShell** в раскрывающемся списке **Тип runbook**.
1. Нажмите кнопку **создания**.
1. На экране **Редактировать runbook PowerShell** введите следующий код и щелкните **Опубликовать**.

    ```powershell
    Param
    (
      [Parameter (Mandatory= $true)]
      [String] $clusterName
    )
    
    ### Authenticate to Azure 
    $Conn = Get-AutomationConnection -Name 'AzureRunAsConnection'
    Add-AzureRMAccount -ServicePrincipal -Tenant $Conn.TenantID -ApplicationId $Conn.ApplicationID -CertificateThumbprint $Conn.CertificateThumbprint
    
    Remove-AzureRmHDInsightCluster -ClusterName $clusterName
    ```

## <a name="execute-runbooks"></a>Выполнение runbook

### <a name="create-a-cluster"></a>Создание кластера

1. Откройте список runbook в учетной записи службы автоматизации, выбрав элемент **Модули Runbook** в разделе **Автоматизация процессов**.
1. Щелкните `hdinsight-cluster-create` или другое имя, которое вы указали при создании runbook для создания кластера.
1. Щелкните **Запустить**, чтобы немедленно выполнить этот runbook. Вы также можете настроить периодическое выполнение runbook. Подробнее см. [Создание расписания для модуля Runbook в службе автоматизации Azure](../automation/shared-resources/schedules.md).
1. Задайте значения для обязательных параметров скрипта и щелкните **OK**. Это действие создаст новый кластер HDInsight с именем, которое вы указали в параметре **CLUSTERNAME**.

    :::image type="content" source="./media/manage-clusters-runbooks/execute-create-runbook.png" alt-text="Выполнение runbook для создания кластера" border="true":::

### <a name="delete-a-cluster"></a>Удаление кластера

Удалите кластер, выбрав созданный ранее runbook `hdinsight-cluster-delete`. Выберите **Запуск**, введите параметр **CLUSTERNAME** и щелкните **ОК**.

## <a name="clean-up-resources"></a>Очистка ресурсов

Удалите созданную учетную запись службы автоматизации Azure, когда закончите работу с ней, чтобы избежать ненужных расходов. Для этого откройте портал Azure, выберите группу ресурсов, в которой вы создали учетную запись службы автоматизации Azure, затем саму учетную запись службы автоматизации и щелкните **Удалить**.

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Управление кластерами Apache Hadoop в HDInsight с помощью Azure PowerShell](hdinsight-administer-use-powershell.md)