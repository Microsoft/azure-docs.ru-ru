---
title: Включение доменных служб Azure DS с помощью шаблона | Документация Майкрософт
description: Узнайте, как настроить и включить доменные службы Azure Active Directory с помощью шаблона Azure Resource Manager
services: active-directory-ds
author: justinha
manager: daveba
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: sample
ms.date: 07/09/2020
ms.author: justinha
ms.openlocfilehash: e18825da64d0d200f55ce72985ac843b93b1e612
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96618796"
---
# <a name="create-an-azure-active-directory-domain-services-managed-domain-using-an-azure-resource-manager-template"></a>Создание управляемого домена доменных служб Azure Active Directory с помощью шаблона Azure Resource Manager

Доменные службы Azure Active Directory (Azure AD DS) предоставляют управляемые доменные службы, отвечающие за присоединение к домену, применение групповой политики, использование протокола LDAP, а также выполнение аутентификации Kerberos или NTLM (полностью поддерживается Windows Server Active Directory). Эти доменные службы можно использовать без необходимости развертывать, администрировать и обновлять контроллеры домена. Azure AD DS интегрируется с существующим клиентом Azure AD. Такая интеграция позволяет пользователям входить в систему с корпоративными учетными данными. При этом вы можете использовать существующие группы и учетные записи пользователей для защиты доступа к ресурсам.

В этой статье показано, как создать управляемый домен с помощью шаблона Azure Resource Manager. Вспомогательные ресурсы создаются с помощью Azure PowerShell.

## <a name="prerequisites"></a>Предварительные требования

Для работы с этой статьей требуются следующие ресурсы:

* Установите и настройте Azure PowerShell.
    * При необходимости выполните инструкции по [установке модуля Azure PowerShell и его подключению к подписке Azure](/powershell/azure/install-az-ps).
    * Войдите в подписку Azure с помощью командлета [Connect-AzAccount][Connect-AzAccount].
* Установите и настройте Azure AD PowerShell.
    * При необходимости выполните инструкции по [установке модуля Azure AD PowerShell и подключению к Azure AD](/powershell/azure/active-directory/install-adv2).
    * Войдите в клиент Azure AD с помощью командлета [Connect-AzureAD][Connect-AzureAD].
* Привилегии *глобального администратора* для клиента Azure AD, чтобы включить доменные службы Azure AD.
* Для создания нужных ресурсов Azure AD DS требуются привилегии *участника* в подписке Azure.

## <a name="dns-naming-requirements"></a>Требования к именованию DNS

При создании управляемого домена Azure AD DS вы должны указать DNS-имя. При выборе имени DNS следует учитывать ряд рекомендаций.

* **Встроенное доменное имя.** По умолчанию используется стандартное доменное имя каталога (с суффиксом *.onmicrosoft.com*). Если вы хотите разрешить для управляемого домена доступ по защищенному протоколу LDAP, вы не сможете создать цифровой сертификат для защиты подключения к домену по умолчанию. Домен *onmicrosoft.com* принадлежит корпорации Майкрософт, поэтому центр сертификации не будет выдавать для него сертификаты.
* **Имена личных доменов.** Популярный подход заключается в том, чтобы указать имя личного домена. Обычно оно принадлежит вашей организации и поддерживает маршрутизацию. При использовании личного домена с поддержкой маршрутизации трафик можно передавать так, нужно приложениям.
* **Суффиксы доменов, не поддерживающие маршрутизацию.** Обычно мы рекомендуем не использовать суффиксы доменных имен, не поддерживающие маршрутизацию, такие как *contoso.local*. Суффикс *.local* не маршрутизируется и его использование может привести к ошибкам при разрешении имен DNS.

> [!TIP]
> Если вы создаете личное доменное имя, важно не допускать конфликтов с существующими пространствами имен DNS. Рекомендуется использовать доменное имя отдельно от существующего локального пространства имен DNS или пространства имен DNS от Azure.
>
> Например, если вы используете существующее пространство имен DNS *contoso.com*, создайте управляемый домен с именем личного домена *aaddscontoso.com*. Если вам нужно использовать защищенный протокол LDAP, необходимо зарегистрировать и присвоить это имя личного домена, чтобы создать необходимые сертификаты.
>
> Возможно, вам потребуется создать дополнительные записи DNS для других служб в среде или условные серверы переадресации DNS между существующими пространствами имен DNS в вашей среде. Например, если корневое имя DNS используется в качестве адреса сайта, размещенного на веб-сервере, могут возникнуть конфликты имен, для устранения которых потребуются дополнительные записи DNS.
>
> В этом примере и инструкциях в качестве примера используется личный домен *aaddscontoso.com*. Используйте во всех командах собственное доменное имя.

