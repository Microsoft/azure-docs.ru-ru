---
title: Предоставление функций с помощью OpenAPI и службы "Управление API Azure"
description: Создание определения OpenAPI, которое позволяет другим приложениям и службам вызывать функцию в Azure.
ms.topic: tutorial
ms.date: 04/21/2020
ms.reviewer: sunayv
ms.custom: devx-track-csharp, mvc, cc996988-fb4f-47, references_regions
ms.openlocfilehash: 9083ff7d8f65c68ce8d173973a4eda650ac355aa
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88212908"
---
# <a name="create-an-openapi-definition-for-a-serverless-api-using-azure-api-management"></a>Создание определения OpenAPI для бессерверного API с помощью Управления API

Интерфейсы REST API часто описываются с использованием определения OpenAPI. Это определение содержит сведения о доступных операция в API и о том, как необходимо структурировать данные запросов и ответов для API.

В этом руководстве вы создадите функцию, определяющую экономичность аварийного ремонта ветроэлектрической установки. Затем создадите определение OpenAPI для приложения-функции с помощью [Управления API Azure](../api-management/api-management-key-concepts.md), чтобы иметь возможность вызвать функцию из приложений и служб.

В этом руководстве описано следующее:

> [!div class="checklist"]
> * создавать функции в Azure;
> * создавать определения OpenAPI с помощью Управления API Azure;
> * проверять определения путем вызова функции.
> * Скачивание определения OpenAPI

## <a name="create-a-function-app"></a>Создание приложения-функции

Для выполнения функций вам понадобится приложение-функция, позволяющее группировать функции в логические единицы и упростить развертывание, масштабирование и совместное использование ресурсов, а также управление ими.

[!INCLUDE [Create function app Azure portal](../../includes/functions-create-function-app-portal.md)]

## <a name="create-the-function"></a>Создание функции

В этом руководстве используется функция, активируемая HTTP, которая принимает два параметра:

* предполагаемое время ремонта турбин (в часах);
* производительность турбин (в киловаттах). 

