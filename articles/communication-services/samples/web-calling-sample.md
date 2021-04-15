---
title: Службы коммуникации Azure — пример веб-вызова
titleSuffix: An Azure Communication Services sample
description: Сведения о примере веб-вызова Служб коммуникации Azure
author: chriswhilar
manager: mariusu-msft
services: azure-communication-services
ms.author: mariusu
ms.date: 03/10/2021
ms.topic: overview
ms.service: azure-communication-services
ms.openlocfilehash: 35c3bd0213b6d895a81605f5858f924182e1e307
ms.sourcegitcommit: d23602c57d797fb89a470288fcf94c63546b1314
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/01/2021
ms.locfileid: "106168864"
---
# <a name="get-started-with-the-web-calling-sample"></a>Начало работы с примером веб-вызова

Примером веб-вызова является веб-приложение, служащее в качестве пошагового руководства по различным возможностям, которые предоставляет пакет SDK для веб-вызовов Служб коммуникации.

Этот пример был создан для разработчиков и позволяет легко приступить к работе со Службами коммуникации. Его пользовательский интерфейс делится на несколько разделов, каждый из которых содержит кнопку отображения кода, с помощью которой можно копировать код непосредственно из браузера в собственное приложение Служб коммуникации.

## <a name="get-started-with-the-web-calling-sample"></a>Начало работы с примером веб-вызова

> [!IMPORTANT]
> [Этот пример можно найти **на сайте GitHub**.](https://github.com/Azure-Samples/communication-services-web-calling-tutorial/)

Следуйте указаниям в файле /Project/readme.md, чтобы настроить проект и выполнить его локально на компьютере.
Если на компьютере выполняется [пример веб-вызова](https://github.com/Azure-Samples/communication-services-web-calling-tutorial), вы увидите следующую целевую страницу:

:::image type="content" source="./media/web-calling-tutorial-page-1.png" alt-text="Учебник по веб-вызовам 1" lightbox="./media/web-calling-tutorial-page-1.png":::

:::image type="content" source="./media/web-calling-tutorial-page-2.png" alt-text="Учебник по веб-вызовам 2" lightbox="./media/web-calling-tutorial-page-2.png":::

## <a name="user-provisioning-and-sdk-initialization"></a>Подготовка пользователей и инициализация пакета SDK

Щелкните Provisioning user and initialize SDK (Подготовка пользователя и инициализация пакета SDK), чтобы инициализировать пакет SDK с помощью маркера, подготовленного службой подготовки маркеров серверной части. Эта серверная служба находится в `/project/webpack.config.js`.

Нажмите кнопку "Показать код", чтобы просмотреть пример кода, который можно использовать в своем решении.

После инициализации пакета SDK на экране должно отображаться следующее:

:::image type="content" source="./media/user-provisioning.png" alt-text="Подготовка пользователей" lightbox="./media/user-provisioning.png":::

Теперь можно приступить к размещению вызовов с помощью ресурса Служб коммуникации.

## <a name="placing-and-receiving-calls"></a>Размещение и получение вызовов

Пакет SDK для веб-вызовов для Служб коммуникации позволяет использовать режимы **1:1**, **1:N** и **групповые** вызовы.

Для исходящих вызовов 1:1 или 1:N можно указать несколько удостоверений пользователей Служб коммуникации, чтобы осуществить вызов с помощью значений, разделенных запятыми. Вы также можете указать традиционные номера телефонов (ТСОП) для вызова с помощью значений, разделенных запятыми.

При вызове телефонных номеров ТСОП укажите альтернативный идентификатор звонящего. Нажмите кнопку "Позвонить", чтобы совершить исходящий звонок:

:::image type="content" source="./media/place-a-call.png" alt-text="Осуществление вызовов" lightbox="./media/place-a-call.png":::

Чтобы присоединиться к групповому вызову, введите идентификатор GUID, который определяет вызов, и нажмите кнопку "Присоединиться к группе":

:::image type="content" source="./media/join-a-group-call.png" alt-text="Присоединение к групповому вызову" lightbox="./media/join-a-group-call.png":::

Нажмите кнопку "Показать код", чтобы просмотреть пример кода для совершения вызовов, получения вызовов и присоединения к групповым вызовам.

Активный вызов выглядит следующим образом:

:::image type="content" source="./media/group-call.png" alt-text="Групповой вызов" lightbox="./media/group-call.png":::

В примере также есть фрагменты кода для следующих возможностей:

  - Включение или отключение видеокамеры по щелчку значка видео.
  - Включение или отключение микрофона по щелчку значка микрофона.
  - Удержание вызова или прекращение его удержания по щелчку значка воспроизведения.
  - Запуск или отмена демонстрации своего экрана по щелчку иконки экрана.
  - Добавление участника к вызову по щелчку значка пользователя.
  - Удаление определенного участника из вызова по щелчку "Удалить участника" в списке участников.


## <a name="next-steps"></a>Дальнейшие действия

>[!div class="nextstepaction"]
>[Скачайте пример с GitHub.](https://github.com/Azure-Samples/communication-services-web-calling-tutorial/)

Дополнительные сведения см. в следующих статьях:

- Узнайте, как правильно [использовать пакет SDK для вызовов](../quickstarts/voice-video-calling/calling-client-samples.md).
- Узнайте больше о [принципе работы функции вызовов](../concepts/voice-video-calling/about-call-types.md).
- Ознакомьтесь со [справочной документацией по API](/javascript/api/azure-communication-services/@azure/communication-calling/).
- Изучите пример приложения [Contoso MED](https://github.com/Azure-Samples/communication-services-contoso-med-app).

## <a name="additional-reading"></a>Дополнительные материалы

- [Примеры](./overview.md): дополнительные примеры см. на нашей странице обзора примеров.
- [Redux](https://redux.js.org/) — управление состоянием на стороне клиента
- [FluentUI](https://aka.ms/fluent-ui) — библиотека пользовательского интерфейса, поддерживаемая корпорацией Майкрософт
- [React](https://reactjs.org/) — библиотека для создания пользовательских интерфейсов
- [ASP.NET Core](/aspnet/core/introduction-to-aspnet-core?preserve-view=true&view=aspnetcore-3.1) — платформа для создания веб-приложений
