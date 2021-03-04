---
title: Настройка учетных данных развертывания
description: Сведения о типах учетных данных развертывания в службе приложений Azure, их настройке и использовании.
ms.topic: article
ms.date: 02/11/2021
ms.reviewer: byvinyal
ms.custom: seodec18
ms.openlocfilehash: c7d3c7c8b5da40a4e9ccd9085af5a850b9ebc3dd
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102052353"
---
# <a name="configure-deployment-credentials-for-azure-app-service"></a>Настройка учетных данных развертывания службы приложений Azure
Чтобы защитить развертывание приложений с локального компьютера, [служба приложений Azure](./overview.md) поддерживает два типа учетных данных для развертывания [локального репозитория Git](deploy-local-git.md) и [развертывания FTP/S](deploy-ftp.md). Эти учетные данные не совпадают с учетными данными подписки Azure.

[!INCLUDE [app-service-deploy-credentials](../../includes/app-service-deploy-credentials.md)]

> [!NOTE]
> Страница « **центр разработки» (классическая)** в портал Azure, которая является старым процессом развертывания, будет признана устаревшей в марте 2021. Это изменение не повлияет на существующие параметры развертывания в приложении, и вы можете продолжить управление развертыванием приложения на странице **центра развертывания** .

## <a name="configure-user-scope-credentials"></a><a name="userscope"></a>Настройка учетных данных области пользователя

# <a name="azure-cli"></a>[Azure CLI](#tab/cli)

Выполните команду [AZ webapp Deployment User Set](/cli/azure/webapp/deployment/user#az-webapp-deployment-user-set) . Вместо \<username> и \<password> укажите имя пользователя и пароль развертывания. 

- Имя пользователя должно быть уникальным в Azure. Кроме того, чтобы отправка в локальный репозиторий Git работала, имя пользователя не должно содержать символ @. 
- Пароль должен содержать не менее восьми символов и включать два из трех следующих элементов: буквы, цифры и символы. 

```azurecli-interactive
az webapp deployment user set --user-name <username> --password <password>
```

В выходных данных JSON пароль отображается как `null`.

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/powershell)

