---
title: Краткое руководство по созданию реестра с помощью PowerShell
description: Быстрый способ изучить создание частного реестра Docker в Реестре контейнеров Azure с помощью PowerShell.
ms.topic: quickstart
ms.date: 01/22/2019
ms.custom: seodec18, mvc, devx-track-azurepowershell
ms.openlocfilehash: b6928f1c45cdac93b70797daf41205b4c5db27e0
ms.sourcegitcommit: b8995b7dafe6ee4b8c3c2b0c759b874dff74d96f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/03/2021
ms.locfileid: "106283824"
---
# <a name="quickstart-create-a-private-container-registry-using-azure-powershell"></a>Краткое руководство. Создание частного реестра контейнеров Docker с помощью Azure PowerShell

Реестр контейнеров Azure — это управляемая служба частного реестра контейнеров Docker для создания, хранения и обслуживания образов контейнеров Docker. В этом руководстве рассматривается создание реестра контейнеров Azure с помощью PowerShell. Затем используйте команды Docker, чтобы отправить образ контейнера в реестр, после чего извлеките образ из контейнера и запустите его.

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Для работы с этим быстрым началом требуется модуль Azure PowerShell. Выполните команду `Get-Module -ListAvailable Az`, чтобы определить установленную версию. Если вам необходимо выполнить установку или обновление, см. статью [об установке модуля Azure PowerShell](/powershell/azure/install-az-ps).

Также необходим локально установленный модуль Docker. Docker предоставляет пакеты для систем под управлением [macOS][docker-mac], [Windows][docker-windows] и [Linux][docker-linux].

Та как в службе Azure Cloud Shell нет всех необходимых компонентов Docker (управляющая программа `dockerd`), ее нельзя использовать в этом руководстве.

## <a name="sign-in-to-azure"></a>Вход в Azure

С помощью команды [Connect-AzAccount][Connect-AzAccount] войдите в подписку Azure и следуйте инструкциям на экране.

```powershell
Connect-AzAccount
```

## <a name="create-resource-group"></a>Создать группу ресурсов

После аутентификации в Azure создайте группу ресурсов с помощью командлета [New-AzResourceGroup][New-AzResourceGroup]. Группа ресурсов — это логический контейнер, в котором происходит развертывание ресурсов Azure и управление ими.

```powershell
New-AzResourceGroup -Name myResourceGroup -Location EastUS
```

## <a name="create-container-registry"></a>Создание реестра контейнеров

Теперь создайте реестр контейнеров в новой группе ресурсов с помощью команды [New-AzContainerRegistry][New-AzContainerRegistry].

Имя реестра должно быть уникальным в пределах Azure и содержать от 5 до 50 буквенно-цифровых символов. В следующем примере создается реестр с именем myContainerRegistry007. Замените имя *myContainerRegistry007* в следующей команде. Затем с ее помощью создайте реестр.

```powershell
$registry = New-AzContainerRegistry -ResourceGroupName "myResourceGroup" -Name "myContainerRegistry007" -EnableAdminUser -Sku Basic
```

В этом кратком руководстве описано, как создать реестр ценовой категории *Базовый*. Это оптимальный (недорогой) вариант для разработчиков, которые знакомятся с Реестром контейнеров Azure. Дополнительные сведения об уровнях служб см. в статье [Уровни служб реестра контейнеров][container-registry-skus].

## <a name="log-in-to-registry"></a>Вход в раздел реестра

Перед отправкой и извлечением образов контейнеров необходимо войти в реестр. Включите режим администратора реестра, выполнив команду [Get-AzContainerRegistryCredential][Get-AzContainerRegistryCredential]. В рабочих сценариях следует использовать альтернативный [метод проверки подлинности](container-registry-authentication.md) для доступа к реестру, например субъект-службу. 

```powershell
$creds = Get-AzContainerRegistryCredential -Registry $registry
```

Затем выполните команду [docker login][docker-login], чтобы войти в систему:

```powershell
$creds.Password | docker login $registry.LoginServer -u $creds.Username --password-stdin
```

По завершении команда возвращает `Login Succeeded`.

> [!TIP]
> Azure CLI предоставляет команду `az acr login`: удобный способ входа в реестр контейнеров с помощью [индивидуальных учетных данных](container-registry-authentication.md#individual-login-with-azure-ad)без передачи учетных данных Docker.


[!INCLUDE [container-registry-quickstart-docker-push](../../includes/container-registry-quickstart-docker-push.md)]

[!INCLUDE [container-registry-quickstart-docker-pull](../../includes/container-registry-quickstart-docker-pull.md)]

## <a name="clean-up-resources"></a>Очистка ресурсов

После завершения работы с ресурсами, созданными в этом кратком руководстве, с помощью команды [Remove-AzResourceGroup][Remove-AzResourceGroup] вы можете удалить группу ресурсов, реестр контейнеров и сохраненные образы контейнеров:

```powershell
Remove-AzResourceGroup -Name myResourceGroup
```

## <a name="next-steps"></a>Дальнейшие действия

С помощью этого краткого руководства вы создали Реестр контейнеров Azure с использованием Azure PowerShell, отправили образ контейнера в него, а затем извлекли этот образ оттуда и запустили его. Чтобы продолжить работу с Реестром контейнеров Azure, перейдите к следующим руководствам.

> [!div class="nextstepaction"]
> [Руководства по использованию Реестра контейнеров Azure][container-registry-tutorial-prepare-registry]

> [!div class="nextstepaction"]
> [Руководство. Создание и развертывание образов контейнера в облаке с помощью службы "Задачи Реестра контейнеров Azure"][container-registry-tutorial-quick-task]

<!-- LINKS - external -->
[docker-linux]: https://docs.docker.com/engine/installation/#supported-platforms
[docker-login]: https://docs.docker.com/engine/reference/commandline/login/
[docker-mac]: https://docs.docker.com/docker-for-mac/
[docker-push]: https://docs.docker.com/engine/reference/commandline/push/
[docker-tag]: https://docs.docker.com/engine/reference/commandline/tag/
[docker-windows]: https://docs.docker.com/docker-for-windows/

<!-- Links - internal -->
[Connect-AzAccount]: /powershell/module/az.accounts/connect-azaccount
[Get-AzContainerRegistryCredential]: /powershell/module/az.containerregistry/get-azcontainerregistrycredential
[Get-Module]: /powershell/module/microsoft.powershell.core/get-module
[New-AzContainerRegistry]: /powershell/module/az.containerregistry/New-AzContainerRegistry
[New-AzResourceGroup]: /powershell/module/az.resources/new-azresourcegroup
[Remove-AzResourceGroup]: /powershell/module/az.resources/remove-azresourcegroup
[container-registry-tutorial-quick-task]: container-registry-tutorial-quick-task.md
[container-registry-skus]: container-registry-skus.md
[container-registry-tutorial-prepare-registry]: container-registry-tutorial-prepare-registry.md
