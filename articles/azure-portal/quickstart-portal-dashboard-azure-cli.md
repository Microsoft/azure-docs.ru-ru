---
title: Создание панели мониторинга портала Azure с помощью Azure CLI
description: Краткое руководство. Узнайте, как создать панель мониторинга на портале Azure с помощью Azure CLI. Панель мониторинга — это организованное и упорядоченное представление облачных ресурсов.
ms.topic: quickstart
ms.custom: devx-track-azurepowershell
ms.date: 12/4/2020
ms.openlocfilehash: ddfee1932c6887c6ca7593ca7a28c03e68e09899
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "104613249"
---
# <a name="quickstart-create-an-azure-portal-dashboard-with-azure-cli"></a>Краткое руководство. Создание панели мониторинга портала Azure с помощью Azure CLI

Панель мониторинга на портале Azure — это организованное и упорядоченное представление облачных ресурсов. В этой статье показано, как создать панель мониторинга с помощью Azure CLI.
На панели мониторинга отображаются сведения о производительности виртуальной машины, а также некоторые статические данные и ссылки.


[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Если вы используете несколько подписок Azure, выберите ту, в которой за ресурсы будут выставляться счета.
Выберите подписку с помощью команды [az account set](/cli/azure/account#az_account_set).

  ```azurecli
  az account set --subscription 00000000-0000-0000-0000-000000000000
  ```

- Создайте [группу ресурсов Azure](../azure-resource-manager/management/overview.md) с помощью команды [az group create](/cli/azure/group#az_group_create), чтобы создать группу ресурсов, или примените существующую группу ресурсов.

  ```azurecli
  az group create --name myResourceGroup --location centralus
  ```

   Группа ресурсов — это логический контейнер, в котором ресурсы Azure развертываются и администрируются как группа.

## <a name="create-a-virtual-machine"></a>Создание виртуальной машины

Создайте виртуальную машину с помощью команды [az vm create](/cli/azure/vm#az_vm_create).

```azurecli
az vm create --resource-group myResourceGroup --name SimpleWinVM --image win2016datacenter \
   --admin-username azureuser --admin-password 1StrongPassword$
```

> [!Note]
> Пароль должен быть сложным.
> Вам нужно создать новые имя пользователя и пароль,
> а не вводить данные от существующей учетной записи, например используемой для входа в Azure.
> Дополнительные сведения см. в описании требований к [имени пользователя](../virtual-machines/windows/faq.md#what-are-the-username-requirements-when-creating-a-vm) и [паролю](../virtual-machines/windows/faq.md#what-are-the-password-requirements-when-creating-a-vm).

Начнется развертывание, которое обычно занимает несколько минут.
По завершении развертывания перейдите к следующему разделу.

## <a name="download-the-dashboard-template"></a>Скачивание шаблона панели мониторинга

Так как панели мониторинга Azure являются ресурсами, то их можно представить в виде кода JSON.
Дополнительные сведения см. в статье [Структура панелей мониторинга Azure](./azure-portal-dashboards-structure.md).

Скачайте следующий файл: [portal-dashboard-template-testvm.json](https://raw.githubusercontent.com/Azure/azure-docs-powershell-samples/master/azure-portal/portal-dashboard-template-testvm.json).

Настройте скачанный шаблон, изменив следующие значения на собственные:

* `<subscriptionID>`. Ваша подписка
* `<rgName>`. Группа ресурсов, например: `myResourceGroup`.
* `<vmName>`. Имя виртуальной машины, например `SimpleWinVM`.
* `<dashboardTitle>`. Заголовок панели мониторинга, например `Simple VM Dashboard`.
* `<location>`. Регион Azure, например `centralus`.

Дополнительные сведения см. в [справочнике по шаблону мониторинга портала Microsoft](/azure/templates/microsoft.portal/dashboards).

## <a name="deploy-the-dashboard-template"></a>Развертывание шаблона панели мониторинга

Теперь шаблон можно развернуть в Azure CLI.

1. Выполните команду [az portal dashboard create](/cli/azure/ext/portal/portal/dashboard#ext_portal_az_portal_dashboard_create) для развертывания шаблона.

   ```azurecli
   az portal dashboard create --resource-group myResourceGroup --name 'Simple VM Dashboard' \
      --input-path portal-dashboard-template-testvm.json --location centralus
   ```

1. Убедитесь, что панель мониторинга успешно создана, с помощью команды [az portal dashboard show](/cli/azure/ext/portal/portal/dashboard#ext_portal_az_portal_dashboard_show).

   ```azurecli
   az portal dashboard show --resource-group myResourceGroup --name 'Simple VM Dashboard'
   ```

Чтобы просмотреть все панели мониторинга для текущей подписки, используйте [az portal dashboard list](/cli/azure/ext/portal/portal/dashboard#ext_portal_az_portal_dashboard_list).

```azurecli
az portal dashboard list
```

Вы также можете просмотреть все панели мониторинга для группы ресурсов.

```azurecli
az portal dashboard list --resource-group myResourceGroup
```

Панель мониторинга можно обновить с помощью команды [az portal dashboard update](/cli/azure/ext/portal/portal/dashboard#ext_portal_az_portal_dashboard_update).

```azurecli
az portal dashboard update --resource-group myResourceGroup --name 'Simple VM Dashboard' \
   --input-path portal-dashboard-template-testvm.json --location centralus
```

[!INCLUDE [azure-portal-review-deployed-resources](../../includes/azure-portal-review-deployed-resources.md)]

## <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы удалить виртуальную машину и связанную с ней панель мониторинга, удалите группу ресурсов, к которой они принадлежат.

> [!CAUTION]
> Следующий пример удаляет указанную группу ресурсов и все содержащиеся в ней ресурсы.
> Если в указанной группе ресурсов существуют другие ресурсы, кроме созданных для этой статьи, они также будут удалены.

```azurecli
az group delete --name myResourceGroup
```

Чтобы удалить только панель мониторинга, воспользуйтесь командой [az portal dashboard delete](/cli/azure/ext/portal/portal/dashboard#ext_portal_az_portal_dashboard_delete).

```azurecli
az portal dashboard delete --resource-group myResourceGroup --name "Simple VM Dashboard"
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о поддержке Azure CLI для панелей мониторинга см. в описании команды [az portal dashboard](/cli/azure/ext/portal/portal/dashboard).
