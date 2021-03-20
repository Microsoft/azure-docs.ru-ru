---
title: Устранение проблем, связанных с приложениями, которые не поддерживают протокол TLS 1.2 | Документация Майкрософт
description: Устранение проблем, связанных с приложениями, которые не поддерживают протокол TLS 1.2
services: cloud-services
documentationcenter: ''
author: tanmaygore
tags: top-support-issue
ms.assetid: ''
ms.service: cloud-services
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: ''
ms.date: 03/16/2020
ms.author: tagore
ms.openlocfilehash: cf7746cc55e81593a1788608cced1253f295a5c4
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101738384"
---
# <a name="troubleshooting-applications-that-dont-support-tls-12"></a>Устранение неполадок в приложениях, которые не поддерживают протокол TLS 1.2

> [!IMPORTANT]
> [Облачные службы Azure (Расширенная поддержка)](../cloud-services-extended-support/overview.md) — это новая модель развертывания на основе Azure Resource Manager для продукта облачных служб Azure.После этого изменения облачные службы Azure, работающие в модели развертывания на основе Service Manager Azure, были переименованы как облачные службы (классические), и все новые развертывания должны использовать [облачные службы (Расширенная поддержка)](../cloud-services-extended-support/overview.md).

В этой статье содержатся сведения о включении старых протоколов TLS (TLS 1.0 и 1.1), а также о применении устаревших наборов шифров для поддержки дополнительных протоколов в веб-ролях и рабочих ролях облачной службы Windows Server 2019. 

Мы понимаем, что, пока ведется работа над выводом протоколов TLS 1.0 и TLS 1.1 из эксплуатации, клиентам может потребоваться поддержка старых протоколов и наборов шифров до того момента, когда они смогут перейти на их новые версии.  Хотя повторно включать эти устаревшие значения не рекомендуется, мы предоставляем специальные инструкции, чтобы помочь клиентам. Прежде чем вносить изменения, описанные в этой статье, следует оценить риск возникновения проблем. 

