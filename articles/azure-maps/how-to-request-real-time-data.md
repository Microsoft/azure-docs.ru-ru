---
title: Запрос общедоступных транзитных данных в реальном времени с помощью Microsoft Azure Maps службы Mobility Services (Предварительная версия)
description: Узнайте, как запрашивать общедоступные транзитные данные в режиме реального времени, например прибытия на транзитном пути. См. Дополнительные сведения об использовании Azure Maps Mobility Services (Предварительная версия) для этой цели.
author: anastasia-ms
ms.author: v-stharr
ms.date: 12/07/2020
ms.topic: how-to
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: mvc
ms.openlocfilehash: d3e3dc4b0e3bc64a38856da8344583b744ea62b6
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96906052"
---
# <a name="request-real-time-public-transit-data-using-the-azure-maps-mobility-services-preview"></a>Запрос общедоступных транзитных данных в реальном времени с помощью Azure Maps служб Mobility Services (Предварительная версия) 

> [!IMPORTANT]
> Службы Mobility Azure Maps в настоящее время доступны в общедоступной предварительной версии.
> Эта предварительная версия предоставляется без соглашения об уровне обслуживания и не рекомендована для использования рабочей среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены. Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).


В этой статье показано, как использовать [службы Mobility](/rest/api/maps/mobility) Azure Maps для запроса общедоступных транзитных данных в режиме реального времени.

В этой статье вы узнаете, как запросить следующие поступления в реальном времени для всех строк, поступающих на заданную точку.

## <a name="prerequisites"></a>Предварительные условия

Сначала необходимо иметь учетную запись Azure Maps и ключ подписки, чтобы выполнять вызовы Azure Maps общедоступных транзитных API. Для получения сведений следуйте инструкциям в разделе [Создание учетной записи](quick-demo-map-app.md#create-an-azure-maps-account) для создания учетной записи Azure Maps. Выполните действия, описанные в разделе [Получение первичного ключа](quick-demo-map-app.md#get-the-primary-key-for-your-account) , чтобы получить первичный ключ для вашей учетной записи. Дополнительные сведения о проверке подлинности в Azure Maps см. в [этой статье](./how-to-manage-authentication.md).

В этой статье для создания вызовов REST используется [приложение Postman](https://www.getpostman.com/apps). Вы можете использовать любую среду разработки API.

## <a name="request-real-time-arrivals-for-a-stop"></a>Запрос прибытия в режиме реального времени для завершения

Чтобы запросить данные о поступлении конкретного открытого транзитного пути в режиме реального времени, необходимо выполнить запрос к [API прибытия в режиме реального времени](/rest/api/maps/mobility/getrealtimearrivalspreview) в [службе Azure Maps Mobility Service (Предварительная версия)](/rest/api/maps/mobility). Для выполнения запроса потребуются **Метроид** и **стопид** . Чтобы узнать больше о том, как запросить эти параметры, ознакомьтесь с нашим руководством по [запросу общих транзитных маршрутов](./how-to-request-transit-data.md).

Давайте будем использовать "522" в качестве идентификатора Metro, который является ИДЕНТИФИКАТОРом Metro для области "Сиэтл – Tacoma – Бельвью, WA". В качестве идентификатора завершения используйте "522---2060603". Эта шина останавливается в "NE 24 St & 162nd Ave NE, Бельвью WA". Чтобы запросить следующие пять данных о поступлении в реальном времени, для следующих прямых появления при этой ошибке выполните следующие действия.

1. Откройте приложение POST и создадим коллекцию для хранения запросов. В верхней части приложения Postman выберите элемент **Создать**. В окне **Create New** (Создание) выберите **Collection** (Коллекция).  Присвойте имя коллекции и нажмите кнопку **Создать**.

2. Чтобы создать запрос, нажмите кнопку **Создать** еще раз. В окне **Create New** (Создание) выберите **Request** (Запрос). Введите **Имя запроса** для запроса. Выберите коллекцию, созданную на предыдущем шаге, в качестве расположения для сохранения запроса. Затем нажмите кнопку **Сохранить**.

    ![Создание запроса в POST](./media/how-to-request-transit-data/postman-new.png)

3. Выберите метод **получения** HTTP на вкладке Построитель и введите следующий URL-адрес для создания запроса GET. Замените на `{subscription-key}` Azure Maps первичный ключ.

    ```HTTP
    https://atlas.microsoft.com/mobility/realtime/arrivals/json?subscription-key={subscription-key}&api-version=1.0&metroId=522&query=522---2060603&transitType=bus
    ```

4. После успешного выполнения запроса вы получите следующий ответ.  Обратите внимание, что параметр "scheduleType" определяет, основывается ли предполагаемое время прибытия на основе данных в режиме реального времени или статических.

    ```JSON
    {
        "results": [
            {
                "arrivalMinutes": 8,
                "scheduleType": "realTime",
                "patternId": "522---4143196",
                "line": {
                    "lineId": "522---3760143",
                    "lineGroupId": "522---666077",
                    "direction": "backward",
                    "agencyId": "522---5872",
                    "agencyName": "Metro Transit",
                    "lineNumber": "249",
                    "lineDestination": "South Bellevue S Kirkland P&R",
                    "transitType": "Bus"
                },
                "stop": {
                    "stopId": "522---2060603",
                    "stopKey": "71300",
                    "stopName": "NE 24th St & 162nd Ave NE",
                    "stopCode": "71300",
                    "position": {
                        "latitude": 47.631504,
                        "longitude": -122.125275
                    },
                    "mainTransitType": "Bus",
                    "mainAgencyId": "522---5872",
                    "mainAgencyName": "Metro Transit"
                }
            },
            {
                "arrivalMinutes": 25,
                "scheduleType": "realTime",
                "patternId": "522---3510227",
                "line": {
                    "lineId": "522---2756599",
                    "lineGroupId": "522---666063",
                    "direction": "forward",
                    "agencyId": "522---5872",
                    "agencyName": "Metro Transit",
                    "lineNumber": "226",
                    "lineDestination": "Bellevue Transit Center Crossroads",
                    "transitType": "Bus"
                },
                "stop": {
                    "stopId": "522---2060603",
                    "stopKey": "71300",
                    "stopName": "NE 24th St & 162nd Ave NE",
                    "stopCode": "71300",
                    "position": {
                        "latitude": 47.631504,
                        "longitude": -122.125275
                    },
                    "mainTransitType": "Bus",
                    "mainAgencyId": "522---5872",
                    "mainAgencyName": "Metro Transit"
                }
            }
        ]
    }
    ```

## <a name="next-steps"></a>Дальнейшие действия

Узнайте, как запросить транзитные данные с помощью служб Mobility Services (Предварительная версия):

> [!div class="nextstepaction"]
> [Запрос передачи данных](how-to-request-transit-data.md)

Изучите документацию по API Azure Maps Mobility Services (Предварительная версия):

> [!div class="nextstepaction"]
> [Документация по API служб Mobility Services](/rest/api/maps/mobility)