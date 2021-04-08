---
author: clemensv
ms.service: service-bus-relay
ms.topic: include
ms.date: 11/09/2018
ms.author: clemensv
ms.openlocfilehash: 500db9f6d299ea81b1f1b1be864df5d91ba1eae7
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "76021000"
---
1. Войдите на [портал Azure][Azure portal].
1. Выберите **Создать ресурс**. Выберите **Интеграция** > **Ретрансляция**. Если пункт **Ретрансляция** в списке отсутствует, щелкните **Просмотреть все** в правом верхнем углу.
1. Выберите **Создать** и введите имя пространства имен в поле **Имя**. Портал Azure проверит, доступно ли это имя.
1. Выберите подписку Azure, в которой будет создано пространство имен.
1. Выберите существующую [группу ресурсов](../articles/azure-resource-manager/management/manage-resource-groups-portal.md), в которую будет размещено это пространство имен, или создайте новую.  
1. Выберите страну или регион для размещения пространства имен.

    ![Создание пространства имен][create-namespace]

1. Нажмите кнопку **создания**. Портал Azure создаст пространство имен и включит его. Через несколько минут система подготовит к работе ресурсы для вашей учетной записи.

### <a name="get-management-credentials"></a>Получение учетных данных управления

1. Нажмите **Все ресурсы** и выберите созданное имя пространства имен.
1. Выберите **Политики общего доступа**.  
1. В окне **Политики общего доступа** выберите **RootManageSharedAccessKey**.
1. В разделе **SAS Policy: RootManageSharedAccessKey** (Политика SAS: RootManageSharedAccessKey) нажмите кнопку **Копировать** рядом с элементом **Первичная строка подключения**. Строка подключения будет скопирована в буфер обмена для дальнейшего использования. Вставьте на время эти значения в Блокноте или любом другом месте.
1. Повторите предыдущий шаг, скопировав и вставив значение **первичного ключа** во временное расположение для последующего использования.  

    ![Строка подключения][connection-string]

<!--Image references-->

[create-namespace]: ./media/relay-create-namespace-portal/create-namespace-vs2019.png
[connection-info]: ./media/relay-create-namespace-portal/connection-info.png
[connection-string]: ./media/relay-create-namespace-portal/connection-string-vs2019.png
[Azure portal]: https://portal.azure.com
