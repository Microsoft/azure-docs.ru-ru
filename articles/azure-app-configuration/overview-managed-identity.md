---
title: Настройка управляемых удостоверений с помощью конфигурации приложений Azure
description: Узнайте, как управляемые удостоверения работают в конфигурации приложений Azure и как настроить управляемое удостоверение.
author: barbkess
ms.topic: article
ms.date: 02/25/2020
ms.author: barbkess
ms.reviewer: lcozzens
ms.service: azure-app-configuration
ms.openlocfilehash: 5424b776b977d8a6939bbb28c5d4c9c428cac444
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102179618"
---
# <a name="how-to-use-managed-identities-for-azure-app-configuration"></a>Использование управляемых удостоверений для конфигурации приложений Azure

В этом разделе показано, как создать управляемое удостоверение для конфигурации приложения Azure. Управляемое удостоверение из Azure Active Directory (AAD) позволяет конфигурации приложений Azure легко получать доступ к другим ресурсам, защищенным AAD, таким как Azure Key Vault. Удостоверение управляется платформой Azure. Для этого не требуется подготавливать или вращать секретные данные. Дополнительные сведения об управляемых удостоверениях в Azure Active Directory см. в статье [Что такое управляемые удостоверения для ресурсов Azure?](../active-directory/managed-identities-azure-resources/overview.md).

Приложению можно предоставить два типа удостоверений:

- **Назначенное системой удостоверение** связано с хранилищем конфигураций. Она удаляется, если хранилище конфигураций удалено. Хранилище конфигураций может иметь только одно назначенное системой удостоверение.
- **Назначаемое пользователем удостоверение** — это автономный ресурс Azure, который можно назначить хранилищу конфигураций. Хранилище конфигураций может иметь несколько назначенных пользователю удостоверений.

## <a name="adding-a-system-assigned-identity"></a>Добавление назначаемого системой удостоверения

Для создания хранилища конфигурации приложений с назначенным системой удостоверением требуется задать дополнительное свойство в хранилище.

### <a name="using-the-azure-cli"></a>Использование Azure CLI

Чтобы настроить управляемое удостоверение с помощью Azure CLI, используйте команду [AZ appconfig Identity Assign] в существующем хранилище конфигураций. Примеры из этого раздела можно выполнять тремя способами:

- использовать [Azure Cloud Shell](../cloud-shell/overview.md) с портала Azure;
- использовать внедренный компонент Azure Cloud Shell с помощью кнопки "Попробовать", расположенной в правом верхнем углу каждого блока кода ниже.
- [Установите последнюю версию Azure CLI](/cli/azure/install-azure-cli) (2,1 или более позднюю), если вы предпочитаете использовать локальную консоль CLI.

Следующие шаги помогут вам создать хранилище конфигураций приложений и назначить ему удостоверение с помощью интерфейса командной строки:

1. Если вы используете Azure CLI в локальной консоли, сначала выполните вход в Azure с помощью команды [az login]. Используйте учетную запись, связанную с подпиской Azure:

    ```azurecli-interactive
    az login
    ```

1. Создание хранилища конфигурации приложений с помощью интерфейса командной строки. Дополнительные примеры использования интерфейса командной строки с конфигурацией приложений Azure см. в разделе [примеры настройки CLI для приложений](scripts/cli-create-service.md).

    ```azurecli-interactive
    az group create --name myResourceGroup --location eastus
    az appconfig create --name myTestAppConfigStore --location eastus --resource-group myResourceGroup --sku Free
    ```

1. Выполните команду [AZ appconfig Identity Assign] , чтобы создать назначенное системой удостоверение для этого хранилища конфигураций:

    ```azurecli-interactive
    az appconfig identity assign --name myTestAppConfigStore --resource-group myResourceGroup
    ```

## <a name="adding-a-user-assigned-identity"></a>Добавление удостоверения, назначенного пользователем

Для создания хранилища конфигурации приложений с назначенным пользователем удостоверением необходимо создать удостоверение, а затем назначить его идентификатор ресурса в хранилище.

### <a name="using-the-azure-cli"></a>Использование Azure CLI

Чтобы настроить управляемое удостоверение с помощью Azure CLI, используйте команду [AZ appconfig Identity Assign] в существующем хранилище конфигураций. Примеры из этого раздела можно выполнять тремя способами:

- использовать [Azure Cloud Shell](../cloud-shell/overview.md) с портала Azure;
- использовать внедренный компонент Azure Cloud Shell с помощью кнопки "Попробовать", расположенной в правом верхнем углу каждого блока кода ниже.
- [установить последнюю версию Azure CLI](/cli/azure/install-azure-cli) (2.0.31 или выше), если вы предпочитаете использовать локальную консоль CLI.

Ниже описано, как создать пользовательское удостоверение и хранилище конфигурации приложения, а затем назначить удостоверение хранилищу с помощью интерфейса командной строки:

1. Если вы используете Azure CLI в локальной консоли, сначала выполните вход в Azure с помощью команды [az login]. Используйте учетную запись, связанную с подпиской Azure:

    ```azurecli-interactive
    az login
    ```

1. Создание хранилища конфигурации приложений с помощью интерфейса командной строки. Дополнительные примеры использования интерфейса командной строки с конфигурацией приложений Azure см. в разделе [примеры настройки CLI для приложений](scripts/cli-create-service.md).

    ```azurecli-interactive
    az group create --name myResourceGroup --location eastus
    az appconfig create --name myTestAppConfigStore --location eastus --resource-group myResourceGroup --sku Free
    ```

1. Создайте пользовательское удостоверение с именем `myUserAssignedIdentity` с помощью интерфейса командной строки.

    ```azurecli-interactive
    az identity create -resource-group myResourceGroup --name myUserAssignedIdentity
    ```

    В выходных данных этой команды Обратите внимание на значение `id` Свойства.

1. Выполните команду [AZ appconfig Identity Assign] , чтобы назначить новое назначенное пользователем удостоверение этому хранилищу конфигураций. Используйте значение `id` свойства, записанное на предыдущем шаге.

    ```azurecli-interactive
    az appconfig identity assign --name myTestAppConfigStore --resource-group myResourceGroup --identities /subscriptions/[subscription id]/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myUserAssignedIdentity
    ```

## <a name="removing-an-identity"></a>Удаление удостоверения

Чтобы удалить назначенное системой удостоверение, отключите эту функцию с помощью команды [AZ appconfig Identity Remove](/cli/azure/appconfig/identity#az-appconfig-identity-remove) в Azure CLI. Назначаемые пользователем удостоверения можно удалить по отдельности. Такое удаление назначаемого системой удостоверения приводит к удалению из AAD. Назначаемые системой удостоверения также удаляются из AAD автоматически, когда удаляется ресурс приложения.

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Создание приложения ASP.NET Core с помощью конфигурации приложения Azure](quickstart-aspnet-core-app.md)

[AZ appconfig Identity Assign]: /cli/azure/appconfig/identity#az-appconfig-identity-assign
[az login]: /cli/azure/reference-index#az-login
