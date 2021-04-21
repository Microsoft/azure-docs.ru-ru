---
title: Краткое руководство. Создание концентратора уведомлений Azure с помощью Azure CLI | Документация Майкрософт
description: В этом руководстве показано, как создать концентратор уведомлений Azure с помощью Azure CLI.
services: notification-hubs
author: dbradish-microsoft
manager: barbkess
editor: sethmanheim
ms.service: notification-hubs
ms.devlang: azurecli
ms.workload: mobile
ms.topic: quickstart
ms.date: 05/27/2020
ms.author: dbradish
ms.reviewer: thsomasu
ms.lastreviewed: 03/18/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: d8400eb051c09fac4cb88863ad2fac12d2ca0a1b
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107789891"
---
# <a name="quickstart-create-an-azure-notification-hub-using-the-azure-cli"></a>Создание концентратора уведомлений Azure с помощью Azure CLI

Центры уведомлений Azure обеспечивают простой в использовании и масштабируемый механизм отправки push-уведомлений, который позволяет отправлять уведомления на любую платформу (iOS, Android, Windows, Kindle, Baidu и т. д.) из любой серверной части (облачной или локальной). Дополнительные сведения о службе см. в статье [Что такое Центры уведомлений Azure?](notification-hubs-push-notification-overview.md).

В этом кратком руководстве показано, как создать концентратор уведомлений с помощью Azure CLI. В первом разделе приводятся шаги по созданию пространства имен Центров уведомлений. Во втором разделе приведены пошаговые инструкции по созданию концентратора уведомлений в существующем пространстве имен. Вы также узнаете, как создать пользовательскую политику доступа.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

