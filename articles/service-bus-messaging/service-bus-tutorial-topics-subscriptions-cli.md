---
title: Создание раздела служебной шины и подписок на него с помощью Azure CLI
description: Из этого краткого руководства вы узнаете, как создать раздел служебной шины и подписки на этот раздел с помощью Azure CLI.
ms.date: 06/23/2020
ms.topic: quickstart
author: spelluru
ms.author: spelluru
ms.openlocfilehash: 080b089efa276779420f6d9bc8e76272f1e65788
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "90069702"
---
# <a name="use-azure-cli-to-create-a-service-bus-topic-and-subscriptions-to-the-topic"></a>Создание раздела служебной шины и подписок на него с помощью Azure CLI
В этом кратком руководстве описано, как создать раздел служебной шины и подписки на него с помощью Azure CLI. 

## <a name="what-are-service-bus-topics-and-subscriptions"></a>Что такое разделы и подписки служебной шины?
Разделы и подписки служебной шины поддерживают модель обмена сообщениями " *публикация и подписка* ". При использовании разделов и подписок компоненты распределенного приложения не взаимодействуют между собой напрямую, а обмениваются сообщениями через раздел, который выступает в качестве посредника.

![TopicConcepts](./media/service-bus-java-how-to-use-topics-subscriptions/sb-topics-01.png)

В отличие от очередей служебной шины, где каждое сообщение обрабатывается одним потребителем, разделы и подписки предоставляют вид связи одного со многими с помощью шаблона публикации/подписки. Можно зарегистрировать несколько подписок на раздел. Когда сообщение отправляется в раздел, оно затем может обрабатываться независимо каждой подпиской. Раздел подписки напоминает виртуальную очередь, которая получает копии сообщений, отправленных в раздел. Вы можете зарегистрировать правила фильтрации для раздела на основе подписки, которые позволят указать, какие сообщения и от каких подписок могут быть получены разделом.

Разделы и подписки служебной шины обеспечивают возможность масштабирования для обработки большого количества сообщений для большого количества пользователей и приложений.

## <a name="prerequisites"></a>Обязательные условия
Если у вас еще нет подписки Azure, вы можете создать [бесплатная учетная запись][free account], прежде чем начинать работу.

Для выполнения инструкций в этом кратком руководстве необходимо воспользоваться Azure Cloud Shell после входа на портал Azure. Дополнительные сведения об Azure Cloud Shell см. в статье [Обзор Azure Cloud Shell](../cloud-shell/overview.md). Вы также можете [установить](/cli/azure/install-azure-cli) и использовать Azure PowerShell на компьютере. 

## <a name="create-a-service-bus-topic-and-subscriptions"></a>Создание раздела и подписок служебной шины
Каждая [подписка на раздел](service-bus-messaging-overview.md#topics) может получать копию любого сообщения. Разделы полностью совместимы с очередями служебной шины на уровнях протоколов и семантики. Разделы служебной шины поддерживают широкий набор правил отбора и условий фильтра, а также позволяют создать действия для присвоения или изменения свойств сообщения. Каждый раз, когда срабатывает правило, создается новое сообщение. Чтобы узнать больше о правилах, фильтрах и действиях, перейдите по [этой ссылке](topic-filters.md).

1. Войдите на [портал Azure](https://portal.azure.com).
2. Запустите Azure Cloud Shell, щелкнув значок, показанный на следующем изображении. Переключитесь на режим **Bash**, если CloudShell работает в режиме **PowerShell**. 

    :::image type="content" source="./media/service-bus-quickstart-powershell/launch-cloud-shell.png" alt-text="Запуск Cloud Shell":::
3. Выполните следующую команду, чтобы создать группу ресурсов Azure. При желании измените имя группы ресурсов и расположение. 

    ```azurecli-interactive
    az group create --name MyResourceGroup --location eastus
    ```
4. Выполните следующую команду, чтобы создать пространство имен Служебной шины для обмена сообщениями. Обновите имя пространства имен, чтобы оно было уникальным. 

    ```azurecli-interactive
    namespaceName=MyNameSpace$RANDOM
    az servicebus namespace create --resource-group MyResourceGroup --name $namespaceName --location eastus
    ```
5. Чтобы создать раздел в пространстве имен, выполните следующую команду. 

    ```azurecli-interactive
    az servicebus topic create --resource-group MyResourceGroup   --namespace-name $namespaceName --name MyTopic
    ```
6. Создание первой подписки на раздел
    
    ```azurecli-interactive
    az servicebus topic subscription create --resource-group MyResourceGroup --namespace-name $namespaceName --topic-name MyTopic --name S1    
    ```
6. Создание второй подписки на раздел
    
    ```azurecli-interactive
    az servicebus topic subscription create --resource-group MyResourceGroup --namespace-name $namespaceName --topic-name MyTopic --name S2    
    ```
6. Создание третьей подписки на раздел
    
    ```azurecli-interactive
    az servicebus topic subscription create --resource-group MyResourceGroup --namespace-name $namespaceName --topic-name MyTopic --name S3    
    ```
7. Создайте фильтр для первой подписки, используя фильтр с пользовательскими свойствами (`StoreId` — одно из этих значений: `Store1`, `Store2` и `Store3`).

    ```azurecli-interactive
    az servicebus topic subscription rule create --resource-group MyResourceGroup --namespace-name $namespaceName --topic-name MyTopic --subscription-name S1 --name MyFilter --filter-sql-expression "StoreId IN ('Store1','Store2','Store3')"    
    ```
8. Создайте фильтр для второй подписки, используя фильтр с пользовательскими свойствами (`StoreId = Store4`).

    ```azurecli-interactive
    az servicebus topic subscription rule create --resource-group MyResourceGroup --namespace-name $namespaceName --topic-name myTopic --subscription-name S2 --name MySecondFilter --filter-sql-expression "StoreId = 'Store4'"    
    ```
9. Создайте фильтр для третьей подписки, используя фильтр с пользовательскими свойствами (`StoreId` не в `Store1`, `Store2`, `Store3` или `Store4`).

    ```azurecli-interactive
    az servicebus topic subscription rule create --resource-group MyResourceGroup --namespace-name $namespaceName --topic-name MyTopic --subscription-name S3 --name MyThirdFilter --filter-sql-expression "StoreId NOT IN ('Store1','Store2','Store3', 'Store4')"     
    ```
10. Выполните следующую команду, чтобы получить первичную строку подключения для пространства имен. Эта строка подключения используется для подключения к очереди и отправки и получения сообщений. 

    ```azurecli-interactive
    az servicebus namespace authorization-rule keys list --resource-group MyResourceGroup --namespace-name $namespaceName --name RootManageSharedAccessKey --query primaryConnectionString --output tsv    
    ```

    Запишите строку подключения и имя раздела. Они используются для отправки и получения сообщений. 
    

## <a name="next-steps"></a>Дальнейшие действия
Чтобы узнать, как отправлять сообщения в раздел и получать эти сообщения с помощью подписки, ознакомьтесь с приведенной ниже статьей (выберите язык программирования в оглавлении). 

> [!div class="nextstepaction"]
> [Шаблон "публикация/подписка" для обмена сообщениями](service-bus-dotnet-how-to-use-topics-subscriptions.md)


[free account]: https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio
[fully qualified domain name]: https://wikipedia.org/wiki/Fully_qualified_domain_name
[Install the Azure CLI]: /cli/azure/install-azure-cli
[az group create]: /cli/azure/group#az_group_create
