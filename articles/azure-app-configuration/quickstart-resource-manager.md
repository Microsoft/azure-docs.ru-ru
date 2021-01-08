---
title: Создание хранилища Конфигурации приложений Azure Service с помощью шаблона Azure Resource Manager
titleSuffix: Azure App Configuration
description: Узнайте, как создать хранилище Конфигурации приложений Azure Service с помощью шаблона Azure Resource Manager (ARM).
author: ZhijunZhao
ms.author: zhijzhao
ms.date: 10/16/2020
ms.service: azure-resource-manager
ms.topic: quickstart
ms.custom: subject-armqs
ms.openlocfilehash: 56505c95e65911cafbaaa403cd09332695439d97
ms.sourcegitcommit: e7179fa4708c3af01f9246b5c99ab87a6f0df11c
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/30/2020
ms.locfileid: "97825663"
---
# <a name="quickstart-create-an-azure-app-configuration-store-by-using-an-arm-template"></a>Краткое руководство. Создание хранилища Конфигурации приложений Azure с помощью шаблона ARM

В этом кратком руководстве описано, как выполнить следующие задачи:

- развертывание хранилища службы "Конфигурация приложений" с помощью шаблона Azure Resource Manager (шаблон ARM);
- создание пар "ключ-значение" в хранилище службы "Конфигурация приложений" с помощью шаблона ARM.
- считывание пар "ключ-значение" в хранилище службы "Конфигурация приложений" с помощью шаблона ARM.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Если среда соответствует предварительным требованиям и вы знакомы с использованием шаблонов ARM, нажмите кнопку **Развертывание в Azure**. Шаблон откроется на портале Azure.

[![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-app-configuration-store-kv%2Fazuredeploy.json)

## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="review-the-template"></a>Изучение шаблона

Шаблон, используемый в этом кратком руководстве, взят из [шаблонов быстрого запуска Azure](https://azure.microsoft.com/resources/templates/101-app-configuration-store-kv/). По нему создается новое хранилище Конфигурации приложений с двумя парами "ключ-значение". Затем с помощью функции `reference` выводятся значения двух ресурсов типа "ключ-значение". Считывание значения ключа таким образом позволяет использовать его в других местах шаблона.

В этом кратком руководстве для создания нескольких экземпляров ресурса типа "ключ-значение" используется элемент `copy`. Дополнительные сведения об элементе `copy` см. в статье [Итерация ресурсов в шаблонах Resource Manager](../azure-resource-manager/templates/copy-resources.md).

> [!IMPORTANT]
> Для этого шаблона требуется поставщик ресурсов Конфигурации приложений версии `2020-07-01-preview` или более новой. В этой версии для считывания пары "ключ-значение" используется функция `reference`. Функция `listKeyValue`, которая использовалась для чтения пары "ключ-значение" в предыдущей версии, недоступна с версии `2020-07-01-preview`.

:::code language="json" source="~/quickstart-templates/101-app-configuration-store-kv/azuredeploy.json":::

В шаблоне определено два ресурса Azure:

- [Microsoft.AppConfiguration/configurationStores](/azure/templates/microsoft.appconfiguration/2020-07-01-preview/configurationstores): создает хранилище службы "Конфигурация приложений".
- [Microsoft.AppConfiguration/configurationStores/keyValues](/azure/templates/microsoft.appconfiguration/2020-07-01-preview/configurationstores/keyvalues): создание пары "ключ-значение" в хранилище Конфигурации приложений.

> [!TIP]
> Имя ресурса `keyValues` представляет собой сочетание ключа и метки. Ключ и метка соединяются разделителем `$`. Метку использовать необязательно. В приведенном выше примере ресурс `keyValues` с именем `myKey` создает пару "ключ-значение" без метки.
>
> Кодирование с помощью знака процента, также называемое кодированием URL-адреса, позволяет использовать в ключах или метках символы, недопустимые в именах ресурсов шаблона Resource Manager. `%` не является допустимым символом, поэтому вместо него используется `~`. Чтобы правильно закодировать имя, выполните указанные ниже действия.
>
> 1. Примените кодирование URL-адреса.
> 2. Замените `~` на `~7E`:
> 3. Замените `%` на `~`:
>
> Например, чтобы создать пару "ключ-значение" с именем ключа `AppName:DbEndpoint` и именем метки `Test`, у ресурса должно быть имя `AppName~3ADbEndpoint$Test`.

> [!NOTE]
> Конфигурация приложения обеспечивает доступ к данным "ключ — значение" по [приватному каналу](concept-private-endpoint.md) из виртуальной сети. По умолчанию, если эта функция включена,все запросы на данные Конфигурации приложения через общедоступную сеть отклоняются. Так как шаблон Resource Manager выполняется за пределами виртуальной сети, доступ к данным из шаблона Resource Manager не разрешен. Чтобы разрешить доступ к данным из шаблона Resource Manager при использовании приватного канала, можно включить доступ к общедоступной сети с помощью следующей команды Azure CLI. Важно учитывать то, какое влияние оказывает включение доступа к общедоступной сети в этом сценарии на безопасность.
>
> ```azurecli-interactive
> az appconfig update -g MyResourceGroup -n MyAppConfiguration --enable-public-network true
> ```

## <a name="deploy-the-template"></a>Развертывание шаблона

Выберите следующее изображение, чтобы войти на портал Azure и открыть шаблон. Шаблон создает хранилище Конфигурации приложений с двумя парами "ключ-значение".

[![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-app-configuration-store-kv%2Fazuredeploy.json)

Кроме того, шаблон можно развернуть с помощью следующего командлета PowerShell. Пара "ключ-значение" будет указана в выходных данных консоли PowerShell.

```azurepowershell-interactive
$projectName = Read-Host -Prompt "Enter a project name that is used for generating resource names"
$location = Read-Host -Prompt "Enter the location (i.e. centralus)"
$templateUri = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-app-configuration-store-kv/azuredeploy.json"

$resourceGroupName = "${projectName}rg"

New-AzResourceGroup -Name $resourceGroupName -Location "$location"
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri $templateUri

Read-Host -Prompt "Press [ENTER] to continue ..."
```

## <a name="review-deployed-resources"></a>Просмотр развернутых ресурсов

1. Войдите на [портал Azure](https://portal.azure.com).
1. В поле поиска портала Azure введите **Конфигурация приложений**. Выберите **Конфигурация приложений** в списке.
1. Выберите созданный ресурс Конфигурации приложений.
1. В разделе **Операции** щелкните **Обозреватель конфигураций**.
1. Убедитесь, что существует две пары "ключ-значение".

## <a name="clean-up-resources"></a>Очистка ресурсов

Ставшие ненужными группу ресурсов, хранилище Конфигурации приложений и все связанные ресурсы можно удалить. Если вы планируете использовать хранилище Конфигурации приложений в дальнейшем, не удаляйте их. Если вам больше не нужно это хранилище, удалите все ресурсы, созданные в ходе работы с этим кратким руководством, выполнив следующий командлет.

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
Remove-AzResourceGroup -Name $resourceGroupName
Write-Host "Press [ENTER] to continue..."
```

## <a name="next-steps"></a>Дальнейшие действия

Сведения о создании других приложений с помощью службы "Конфигурация приложений Azure" см. в этой статье:

> [!div class="nextstepaction"]
> [Краткое руководство. Создание приложения ASP.NET Core с помощью службы "Конфигурация приложений Azure"](quickstart-aspnet-core-app.md)
