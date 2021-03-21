---
title: Синхронизация содержимого из облачной папки
description: Узнайте, как развернуть приложение в службе приложений Azure с помощью синхронизации содержимого из облачной папки, включая OneDrive или Dropbox.
ms.assetid: 88d3a670-303a-4fa2-9de9-715cc904acec
ms.topic: article
ms.date: 02/25/2021
ms.reviewer: dariac
ms.custom: seodec18
ms.openlocfilehash: bfee320c7a8b4cbe8439c376350d1234b393bfb5
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102051245"
---
# <a name="sync-content-from-a-cloud-folder-to-azure-app-service"></a>Синхронизация содержимого из папки в облаке со службами приложений Azure
В этой статье показано, как синхронизировать содержимое [службы приложений Azure](./overview.md) из Dropbox и OneDrive. 

С помощью синхронизации содержимого вы работаете с кодом приложения и содержимым в назначенной облачной папке, чтобы убедиться, что они находятся в состоянии готовности к развертыванию, а затем синхронизируются со службой приложений нажатием кнопки. 

Из-за основных различий в интерфейсах API **OneDrive для бизнеса** пока не поддерживается.

> [!NOTE]
> Страница « **центр разработки» (классическая)** в портал Azure, которая является старым процессом развертывания, будет признана устаревшей в марте 2021. Это изменение не повлияет на существующие параметры развертывания в приложении, и вы можете продолжить управление развертыванием приложения на странице **центра развертывания** .

## <a name="enable-content-sync-deployment"></a>Включение развертывания синхронизации содержимого

1. В [портал Azure](https://portal.azure.com)перейдите на страницу управления для приложения службы приложений.

1. В меню слева щелкните Параметры **центра развертывания**  >  . 

1. В списке **источник** выберите **OneDrive** или **Dropbox**.

1. Щелкните **авторизовать** и следуйте запросам авторизации. 

    ![Показывает, как авторизовать OneDrive или Dropbox в центре развертывания портал Azure.](media/app-service-deploy-content-sync/choose-source.png)

    Для учетной записи Azure требуется только Авторизация в OneDrive или Dropbox. Чтобы авторизовать другую учетную запись OneDrive или Dropbox для приложения, нажмите кнопку **изменить учетную запись**.

1. В поле **Папка** выберите папку для синхронизации. Эта папка создается в следующий указанный путь к содержимому в OneDrive или Dropbox. 
   
    * **OneDrive**: `Apps\Azure Web Apps`
    * **Dropbox**: `Apps\Azure`
    
1. Выберите команду **Сохранить**.

## <a name="synchronize-content"></a>Синхронизация содержимого

# <a name="azure-portal"></a>[Портал Azure](#tab/portal)

1. В [портал Azure](https://portal.azure.com)перейдите на страницу управления для приложения службы приложений.

1. В меню слева щелкните **центр развертывания**  >  **Повторное развертывание или синхронизация**. 

    ![Здесь показано, как синхронизировать облачную папку со службой приложений.](media/app-service-deploy-content-sync/synchronize.png)
   
1. Нажмите кнопку **ОК** , чтобы подтвердить синхронизацию.

# <a name="azure-cli"></a>[Azure CLI](#tab/cli)

Запустите синхронизацию, выполнив следующую команду и заменив \<group-name> и \<app-name> :

```azurecli-interactive
az webapp deployment source sync –-resource-group <group-name> –-name <app-name>
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/powershell)

Запустите синхронизацию, выполнив следующую команду и заменив \<group-name> и \<app-name> :

```azurepowershell-interactive
Invoke-AzureRmResourceAction -ResourceGroupName <group-name> -ResourceType Microsoft.Web/sites -ResourceName <app-name> -Action sync -ApiVersion 2019-08-01 -Force –Verbose
```

-----

## <a name="disable-content-sync-deployment"></a>Отключение развертывания синхронизации содержимого

1. В [портал Azure](https://portal.azure.com)перейдите на страницу управления для приложения службы приложений.

1. В меню слева щелкните Параметры **центра развертывания**  >    >  **Отключить**. 

    ![Показано, как отключить синхронизацию облачных папок с приложением службы приложений в портал Azure.](media/app-service-deploy-content-sync/disable.png)

[!INCLUDE [What happens to my app during deployment?](../../includes/app-service-deploy-atomicity.md)]

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [развертывание из локального репозитория Git](deploy-local-git.md)