---
title: Учебник — создание и публикация продукта в службе управления API Azure
description: В этом учебнике мы покажем, как создать и опубликовать продукт в службе управления API Azure. Как только продукт будет опубликован, разработчики могут начать использовать его интерфейсы API.
author: mikebudzynski
ms.service: api-management
ms.topic: tutorial
ms.date: 02/09/2021
ms.author: apimpm
ms.openlocfilehash: d0420b92fc94e0a1a9c8a4057f419a57a9909223
ms.sourcegitcommit: a67b972d655a5a2d5e909faa2ea0911912f6a828
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "100545162"
---
# <a name="tutorial-create-and-publish-a-product"></a>Руководство по создавать и публиковать продукт;  

В службе управления API Azure [*продукт*](api-management-terminology.md#term-definitions) содержит один или несколько API вместе с квотой и условиями использования. Как только продукт будет опубликован, разработчики могут подписаться на него и начать использовать его интерфейсы API.  

В этом руководстве описано следующее:

> [!div class="checklist"]
> * создавать и публиковать продукт;
> * добавлять API к продукту.

:::image type="content" source="media/api-management-howto-add-products/added-product.png" alt-text="Продукты управления API на портале":::


## <a name="prerequisites"></a>Предварительные требования

+ Ознакомьтесь с [терминологией службы управления API в Azure](api-management-terminology.md).
+ Выполните задачи в кратком руководстве по [созданию экземпляра службы управления API Azure](get-started-create-service-instance.md).
+ Также выполните инструкции из руководства [Импорт и публикация первого API](import-and-publish.md).

## <a name="create-and-publish-a-product"></a>создавать и публиковать продукт;

### <a name="portal"></a>[Портал](#tab/azure-portal)

1. Войдите на портал Azure и перейдите к своему экземпляру службы управления API.
1. На панели навигации слева выберите **Продукты** >  **+ Добавить**.
1.  В окне **Добавить продукт** введите значения, описанные в следующей таблице, чтобы создать продукт.

    :::image type="content" source="media/api-management-howto-add-products/02-create-publish-product-01.png" alt-text="Добавление продукта на портале":::

    | Имя                     | Описание                                                                                                                                                                                                                                                                                                             |
    |--------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
    | Отображаемое имя             | Имя, которое будет отображаться на [портале разработчика](api-management-howto-developer-portal.md).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
    | Описание              | Укажите подробные сведения о продукте, например его назначение, интерфейсы API, к которым он предоставляет доступ, и другую информацию.                                                                                                                                               |
    | Состояние                    | Нажмите кнопку **Опубликовано**, чтобы опубликовать продукт. Прежде чем можно будет вызывать интерфейсы API в продукте, сам продукт должен быть опубликован. По умолчанию новые продукты не опубликованы и видны только группе **Администраторы**.                                                                                      |
    | Требуется подписка    | Укажите, требуется ли оформлять подписку на продукт для его использования.                                                                                                                                                                                                                                   |
    | Запрос утверждения        | Укажите, требуется ли, чтобы администратор рассматривал и принимал или отклонял попытки подписаться на этот продукт. Если флажок не установлен, попытки подписаться будут утверждаться автоматически.                                                                                                                         |
    | Ограничение числа подписок | При необходимости ограничьте число одновременных подписок.                                                                                                                                                                                                                                |
    | Условия использования              | Можно указать условия использования продукта, которые должны принимать подписчики, если они хотят использовать этот продукт.                                                                                                                                                                                                             |
    | Программные интерфейсы                     | Выберите один или несколько API. Вы также можете добавить API после создания продукта. Дополнительные сведения см. в разделе [Добавление API к продукту](#add-apis-to-a-product) далее в этой статье. |

3. Нажмите **Создать**, чтобы создать продукт.

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Чтобы начать работу с Azure CLI:

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

Чтобы создать продукт, выполните команду [az apim product create](/cli/azure/apim/product#az_apim_product_create).

```azurecli
az apim product create --resource-group apim-hello-word-resource-group \
    --product-name "Contoso product" --product-id contoso-product \
    --service-name apim-hello-world --subscription-required true \
    --state published --description "This is a test."
```

Вы можете указать разные значения для этого продукта:

   | Параметр | Описание |
   |-----------|-------------|
   | `--product-name` | Имя, которое будет отображаться на [портале разработчика](api-management-howto-developer-portal.md). |
   | `--description`  | Укажите подробные сведения о продукте, например его назначение, интерфейсы API, к которым он предоставляет доступ, и другую информацию. |
   | `--state`        | Щелкните **Опубликовано**, чтобы опубликовать продукт. Прежде чем можно будет вызывать интерфейсы API в продукте, сам продукт должен быть опубликован. По умолчанию новые продукты не опубликованы и видны только группе **Администраторы**. |
   | `--subscription-required` | Укажите, требуется ли оформлять подписку на продукт для его использования. |
   | `--approval-required` | Укажите, требуется ли, чтобы администратор рассматривал и принимал или отклонял попытки подписаться на этот продукт. Если флажок не установлен, попытки подписаться будут утверждаться автоматически. |
   | `--subscriptions-limit` | При необходимости ограничьте число одновременных подписок.|
   | `--legal-terms`         | Можно указать условия использования продукта, которые должны принимать подписчики, если они хотят использовать этот продукт. |

Чтобы просмотреть текущие продукты, выполните команду [az apim product list](/cli/azure/apim/product#az_apim_product_list).

```azurecli
az apim product list --resource-group apim-hello-word-resource-group \
    --service-name apim-hello-world --output table
```

Чтобы удалить продукт, выполните команду [az apim product delete](/cli/azure/apim/product#az_apim_product_delete):

```azurecli
az apim product delete --product-id contoso-product \
    --resource-group apim-hello-word-resource-group \
    --service-name apim-hello-world --delete-subscriptions true
```

---

### <a name="add-more-configurations"></a>Добавление дополнительных конфигураций

Продолжайте настройку продукта после его сохранения. В экземпляре управления API выберите продукт в окне **Продукты**. Добавление или обновление:

|Элемент   |Описание  |
|---------|---------|
|Параметры     |    Метаданные и состояние продукта     |
|Программные интерфейсы     |  API, связанные с продуктом       |
|[Политики](api-management-howto-policies.md)     |  Политики, применяемые к API продукта      |
|Управление доступом     |  Видимость продукта для разработчиков или гостей       |
|[Подписки](api-management-subscriptions.md)    |    Подписчики продукта     |

## <a name="add-apis-to-a-product"></a>Добавление интерфейсов API в продукт

Продуктами называют ассоциации из одного или нескольких API. Вы можете объединить в них определенное количество API и предлагать их разработчикам через портал разработчика. Во время создания продукта можно добавить один или несколько существующих API. API также можно добавить к продукту позже с помощью страницы **Настройки** для продуктов или при создании самого API.

Чтобы получить доступ к API, разработчикам необходимо сначала подписаться на продукт. После этого они получат ключ подписки, который подходит для любого API в этом продукте. Создавая экземпляр службы управления API, вы автоматически становитесь его администратором. Поэтому вы по умолчанию будете подписаны на все продукты.

### <a name="add-an-api-to-an-existing-product"></a>Добавление API к существующему продукту

### <a name="portal"></a>[Портал](#tab/azure-portal)

1. В области навигации экземпляра управления API слева выберите **Продукты**.
1. Выберите продукт, а затем выберите **API**.
1. Щелкните **+ Добавить**.
1. Выберите один или несколько API, а затем нажмите **Выбрать**.

:::image type="content" source="media/api-management-howto-add-products/02-create-publish-product-02.png" alt-text="Добавление API к существующему продукту":::

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

1. Чтобы просмотреть управляемые API, выполните команду [az apim api list](/cli/azure/apim/api#az_apim_api_list):

   ```azurecli
   az apim api list --resource-group apim-hello-word-resource-group \
       --service-name apim-hello-world --output table
   ```

1. Чтобы добавить API в продукт, выполните команду [az apim product api add](/cli/azure/apim/product/api#az_apim_product_api_add):

   ```azurecli
   az apim product api add --resource-group apim-hello-word-resource-group \
       --api-id demo-conference-api --product-id contoso-product \
       --service-name apim-hello-world
   ```

1. Чтобы проверить добавление, выполните команду [az apim product api list](/cli/azure/apim/product/api#az_apim_product_api_list):

   ```azurecli
   az apim product api list --resource-group apim-hello-word-resource-group \
       --product-id contoso-product --service-name apim-hello-world --output table
   ```

Чтобы удалить API из продукта, выполните команду [az apim product api delete](/cli/azure/apim/product/api#az_apim_product_api_delete) :

```azurecli
az apim product api delete --resource-group apim-hello-word-resource-group \
    --api-id demo-conference-api --product-id contoso-product \
    --service-name apim-hello-world
```

---

> [!TIP]
> Подписку пользователя на продукт с индивидуальными ключами подписки можно создать или обновить с помощью [REST API](/rest/api/apimanagement/2019-12-01/subscription/createorupdate) или команды PowerShell.

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы узнали, как выполнять следующие задачи:

> [!div class="checklist"]
> * создавать и публиковать продукт;
> * добавлять API к продукту.

Перейдите к следующему руководству:

> [!div class="nextstepaction"]
> [Create blank API and mock API responses](mock-api-responses.md) (Создание пустого API и имитация ответов API)
