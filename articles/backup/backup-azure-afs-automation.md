---
title: Резервное копирование файлового ресурса Azure с помощью PowerShell
description: Из этой статьи вы узнаете, как создать резервную копию файлового ресурса Azure с помощью службы Azure Backup и PowerShell.
ms.topic: conceptual
ms.date: 08/20/2019
ms.openlocfilehash: 948931764769bc967b88e7942b7e8384b0f93dff
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87077003"
---
# <a name="back-up-an-azure-file-share-by-using-powershell"></a>Резервное копирование файлового ресурса Azure с помощью PowerShell

В этой статье описывается, как использовать Azure PowerShell для резервного копирования файлового ресурса службы файлов Azure через хранилище служб восстановления [Azure Backup](backup-overview.md) .

В статье описывается выполнение следующих задач:

> [!div class="checklist"]
>
> * Настройте PowerShell и зарегистрируйте поставщик служб восстановления.
> * Создайте хранилище служб восстановления,
> * Настройте резервное копирование для файлового ресурса Azure.
> * Запустите задание резервного копирования.

## <a name="before-you-start"></a>Перед началом работы

* Дополнительные [сведения](backup-azure-recovery-services-vault-overview.md) о хранилищах служб восстановления.
* Ознакомьтесь со справочником по [командлету](/powershell/module/az.recoveryservices) AZ. RecoveryServices в библиотеке Azure.
* Проверьте следующую иерархию объектов PowerShell для служб восстановления.

  ![Иерархия объектов служб восстановления](./media/backup-azure-vms-arm-automation/recovery-services-object-hierarchy.png)

## <a name="set-up-powershell"></a>Настройка PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Настройте PowerShell следующим образом.

1. [Скачайте последнюю версию Azure PowerShell](/powershell/azure/install-az-ps).

    > [!NOTE]
    > Минимальная версия PowerShell, необходимая для резервного копирования файловых ресурсов Azure, — AZ. RecoveryServices 2.6.0. Использование последней версии или по крайней мере минимальной версии позволяет избежать проблем с существующими сценариями. Установите минимальную версию с помощью следующей команды PowerShell:
    >
    > ```powershell
    > Install-module -Name Az.RecoveryServices -RequiredVersion 2.6.0
    > ```

2. Найдите командлеты PowerShell для Azure Backup с помощью следующей команды:

    ```powershell
    Get-Command *azrecoveryservices*
    ```

3. Проверьте псевдонимы и командлеты для Azure Backup, Azure Site Recovery и хранилища служб восстановления. Ниже приведен пример того, что можно увидеть. Это не полный список командлетов.

    ![Список командлетов Служб восстановления](./media/backup-azure-afs-automation/list-of-recoveryservices-ps-az.png)

4. Чтобы войти в учетную запись Azure, используйте командлет **Connect-AzAccount**.
5. На появившейся веб-странице вам будет предложено ввести учетные данные.

    Кроме того, можно включить учетные данные учетной записи в качестве параметра командлета **Connect-азаккаунт** , используя параметр **-Credential**.

    Если вы являетесь партнером CSP, работающим от имени клиента, укажите клиента в качестве клиента. Используйте идентификатор клиента или основное доменное имя клиента. Например, **Connect-азаккаунт-клиент "fabrikam.com"**.

6. Свяжите подписку, которую вы хотите использовать с учетной записью, так как у учетной записи может быть несколько подписок:

    ```powershell
    Select-AzSubscription -SubscriptionName $SubscriptionName
    ```

7. Если вы используете Azure Backup в первый раз, используйте командлет **Register-азресаурцепровидер** , чтобы зарегистрировать поставщик служб восстановления Azure в своей подписке.

    ```powershell
    Register-AzResourceProvider -ProviderNamespace "Microsoft.RecoveryServices"
    ```

8. Убедитесь, что поставщики успешно зарегистрированы:

    ```powershell
    Get-AzResourceProvider -ProviderNamespace "Microsoft.RecoveryServices"
    ```

9. Убедитесь, что в выходных данных команды **RegistrationState** изменения **зарегистрированы**. Если это не так, выполните командлет **Register-азресаурцепровидер** еще раз.

## <a name="create-a-recovery-services-vault"></a>Создание хранилища Служб восстановления

Хранилище служб восстановления — это диспетчер ресурсов ресурс, поэтому его необходимо поместить в группу ресурсов. Можно использовать существующую группу ресурсов или создать группу ресурсов с помощью командлета **New-азресаурцеграуп** . При создании группы ресурсов укажите ее имя и расположение.

