---
title: Создание экземпляра службы "Управление API Azure" с помощью Visual Studio Code | Документация Майкрософт
description: Создание экземпляра службы "Управление API Azure" с помощью Visual Studio Code.
ms.service: api-management
ms.workload: integration
author: vladvino
ms.author: apimpm
ms.topic: quickstart
ms.date: 09/14/2020
ms.openlocfilehash: 3105b6f34d7ece81e8145fdd9e89568e66360ddb
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101649518"
---
# <a name="quickstart-create-a-new-azure-api-management-service-instance-using-visual-studio-code"></a>Краткое руководство. Создание экземпляра службы "Управление API Azure" с помощью Visual Studio Code

Служба управления API Azure помогает организациям публиковать API-интерфейсы для внешних пользователей, партнеров и собственных разработчиков, раскрывая таким образом потенциал своих данных и услуг. Служба управления API предоставляет базовые возможности для успешного выполнения программы API за счет привлечения разработчиков, бизнес-аналитики, анализа, безопасности и защищенности. Служба управления API позволяет создавать современные шлюзы API для существующих серверных служб, размещенных в любом месте, и управлять ими. Дополнительные сведения см. в разделе [общих сведений](api-management-key-concepts.md).

В этом кратком руководстве описаны шаги по созданию экземпляра службы "Управление API" с помощью *расширения службы "Управление API Azure"* для Visual Studio Code. Кроме того, расширение можно использовать для выполнения общих операций управления в экземпляре службы "Управление API".

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

Убедитесь, что установлены следующие компоненты:

- [Visual Studio Code](https://code.visualstudio.com/)

- [Расширение службы "Управление API Azure" для Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-apimanagement&ssr=false#overview)

## <a name="sign-in-to-azure"></a>Вход в Azure

Запустите Visual Studio Code и откройте расширение Azure. (Если значок Azure не отображается на панели действий, убедитесь, что *расширение* службы "Управление API Azure" включено.)

Выберите **Войти в Azure...** , чтобы открыть окно браузера и войти в учетную запись Майкрософт.

![Вход в Azure из расширения Управления API для VS Code](./media/vscode-create-service-instance/vscode-apim-login.png)

## <a name="create-an-api-management-service"></a>Создание службы управления API

После входа в учетную запись Майкрософт в области обозревателя *Управление API Azure* будут перечислены ваши подписки Azure.

Щелкните правой кнопкой мыши подписку, которую вы хотите использовать, и выберите **Create API Management in Azure** (Создать службу "Управление API" в Azure).

![Мастер создания службы "Управление API" в VS Code](./media/vscode-create-service-instance/vscode-apim-create.png)

В открывшейся области укажите имя нового экземпляра службы "Управление API". Оно должно быть глобально уникальным в пределах Azure и содержать от 1 до 50 буквенно-цифровых символов и (или) дефисов,а также начинаться с буквы и заканчиваться буквенно-цифровым символом.

Будет создан экземпляр службы "Управление API" (и родительская группа ресурсов) с указанным именем. По умолчанию экземпляр создается в регионе *Западная часть США* с SKU *Consumption*.

> [!TIP]
> Если включить вариант **Advanced Creation** (Дополнительное создание) в *параметрах расширения службы "Управление API Azure"* , можно также указать [номер SKU для службы "Управление API"](https://azure.microsoft.com/pricing/details/api-management/), [регион Azure](https://status.azure.com/en-us/status) и [группу ресурсов](../azure-resource-manager/management/overview.md) для развертывания экземпляра службы "Управление API".
>
> Хотя подготовка SKU *Consumption* занимает меньше минуты, чтобы создать другие SKU обычно требуется 30–40 минут.

Теперь можно импортировать и опубликовать первый API. Кроме этого можно выполнять общие операции Управления API в расширении для Visual Studio Code. Дополнительные сведения см. в [этом руководстве](visual-studio-code-tutorial.md).

![Созданный экземпляр службы "Управление API" в области расширения службы "Управление API" для VS Code](./media/vscode-create-service-instance/vscode-apim-instance.png)

## <a name="clean-up-resources"></a>Очистка ресурсов

Удалите экземпляр службы "Управление API", щелкнув правой кнопкой мыши и выбрав пункт **Открыть на портале**, чтобы [удалить службу Управление API](get-started-create-service-instance.md#clean-up-resources) и связанную с ней группу ресурсов.

Кроме того, можно нажать кнопку **Delete API Management** (Удалить службу "Управление API"), чтобы удалить только экземпляр службы "Управление API" (без группы ресурсов).

![Удаление экземпляра службы "Управление API" из VS Code](./media/vscode-create-service-instance/vscode-apim-delete.png)

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Импорт API и управление ими с помощью расширения службы "Управление API"](visual-studio-code-tutorial.md)
