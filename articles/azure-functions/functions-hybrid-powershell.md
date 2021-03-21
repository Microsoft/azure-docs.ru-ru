---
title: Управление удаленными локальными ресурсами с помощью функций PowerShell
description: Узнайте, как настроить гибридные подключения в Azure Relay для подключения приложения-функции PowerShell к локальным ресурсам, которые затем можно использовать для удаленного управления локальным ресурсом.
author: eamono
ms.topic: conceptual
ms.date: 04/26/2020
ms.author: eamono
ms.openlocfilehash: 5e01ffd8e17fda9113c7ec0fdb2c7f436b39c810
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97936912"
---
# <a name="managing-hybrid-environments-with-powershell-in-azure-functions-and-app-service-hybrid-connections"></a>Управление гибридными средами с помощью PowerShell в функциях Azure и гибридные подключения службы приложений

Функция гибридные подключения службы приложений Azure обеспечивает доступ к ресурсам в других сетях. Дополнительные сведения об этой возможности можно узнать в документации по [гибридные подключения](../app-service/app-service-hybrid-connections.md) . В этой статье описывается, как использовать эту возможность для запуска функций PowerShell, предназначенных для локального сервера. Затем этот сервер можно использовать для управления всеми ресурсами в локальной среде из функции Azure PowerShell.


## <a name="configure-an-on-premises-server-for-powershell-remoting"></a>Настройка локального сервера для удаленного взаимодействия PowerShell

Следующий скрипт включает удаленное взаимодействие PowerShell и создает новое правило брандмауэра и прослушиватель HTTPS для WinRM. В целях тестирования используется самозаверяющий сертификат. В рабочей среде рекомендуется использовать подписанный сертификат.

```powershell
# For configuration of WinRM, see
# https://docs.microsoft.com/windows/win32/winrm/installation-and-configuration-for-windows-remote-management.

# Enable PowerShell remoting.
Enable-PSRemoting -Force

# Create firewall rule for WinRM. The default HTTPS port is 5986.
New-NetFirewallRule -Name "WinRM HTTPS" `
                    -DisplayName "WinRM HTTPS" `
                    -Enabled True `
                    -Profile "Any" `
                    -Action "Allow" `
                    -Direction "Inbound" `
                    -LocalPort 5986 `
                    -Protocol "TCP"

# Create new self-signed-certificate to be used by WinRM.
$Thumbprint = (New-SelfSignedCertificate -DnsName $env:COMPUTERNAME  -CertStoreLocation Cert:\LocalMachine\My).Thumbprint