Чтобы создать хранилище служб восстановления, выполните следующие действия.

1. Если у вас нет группы ресурсов, создайте ее с помощью командлета [New-азресаурцеграуп](/powershell/module/az.resources/new-azresourcegroup) . В этом примере мы создадим группу ресурсов в регионе "Западная часть США":

   ```powershell
   New-AzResourceGroup -Name "test-rg" -Location "West US"
   ```

1. Используйте командлет [New-азрековерисервицесваулт](/powershell/module/az.recoveryservices/new-azrecoveryservicesvault) , чтобы создать хранилище. Укажите то же расположение для хранилища, которое использовалось для группы ресурсов.

    ```powershell
    New-AzRecoveryServicesVault -Name "testvault" -ResourceGroupName "test-rg" -Location "West US"
    ```

### <a name="view-the-vaults-in-a-subscription"></a>Просмотр хранилищ в подписке

Чтобы просмотреть все хранилища в подписке, используйте команду [Get-азрековерисервицесваулт](/powershell/module/az.recoveryservices/get-azrecoveryservicesvault):

```powershell
Get-AzRecoveryServicesVault
```

Выходные данные похожи на приведенные ниже. Обратите внимание, что выходные данные содержат связанную группу ресурсов и расположение.

```powershell
Name              : Contoso-vault
ID                : /subscriptions/1234
Type              : Microsoft.RecoveryServices/vaults
Location          : WestUS
ResourceGroupName : Contoso-docs-rg
SubscriptionId    : 1234-567f-8910-abc
Properties        : Microsoft.Azure.Commands.RecoveryServices.ARSVaultProperties
```

### <a name="set-the-vault-context"></a>Задание контекста хранилища

Сохраните объект хранилища в переменной и задайте контекст хранилища.

Многие командлеты Azure Backup нуждаются в объекте хранилища служб восстановления в качестве входных данных, поэтому удобно хранить объект хранилища в переменной.

Контекст хранилища — это тип данных, защищаемых в хранилище. Задайте его с помощью [Set-азрековерисервицесваултконтекст](/powershell/module/az.recoveryservices/set-azrecoveryservicesvaultcontext). После установки контекста он применяется ко всем последующим командлетам.

В следующем примере задается контекст хранилища для **testvault задано**:

```powershell
Get-AzRecoveryServicesVault -Name "testvault" | Set-AzRecoveryServicesVaultContext
```

### <a name="fetch-the-vault-id"></a>Получение идентификатора хранилища

Мы планируем использовать параметр контекста хранилища в соответствии с рекомендациями Azure PowerShell. Вместо этого можно сохранить или извлечь идентификатор хранилища и передать его в соответствующие команды. Если вы не указали контекст хранилища или хотите указать команду для выполнения в определенном хранилище, передайте идентификатор хранилища во `-vaultID` все соответствующие команды следующим образом:

```powershell
$vaultID = Get-AzRecoveryServicesVault -ResourceGroupName "Contoso-docs-rg" -Name "testvault" | select -ExpandProperty ID
New-AzRecoveryServicesBackupProtectionPolicy -Name "NewAFSPolicy" -WorkloadType "AzureFiles" -RetentionPolicy $retPol -SchedulePolicy $schPol -VaultID $vaultID
```

## <a name="configure-a-backup-policy"></a>Настройка политики резервного копирования

Политика архивации определяет расписание резервного копирования и время хранения резервных точек восстановления.

Политика резервного копирования связана по крайней мере с одной политикой хранения. Политика хранения определяет продолжительность хранения точки восстановления до ее удаления. Резервные копии можно настроить ежедневно, еженедельно, ежемесячно или ежегодно.

Ниже приведены некоторые командлеты для политик архивации.

* Просмотрите политику резервного копирования по умолчанию с помощью команды [Get-азрековерисервицесбаккупретентионполициобжект](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupretentionpolicyobject).
* Просмотрите расписание политики архивации по умолчанию с помощью команды [Get-азрековерисервицесбаккупсчедулеполициобжект](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupschedulepolicyobject).
* Создайте новую политику резервного копирования с помощью команды [New-азрековерисервицесбаккуппротектионполици](/powershell/module/az.recoveryservices/set-azrecoveryservicesbackupprotectionpolicy). Вы вводите в качестве входных данных объекты политики расписания и хранения.

По умолчанию время начала определяется в объекте политики Schedule. Используйте следующий пример, чтобы изменить время начала на нужное время начала. Нужное время начала должно быть в формате UTC. В примере предполагается, что требуемое время начала для ежедневного резервного копирования составляет 01:00 часов по ГРИНВИЧу.

