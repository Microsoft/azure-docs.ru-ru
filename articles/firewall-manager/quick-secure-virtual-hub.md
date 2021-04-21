---
title: Краткое руководство. Защита виртуального концентратора с помощью Диспетчера брандмауэра Azure — шаблон Resource Manager
description: В этом кратком руководстве показано, как защитить виртуальный концентратор с помощью Диспетчера брандмауэра Azure.
services: firewall-manager
author: vhorne
ms.author: victorh
ms.date: 08/28/2020
ms.topic: quickstart
ms.service: firewall-manager
ms.custom:
- subject-armqs
- mode-arm
ms.openlocfilehash: 88b961f269c89a1e19c0fd4419ffdc33706423de
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/16/2021
ms.locfileid: "107529656"
---
# <a name="quickstart-secure-your-virtual-hub-using-azure-firewall-manager---arm-template"></a>Краткое руководство. Защита виртуального концентратора с помощью Диспетчера брандмауэра Azure — шаблон ARM

В этом кратком руководстве показано, как с помощью шаблона Azure Resource Manager (шаблона ARM) защитить виртуальный концентратор с использованием Диспетчера брандмауэра Azure. Развернутый брандмауэр имеет правило приложения, которое разрешает подключения к `www.microsoft.com`. Для тестирования брандмауэра развертываются две виртуальные машины Windows Server 2019. Для подключения к серверу рабочей нагрузки используется один сервер переходов. С сервера рабочей нагрузки можно подключаться только к `www.microsoft.com`.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Дополнительные сведения см. в статье [Общие сведения о Диспетчере брандмауэра Azure](overview.md).

Если среда соответствует предварительным требованиям и вы знакомы с использованием шаблонов ARM, нажмите кнопку **Развертывание в Azure**. Шаблон откроется на портале Azure.

[![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Ffwm-docs-qs%2Fazuredeploy.json)

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.

## <a name="review-the-template"></a>Изучение шаблона

С помощью Диспетчера брандмауэра Azure этот шаблон создает защищенный виртуальный концентратор, а также необходимые ресурсы для поддержки этого сценария.

Шаблон, используемый в этом кратком руководстве, взят из [шаблонов быстрого запуска Azure](https://azure.microsoft.com/resources/templates/fwm-docs-qs/).

:::code language="json" source="~/quickstart-templates/fwm-docs-qs/azuredeploy.json":::

В шаблоне определено несколько ресурсов Azure:

- [**Microsoft.Network/virtualWans**](/azure/templates/microsoft.network/virtualWans)
- [**Microsoft.Network/virtualHubs**](/azure/templates/microsoft.network/virtualHubs)
- [**Microsoft.Network/firewallPolicies**](/azure/templates/microsoft.network/firewallPolicies)
- [**Microsoft.Network/azureFirewalls**](/azure/templates/microsoft.network/azureFirewalls)
- [**Microsoft.Network/virtualNetworks**](/azure/templates/microsoft.network/virtualnetworks);
- [**Microsoft.Compute/virtualMachines**](/azure/templates/microsoft.compute/virtualmachines)
- [**Microsoft.Storage/storageAccounts**](/azure/templates/microsoft.storage/storageAccounts)
- [**Microsoft.Network/networkInterfaces**](/azure/templates/microsoft.network/networkinterfaces)
- [**Microsoft.Network/networkSecurityGroups;**](/azure/templates/microsoft.network/networksecuritygroups)
- [**Microsoft.Network/publicIPAddresses**](/azure/templates/microsoft.network/publicipaddresses)
- [**Microsoft.Network/routeTables**](/azure/templates/microsoft.network/routeTables)

## <a name="deploy-the-template"></a>Развертывание шаблона

Разверните шаблон ARM в Azure:

1. Выберите элемент **Развертывание в Azure**, чтобы войти в Azure и открыть шаблон. Шаблон создает Брандмауэр Azure, виртуальную глобальную сеть и виртуальный концентратор, сетевую инфраструктуру и две виртуальные машины.

   [![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Ffwm-docs-qs%2Fazuredeploy.json)

2. На портале на странице **Защищенные виртуальные концентраторы** укажите следующие значения.
   - Подписка: выберите из существующих подписок. 
   - Группа ресурсов:  выберите из существующих групп ресурсов или выберите **Создать** и нажмите кнопку **ОК**.
   - Расположение. Выберите расположение
   - Имя пользователя администратора: введите имя учетной записи пользователя с правами администратора. 
   - Пароль администратора: введите пароль администратора или ключ.

3. Выберите **Просмотр и создание**, а затем щелкните **Создать**. Для развертывания может потребоваться 10 минут или более.

## <a name="validate-the-deployment"></a>Проверка развертывания

Теперь проверьте правила брандмауэра, чтобы убедиться в том, что они работают должным образом.

1. На портале Azure просмотрите параметры сети для виртуальной машины **Workload-Srv** и запишите частный IP-адрес.
2. Подключите удаленный рабочий стол к виртуальной машине **Jump-Srv** и выполните вход. Откройте из нее подключение удаленного рабочего стола к частному IP-адресу **Workload-Srv**.

3. Откройте браузер Internet Explorer и перейдите на сайт [https://www.google.com]\(`www.microsoft.com` ).
4. Если отобразятся системы оповещения безопасности Internet Explorer, выберите **ОК** > **Закрыть**.

   Откроется домашняя страница Майкрософт.

5. Перейдите по адресу `www.google.com`.

   Брандмауэр должен вас заблокировать.

Итак, теперь вы убедились в том, что правила брандмауэра работают:

- Вы можете перейти только к одному разрешенному имени FQDN.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вам уже не нужны ресурсы, созданные с помощью брандмауэра, удалите группу ресурсов. При этом будут удалены брандмауэры и все связанные с ним ресурсы.

Чтобы удалить группу ресурсов, вызовите командлет `Remove-AzResourceGroup`:

```azurepowershell-interactive
Remove-AzResourceGroup -Name "<your resource group name>"
```

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Общие сведения о поставщиках партнера по безопасности](trusted-security-partners.md)
