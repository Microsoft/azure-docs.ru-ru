---
title: Создание очереди Служебной шины с помощью Azure PowerShell
description: Из этого краткого руководства вы узнаете, как создать пространство имен и очередь в пространстве имен Служебной шины с помощью Azure PowerShell.
author: spelluru
ms.devlang: dotnet
ms.topic: quickstart
ms.date: 08/12/2020
ms.author: spelluru
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 8cf024735c66e6bae9d334e3d8ce8d0a0eed1426
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "95799107"
---
# <a name="use-azure-powershell-to-create-a-service-bus-namespace-and-a-queue"></a>Создание пространства имен и очереди Служебной шины с помощью Azure PowerShell
Из этого краткого руководства вы узнаете, как создать пространство имен и очередь Служебной шины с помощью Azure PowerShell. В нем также описывается, как получить учетные данные авторизации, которые могут использоваться клиентским приложением для отправки сообщений в очередь и их получения из нее. 

[!INCLUDE [howto-service-bus-queues](../../includes/howto-service-bus-queues.md)]


## <a name="prerequisites"></a>Обязательные условия

Для работы с этим руководством требуется подписка Azure. Если у вас еще нет подписки Azure, вы можете создать [бесплатная учетная запись][], прежде чем начинать работу. 

Для выполнения инструкций в этом кратком руководстве необходимо воспользоваться Azure Cloud Shell после входа на портал Azure. Дополнительные сведения об Azure Cloud Shell см. в статье [Обзор Azure Cloud Shell](../cloud-shell/overview.md). Вы также можете [установить](/powershell/azure/install-Az-ps) и использовать Azure PowerShell на компьютере. 


## <a name="provision-resources"></a>Подготовка ресурсов
1. Войдите на [портал Azure](https://portal.azure.com).
2. Запустите Azure Cloud Shell, щелкнув значок, показанный на следующем изображении. 

    :::image type="content" source="./media/service-bus-quickstart-powershell/launch-cloud-shell.png" alt-text="Запуск Cloud Shell":::
3. В нижней части окна Cloud Shell переключитесь с **Bash** на **PowerShell**. 

    :::image type="content" source="./media/service-bus-quickstart-powershell/cloud-power-shell.png" alt-text="Переключение в режим PowerShell":::    
4. Выполните следующую команду, чтобы создать группу ресурсов Azure. При желании измените имя группы ресурсов и расположение. 

    ```azurepowershell-interactive
    New-AzResourceGroup –Name ContosoRG –Location eastus
    ```
5. Выполните следующую команду, чтобы создать пространство имен Служебной шины для обмена сообщениями. В этом примере `ContosoRG` — это группа ресурсов, созданная на предыдущем шаге. `ContosoSBusNS` — это имя очереди в пространстве имен Служебной шины, созданном в такой группе ресурсов. 

    ```azurepowershell-interactive
    New-AzServiceBusNamespace -ResourceGroupName ContosoRG -Name ContosoSBusNS -Location eastus
    ```
6. Выполните следующую команду, чтобы создать очередь в пространстве имен, созданном на предыдущем шаге. 

    ```azurepowershell-interactive
    New-AzServiceBusQueue -ResourceGroupName ContosoRG -NamespaceName ContosoSBusNS -Name ContosoOrdersQueue 
    ```
7. Получите первичную строку подключения для пространства имен. Эта строка подключения используется для подключения к очереди и отправки и получения сообщений. 

    ```azurepowershell-interactive    
    Get-AzServiceBusKey -ResourceGroupName ContosoRG -Namespace ContosoSBusNS -Name RootManageSharedAccessKey
    ```

    Запишите строку подключения и имя очереди. Они используются для отправки и получения сообщений. 


## <a name="next-steps"></a>Дальнейшие действия
С помощью инструкций из этой статьи вы создали пространство имен и очередь Служебной шины. Чтобы узнать, как отправлять сообщения в очередь и получать их из нее, см. одно из следующих кратких руководств в разделе **Отправка и получение сообщений**. 

- [.NET](service-bus-dotnet-get-started-with-queues.md)
- [Java](service-bus-java-how-to-use-queues.md)
- [JavaScript](service-bus-nodejs-how-to-use-queues.md)
- [Python](service-bus-python-how-to-use-queues.md)
- [PHP](service-bus-php-how-to-use-queues.md)
- [Ruby](service-bus-ruby-how-to-use-queues.md)

[бесплатная учетная запись]: https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio

