---
title: Применение расширения системы диагностики Microsoft Azure в облачных службах (Расширенная поддержка)
description: Применение расширения системы диагностики Windows Azure для облачных служб (Расширенная поддержка)
ms.topic: how-to
ms.service: cloud-services-extended-support
author: gachandw
ms.author: gachandw
ms.reviewer: mimckitt
ms.date: 10/13/2020
ms.custom: ''
ms.openlocfilehash: 14b1661792ca5276bd6ebfa4cee1c4b46f94764d
ms.sourcegitcommit: f611b3f57027a21f7b229edf8a5b4f4c75f76331
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/22/2021
ms.locfileid: "104780452"
---
# <a name="apply-the-windows-azure-diagnostics-extension-in-cloud-services-extended-support"></a>Применение расширения системы диагностики Microsoft Azure в облачных службах (Расширенная поддержка) 
Вы можете отслеживать для облачных служб ключевые показатели производительности. Каждая роль облачной службы собирает минимальный набор данных: использование ЦП, сетевых и дисковых ресурсов. Если облачная служба имеет расширение Microsoft. Azure. Diagnostics, к которому применяется роль, эта роль может выполнять получение дополнительных точек данных. Дополнительные сведения см. в разделе [Общие сведения о расширениях](extensions.md) .

Расширение система диагностики Azure Windows можно включить для облачных служб (Расширенная поддержка) с помощью [PowerShell](deploy-powershell.md) или [шаблона ARM](deploy-template.md) .

## <a name="apply-windows-azure-diagnostics-extension-using-powershell"></a>Применение расширения система диагностики Azure Windows с помощью PowerShell

```powershell
# Create WAD extension object
$storageAccountKey = Get-AzStorageAccountKey -ResourceGroupName "ContosOrg" -Name "contosostorageaccount"
$configFilePath = "<Insert WAD public configuration file path>"
$wadExtension = New-AzCloudServiceDiagnosticsExtension -Name "WADExtension" -ResourceGroupName "ContosOrg" -CloudServiceName "ContosoCS" -StorageAccountName "contosostorageaccount" -StorageAccountKey $storageAccountKey[0].Value -DiagnosticsConfigurationPath $configFilePath -TypeHandlerVersion "1.5" -AutoUpgradeMinorVersion $true 

# Add <privateConfig> settings
$wadExtension.ProtectedSetting = "<Insert WAD Private Configuration as raw string here>"

# Get existing Cloud Service
$cloudService = Get-AzCloudService -ResourceGroup "ContosOrg" -CloudServiceName "ContosoCS"

# Add WAD extension to existing Cloud Service extension object
$cloudService.ExtensionProfile.Extension = $cloudService.ExtensionProfile.Extension + $wadExtension

# Update Cloud Service
$cloudService | Update-AzCloudService
```
Скачайте общедоступное определение схемы файла конфигурации, выполнив следующую команду PowerShell:

```powershell
(Get-AzureServiceAvailableExtension -ExtensionName 'PaaSDiagnostics' -ProviderNamespace 'Microsoft.Azure.Diagnostics').PublicConfigurationSchema | Out-File -Encoding utf8 -FilePath 'PublicWadConfig.xsd'
```
Ниже приведен пример XML-файла общедоступной конфигурации.
```
<?xml version="1.0" encoding="utf-8"?>
<PublicConfig xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">
  <WadCfg>
    <DiagnosticMonitorConfiguration overallQuotaInMB="25000">
      <PerformanceCounters scheduledTransferPeriod="PT1M">
        <PerformanceCounterConfiguration counterSpecifier="\Processor(_Total)\% Processor Time" sampleRate="PT1M" unit="percent" />
        <PerformanceCounterConfiguration counterSpecifier="\Memory\Committed Bytes" sampleRate="PT1M" unit="bytes"/>
      </PerformanceCounters>
      <EtwProviders>
        <EtwEventSourceProviderConfiguration provider="SampleEventSourceWriter" scheduledTransferPeriod="PT5M">
          <Event id="1" eventDestination="EnumsTable"/>
          <DefaultEvents eventDestination="DefaultTable" />
        </EtwEventSourceProviderConfiguration>
      </EtwProviders>
    </DiagnosticMonitorConfiguration>
  </WadCfg>
</PublicConfig>
```
Скачайте определение схемы файла закрытой конфигурации, выполнив следующую команду PowerShell:

```powershell
(Get-AzureServiceAvailableExtension -ExtensionName 'PaaSDiagnostics' -ProviderNamespace 'Microsoft.Azure.Diagnostics').PrivateConfigurationSchema | Out-File -Encoding utf8 -FilePath 'PrivateWadConfig.xsd'
```
Ниже приведен пример XML-файла закрытой конфигурации.

```
<?xml version="1.0" encoding="utf-8"?>
<PrivateConfig xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">
  <StorageAccount name="string" key="string" />
  <AzureMonitorAccount>
    <ServicePrincipalMeta>
      <PrincipalId>string</PrincipalId>
      <Secret>string</Secret>
    </ServicePrincipalMeta>
  </AzureMonitorAccount>
  <SecondaryStorageAccounts>
    <StorageAccount name="string" />
  </SecondaryStorageAccounts>
  <SecondaryEventHubs>
    <EventHub Url="string" SharedAccessKeyName="string" SharedAccessKey="string" />
  </SecondaryEventHubs>
</PrivateConfig>
```

## <a name="apply-windows-azure-diagnostics-extension-using-arm-template"></a>Применение расширения система диагностики Azure Windows с помощью шаблона ARM
```json
"extensionProfile": { 
          "extensions": [ 
            { 
              "name": "Microsoft.Insights.VMDiagnosticsSettings_WebRole1", 
              "properties": { 
                "autoUpgradeMinorVersion": true, 
                "publisher": "Microsoft.Azure.Diagnostics", 
                "type": "PaaSDiagnostics", 
                "typeHandlerVersion": "1.5", 
                "settings": "Include PublicConfig XML as a raw string", 
                "protectedSettings": "Include PrivateConfig XML as a raw string”", 
                "rolesAppliedTo": [ 
                  "WebRole1" 
                ] 
              } 
            }
          ]
        },

```


## <a name="next-steps"></a>Следующие шаги 
- Ознакомьтесь с [предварительными требованиями для развертывания](deploy-prerequisite.md) облачных служб (Расширенная поддержка).
- Ознакомьтесь с [часто задаваемыми вопросами об Облачных службах (расширенная поддержка)](faq.md).
- Разверните Облачную службу (расширенная поддержка) с помощью [портала Azure](deploy-portal.md), [PowerShell](deploy-powershell.md), [шаблона](deploy-template.md) или [Visual Studio](deploy-visual-studio.md).