Также применимы следующие ограничения в отношении DNS:

* **Ограничения для префикса домена.** Невозможно создать управляемый домен с префиксом длиннее 15 символов. Длина указанного префикса доменного имени (например, *aaddscontoso* в доменном имени *aaddscontoso.com*) не должна превышать 15 символов.
* **Конфликты сетевых имен.** Нельзя использовать для управляемого домена имя DNS, которое уже существует в виртуальной сети. В частности, к конфликту имен приведут следующие сценарии:
    * Если в виртуальной сети уже существует домен Active Directory с таким же доменным именем DNS.
    * Если для виртуальной сети, в которой планируется использовать управляемый домен, установлено VPN-подключение к локальной сети. в этом случае необходимо убедиться в том, что в локальной сети нет домена с таким же DNS-именем домена;
    * Если в виртуальной сети Azure существует облачная служба Azure с таким же именем.

## <a name="create-required-azure-ad-resources"></a>Создание необходимых ресурсов Azure AD

Для доменных служб Azure Active Directory требуется субъект-служба и группа Azure AD. Эти ресурсы позволяют управляемому домену синхронизировать данные и определять, каким пользователям предоставлять разрешения администратора в управляемом домене.

Сначала зарегистрируйте поставщика ресурсов доменных служб Azure AD с помощью командлета [Register-AzResourceProvider][Register-AzResourceProvider]:

```powershell
Register-AzResourceProvider -ProviderNamespace Microsoft.AAD
```

Создайте субъект-службу Azure AD с помощью командлета [New-AzureADServicePrincipal][New-AzureADServicePrincipal] для Azure AD DS, чтобы обеспечить возможность взаимодействия и проверки подлинности. Используется определенный идентификатор приложения с названием *Domain Controller Services*(Службы контроллера домена) и номером идентификатора *6ba9a5d4-8456-4118-b521-9c5ca10cdf84*. Не изменяйте этот идентификатор приложения.

```powershell
New-AzureADServicePrincipal -AppId "6ba9a5d4-8456-4118-b521-9c5ca10cdf84"
```

Создайте группу Azure AD с именем *Администраторы контроллера домена AAD* с помощью командлета [New-AzureADGroup][New-AzureADGroup]. Пользователи, добавленные в эту группу, получают разрешения на выполнение задач администрирования в управляемом домене.

```powershell
New-AzureADGroup -DisplayName "AAD DC Administrators" `
  -Description "Delegated group to administer Azure AD Domain Services" `
  -SecurityEnabled $true -MailEnabled $false `
  -MailNickName "AADDCAdministrators"
```

После создания группы *Администраторы контроллера домена AAD* добавьте в группу пользователя с помощью командлета [Add-AzureADGroupMember][Add-AzureADGroupMember]. Сначала получите идентификатор объекта группы *Администраторы контроллера домена AAD* с помощью командлета [Get-AzureADGroup][Get-AzureADGroup], затем получите идентификатор объекта нужного пользователя с помощью командлета [Get-AzureADUser][Get-AzureADUser].

В приведенном ниже примере показан идентификатор объекта пользователя для учетной записи с именем участника-пользователя `admin@contoso.onmicrosoft.com`. Используйте вместо этой учетной записи пользователя имя участника-пользователя, которого нужно добавить в группу *Администраторы контроллера домена AAD*

```powershell
# First, retrieve the object ID of the newly created 'AAD DC Administrators' group.
$GroupObjectId = Get-AzureADGroup `
  -Filter "DisplayName eq 'AAD DC Administrators'" | `
  Select-Object ObjectId

# Now, retrieve the object ID of the user you'd like to add to the group.
$UserObjectId = Get-AzureADUser `
  -Filter "UserPrincipalName eq 'admin@contoso.onmicrosoft.com'" | `
  Select-Object ObjectId