```powershell
$schPol = Get-AzRecoveryServicesBackupSchedulePolicyObject -WorkloadType "AzureFiles"
$UtcTime = Get-Date -Date "2019-03-20 01:30:00Z"
$UtcTime = $UtcTime.ToUniversalTime()
$schpol.ScheduleRunTimes[0] = $UtcTime
```

> [!IMPORTANT]
> Необходимо указать время начала в 30-минутном виде. В предыдущем примере это может быть только "01:00:00" или "02:30:00". Время начала не может быть "01:15:00".

В следующем примере показано сохранение политик расписания и хранения в переменных. Затем эти переменные используются в качестве параметров для новой политики (**невафсполици**). **NewAFSPolicy** выполняет ежедневное резервное копирование и хранит его в течение 30 дней.

```powershell
$schPol = Get-AzRecoveryServicesBackupSchedulePolicyObject -WorkloadType "AzureFiles"
$retPol = Get-AzRecoveryServicesBackupRetentionPolicyObject -WorkloadType "AzureFiles"
New-AzRecoveryServicesBackupProtectionPolicy -Name "NewAFSPolicy" -WorkloadType "AzureFiles" -RetentionPolicy $retPol -SchedulePolicy $schPol
```

Выход аналогичен приведенному ниже:

```powershell
Name                 WorkloadType       BackupManagementType BackupTime                DaysOfWeek
----                 ------------       -------------------- ----------                ----------
NewAFSPolicy           AzureFiles            AzureStorage              10/24/2019 1:30:00 AM
```

## <a name="enable-backup"></a>Включение резервного копирования

После определения политики архивации можно включить защиту для файлового ресурса Azure с помощью политики.

### <a name="retrieve-a-backup-policy"></a>Получение политики архивации

Чтобы получить соответствующий объект политики, используйте [Get-азрековерисервицесбаккуппротектионполици](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupprotectionpolicy). Используйте этот командлет для просмотра политик, связанных с типом рабочей нагрузки, или для получения определенной политики.

#### <a name="retrieve-a-policy-for-a-workload-type"></a>Получение политики для типа рабочей нагрузки

В следующем примере извлекаются политики для типа рабочей нагрузки **AzureFiles**:

```powershell
Get-AzRecoveryServicesBackupProtectionPolicy -WorkloadType "AzureFiles"
```

Выход аналогичен приведенному ниже:

```powershell
Name                 WorkloadType       BackupManagementType BackupTime                DaysOfWeek
----                 ------------       -------------------- ----------                ----------
dailyafs             AzureFiles         AzureStorage         1/10/2018 12:30:00 AM
```

> [!NOTE]
> Часовой **пояс поля с** недоступностью в PowerShell находится в формате UTC. Однако при отображении времени архивации на портале Azure время меняется в соответствии с локальным часовым поясом.

#### <a name="retrieve-a-specific-policy"></a>Получение определенной политики

Следующая политика получает политику резервного копирования с именем **даиляфс**:

```powershell
$afsPol =  Get-AzRecoveryServicesBackupProtectionPolicy -Name "dailyafs"
```

### <a name="enable-protection-and-apply-the-policy"></a>Включение защиты и применение политики

Включите защиту с помощью команды [Enable-азрековерисервицесбаккуппротектион](/powershell/module/az.recoveryservices/enable-azrecoveryservicesbackupprotection). После того как политика будет связана с хранилищем, резервные копии будут активированы в соответствии с расписанием политики.

В следующем примере включается защита файлового ресурса Azure **тестазурефилешаре** в учетной записи хранения **тестсторажеаккт** с политикой **даиляфс**:

```powershell
Enable-AzRecoveryServicesBackupProtection -StorageAccountName "testStorageAcct" -Name "testAzureFS" -Policy $afsPol
```

Команда ожидает завершения задания настройки защиты и выдает выходные данные, аналогичные приведенным в следующем примере:

```cmd
WorkloadName       Operation            Status                 StartTime                                                                                                         EndTime                   JobID
------------             ---------            ------               ---------                                  -------                   -----
testAzureFS       ConfigureBackup      Completed            11/12/2018 2:15:26 PM     11/12/2018 2:16:11 PM     ec7d4f1d-40bd-46a4-9edb-3193c41f6bf6
```

Дополнительные сведения о том, как получить список общих файловых ресурсов для учетной записи хранения, см. в [этой статье](/powershell/module/az.storage/get-azstorageshare).

