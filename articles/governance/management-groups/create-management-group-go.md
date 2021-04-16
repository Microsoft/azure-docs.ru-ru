---
title: Краткое руководство. Создание группы управления с помощью Go
description: В этом кратком руководстве показано, как с помощью Go создать группу управления для организации ресурсов в иерархии ресурсов.
ms.date: 03/31/2021
ms.topic: quickstart
ms.custom: devx-track-csharp
ms.openlocfilehash: bf2d2c556cfd6ada6d31fc6ee797888ed0899573
ms.sourcegitcommit: 99fc6ced979d780f773d73ec01bf651d18e89b93
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106091452"
---
# <a name="quickstart-create-a-management-group-with-go"></a>Краткое руководство. Создание группы управления с помощью Go

Группы управления — это контейнеры, которые помогают управлять доступом, политикой и соответствием требованиям в нескольких подписках. Создание таких контейнеров позволяет построить эффективную и экономную иерархию, которую можно использовать с [политикой Azure](../policy/overview.md) и [элементами управления доступом на основе ролей Azure](../../role-based-access-control/overview.md). Дополнительные сведения о группах управления см. в статье [Упорядочение ресурсов с помощью групп управления Azure](overview.md).

Создание первой группы управления в каталоге может занять до 15 минут. Существуют процессы, выполняемые в первый раз при настройке службы групп управления в Azure для вашего каталога. По завершении процесса вы получите уведомление. См. сведения о [начальной настройке групп управления](./overview.md#initial-setup-of-management-groups).

## <a name="prerequisites"></a>Предварительные требования

- Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.

- Субъект-служба Azure, включая _clientId_ и _clientSecret_. Если у вас нет субъекта-службы, который можно использовать для Политики Azure, или вы хотите создать новый субъект-службу, см. статью о [библиотеках управления Azure для аутентификации .NET](/dotnet/azure/sdk/authentication#mgmt-auth).
  Пропустите этот шаг, чтобы установить пакеты .NET Core, так как мы выполним это действие на следующих шагах.

- Любой пользователь Azure AD в арендаторе может создать группу управления без разрешения на запись для группы управления, назначенного этому пользователю, если [защита иерархии](./how-to/protect-resource-hierarchy.md#setting---require-authorization) не включена. Эта новая группа управления становится дочерней по отношению к корневой группе управления или [группе управления по умолчанию](./how-to/protect-resource-hierarchy.md#setting---default-management-group), а ее создателю назначается роль "Владелец". Служба группы управления обеспечивает эту возможность, чтобы назначения ролей не требовались на корневом уровне. У пользователей нет доступа к корневой группе управления при ее создании. Чтобы избежать трудностей при поиске глобальных администраторов Azure AD для настройки работы с группами управления, мы разрешаем создавать исходные группы управления на корневом уровне.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="add-the-management-group-package"></a>Добавление пакета группы управления

Чтобы можно было управлять группами управления с помощью Go, необходимо добавить пакет. Этот пакет работает во всех средах, где можно использовать Go, в том числе в [Bash для Windows 10](/windows/wsl/install-win10). Также его можно установить локально.

1. Убедитесь, что установлена последняя версия Go (не ниже **1.15**). Если она еще не установлена, скачайте ее на сайте [Golang.org](https://golang.org/dl/).

1. Убедитесь, что установлена последняя версия Azure CLI (не ниже **2.5.1**). Если Azure CLI еще не установлен, см. [эту статью](/cli/azure/install-azure-cli).

   > [!NOTE]
   > Azure CLI требуется для того, чтобы в Go можно было использовать метод `auth.NewAuthorizerFromCLI()` в следующем примере. См. сведения о других параметрах в разделе [Azure SDK для Go — дополнительные сведения о проверке подлинности](https://github.com/Azure/azure-sdk-for-go#more-authentication-details).

1. Проверка подлинности с помощью Azure CLI.

   ```azurecli
   az login
   ```

1. В выбранной вами среде Go установите необходимые пакеты для групп управления:

   ```bash
   # Add the management group package for Go
   go get -u github.com/Azure/azure-sdk-for-go/services/preview/resources/mgmt/2018-03-01-preview/managementgroups

   # Add the Azure auth package for Go
   go get -u github.com/Azure/go-autorest/autorest/azure/auth
   ```

## <a name="application-setup"></a>Настройка приложения

После добавления пакетов Go в выбранную вами среду настройте приложение Go, которое может создать группу управления.

1. Создайте приложение Go и сохраните следующий исходный код как `mgCreate.go`.

   ```Go
   package main
   
   import (
    "context"
    "fmt"
    "os"
   
    mg "github.com/Azure/azure-sdk-for-go/services/preview/resources/mgmt/2018-03-01-preview/managementgroups"
    "github.com/Azure/go-autorest/autorest/azure/auth"
   )
   
   func main() {
    // Get variables from command line arguments
    var mgName = os.Args[1]
   
    // Create and authorize a client
    mgClient := mg.NewClient()
    authorizer, err := auth.NewAuthorizerFromCLI()
    if err == nil {
        mgClient.Authorizer = authorizer
    } else {
        fmt.Printf(err.Error())
    }
   
    // Create the request
    Request := mg.CreateManagementGroupRequest{
        Name: &mgName,
    }
   
    // Run the query and get the results
    var results, queryErr = mgClient.CreateOrUpdate(context.Background(), mgName, Request, "no-cache")
    if queryErr == nil {
        fmt.Printf("Results: " + fmt.Sprint(results) + "\n")
    } else {
        fmt.Printf(queryErr.Error())
    }
   }
   ```

1. Выполните сборку приложения Go.

   ```bash
   go build mgCreate.go
   ```

1. Создайте группу управления с помощью скомпилированного приложения Go. Замените `<Name>` именем новой группы управления:

   ```bash
   mgCreate "<Name>"
   ```

В результате выполнения в корневой группе управления появится новая группа управления.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы хотите удалить установленные пакеты из среды Go, это можно сделать с помощью следующей команды.

```bash
# Remove the installed packages from the Go environment
go clean -i github.com/Azure/azure-sdk-for-go/services/preview/resources/mgmt/2018-03-01-preview/managementgroups
go clean -i github.com/Azure/go-autorest/autorest/azure/auth
```

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве показано, как создать группу управления для организации иерархии ресурсов. Группа управления может содержать подписки и другие группы управления.

Чтобы узнать больше о группах управления и управлении иерархией ресурсов, см. следующее руководство:

> [!div class="nextstepaction"]
> [Управление ресурсами с помощью групп управления](./manage.md)