> [!NOTE]
> В выпуске семейства 6 гостевой ОС протокол TLS 1.2 принудительно применяется путем явного отключения протоколов TLS 1.0 и 1.1 и определения конкретного набора шифров. Дополнительные сведения о семействах гостевых ОС см. в статье о [выпуске гостевой ОС](./cloud-services-guestos-update-matrix.md#family-6-releases).


## <a name="dropping-support-for-tls-10-tls-11-and-older-cipher-suites"></a>Отмена поддержки протоколов TLS 1.0, TLS 1.1 и старых наборов шифров 
В соответствии с обязательством по обеспечению наилучшего в своем классе шифрования корпорация Майкрософт ведет долгосрочную кампанию по отказу от устаревших протоколов TLS 1.0 и 1.1 с июня 2017 года.   С момента этого первоначального заявления корпорация Майкрософт сообщила о своем намерении отключить протокол TLS 1.0 и 1.1 по умолчанию в поддерживаемых версиях Microsoft Edge и Internet Explorer 11 в первом полугодии 2020 года.  Аналогичные объявления от Apple, Google и Mozilla указывают направление, в котором движется отрасль.   

Дополнительные сведения см. в статье [Подготовка к TLS 1.2 в Microsoft Azure](https://azure.microsoft.com/updates/azuretls12/)

## <a name="tls-configuration"></a>Конфигурация протокола TLS  
Образ облачного сервера Windows Server 2019, настроенный для использования TLS 1.0 и TLS 1.1, отключен на уровне реестра. Это означает, что приложения, развернутые в этой версии Windows и использующие стек Windows для согласования TLS, запрещают взаимодействие по протоколам TLS 1.0 и TLS 1.1.   

Сервер также поставляется с ограниченным набором шифров: 

```
    TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256 
    TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384 
    TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256 
    TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384 
    TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256 
    TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384 
    TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256 
    TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384 
```

## <a name="step-1-create-the-powershell-script-to-enable-tls-10-and-tls-11"></a>Шаг 1. Создание сценария PowerShell для включения протоколов TLS 1.0 и TLS 1.1 

Используйте следующий код в качестве примера создания сценария, включающего старые протоколы и наборы шифров. В этой документации сценарию буде задано следующее имя: **TLSsettings.ps1**. Сохраните этот сценарий на локальном рабочем столе для быстрого доступа к нему в последующих шагах. 


```Powershell
# You can use the -SetCipherOrder (or -sco) option to also set the TLS cipher 
# suite order. Change the cipherorder variable below to the order you want to set on the 
# server. Setting this requires a reboot to take effect.

Param(
 [parameter(Mandatory=$false)]
 [alias("sco")]
 [switch]$SetCipherOrder)

 Function DisableRC4 {
   param ( $restart)
  $subkeys = Get-Item -Path "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL"
  $ciphers = $subkeys.OpenSubKey("Ciphers", $true)

  if($ciphers.SubKeyCount -eq 0) {
    $k1 = $ciphers.CreateSubKey("RC4 128/128")
    $k1.SetValue("Enabled", 0, [Microsoft.Win32.RegistryValueKind]::DWord)
    $restart = $true
    $k2 = $ciphers.CreateSubKey("RC4 64/128")
    $k2.SetValue("Enabled", 0, [Microsoft.Win32.RegistryValueKind]::DWord)
    $k3 = $ciphers.CreateSubKey("RC4 56/128")
    $k3.SetValue("Enabled", 0, [Microsoft.Win32.RegistryValueKind]::DWord)
    $k4 = $ciphers.CreateSubKey("RC4 40/128")
    $k4.SetValue("Enabled", 0, [Microsoft.Win32.RegistryValueKind]::DWord)
  }

  $restart

}

Function Set-CryptoSetting {
  param (
    $keyindex,
    $value,
    $valuedata,
    $valuetype,
    $restart
  )

  # Check for existence of registry key, and create if it does not exist
  If (!(Test-Path -Path $regkeys[$keyindex])) {
    New-Item $regkeys[$keyindex] | Out-Null
  }

  # Get data of registry value, or null if it does not exist
  $val = (Get-ItemProperty -Path $regkeys[$keyindex] -Name $value -ErrorAction SilentlyContinue).$value

  If ($null -eq $val) {
    # Value does not exist - create and set to desired value
    New-ItemProperty -Path $regkeys[$keyindex] -Name $value -Value $valuedata -PropertyType $valuetype | Out-Null
    $restart = $True
    Write-Host "Configuring $regkeys[$keyindex]...."

  } Else {

    # Value does exist - if not equal to desired value, change it
    If ($val -ne $valuedata) {
      Set-ItemProperty -Path $regkeys[$keyindex] -Name $value -Value $valuedata
      $restart = $True
      Write-Host "Configuring $regkeys[$keyindex]..."
    }
  }

  $restart

}

$regkeys = @(
"HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.0",
"HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.0\Client",
"HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.0\Server", #2
"HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.1",
"HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.1\Client", #4
"HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.1\Server",
"HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2",        #6
"HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client",
"HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server", #8
"HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\SSL 2.0",
"HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\SSL 2.0\Client", #10
"HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\SSL 2.0\Server",
"HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\SSL 3.0",        #12
"HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\SSL 3.0\Client",
"HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\SSL 3.0\Server", #14
"HKLM:\SOFTWARE\Policies\Microsoft\Cryptography\Configuration\SSL\00010002"
)

Function Set-Windows10PlusCurveOrder {
    param ( $reboot)
    $desiredOrder = "NistP384;NistP256".Split(";")
    If ([Environment]::OSVersion.Version.Major -ge 10) {
        If (!(Test-Path -Path $regkeys[15])) {
            New-Item $regkeys[15] | Out-Null
            $reboot = $True
        }

        $val = (Get-Item -Path $regkeys[15] -ErrorAction SilentlyContinue).GetValue("EccCurves", $null)

        if( $null -eq $val) {
            New-ItemProperty -Path $regkeys[15] -Name EccCurves -Value $desiredOrder -PropertyType MultiString | Out-Null
            $reboot = $True

        } else {

            if ([System.String]::Join(';', $val) -ne [System.String]::Join(';', $desiredOrder)) {
                Write-Host "The original curve order ", `n, $val, `n, "needs to be updated to ", $desiredOrder
                Set-ItemProperty -Path $regkeys[15] -Name EccCurves -Value $desiredOrder
                $reboot = $True
            }
        }
    }

    $reboot

}

If ([Environment]::OSVersion.Version.Major -lt 10) {
  # This is for Windows before 10 
  Write-Host "Configuring Windows before 10..."
  $cipherorder =  "TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384_P384,TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256_P256,"
  $cipherorder += "TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384_P384,TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256_P256,"
  $cipherorder += "TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384_P256,TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256_P256,"
  $cipherorder += "TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA_P256,TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA_P256,"
  $cipherorder += "TLS_RSA_WITH_AES_256_GCM_SHA384,TLS_RSA_WITH_AES_128_GCM_SHA256,"
  $cipherorder += "TLS_RSA_WITH_AES_256_CBC_SHA256,TLS_RSA_WITH_AES_128_CBC_SHA256,"
  $cipherorder += "TLS_RSA_WITH_AES_256_CBC_SHA,TLS_RSA_WITH_AES_128_CBC_SHA"

} Else {

 # this is for windows 10 or above
 Write-Host "Configuring Windows 10+..."
 $cipherorder = "TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,"
 $cipherorder += "TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,"
 $cipherorder += "TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384,TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256,"
 $cipherorder += "TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384,TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256,"
 $cipherorder += "TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA,TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA,"
 $cipherorder += "TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA,TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA,"
 $cipherorder += "TLS_RSA_WITH_AES_256_GCM_SHA384,TLS_RSA_WITH_AES_128_GCM_SHA256,"
 $cipherorder += "TLS_RSA_WITH_AES_256_CBC_SHA256,TLS_RSA_WITH_AES_128_CBC_SHA256,"
 $cipherorder += "TLS_RSA_WITH_AES_256_CBC_SHA,TLS_RSA_WITH_AES_128_CBC_SHA"
}

# If any settings are changed, this will change to $True and the server will reboot
$reboot = $False

# Check for existence of registry keys (SSL 2.0, SSL 3.0, TLS 1.0, TLS 1.1, TLS 1.2), and create if they do not exist
For ($i = 0; $i -le 14; $i = $i + 1) {
  If (!(Test-Path -Path $regkeys[$i])) {
    New-Item $regkeys[$i] | Out-Null
  }
}

# Ensure SSL 2.0 disabled for client/server
$reboot = Set-CryptoSetting 10 DisabledByDefault 1 DWord $reboot
$reboot = Set-CryptoSetting 10 Enabled 0 DWord $reboot
$reboot = Set-CryptoSetting 11 DisabledByDefault 1 DWord $reboot
$reboot = Set-CryptoSetting 11 Enabled 0 DWord $reboot

# Ensure SSL 3.0 disabled for client/server
$reboot = Set-CryptoSetting 13 DisabledByDefault 1 DWord $reboot
$reboot = Set-CryptoSetting 13 Enabled 0 DWord $reboot
$reboot = Set-CryptoSetting 14 DisabledByDefault 1 DWord $reboot
$reboot = Set-CryptoSetting 14 Enabled 0 DWord $reboot

# Ensure TLS 1.0 enabled for client/server
$reboot = Set-CryptoSetting 1 DisabledByDefault 0 DWord $reboot
$reboot = Set-CryptoSetting 1 Enabled 1 DWord $reboot
$reboot = Set-CryptoSetting 2 DisabledByDefault 0 DWord $reboot
$reboot = Set-CryptoSetting 2 Enabled 1 DWord $reboot

# Ensure TLS 1.1 enabled for client/server
$reboot = Set-CryptoSetting 4 DisabledByDefault 0 DWord $reboot
$reboot = Set-CryptoSetting 4 Enabled 1 DWord $reboot
$reboot = Set-CryptoSetting 5 DisabledByDefault 0 DWord $reboot
$reboot = Set-CryptoSetting 5 Enabled 1 DWord $reboot

# Ensure TLS 1.2 enabled for client/server
$reboot = Set-CryptoSetting 7 DisabledByDefault 0 DWord $reboot
$reboot = Set-CryptoSetting 7 Enabled 1 DWord $reboot
$reboot = Set-CryptoSetting 8 DisabledByDefault 0 DWord $reboot
$reboot = Set-CryptoSetting 8 Enabled 1 DWord $reboot

$reboot = DisableRC4($reboot)

If ($SetCipherOrder) {
      If (!(Test-Path -Path $regkeys[15])) {
        New-Item $regkeys[15] | Out-Null
        $reboot = $True
      }

      $val = (Get-Item -Path $regkeys[15] -ErrorAction SilentlyContinue).GetValue("Functions", $null)

      if ($val -ne $cipherorder)
      {
        Write-Host "The original cipher suite order needs to be updated", `n, $val
        Set-ItemProperty -Path $regkeys[15] -Name Functions -Value $cipherorder
        $reboot = $True
      }
  }

