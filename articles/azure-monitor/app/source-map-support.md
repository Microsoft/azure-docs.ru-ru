---
title: Поддержка карт исходного кода для приложений JavaScript — Azure Monitor Application Insights
description: Узнайте, как отправить карты исходного кода в контейнер больших двоичных объектов учетной записи хранения с помощью Application Insights.
ms.topic: conceptual
author: DavidCBerry13
ms.author: daberry
ms.date: 06/23/2020
ms.custom: devx-track-js
ms.openlocfilehash: 79462dcb503e5835254b89f7ebaa7a9ee34946d3
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98933949"
---
# <a name="source-map-support-for-javascript-applications"></a>Поддержка карт исходного кода для приложений JavaScript

Application Insights поддерживает отправку карт исходного кода в контейнер больших двоичных объектов учетной записи хранения.
Карты исходного кода можно использовать для деминификацию стеков вызовов, найденных на странице сведений о сквозной транзакции. Любое исключение, отправленное [пакетом SDK для JavaScript][ApplicationInsights-JS] или [ пакетом SDK дляNode.js][ApplicationInsights-Node.js] , можно унминифиед с картами исходного кода.

![Деминификацию стек вызовов путем связывания с учетной записью хранения](./media/source-map-support/details-unminify.gif)

## <a name="create-a-new-storage-account-and-blob-container"></a>Создание новой учетной записи хранения и контейнера больших двоичных объектов

Если у вас уже есть учетная запись хранения или контейнер больших двоичных объектов, этот шаг можно пропустить.

1. [Создание новой учетной записи хранения][create storage account]
2. [Создайте контейнер больших двоичных объектов][create blob container] в вашей учетной записи хранения. Не забудьте установить для параметра "общий доступ" значение `Private` , чтобы убедиться, что исходные карты не являются общедоступными.

> [!div class="mx-imgBorder"]
>![Для уровня доступа к контейнеру необходимо задать значение "частный"](./media/source-map-support/container-access-level.png)

## <a name="push-your-source-maps-to-your-blob-container"></a>Отправка карт исходного кода в контейнер больших двоичных объектов

Вы должны интегрировать конвейер непрерывного развертывания с учетной записью хранения, настроив его на автоматическую отправку исходных карт в настроенный контейнер больших двоичных объектов.

Карты исходного кода можно отправить в контейнер хранилища BLOB-объектов с той же структурой папок, которая была скомпилирована & развернута с помощью. Распространенным вариантом использования является префикс папки развертывания с ее версией, например `1.2.3/static/js/main.js` . При унминифинг через контейнер больших двоичных объектов Azure `sourcemaps` попытается получить карту источника, расположенную по адресу `sourcemaps/1.2.3/static/js/main.js.map` .

### <a name="upload-source-maps-via-azure-pipelines-recommended"></a>Отправка карт источника с помощью Azure Pipelines (рекомендуется)

Если вы используете Azure Pipelines для непрерывного создания и развертывания приложения, добавьте в конвейер задачу [копирования файлов Azure][azure file copy] для автоматической отправки карт исходного кода.

> [!div class="mx-imgBorder"]
> ![Добавление задачи копирования файлов Azure в конвейер для передачи исходных карт в хранилище BLOB-объектов Azure](./media/source-map-support/azure-file-copy.png)

## <a name="configure-your-application-insights-resource-with-a-source-map-storage-account"></a>Настройка ресурса Application Insights с помощью исходной учетной записи хранения

### <a name="from-the-end-to-end-transaction-details-page"></a>Страница сведений о сквозной транзакции

На вкладке сведения о сквозной транзакции можно щелкнуть *деминификацию* и отобразить запрос на настройку, если ресурс не настроен.

1. На портале просмотрите сведения об исключении минифицированные.
2. Щелкните *деминификацию*
3. Если ресурс не был настроен, появится сообщение с запросом на настройку.

### <a name="from-the-properties-page"></a>На странице «свойства»

Если вы хотите настроить или изменить учетную запись хранения или контейнер больших двоичных объектов, связанный с Application Insightsным ресурсом, это можно сделать, просмотрев вкладку *свойств* ресурса Application Insights.

1. Перейдите на вкладку *Свойства* ресурса Application Insights.
2. Щелкните ссылку *изменить источник больших двоичных объектов для гиперкарты*.
3. Выберите другой контейнер больших двоичных объектов в качестве контейнера карт исходного кода.
4. Нажмите кнопку `Apply`.

> [!div class="mx-imgBorder"]
> ![Перенастройте выбранный контейнер больших двоичных объектов Azure, перейдя в колонку свойств.](./media/source-map-support/reconfigure.png)

## <a name="troubleshooting"></a>Устранение неполадок

### <a name="required-azure-role-based-access-control-azure-rbac-settings-on-your-blob-container"></a>Необходимые параметры управления доступом на основе ролей (Azure RBAC) в контейнере больших двоичных объектов

Любой пользователь на портале, использующий эту функцию, должен быть по крайней мере назначен [модулем чтения данных BLOB-объекта хранилища][storage blob data reader] для контейнера больших двоичных объектов. Эту роль необходимо назначить другим пользователям, которые будут использовать карты исходного кода с помощью этой функции.

> [!NOTE]
> В зависимости от способа создания контейнера это может быть не было автоматически назначено вам или вашей команде.

### <a name="source-map-not-found"></a>Исходная таблица не найдена

1. Убедитесь, что соответствующая исходная таблица отправлена в правильный контейнер больших двоичных объектов.
2. Убедитесь, что исходный файл сопоставления назван после файла JavaScript, к которому он соответствует, с суффиксом `.map` .
    - Например, `/static/js/main.4e2ca5fa.chunk.js` будет искать большой двоичный объект с именем `main.4e2ca5fa.chunk.js.map`
3. Проверьте наличие ошибок в консоли браузера. Включите это в любой запрос в службу поддержки.

## <a name="next-steps"></a>Дальнейшие действия

* [Задача копирования файлов Azure](/azure/devops/pipelines/tasks/deploy/azure-file-copy)


<!-- Remote URLs -->
[create storage account]: ../../storage/common/storage-account-create.md?toc=%2Fazure%2Fstorage%2Fblobs%2Ftoc.json&tabs=azure-portal
[create blob container]: ../../storage/blobs/storage-quickstart-blobs-portal.md
[storage blob data reader]: ../../role-based-access-control/built-in-roles.md#storage-blob-data-reader
[ApplicationInsights-JS]: https://github.com/microsoft/applicationinsights-js
[ApplicationInsights-Node.js]: https://github.com/microsoft/applicationinsights-node.js
[azure file copy]: https://aka.ms/azurefilecopyreadme
