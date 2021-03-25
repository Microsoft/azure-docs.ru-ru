---
title: Краткое руководство. Создание очереди Служебной шины Azure с помощью Azure CLI | Документация Майкрософт
description: Из этого краткого руководства вы узнаете, как с помощью Azure CLI создать пространство имен и очередь в пространстве имен Служебной шины.
author: spelluru
ms.topic: quickstart
ms.date: 08/12/2020
ms.author: spelluru
ms.openlocfilehash: 14892b114f3bb8a0bb6c07f7cdd01a174b931cf9
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95810624"
---
# <a name="use-the-azure-cli-to-create-a-service-bus-namespace-and-a-queue"></a>Использование Azure CLI для создания пространства имен и очереди Служебной шины
Из этого краткого руководства вы узнаете, как создать пространство имен и очередь Служебной шины с помощью Azure CLI. В нем также описывается, как получить учетные данные авторизации, которые могут использоваться клиентским приложением для отправки сообщений в очередь и их получения из нее. 

[!INCLUDE [howto-service-bus-queues](../../includes/howto-service-bus-queues.md)]

## <a name="prerequisites"></a>Обязательные условия
Если у вас еще нет подписки Azure, вы можете создать [бесплатная учетная запись][free account], прежде чем начинать работу.

Для выполнения инструкций в этом кратком руководстве необходимо воспользоваться Azure Cloud Shell после входа на портал Azure. Дополнительные сведения об Azure Cloud Shell см. в статье [Обзор Azure Cloud Shell](../cloud-shell/overview.md). Вы также можете [установить](/cli/azure/install-azure-cli) и использовать Azure PowerShell на компьютере. 

## <a name="provision-resources"></a>Подготовка ресурсов
1. Войдите на [портал Azure](https://portal.azure.com).
2. Запустите Azure Cloud Shell, щелкнув значок, показанный на следующем изображении. Переключитесь на режим **Bash**, если CloudShell работает в режиме **PowerShell**. 

    :::image type="content" source="./media/service-bus-quickstart-powershell/launch-cloud-shell.png" alt-text="Запуск Cloud Shell":::
3. Выполните следующую команду, чтобы создать группу ресурсов Azure. При желании измените имя группы ресурсов и расположение. 

    ```azurecli-interactive
    az group create --name ContosoRG --location eastus
    ```
4. Выполните следующую команду, чтобы создать пространство имен Служебной шины для обмена сообщениями.

    ```azurecli-interactive
    az servicebus namespace create --resource-group ContosoRG --name ContosoSBusNS --location eastus
    ```
5. Выполните следующую команду, чтобы создать очередь в пространстве имен, созданном на предыдущем шаге. В этом примере `ContosoRG` — это группа ресурсов, созданная на предыдущем шаге. `ContosoSBusNS` — это имя очереди в пространстве имен Служебной шины, созданном в такой группе ресурсов. 

    ```azurecli-interactive
    az servicebus queue create --resource-group ContosoRG --namespace-name ContosoSBusNS --name ContosoOrdersQueue
    ```
6. Выполните следующую команду, чтобы получить первичную строку подключения для пространства имен. Эта строка подключения используется для подключения к очереди и отправки и получения сообщений. 

    ```azurecli-interactive
    az servicebus namespace authorization-rule keys list --resource-group ContosoRG --namespace-name ContosoSBusNS --name RootManageSharedAccessKey --query primaryConnectionString --output tsv    
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

[free account]: https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio

