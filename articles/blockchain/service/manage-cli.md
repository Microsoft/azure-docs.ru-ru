---
title: Управление службой Блокчейн Azure с помощью Azure CLI
description: Управление службой Блокчейн Azure с помощью Azure CLI
ms.date: 07/23/2020
ms.topic: how-to
ms.reviewer: ravastra
ms.openlocfilehash: 36b012c486c0c7d3303a81998e88f1605999c899
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87170860"
---
# <a name="manage-azure-blockchain-service-using-azure-cli"></a>Управление службой Блокчейн Azure с помощью Azure CLI

В дополнение к портал Azure можно использовать Azure CLI для управления членами блокчейн и узлами транзакций службы Блокчейн Azure.

## <a name="launch-azure-cloud-shell"></a>Запуск Azure Cloud Shell

Azure Cloud Shell — это бесплатная интерактивная оболочка, с помощью которой можно выполнять действия, описанные в этой статье. Она включает предварительно установленные общие инструменты Azure и настроена для использования с вашей учетной записью.

Чтобы открыть Cloud Shell, просто выберите **Попробовать** в правом верхнем углу блока кода. Cloud Shell можно также запустить в отдельной вкладке браузера, перейдя на страницу [https://shell.azure.com/bash](https://shell.azure.com/bash). Нажмите кнопку **Копировать**, чтобы скопировать блоки кода. Вставьте код в Cloud Shell и нажмите клавишу "ВВОД", чтобы выполнить его.

Если вы предпочитаете установить и использовать интерфейс командной строки локально, см. раздел [install Azure CLI](/cli/azure/install-azure-cli).

## <a name="prepare-your-environment"></a>Подготовка среды

1. Войдите.

    Выполните вход с помощью команды [az login](/cli/azure/reference-index#az-login), если вы используете локальную установку CLI.

    ```azurecli
    az login
    ```

    Выполните аутентификацию, следуя инструкциям в окне терминала.

1. Установите расширение Azure CLI.

    При работе со ссылками на расширения для Azure CLI необходимо сначала установить расширение.  Расширения Azure CLI предоставляют доступ к экспериментальным командам и предварительным выпускам команд, которые еще не поставлялись как часть основного CLI.  Дополнительные сведения о расширениях, включая обновление и удаление, см. в статье [Использование расширений с Azure CLI](/cli/azure/azure-cli-extensions-overview).

    Установите [расширение для службы "Блокчейн Azure"](/cli/azure/ext/blockchain/blockchain), выполнив следующую команду.

    ```azurecli-interactive
    az extension add --name blockchain
    ```

## <a name="create-blockchain-member"></a>Создание элемента блокчейн

Пример [создает элемент блокчейн](/cli/azure/ext/blockchain/blockchain/member#ext-blockchain-az-blockchain-member-create) в службе блокчейн Azure, который выполняет протокол кворума в новом консорциуме.

```azurecli
az blockchain member create \
                            --resource-group <myResourceGroup> \
                            --name <myMemberName> \
                            --location <myBlockchainLocation> \
                            --password <strongMemberAccountPassword> \
                            --protocol "Quorum" \
                            --consortium <myConsortiumName> \
                            --consortium-management-account-password <strongConsortiumManagementPassword> \
                            --sku <skuName>
```

| Параметр | Описание |
|---------|-------------|
| **resource-group** | Имя группы ресурсов, в которой создаются ресурсы службы "Блокчейн Azure". |
| **name** | Уникальное имя, идентифицирующее участника блокчейн-сети в службе "Блокчейн Azure". Это имя используется в адресе общедоступной конечной точки. Например, `myblockchainmember.blockchain.azure.com`. |
| **расположение** | Регион Azure, в котором создается участник блокчейн-сети. Например, `eastus`. Выберите расположение, наиболее близкое к пользователям или другим приложениям Azure. Компоненты могут быть недоступны в некоторых регионах. |
| **password** | Пароль для узла транзакций по умолчанию участника. Этот пароль используется для обычной проверки подлинности при подключении к общедоступной конечной точке узла транзакций участника блокчейна. Пароль должен соответствовать трем из следующих четырех требований: длина должна составлять от 12 & 72 символов, строчных символов, прописных букв, 1 числа и специальных символов, которые не являются решетками (#), percent (%), запятой (,), звездочки (*), обратной кавычки ( \` ), двойных кавычек ("), одинарной кавычки ('), тире (-) и семиколумн (;)|
| **protocol** | Протокол блокчейна. В настоящее время поддерживается протокол *Quorum*. |
| **consortium** | Имя консорциума, который создается или к которому присоединяется участник. Дополнительные сведения о консорциумах см. в разделе [Консорциум службы "Блокчейн Azure"](consortium.md). |
| **consortium-management-account-password** | Пароль учетной записи консорциума, также называется паролем учетной записи участника. Пароль учетной записи участника используется для шифрования закрытого ключа для учетной записи Ethereum, созданного для вашего участника. Учетная запись участника и ее пароль используются для управления консорциумом. |
| **sku** | Тип уровня. *Стандартный* или *Базовый*. Используйте уровень *Базовый* для разработки, тестирования и подтверждения концепций. Используйте уровень *Стандартный* для рабочих развертываний. Если вы работаете с Диспетчером данных блокчейна или отправляете большой объем частных транзакций, используйте ценовую категорию *Стандартный*. Изменение ценовой категории "Базовый" и "Стандартный" после создания элемента не поддерживается. |

## <a name="change-blockchain-member-passwords-or-firewall-rules"></a>Изменение паролей членов блокчейн или правил брандмауэра

Пример [обновляет пароль участника блокчейн](/cli/azure/ext/blockchain/blockchain/member#ext-blockchain-az-blockchain-member-update), пароль управления консорциумом W3C и правило брандмауэра.

```azurecli
az blockchain member update \
                     --resource-group <myResourceGroup> \
                     --name <myMemberName> \
                     --password <strongMemberAccountPassword> \
                     --consortium-management-account-password <strongConsortiumManagementPassword> \
                     --firewall-rules <firewallRules>
```

| Параметр | Описание |
|---------|-------------|
| **resource-group** | Имя группы ресурсов, в которой создаются ресурсы службы "Блокчейн Azure". |
| **name** | Имя, идентифицирующее участника службы Azure Блокчейн. |
| **password** | Пароль для узла транзакций по умолчанию участника. Этот пароль используется для обычной проверки подлинности при подключении к общедоступной конечной точке узла транзакций участника блокчейна. Пароль должен соответствовать трем из следующих четырех требований: длина должна составлять от 12 & 72 символов, строчных символов, прописных букв, 1 числа и специальных символов, которые не являются решетками (#), percent (%), запятой (,), звездочки (*), обратной кавычки ( \` ), двойных кавычек ("), одинарной кавычки ('), тире (-) и семиколумн (;)|
| **consortium-management-account-password** | Пароль учетной записи консорциума, также называется паролем учетной записи участника. Пароль учетной записи участника используется для шифрования закрытого ключа для учетной записи Ethereum, созданного для вашего участника. Учетная запись участника и ее пароль используются для управления консорциумом. |
| **правила брандмауэра** | Начальный и конечный IP-адрес для списка разрешений IP-адресов. |

## <a name="create-transaction-node"></a>Создание узла транзакций

[Создайте узел транзакции](/cli/azure/ext/blockchain/blockchain/transaction-node#ext-blockchain-az-blockchain-transaction-node-create) внутри существующего элемента блокчейн. Добавляя узлы транзакций, можно повысить изоляцию безопасности и распределить нагрузку. Например, у вас может быть конечная точка узла транзакции для разных клиентских приложений.

```azurecli
az blockchain transaction-node create \
                     --resource-group <myResourceGroup> \
                     --member-name <myMemberName> \
                     --password <strongTransactionNodePassword> \
                     --name <myTransactionNodeName>
```

| Параметр | Описание |
|---------|-------------|
| **resource-group** | Имя группы ресурсов, в которой создаются ресурсы службы "Блокчейн Azure". |
| **расположение** | Регион Azure члена блокчейн. |
| **имя члена** | Имя, идентифицирующее участника службы Azure Блокчейн. |
| **password** | Пароль для узла транзакции. Используйте пароль для обычной проверки подлинности при соединении с общедоступной конечной точкой узла транзакции. Пароль должен соответствовать трем из следующих четырех требований: длина должна составлять от 12 & 72 символов, строчных символов, прописных букв, 1 числа и специальных символов, которые не являются решетками (#), percent (%), запятой (,), звездочки (*), обратной кавычки ( \` ), двойных кавычек ("), одинарной кавычки ('), тире (-) и семиколумн (;)|
| **name** | Имя узла транзакций. |

## <a name="change-transaction-node-password"></a>Изменение пароля узла транзакции

Пример [обновляет пароль узла транзакции](/cli/azure/ext/blockchain/blockchain/transaction-node#ext-blockchain-az-blockchain-transaction-node-update) .

```azurecli
az blockchain transaction-node update \
                     --resource-group <myResourceGroup> \
                     --member-name <myMemberName> \
                     --password <strongTransactionNodePassword> \
                     --name <myTransactionNodeName>
```

| Параметр | Описание |
|---------|-------------|
| **resource-group** | Имя группы ресурсов, в которой существуют ресурсы службы Блокчейн Azure. |
| **имя члена** | Имя, идентифицирующее участника службы Azure Блокчейн. |
| **password** | Пароль для узла транзакции. Используйте пароль для обычной проверки подлинности при соединении с общедоступной конечной точкой узла транзакции. Пароль должен соответствовать трем из следующих четырех требований: длина должна составлять от 12 & 72 символов, строчных символов, прописных букв, 1 числа и специальных символов, которые не являются решетками (#), percent (%), запятой (,), звездочки (*), обратной кавычки ( \` ), двойных кавычек ("), одинарной кавычки ('), тире (-) и семиколумн (;)|
| **name** | Имя узла транзакций. |

## <a name="list-api-keys"></a>Список ключей API

Ключи API можно использовать для доступа к узлам аналогично имени пользователя и паролю. Существует два ключа API для поддержки смены ключей. Используйте следующую команду, чтобы получить [список ключей API](/cli/azure/ext/blockchain/blockchain/member#ext-blockchain-az-blockchain-transaction-node-list-api-key).

```azurecli
az blockchain member list-api-key \
                            --resource-group <myResourceGroup> \
                            --name <myMemberName>
```

| Параметр | Описание |
|---------|-------------|
| **resource-group** | Имя группы ресурсов, в которой существуют ресурсы службы Блокчейн Azure. |
| **name** | Имя участника блокчейн службы Azure Блокчейн |

## <a name="regenerate-api-keys"></a>Повторное создание ключей API

Используйте следующую команду, чтобы [повторно создать ключи API](/cli/azure/ext/blockchain/blockchain/member#ext-blockchain-az-blockchain-transaction-node-regenerate-api-key).

```azurecli
az blockchain member regenerate-api-key \
                            --resource-group <myResourceGroup> \
                            --name <myMemberName> \
                            [--key-name {<keyValue1>, <keyValue2>}]
```

| Параметр | Описание |
|---------|-------------|
| **resource-group** | Имя группы ресурсов, в которой существуют ресурсы службы Блокчейн Azure. |
| **name** | Имя члена блокчейн службы Azure Блокчейн. |
| **keyName** | Замените \<keyValue\> либо Key1, Key2, либо обоими. |

## <a name="delete-a-transaction-node"></a>Удаление узла транзакции

Пример [удаляет узел транзакции-члена блокчейн](/cli/azure/ext/blockchain/blockchain/transaction-node#ext-blockchain-az-blockchain-transaction-node-delete).

```azurecli
az blockchain transaction-node delete \
                     --resource-group <myResourceGroup> \
                     --member-name <myMemberName> \
                     --name <myTransactionNode>
```

| Параметр | Описание |
|---------|-------------|
| **resource-group** | Имя группы ресурсов, в которой существуют ресурсы службы Блокчейн Azure. |
| **имя члена** | Имя члена блокчейн службы Azure Блокчейн, который также включает имя удаляемого узла транзакции. |
| **name** | Имя удаляемого узла транзакции. |

## <a name="delete-a-blockchain-member"></a>Удаление элемента блокчейн

Пример [удаляет элемент блокчейн](/cli/azure/ext/blockchain/blockchain/member#ext-blockchain-az-blockchain-member-delete).

```azurecli
az blockchain member delete \
                     --resource-group <myResourceGroup> \
                     --name <myMemberName>

```

| Параметр | Описание |
|---------|-------------|
| **resource-group** | Имя группы ресурсов, в которой существуют ресурсы службы Блокчейн Azure. |
| **name** | Имя удаляемого члена Azure Блокчейн Service блокчейн. |

## <a name="azure-active-directory"></a>Azure Active Directory

### <a name="grant-access-for-azure-ad-user"></a>Предоставление доступа для пользователя Azure AD

```azurecli
az role assignment create \
                            --role <role> \
                            --assignee <assignee> \
                            --scope /subscriptions/<subId>/resourceGroups/<groupName>/providers/Microsoft.Blockchain/blockchainMembers/<myMemberName>
```

| Параметр | Описание |
|---------|-------------|
| **role** | Имя роли Azure AD. |
| **assignee** | Идентификатор пользователя Azure AD. Например: `user@contoso.com` |
| **область** | Область назначения роли. Может быть либо блокчейн элементом, либо узлом транзакции. |

**Пример**.

Предоставьте доступ к узлу для пользователя Azure AD **члену** блокчейн:

```azurecli
az role assignment create \
                            --role 'myRole' \
                            --assignee user@contoso.com \
                            --scope /subscriptions/mySubscriptionId/resourceGroups/contosoResourceGroup/providers/Microsoft.Blockchain/blockchainMembers/contosoMember1
```

**Пример**.

Предоставьте доступ к узлу для пользователя Azure AD на **узел транзакции** блокчейн:

```azurecli
az role assignment create \
                            --role 'MyRole' \
                            --assignee user@contoso.com \
                            --scope /subscriptions/mySubscriptionId/resourceGroups/contosoResourceGroup/providers/Microsoft.Blockchain/blockchainMembers/contosoMember1/transactionNodes/contosoTransactionNode1
```

### <a name="grant-node-access-for-azure-ad-group-or-application-role"></a>Предоставление доступа к узлу для роли приложения или группы Azure AD

```azurecli
az role assignment create \
                            --role <role> \
                            --assignee-object-id <assignee_object_id>
```

| Параметр | Описание |
|---------|-------------|
| **role** | Имя роли Azure AD. |
| **assigns-Object-ID** | Идентификатор группы Azure AD или идентификатор приложения. |
| **область** | Область назначения роли. Может быть либо блокчейн элементом, либо узлом транзакции. |

**Пример**.

Предоставление доступа к узлу для **роли приложения**

```azurecli
az role assignment create \
                            --role 'myRole' \
                            --assignee-object-id 22222222-2222-2222-2222-222222222222 \
                            --scope /subscriptions/mySubscriptionId/resourceGroups/contosoResourceGroup/providers/Microsoft.Blockchain/blockchainMembers/contosoMember1
```

### <a name="remove-azure-ad-node-access"></a>Удаление доступа к узлу Azure AD

```azurecli
az role assignment delete \
                            --role <myRole> \
                            --assignee <assignee> \
                            --scope /subscriptions/mySubscriptionId/resourceGroups/<myResourceGroup>/providers/Microsoft.Blockchain/blockchainMembers/<myMemberName>/transactionNodes/<myTransactionNode>
```

| Параметр | Описание |
|---------|-------------|
| **role** | Имя роли Azure AD. |
| **assignee** | Идентификатор пользователя Azure AD. Например: `user@contoso.com` |
| **область** | Область назначения роли. Может быть либо блокчейн элементом, либо узлом транзакции. |

## <a name="next-steps"></a>Дальнейшие действия

Узнайте, как [настроить узлы транзакций службы Блокчейн Azure с помощью портал Azure](configure-transaction-nodes.md).