# Create WinRM HTTPS listener.
$Cmd = "winrm create winrm/config/Listener?Address=*+Transport=HTTPS @{Hostname=""$env:COMPUTERNAME ""; CertificateThumbprint=""$Thumbprint""}"
cmd.exe /C $Cmd
```

## <a name="create-a-powershell-function-app-in-the-portal"></a>Создание приложения-функции PowerShell на портале

Функция гибридные подключения службы приложений доступна только в ценовых планах "базовый", "Стандартный" и "изолированный". При создании приложения функции с помощью PowerShell создайте или выберите один из этих планов.

1. На **домашней странице** или в меню портала Azure выберите **Создать ресурс**.

1. На странице **Создать** щелкните **Вычислительные ресурсы** > **Приложение-функция**.

1. На странице **Основные сведения** используйте параметры приложения-функции как указано в таблице ниже.

    | Параметр      | Рекомендуемое значение  | Описание |
    | ------------ | ---------------- | ----------- |
    | **Подписка** | Ваша подписка | Подписка, в которой создано приложение-функция. |
    | **[Группа ресурсов](../azure-resource-manager/management/overview.md)** |  *myResourceGroup* | Имя новой группы ресурсов, в которой создается приложение-функция. |
    | **Имя приложения-функции** | Глобально уникальное имя | Имя, которое идентифицирует ваше новое приложение-функцию. Допустимые символы: `a-z` (без учета регистра), `0-9`и `-`.  |
    |**Опубликовать**| Код | Параметр для публикации файлов кода или контейнера Docker. |
    | **Стек среды выполнения** | Предпочитаемый язык | Выберите PowerShell Core. |
    |**Версия**| номер версии; | Выберите версию установленной среды выполнения.  |
    |**Регион**| Предпочтительный регион | Выберите ближайший [регион](https://azure.microsoft.com/regions/) или регион рядом с другими службами, к которому получают доступ ваши функции. |

    :::image type="content" source="./media/functions-hybrid-powershell/function-app-create-basics.png" alt-text="Создание приложения-функции — основы." border="true":::

1. Выберите **Далее: Размещение**. Введите следующие параметры на странице **Размещение**.

    | Параметр      | Рекомендуемое значение  | Описание |
    | ------------ | ---------------- | ----------- |
    | **[Учетная запись хранения](../storage/common/storage-account-create.md)** |  Глобально уникальное имя |  Создайте учетную запись хранения для использования приложением-функцией. Имя учетной записи хранения должно содержать от 3 до 24 символов и состоять только из цифр и строчных букв. Можно также использовать существующую учетную запись при условии, что она соответствует [требованиям учетной записи хранилища](../azure-functions/storage-considerations.md#storage-account-requirements). |
    |**Операционная система**| Предпочтительная операционная система | Операционная система предварительно выбирается с учетом выбранного стека среды выполнения, но при необходимости ее можно изменить. |
    | **[Тип плана](../azure-functions/functions-scale.md)** | **План службы приложений** | Выберите **план службы приложений**. Когда выполняется план службы приложений необходимо управлять [масштабированием приложения-функции](../azure-functions/functions-scale.md).  |

    :::image type="content" source="./media/functions-hybrid-powershell/function-app-create-hosting.png" alt-text="Создание приложения-функции размещение." border="true":::

1. Выберите **Далее: Мониторинг**. Введите следующие параметры на странице **Мониторинг**.

    | Параметр      | Рекомендуемое значение  | Описание |
    | ------------ | ---------------- | ----------- |
    | **[Application Insights](../azure-functions/functions-monitoring.md)** | По умолчанию | Создает ресурс Application Insights с таким же *именем приложения* в ближайшем поддерживаемом регионе. Развернув этот параметр или выбрав **Создать новый**, можно изменить имя Application Insights или выбрать другое расположение в [географическом регионе Azure](https://azure.microsoft.com/global-infrastructure/geographies/), где будут хранится данные. |

    :::image type="content" source="./media/functions-hybrid-powershell/function-app-create-monitoring.png" alt-text="Создайте функцию мониторинга приложений." border="true":::

1. Выберите **Просмотр и создание** , чтобы просмотреть выбранные параметры конфигурации приложения.

1. Проверьте параметры на странице **Просмотр и создание**, после чего выберите **Создать** для подготовки и развертывания приложения-функции.

1. Выберите значок **Уведомления** в правом верхнем углу портала. Вы должны увидеть сообщение **Развертывание выполнено**.

1. Выберите **Перейти к ресурсу** для просмотра нового приложения-функции. Можно также установить флажок **Закрепить на панели мониторинга**. Это упростит возвращение к этому ресурсу приложения-функции из панели мониторинга.

## <a name="create-a-hybrid-connection-for-the-function-app"></a>Создание гибридного подключения для приложения-функции

Гибридные подключения настраиваются из раздела сети приложения функции:

1. В разделе **Параметры** только что созданного приложения функции выберите **сеть**. 
1. Выберите **Настройка конечных точек гибридных подключений**.
   
    :::image type="content" source="./media/functions-hybrid-powershell/configure-hybrid-connection-endpoint.png" alt-text="Настройте конечные точки гибридного подключения." border="true":::

1. Выберите **Добавить гибридное подключение**.
   
    :::image type="content" source="./media/functions-hybrid-powershell/hybrid-connection-overview.png" alt-text="Добавьте гибридное подключение." border="true":::

1. Введите сведения о гибридном подключении, как показано сразу после следующего снимка экрана. Можно сделать так, чтобы параметр **узла конечной точки** совпадал с именем узла локального сервера, чтобы было проще запомнить сервер при выполнении удаленных команд. Порт совпадает с портом службы удаленного управления Windows по умолчанию, который был определен на сервере ранее.
  
      :::image type="content" source="./media/functions-hybrid-powershell/add-hybrid-connection.png" alt-text="Добавление гибридного подключения." border="true":::

    | Параметр      | Рекомендуемое значение  |
    | ------------ | ---------------- |
    | **Имя гибридного подключения** | контосохибридонпремисессервер |
    | **Узел конечной точки** | finance1 |
    | **Порт конечной точки** | 5986 |
    | **Пространство имен servicebus** | Создать |
    | **Расположение** | Выберите доступное расположение |
    | **имя**; | контосоповершеллхибрид | 

1. Нажмите кнопку **ОК** , чтобы создать гибридное подключение.

## <a name="download-and-install-the-hybrid-connection"></a>Скачивание и установка гибридного подключения

1. Выберите **загрузить диспетчер подключений** , чтобы сохранить *MSI* файл на локальном компьютере.

    :::image type="content" source="./media/functions-hybrid-powershell/download-hybrid-connection-installer.png" alt-text="Скачайте установщик." border="true":::

1. Скопируйте *MSI* -файл с локального компьютера на локальный сервер.
1. Запустите установщик Диспетчер гибридных подключений, чтобы установить службу на локальном сервере.

    :::image type="content" source="./media/functions-hybrid-powershell/hybrid-installation.png" alt-text="Установите гибридное подключение." border="true":::

1. На портале откройте гибридное подключение, а затем скопируйте строку подключения шлюза в буфер обмена.

    :::image type="content" source="./media/functions-hybrid-powershell/copy-hybrid-connection.png" alt-text="Скопируйте строку гибридного подключения." border="true":::

1. Откройте пользовательский интерфейс Диспетчер гибридных подключений на локальном сервере.

    :::image type="content" source="./media/functions-hybrid-powershell/hybrid-connection-ui.png" alt-text="Откройте пользовательский интерфейс гибридного подключения." border="true":::

1. Нажмите клавишу **Ввод вручную** и вставьте строку подключения из буфера обмена.

    :::image type="content" source="./media/functions-hybrid-powershell/enter-manual-connection.png" alt-text="Вставьте гибридное подключение." border="true":::

1. Перезапустите Диспетчер гибридных подключений из PowerShell, если он не отображается как подключенный.
    ```powershell
    Restart-Service HybridConnectionManager
    ```

## <a name="create-an-app-setting-for-the-password-of-an-administrator-account"></a>Создание параметра приложения для пароля учетной записи администратора

1. В разделе **Параметры** приложения функции выберите **Конфигурация**. 
1. Выберите **+ создать параметр приложения**.

    :::image type="content" source="./media/functions-hybrid-powershell/select-configuration.png" alt-text="Настройте пароль для учетной записи администратора." border="true":::

1. Присвойте параметру имя **контосаусерпассворд** и введите пароль. Щелкните **ОК**.
1. Выберите **сохранить** , чтобы сохранить пароль в приложении функции.

    :::image type="content" source="./media/functions-hybrid-powershell/save-administrator-password.png" alt-text="Сохраните пароль для учетной записи администратора." border="true":::

## <a name="create-a-function-http-trigger"></a>Создание триггера HTTP функции

1. В приложении функции выберите **функции**, а затем щелкните **+ Добавить**.

    :::image type="content" source="./media/functions-hybrid-powershell/create-http-trigger-function.png" alt-text="Создайте новый триггер HTTP." border="true":::

1. Выберите шаблон **триггера HTTP** .

    :::image type="content" source="./media/functions-hybrid-powershell/select-http-trigger-template.png" alt-text="Выберите шаблон триггера HTTP." border="true":::

1. Назовите новую функцию и выберите **создать функцию**.

    :::image type="content" source="./media/functions-hybrid-powershell/create-new-http-function.png" alt-text="Назовите и создайте новую функцию триггера HTTP." border="true":::

## <a name="test-the-function"></a>Проверка функции

1. В новой функции выберите **код + тест**. Замените код PowerShell из шаблона следующим кодом:

    ```powershell
    # Input bindings are passed in via param block.
    param($Request, $TriggerMetadata)
    
    # Write to the Azure Functions log stream.
    Write-Output "PowerShell HTTP trigger function processed a request."
    
    # Note that ContosoUserPassword is a function app setting, so I can access it as $env:ContosoUserPassword.
    $UserName = "ContosoUser"
    $securedPassword = ConvertTo-SecureString  $Env:ContosoUserPassword -AsPlainText -Force
    $Credential = [System.management.automation.pscredential]::new($UserName, $SecuredPassword)
    
    # This is the name of the hybrid connection Endpoint.
    $HybridEndpoint = "finance1"
    
    $Script = {
        Param(
            [Parameter(Mandatory=$True)]
            [String] $Service
        )
        Get-Service $Service
    }
    
    Write-Output "Scenario 1: Running command via Invoke-Command"
    Invoke-Command -ComputerName $HybridEndpoint `
                   -Credential $Credential `
                   -Port 5986 `
                   -UseSSL `
                   -ScriptBlock $Script `
                   -ArgumentList "*" `
                   -SessionOption (New-PSSessionOption -SkipCACheck)
    ```

1. Щелкните **Сохранить**.

    :::image type="content" source="./media/functions-hybrid-powershell/save-http-function.png" alt-text="Измените код PowerShell и сохраните функцию триггера HTTP." border="true":::

 1. Выберите **тест** и нажмите кнопку **выполнить** , чтобы проверить функцию. Проверьте журналы, чтобы убедиться, что проверка прошла успешно.

     :::image type="content" source="./media/functions-hybrid-powershell/test-function-hybrid.png" alt-text="Проверка функции триггера HTTP." border="true":::

## <a name="managing-other-systems-on-premises"></a>Управление другими системами в локальной среде

Подключенный локальный сервер можно использовать для подключения к другим серверам и системам управления в локальной среде. Это позволяет управлять операциями центра обработки данных из Azure с помощью функций PowerShell. Следующий скрипт регистрирует сеанс конфигурации PowerShell, который выполняется с указанными учетными данными. Эти учетные данные должны быть предоставлены администратору на удаленных серверах. Затем эту конфигурацию можно использовать для доступа к другим конечным точкам на локальном сервере или в центре обработки данных.

```powershell
# Input bindings are passed in via param block.
param($Request, $TriggerMetadata)

