---
title: 'ML Studio (классическая модель): повторное обучение классической веб-службы в Azure'
description: Узнайте, как переучить модель и обновить классическую веб-службу для использования недавно обученной модели в Машинное обучение Azure Studio (классическая модель).
services: machine-learning
ms.service: machine-learning
ms.subservice: studio-classic
ms.topic: how-to
author: peterclu
ms.author: peterlu
ms.custom: seodec18, previous-ms.author=yahajiza, previous-author=YasinMSFT, devx-track-csharp
ms.date: 02/14/2019
ms.openlocfilehash: 90c968ee953e80238775639964cb09a25741b33d
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100517575"
---
# <a name="retrain-and-deploy-a-classic-studio-classic-web-service"></a>Повторное обучение и развертывание классической веб-службы (классическая модель)

**применимо к:** ![ Зеленая галочка. ](../../../includes/media/aml-applies-to-skus/yes.png) Машинное обучение Studio (классическая модель) ![ X, указывающее на отсутствие.](../../../includes/media/aml-applies-to-skus/no.png)[машинное обучение Azure](../overview-what-is-machine-learning-studio.md#ml-studio-classic-vs-azure-machine-learning-studio)  


Повторное обучение моделей машинного обучения является одним из способов обеспечения точности и основывается на самых актуальных доступных данных. В этой статье показано, как переучить классическую веб-службу Studio (классической). Инструкции по переучению новой веб-службы Studio (классической) см [. в этой статье.](retrain-machine-learning-model.md)

## <a name="prerequisites"></a>Предварительные условия

В этой статье предполагается, что у вас уже есть повторно обученный и прогнозный эксперимент. Эти шаги описаны в статье [Переобучение модели Студии машинного обучения Azure](./retrain-machine-learning-model.md). Однако вместо развертывания вашей модели машинного обучения в качестве новой классической веб-службы вы развернете свой прогнозный эксперимент.
     
## <a name="add-a-new-endpoint"></a>Добавление новой конечной точки

Прогнозная веб-служба, которую вы развернули, содержит конечную точку по умолчанию, синхронизированную с исходной обученной моделью обучающего и оценивающего экспериментов. Чтобы обновить веб-службу с помощью новой обученной модели, необходимо создать конечную точку оценки.

Новую конечную точку можно добавить в веб-службу двумя способами:

* Программным образом
* С помощью портала веб-служб Azure.

> [!NOTE]
> Важно добавить конечную точку к прогнозной веб-службе, а не к обучающей. Если вы правильно установили обучающую и прогнозную веб-службы, вы увидите две отдельные веб-службы. Имя прогнозной веб-службы должно заканчиваться на [predictive exp.].
>

### <a name="programmatically-add-an-endpoint"></a>Добавление конечной точки программно

Конечные точки оценки можно добавить с помощью примера кода, приведенного в этом [репозитории GitHub](https://github.com/hning86/azuremlps#add-amlwebserviceendpoint).

### <a name="use-the-azure-web-services-portal-to-add-an-endpoint"></a>Добавление конечной точки с помощью портала веб-служб Azure

1. В Машинное обучение Studio (классическая модель) в колонке навигации слева щелкните веб-службы.
1. В нижней части панели мониторинга веб-службы щелкните **Управление конечными точками Предварительная версия**.
1. Нажмите кнопку **Добавить**.
1. Введите имя и описание новой конечной точки. Выберите уровень ведения журнала и укажите, включать ли образцы данных. Дополнительные сведения о ведении журнала см. в статье [Включение ведения журнала для веб-служб машинное обучение](web-services-logging.md).

## <a name="update-the-added-endpoints-trained-model"></a>Обновление обученной модели добавленной конечной точки

### <a name="retrieve-patch-url"></a>Извлечение URL-адреса PATCH

Выполните следующие действия, чтобы получить правильный URL-адрес PATCH с помощью веб-портала.

1. Войдите на портал [веб-служб машинного обучения Azure](https://services.azureml.net/).
1. Щелкните **Веб-службы** или **Classic Web Services** (Классические веб-службы) в верхней части экрана.
1. Щелкните веб-службу оценки, с которой вы работаете (если вы не изменяли имя веб-службы по умолчанию, оно будет заканчиваться на "[Scoring Exp.]").
1. Щелкните **+ создать**.
1. После добавления конечной точки щелкните ее имя.
1. В поле URL-адреса **исправления** щелкните **API Help** (Справка по API), чтобы открыть страницу справки по исправлению.

> [!NOTE]
> Если вы добавили конечную точку в веб-службу обучения, а не на прогнозную веб-службу, вы получите следующую ошибку при щелчке ссылки " **обновить ресурс** ": "Извините, но эта функция не поддерживается или недоступна в этом контексте. Эта веб-служба не содержит обновляемых ресурсов. We apologize for the inconvenience and are working on improving this workflow" (К сожалению, эта функция не поддерживается или недоступна в данном контексте. Приносим извинения за неудобства. Мы работаем над исправлением этой ошибки).
>

Страница исправления содержит URL-адрес исправления, который нужно использовать, и пример кода, с помощью которого можно выполнить вызов.

![URL-адрес исправления.](./media/retrain-classic/ml-help-page-patch-url.png)

### <a name="update-the-endpoint"></a>Обновление конечной точки

Теперь обученную модель можно использовать для обновления созданной конечной точки оценки.

В следующем примере кода показано, как использовать параметры *BaseLocation*, *RelativeLocation*, *SasBlobToken* и URL-адрес исправления для обновления конечной точки.

```csharp
private async Task OverwriteModel()
{
    var resourceLocations = new
    {
        Resources = new[]
        {
            new
            {
                Name = "Census Model [trained model]",
                Location = new AzureBlobDataReference()
                {
                    BaseLocation = "https://esintussouthsus.blob.core.windows.net/",
                    RelativeLocation = "your endpoint relative location", //from the output, for example: "experimentoutput/8946abfd-79d6-4438-89a9-3e5d109183/8946abfd-79d6-4438-89a9-3e5d109183.ilearner"
                    SasBlobToken = "your endpoint SAS blob token" //from the output, for example: "?sv=2013-08-15&sr=c&sig=37lTTfngRwxCcf94%3D&st=2015-01-30T22%3A53%3A06Z&se=2015-01-31T22%3A58%3A06Z&sp=rl"
                }
            }
        }
    };

    using (var client = new HttpClient())
    {
        client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", apiKey);

        using (var request = new HttpRequestMessage(new HttpMethod("PATCH"), endpointUrl))
        {
            request.Content = new StringContent(JsonConvert.SerializeObject(resourceLocations), System.Text.Encoding.UTF8, "application/json");
            HttpResponseMessage response = await client.SendAsync(request);

            if (!response.IsSuccessStatusCode)
            {
                await WriteFailedResponse(response);
            }

            // Do what you want with a successful response here.
        }
    }
}
```

Параметры *apiKey* и *endpointUrl* для этого вызова можно получить на панели мониторинга конечной точки.

Значение параметра *Name* в *Resources* должно совпадать с именем ресурса сохраненной обученной модели в прогнозном эксперименте. Чтобы получить имя ресурса:

1. Войдите на [портал Azure](https://portal.azure.com).
1. В меню слева щелкните **машинное обучение**.
1. В поле "Имя" выберите рабочую область, а затем щелкните **Веб-службы**.
1. В поле Имя выберите пункт **модель переписи [прогнозная Exp.]**.
1. Щелкните новую добавленную конечную точку.
1. На панели мониторинга конечной точки щелкните **Обновить ресурс**.
1. На странице документации по API обновления ресурсов для веб-службы можно найти **имя ресурса** в разделе **обновляемых ресурсов**.

Если срок действия маркера SAS истекает до завершения обновления конечной точки, необходимо выполнить операцию GET с использованием идентификатора задания, чтобы получить новый маркер.

Если код выполнен успешно, новая конечная точка начнет использовать переобученную модель примерно через 30 секунд.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о том, как управлять веб-службами или отслеживать несколько экспериментов, см. в следующих статьях:

* [Обзор портала веб-служб](manage-new-webservice.md)
* [Управление итерациями эксперимента](manage-experiment-iterations.md)