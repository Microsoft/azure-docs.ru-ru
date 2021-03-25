---
title: Краткое руководство. Создание экземпляра службы "Управление API Azure" с помощью PowerShell | Документация Майкрософт
description: Создание экземпляра службы "Управление API Azure" с помощью Azure PowerShell.
services: api-management
documentationcenter: ''
author: vladvino
ms.service: api-management
ms.topic: quickstart
ms.custom: mvc
ms.date: 09/14/2020
ms.author: apimpm
ms.openlocfilehash: eb2c42d26a85a07518a018ba5b8817f13d3cd17f
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "90707069"
---
# <a name="quickstart-create-a-new-azure-api-management-service-instance-by-using-powershell"></a>Краткое руководство. Создание экземпляра службы "Управление API Azure" с помощью PowerShell

Служба управления API Azure помогает организациям публиковать API-интерфейсы для внешних пользователей, партнеров и собственных разработчиков, раскрывая таким образом потенциал своих данных и услуг. Служба управления API предоставляет базовые возможности для успешного выполнения программы API за счет привлечения разработчиков, бизнес-аналитики, анализа, безопасности и защищенности. Служба управления API позволяет создавать современные шлюзы API для существующих серверных служб, размещенных в любом месте, и управлять ими. Дополнительные сведения см. в статье [Общие сведения о Службе контейнеров Azure](api-management-key-concepts.md).

В этом кратком руководстве описаны шаги по созданию экземпляра службы "Управление API" с помощью командлетов Azure PowerShell.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Чтобы установить и использовать PowerShell локально для работы с этим руководством, вам понадобится модуль Azure PowerShell 1.0 или последующих версий. Чтобы узнать версию, выполните команду `Get-Module -ListAvailable Az`. Если вам необходимо выполнить обновление, ознакомьтесь со статьей, посвященной [установке модуля Azure PowerShell](/powershell/azure/install-Az-ps). При использовании PowerShell на локальном компьютере также нужно запустить `Connect-AzAccount`, чтобы создать подключение к Azure.

## <a name="create-resource-group"></a>Создать группу ресурсов

Создайте группу ресурсов Azure с помощью командлета [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Группа ресурсов — это логический контейнер, в котором происходит развертывание ресурсов Azure и управление ими. 

В следующей команде создается группа ресурсов с именем *myResourceGroup* в расположении "западная часть США".

```azurepowershell-interactive
New-AzResourceGroup -Name myResourceGroup -Location WestUS
```

## <a name="create-an-api-management-service"></a>Создание службы управления API

Теперь, когда у вас есть группа ресурсов, можно создать экземпляр службы "Управление API". Создайте его с помощью команды [New-AzApiManagement](/powershell/module/az.apimanagement/new-azapimanagement), а также укажите имя службы и сведения об издателе. Имя пользователя должно быть уникальным в пределах Azure.

В следующем примере для имени службы используется *myapim*. Замените его уникальным значением. Также обновите имя организации издателя API и адрес электронной почты администратора для получения уведомлений.

По умолчанию команда создает экземпляр на уровне разработчика, экономичный вариант для оценки службы "Управление API Azure". Этот уровень не предназначен для использования в рабочей среде. Дополнительные сведения о масштабировании категорий службы управления API см. в статье о [повышении категории и масштабировании](upgrade-and-scale.md).

> [!NOTE]
> Выполнение этой операции занимает длительное время. Обычно создание и активация службы "Управление API" на этом уровне занимает от 30 до 40 минут.

```azurepowershell-interactive
New-AzApiManagement -Name "myapim" -ResourceGroupName "myResourceGroup" `
  -Location "West US" -Organization "Contoso" -AdminEmail "admin@contoso.com" 
```

При возвращении команды выполните команду [Get-AzApiManagement](/powershell/module/az.apimanagement/get-azapimanagement), чтобы просмотреть свойства службы "Управление API Azure". После активации состояние подготовки изменится на "Успешно", а экземпляр службы получит несколько связанных URL-адресов. Например, такое:

```azurepowershell-interactive
Get-AzApiManagement -Name "myapim" -ResourceGroupName "myResourceGroup" 
```

Выходные данные примера:

```console
PublicIPAddresses                     : {203.0.113.1}
PrivateIPAddresses                    :
Id                                    : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroup/providers/Microsoft.ApiManagement/service/myapim
Name                                  : myapim
Location                              : West US
Sku                                   : Developer
Capacity                              : 1
CreatedTimeUtc                        : 9/9/2020 9:07:43 PM
ProvisioningState                     : Succeeded
RuntimeUrl                            : https://myapim.azure-api.net
RuntimeRegionalUrl                    : https://myapi-westus-01.regional.azure-api.net
PortalUrl                             : https://myapim.portal.azure-api.net
DeveloperPortalUrl                    : https://myapim.developer.azure-api.net
ManagementApiUrl                      : https://myapim.management.azure-api.net
ScmUrl                                : https://myapim.scm.azure-api.net
PublisherEmail                        : admin@contoso.com
OrganizationName                      : Contoso
NotificationSenderEmail               : apimgmt-noreply@mail.windowsazure.com
VirtualNetwork                        :
VpnType                               : None
PortalCustomHostnameConfiguration     :
ProxyCustomHostnameConfiguration      : {myapim.azure-api.net}
ManagementCustomHostnameConfiguration :
ScmCustomHostnameConfiguration        :
DeveloperPortalHostnameConfiguration  :
SystemCertificates                    :
Tags                                  : {}
AdditionalRegions                     : {}
SslSetting                            : Microsoft.Azure.Commands.ApiManagement.Models.PsApiManagementSslSetting
Identity                              :
EnableClientCertificate               :
ResourceGroupName                     : myResourceGroup

```

Когда экземпляр службы "Управление API" будет развернут, его можно использовать. Изучите сведения из учебника, чтобы [импортировать и опубликовать первый API](import-and-publish.md).

## <a name="clean-up-resources"></a>Очистка ресурсов

Вы можете удалить ставшие ненужными группу ресурсов и все связанные с ней ресурсы, выполнив команду [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup).

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroup
```

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Импорт и публикация первого API](import-and-publish.md)
