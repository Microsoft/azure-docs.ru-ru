---
title: Создание ресурса Персонализатора
description: В этой статье вы узнаете, как создать ресурс персонализации в портал Azure для каждого цикла обратной связи.
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: how-to
ms.date: 03/26/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: 0c75d917f1abe72af2f4aa56b0f67dbb7bcd24a1
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91303562"
---
# <a name="create-a-personalizer-resource"></a>Создание ресурса персонализации

Ресурс персонализации — это то же самое, что и цикл обучения персонализации. Отдельный ресурс или цикл обучения создается для каждого предметного домена или области содержимого. Не используйте несколько областей содержимого в одном цикле, так как это приведет к путанице в цикле обучения и предложит плохие прогнозы.

Если вы хотите, чтобы Персонализация выберет лучшее содержимое для нескольких областей содержимого веб-страницы, используйте для каждого из них другой цикл обучения.


## <a name="create-a-resource-in-the-azure-portal"></a>Создание ресурса на портале Azure

Создайте ресурс Персонализатора для каждого цикла обратной связи.

1. Войдите на [портал Azure](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesPersonalizer). Предыдущая ссылка ведет на страницу **Создание** Персонализатора.
1. Введите имя службы, выберите подписку, расположение, ценовую категорию и группу ресурсов.

    > [!div class="mx-imgBorder"]
    > ![Используйте портал Azure для создания ресурса персонализации, также называемого циклом обучения.](./media/how-to-create-resource/how-to-create-personalizer-resource-learning-loop.png)

1. Выберите **создать** , чтобы создать ресурс.

1. После развертывания ресурса нажмите кнопку **Переход к ресурсу** , чтобы открыть ресурс персонализации.

1. Выберите страницу **быстрого запуска** для ресурса, а затем скопируйте значения для конечной точки и ключа. Для использования API ранжирования и наград требуется как конечная точка, так и ключ ресурса.

1. Выберите страницу **Конфигурация** для нового ресурса, чтобы [настроить цикл обучения](how-to-settings.md).

## <a name="create-a-resource-with-the-azure-cli"></a>Создание ресурса с Azure CLI

1. Войдите в Azure CLI с помощью следующей команды:

    ```azurecli-interactive
    az login
    ```

1. Создайте группу ресурсов — логическую группировку для управления всеми ресурсами Azure, которые вы собираетесь использовать с ресурсом персонализации.


    ```azurecli-interactive
    az group create \
        --name your-personalizer-resource-group \
        --location westus2
    ```

1. Создайте новый ресурс- _приемник, обучающий цикл_, с помощью следующей команды для существующей группы ресурсов.

    ```azurecli-interactive
    az cognitiveservices account create \
        --name your-personalizer-learning-loop \
        --resource-group your-personalizer-resource-group \
        --kind Personalizer \
        --sku F0 \
        --location westus2 \
        --yes
    ```

    Он возвращает объект JSON, который включает **конечную точку ресурса**.

1. Чтобы получить **ключ ресурса**, используйте следующую команду Azure CLI.

    ```azurecli-interactive
        az cognitiveservices account keys list \
        --name your-personalizer-learning-loop \
        --resource-group your-personalizer-resource-group
    ```

    Для использования API ранжирования и наград требуется как конечная точка, так и ключ ресурса.

## <a name="next-steps"></a>Дальнейшие действия

* [Настройка](how-to-settings.md) Цикл обучения персонализации