$reboot = Set-Windows10PlusCurveOrder $reboot

If ($reboot) {
  # Randomize the reboot timing since it could be run in a large cluster.
  $tick = [System.Int32]([System.DateTime]::Now.Ticks % [System.Int32]::MaxValue)
  $rand = [System.Random]::new($tick)
  $sec = $rand.Next(30, 600)
  Write-Host "Rebooting after", $sec, " second(s)..."
  Write-Host  "shutdown.exe /r /t $sec /c ""Crypto settings changed"" /f /d p:2:4"
  shutdown.exe /r /t $sec /c "Crypto settings changed" /f /d p:2:4

} Else {

  Write-Host "Nothing get updated."
}
```

## <a name="step-2-create-a-command-file"></a>Шаг 2. Создание файла команд 

Создайте CMD-файл с именем **RunTLSSettings.cmd**, используя приведенную ниже команду. Сохраните этот сценарий на локальном рабочем столе для быстрого доступа к нему в последующих шагах. 

```cmd
SET LOG_FILE="%TEMP%\StartupLog.txt"
SET EXECUTE_PS1=0

IF "%ComputeEmulatorRunning%" == "" (
       SET EXECUTE_PS1=1
)

IF "%ComputeEmulatorRunning%" == "false" (
       SET EXECUTE_PS1=1
) 

