---
title: Краткое руководство. Создание частной конечной точки с помощью шаблона ARM
description: В этом кратком руководстве показано, как создать частную конечную точку с помощью шаблона Azure Resource Manager (шаблона ARM).
services: private-link
author: asudbring
ms.service: private-link
ms.topic: quickstart
ms.custom: subject-armqs
ms.date: 05/26/2020
ms.author: allensu
ms.openlocfilehash: e80adc16e08e676ac2daabec01a11c10d537c547
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102562951"
---
# <a name="quickstart-create-a-private-endpoint-by-using-an-arm-template"></a>Краткое руководство. Создание частной конечной точки с помощью шаблона ARM

В этом кратком руководстве показано, как создать частную конечную точку с помощью шаблона Azure Resource Manager (шаблона ARM).

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Инструкции в этом кратком руководстве можно также выполнить с помощью [портала Azure](create-private-endpoint-portal.md), [Azure PowerShell](create-private-endpoint-powershell.md) или [Azure CLI](create-private-endpoint-cli.md).

Если среда соответствует предварительным требованиям и вы знакомы с использованием шаблонов ARM, нажмите кнопку **Развертывание в Azure**. Шаблон откроется на портале Azure.

[![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-private-endpoint-sql%2Fazuredeploy.json)

## <a name="prerequisites"></a>Предварительные требования

Вам потребуется учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.

## <a name="review-the-template"></a>Изучение шаблона

Этот шаблон позволяет создать частную конечную точку для экземпляра базы данных SQL Azure.

Шаблон, используемый в этом кратком руководстве, взят из [шаблонов быстрого запуска Azure](https://azure.microsoft.com/resources/templates/101-private-endpoint-sql/).

:::code language="json" source="~/quickstart-templates/101-private-endpoint-sql/azuredeploy.json":::

В шаблоне определено несколько ресурсов Azure:

- [**Microsoft.Sql/servers**](/azure/templates/microsoft.sql/servers). Экземпляр Базы данных SQL с примером базы данных.
- [**Microsoft.Sql/servers/databases**](/azure/templates/microsoft.sql/servers/databases). Пример базы данных.
- [**Microsoft.Network/virtualNetworks**](/azure/templates/microsoft.network/virtualnetworks). Виртуальная сеть, в которой развертывается частная конечная точка.
- [**Microsoft.Network/privateEndpoints**](/azure/templates/microsoft.network/privateendpoints). Частная конечная точка для доступа к экземпляру Базы данных SQL.
- [**Microsoft.Network/privateDnsZones**](/azure/templates/microsoft.network/privatednszones). Зона, используемая для разрешения IP-адреса частной конечной точки.
- [**Microsoft.Network/privateDnsZones/virtualNetworkLinks**](/azure/templates/microsoft.network/privatednszones/virtualnetworklinks).
- [**Microsoft.Network/privateEndpoints/privateDnsZoneGroups**](/azure/templates/microsoft.network/privateendpoints/privateDnsZoneGroups). Группа зон, используемая для сопоставления частной конечной точки с частной зоной DNS.
- [**Microsoft.Network/publicIpAddresses**](/azure/templates/microsoft.network/publicIpAddresses). Общедоступный IP-адрес для доступа к виртуальной машине.
- [**Microsoft.Network/networkInterfaces**](/azure/templates/microsoft.network/networkinterfaces). Сетевой интерфейс для виртуальной машины.
- [**Microsoft.Compute/virtualMachines**](/azure/templates/microsoft.compute/virtualmachines): Виртуальная машина, используемая для проверки частного подключения к экземпляру Базы данных SQL через частную конечную точку.

## <a name="deploy-the-template"></a>Развертывание шаблона

Чтобы развернуть шаблон ARM в Azure, сделайте следующее:

1. Выберите элемент **Развертывание в Azure**, чтобы войти в Azure и открыть шаблон. Этот шаблон позволяет создать частную конечную точку, экземпляр Базы данных SQL, сетевую инфраструктуру и виртуальную машину для проверки.

   [![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-private-endpoint-sql%2Fazuredeploy.json)

2. Выберите или создайте группу ресурсов.
3. Введите учетную запись и пароль администратора SQL.
4. Введите имя и пароль администратора для виртуальной машины.
5. Ознакомьтесь с условиями. Если вы согласны с ними, выберите **Я принимаю указанные выше условия** > **Покупка**. Для развертывания может потребоваться 20 минут или более.

## <a name="validate-the-deployment"></a>Проверка развертывания

> [!NOTE]
> Шаблон ARM позволяет создать уникальное имя для ресурса виртуальной машины myVm<b>{uniqueid}</b> и для ресурса Базы данных SQL sqlserver<b>{uniqueid}</b>. Подставьте созданное значение вместо **{uniqueid}** .

### <a name="connect-to-a-vm-from-the-internet"></a>Подключение к виртуальной машине из Интернета

Подключитесь к виртуальной машине _myVm{uniqueid}_ из Интернета, выполнив следующие действия:

1. В строке поиска на портале введите _myVm{uniqueid}_ .

2. Выберите **Подключиться**. Откроется страница **Подключение к виртуальной машине**.

3. Щелкните **Скачать RDP-файл**. Azure создаст и скачает на ваш компьютер файл протокола удаленного рабочего стола (_RDP_).

4. Откройте загруженный RDP-файл.

   а. При появлении запроса выберите **Подключиться**.

   b. Введите имя пользователя и пароль, которые вы указали при создании виртуальной машины.

      > [!NOTE]
      > Возможно, потребуется выбрать элементы **Дополнительные варианты** > **Использовать другую учетную запись**, чтобы указать учетные данные, введенные при создании виртуальной машины.

5. Щелкните **ОК**.

6. При входе в систему может появиться предупреждение о сертификате. В таком случае выберите **Да** или **Продолжить**.

7. Когда откроется рабочий стол виртуальной машины, сверните его, чтобы вернуться на локальный рабочий стол.

### <a name="access-the-sql-database-server-privately-from-the-vm"></a>Частный доступ к серверу Базы данных SQL с виртуальной машины

Далее описано, как подключиться к серверу Базы данных SQL с виртуальной машины с помощью частной конечной точки.

1.  Откройте PowerShell на удаленном рабочем столе виртуальной машины _myVm{uniqueid}_ .
2.  Введите следующую команду: nslookup sqlserver{uniqueid}.database.windows.net. 
    Должно появиться сообщение следующего вида:

    ```
      Server:  UnKnown
      Address:  168.63.129.16
      Non-authoritative answer:
      Name:    sqlserver.privatelink.database.windows.net
      Address:  10.0.0.5
      Aliases:  sqlserver.database.windows.net
    ```

3.  Установите SQL Server Management Studio.
4.  В окне  **Подключение к серверу** введите или выберите следующую информацию:
    - **Тип сервера**: Выберите **Ядро СУБД**.
    - **Имя сервера**: Выберите **sqlserver{uniqueid}.database.windows.net**.
    - **Имя пользователя**. Введите имя пользователя, указанное во время создания.
    - **Пароль**. Введите пароль, указанный во время создания.
    - **Запомнить пароль**: Выберите  **Да**.

5.  Выберите **Подключиться**.
6.  В меню слева перейдите к разделу **Базы данных**.
7.  При желании вы можете создать или запросить сведения из _sample-db_.
8.  Закройте подключение удаленного рабочего стола к _myVm{uniqueid}_ .

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вам уже не нужны ресурсы, созданные вместе с частной конечной точкой, удалите эту группу ресурсов. При этом будут удалены частная конечная точка и все связанные с ней ресурсы.

Чтобы удалить группу ресурсов, вызовите командлет `Remove-AzResourceGroup`:

```azurepowershell-interactive
Remove-AzResourceGroup -Name <your resource group name>
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о службах, поддерживающих частную конечную точку, см. в следующей статье:
> [!div class="nextstepaction"]
> [Доступность Приватного канала](private-link-overview.md#availability)
