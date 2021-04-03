---
title: Отправка событий хранилища BLOB-объектов в конечную веб-точку с помощью шаблона
description: Использование шаблона службы "Сетка событий Azure" и Resource Manager для создания учетной записи хранения BLOB-объектов и подписки на связанные события. Отправка событий в веб-перехватчик.'
ms.date: 07/07/2020
ms.topic: quickstart
ms.custom: subject-armqs
ms.openlocfilehash: bfaee324f3e46f64fd4ad0d8b7e1240331b56c27
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92093656"
---
# <a name="quickstart-route-blob-storage-events-to-web-endpoint-by-using-an-arm-template"></a>Краткое руководство. Маршрутизация событий хранилища больших двоичных объектов в конечную веб-точку с помощью шаблона ARM

"Сетка событий Azure" — это служба обработки событий для облака. В этой статье описано, как с помощью шаблона Resource Manager (ARM) создать учетную запись хранения BLOB-объектов и подписку на связанные события этого хранилища больших двоичных объектов, а также активировать событие для просмотра результата. Как правило, события отправляются на конечную точку, которая обрабатывает данные событий и выполняет соответствующие действия. Но в этой статье для простоты события отправляются в веб-приложение, которое собирает и отображает сообщения.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Если среда соответствует предварительным требованиям и вы знакомы с использованием шаблонов ARM, нажмите кнопку **Развертывание в Azure**. Шаблон откроется на портале Azure.

[![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-event-grid-subscription-and-storage%2Fazuredeploy.json)

## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/), прежде чем начинать работу.

### <a name="create-a-message-endpoint"></a>Создание конечной точки сообщения

Перед созданием подписки на события хранилища BLOB-объектов необходимо создать конечную точку для сообщения о событии. Обычно конечная точка выполняет действия на основе данных событий. Чтобы упростить работу с этим руководством, разверните [готовое веб-приложение](https://github.com/Azure-Samples/azure-event-grid-viewer), которое отображает сообщения о событиях. Развернутое решение содержит план службы приложений, веб-приложение службы приложений и исходный код из GitHub.

1. Выберите **Развернуть в Azure**, чтобы развернуть решение в своей подписке. На портале Azure укажите значения остальных параметров.

    [Развертывание в Azure](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2Fazure-event-grid-viewer%2Fmaster%2Fazuredeploy.json)
1. Завершение развертывания может занять несколько минут. Когда развертывание успешно завершится, откройте веб-приложение и убедитесь, что оно работает. Откройте браузер и перейдите по адресу `https://<your-site-name>.azurewebsites.net`.

1. Вы увидите сайт без опубликованных событий.

   ![Представление нового сайта](./media/blob-event-quickstart-portal/view-site.png)

## <a name="review-the-template"></a>Изучение шаблона

Шаблон, используемый в этом кратком руководстве, взят из [шаблонов быстрого запуска Azure](https://azure.microsoft.com/resources/templates/101-event-grid-subscription-and-storage/).

:::code language="json" source="~/quickstart-templates/101-event-grid-subscription-and-storage/azuredeploy.json":::

В шаблоне определено два ресурса Azure:

* [**Microsoft.Storage/storageAccounts**](/azure/templates/microsoft.storage/storageaccounts) создает учетную запись хранения.
* [**Microsoft.EventGrid/systemTopics**](/azure/templates/microsoft.eventgrid/systemtopics) создает системный раздел с указанным именем для учетной записи хранения.
* [**Microsoft.EventGrid/systemTopics/eventSubscriptions**](/azure/templates/microsoft.eventgrid/systemtopics/eventsubscriptions) создает подписку на Сетку событий Azure для системного раздела.

## <a name="deploy-the-template"></a>Развертывание шаблона

1. Выберите следующую ссылку, чтобы войти на портал Azure и открыть шаблон. Шаблон создает хранилище ключей и секрет.

    [![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-event-grid-subscription-and-storage%2Fazuredeploy.json)

2. Укажите **конечную точку**: предоставьте URL-адрес веб-приложения и добавьте `api/updates` к URL-адресу домашней страницы.
3. Щелкните **Приобрести**, чтобы развернуть шаблон.

  Для развертывания шаблона здесь используется портал Azure. Вы можете также использовать Azure PowerShell, Azure CLI и REST API. Дополнительные сведения о других методах развертывания см. в статье о [развертывании с использованием шаблонов](../azure-resource-manager/templates/deploy-powershell.md).

> [!NOTE]
> Дополнительные примеры шаблонов службы "Сетка событий Azure" можно найти [здесь](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Eventgrid&pageNumber=1&sort=Popular).

## <a name="validate-the-deployment"></a>Проверка развертывания

Теперь снова откройте веб-приложение и убедитесь, что оно успешно получило отправленное событие подтверждения подписки. Щелкните значок с изображением глаза, чтобы развернуть данные события. Сетка событий отправляет событие подтверждения, чтобы конечная точка могла подтвердить, что она готова получать данные события. Веб-приложение содержит код для проверки подписки.

![Просмотр события подписки](./media/blob-event-quickstart-portal/view-subscription-event.png)

Теперь необходимо активировать событие, чтобы увидеть, как Сетка событий Azure распределяет сообщение к вашей конечной точке.

Событие для хранилища BLOB-объектов активируется при отправке файла. Определенное содержимое для файла не требуется. В статье предполагается, что у вас есть файл с именем testfile.txt, но вы также можете использовать любой другой файл.

При передаче файла в хранилище BLOB-объектов Azure, служба "Сетка событий" отправляет сообщение в конечную точку, настроенную при подписке. Сообщение имеет формат JSON и содержит массив с одним или несколькими событиями. В следующем примере сообщение JSON содержит массив с одним событием. Просмотрите веб-приложение. Вы увидите полученное событие создания большого двоичного объекта.

![Просмотр результатов](./media/blob-event-quickstart-portal/view-results.png)

## <a name="clean-up-resources"></a>Очистка ресурсов

[Удалите группу ресурсов](../azure-resource-manager/management/delete-resource-group.md?tabs=azure-portal#delete-resource-group
), если она больше не нужна.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о шаблонах Azure Resource Manager см. в перечисленных ниже статьях.

* [Документация по Azure Resource Manager](../azure-resource-manager/index.yml)
* [Define resources in Azure Resource Manager templates](/azure/templates/) (Определение ресурсов в шаблонах Azure Resource Manager)
* [Шаблоны быстрого запуска Azure](https://azure.microsoft.com/resources/templates/)
* [Шаблоны службы "Сетка событий Azure"](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Eventgrid)
