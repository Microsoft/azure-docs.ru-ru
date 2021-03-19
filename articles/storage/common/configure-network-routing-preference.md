---
title: Настройка предпочтения маршрутизации для сети
titleSuffix: Azure Storage
description: Настройте параметры сетевой маршрутизации для учетной записи хранения Azure, чтобы указать, как сетевой трафик направляется в вашу учетную запись от клиентов через Интернет.
services: storage
author: normesta
ms.service: storage
ms.topic: how-to
ms.date: 03/17/2021
ms.author: normesta
ms.reviewer: santoshc
ms.subservice: common
ms.openlocfilehash: 0738f7e427c2ff094c9b6df7539ba67dff80d095
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104589860"
---
# <a name="configure-network-routing-preference-for-azure-storage"></a>Настройка параметров сетевой маршрутизации для службы хранилища Azure

В этой статье описывается, как настроить параметры сетевой маршрутизации и конечные точки для конкретных маршрутов для вашей учетной записи хранения. 

Параметр сетевая маршрутизация указывает, как сетевой трафик направляется в вашу учетную запись от клиентов через Интернет. Конечные точки для конкретных маршрутов — это новые конечные точки, создаваемые службой хранилища Azure для вашей учетной записи хранения. Эти конечные точки направляют трафик по нужному пути, не меняя настройки маршрутизации по умолчанию. Дополнительные сведения см. в статье [Параметры сетевой маршрутизации для службы хранилища Azure](network-routing-preference.md).

## <a name="configure-the-routing-preference-for-the-default-public-endpoint"></a>Настройка параметров маршрутизации для общедоступной конечной точки по умолчанию

По умолчанию параметр маршрутизации для общедоступной конечной точки учетной записи хранения имеет значение Глобальная сеть Майкрософт. Вы можете выбрать для общедоступной конечной точки учетной записи хранения один из двух предпочтительных вариантов маршрутизации по умолчанию: через глобальную сеть Майкрософт или через Интернет. Дополнительные сведения о различиях между этими двумя типами маршрутизации см. в статье [Параметры сетевой маршрутизации для службы хранилища Azure](network-routing-preference.md). 

### <a name="portal"></a>[Портал](#tab/azure-portal)

Чтобы изменить параметры маршрутизации для маршрутизации через Интернет, выполните следующие действия.