Учетные данные области пользователя нельзя настроить с помощью Azure PowerShell. Используйте другой метод или рассмотрите возможность [использования учетных данных области приложения](#appscope). 

# <a name="azure-portal"></a>[Портал Azure](#tab/portal)

Учетные данные области пользователя можно настроить на [странице ресурсов](../azure-resource-manager/management/manage-resources-portal.md#manage-resources)любого приложения. Независимо от того, в каком приложении вы настраиваете эти учетные данные, оно применяется ко всем приложениям для всех подписок в учетной записи Azure. 

В [портал Azure](https://portal.azure.com)необходимо иметь по крайней мере одно приложение, чтобы получить доступ к странице учетные данные для развертывания. Чтобы настроить учетные данные области пользователя, выполните следующие действия.

1. В левом меню приложения выберите > **центр развертывания**  >  **FTPS учетные данные** или **локальные учетные данные Git/FTPS**.

    ![Здесь показано, как можно выбрать панель мониторинга FTP в центре развертывания в службах приложений Azure.](./media/app-service-deployment-credentials/access-no-git.png)

2. Прокрутите вниз до **области пользователь**, настройте **имя пользователя** и **пароль**, а затем нажмите кнопку **сохранить**.

Когда вы настроите учетные данные развертывания, имя пользователя для развертывания *Git* можно будет найти на странице **Обзор** приложения.

![Здесь показано, как найти имя пользователя развертывания Git на странице обзора приложения.](./media/app-service-deployment-credentials/deployment_credentials_overview.png)

Если развертывание Git настроено, на странице отображается **имя пользователя Git или развертывания**; в противном случае **имя пользователя FTP или развертывания**.

> [!NOTE]
> В Azure не отображается пароль развертывания области пользователя. Если вы забыли пароль, можно сбросить учетные данные, выполнив действия, описанные в этом разделе.
>
> 

-----

## <a name="use-user-scope-credentials-with-ftpftps"></a>Использовать учетные данные области пользователя с помощью FTP или FTPS

Для проверки подлинности в конечной точке FTP или FTPS с использованием учетных данных области пользователя требуется имя пользователя в следующем формате: `<app-name>\<user-name>`

Так как учетные данные области пользователя связаны с пользователем, а не с конкретным ресурсом, имя пользователя должно быть в этом формате для направления действия входа в подходящую конечную точку приложения.

## <a name="get-application-scope-credentials"></a><a name="appscope"></a>Получить учетные данные области приложения

# <a name="azure-cli"></a>[Azure CLI](#tab/cli)

Получите учетные данные области приложения с помощью команды [AZ webapp Deployment List-Publishing-Profiles](/cli/azure/webapp/deployment#az_webapp_deployment_list_publishing_profiles) . Пример.

```azurecli-interactive
az webapp deployment list-publishing-profiles --resource-group <group-name> --name <app-name>
```

Для [локального развертывания Git](deploy-local-git.md)можно также использовать команду [AZ webapp Deployment List-Publishing-Credentials](/cli/azure/webapp/deployment#az_webapp_deployment_list_publishing_credentials) , чтобы получить удаленный универсальный код ресурса (URI) Git для приложения с уже внедренными учетными данными области приложения. Пример.

```azurecli-interactive
az webapp deployment list-publishing-credentials --resource-group <group-name> --name <app-name> --query scmUri
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/powershell)

Получите учетные данные области приложения с помощью команды [Get-азвебапппублишингпрофиле](/powershell/module/az.websites/get-azwebapppublishingprofile) . Пример.

```azurepowershell-interactive
Get-AzWebAppPublishingProfile -ResourceGroupName <group-name> -Name <app-name>
```

# <a name="azure-portal"></a>[Портал Azure](#tab/portal)

1. В левом меню приложения выберите **центр развертывания**  >  **FTPS учетные данные** или **локальные учетные данные Git/FTPS**.

    ![Здесь показано, как можно выбрать панель мониторинга FTP в центре развертывания в службах приложений Azure.](./media/app-service-deployment-credentials/access-no-git.png)

2. В разделе **область приложения** щелкните ссылку **Копировать** , чтобы скопировать имя пользователя или пароль.

-----

## <a name="reset-application-scope-credentials"></a>Сбросить учетные данные области приложения

# <a name="azure-cli"></a>[Azure CLI](#tab/cli)

Сбросьте учетные данные области приложения с помощью команды [AZ Resource Invoke-Action](/cli/azure/resource#az_resource_invoke_action) :

```azurecli-interactive
az resource invoke-action --action newpassword --resource-group <group-name> --name <app-name> --resource-type Microsoft.Web/sites
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/powershell)

Сбросьте учетные данные области приложения с помощью команды [Invoke-азресаурцеактион](/powershell/module/az.resources/invoke-azresourceaction) :

```azurepowershell-interactive
Invoke-AzResourceAction -ResourceGroupName <group-name> -ResourceType Microsoft.Web/sites -ResourceName <app-name> -Action newpassword
```

# <a name="azure-portal"></a>[Портал Azure](#tab/portal)

1. В левом меню приложения выберите **центр развертывания**  >  **FTPS учетные данные** или **локальные учетные данные Git/FTPS**.

    ![Здесь показано, как можно выбрать панель мониторинга FTP в центре развертывания в службах приложений Azure.](./media/app-service-deployment-credentials/access-no-git.png)

2. В разделе **область приложения** выберите **сбросить**.

-----

## <a name="disable-basic-authentication"></a>Отключить обычную проверку подлинности

Некоторые организации должны удовлетворять требованиям безопасности, а отключать доступ через FTP или WebDeploy. Таким образом, члены Организации могут получать доступ к своим службам приложений только через API-интерфейсы, управляемые Azure Active Directory (Azure AD).

### <a name="ftp"></a>FTP

Чтобы отключить FTP-доступ к сайту, выполните следующую команду интерфейса командной строки. Замените заполнители своей группой ресурсов и именем сайта. 

```azurecli-interactive
az resource update --resource-group <resource-group> --name ftp --namespace Microsoft.Web --resource-type basicPublishingCredentialsPolicies --parent sites/<site-name> --set properties.allow=false
```

Чтобы убедиться, что FTP-доступ заблокирован, можно попытаться выполнить проверку подлинности с помощью FTP-клиента, например FileZilla. Чтобы получить учетные данные публикации, перейдите в колонку "Обзор" сайта и щелкните Скачать профиль публикации. Для проверки подлинности используйте имя узла FTP, имя пользователя и пароль, и вы получите ответ об ошибке 401, указывающий на то, что вы не прошли авторизацию.

### <a name="webdeploy-and-scm"></a>WebDeploy и SCM

Чтобы отключить базовый доступ с проверкой подлинности к порту веб-развертывания и сайту SCM, выполните следующую команду CLI. Замените заполнители своей группой ресурсов и именем сайта. 

```azurecli-interactive
az resource update --resource-group <resource-group> --name scm --namespace Microsoft.Web --resource-type basicPublishingCredentialsPolicies --parent sites/<site-name> --set properties.allow=false
```

Чтобы убедиться, что учетные данные профиля публикации заблокированы для WebDeploy, попробуйте [Опубликовать веб-приложение с помощью Visual Studio 2019](/visualstudio/deployment/quickstart-deploy-to-azure).

### <a name="disable-access-to-the-api"></a>Отключить доступ к API

API в предыдущем разделе является резервной системой управления доступом на основе ролей Azure (Azure RBAC). Это означает, что вы можете [создать настраиваемую роль](../role-based-access-control/custom-roles.md#steps-to-create-a-custom-role) и назначить ей пользователей с низкими привелджедми, чтобы они не могли включить обычную проверку подлинности на всех сайтах. Чтобы настроить пользовательскую роль, [выполните следующие инструкции](https://azure.github.io/AppService/2020/08/10/securing-data-plane-access.html#create-a-custom-rbac-role).

Можно также использовать [Azure Monitor](https://azure.github.io/AppService/2020/08/10/securing-data-plane-access.html#audit-with-azure-monitor) для аудита любых успешных запросов проверки подлинности и применения [политики Azure](https://azure.github.io/AppService/2020/08/10/securing-data-plane-access.html#enforce-compliance-with-azure-policy) для применения этой конфигурации ко всем сайтам в подписке.

## <a name="next-steps"></a>Дальнейшие действия

Узнайте, как использовать эти учетные данные для развертывания приложения из [локального репозитория Git](deploy-local-git.md) или с помощью [FTP(S)](deploy-ftp.md).
