---
title: Установка локального шлюза данных для Azure Analysis Services | Документация Майкрософт
description: Узнайте, как установить и настроить локальный шлюз данных для подключения к локальным источникам данных с сервера Azure Analysis Services.
author: minewiskan
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 07/29/2020
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: 64bd9e4a4cf78d2628e946af30c2d290ff002cf7
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93081150"
---
# <a name="install-and-configure-an-on-premises-data-gateway"></a>Установка и настройка локального шлюза данных

Локальный шлюз данных является обязательным, когда один или несколько серверов Azure Analysis Services в том же регионе подключаются к локальным источникам данных.  Хотя устанавливаемый Шлюз совпадает с используемыми другими службами, такими как Power BI, Power Apps и Logic Apps, при установке для Azure Analysis Services необходимо выполнить некоторые дополнительные действия. Эта статья об установке относится к **Azure Analysis Services**. 

Дополнительные сведения о том, как Azure Analysis Services работает с шлюзом, см. в разделе [Подключение к локальным источникам данных](analysis-services-gateway.md). Дополнительные сведения о расширенных сценариях установки и шлюзе см. в [документации по локальным шлюзам данных](/data-integration/gateway/service-gateway-onprem).

## <a name="prerequisites"></a>Предварительные условия

**Минимальные требования:**

* .NET Framework 4.5;
* 64-разрядная версия Windows 8/Windows Server 2012 R2 (или более поздней версии)

**Рекомендуется:**

* 8-ядерный ЦП;
* 8 ГБ ОЗУ;
* 64-разрядная версия Windows 8/Windows Server 2012 R2 (или более поздней версии)

**Важные моменты:**

