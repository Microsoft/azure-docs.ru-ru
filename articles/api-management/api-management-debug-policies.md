---
title: Отладка политик управления API Azure в Visual Studio Code | Документация Майкрософт
description: Узнайте, как выполнять отладку политик управления API Azure с помощью расширения Azure API Management Visual Studio Code.
services: api-management
documentationcenter: ''
author: miaojiang
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 09/22/2020
ms.author: apimpm
ms.openlocfilehash: 2e45d1274cf7332dbca70eaa8fc51f0ac98e5359
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101648022"
---
# <a name="debug-azure-api-management-policies-in-visual-studio-code"></a>Отладка политик управления API Azure в Visual Studio Code

[Политики](api-management-policies.md) в службе управления API Azure предоставляют мощные возможности, которые помогают издателям API устранять перекрестные задачи, такие как проверка подлинности, авторизация, регулирование, кэширование и преобразование. Политика — это коллекция правил, которые выполняются последовательно над запросом или ответом API. 

В этой статье описывается, как выполнить отладку политик управления API с помощью [расширения управления API Azure для Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-apimanagement). 

## <a name="prerequisites"></a>Предварительные условия

* Сначала создайте экземпляр уровня разработчика управления API, следуя этому [краткому](get-started-create-service-instance.md) руководству.

* Установите [Visual Studio Code](https://code.visualstudio.com/) и последнюю версию [расширения управления API Azure для Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-apimanagement). 

* Импортируйте API в экземпляр управления API. Примеры шагов см. [в разделе Учебник. Использование расширения управления API для Visual Studio Code для импорта API-интерфейсов и управления ими](visual-studio-code-tutorial.md).

## <a name="restrictions-and-limitations"></a>Ограничения

Эта функция доступна только на уровне разработчика управления API. Каждый экземпляр управления API поддерживает только один параллельный сеанс отладки.

## <a name="initiate-a-debugging-session"></a>Запуск сеанса отладки

1. Запустите Visual Studio Code.
2. Перейдите к расширению управления API в разделе расширения Azure.
3. Поиск экземпляра управления API для отладки
4. Поиск API и операции для отладки
5. Щелкните операцию правой кнопкой мыши и выберите команду **начать отладку политики** .

На этом этапе расширение попытается инициировать и установить сеанс отладки с помощью шлюза управления API.

![начать отладку](media/api-management-debug-policies/initiate-debugging-session.png)

## <a name="send-a-test-request"></a>Отправить тестовый запрос
При установке сеанса отладки расширение откроет новый редактор, позволяющий создать и отправить в эту операцию тестовый HTTP-запрос с использованием [расширения клиента RESTful](https://marketplace.visualstudio.com/items?itemName=humao.rest-client).

Вы увидите, что заголовок **OCP-Apim-Debug** уже добавлен в запрос. Этот заголовок является обязательным, и значение должно быть задано для ключа подписки уровня службы, доступного для доступа, чтобы активировать функции отладки в шлюзе управления API.

Измените HTTP-запрос в редакторе в соответствии с вашим сценарием тестирования. Затем нажмите кнопку **Отправить запрос** , чтобы отправить тестовый запрос шлюзу управления API.

![отправить тестовый запрос](media/api-management-debug-policies/rest-client.png)

## <a name="debug-policies"></a>Политики отладки
После отправки тестового HTTP-запроса расширение откроет окно Отладка, в котором отобразятся действующие политики этой операции и будет останавливаться на первой действующей политике. 

![политики отладки](media/api-management-debug-policies/main-window.png)

Для выполнения конвейера политики можно выполнить отдельные действия по отдельным политикам или установить точку останова в политике и перейти непосредственно к этой политике. 

На панели **переменные** можно проверять значения созданных системой и созданных пользователем переменных. На панели **точки останова** можно просмотреть список всех установленных точек останова. На панели **Стек вызовов** можно увидеть текущую действующую область политики. 

При возникновении ошибки во время выполнения политики вы увидите подробные сведения об ошибке в политике, где она произошла. 

![исключения](media/api-management-debug-policies/exception.png)

> [!TIP]
> Не забудьте выйти из сеанса отладки, нажав кнопку " **Закрыть** " после завершения.


## <a name="next-steps"></a>Дальнейшие действия

+ Дополнительные сведения о [расширении управления API для Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-apimanagement). 
+ Сообщить о проблемах в [репозитории GitHub](https://github.com/Microsoft/vscode-apimanagement)