> [!IMPORTANT]
> Для работы с концентраторами уведомлений требуется Azure CLI версии не ниже 2.0.67. Выполните команду [az version](/cli/azure/reference-index#az_version), чтобы узнать установленную версию и зависимые библиотеки. Чтобы обновиться до последней версии, выполните команду [az upgrade](/cli/azure/reference-index#az_upgrade).

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Центры уведомлений Microsoft Azure, как и все ресурсы Azure, должны быть развернуты в группе ресурсов.  Группы ресурсов позволяют организовать соответствующие ресурсы Azure и управлять ими.  Дополнительные сведения о группе ресурсов см. в статье [Azure Resource Manager](../azure-resource-manager/management/overview.md).

Для работы с этим кратким руководством создайте группу ресурсов с именем **spnhubrg** в регионе **eastus** с помощью следующей команды [az group create](/cli/azure/group#az_group_create).

```azurecli
az group create --name spnhubrg --location eastus
```

## <a name="create-a-notification-hubs-namespace"></a>Создание пространства имен Центров уведомлений

1. Создайте пространство имен для концентраторов уведомлений.

   Пространство имен содержит один или несколько концентраторов, и имя должно быть уникальным для всех подписок Azure и иметь длину не менее шести символов. Чтобы проверить доступность имени, выполните команду [az notification-hub namespace check-availability](/cli/azure/ext/notification-hub/notification-hub/namespace#ext-notification-hub-az-notification-hub-namespace-check-availability).

   ```azurecli
   az notification-hub namespace check-availability --name spnhubns
   ```

   Azure CLI отвечает на запрос о доступности, отображая следующие выходные данные консоли:

   ```shell
   {
   "id": "/subscriptions/yourSubscriptionID/providers/Microsoft.NotificationHubs/checkNamespaceAvailability",
   "isAvailable": true,
   "location": null,
   "name": "spnhubns",
   "properties": false,
   "sku": null,
   "tags": null,
   "type": "Microsoft.NotificationHubs/namespaces/checkNamespaceAvailability"
   }
   ```

   Обратите внимание на вторую строку в ответе Azure CLI (`"isAvailable": true`). Эта строка будет иметь значение `false`, если необходимое имя, указанное для пространства имен, недоступно. Убедившись в доступности имени, выполните команду [az notification-hub namespace create](/cli/azure/ext/notification-hub/notification-hub/namespace#ext-notification-hub-az-notification-hub-namespace-create), чтобы создать пространство имен.  

   ```azurecli
   az notification-hub namespace create --resource-group spnhubrg --name spnhubns  --location eastus --sku Free
   ```

   Если элемент `--name`, предоставленный для команды `az notification-hub namespace create`, недоступен или не соответствует [правилам именования и ограничениям для ресурсов Azure](../azure-resource-manager/management/resource-name-rules.md), Azure CLI отобразит следующие выходные данные консоли:

   ```shell
   #the name is not available
   The specified name is not available. For more information visit https://aka.ms/eventhubsarmexceptions.

   #the name is invalid
   The specified service namespace is invalid.
   ```

   Если первое имя, которое вы использовали, оказалось неподходящим, выберите другое имя для нового пространства имен и снова выполните команду `az notification-hub namespace create`.

   > [!NOTE]
   > На этом и последующих шагах вам необходимо заменять значение параметра `--namespace` в каждой команде Azure CLI, которую вы копируете из этого краткого руководства.

2. Получите список пространств имен.

   Чтобы просмотреть сведения о новом пространстве имен, выполните команду [az notification-hub namespace list](/cli/azure/ext/notification-hub/notification-hub/namespace#ext-notification-hub-az-notification-hub-namespace-list). Параметр `--resource-group` является необязательным, если требуется просмотреть все пространства имен для подписки.

   ```azurecli
   az notification-hub namespace list --resource-group spnhubrg
   ```

## <a name="create-notification-hubs"></a>Создание концентраторов уведомлений

1. Создайте первый концентратор уведомлений.

   Теперь можно создать один или несколько концентраторов уведомлений в новом пространстве имен. Чтобы создать концентратор уведомлений, выполните команду [az notification-hub create](/cli/azure/ext/notification-hub/notification-hub#ext-notification-hub-az-notification-hub-create).

   ```azurecli
   az notification-hub create --resource-group spnhubrg --namespace-name spnhubns --name spfcmtutorial1nhub --location eastus --sku Free
   ```

2. Создайте второй концентратор уведомлений.

   Вы можете создать несколько концентраторов уведомлений в одном пространстве имен. Чтобы создать второй концентратор уведомлений в том же пространстве имен, выполните команду `az notification-hub create` еще раз с другим именем концентратора.

   ```azurecli
   az notification-hub create --resource-group spnhubrg --namespace-name spnhubns --name mysecondnhub --location eastus --sku Free
   ```

3. Получите список концентраторов уведомлений.

   С каждой выполненной командой Azure CLI возвращает либо успешное сообщение, либо сообщение об ошибке. Тем не менее, вы всегда можете запросить список концентраторов уведомлений. Для этой цели была разработана команда [az notification-hub list](/cli/azure/ext/notification-hub/notification-hub#ext-notification-hub-az-notification-hub-list).

   ```azurecli
   az notification-hub list --resource-group spnhubrg --namespace-name spnhubns --output table
   ```

## <a name="work-with-access-policies"></a>Работа с политиками доступа

1. Центры уведомлений Azure используют [безопасность на базе подписанного URL-адреса (SAS)](./notification-hubs-push-notification-security.md) с помощью политик доступа. При создании концентратора уведомлений автоматически создаются две политики. Строки подключения из этих политик необходимы для настройки push-уведомлений. Команда [az notification-hub authorization-rule list](/cli/azure/ext/notification-hub/notification-hub/authorization-rule#ext-notification-hub-az-notification-hub-authorization-rule-list) предоставляет список имен политик и их соответствующих групп ресурсов.

   ```azurecli
   az notification-hub authorization-rule list --resource-group spnhubrg --namespace-name spnhubns --notification-hub-name spfcmtutorial1nhub --output table
   ```

   > [!IMPORTANT]
   > Не используйте в приложении политику _DefaultFullSharedAccessSignature_. Эту политику можно использовать только в серверной части. Используйте только политики доступа `Listen` в клиентском приложении.

2. Если вы хотите создать дополнительные правила авторизации с информативными именами, можно создать и настроить собственную политику доступа с помощью команды [az notification-hub authorization-rule create](/cli/azure/ext/notification-hub/notification-hub/authorization-rule#ext-notification-hub-az-notification-hub-authorization-rule-create). Параметр `--rights` представляет собой список разрешений с разделителями-пробелами, которые вы хотите назначить.

   ```azurecli
   az notification-hub authorization-rule create --resource-group spnhubrg --namespace-name spnhubns --notification-hub-name spfcmtutorial1nhub --name spnhub1key --rights Listen Manage Send
   ```

3. Для каждой политики доступа определены два набора ключей и строк подключения. Они потребуются позже, чтобы [настроить концентратор уведомлений](./configure-notification-hub-portal-pns-settings.md). Чтобы получить список ключей и строк подключения для политики доступа Центров уведомлений, выполните команду [az notification-hub authorization-rule list-keys](/cli/azure/ext/notification-hub/notification-hub/authorization-rule#ext-notification-hub-az-notification-hub-authorization-rule-list-keys).

   ```azurecli
   # query the keys and connection strings for DefaultListenSharedAccessSignature
   az notification-hub authorization-rule list-keys --resource-group spnhubrg --namespace-name spnhubns --notification-hub-name spfcmtutorial1nhub --name DefaultListenSharedAccessSignature --output table
   ```

   ```azurecli
   # query the keys and connection strings for a custom policy
   az notification-hub authorization-rule list-keys --resource-group spnhubrg --namespace-name spnhubns --notification-hub-name spfcmtutorial1nhub --name spnhub1key --output table
   ```

   > [!NOTE]
   > Для [пространства имен Центров уведомлений](/cli/azure/ext/notification-hub/notification-hub/namespace/authorization-rule#ext-notification-hub-az-notification-hub-namespace-authorization-rule-list-keys) и самого [концентратора уведомлений](/cli/azure/ext/notification-hub/notification-hub/authorization-rule#ext-notification-hub-az-notification-hub-authorization-rule-list-keys) действуют разные политики доступа. Убедитесь, что при запросе ключей и строк подключения используется правильная ссылка на Azure CLI.

## <a name="clean-up-resources"></a>Очистка ресурсов

Вы можете удалить ставшие ненужными группу ресурсов и все связанные с ней ресурсы, выполнив команду [az group delete](/cli/azure/group):

```azurecli
az group delete --name spnhubrg
```

## <a name="next-steps"></a>Дальнейшие действия

* В этом кратком руководстве вы создали центр уведомлений. Сведения о настройке концентратора с помощью параметров системы отправки уведомлений платформы (PNS) см. в статье [Краткое руководство. Настройка push-уведомлений в центре уведомлений](configure-notification-hub-portal-pns-settings.md)

* Узнайте о широких возможностях управления концентраторами уведомлений с помощью Azure CLI:

  [Полный справочный список по Центрам уведомлений](/cli/azure/ext/notification-hub/notification-hub)

  [Справочный список по пространствам имен Центров уведомлений](/cli/azure/ext/notification-hub/notification-hub/namespace)

  [Справочный список по правилам авторизации Центров уведомлений](/cli/azure/ext/notification-hub/notification-hub/authorization-rule)

  [Справочный список по учетным данным Центров уведомлений](/cli/azure/ext/notification-hub/notification-hub/credential)