## <a name="important-notice-backup-item-identification"></a>Важное замечание: идентификация элемента резервного копирования

В этом разделе описаны важные изменения в резервных копиях файловых ресурсов Azure при подготовке к общедоступной версии.

При включении резервного копирования для файловых ресурсов Azure пользователь предоставляет клиенту имя файлового ресурса в качестве имени сущности и создается элемент резервного копирования. Имя элемента резервного копирования — это уникальный идентификатор, создаваемый службой Azure Backup. Обычно идентификатор является понятным именем пользователя. Но для решения сценария обратимого удаления, в котором можно удалить общую папку и создать другую общую папку с тем же именем, уникальный идентификатор файлового ресурса Azure теперь является ИДЕНТИФИКАТОРом.

Чтобы узнать уникальный идентификатор каждого элемента, выполните команду **Get-азрековерисервицесбаккупитем** с соответствующими фильтрами для **баккупманажементтипе** и **ворклоадтипе** , чтобы получить все соответствующие элементы. Затем просмотрите поле Name в возвращенном объекте или ответе PowerShell.

Рекомендуется перечислить элементы, а затем извлечь их уникальное имя из поля имя в ответе. Используйте это значение для фильтрации элементов с помощью параметра *Name* . В противном случае используйте параметр *FriendlyName* для получения элемента с его идентификатором.

> [!IMPORTANT]
> Убедитесь, что PowerShell обновлен до минимальной версии (az. RecoveryServices 2.6.0) для резервного копирования файловых ресурсов Azure. В этой версии для команды **Get-азрековерисервицесбаккупитем** доступен фильтр *FriendlyName* .
>
> Передайте имя общего файлового ресурса Azure в параметр *FriendlyName* . Если имя общей папки передается в параметр *Name* , эта версия выдает предупреждение, чтобы передать имя в параметр *FriendlyName* .
>
> Установка минимальной версии может привести к сбою существующих скриптов. Установите минимальную версию PowerShell с помощью следующей команды:
>
>```powershell
>Install-module -Name Az.RecoveryServices -RequiredVersion 2.6.0
>```

## <a name="trigger-an-on-demand-backup"></a>Активирование резервного копирования по запросу

Используйте [BACKUP-азрековерисервицесбаккупитем](/powershell/module/az.recoveryservices/backup-azrecoveryservicesbackupitem) для запуска резервного копирования по запросу для защищенного файлового ресурса Azure.

1. Получите учетную запись хранения из контейнера в хранилище, в котором хранятся резервные копии данных, с помощью команды [Get-азрековерисервицесбаккупконтаинер](/powershell/module/az.recoveryservices/get-Azrecoveryservicesbackupcontainer).
2. Чтобы запустить задание резервного копирования, получите сведения о файловом ресурсе Azure с помощью команды [Get-азрековерисервицесбаккупитем](/powershell/module/az.recoveryservices/Get-AzRecoveryServicesBackupItem).
3. Выполните резервное копирование по запросу с помощью [BACKUP-азрековерисервицесбаккупитем](/powershell/module/az.recoveryservices/backup-Azrecoveryservicesbackupitem).

Выполните резервное копирование по запросу следующим образом:

```powershell
$afsContainer = Get-AzRecoveryServicesBackupContainer -FriendlyName "testStorageAcct" -ContainerType AzureStorage
$afsBkpItem = Get-AzRecoveryServicesBackupItem -Container $afsContainer -WorkloadType "AzureFiles" -FriendlyName "testAzureFS"
$job =  Backup-AzRecoveryServicesBackupItem -Item $afsBkpItem
```

Команда возвращает задание с ИДЕНТИФИКАТОРом для трассировки, как показано в следующем примере:

```powershell
WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
------------     ---------            ------               ---------                 -------                   -----
testAzureFS       Backup               Completed            11/12/2018 2:42:07 PM     11/12/2018 2:42:11 PM     8bdfe3ab-9bf7-4be6-83d6-37ff1ca13ab6
```

Моментальные снимки файлового ресурса Azure используются во время резервного копирования. Обычно задание завершается по времени, когда команда возвращает эти выходные данные.

## <a name="next-steps"></a>Дальнейшие действия

* Сведения о [резервном копировании файлов Azure в портал Azure](backup-afs.md).
* Чтобы запланировать резервное копирование, см. [Пример скрипта в GitHub](https://github.com/Azure-Samples/Use-PowerShell-for-long-term-retention-of-Azure-Files-Backup) для использования модуля Runbook службы автоматизации Azure.
