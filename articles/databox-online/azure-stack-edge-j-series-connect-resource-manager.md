---
title: Подключение к Azure Resource Manager на устройстве Azure Stack ребра Pro GPU
description: В этой статье описывается, как подключиться к Azure Resource Manager, работающей на графическом процессоре Azure Stack ребра Pro, с помощью Azure PowerShell.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 03/01/2021
ms.author: alkohli
ms.openlocfilehash: 37d1b1613f4d346fa22d10e338f442c17b8425a0
ms.sourcegitcommit: 5bbc00673bd5b86b1ab2b7a31a4b4b066087e8ed
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102439006"
---
# <a name="connect-to-azure-resource-manager-on-your-azure-stack-edge-pro-device"></a>Подключение к Azure Resource Manager на устройстве Azure Stack ребра Pro

[!INCLUDE [applies-to-GPU-and-pro-r-and-mini-r-skus](../../includes/azure-stack-edge-applies-to-gpu-pro-r-mini-r-sku.md)]

Azure Resource Manager обеспечивает уровень управления, позволяющий создавать, обновлять и удалять ресурсы в подписке Azure. Устройство Azure Stack пограничной Pro поддерживает те же интерфейсы API Azure Resource Manager для создания, обновления и удаления виртуальных машин в локальной подписке. Эта поддержка позволяет управлять устройством способом, согласованным с облаком. 

В этом руководстве описывается, как подключиться к локальным API на устройстве Azure Stack ребра Pro с помощью Azure Resource Manager Azure PowerShell.

## <a name="about-azure-resource-manager"></a>Общие сведения об Azure Resource Manager

Azure Resource Manager предоставляет последовательный уровень управления для вызова API-интерфейса устройства Azure Stack ребра и выполнения таких операций, как создание, обновление и удаление виртуальных машин. Архитектура Azure Resource Manager подробно описана на следующей схеме.

![Схема для Azure Resource Manager](media/azure-stack-edge-j-series-connect-resource-manager/edge-device-flow.svg)


## <a name="endpoints-on-azure-stack-edge-pro-device"></a>Конечные точки на устройстве Pro Azure Stack

В следующей таблице перечислены различные конечные точки, предоставляемые на устройстве, Поддерживаемые протоколы и порты для доступа к этим конечным точкам. На протяжении всей статьи вы найдете ссылки на эти конечные точки.

| # | Конечная точка | Поддерживаемые протоколы | Используемый порт | Используется для |
| --- | --- | --- | --- | --- |
| 1. | Azure Resource Manager | HTTPS | 443 | Подключение к Azure Resource Manager для автоматизации |
| 2. | Служба маркеров безопасности | HTTPS | 443 | Проверка подлинности с помощью маркеров доступа и обновления |
| 3. | BLOB-объект | HTTPS | 443 | Подключение к хранилищу BLOB-объектов с помощью протокола RESTFUL |


## <a name="connecting-to-azure-resource-manager-workflow"></a>Подключение к рабочему процессу Azure Resource Manager

Процесс подключения к локальным API-интерфейсам устройства с помощью Azure Resource Manager требует выполнения следующих действий.