* Во время установки и регистрации шлюза в Azure выбирается регион по умолчанию для вашей подписки. Можно выбрать другую подписку и регион. При наличии серверов в нескольких регионах необходимо установить шлюз для каждого региона. 
* Шлюз нельзя устанавливать на контроллере домена.
* На одном компьютере можно установить только один шлюз.
* Устанавливайте шлюз на компьютере, который не выключается и не переходит в спящий режим.
* Не устанавливайте шлюз на компьютере с беспроводным подключением к сети. Это может стать причиной снижения производительности.
* Чтобы установить шлюз, учетная запись пользователя, с помощью которой вы вошли на компьютер, должна предоставлять право входа в систему в качестве службы. После установки служба локального шлюза данных использует учетную запись NT SERVICE\PBIEgwService для входа в систему в качестве службы. Учетную запись можно изменить при настройке или в разделе "Службы" после настройки. Убедитесь, что параметры Групповой политики позволяют предоставить права входа в систему в качестве службы для учетной записи, с помощью которой вы вошли в систему при установке, и выбранной учетной записи службы.
* Войдите в Azure с помощью учетной записи в Azure AD для того же [клиента](/previous-versions/azure/azure-services/jj573650(v=azure.100)#what-is-an-azure-ad-tenant), что и подписка, в которой вы регистрируете шлюз. Учетные записи Azure B2B (гостевые) не поддерживаются при установке и регистрации шлюза.
* Если источники данных находятся в виртуальной сети Azure, необходимо настроить свойство сервера [AlwaysUseGateway](analysis-services-vnet-gateway.md).

## <a name="download"></a>Скачать

 [Скачайте шлюз](https://go.microsoft.com/fwlink/?LinkId=820925&clcid=0x409).

## <a name="install"></a>Установка

1. Запустите программу установки.

2. Выберите **локальный шлюз данных**.

   ![Выберите пункт](media/analysis-services-gateway-install/aas-gateway-installer-select.png)

2. Выберите расположение, примите условия соглашения и нажмите кнопку **Установить**.

   ![Расположение для установки и условия лицензии](media/analysis-services-gateway-install/aas-gateway-installer-accept.png)

3. Войдите в Azure. Учетная запись должна быть у вашего Azure Active Directory клиента. Эта учетная запись используется для администратора шлюза. Учетные записи Azure B2B (гостевые) не поддерживаются при установке и регистрации шлюза.

   ![Вход в Azure](media/analysis-services-gateway-install/aas-gateway-installer-account.png)

   > [!NOTE]
   > При входе с использованием учетной записи домена она сопоставляется с учетной записью организации в Azure AD. Учетная запись вашей организации используется как администратор шлюза.

## <a name="register"></a>Зарегистрировать

Чтобы создать ресурс шлюза в Azure, необходимо зарегистрировать локальный экземпляр, установленный с помощью облачной службы шлюза. 

1.  Выберите **Регистрация нового шлюза на этом компьютере**.

    ![Снимок экрана, посвященный параметру регистрация нового шлюза на этом компьютере.](media/analysis-services-gateway-install/aas-gateway-register-new.png)

2. Введите имя и ключ восстановления для шлюза. По умолчанию шлюз использует регион, установленный по умолчанию для вашей подписки. Если вам необходим другой регион, выберите **Смена региона**.

    > [!IMPORTANT]
    > Сохраните ключ восстановления в безопасном месте. Ключ восстановления требуется для перехвата, переноса или восстановления шлюза. 

   ![Зарегистрировать](media/analysis-services-gateway-install/aas-gateway-register-name.png)


## <a name="create-an-azure-gateway-resource"></a>Создание ресурса шлюза Azure

После установки и регистрации шлюза необходимо создать ресурс шлюза в Azure. Войдите в Azure с помощью учетной записи, использованной при регистрации шлюза.

1. В портал Azure щелкните **создать ресурс**, а затем найдите **локальный шлюз данных** и нажмите кнопку **создать**.

   ![Создание ресурса шлюза](media/analysis-services-gateway-install/aas-gateway-new-azure-resource.png)

2. В разделе **Create connection gateway** (Создать шлюз для подключения) введите следующие параметры:

   * **Имя.** Введите имя для вашего ресурса шлюза. 

   * **Подписка.** Выберите подписку Azure, связанную с вашим ресурсом шлюза. 
   
     Подписка по умолчанию основана на учетной записи Azure, которая использовалась для входа.

   * **Группа ресурсов.** Создайте группу ресурсов Azure или выберите имеющуюся.

   * **Местоположение.** Выберите регион, в котором зарегистрирован шлюз.

   * **Имя установки**: Если установка шлюза еще не выбрана, выберите шлюз, установленный на компьютере и зарегистрированный. 

     Когда все будет готово, нажмите **Создать**.

## <a name="connect-gateway-resource-to-server"></a>Подключение ресурса шлюза к серверу

> [!NOTE]
> Подключение к ресурсу шлюза в другой подписке с сервера не поддерживается на портале, но поддерживается с помощью PowerShell.

# <a name="portal"></a>[Портал](#tab/azure-portal)

1. В обзоре сервера Azure Analysis Services щелкните **Локальный шлюз данных**.

   ![Подключение сервера к шлюзу](media/analysis-services-gateway-install/aas-gateway-connect-server.png)

2. В окне **Pick an On-Premises Data Gateway to connect** (Выбрать локальный шлюз данных для подключения) выберите ресурс шлюза и щелкните **Connect selected gateway** (Подключить выбранный шлюз).

   ![Подключение сервера к ресурсу шлюза](media/analysis-services-gateway-install/aas-gateway-connect-resource.png)

    > [!NOTE]
    > Если шлюз не отображается в списке, сервер, вероятно, находится не в том же районе, что и регионы, указанные при регистрации шлюза.

    При успешном подключении сервера к ресурсу шлюза состояние будет отображаться как **подключено**.


    ![Подключение сервера к ресурсу шлюза успешно выполнено](media/analysis-services-gateway-install/aas-gateway-connect-success.png)

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Чтобы получить ResourceID шлюза, используйте [Get-азресаурце](/powershell/module/az.resources/get-azresource) . Затем подключите ресурс шлюза к существующему или новому серверу, указав параметр **-гатевайресаурцеид** в [Set-азаналисиссервицессервер](/powershell/module/az.analysisservices/set-azanalysisservicesserver) или [New-азаналисиссервицессервер](/powershell/module/az.analysisservices/new-azanalysisservicesserver).

Чтобы получить идентификатор ресурса шлюза, выполните следующие действия.

```azurepowershell-interactive
Connect-AzAccount -Tenant $TenantId -Subscription $subscriptionIdforGateway -Environment "AzureCloud"
$GatewayResourceId = $(Get-AzResource -ResourceType "Microsoft.Web/connectionGateways" -Name $gatewayName).ResourceId  

```

Чтобы настроить существующий сервер, выполните следующие действия.

```azurepowershell-interactive
Connect-AzAccount -Tenant $TenantId -Subscription $subscriptionIdforAzureAS -Environment "AzureCloud"
Set-AzAnalysisServicesServer -ResourceGroupName $RGName -Name $servername -GatewayResourceId $GatewayResourceId

```
---

Вот и все. Если необходимо открывать порты или устранять неполадки, требуется извлечь [локальный шлюз данных](analysis-services-gateway.md).

## <a name="next-steps"></a>Дальнейшие действия

* [Управление службами Analysis Services](analysis-services-manage.md)   
* [Получение данных из Azure Analysis Services](analysis-services-connect.md)   
* [Использование шлюза для источников данных в виртуальной сети Azure](analysis-services-vnet-gateway.md)