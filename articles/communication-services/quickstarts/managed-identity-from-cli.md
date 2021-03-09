---
title: Создание Azure Active Directory управляемого приложения удостоверений из Azure CLI
titleSuffix: An Azure Communication Services quickstart
description: Управляемые удостоверения позволяют авторизовать доступ к службам связи Azure из приложений, работающих на виртуальных машинах Azure, в приложениях функций и других ресурсах. Это краткое руководство посвящено управлению удостоверениями с помощью Azure CLI.
services: azure-communication-services
author: jbeauregardb
ms.service: azure-communication-services
ms.topic: how-to
ms.date: 02/25/2021
ms.author: jbeauregardb
ms.reviewer: mikben
ms.openlocfilehash: d9e60b02f34c020cfa180f0348fcc8e70998d7f5
ms.sourcegitcommit: 8d1b97c3777684bd98f2cfbc9d440b1299a02e8f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102486562"
---
# <a name="authorize-access-with-managed-identity-to-your-communication-resource-in-your-development-environment"></a>Авторизация доступа с помощью управляемого удостоверения к ресурсу связи в среде разработки

Клиентская библиотека удостоверений Azure предоставляет поддержку проверки подлинности маркеров Azure Active Directory (Azure AD) для пакета SDK Azure. Последние версии клиентских библиотек служб связи Azure для .NET, Java, Python и JavaScript интегрируются с библиотекой удостоверений Azure для предоставления простого и безопасного способа получения маркера OAuth 2,0 для авторизации запросов служб связи Azure.

Преимущество клиентской библиотеки Azure Identity заключается в том, что она позволяет использовать один и тот же код для проверки подлинности в нескольких службах независимо от того, выполняется ли приложение в среде разработки или в Azure. Клиентская библиотека удостоверений Azure проверяет подлинность субъекта безопасности. Когда код выполняется в Azure, участник безопасности является управляемым удостоверением для ресурсов Azure. В среде разработки управляемое удостоверение не существует, поэтому клиентская библиотека проверяет подлинность пользователя или зарегистрированного приложения в целях тестирования.

## <a name="prerequisites"></a>Предварительные требования

 - Azure CLI. [Руководство по установке](https://docs.microsoft.com/cli/azure/install-azure-cli)
 - Учетная запись Azure с активной подпиской. [Создать учетную запись бесплатно](https://azure.microsoft.com/free)

## <a name="setting-up"></a>Настройка

Управляемые удостоверения должны быть включены в ресурсах Azure, для которых выполняется авторизация. Сведения о включении управляемых удостоверений для ресурсов Azure см. в одной из следующих статей.

- [Портал Azure](../../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md)
- [Azure PowerShell](../../active-directory/managed-identities-azure-resources/qs-configure-powershell-windows-vm.md)
- [Azure CLI](../../active-directory/managed-identities-azure-resources/qs-configure-cli-windows-vm.md)
- [Шаблон Azure Resource Manager](../../active-directory/managed-identities-azure-resources/qs-configure-template-windows-vm.md)
- [Клиентские библиотеки Azure Resource Manager](../../active-directory/managed-identities-azure-resources/qs-configure-sdk-windows-vm.md)
- [Услуги для приложений](../../app-service/overview-managed-identity.md)

## <a name="authenticate-a-registered-application-in-the-development-environment"></a>Проверка подлинности зарегистрированного приложения в среде разработки

Если среда разработки не поддерживает единый вход или вход через веб-браузер, можно использовать зарегистрированное приложение для проверки подлинности из среды разработки.

### <a name="creating-an-azure-active-directory-registered-application"></a>Создание зарегистрированного приложения Azure Active Directory

Чтобы создать зарегистрированное приложение на основе Azure CLI, необходимо войти в учетную запись Azure, в которой должны выполняться операции. Для этого можно использовать `az login` команду и ввести учетные данные в браузере. После входа в учетную запись Azure из интерфейса командной строки можно вызвать `az ad sp create-for-rbac` команду, чтобы создать зарегистрированное приложение.

В следующих примерах используется Azure CLI для создания нового зарегистрированного приложения.

```azurecli
az ad sp create-for-rbac --name <application-name> 
```

`az ad sp create-for-rbac`Команда вернет список свойств субъекта-службы в формате JSON. Скопируйте эти значения, чтобы их можно было использовать для создания необходимых переменных среды на следующем шаге.

```json
{
    "appId": "generated-app-ID",
    "displayName": "service-principal-name",
    "name": "http://service-principal-uri",
    "password": "generated-password",
    "tenant": "tenant-ID"
}
```
> [!IMPORTANT]
> Назначение ролей Azure может занимать несколько минут.

#### <a name="set-environment-variables"></a>Настройка переменных среды

Клиентская библиотека удостоверений Azure считывает значения из трех переменных среды во время выполнения для проверки подлинности приложения. В следующей таблице описывается значение, которое задается для каждой переменной среды.

|Переменная среды|Значение
|-|-
|`AZURE_CLIENT_ID`|`appId` значение из созданного JSON 
|`AZURE_TENANT_ID`|`tenant` значение из созданного JSON
|`AZURE_CLIENT_SECRET`|`password` значение из созданного JSON

> [!IMPORTANT]
> После задания переменных среды закройте и снова откройте окно консоли. Если вы используете Visual Studio или другую среду разработки, может потребоваться перезапустить ее, чтобы зарегистрировать новые переменные среды.


## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Сведения о проверке подлинности](../concepts/authentication.md)

Полезные ссылки

- [Дополнительные сведения о библиотеке удостоверений Azure](/dotnet/api/overview/azure/identity-readme)