| Номер шага | Вы выполните этот шаг... | .. в этом расположении. |
| --- | --- | --- |
| 1. | [Настройка устройства Azure Stack ребра Pro](#step-1-configure-azure-stack-edge-pro-device) | Локальный пользовательский веб-интерфейс |
| 2. | [Создание и установка сертификатов](#step-2-create-and-install-certificates) | Клиентский или локальный веб-интерфейс Windows |
| 3. | [Проверка и настройка необходимых компонентов](#step-3-install-powershell-on-the-client) | Клиент Windows |
| 4. | [Настройка Azure PowerShell на клиенте](#step-4-set-up-azure-powershell-on-the-client) | Клиент Windows |
| 5. | [Изменение файла узла для разрешения имени конечной точки](#step-5-modify-host-file-for-endpoint-name-resolution) | Клиент Windows или DNS-сервер |
| 6. | [Проверка разрешения имени конечной точки](#step-6-verify-endpoint-name-resolution-on-the-client) | Клиент Windows |
| 7. | [Используйте командлеты Azure PowerShell, чтобы проверить подключение к Azure Resource Manager](#step-7-set-azure-resource-manager-environment) | Клиент Windows |

В следующих разделах подробно описаны все шаги, описанные в разделе Подключение к Azure Resource Manager.

## <a name="prerequisites"></a>Предварительные требования

Прежде чем начать, убедитесь, что клиент, используемый для подключения к устройству через Azure Resource Manager, использует TLS 1,2. Дополнительные сведения см. в подразделах [Настройка TLS 1,2 на клиенте Windows, обращающемся к Azure Stack пограничном устройстве Pro](azure-stack-edge-j-series-configure-tls-settings.md).

## <a name="step-1-configure-azure-stack-edge-pro-device"></a>Шаг 1. Настройка устройства Azure Stack ребра Pro 

Выполните следующие действия в локальном веб-ИНТЕРФЕЙСе устройства Azure Stack ребра Pro.

1. Заполните параметры сети для устройства Azure Stack ребра Pro. 

    ![Страница "Параметры сети" в локальном веб-интерфейсе](./media/azure-stack-edge-gpu-deploy-configure-network-compute-web-proxy/compute-network-2.png)


    Запишите IP-адрес устройства. Этот IP-адрес будет использоваться позже.

2. Настройте имя устройства и домен DNS на странице **устройства** . Запишите имя устройства и домен DNS, которые будут использоваться позже.

    ![Страница "Устройство" в локальном пользовательском веб-интерфейсе](./media/azure-stack-edge-gpu-deploy-set-up-device-update-time/device-2.png)

    > [!IMPORTANT]
    > Имя устройства, домен DNS будет использоваться для формирования конечных точек, которые предоставляются.
    > Используйте конечные точки Azure Resource Manager и больших двоичных объектов на странице **устройства** в локальном веб-интерфейсе.


## <a name="step-2-create-and-install-certificates"></a>Шаг 2. Создание и установка сертификатов

Сертификаты гарантируют, что ваш обмен данными является доверенным. На устройстве Azure Stack пограничной Pro автоматически создаются самозаверяющие устройства, BLOB-объекты и Azure Resource Manager сертификаты. При необходимости можно также включить собственный подписанный BLOB-объект и Azure Resource Manager сертификаты.

Если вы понесете собственный подписанный сертификат, вам также потребуется соответствующая цепочка подписывания сертификата. Для цепочки подписывания Azure Resource Manager и сертификатов больших двоичных объектов на устройстве потребуются также соответствующие сертификаты на клиентском компьютере для проверки подлинности и взаимодействия с устройством.

Чтобы подключиться к Azure Resource Manager, необходимо создать или получить цепочку подписывания и сертификаты конечной точки, импортировать эти сертификаты в клиенте Windows и, наконец, передать эти сертификаты на устройство.

### <a name="create-certificates-optional"></a>Создание сертификатов (необязательно)

Для использования только для тестирования и разработки можно использовать Windows PowerShell для создания сертификатов в локальной системе. При создании сертификатов для клиента следуйте приведенным ниже рекомендациям.

1. Сначала необходимо создать корневой сертификат для цепочки подписывания. Дополнительные сведения см. в разделе шаги по [созданию сертификатов цепочки подписывания](azure-stack-edge-gpu-manage-certificates.md#create-signing-chain-certificate).

2. Затем можно создать сертификаты конечной точки для большого двоичного объекта и Azure Resource Manager. Эти конечные точки можно получить на странице **устройства** в локальном веб-интерфейсе. См. инструкции по [созданию сертификатов конечной точки](azure-stack-edge-gpu-manage-certificates.md#create-signed-endpoint-certificates).

3. Для всех этих сертификатов убедитесь, что имя субъекта и альтернативное имя субъекта соответствуют следующим рекомендациям:

    |Type |Имя субъекта (SN)  |Альтернативное имя субъекта (SAN)  |Пример имени субъекта |
    |---------|---------|---------|---------|
    |Azure Resource Manager|`management.<Device name>.<Dns Domain>`|`login.<Device name>.<Dns Domain>`<br>`management.<Device name>.<Dns Domain>`|`management.mydevice1.microsoftdatabox.com` |
    |Хранилище BLOB-объектов|`*.blob.<Device name>.<Dns Domain>`|`*.blob.< Device name>.<Dns Domain>`|`*.blob.mydevice1.microsoftdatabox.com` |
    |Один сертификат с несколькими SAN для обеих конечных точек|`<Device name>.<dnsdomain>`|`login.<Device name>.<Dns Domain>`<br>`management.<Device name>.<Dns Domain>`<br>`*.blob.<Device name>.<Dns Domain>`|`mydevice1.microsoftdatabox.com` |

Дополнительные сведения о сертификатах см. в руководствах по [управлению сертификатами](azure-stack-edge-gpu-manage-certificates.md).

### <a name="upload-certificates-on-the-device"></a>Отправка сертификатов на устройство

Сертификаты, созданные на предыдущем шаге, будут находиться в личном хранилище на клиенте. Эти сертификаты необходимо экспортировать на клиент в соответствующие файлы форматирования, которые затем можно отправить на устройство.

1. Корневой сертификат необходимо экспортировать как файл формата DER с расширением *CER* . Подробные инструкции см. в разделе [Экспорт сертификатов в виде CER-файла форматирования](azure-stack-edge-gpu-manage-certificates.md#export-certificates-as-der-format).

2. Сертификаты конечной точки должны быть экспортированы в виде *PFX* -файлов с закрытыми ключами. Подробные инструкции см. в разделе [Экспорт сертификатов как PFX-файла с закрытыми ключами](azure-stack-edge-gpu-manage-certificates.md#export-certificates-as-pfx-format-with-private-key).

3. Затем корневые и конечные сертификаты передаются на устройство с помощью параметра **+ Добавить сертификат** на странице **Сертификаты** в локальном веб-интерфейсе. Чтобы передать сертификаты, выполните действия, описанные в разделе [Отправка сертификатов](azure-stack-edge-gpu-manage-certificates.md#upload-certificates).


### <a name="import-certificates-on-the-client-running-azure-powershell"></a>Импорт сертификатов на клиенте, на котором работает Azure PowerShell

Клиент Windows, на котором будет вызывать API Azure Resource Manager, должен установить доверие с устройством. Для этого сертификаты, созданные на предыдущем шаге, необходимо импортировать на клиент Windows в соответствующее хранилище сертификатов.

1. Корневой сертификат, экспортированный в формате DER с расширением *CER* , теперь должен импортироваться в Доверенные корневые центры сертификации в клиентской системе. Подробные инструкции см. в разделе [Импорт сертификатов в хранилище доверенных корневых центров сертификации.](azure-stack-edge-gpu-manage-certificates.md#import-certificates-as-der-format)

2. Сертификаты конечной точки, экспортированные в виде *PFX* -файла, должны быть экспортированы в виде *CER*-файла. Затем этот *CER* -объект импортируется в **личное** хранилище сертификатов в вашей системе. Подробные инструкции см. [в разделе Импорт сертификатов в личное хранилище](azure-stack-edge-gpu-manage-certificates.md#import-certificates-as-der-format).

## <a name="step-3-install-powershell-on-the-client"></a>Шаг 3. Установка PowerShell на клиенте 

Клиент Windows должен соответствовать следующим предварительным требованиям:

1. Запустите PowerShell версии 5,0. Необходимо наличие PowerShell версии 5,0. PowerShell Core не поддерживается. Чтобы проверить версию PowerShell в системе, выполните следующий командлет:

    ```powershell
    $PSVersionTable.PSVersion
    ```

    Сравните **основную** версию и убедитесь, что она 5,0 или более поздней версии.

    Если у вас более старая версия, см. раздел [Обновление существующей версии Windows PowerShell](/powershell/scripting/install/installing-windows-powershell?view=powershell-6&preserve-view=true#upgrading-existing-windows-powershell).

    Если у вас \' нет PowerShell 5,0, следуйте инструкциям по [установке Windows PowerShell](/powershell/scripting/install/installing-windows-powershell?view=powershell-6&preserve-view=true).

    Результат выполнения команды показан ниже.

    ```powershell
    Windows PowerShell
    Copyright (C) Microsoft Corporation. All rights reserved. 
    Try the new cross-platform PowerShell https://aka.ms/pscore6
    PS C:\windows\system32> $PSVersionTable.PSVersion
    Major  Minor  Build  Revision
    -----  -----  -----  --------
    5      1      18362  145
    ```
    
2. Вы можете получить доступ к коллекция PowerShell.

    Запустите оболочку PowerShell от имени администратора. Проверьте, зарегистрирован ли объект в `PSGallery` качестве репозитория.

    ```powershell
    Import-Module -Name PowerShellGet -ErrorAction Stop
    Import-Module -Name PackageManagement -ErrorAction Stop
    Get-PSRepository -Name "PSGallery"
    ```
    
    Результат выполнения команды показан ниже.
    
    ```powershell
    PS C:\windows\system32> Import-Module -Name PowerShellGet -ErrorAction Stop
    PS C:\windows\system32> Import-Module -Name PackageManagement -ErrorAction Stop
    PS C:\windows\system32> Get-PSRepository -Name "PSGallery"
    Name                      InstallationPolicy   SourceLocation
    ----                      ------------------   --------------
    PSGallery                 Trusted              https://www.powershellgallery.com/api/v2
    ```
    
Если репозиторий не является доверенным или требуются дополнительные сведения, см. [статью Проверка специальных возможностей коллекция PowerShell](/azure-stack/operator/azure-stack-powershell-install?view=azs-1908&preserve-view=true&preserve-view=true#2-validate-the-powershell-gallery-accessibility).

## <a name="step-4-set-up-azure-powershell-on-the-client"></a>Шаг 4. Настройка Azure PowerShell на клиенте 

<!--1. Verify the API profile of the client and identify which version of the Azure PowerShell modules and libraries to include on your client. In this example, the client system will be running Azure Stack 1904 or later. For more information, see [Azure Resource Manager API profiles](/azure-stack/user/azure-stack-version-profiles?view=azs-1908#azure-resource-manager-api-profiles).-->

1. На клиенте будут установлены модули Azure PowerShell, которые будут работать с устройством.

    а. Откройте сеанс PowerShell от имени администратора. Вам нужен доступ к коллекции PowerShell. 


    b. Чтобы установить необходимые модули Azure PowerShell из коллекция PowerShell, выполните следующую команду:

    ```powershell
    # Install the AzureRM.BootStrapper module. Select Yes when prompted to install NuGet.
    
    Install-Module -Name AzureRM.BootStrapper
    
   # Install and import the API Version Profile into the current PowerShell session.
    
    Use-AzureRmProfile -Profile 2019-03-01-hybrid -Force
    
    # Confirm the installation of PowerShell
    Get-Module -Name "Azure*" -ListAvailable
    ```
    
    Убедитесь, что в конце установки установлен модуль Azure-RM версии 2.5.0. 
    При наличии существующей версии модуля Azure RM, которая не соответствует требуемой версии, удалите ее с помощью следующей команды:

    `Get-Module -Name Azure* -ListAvailable | Uninstall-Module -Force -Verbose`

    Теперь необходимо снова установить требуемую версию.
   

Ниже приведен пример выходных данных, указывающий на то, что модули AzureRM версии 2.5.0 успешно установлены.

```powershell
PS C:\windows\system32> Install-Module -Name AzureRM.BootStrapper
PS C:\windows\system32> Use-AzureRmProfile -Profile 2019-03-01-hybrid -Force
Loading Profile 2019-03-01-hybrid
PS C:\windows\system32> Get-Module -Name "Azure*" -ListAvailable
 
    Directory: C:\Program Files\WindowsPowerShell\Modules
 
ModuleType Version    Name                          ExportedCommands
---------- -------    ----                          ----------------
Script     4.5.0      Azure.Storage                       {Get-AzureStorageTable, New-AzureStorageTableSASToken, New...
Script     2.5.0      AzureRM
Script     0.5.0      AzureRM.BootStrapper                {Update-AzureRmProfile, Uninstall-AzureRmProfile, Install-...
Script     4.6.1      AzureRM.Compute                     {Remove-AzureRmAvailabilitySet, Get-AzureRmAvailabilitySet...
Script     3.5.1      AzureRM.Dns                         {Get-AzureRmDnsRecordSet, New-AzureRmDnsRecordConfig, Remo...
Script     5.1.5      AzureRM.Insights                    {Get-AzureRmMetricDefinition, Get-AzureRmMetric, Remove-Az...
Script     4.2.0      AzureRM.KeyVault                    {Add-AzureKeyVaultCertificate, Set-AzureKeyVaultCertificat...
Script     5.0.1      AzureRM.Network                     {Add-AzureRmApplicationGatewayAuthenticationCertificate, G...
Script     5.8.3      AzureRM.profile                     {Disable-AzureRmDataCollection, Disable-AzureRmContextAuto...
Script     6.4.3      AzureRM.Resources                   {Get-AzureRmProviderOperation, Remove-AzureRmRoleAssignmen...
Script     5.0.4      AzureRM.Storage                     {Get-AzureRmStorageAccount, Get-AzureRmStorageAccountKey, ...
Script     4.0.2      AzureRM.Tags                        {Remove-AzureRmTag, Get-AzureRmTag, New-AzureRmTag}
Script     4.0.3      AzureRM.UsageAggregates             Get-UsageAggregates
Script     5.0.1      AzureRM.Websites                    {Get-AzureRmAppServicePlan, Set-AzureRmAppServicePlan, New...

 
    Directory: C:\Program Files (x86)\Microsoft Azure Information Protection\Powershell
 
ModuleType Version    Name                            ExportedCommands
---------- -------    ----                           ----------------
Binary     1.48.204.0 AzureInformationProtection          {Clear-RMSAuthentication, Get-RMSFileStatus, Get-RMSServer...
```


## <a name="step-5-modify-host-file-for-endpoint-name-resolution"></a>Шаг 5. изменение файла узла для разрешения имени конечной точки 

Теперь вы добавите в Azure виртуальный IP-адрес, который вы определили в локальном пользовательском интерфейсе устройства, в следующее:

- файл узла на клиенте ИЛИ
- конфигурацию DNS-сервера.

> [!IMPORTANT]
> Рекомендуется изменить конфигурацию DNS-сервера для разрешения имен конечных точек.

В клиенте Windows, который используется для подключения к устройству, выполните приведенные ниже действия.

1. Запустите **Блокнот** с правами администратора, а затем откройте файл **hosts** , расположенный по адресу C:\Windows\System32\Drivers\etc.

    ![Файл hosts в проводнике](media/azure-stack-edge-j-series-connect-resource-manager/hosts-file.png)

2. Добавьте следующие записи в файл **hosts**, указав соответствующие значения для вашего устройства: 

    ```
    <Device IP> login.<appliance name>.<DNS domain>
    <Device IP> management.<appliance name>.<DNS domain>
    <Device IP> <storage name>.blob.<appliance name>.<DNS domain>
    ```

    > [!IMPORTANT]
    > Запись в файле Hosts должна точно совпадать с указанной для подключения к Azure Resource Manager на более позднем этапе. Убедитесь, что запись домена DNS находится в нижнем регистре.

    Вы сохранили IP-адрес устройства из локального веб-интерфейса на предыдущем шаге.

    Имя \<appliance name\> входа.\<DNS domain\> Entry — это конечная точка для службы маркеров безопасности (STS). STS отвечает за создание, проверку, продление и отмену маркеров безопасности. Служба маркеров безопасности используется для создания маркера доступа и маркера обновления, который используется для непрерывного взаимодействия между устройством и клиентом.

3. Для ссылки используйте следующее изображение. Сохраните файл **hosts**.

    ![Файл hosts в Блокноте](media/azure-stack-edge-j-series-connect-resource-manager/hosts-file-notepad.png)

## <a name="step-6-verify-endpoint-name-resolution-on-the-client"></a>Шаг 6. Проверка разрешения имени конечной точки на клиенте

Проверьте, разрешено ли имя конечной точки на клиенте, который вы используете для подключения к виртуальному IP-адресу, который используется в Azure.

1. Чтобы проверить, разрешено ли имя конечной точки, можно использовать служебную программу командной строки ping.exe. При наличии IP-адреса команда ping возвратит имя узла TCP/IP для \' перетрассировки компьютера.

    Добавьте `-a` параметр в командную строку, как показано в примере ниже. Если имя узла является возвращаемым, оно также будет возвращать потенциально ценную информацию в ответе.

    ![Проверка связи в командной строке](media/azure-stack-edge-j-series-connect-resource-manager/ping-command-prompt.png)



## <a name="step-7-set-azure-resource-manager-environment"></a>Шаг 7. Настройка среды Azure Resource Manager

Задайте среду Azure Resource Manager и убедитесь, что устройство может обмениваться данными с помощью Azure Resource Manager работает нормально. Для этой проверки выполните следующие действия.


1. Используйте командлет `Add-AzureRmEnvironment` для проверки того, что передача данных через Azure Resource Manager работает правильно и вызовы API проходят через порт, выделенный для Azure Resource Manager — 443.

    Командлет `Add-AzureRmEnvironment` добавляет конечные точки и метаданные, чтобы разрешить командлетам Azure Resource Manager подключаться к новому экземпляру Azure Resource Manager. 


    > [!IMPORTANT]
    > URL-адрес конечной точки Azure Resource Manager, который вы задаете в следующем командлете, учитывает регистр. Убедитесь, что URL-адрес конечной точки находится в нижнем регистре и соответствует тому, что указано в файле hosts. Если регистр не совпадает, вы увидите ошибку.

    ```powershell
    Add-AzureRmEnvironment -Name <Environment Name> -ARMEndpoint "https://management.<appliance name>.<DNSDomain>/"
    ```

    Результат выполнения команды показан ниже:
    
    ```powershell
    PS C:\windows\system32> Add-AzureRmEnvironment -Name AzDBE -ARMEndpoint https://management.dbe-n6hugc2ra.microsoftdatabox.com/
    
    Name  Resource Manager Url                    ActiveDirectory Authority
    ----  --------------------                   -------------------------
    AzDBE https://management.dbe-n6hugc2ra.microsoftdatabox.com https://login.dbe-n6hugc2ra.microsoftdatabox.com/adfs/
    ```

2. Задайте Azure Stack Edge Pro в качестве среды и порт 443, который будет использоваться для вызовов Azure Resource Manager. Среду можно определить двумя способами:

    - Настройте среду. Введите следующую команду:

    ```powershell
    Set-AzureRMEnvironment -Name <Environment Name>
    ```
    
    Дополнительные сведения см. в подразделе [Set-AzureRMEnvironment](/powershell/module/azurerm.profile/set-azurermenvironment?view=azurermps-6.13.0&preserve-view=true).

    - Определите встроенную среду для каждого выполняемого командлета. Это гарантирует, что все вызовы API будут проходить через правильную среду. По умолчанию вызовы будут проходить через общедоступную версию Azure, но вы хотите, чтобы они перейдут в среду, настроенную для Azure Stack устройства Pro.

    - См. Дополнительные сведения о [том, как переключить среды AzureRM](#switch-environments).

2. Вызовите API локальных устройств для проверки подлинности подключений к Azure Resource Manager. 

    1. Эти учетные данные предназначены для учетной записи локального компьютера и используются только для доступа к API.

    2. Можно подключиться с помощью `login-AzureRMAccount` команды или через `Connect-AzureRMAccount` команду. 

        1. Чтобы войти, введите следующую команду. Идентификатор клиента в этом экземпляре жестко закодирован — c0257de7-538f-415c-993a-1b87a031879d. Используйте следующие имя пользователя и пароль.

            - **Имя пользователя**  -  *Еджеармусер*

            - **Пароль**  -  [Задайте пароль для Azure Resource Manager](azure-stack-edge-j-series-set-azure-resource-manager-password.md) и используйте этот пароль для входа. 

            ```powershell
            PS C:\windows\system32> $pass = ConvertTo-SecureString "<Your password>" -AsPlainText -Force;
            PS C:\windows\system32> $cred = New-Object System.Management.Automation.PSCredential("EdgeArmUser", $pass)
            PS C:\windows\system32> Connect-AzureRmAccount -EnvironmentName AzDBE -TenantId c0257de7-538f-415c-993a-1b87a031879d -credential $cred
            
            Account       SubscriptionName   TenantId            Environment
            -------       ----------------   --------            -----------
            EdgeArmUser@localhost Default Provider Subscription c0257de7-538f-415c-993a-1b87a031879d AzDBE
            
            PS C:\windows\system32>
            ```
                   
        
            Альтернативным способом входа в систему является использование `login-AzureRmAccount` командлета. 
            
            `login-AzureRMAccount -EnvironmentName <Environment Name>` -TenantId c0257de7-538f-415c-993a-1b87a031879d 

            Ниже приведен пример выходных данных команды. 
         
            ```powershell
            PS C:\Users\Administrator> login-AzureRMAccount -EnvironmentName AzDBE -TenantId c0257de7-538f-415c-993a-1b87a031879d
            
            Account         SubscriptionName  TenantId              Environment
            -------         ----------------  --------              -------
            EdgeArmUser@localhost Default Provider Subscription c0257de7-538f-415c-993a-1b87a031879d AzDBE
            PS C:\Users\Administrator>
            ```



> [!IMPORTANT]
> Срок действия подключения к Azure Resource Manager истекает каждые 1,5 часов или при перезапуске устройства Azure Stack с пограничным Pro. В этом случае все выполняемые командлеты будут возвращать сообщения об ошибках, которые больше не подключены к Azure. Вам потребуется снова войти в систему.

## <a name="switch-environments"></a>Переключение сред

Выполните `Disconnect-AzureRmAccount` команду, чтобы переключиться на другой `AzureRmEnvironment` . 

При использовании `Set-AzureRmEnvironment` и `Login-AzureRmAccount` без использования `Disconnect-AzureRmAccount` Среда фактически не переключается.  

В следующих примерах показано, как переключаться между двумя средами `AzDBE1` и `AzDBE2` .

Сначала перечислите все существующие среды на клиенте.


```azurepowershell
PS C:\WINDOWS\system32> Get-AzureRmEnvironment
Name    Resource Manager Url     ActiveDirectory Authority
----    --------------------      -------------------------
AzureChinaCloud   https://management.chinacloudapi.cn/                 https://login.chinacloudapi.cn/
AzureCloud        https://management.azure.com/                        https://login.microsoftonline.com/
AzureGermanCloud  https://management.microsoftazure.de/                https://login.microsoftonline.de/
AzDBE1            https://management.HVTG1T2-Test.microsoftdatabox.com https://login.hvtg1t2-test.microsoftdatabox.com/adfs/
AzureUSGovernment https://management.usgovcloudapi.net/                https://login.microsoftonline.us/
AzDBE2            https://management.CVV4PX2-Test.microsoftdatabox.com https://login.cvv4px2-test.microsoftdatabox.com/adfs/
```

Затем Узнайте, к какой среде вы подключены в данный момент с помощью Azure Resource Manager.

```azurepowershell
PS C:\WINDOWS\system32> Get-AzureRmContext |fl *

Name               : Default Provider Subscription (A4257FDE-B946-4E01-ADE7-674760B8D1A3) - EdgeArmUser@localhost
Account            : EdgeArmUser@localhost
Environment        : AzDBE2
Subscription       : a4257fde-b946-4e01-ade7-674760b8d1a3
Tenant             : c0257de7-538f-415c-993a-1b87a031879d
TokenCache         : Microsoft.Azure.Commands.Common.Authentication.ProtectedFileTokenCache
VersionProfile     :
ExtendedProperties : {}
```

Теперь необходимо отключиться от текущей среды перед переключением в другую среду.


```azurepowershell
PS C:\WINDOWS\system32> Disconnect-AzureRmAccount

Id                    : EdgeArmUser@localhost
Type                  : User
Tenants               : {c0257de7-538f-415c-993a-1b87a031879d}
AccessToken           :
Credential            :
TenantMap             : {}
CertificateThumbprint :
ExtendedProperties    : {[Subscriptions, A4257FDE-B946-4E01-ADE7-674760B8D1A3], [Tenants, c0257de7-538f-415c-993a-1b87a031879d]}
```

Войдите в другую среду. Пример выходных данных показан ниже.

```azurepowershell
PS C:\WINDOWS\system32> Login-AzureRmAccount -Environment "AzDBE1" -TenantId $ArmTenantId

Account     SubscriptionName   TenantId        Environment
-------     ----------------   --------        -----------
EdgeArmUser@localhost Default Provider Subscription c0257de7-538f-415c-993a-1b87a031879d AzDBE1
```

Запустите этот командлет, чтобы проверить, к какой среде вы подключены.

```azurepowershell
PS C:\WINDOWS\system32> Get-AzureRmContext |fl *

Name               : Default Provider Subscription (A4257FDE-B946-4E01-ADE7-674760B8D1A3) - EdgeArmUser@localhost
Account            : EdgeArmUser@localhost
Environment        : AzDBE1
Subscription       : a4257fde-b946-4e01-ade7-674760b8d1a3
Tenant             : c0257de7-538f-415c-993a-1b87a031879d
TokenCache         : Microsoft.Azure.Commands.Common.Authentication.ProtectedFileTokenCache
VersionProfile     :
ExtendedProperties : {}
```
Теперь вы перешли на предполагаемую среду.

## <a name="next-steps"></a>Дальнейшие действия

[Развертывание виртуальных машин на устройстве Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-virtual-machine-powershell.md)