IF %EXECUTE_PS1% EQU 1 (
       echo "Invoking TLSsettings.ps1 on Azure service at %TIME% on %DATE%" >> %LOG_FILE% 2>&1       
       PowerShell -ExecutionPolicy Unrestricted %~dp0TLSsettings.ps1 -sco  >> %LOG_FILE% 2>&1
) ELSE (
       echo "Skipping TLSsettings.ps1 invocation on emulated environment" >> %LOG_FILE% 2>&1       
)    

EXIT /B %ERRORLEVEL%

```

## <a name="step-3-add-the-startup-task-to-the-roles-service-definition-csdef"></a>Шаг 3. Добавление автозагружаемой задачи в определение службы роли (csdef) 

Добавьте следующий фрагмент кода в существующий файл определения службы. 

```
    <Startup> 
        <Task executionContext="elevated" taskType="simple" commandLine="RunTLSSettings.cmd"> 
        </Task> 
    </Startup> 
```

Ниже приведен пример, демонстрирующий рабочую роль и веб-роль. 

```
<?xmlversion="1.0" encoding="utf-8"?> 
<ServiceDefinitionname="CloudServiceName" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition" schemaVersion="2015-04.2.6"> 
    <WebRolename="WebRole1" vmsize="Standard_D1_v2"> 
        <Sites> 
            <Sitename="Web"> 
                <Bindings> 
                    <Bindingname="Endpoint1" endpointName="Endpoint1"/> 
                </Bindings> 
            </Site> 
        </Sites> 
        <Startup> 
            <Task executionContext="elevated" taskType="simple" commandLine="RunTLSSettings.cmd"> 
            </Task> 
        </Startup> 
        <Endpoints> 
            <InputEndpointname="Endpoint1" protocol="http" port="80"/> 
        </Endpoints> 
    </WebRole> 
<WorkerRolename="WorkerRole1" vmsize="Standard_D1_v2"> 
    <Startup> 
        <Task executionContext="elevated" taskType="simple" commandLine="RunTLSSettings.cmd"> 
        </Task> 
    </Startup> 
</WorkerRole> 
</ServiceDefinition> 
```

## <a name="step-4-add-the-scripts-to-your-cloud-service"></a>Шаг 4. Добавление сценариев в облачную службу 

1) В Visual Studio щелкните правой кнопкой мыши веб-роль или рабочую роль.
2) Выберите **Добавить**.
3) Выберите **Существующий элемент**.
4) В проводнике перейдите на рабочий стол, где сохранены файлы **TLSsettings.ps1** и файлы **RunTLSSettings.cmd**. 
5) Выберите два файла, чтобы добавить их в проект облачных служб.

## <a name="step-5-enable-copy-to-output-directory"></a>Шаг 5. Включение параметра копирования в выходной каталог

Чтобы обеспечить отправку сценариев при каждом обновлении, переданном из Visual Studio, параметру *Копировать в выходной каталог* необходимо задать значение *Всегда копировать*.

1) В разделе веб-роли или рабочей роли щелкните правой кнопкой мыши файл RunTLSSettings.cmd.
2) Выберите пункт **Свойства**.
3) На вкладке "Свойства" задайте для параметра *Копировать в выходной каталог* значение *Всегда копировать*.
4) Повторите действия для файла **TLSsettings.ps1**.

## <a name="step-6-publish--validate"></a>Шаг 6. Публикация и проверка

После завершения указанных выше шагов опубликуйте обновление в существующую облачную службу. 

Для проверки состояния конечных точек TLS можно использовать [SSLLabs](https://www.ssllabs.com/). 

