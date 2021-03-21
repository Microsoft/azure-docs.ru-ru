---
title: Создание контейнера облачной службы (классическая модель) с помощью PowerShell | Документация Майкрософт
description: В этой статье объясняется, как создать контейнер облачной службы с помощью PowerShell. В контейнере размещаются веб- и рабочие роли.
ms.topic: article
ms.service: cloud-services
ms.date: 10/14/2020
ms.author: tagore
author: tanmaygore
ms.reviewer: mimckitt
ms.custom: ''
ms.openlocfilehash: 39fe439e37b1af4e833396ef83205729af8c7ad3
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102610422"
---
# <a name="use-an-azure-powershell-command-to-create-an-empty-cloud-service-classic-container"></a>Использование команды Azure PowerShell для создания пустого контейнера облачной службы (классический)

> [!IMPORTANT]
> [Облачные службы Azure (Расширенная поддержка)](../cloud-services-extended-support/overview.md) — это новая модель развертывания на основе Azure Resource Manager для продукта облачных служб Azure.После этого изменения облачные службы Azure, работающие в модели развертывания на основе Service Manager Azure, были переименованы как облачные службы (классические), и все новые развертывания должны использовать [облачные службы (Расширенная поддержка)](../cloud-services-extended-support/overview.md).

В этой статье объясняется, как быстро создать контейнер облачных служб с помощью командлетов Azure PowerShell. Выполните следующие действия.

1. Установите командлет Microsoft Azure PowerShell со страницы [Загрузки Azure PowerShell](https://aka.ms/webpi-azps) .
2. Запустите командную строку PowerShell.
3. Войдите, используя [Add-AzureAccount](/powershell/module/servicemanagement/azure.service/add-azureaccount) .

   > [!NOTE]
   > Инструкции по установке командлета Azure PowerShell и подключению к подписке Azure см. в статье [Установка и настройка Azure PowerShell](/powershell/azure/).
   >
   >
4. Используйте командлет **New-AzureService** , чтобы создать пустой контейнер облачной службы Azure.

   ```
   New-AzureService [-ServiceName] <String> [-AffinityGroup] <String> [[-Label] <String>] [[-Description] <String>] [[-ReverseDnsFqdn] <String>] [<CommonParameters>]
   New-AzureService [-ServiceName] <String> [-Location] <String> [[-Label] <String>] [[-Description] <String>] [[-ReverseDnsFqdn] <String>] [<CommonParameters>]
   ```

5. Ниже приводится пример вызова командлета:

   ```powershell
   New-AzureService -ServiceName "mytestcloudservice" -Location "Central US" -Label "mytestcloudservice"
   ```

Чтобы получить дополнительные сведения о создании облачной службы Azure, выполните следующую команду:

```powershell
Get-help New-AzureService
```

### <a name="next-steps"></a>Следующие шаги

* Чтобы управлять развертыванием облачной службы, используйте команды [Get-AzureService](/powershell/module/servicemanagement/azure.service/Get-AzureService), [Remove-AzureService](/powershell/module/servicemanagement/azure.service/Remove-AzureService) и [Set-AzureService](/powershell/module/servicemanagement/azure.service/set-azureservice). Кроме того, дополнительные сведения можно получить в статье [Настройка облачных служб](cloud-services-how-to-configure-portal.md) .
* Чтобы опубликовать проект облачной службы в Azure, используйте пример кода **PublishCloudService.ps1** из [репозитория заархивированных облачных служб](https://github.com/MicrosoftDocs/azure-cloud-services-files/tree/master/Scripts/cloud-services-continuous-delivery).