# Add the user to the 'AAD DC Administrators' group.
Add-AzureADGroupMember -ObjectId $GroupObjectId.ObjectId -RefObjectId $UserObjectId.ObjectId
```

Затем создайте группу ресурсов с помощью командлета [New-AzResourceGroup][New-AzResourceGroup]. В следующем примере создается группа ресурсов с именем *myResourceGroup* в регионе *westus*. Используйте желаемое имя и регион:

```powershell
New-AzResourceGroup `
  -Name "myResourceGroup" `
  -Location "WestUS"
```

Если вы выбрали регион, который поддерживает зоны доступности, ресурсы Azure AD DS распределяются между зонами для дополнительной избыточности. Зоны доступности — уникальные физические расположения в пределах одного региона Azure. Каждая зона состоит из одного или нескольких центров обработки данных, оснащенных независимыми системами электроснабжения, охлаждения и сетевого взаимодействия. Чтобы обеспечить устойчивость, во всех включенных областях используются минимум три отдельные зоны.

Вы не можете настроить распределение Azure AD DS между зонами. Платформа Azure автоматически обрабатывает распределение ресурсов зоны. Дополнительные сведения о зонах доступности и регионах см. в статье [Что такое зоны доступности в Azure?][availability-zones].

## <a name="resource-definition-for-azure-ad-ds"></a>Определение ресурса для Azure AD DS

Для определения ресурса Resource Manager требуются следующие параметры конфигурации:

| Параметр               | Значение |
|-------------------------|---------|
| domainName              | Доменное имя DNS для управляемого домена, учитывая описанные выше ограничения на именование префиксов и конфликтов. |
| filteredSync            | Службы Azure AD DS позволяют синхронизировать *всех пользователей и все группы*, которые доступны в Azure AD, или только *определенные группы*.<br /><br /> См. сведения об [ограниченной синхронизации в области в доменных службах Azure AD][scoped-sync].|
| notificationSettings    | Если в управляемом домене поддерживается генерация оповещений, можно отправлять уведомления по электронной почте. <br /><br />Для этих уведомлений можно *включить* *глобальных администраторов* клиента Azure и членов группы *Администраторы контроллера домена AAD*.<br /><br /> Если это необходимо, вы можете добавить дополнительных получателей уведомлений при наличии важных предупреждений.|
| domainConfigurationType | По умолчанию управляемый домен создается как лес объектов *User*. Лес этого типа синхронизирует все объекты из Azure AD, включая учетные записи пользователей, созданные в локальной среде AD DS. Для создания пользовательского леса не нужно указывать значение *domainConfiguration*.<br /><br /> Лес *ресурсов* синхронизирует только пользователей и группы, созданные непосредственно в Azure AD. Чтобы создать лес ресурсов, задайте значение *ResourceTrusting*.<br /><br />Дополнительные сведения о лесах *ресурсов*, в том числе об их предназначении и о создании доверия между лесами и локальными доменами AD DS, см. в [этой статье][resource-forests].|

В приведенном ниже сокращенном определении параметров показано, как объявляются эти значения. Создается лес пользователей *aaddscontoso.com*, в котором все пользователи Azure AD синхронизируются с управляемым доменом.

```json
"parameters": {
    "domainName": {
        "value": "aaddscontoso.com"
    },
    "filteredSync": {
        "value": "Disabled"
    },
    "notificationSettings": {
        "value": {
            "notifyGlobalAdmins": "Enabled",
            "notifyDcAdmins": "Enabled",
            "additionalRecipients": []
        }
    },
    [...]
}
```

Затем применяется следующий сжатый шаблон Resource Manager для типа ресурса, чтобы определить и создать управляемый домен. Виртуальная сеть и подсеть Azure должны уже существовать или быть созданы как часть шаблона Resource Manager. Управляемый домен должен быть подключен к этой подсети.

```json
"resources": [
    {
        "apiVersion": "2017-06-01",
        "type": "Microsoft.AAD/DomainServices",
        "name": "[parameters('domainName')]",
        "location": "[parameters('location')]",
        "dependsOn": [
            "[concat('Microsoft.Network/virtualNetworks/', parameters('vnetName'))]"
        ],
        "properties": {
            "domainName": "[parameters('domainName')]",
            "subnetId": "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/', resourceGroup().name, '/providers/Microsoft.Network/virtualNetworks/', parameters('vnetName'), '/subnets/', parameters('subnetName'))]",
            "filteredSync": "[parameters('filteredSync')]",
            "notificationSettings": "[parameters('notificationSettings')]"
        }
    },
    [...]
]
```

Эти параметры и тип ресурса можно использовать как часть более широкого шаблона Resource Manager для развертывания управляемого домена, как показано в следующем разделе.

## <a name="create-a-managed-domain-using-sample-template"></a>Создание управляемого домена с помощью примера шаблона

Следующий полный пример шаблона Resource Manager позволяет создать управляемый домен, вспомогательную виртуальную сеть, подсеть и правила группы безопасности сети. Правила группы безопасности сети необходимы для защиты управляемого домена и обеспечения бесперебойного трафика. Создается пользовательский лес с DNS-именем *aaddscontoso.com* и выполняется синхронизация всех пользователей из Azure AD:

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "apiVersion": {
            "value": "2017-06-01"
        },
        "domainConfigurationType": {
            "value": "FullySynced"
        },
        "domainName": {
            "value": "aaddscontoso.com"
        },
        "filteredSync": {
            "value": "Disabled"
        },
        "location": {
            "value": "westus"
        },
        "notificationSettings": {
            "value": {
                "notifyGlobalAdmins": "Enabled",
                "notifyDcAdmins": "Enabled",
                "additionalRecipients": []
            }
        },
        "subnetName": {
            "value": "aadds-subnet"
        },
        "vnetName": {
            "value": "aadds-vnet"
        },
        "vnetAddressPrefixes": {
            "value": [
                "10.1.0.0/24"
            ]
        },
        "subnetAddressPrefix": {
            "value": "10.1.0.0/24"
        },
        "nsgName": {
            "value": "aadds-nsg"
        }
    },
    "resources": [
        {
            "apiVersion": "2017-06-01",
            "type": "Microsoft.AAD/DomainServices",
            "name": "[parameters('domainName')]",
            "location": "[parameters('location')]",
            "dependsOn": [
                "[concat('Microsoft.Network/virtualNetworks/', parameters('vnetName'))]"
            ],
            "properties": {
                "domainName": "[parameters('domainName')]",
                "subnetId": "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/', resourceGroup().name, '/providers/Microsoft.Network/virtualNetworks/', parameters('vnetName'), '/subnets/', parameters('subnetName'))]",
                "filteredSync": "[parameters('filteredSync')]",
                "domainConfigurationType": "[parameters('domainConfigurationType')]",
                "notificationSettings": "[parameters('notificationSettings')]"
            }
        },
        {
            "type": "Microsoft.Network/NetworkSecurityGroups",
            "name": "[parameters('nsgName')]",
            "location": "[parameters('location')]",
            "properties": {
                "securityRules": [
                    {
                        "name": "AllowSyncWithAzureAD",
                        "properties": {
                            "access": "Allow",
                            "priority": 101,
                            "direction": "Inbound",
                            "protocol": "Tcp",
                            "sourceAddressPrefix": "AzureActiveDirectoryDomainServices",
                            "sourcePortRange": "*",
                            "destinationAddressPrefix": "*",
                            "destinationPortRange": "443"
                        }
                    },
                    {
                        "name": "AllowPSRemoting",
                        "properties": {
                            "access": "Allow",
                            "priority": 301,
                            "direction": "Inbound",
                            "protocol": "Tcp",
                            "sourceAddressPrefix": "AzureActiveDirectoryDomainServices",
                            "sourcePortRange": "*",
                            "destinationAddressPrefix": "*",
                            "destinationPortRange": "5986"
                        }
                    },
                    {
                        "name": "AllowRD",
                        "properties": {
                            "access": "Allow",
                            "priority": 201,
                            "direction": "Inbound",
                            "protocol": "Tcp",
                            "sourceAddressPrefix": "CorpNetSaw",
                            "sourcePortRange": "*",
                            "destinationAddressPrefix": "*",
                            "destinationPortRange": "3389"
                        }
                    }
                ]
            },
            "apiVersion": "2018-04-01"
        },
        {
            "type": "Microsoft.Network/virtualNetworks",
            "name": "[parameters('vnetName')]",
            "location": "[parameters('location')]",
            "apiVersion": "2018-04-01",
            "dependsOn": [
                "[concat('Microsoft.Network/NetworkSecurityGroups/', parameters('nsgName'))]"
            ],
            "properties": {
                "addressSpace": {
                    "addressPrefixes": "[parameters('vnetAddressPrefixes')]"
                },
                "subnets": [
                    {
                        "name": "[parameters('subnetName')]",
                        "properties": {
                            "addressPrefix": "[parameters('subnetAddressPrefix')]",
                            "networkSecurityGroup": {
                                "id": "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/', resourceGroup().name, '/providers/Microsoft.Network/NetworkSecurityGroups/', parameters('nsgName'))]"
                            }
                        }
                    }
                ]
            }
        }
    ],
    "outputs": {}
}
```