1. Войдите на [портал Azure](https://portal.azure.com).

2. Перейдите к своей учетной записи хранения на портале.

3. В разделе **Параметры** выберите **сеть**.

    > [!div class="mx-imgBorder"]
    > ![Меню "сеть"](./media/configure-network-routing-preference/networking-option.png)

4.  На вкладке **брандмауэры и виртуальные сети** в разделе **сетевая маршрутизация** измените параметр параметры **маршрутизации** на **маршрутизацию через Интернет**.

5.  Выберите команду **Сохранить**.

    > [!div class="mx-imgBorder"]
    > ![параметр «маршрутизация через Интернет»](./media/configure-network-routing-preference/internet-routing-option.png)

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Чтобы выполнить проверку подлинности, войдите в подписку Azure с помощью команды `Connect-AzAccount` и следуйте инструкциям на экране.

   ```powershell
   Connect-AzAccount
   ```

2. Если удостоверение связано с несколькими подписками, установите активную подписку в качестве подписки учетной записи хранения, в которой будет размещен статический веб-сайт.

   ```powershell
   $context = Get-AzSubscription -SubscriptionId <subscription-id>
   Set-AzContext $context
   ```

   Замените значение заполнителя `<subscription-id>` идентификатором своей подписки.

3. Чтобы изменить параметры маршрутизации для маршрутизации через Интернет, используйте команду [Set-азсторажеаккаунт](/powershell/module/az.storage/set-azstorageaccount) и задайте `--routing-choice` для параметра значение `InternetRouting` .

   ```powershell
   Set-AzStorageAccount -ResourceGroupName <resource-group-name> `
    -AccountName <storage-account-name> `
    -RoutingChoice InternetRouting
   ```

   Замените `<resource-group-name>` значение заполнителя именем группы ресурсов, содержащей учетную запись хранения.

   Замените `<storage-account-name>` значение заполнителя именем учетной записи хранения.

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

1. Войдите в подписку Azure.

   - Чтобы запустить Azure Cloud Shell, войдите в [портал Azure](https://portal.azure.com).

   - Чтобы войти в локальную установку интерфейса командной строки, выполните команду [AZ login](/cli/azure/reference-index#az-login) :

     ```azurecli
     az login
     ```
2. Если удостоверение связано с несколькими подписками, установите активную подписку в качестве подписки учетной записи хранения, в которой будет размещен статический веб-сайт.

   ```azurecli
   az account set --subscription <subscription-id>
   ```

   Замените значение заполнителя `<subscription-id>` идентификатором своей подписки.

3. Чтобы изменить параметры маршрутизации для маршрутизации через Интернет, используйте команду [AZ Storage Account Update](/cli/azure/storage/account#az_storage_account_update) и задайте `--routing-choice` для параметра значение `InternetRouting` .

   ```azurecli
   az storage account update --name <storage-account-name> --routing-choice InternetRouting
   ```

   Замените значение заполнителя `<storage-account-name>` именем вашей учетной записи хранения.

---

## <a name="configure-a-route-specific-endpoint"></a>Настройка конечной точки для конкретного маршрута

Можно также настроить конечную точку конкретного маршрута. Например, можно задать параметры маршрутизации конечной точки по умолчанию для *маршрутизации через Интернет*, а затем опубликовать конечную точку конкретного маршрута, которая разрешает маршрутизацию трафика между клиентами в Интернете и учетной записью хранения через глобальную сеть Майкрософт.

Этот параметр влияет только на конечную точку конкретного маршрута. Этот параметр не влияет на параметры маршрутизации по умолчанию.  

### <a name="portal"></a>[Портал](#tab/azure-portal)

1.  Перейдите к своей учетной записи хранения на портале.

2.  В разделе **Параметры** выберите **сеть**.

3.  На вкладке **брандмауэры и виртуальные сети** в разделе **Публикация конечных точек, связанных с маршрутами** выберите параметры маршрутизации конечной точки, зависящей от маршрута, и нажмите кнопку **сохранить**.

    На следующем рисунке показан выбранный параметр **сетевая маршрутизация (Майкрософт** ).

    > [!div class="mx-imgBorder"]
    > ![Параметр сетевой маршрутизации (Майкрософт)](./media/configure-network-routing-preference/microsoft-network-routing-option.png)

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Чтобы настроить конечную точку для конкретного маршрута, используйте команду [Set-азсторажеаккаунт](/powershell/module/az.storage/set-azstorageaccount) . 

   - Чтобы создать конечную точку для конкретного маршрута, использующую параметры сетевой маршрутизации (Майкрософт), задайте `-PublishMicrosoftEndpoint` для параметра значение `true` . 

   - Чтобы создать конечную точку для конкретного маршрута, использующую параметры маршрутизации в Интернете, присвойте `-PublishInternetEndpointTo` параметру значение `true` .  

   В следующем примере создается конечная точка конкретного маршрута, использующая параметры сетевой маршрутизации (Майкрософт).

   ```powershell
   Set-AzStorageAccount -ResourceGroupName <resource-group-name> `
    -AccountName <storage-account-name> `
    -PublishMicrosoftEndpoint $true
   ```

   Замените `<resource-group-name>` значение заполнителя именем группы ресурсов, содержащей учетную запись хранения.

   Замените `<storage-account-name>` значение заполнителя именем учетной записи хранения.

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

1. Чтобы настроить конечную точку для конкретного маршрута, используйте команду [AZ Storage Account Update](/azure/storage/account#az-storage-account-update) . 

   - Чтобы создать конечную точку для конкретного маршрута, использующую параметры сетевой маршрутизации (Майкрософт), задайте `--publish-microsoft-endpoints` для параметра значение `true` . 

   - Чтобы создать конечную точку для конкретного маршрута, использующую параметры маршрутизации в Интернете, присвойте `--publish-internet-endpoints` параметру значение `true` .  

   В следующем примере создается конечная точка конкретного маршрута, использующая параметры сетевой маршрутизации (Майкрософт).

   ```azurecli
   az storage account update --name <storage-account-name> --publish-microsoft-endpoints true
   ```

   Замените `<storage-account-name>` значение заполнителя именем учетной записи хранения.

---

## <a name="find-the-endpoint-name-for-a-route-specific-endpoint"></a>Поиск имени конечной точки для конечной точки, относящейся к маршруту

Если вы настроили конечную точку для конкретного маршрута, конечную точку можно найти в свойствах учетной записи хранения.

### <a name="portal"></a>[Портал](#tab/azure-portal)

1.  В разделе **Параметры** выберите **свойства**.

    > [!div class="mx-imgBorder"]
    > ![пункт меню "Свойства"](./media/configure-network-routing-preference/properties.png)

2.  Конечная точка **сетевой маршрутизации (Майкрософт** ) отображается для каждой службы, которая поддерживает параметры маршрутизации. На этом рисунке показана конечная точка для BLOB-объектов и файловых служб.

    > [!div class="mx-imgBorder"]
    > ![Параметр сетевой маршрутизации (Майкрософт) для конечных точек, связанных с маршрутом](./media/configure-network-routing-preference/routing-url.png)

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Чтобы вывести конечные точки на консоль, используйте `PrimaryEndpoints` свойство объекта учетной записи хранения.

   ```powershell
   Get-AzStorageAccount -ResourceGroupName <resource-group-name> -Name <storage-account-name>
   write-Output $StorageAccount.PrimaryEndpoints
   ```

   Замените `<resource-group-name>` значение заполнителя именем группы ресурсов, содержащей учетную запись хранения.

   Замените `<storage-account-name>` значение заполнителя именем учетной записи хранения.

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

1. Чтобы напечатать конечные точки в консоли, используйте свойство учетная [запись хранения AZ](/cli/azure/storage/account#az_storage_account_show) .

   ```azurecli
   az storage account show -g <resource-group-name> -n <storage-account-name>
   ```

   Замените `<resource-group-name>` значение заполнителя именем группы ресурсов, содержащей учетную запись хранения.

   Замените `<storage-account-name>` значение заполнителя именем учетной записи хранения.

---

## <a name="see-also"></a>См. также раздел

- [Параметры сетевой маршрутизации](network-routing-preference.md)
- [Настройка брандмауэров службы хранилища Azure и виртуальных сетей](storage-network-security.md)
- [Рекомендации по обеспечению безопасности для хранилища BLOB-объектов](../blobs/security-recommendations.md)