# Write to the Azure Functions log stream.
Write-Host "PowerShell HTTP trigger function processed a request."

# Note that ContosoUserPassword is a function app setting, so I can access it as $env:ContosoUserPassword.
$UserName = "ContosoUser"
$SecuredPassword = ConvertTo-SecureString  $Env:ContosoUserPassword -AsPlainText -Force
$Credential = [System.management.automation.pscredential]::new($UserName, $SecuredPassword)

# This is the name of the hybrid connection Endpoint.
$HybridEndpoint = "finance1"

# The remote server that will be connected to run remote PowerShell commands on
$RemoteServer = "finance2".

Write-Output "Use hybrid connection server as a jump box to connect to a remote machine"

# We are registering an endpoint that runs under credentials ($Credential) that has access to the remote server.
$SessionName = "HybridSession"
$ScriptCommand = {
    param (
        [Parameter(Mandatory=$True)]
        $SessionName)

    if (-not (Get-PSSessionConfiguration -Name $SessionName -ErrorAction SilentlyContinue))
    {
        Register-PSSessionConfiguration -Name $SessionName -RunAsCredential $Using:Credential
    }
}

Write-Output "Registering session on hybrid connection jumpbox"
Invoke-Command -ComputerName $HybridEndpoint `
               -Credential $Credential `
               -Port 5986 `
               -UseSSL `
               -ScriptBlock $ScriptCommand `
               -ArgumentList $SessionName `
               -SessionOption (New-PSSessionOption -SkipCACheck)