Этот шаблон можно развернуть с помощью предпочтительного метода развертывания, например с помощью [портала Azure][portal-deploy], [Azure PowerShell][powershell-deploy] или конвейера CI/CD. В следующем примере используется командлет [New-AzResourceGroupDeployment][New-AzResourceGroupDeployment]. Укажите собственное имя группы ресурсов и имя файла шаблона:

```powershell
New-AzResourceGroupDeployment -ResourceGroupName "myResourceGroup" -TemplateFile <path-to-template>
```

Создание ресурса и возврат управления в командную строку PowerShell занимает несколько минут. Подготовка к работе управляемого домена продолжается в фоновом режиме. На завершение развертывания может понадобиться до часа. На портале Azure на странице **Обзор** для управляемого домена на этом этапе развертывания отображаются сведения о текущем состоянии.

Когда на портале Azure отобразятся сведения о том, что подготовка управляемого домена к работе завершена, выполните следующие задачи.

* Обновите параметры DNS для виртуальной сети, чтобы виртуальные машины могли найти управляемый домен для присоединения к нему или для аутентификации.
    * Чтобы настроить DNS, выберите на портале управляемый домен. В окне **Обзор** отобразится запрос на автоматическую настройку этих параметров DNS.
* [Включите синхронизацию паролей с Azure AD DS](tutorial-create-instance.md#enable-user-accounts-for-azure-ad-ds), чтобы пользователи могли входить в управляемый домен с помощью рабочих учетных данных.

## <a name="next-steps"></a>Дальнейшие действия

Чтобы увидеть управляемый домен в действии, [присоедините к нему виртуальную машину Windows][windows-join], настройте [защищенный протокол LDAP][tutorial-ldaps] и [синхронизацию хэша паролей][tutorial-phs].

<!-- INTERNAL LINKS -->
[windows-join]: join-windows-vm.md
[tutorial-ldaps]: tutorial-configure-ldaps.md
[tutorial-phs]: tutorial-configure-password-hash-sync.md
[availability-zones]: ../availability-zones/az-overview.md
[portal-deploy]: ../azure-resource-manager/templates/deploy-portal.md
[powershell-deploy]: ../azure-resource-manager/templates/deploy-powershell.md
[scoped-sync]: scoped-synchronization.md
[resource-forests]: concepts-resource-forest.md

<!-- EXTERNAL LINKS -->
[Connect-AzAccount]: /powershell/module/Az.Accounts/Connect-AzAccount
[Connect-AzureAD]: /powershell/module/AzureAD/Connect-AzureAD
[New-AzureADServicePrincipal]: /powershell/module/AzureAD/New-AzureADServicePrincipal
[New-AzureADGroup]: /powershell/module/AzureAD/New-AzureADGroup
[Add-AzureADGroupMember]: /powershell/module/AzureAD/Add-AzureADGroupMember
[Get-AzureADGroup]: /powershell/module/AzureAD/Get-AzureADGroup
[Get-AzureADUser]: /powershell/module/AzureAD/Get-AzureADUser
[Register-AzResourceProvider]: /powershell/module/Az.Resources/Register-AzResourceProvider
[New-AzResourceGroup]: /powershell/module/Az.Resources/New-AzResourceGroup
[Get-AzSubscription]: /powershell/module/Az.Accounts/Get-AzSubscription
[cloud-shell]: ../cloud-shell/cloud-shell-windows-users.md
[naming-prefix]: /windows-server/identity/ad-ds/plan/selecting-the-forest-root-domain
[New-AzResourceGroupDeployment]: /powershell/module/Az.Resources/New-AzResourceGroupDeployment