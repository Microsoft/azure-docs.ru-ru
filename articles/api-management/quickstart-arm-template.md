---
title: Краткое руководство. Создание экземпляра службы "Управление API Azure" с помощью шаблона Resource Manager
description: Узнайте, как создать экземпляр службы "Управление API Azure" Azure на уровне "Разработка" с помощью шаблона Azure Resource Manager (шаблона ARM).
services: azure-resource-manager
author: dlepow
ms.service: azure-resource-manager
ms.topic: quickstart
ms.custom: subject-armqs
ms.author: danlep
ms.date: 10/09/2020
ms.openlocfilehash: 1610c51b613712c06410247e2eb673a50a9988b3
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92792248"
---
# <a name="quickstart-create-a-new-azure-api-management-service-instance-using-an-arm-template"></a>Краткое руководство. Создание экземпляра службы "Управление API Azure" с помощью шаблона ARM

В этом руководстве объясняется, как создать экземпляр службы "Управление API Azure" (APIM) с использованием шаблона Azure Resource Manager (ARM). С помощью APIM организации могут публиковать интерфейсы API для внешних и внутренних разработчиков, а также партнеров, что позволяет раскрыть потенциал их данных и служб. Служба управления API предоставляет базовые возможности для успешного выполнения программы API за счет привлечения разработчиков, бизнес-аналитики, анализа, безопасности и защищенности. Служба "Управление API" позволяет создавать современные шлюзы API для существующих серверных служб, размещенных в любом месте, и управлять ими. Дополнительные сведения см. в статье [Общие сведения о Службе контейнеров Azure](api-management-key-concepts.md).

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Если среда соответствует предварительным требованиям и вы знакомы с использованием шаблонов ARM, нажмите кнопку **Развертывание в Azure**. Шаблон откроется на портале Azure.

[![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-azure-api-management-create%2Fazuredeploy.json)

## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="review-the-template"></a>Изучение шаблона

Шаблон, используемый в этом кратком руководстве, взят из [шаблонов быстрого запуска Azure](https://azure.microsoft.com/resources/templates/101-azure-api-management-create/).

:::code language="json" source="~/quickstart-templates/101-azure-api-management-create/azuredeploy.json":::

В этом шаблоне определяется следующий ресурс.

- **[Microsoft.ApiManagement/service](/azure/templates/microsoft.apimanagement/service)**

Дополнительные примеры шаблонов APIM можно найти в [шаблонах быстрого запуска Azure](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Apimanagement&pageNumber=1&sort=Popular).

## <a name="deploy-the-template"></a>Развертывание шаблона

1. Выберите следующее изображение, чтобы войти на портал Azure и открыть шаблон. Шаблон создает экземпляр службы "Управление API" с автоматически созданным именем.

    [![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-azure-api-management-create%2Fazuredeploy.json)

    В этом примере экземпляр настраивается на уровне "Разработка" (экономичный вариант для оценки службы "Управление API Azure"). Этот уровень не предназначен для использования в рабочей среде. Дополнительные сведения о масштабировании категорий службы управления API см. в статье о [повышении категории и масштабировании](upgrade-and-scale.md).

1. Введите или выберите следующие значения.
    - **Подписка**. Выберите нужную подписку Azure.
    - **Группа ресурсов.** Щелкните **Создать**, введите уникальное имя новой группы ресурсов и щелкните **ОК**.
    - **Регион**. Выберите расположение группы ресурсов. Пример **Центральная часть США.**
    - **Электронная почта издателя**. Введите адрес электронной почты для получения уведомлений.
    - **Имя издателя**. Введите имя, выбранное для издателя API.
    - **SKU**. Примите значение по умолчанию **Разработка**.
    - **Sku Count** (Число SKU). Примите значение по умолчанию.
    - **Расположение**. Примите созданное расположение для службы "Управление API".

    :::image type="content" source="media/quickstart-arm-template/create-instance-template.png" alt-text="Свойства шаблона Управление API":::

1. Нажмите кнопку **Просмотр и создание**, затем ознакомьтесь с условиями. Если вы принимаете их, нажмите кнопку **Создать**.

    > [!TIP]
    >  Обычно создание и активация службы "Управление API" на уровне "Разработка" занимает от 30 до 40 минут.

1. После успешного создания экземпляра вы получите уведомление.

    :::image type="content" source="media/quickstart-arm-template/deployment-notification.png" alt-text="Развертывание уведомления":::

 Для развертывания шаблона используется портал Azure. В дополнение к порталу Azure можно также использовать Azure PowerShell, Azure CLI и REST API. Дополнительные сведения о других методах развертывания см. в статье о [развертывании с использованием шаблонов](../azure-resource-manager/templates/deploy-cli.md).

## <a name="review-deployed-resources"></a>Просмотр развернутых ресурсов

Используйте портал Azure, чтобы проверить развернутые ресурсы, либо используйте средства, такие как Azure CLI или Azure PowerShell, чтобы получить список развернутых ресурсов.

1. На [портале Azure](https://portal.azure.com) найдите и выберите **Службы управления API**, а затем — созданный экземпляр службы.
1. Проверьте свойства службы на странице **Обзор**.

:::image type="content" source="media/quickstart-arm-template/service-instance-created.png" alt-text="Страница обзора службы":::

Когда экземпляр службы "Управление API" находится в сети, его можно использовать. Изучите сведения из учебника, чтобы [импортировать и опубликовать первый API](import-and-publish.md).

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы планируете продолжить работу с последующими учебниками, можно не удалять экземпляр APIM. Удалите ненужную группу ресурсов. Ресурсы в ней будут также удалены.

1. На [портале Azure](https://portal.azure.com) найдите и выберите элемент **Группы ресурсов**. Можно также выбрать **Группы ресурсов** на **домашней странице**.
1. На странице **Группы ресурсов** выберите свою группу ресурсов.
1. На странице группы ресурсов выберите **Удалить группу ресурсов**.

    Удаление группы ресурсов
1. Введите имя группы ресурсов, которую необходимо удалить, и щелкните **Удалить**.

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Учебник. Импорт и публикация первого API](import-and-publish.md)