Затем функция вычисляет стоимость ремонта и доход от турбины за 24 часа. Чтобы создать функцию, активируемую HTTP, на [портале Azure](https://portal.azure.com), выполните следующие действия.

1. В приложении-функции в меню слева выберите **Функции**, а затем в меню вверху выберите **Добавить**.

1. В окне **Новая функция** выберите **Триггер HTTP**.

1. В поле **Новая функция** введите значение `TurbineRepair`. 

1. Из раскрывающегося списка **[Уровень авторизации](functions-bindings-http-webhook-trigger.md#http-auth)** выберите **Функция**, а затем щелкните **Создать функцию**.

    :::image type="content" source="media/functions-openapi-definition/select-http-trigger-openapi.png" alt-text="Создание функции HTTP для OpenAPI":::

1. Выберите **Code + Test** (Код и тест), а затем в раскрывающемся списке выберите **run.csx**. Замените содержимое файла сценария C# (run.csx) следующим кодом и выберите **Сохранить**.

    ```csharp
    #r "Newtonsoft.Json"
    
    using System.Net;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.Extensions.Primitives;
    using Newtonsoft.Json;
    
    const double revenuePerkW = 0.12;
    const double technicianCost = 250;
    const double turbineCost = 100;
    
    public static async Task<IActionResult> Run(HttpRequest req, ILogger log)
    {
        // Get query strings if they exist
        int tempVal;
        int? hours = Int32.TryParse(req.Query["hours"], out tempVal) ? tempVal : (int?)null;
        int? capacity = Int32.TryParse(req.Query["capacity"], out tempVal) ? tempVal : (int?)null;
    
        // Get request body
        string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
        dynamic data = JsonConvert.DeserializeObject(requestBody);
    
        // Use request body if a query was not sent
        capacity = capacity ?? data?.capacity;
        hours = hours ?? data?.hours;
    
        // Return bad request if capacity or hours are not passed in
        if (capacity == null || hours == null){
            return new BadRequestObjectResult("Please pass capacity and hours on the query string or in the request body");
        }
        // Formulas to calculate revenue and cost
        double? revenueOpportunity = capacity * revenuePerkW * 24;  
        double? costToFix = (hours * technicianCost) +  turbineCost;
        string repairTurbine;
    
        if (revenueOpportunity > costToFix){
            repairTurbine = "Yes";
        }
        else {
            repairTurbine = "No";
        };
    
        return (ActionResult)new OkObjectResult(new{
            message = repairTurbine,
            revenueOpportunity = "$"+ revenueOpportunity,
            costToFix = "$"+ costToFix
        });
    }
    ```

    Этот код функции возвращает сообщение `Yes` или `No`, чтобы указать, является ли аварийный ремонт экономично выгодным. Он также сообщает возможность получения дохода, которую предоставляет турбина, и стоимость ремонта этой турбины.

1. Чтобы протестировать функцию, выберите **Тест**, перейдите на вкладку **Входные данные**, введите следующие входные данные в поле **Текст** и щелкните **Запустить**.

    ```json
    {
    "hours": "6",
    "capacity": "2500"
    }
    ```

    :::image type="content" source="media/functions-openapi-definition/test-function.png" alt-text="Тестирование функции на портале Azure":::

    На вкладке **Выходные данные** возвращается результат следующего вида:

    ```json
    {"message":"Yes","revenueOpportunity":"$7200","costToFix":"$1600"}
    ```

Теперь у вас есть функция, определяющая экономичность аварийного ремонта. Далее необходимо создать определение OpenAPI для приложения-функции.

## <a name="generate-the-openapi-definition"></a>Создание определения OpenAPI

Чтобы создать определение OpenAPI, выполните следующее:

1. Выберите приложение-функцию, затем щелкните **Управление API** в меню слева и выберите **Создать** в разделе **Управление API**.

    :::image type="content" source="media/functions-openapi-definition/select-all-settings-openapi.png" alt-text="Выбор раздела &quot;Управление API&quot;":::


1. В разделе "Управление API" настройте параметры, как указано в следующей таблице:

    | Параметр      | Рекомендуемое значение  | Описание                                        |
    | ------------ |  ------- | -------------------------------------------------- |
    | **имя**; | Глобально уникальное имя | Имя создается на основе имени приложения-функции. |
    | **Подписка** | Ваша подписка | Подписка, в которой создан ресурс. |  
    | **[Группа ресурсов](../azure-resource-manager/management/overview.md)** |  myResourceGroup | Та же группа, в которую входит ваше приложение-функция. Значение должно быть установлено. |
    | **Расположение** | западная часть США | Для расположения выберите западную часть США. |
    | **Название организации** | Contoso | Название организации, используемое на портале разработчика и для уведомлений по электронной почте. |
    | **Адрес электронной почты администратора** | Ваш адрес электронной почты | Адрес электронной почты, на которую отправляются системные уведомления от службы "Управление API". |
    | **Ценовая категория** | Потребление | Уровень потребления доступен не во всех регионах. Подробные сведения о ценах на Управление API см. на [этой странице](https://azure.microsoft.com/pricing/details/api-management/). |

    ![Создание службы "Управление API"](media/functions-openapi-definition/new-apim-service-openapi.png)

1. Щелкните **Создать**, чтобы создать экземпляр Управления API. Этот процесс может занять несколько минут.

1. После создания экземпляра Azure открывает на этой странице параметр **Включить Application Insights**. Установите этот флажок, чтобы отправлять журналы в то же расположение, что и из приложения-функции, а затем щелкните **Привязать API**.

1. Откроется страница **Импорт функций Azure** с выделенной функцией **TurbineRepair**. Нажмите кнопку **Выбрать**, чтобы продолжить.

    ![Импорт Функций Azure в Управление API](media/functions-openapi-definition/import-function-openapi.png)

1. На странице **Создание из приложения-функции** подтвердите значения по умолчанию и щелкните **Создать**.

    :::image type="content" source="media/functions-openapi-definition/create-function-openapi.png" alt-text="Создание из приложения-функции":::

    Azure создаст API для выбранной функции.

## <a name="test-the-api"></a>Проверка API

Прежде чем использовать определение OpenAPI, нужно убедиться в том, что API функционирует.

1. На странице приложения-функции выберите **Управление API**, затем перейдите на вкладку **Тест** и щелкните **POST TurbineRepair**. 

1. В поле **Текст запроса** вставьте следующий код:

    ```json
    {
    "hours": "6",
    "capacity": "2500"
    }
    ```

1. Щелкните **Отправить**, а затем просмотрите **HTTP-ответ**.

    :::image type="content" source="media/functions-openapi-definition/test-function-api-openapi.png" alt-text="Тестирование API функции":::

## <a name="download-the-openapi-definition"></a>Скачивание определения OpenAPI

Если ваш API работает правильно, можно скачать определение OpenAPI.

1. Выберите **Скачать определение OpenAPI** в верхней части страницы.
   
   ![Скачивание определения OpenAPI](media/functions-openapi-definition/download-definition.png)

2. Сохраните скачанный JSON-файл и откройте его в любом редакторе. Изучите определение.

[!INCLUDE [clean-up-section-portal](../../includes/clean-up-section-portal.md)]

## <a name="next-steps"></a>Дальнейшие действия

Вы использовали интеграцию со службой "Управление API", чтобы создать определение OpenAPI для функций. Теперь вы можете изменить определение в службе "Управление API" на портале. См. [дополнительные сведения о службе "Управление API"](../api-management/api-management-key-concepts.md).

> [!div class="nextstepaction"]
> [Изменение определения OpenAPI в службе "Управление API"](../api-management/edit-api.md)
