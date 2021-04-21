---
title: Краткое руководство. Создание геореплицированного шаблона реестра — шаблон Azure Resource Manager
description: Узнайте, как создать геореплицированный реестр контейнеров Azure с помощью шаблона Azure Resource Manager.
services: azure-resource-manager
author: dlepow
ms.author: danlep
ms.date: 10/06/2020
ms.topic: quickstart
ms.service: azure-resource-manager
ms.custom:
- subject-armqs
- mode-arm
ms.openlocfilehash: c59c7897054b2ad65a76353e6d886af46cac91e0
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/16/2021
ms.locfileid: "107537435"
---
# <a name="quickstart-create-a-geo-replicated-container-registry-by-using-an-arm-template"></a>Краткое руководство. Создание геореплицированного реестра контейнеров с помощью шаблона ARM

В этом кратком руководстве показано, как создать экземпляр Реестра контейнеров Azure с помощью шаблона Azure Resource Manager (шаблона ARM). Шаблон настраивает [геореплицированный](container-registry-geo-replication.md) реестр, который автоматически синхронизирует содержимое реестра в нескольких регионах Azure. Георепликация дает возможность получить ограниченный сетью доступ к изображениям из региональных развертываний, предоставляя единую среду управления. Эта функция относится к уровню служб реестра [Премиум](container-registry-skus.md).

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Если среда соответствует предварительным требованиям и вы знакомы с использованием шаблонов ARM, нажмите кнопку **Развертывание в Azure**. Шаблон откроется на портале Azure.

[![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-container-registry-geo-replication%2Fazuredeploy.json)

## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.

## <a name="review-the-template"></a>Изучение шаблона

Шаблон, используемый в этом кратком руководстве, взят из [шаблонов быстрого запуска Azure](https://azure.microsoft.com/resources/templates/101-container-registry-geo-replication/). Шаблон настраивает реестр и дополнительную региональную реплику.

:::code language="json" source="~/quickstart-templates/101-container-registry-geo-replication/azuredeploy.json":::

В шаблоне определены следующие ресурсы:

* **[Microsoft.ContainerRegistry/registries](/azure/templates/microsoft.containerregistry/registries)** : создание реестра контейнеров Azure
* **[Microsoft.ContainerRegistry/registries/replications](/azure/templates/microsoft.containerregistry/registries/replications)** : создание реплики реестра контейнеров

Другие примеры шаблонов службы "Реестр контейнеров Azure" можно найти в [коллекции шаблонов быстрого запуска](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Containerregistry&pageNumber=1&sort=Popular).

## <a name="deploy-the-template"></a>Развертывание шаблона

 1. Выберите следующее изображение, чтобы войти на портал Azure и открыть шаблон.

    [![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-container-registry-geo-replication%2Fazuredeploy.json)

 1. Введите или выберите следующие значения.

    * **Подписка**. Выберите нужную подписку Azure.
    * **Группа ресурсов.** Щелкните **Создать**, введите уникальное имя новой группы ресурсов и щелкните **ОК**.
    * **Регион**. Выберите расположение группы ресурсов. Пример **Центральная часть США.**
    * **Имя записи контроля доступа**. Используйте имя, созданное для реестра, или введите другое имя. Оно должно быть глобально уникальным.
    * **Пользователь с правами администратора ACR включен**. Сохраните значение по умолчанию.
    * **Местоположение**. Примите созданное для основной реплики реестра местоположение или введите другое местоположение, например **Центральная часть США**. 
    * **Номер SKU ACR**. Примите значение по умолчанию.
    * **Acr Replica Location** (Местоположение реплики Acr). Введите местоположения реплики реестра, указав короткое имя региона. Оно должно отличаться от местоположения основного реестра. Пример: **westeurope**.

        :::image type="content" source="media/container-registry-get-started-geo-replication-template/template-properties.png" alt-text="Свойства шаблона":::

1. Нажмите кнопку **Просмотр и создание**, затем ознакомьтесь с условиями. Если вы принимаете их, нажмите кнопку **Создать**.

1. После успешного создания реестра вы получите уведомление:

     :::image type="content" source="media/container-registry-get-started-geo-replication-template/deployment-notification.png" alt-text="Уведомление на портале":::

 Для развертывания шаблона используется портал Azure. Кроме портала Azure, вы можете использовать Azure PowerShell, Azure CLI и REST API. Дополнительные сведения о других методах развертывания см. в статье о [развертывании с использованием шаблонов](../azure-resource-manager/templates/deploy-cli.md).

## <a name="review-deployed-resources"></a>Просмотр развернутых ресурсов

Для просмотра свойств реестра контейнеров используйте портал Azure или такое средство, как Azure CLI.

1. На портале найдите реестры контейнеров и выберите созданный реестр.

1. Запишите значение **Сервер входа** реестра а странице **Обзор**. Используйте этот универсальный код ресурса (URI) в Docker, чтобы отмечать образы и отправлять их в реестр. Дополнительные сведения см. в статье [Отправка первого образа с помощью интерфейса командной строки Docker](container-registry-get-started-docker-cli.md).

    :::image type="content" source="media/container-registry-get-started-geo-replication-template/registry-overview.png" alt-text="Общие сведения о реестре":::

1. На странице **Репликации** проверьте местоположения основной реплики и реплики, добавленной с помощью шаблона. При необходимости добавьте дополнительные реплики на этой странице.

    :::image type="content" source="media/container-registry-get-started-geo-replication-template/registry-replications.png" alt-text="Репликации реестра":::

## <a name="clean-up-resources"></a>Очистка ресурсов

Ставшие ненужными группу ресурсов, реестр и реплику реестра можно удалить. Для этого на портале Azure выберите группу ресурсов, содержащую реестр, и щелкните **Удалить группу ресурсов**.

Удаление группы ресурсов

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве показано, как создать Реестр контейнеров Azure с помощью шаблона ARM и настроить реплику реестра в другом расположении. Чтобы продолжить работу с Реестром контейнеров Azure, перейдите к следующим руководствам.

> [!div class="nextstepaction"]
> [Руководства по использованию Реестра контейнеров Azure](container-registry-tutorial-prepare-registry.md)

Пошаговые инструкции по созданию шаблона см. в следующей статье:

> [!div class="nextstepaction"]
> Создание и развертывание первого шаблона ARM[
