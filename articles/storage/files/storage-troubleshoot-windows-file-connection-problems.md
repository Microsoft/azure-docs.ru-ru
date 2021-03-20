---
title: Устранение неполадок службы файлов Azure в Windows
description: Устранение неполадок службы файлов Azure в Windows. Ознакомьтесь с общими проблемами, относящимися к службе файлов Azure при подключении из клиентов Windows, и ознакомьтесь с возможными решениями. Только для общих ресурсов SMB
author: jeffpatt24
ms.service: storage
ms.topic: troubleshooting
ms.date: 09/13/2019
ms.author: jeffpatt
ms.subservice: files
ms.openlocfilehash: 242c0819e916f3ea7912d4d57b7d3e338152e4d9
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98878516"
---
# <a name="troubleshoot-azure-files-problems-in-windows-smb"></a>Устранение неполадок службы файлов Azure в Windows (SMB)

В этой статье приведен список распространенных проблем, возникающих в службе файлов Microsoft Azure при подключении из клиентов Windows. Кроме того, здесь представлены возможные причины этих проблем и способы их устранения. Помимо действий по устранению неполадок, описываемых в этой статье, можно также использовать [AzFileDiagnostics](https://github.com/Azure-Samples/azure-files-samples/tree/master/AzFileDiagnostics/Windows), чтобы обеспечить выполнение необходимых условий для среды клиента Windows. AzFileDiagnostics автоматизирует обнаружение большинства симптомов, упомянутых в этой статье, и помогает настроить среду для достижения оптимальной производительности.

> [!IMPORTANT]
> Содержимое этой статьи относится только к общим ресурсам SMB. Дополнительные сведения о общих ресурсах NFS см. в разделе [Устранение неполадок файловых ресурсов NFS Azure](storage-troubleshooting-files-nfs.md).

<a id="error5"></a>
## <a name="error-5-when-you-mount-an-azure-file-share"></a>Ошибка 5 при подключении файлового ресурса Azure

При попытке подключить файловый ресурс может выдаваться следующая ошибка:

- "Произошла системная ошибка 5. Отказано в доступе".

### <a name="cause-1-unencrypted-communication-channel"></a>Причина 1: незашифрованный коммуникационный канал

Из соображений безопасности, если коммуникационный канал не зашифрован, а попытка подключения осуществляется не из того же центра обработки данных, в котором находятся файловые ресурсы Azure, то подключение к ним Azure будет заблокировано. Незашифрованные подключения в одном центре обработки данных также могут быть заблокированы, если в учетной записи хранения включен параметр [Требуется безопасное перемещение](../common/storage-require-secure-transfer.md). Шифрование коммуникационного канала выполняется, только если клиентская ОС пользователя поддерживает шифрование SMB.

Windows 8, Windows Server 2012 или более поздние версии этих ОС отправляют запрос на согласование, который включает протокол в себя SMB 3.0, поддерживающий шифрование.

### <a name="solution-for-cause-1"></a>Решение для причины 1

1. Подключитесь из клиента, поддерживающего шифрование SMB (Windows 8, Windows Server 2012 или более поздней версии), или из виртуальной машины, размещенной в том же центре обработки данных, что и учетная запись хранения Azure, используемая для файлового ресурса Azure.
2. Убедитесь, что параметр [Требуется безопасное перемещение](../common/storage-require-secure-transfer.md) отключен в учетной записи хранения, если клиент не поддерживает шифрование SMB.

### <a name="cause-2-virtual-network-or-firewall-rules-are-enabled-on-the-storage-account"></a>Причина 2. в учетной записи хранения включены виртуальная сеть или правила брандмауэра. 

Если для учетной записи хранения настроены правила виртуальной сети и брандмауэра, доступ сетевого трафика будет запрещен, если IP-адрес клиента или виртуальная сеть не разрешают доступ.

### <a name="solution-for-cause-2"></a>Решение для причины 2

Убедитесь, что правила виртуальной сети и брандмауэра настроены надлежащим образом для учетной записи хранения. Чтобы проверить, связана ли проблема с правилами виртуальной сети или брандмауэра, временно задайте для учетной записи хранения параметр **Разрешить доступ из всех сетей**. Подробнее см. в статье [Настройка брандмауэров службы хранилища Azure и виртуальных сетей (предварительная версия)](../common/storage-network-security.md).

### <a name="cause-3-share-level-permissions-are-incorrect-when-using-identity-based-authentication"></a>Причина 3. при использовании проверки подлинности на основе удостоверений неправильные разрешения на уровне общего ресурса

Если пользователи обращаются к файловому ресурсу Azure с помощью проверки подлинности Active Directory (AD) или Azure Active Directory доменных служб (Azure AD DS), доступ к общей папке завершится ошибкой "доступ запрещен", если разрешения уровня общего доступа неверны. 

### <a name="solution-for-cause-3"></a>Решение для причины 3

Проверьте правильность настройки разрешений.

- **Active Directory (AD)** см. [в разделе Назначение разрешений на уровне общего ресурса удостоверению](./storage-files-identity-ad-ds-assign-permissions.md).

    Назначения разрешений на уровне общего ресурса поддерживаются для групп и пользователей, синхронизированных из Active Directory (AD) с Azure Active Directory (Azure AD) с помощью Azure AD Connect.  Убедитесь, что группы и пользователи, которым назначаются разрешения уровня общего доступа, не поддерживаются "облачными" группами.
- **Azure Active Directory доменных служб (Azure AD DS)** см. раздел [Назначение разрешений на доступ к удостоверению](./storage-files-identity-auth-active-directory-domain-service-enable.md?tabs=azure-portal#assign-access-permissions-to-an-identity).

<a id="error53-67-87"></a>
## <a name="error-53-error-67-or-error-87-when-you-mount-or-unmount-an-azure-file-share"></a>Ошибка 53, 67 или 87 при попытке подключения или отключения файлового ресурса Azure

При попытке подключить файловый ресурс из локальной среды или другого центра обработки данных могут возникнуть следующие ошибки:

- "Произошла системная ошибка 53. Сетевой путь не найден".
- "Произошла системная ошибка 67. Не найдено сетевое имя".
- "Произошла системная ошибка 87. Неправильный параметр".

### <a name="cause-1-port-445-is-blocked"></a>Причина 1: порт 445 заблокирован

Системная ошибка 53 или 67 может произойти, если исходящие подключения через порт 445 к центру обработки данных службы файлов Azure заблокированы. Чтобы просмотреть сведения о поставщиках услуг Интернета, поддерживающих (или не поддерживающих) подключение через порт 445, перейдите на сайт [TechNet](https://social.technet.microsoft.com/wiki/contents/articles/32346.azure-summary-of-isps-that-allow-disallow-access-from-port-445.aspx).

Чтобы проверить, не блокирует ли брандмауэр или поставщик услуг Интернета порт 445, используйте средство [AzFileDiagnostics](https://github.com/Azure-Samples/azure-files-samples/tree/master/AzFileDiagnostics/Windows) или командлет `Test-NetConnection`. 

Чтобы использовать `Test-NetConnection` командлет, необходимо установить модуль Azure PowerShell. Дополнительные сведения см. в разделе [Установка Azure PowerShell module](/powershell/azure/install-Az-ps) . Не забудьте заменить `<your-storage-account-name>` и `<your-resource-group-name>` соответствующими именами для вашей учетной записи хранения.

   
```azurepowershell
$resourceGroupName = "<your-resource-group-name>"
$storageAccountName = "<your-storage-account-name>"

# This command requires you to be logged into your Azure account, run Login-AzAccount if you haven't
# already logged in.
$storageAccount = Get-AzStorageAccount -ResourceGroupName $resourceGroupName -Name $storageAccountName

# The ComputerName, or host, is <storage-account>.file.core.windows.net for Azure Public Regions.
# $storageAccount.Context.FileEndpoint is used because non-Public Azure regions, such as sovereign clouds
# or Azure Stack deployments, will have different hosts for Azure file shares (and other storage resources).
Test-NetConnection -ComputerName ([System.Uri]::new($storageAccount.Context.FileEndPoint).Host) -Port 445
```
    
Если установка прошла успешно, отобразятся следующие выходные данные.
    
  
```azurepowershell
ComputerName     : <your-storage-account-name>
RemoteAddress    : <storage-account-ip-address>
RemotePort       : 445
InterfaceAlias   : <your-network-interface>
SourceAddress    : <your-ip-address>
TcpTestSucceeded : True
```
 

> [!Note]  
> Вышеуказанная команда возвращает текущий IP-адрес учетной записи хранения. Не гарантируется, что IP-адрес будет постоянным. Он может измениться в любое время. Не следует жестко задавать этот IP-адрес в скриптах или в конфигурации брандмауэра.

### <a name="solution-for-cause-1"></a>Решение для причины 1

#### <a name="solution-1---use-azure-file-sync"></a>Решение 1. Использование Синхронизации файлов Azure
Синхронизация файлов Azure можете преобразовать локальный сервер Windows Server в оперативный кэш файлового ресурса Azure. Для локального доступа к данным вы можете использовать любой протокол, доступный в Windows Server, в том числе SMB, NFS и FTPS. Синхронизация файлов Azure работает через порт 443 и, таким образом, может использоваться в качестве обходного пути для доступа к Файлам Azure клиентов, у которых заблокирован порт 445. [Узнайте, как настроить синхронизация файлов Azure](./storage-sync-files-extend-servers.md).

#### <a name="solution-2---use-vpn"></a>Решение 2. Использование VPN
Настроив VPN для конкретной учетной записи хранения, трафик будет проходить через защищенный туннель, а не через Интернет. Выполните [инструкции по настройке VPN](storage-files-configure-p2s-vpn-windows.md) для доступа к Файлам Azure из Windows.

#### <a name="solution-3---unblock-port-445-with-help-of-your-ispit-admin"></a>Решение 3. Разблокировка порта 445 с помощью поставщика услуг Интернета или ИТ-администратора
Обратитесь к своему ИТ-отделу или поставщику услуг Интернета, чтобы открыть порт 445 для исходящего трафика в [диапазоны IP Azure](https://www.microsoft.com/download/details.aspx?id=41653).

#### <a name="solution-4---use-rest-api-based-tools-like-storage-explorerpowershell"></a>Решение 4. Использование таких средств на основе REST API, как Обозреватель службы хранилища или PowerShell
Кроме того, служба "файлы Azure" поддерживает все остальное в дополнение к SMB. Доступ к REST осуществляется через порт 443 (стандартный протокол TCP). Существуют различные средства, написанные с помощью REST API, которые позволяют работать с многофункциональным пользовательским интерфейсом. [Обозреватель службы хранилища](../../vs-azure-tools-storage-manage-with-storage-explorer.md?tabs=windows) является одним из них. [Скачайте и установите Обозреватель службы хранилища](https://azure.microsoft.com/features/storage-explorer/), а затем подключитесь к общей папке с помощью Файлов Azure. Также можно использовать [PowerShell](./storage-how-to-use-files-powershell.md) , который также REST API пользователем.

### <a name="cause-2-ntlmv1-is-enabled"></a>Причина 2: Аутентификация ntlmv1 включен

Системная ошибка 53 или 87 может произойти, если в клиенте включена аутентификация NTLMv1. Служба файлов Azure поддерживает только проверку подлинности NTLMv2. Протокол проверки подлинности NTLMv1 делает клиент менее безопасным, в результате чего взаимодействие со службой файлов Azure блокируется. 

Чтобы определить, является ли причина ошибки, убедитесь, что в следующем подразделе реестра не указано значение меньше 3:

**HKLM\SYSTEM\CurrentControlSet\Control\Lsa > LmCompatibilityLevel**

Дополнительные сведения см. в статье [LmCompatibilityLevel](/previous-versions/windows/it-pro/windows-2000-server/cc960646(v=technet.10)) на сайте TechNet.

### <a name="solution-for-cause-2"></a>Решение для причины 2

Верните параметру **LmCompatibilityLevel** значение по умолчанию (3) в следующем подразделе реестра:

  **HKLM\SYSTEM\CurrentControlSet\Control\Lsa**

<a id="error1816"></a>
## <a name="error-1816---not-enough-quota-is-available-to-process-this-command"></a>Ошибка 1816-недостаточно квоты для обработки этой команды

### <a name="cause"></a>Причина

Ошибка 1816 возникает при достижении верхнего предела параллельных открытых дескрипторов, разрешенных для файла или каталога в общем файловом ресурсе Azure. Дополнительные сведения см. в разделе [Целевые показатели масштабируемости службы файлов Azure](./storage-files-scale-targets.md#azure-files-scale-targets).

### <a name="solution"></a>Решение

Сократите количество одновременно открытых дескрипторов, закрыв некоторые из них, и повторите попытку. Дополнительные сведения см. в разделе [Служба хранилища Microsoft Azure производительность и контрольный список масштабируемости](../blobs/storage-performance-checklist.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json).

Чтобы просмотреть открытые дескрипторы для общей папки, каталога или файла, используйте командлет PowerShell [Get-азсторажефилехандле](/powershell/module/az.storage/get-azstoragefilehandle) .  

Чтобы закрыть открытые дескрипторы для общей папки, каталога или файла, используйте командлет PowerShell [Close-азсторажефилехандле](/powershell/module/az.storage/close-azstoragefilehandle) .

> [!Note]  
> Командлеты Get-AzStorageFileHandle и Close-AzStorageFileHandle включены в команду AZ PowerShell Module 2,4 или более поздней версии. Чтобы установить последний модуль AZ PowerShell, см. статью [Установка модуля Azure PowerShell](/powershell/azure/install-az-ps).

<a id="noaaccessfailureportal"></a>
## <a name="error-no-access-when-you-try-to-access-or-delete-an-azure-file-share"></a>Ошибка "нет доступа" при попытке доступа или удаления файлового ресурса Azure  
При попытке доступа или удаления файлового ресурса Azure на портале может появиться следующее сообщение об ошибке:

Нет доступа  
Код ошибки: 403 

### <a name="cause-1-virtual-network-or-firewall-rules-are-enabled-on-the-storage-account"></a>Причина 1. в учетной записи хранения включены виртуальная сеть или правила брандмауэра

### <a name="solution-for-cause-1"></a>Решение для причины 1

Убедитесь, что правила виртуальной сети и брандмауэра настроены надлежащим образом для учетной записи хранения. Чтобы проверить, связана ли проблема с правилами виртуальной сети или брандмауэра, временно задайте для учетной записи хранения параметр **Разрешить доступ из всех сетей**. Подробнее см. в статье [Настройка брандмауэров службы хранилища Azure и виртуальных сетей (предварительная версия)](../common/storage-network-security.md).

### <a name="cause-2-your-user-account-does-not-have-access-to-the-storage-account"></a>Причина 2. у вашей учетной записи пользователя нет доступа к учетной записи хранения

### <a name="solution-for-cause-2"></a>Решение для причины 2

Перейдите в учетную запись хранения, в которой размещается общий файловый ресурс Azure, щелкните **Управление доступом (IAM)** и убедитесь, что учетная запись пользователя имеет доступ к учетной записи хранения. Дополнительные сведения см. в статье [Защита учетной записи хранения с помощью управления доступом на основе ролей Azure (Azure RBAC)](../blobs/security-recommendations.md#data-protection).

<a id="open-handles"></a>
## <a name="unable-to-modify-moverename-or-delete-a-file-or-directory"></a>Не удается изменить, переместить, переименовать или удалить файл или каталог
Одной из ключевых целей общей папки является то, что несколько пользователей и приложений могут одновременно взаимодействовать с файлами и каталогами в общей папке. Для помощи в этом взаимодействии файловые ресурсы предоставляют несколько способов обрабатывают доступа к файлам и каталогам.

При открытии файла из подключенного файлового ресурса Azure по протоколу SMB приложение или операционная система запрашивают файловый маркер, который является ссылкой на этот файл. Помимо прочего, приложение задает режим совместного использования файлов при запросе маркера файла, указывающего уровень исключительные права доступа к файлу, принудительно применяемый службой "файлы Azure": 

- `None`. у вас есть исключительный доступ. 
- `Read`: другие могут прочитать файл, пока он открыт.
- `Write`: другие могут записывать в файл, пока он открыт. 
- `ReadWrite`— сочетание обоих `Read` `Write` режимов совместного использования.
- `Delete`: другие могут удалить файл, пока он открыт. 

Несмотря на то, что в качестве протокола без отслеживания состояния протокол Филерест не имеет концепции дескрипторов файлов, он предоставляет аналогичный механизм для обеспечения доступа к файлам и папкам, которые могут использоваться сценарием, приложением или службой: арендами файлов. При аренде файла он считается эквивалентом обработчика файлов с режимом общего доступа к файлам `None` . 

Хотя дескрипторы файлов и аренды служат важной целью, иногда дескрипторы файлов и аренды могут быть утеряны. В этом случае это может привести к проблемам с изменением или удалением файлов. Могут отображаться такие сообщения об ошибках:

- The process cannot access the file because it is being used by another process.
- Невозможно выполнить действие, так как файл открыт в другой программе.
- Документ заблокирован для редактирования другим пользователем.
- Указанный ресурс помечен для удаления клиентом SMB.

Решение этой проблемы зависит от того, вызвана ли эта проблема потерянным обработчиком или арендой файлов. 

### <a name="cause-1"></a>Причина 1
Маркер файла предотвращает изменение или удаление файла или каталога. Для просмотра открытых дескрипторов можно использовать командлет PowerShell [Get-азсторажефилехандле](/powershell/module/az.storage/get-azstoragefilehandle) . 

Если все клиенты SMB закрыли открытые дескрипторы в файле или каталоге и проблема не будет устранена, можно принудительно закрыть дескриптор файла.

### <a name="solution-1"></a>Решение 1
Чтобы принудительно закрыть обработчик файлов, используйте командлет PowerShell [Close-азсторажефилехандле](/powershell/module/az.storage/close-azstoragefilehandle) . 

> [!Note]  
> Командлеты Get-AzStorageFileHandle и Close-AzStorageFileHandle включены в команду AZ PowerShell Module 2,4 или более поздней версии. Чтобы установить последний модуль AZ PowerShell, см. статью [Установка модуля Azure PowerShell](/powershell/azure/install-az-ps).

### <a name="cause-2"></a>Причина 2
Аренда файла предотвращает изменение или удаление файла. Вы можете проверить, имеет ли файл аренду файла со следующей оболочкой PowerShell, заменив `<resource-group>` ,, `<storage-account>` `<file-share>` и `<path-to-file>` соответствующими значениями для вашей среды:

```PowerShell
# Set variables 
$resourceGroupName = "<resource-group>"
$storageAccountName = "<storage-account>"
$fileShareName = "<file-share>"
$fileForLease = "<path-to-file>"

# Get reference to storage account
$storageAccount = Get-AzStorageAccount `
        -ResourceGroupName $resourceGroupName `
        -Name $storageAccountName

# Get reference to file
$file = Get-AzStorageFile `
        -Context $storageAccount.Context `
        -ShareName $fileShareName `
        -Path $fileForLease

$fileClient = $file.ShareFileClient

# Check if the file has a file lease
$fileClient.GetProperties().Value
```

Если у файла есть аренда, возвращаемый объект должен содержать следующие свойства:

```Output
LeaseDuration         : Infinite
LeaseState            : Leased
LeaseStatus           : Locked
```

### <a name="solution-2"></a>Решение 2
Чтобы удалить аренду файла, можно освободить аренду или прервать ее. Чтобы освободить аренду, требуется идентификатор LeaseId аренды, который задается при создании аренды. Для прерывания аренды LeaseId не требуется.

В следующем примере показано, как прервать аренду для файла, указанного в причине 2 (этот пример продолжится с переменными PowerShell из причины 2):

```PowerShell
$leaseClient = [Azure.Storage.Files.Shares.Specialized.ShareLeaseClient]::new($fileClient)
$leaseClient.Break() | Out-Null
```

<a id="slowfilecopying"></a>
## <a name="slow-file-copying-to-and-from-azure-files-in-windows"></a>Медленное копирование файлов в службе файлов Azure и из нее в Windows

При попытке передать файлы в службу файлов Azure можно наблюдать низкую производительность.

- Если определенные требования к минимальному размеру операций ввода-вывода отсутствуют, для оптимальной производительности мы рекомендуем использовать 1 МиБ.
-   Если окончательный размер файла, в который выполняются операции записи, известен, а у программного обеспечения нет проблем совместимости, и если еще не записанный заключительный фрагмент файла содержит нули, укажите размер файла заранее вместо того, чтобы расширять его для каждой операции записи.
-   Используйте правильный метод копирования:
    -   Используйте [AZCopy](../common/storage-use-azcopy-v10.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json) для передачи данных между двумя файловыми ресурсами.
    -   Используйте [Robocopy](./storage-how-to-create-file-share.md) для передачи данных между файловыми ресурсами и локальным компьютером.

### <a name="considerations-for-windows-81-or-windows-server-2012-r2"></a>Рекомендации для Windows 8.1 или Windows Server 2012 R2

Для клиентов, использующих Windows 8.1 или Windows Server 2012 R2, следует установить исправление [KB3114025](https://support.microsoft.com/help/3114025). Это исправление повышает производительность операций создания и закрытия дескрипторов.

Выполните следующий сценарий, чтобы проверить, установлено ли это исправление.

`reg query HKLM\SYSTEM\CurrentControlSet\Services\LanmanWorkstation\Parameters\Policies`

Если это так, выводятся следующие данные:

`HKEY_Local_MACHINE\SYSTEM\CurrentControlSet\Services\LanmanWorkstation\Parameters\Policies {96c345ef-3cac-477b-8fcd-bea1a564241c} REG_DWORD 0x1`

> [!Note]
> Начиная с декабря 2015 года в образах Windows Server 2012 R2, содержащихся в Azure Marketplace, исправление KB3114025 установлено по умолчанию.

<a id="shareismissing"></a>
## <a name="no-folder-with-a-drive-letter-in-my-computer-or-this-pc"></a>Нет папки с буквой диска в "Мой компьютер" или "этом компьютере"

Если вы сопоставили файловый ресурс Azure от имени администратора с помощью команды net use, то может показаться, что он отсутствует.

### <a name="cause"></a>Причина

По умолчанию проводник не запускается от имени администратора. При выполнении команды net use из командной строки администрирования пользователь подключает сетевой диск от имени администратора. Подключенные диски ориентированы на пользователя. Если для их подключения использовалась одна учетная запись, а пользователь вошел в систему с помощью другой, то диски отображаться не будут.

### <a name="solution"></a>Решение
Подключите общую папку из командной строки без прав администратора. Кроме того, вы можете воспользоваться [этим разделом TechNet](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/ee844140(v=ws.10)) , чтобы настроить значение реестра **енаблелинкедконнектионс** .

<a id="netuse"></a>
## <a name="net-use-command-fails-if-the-storage-account-contains-a-forward-slash"></a>Если учетная запись хранения содержит косую черту (/), то выполнение команды net use завершается сбоем

### <a name="cause"></a>Причина

Команда net use интерпретирует косую черту (/) как параметр командной строки. Если имя учетной записи пользователя начинается с косой черты, то сопоставление диска завершится сбоем.

### <a name="solution"></a>Решение

Чтобы обойти эту проблему, выполните одно из следующих действий.

- Выполните следующую команду PowerShell:

  `New-SmbMapping -LocalPath y: -RemotePath \\server\share -UserName accountName -Password "password can contain / and \ etc"`

  Из пакетного файла выполнить команду можно следующим образом.

  `Echo new-smbMapping ... | powershell -command –`

- Заключите значение ключа в двойные прямые кавычки (если первый знак не является косой чертой). Если значение ключа начинается с косой черты (/), используйте интерактивный режим и введите пароль отдельно или повторно создайте ключи, которые не начинаются с этого знака.

<a id="cannotaccess"></a>
## <a name="application-or-service-cannot-access-a-mounted-azure-files-drive"></a>Приложение или служба не может получить доступ к подключенному диску службы файлов Azure

### <a name="cause"></a>Причина

Диски подключаются для каждого пользователя. Если приложение или служба выполняется не под той учетной записью, к которой относится подключенный диск, то приложение не увидит этот диск.

### <a name="solution"></a>Решение

Используйте одно из следующих решений:

-   Используйте для подключения дисков учетную запись пользователя, содержащую это приложение. Можно использовать такой инструмент, как PsExec.
- Передайте имя и ключ учетной записи хранения в параметры имени пользователя и пароля команды net use.
- Чтобы добавить учетные данные в диспетчере учетных данных, используйте команду cmdkey. Это можно сделать из командной строки в контексте учетной записи службы с помощью интерактивного входа или с помощью `runas` .
  
  `cmdkey /add:<storage-account-name>.file.core.windows.net /user:AZURE\<storage-account-name> /pass:<storage-account-key>`
- Сопоставьте общий ресурс напрямую, не используя букву подключенного диска. Если используется буква диска, некоторые приложения могут неправильно подключиться к диску, поэтому надежнее использовать полный UNC-путь. 

  `net use * \\storage-account-name.file.core.windows.net\share`

После выполнения этих инструкций при выполнении команды net use для учетной записи системы или сети может появиться следующее сообщение об ошибке: "Произошла системная ошибка 1312. A specified logon session does not exist. Возможно, он уже завершен". В этом случае убедитесь, что имя пользователя, переданное в net use, содержит сведения о домене (например, [имя_учетной_записи_хранения].file.core.windows.net).

<a id="doesnotsupportencryption"></a>
## <a name="error-you-are-copying-a-file-to-a-destination-that-does-not-support-encryption"></a>Ошибка "Вы копируете файл в место, которое не поддерживает шифрование"

Когда файл копируется по сети, он расшифровывается на исходном компьютере, передается в виде обычного текста и повторно шифруется в месте назначения. Тем не менее при попытке скопировать зашифрованный файл может появиться следующее сообщение об ошибке: "Вы копируете файл в место, которое не поддерживает шифрование".

### <a name="cause"></a>Причина
Эта проблема может возникнуть при использовании шифрованной файловой системы (EFS). Файлы с шифрованием BitLocker нельзя копировать в службу файлов Azure. Однако эта служба не поддерживает шифрованную файловую систему (EFS) NTFS.

### <a name="workaround"></a>Обходной путь
Чтобы скопировать файл по сети, его сначала необходимо расшифровать. Используйте один из следующих методов.

- Используйте команду **copy /d**. Она позволяет сохранить зашифрованные файлы как расшифрованные файлы в месте назначения.
- Настройте следующий раздел реестра:
  - путь: HKLM\Software\Policies\Microsoft\Windows\System;
  - тип значения: DWORD;
  - Name = CopyFileAllowDecryptedRemoteDestination;
  - Value = 1.

Помните, что настройка раздела реестра влияет на все операции копирования в сетевые общие папки.

## <a name="slow-enumeration-of-files-and-folders"></a>Медленное перечисление файлов и папок

### <a name="cause"></a>Причина

Эта проблема может возникнуть, если на клиентском компьютере для больших каталогов недостаточно кэша.

### <a name="solution"></a>Решение

Чтобы устранить эту проблему, настройте значение реестра **DirectoryCacheEntrySizeMax** разрешать кэшировать большие списки каталогов на компьютере клиента.

- Расположение: HKLM\System\CCS\Services\Lanmanworkstation\Parameters
- Название значения: DirectoryCacheEntrySizeMax 
- Тип значения: DWORD
 
 
Например, его можно присвоить 0x100000 и просмотреть станет ли производительность лучше.

## <a name="error-aaddstenantnotfound-in-enabling-azure-active-directory-domain-service-azure-ad-ds-authentication-for-azure-files-unable-to-locate-active-tenants-with-tenant-id-aad-tenant-id"></a>Ошибка Ааддстенантнотфаунд при включении проверки подлинности Azure Active Directory службы домена (Azure AD DS) для файлов Azure "не удается нахождение активных клиентов с ИДЕНТИФИКАТОРом клиента AAD-клиент-ID"

### <a name="cause"></a>Причина

Ошибка Ааддстенантнотфаунд возникает при попытке [включить проверку подлинности Azure Active Directory доменных служб (azure AD DS) в службе файлов Azure](storage-files-identity-auth-active-directory-domain-service-enable.md) в учетной записи хранения, где [Служба домена Azure ad (Azure AD DS)](../../active-directory-domain-services/overview.md) не создана в клиенте Azure AD связанной подписки.  

### <a name="solution"></a>Решение

Включите AD DS Azure в клиенте Azure AD подписки, в которой развернута ваша учетная запись хранения. Для создания управляемого домена требуются права администратора клиента Azure AD. Если вы не являетесь администратором клиента Azure AD, обратитесь к администратору и следуйте пошаговым инструкциям, чтобы [создать и настроить управляемый домен доменных служб Azure Active Directory](../../active-directory-domain-services/tutorial-create-instance.md).

[!INCLUDE [storage-files-condition-headers](../../../includes/storage-files-condition-headers.md)]

## <a name="unable-to-mount-azure-files-with-ad-credentials"></a>Не удается подключить службу файлов Azure с учетными данными AD 

### <a name="self-diagnostics-steps"></a>Действия самодиагностики
Сначала убедитесь, что выполнены все четыре шага, чтобы [включить проверку подлинности AD для службы файлов Azure](./storage-files-identity-auth-active-directory-enable.md).

Во вторых, попробуйте подключить [файловый ресурс Azure к ключу учетной записи хранения](./storage-how-to-use-files-windows.md). Если не удалось выполнить подключение, скачайте [AzFileDiagnostics.ps1](https://github.com/Azure-Samples/azure-files-samples/tree/master/AzFileDiagnostics/Windows) , чтобы помочь вам проверить работающую среду клиента, определить несовместимую конфигурацию клиента, которая вызовет сбой доступа для службы "файлы Azure", предоставит рекомендации по самостоятельному исправлению и собирайте трассировки диагностики.

В-третьих, можно запустить командлет Debug-AzStorageAccountAuth, чтобы выполнить ряд основных проверок конфигурации AD с пользователем, вошедшим в систему Active Directory. Этот командлет поддерживается в [AzFilesHybrid версии 0.1.2+](https://github.com/Azure-Samples/azure-files-samples/releases). Необходимо выполнить этот командлет от имени пользователя AD, имеющего разрешение владельца целевой учетной записи хранения.  
```PowerShell
$ResourceGroupName = "<resource-group-name-here>"
$StorageAccountName = "<storage-account-name-here>"

Debug-AzStorageAccountAuth -StorageAccountName $StorageAccountName -ResourceGroupName $ResourceGroupName -Verbose
```
Командлет выполняет следующие проверки по порядку и предоставляет рекомендации по сбоям:
1. Чеккадобжектпассвордискоррект: Убедитесь, что пароль, настроенный для удостоверения AD, представляющего учетную запись хранения, совпадает с ключом kerb1 или kerb2 учетной записи хранения. Если пароль неверный, можно выполнить команду [Update-азсторажеаккаунтадобжектпассворд](./storage-files-identity-ad-ds-update-password.md) , чтобы сбросить пароль. 
2. Чеккадобжект: Убедитесь, что в Active Directory есть объект, который представляет учетную запись хранения и имеет правильное имя участника-службы (SPN). Если имя субъекта-службы не настроено правильно, выполните командлет Set AD, возвращенный в командлете Debug, чтобы настроить имя SPN.
3. Чеккдомаинжоинед: Проверка того, что компьютер клиента присоединен к домену AD. Если компьютер не присоединен к домену AD, обратитесь к этой [статье](/windows-server/identity/ad-fs/deployment/join-a-computer-to-a-domain) для участия в инструкции по присоединению к домену.
4. CheckPort445Connectivity: Убедитесь, что для подключения SMB открыт порт 445. Если требуемый порт не открыт, обратитесь к средству устранения неполадок [AzFileDiagnostics.ps1](https://github.com/Azure-Samples/azure-files-samples/tree/master/AzFileDiagnostics/Windows) для устранения проблем с подключением к службе файлов Azure.
5. Чекксидхасаадусер: Убедитесь, что вошедший в систему пользователь AD синхронизирован с Azure AD. Если вы хотите узнать, синхронизирован ли конкретный пользователь AD с Azure AD, можно указать параметры-UserName и-domain во входных параметрах. 
6. Чеккжеткерберостиккет: попытка получить билет Kerberos для подключения к учетной записи хранения. Если нет допустимого маркера Kerberos, выполните командлет klist Get CIFS/Storage-Account-Name. File. Core. Windows. NET и изучите код ошибки, чтобы вызвать ошибку получения билета.
7. Чекксторажеаккаунтдомаинжоинед: Проверка включения проверки подлинности AD и заполнение свойств учетной записи AD. Если [это](./storage-files-identity-ad-ds-enable.md) не так, см. инструкции по включению AD DS проверки подлинности в службе файлов Azure. 
8. Чеккусеррбакассигнмент: Проверьте, имеет ли пользователь AD правильное назначение роли RBAC, чтобы предоставить разрешение на уровне общего ресурса для доступа к службе "файлы Azure". Если это не так, см. [инструкции по настройке разрешения на уровне](./storage-files-identity-ad-ds-assign-permissions.md) общего ресурса. (Поддерживается в версии Азфилешибрид v 0.2.3 +)
9. Чеккусерфилеакцесс: Проверьте, имеет ли пользователь AD соответствующие разрешения на доступ к файлам и каталогам (ACL Windows) для пользователя Active Directory. Если это не так, см. [инструкции по настройке разрешения на уровне](./storage-files-identity-ad-ds-configure-permissions.md) каталога и файлов. (Поддерживается в версии Азфилешибрид v 0.2.3 +)

## <a name="unable-to-configure-directoryfile-level-permissions-windows-acls-with-windows-file-explorer"></a>Не удается настроить разрешения на уровне каталога или файла (списки управления доступом Windows) с помощью проводника Windows

### <a name="symptom"></a>Симптом

При попытке настроить списки управления доступом Windows с помощью проводника на подключенном файловом ресурсе могут возникнуть описанные ниже симптомы.
- После щелчка разрешения изменить на вкладке Безопасность мастер разрешений не загружается. 
- При попытке выбрать нового пользователя или группу в расположении домена не отображается правильный домен AD DS. 

### <a name="solution"></a>Решение

Рекомендуется использовать [средство icacls](/windows-server/administration/windows-commands/icacls) для настройки разрешений на уровне каталога или файла в качестве обходного пути. 

## <a name="errors-when-running-join-azstorageaccountforauth-cmdlet"></a>Ошибки при выполнении командлета Join-AzStorageAccountForAuth

### <a name="error-the-directory-service-was-unable-to-allocate-a-relative-identifier"></a>Ошибка: "службе каталогов не удалось выделить относительный идентификатор"

Эта ошибка может возникать, если контроллер домена, на котором находится роль FSMO хозяина RID, недоступен или был удален из домена и восстановлен из резервной копии.  Убедитесь, что все контроллеры домена работают и доступны.

### <a name="error-cannot-bind-positional-parameters-because-no-names-were-given"></a>Ошибка: "Не удается привязать позиционные параметры, поскольку имена не предоставлены"

Эта ошибка чаще всего активируется при синтаксической ошибке в команде Join-AzStorageAccountforAuth.  Проверьте правильность написания или синтаксических ошибок в команде и убедитесь, что установлена последняя версия модуля Азфилешибрид ( https://github.com/Azure-Samples/azure-files-samples/releases) установлен.  

## <a name="azure-files-on-premises-ad-ds-authentication-support-for-aes-256-kerberos-encryption"></a>Локальная поддержка проверки подлинности AD DS файлов Azure для шифрования AES 256 Kerberos

Мы предоставили поддержку шифрования Kerberos 256 для локальных AD DS аутентификации в службе файлов Azure с помощью [модуля азфилешибрид v 0.2.2](https://github.com/Azure-Samples/azure-files-samples/releases). Если вы включили AD DSную проверку подлинности с версией модуля ниже v 0.2.2, вам потребуется скачать последнюю версию модуля Азфилешибрид (v 0.2.2 +) и запустить PowerShell ниже. Если вы еще не включили проверку подлинности AD DS в учетной записи хранения, вы можете воспользоваться этим [руководством](./storage-files-identity-ad-ds-enable.md#option-one-recommended-use-azfileshybrid-powershell-module) для включения. 

```PowerShell
$ResourceGroupName = "<resource-group-name-here>"
$StorageAccountName = "<storage-account-name-here>"

Update-AzStorageAccountAuthForAES256 -ResourceGroupName $ResourceGroupName -StorageAccountName $StorageAccountName
```


## <a name="need-help-contact-support"></a>Требуется помощь? Обратитесь в службу поддержки.
Если вам все еще нужна помощь, [обратитесь в службу поддержки](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade), которая поможет быстро устранить проблему.