# Script to run on the jump box to run against the second machine.
$RemoteScriptCommand = {
    param (
        [Parameter(Mandatory=$True)]
        $ComputerName)
        # Write out the hostname of the hybrid connection server.
        hostname
        # Write out the hostname of the remote server.
        Invoke-Command -ComputerName $ComputerName -Credential $Using:Credential -ScriptBlock {hostname} `
                        -UseSSL -Port 5986 -SessionOption (New-PSSessionOption -SkipCACheck)
}

Write-Output "Running command against remote machine via jumpbox by connecting to the PowerShell configuration session"
Invoke-Command -ComputerName $HybridEndpoint `
               -Credential $Credential `
               -Port 5986 `
               -UseSSL `
               -ScriptBlock $RemoteScriptCommand `
               -ArgumentList $RemoteServer `
               -SessionOption (New-PSSessionOption -SkipCACheck) `
               -ConfigurationName $SessionName
```

Замените следующие переменные в этом скрипте соответствующими значениями из вашей среды:
* $HybridEndpoint
* $RemoteServer

В двух предыдущих сценариях вы можете подключать локальные среды и управлять ими с помощью PowerShell в функциях Azure и гибридные подключения. Мы рекомендуем вам узнать больше о [гибридные подключения](../app-service/app-service-hybrid-connections.md) и [PowerShell в функциях](./functions-reference-powershell.md).

[Виртуальные сети](./functions-create-vnet.md) Azure также можно использовать для подключения к локальной среде с помощью функций Azure.

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"] 
> [Дополнительные сведения о работе с функциями PowerShell](functions-reference-powershell.md)
