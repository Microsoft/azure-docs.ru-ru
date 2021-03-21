---
title: Пример приложения для доступа к данным анализа коммерческих рынков
description: Используйте пример приложения для создания собственного коммерческого приложения аналитики Marketplace.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: article
author: sayantanroy83
ms.author: sroy
ms.date: 3/08/2021
ms.openlocfilehash: 8fe2301c4d4ed2a582774c65d5dbe68bab416aa3
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102584076"
---
# <a name="sample-application-for-accessing-commercial-marketplace-analytics-data"></a>Пример приложения для доступа к данным анализа коммерческих рынков

Примеры приложений создаются на языках C# и JAVA и доступны на сайте [GitHub](https://github.com/partneranalytics).

- [Пример приложения на C#](https://github.com/partneranalytics/ProgrammaticExportSampleAppISV)
- [Пример приложения JAVA](https://github.com/partneranalytics/ProgrammaticExportSampleAppISV_Java)

Вы можете принять участие в этом примере приложения и создать собственное приложение на любом языке.

Пример приложения позволяет достичь следующих целей:

- Создает маркер Azure Active Directory (Azure AD).
- Получает доступные наборы данных.
- Создает пользовательские запросы.
- Возвращает пользовательские и системные запросы.
- Планирует отчет.

Пример приложения не охватывает метод вызова API для других функций. Однако процесс вызова других API-интерфейсов остается таким же, как описано выше.

## <a name="how-to-run-the-application"></a>Запуск приложения

1. Выполните клонирование репозитория в локальную систему с помощью следующей команды:

    `git clone https://github.com/partneranalytics/ProgrammaticExportSampleAppISV.git`

    > [!NOTE]
    > Дополнительные инструкции см `ProgrammaticExportSampleAppISV/README.md` . в файле в [репозитории](https://github.com/partneranalytics/ProgrammaticExportSampleAppISV.git)GitHub.

1. Чтобы быстро запустить приложение, обновите идентификатор клиента и секрет клиента в **appsettings.Development.jsна**

    :::image type="content" source="./media/analytics-programmatic-access/appsettings-development-json.png" alt-text="Иллюстрирует фрагмент appsettings.Development.jsфайла.":::

При запуске приложения запустится локальный веб-сервер, и откроется страница ( `https://localhost:44365/ProgrammaticExportSampleApp/sample` ).

[![Иллюстрирует страницу отчета по расписанию.](./media/analytics-programmatic-access/schedule-report.png)](./media/analytics-programmatic-access/schedule-report.png#lightbox)

На этой странице будут выполняться вызовы API на веб-сервер, работающий на локальном компьютере, который, в свою очередь, сделает фактические вызовы API программного доступа.

## <a name="code-snippets"></a>Фрагменты кода

Базовая структура кода C# для выполнения вызовов API программного доступа выглядит следующим образом:

:::image type="content" source="./media/analytics-programmatic-access/code-snippets.png" alt-text="Снимок экрана вызовов API.":::

## <a name="next-steps"></a>Дальнейшие действия

- [Интерфейсы API для доступа к данным анализа коммерческих рынков](analytics-available-apis.md)
