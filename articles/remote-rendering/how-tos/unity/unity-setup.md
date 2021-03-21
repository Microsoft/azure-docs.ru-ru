---
title: Настройка Удаленной отрисовки для Unity
description: Как инициализировать удаленную визуализацию Azure в проекте Unity
author: jakrams
ms.author: jakras
ms.date: 02/27/2020
ms.topic: how-to
ms.custom: devx-track-csharp
ms.openlocfilehash: 48f01058d8e879a9610e76638215214c059982fa
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99594220"
---
# <a name="set-up-remote-rendering-for-unity"></a>Настройка Удаленной отрисовки для Unity

Чтобы включить удаленную визуализацию Azure (ARR) в Unity, мы предоставляем выделенные методы, которые отвечают определенным аспектам Unity.

## <a name="startup-and-shutdown"></a>Запуск и завершение работы

Чтобы инициализировать удаленную визуализацию, используйте `RemoteManagerUnity` . Этот класс вызывает универсальный интерфейс, `RenderingConnection` но уже реализует сведения, относящиеся к Unity. Например, Unity использует определенную систему координат. При вызове `RemoteManagerUnity.Initialize` будет настроено правильное соглашение. Кроме того, вызов требует предоставления камеры Unity, которая должна использоваться для отображения содержимого, отображаемого удаленно.

```cs
// initialize Azure Remote Rendering for use in Unity:
// it needs to know which camera is used for rendering the scene
RemoteUnityClientInit clientInit = new RemoteUnityClientInit(Camera.main);
RemoteManagerUnity.InitializeManager(clientInit);
```

Для выключения удаленной отрисовки вызовите `RemoteManagerStatic.ShutdownRemoteRendering()` .

После `RenderingSession` создания и выбора в качестве основного сеанса визуализации он должен быть зарегистрирован в `RemoteManagerUnity` :

```cs
RemoteManagerUnity.CurrentSession = ...
```

### <a name="full-example-code"></a>Полный пример кода

В приведенном ниже коде показаны все шаги, необходимые для инициализации удаленной отрисовки Azure в Unity.

```cs
// initialize Remote Rendering
RemoteUnityClientInit clientInit = new RemoteUnityClientInit(Camera.main);
RemoteManagerUnity.InitializeManager(clientInit);

// create a frontend
SessionConfiguration sessionConfig = new SessionConfiguration();
// ... fill out sessionConfig ...
RemoteRenderingClient client = new RemoteRenderingClient(sessionConfig);

// start a session
CreateRenderingSessionResult result = await client.CreateNewRenderingSessionAsync(new RenderingSessionCreationOptions(RenderingSessionVmSize.Standard, 0, 30));
RenderingSession session = result.Session;

// let RemoteManagerUnity know about the session we want to use
RemoteManagerUnity.CurrentSession = session;

await session.ConnectAsync(new RendererInitOptions());

/// When connected, load and modify content

RemoteManagerStatic.ShutdownRemoteRendering();
```

## <a name="convenience-functions"></a>Удобные функции

### <a name="session-state-events"></a>События состояния сеанса

`RemoteManagerUnity.OnSessionUpdate` выдает события при изменении состояния сеанса. подробные сведения см. в документации по коду.

### <a name="arrserviceunity"></a>аррсервицеунити

`ARRServiceUnity` — Это необязательный компонент для упрощения установки и управления сеансами. Он содержит параметры для автоматической прекращения сеанса при выходе или выходе из режима воспроизведения приложения, а также при необходимости автоматически обновлять аренду сеанса. Он кэширует такие данные, как свойства сеанса (см. его `LastProperties` переменную), и предоставляет события для изменений состояния сеанса и ошибок сеанса.

В каждый момент времени не может быть более одного экземпляра `ARRServiceUnity` . Он предназначен для быстрого начала работы путем реализации некоторых распространенных функциональных возможностей. Однако для более крупного приложения может быть предпочтительным выполнить эти действия самостоятельно.

Пример настройки и использования `ARRServiceUnity` см. в разделе Обзор [моделей с удаленным](../../tutorials/unity/view-remote-models/view-remote-models.md)просмотром.

## <a name="next-steps"></a>Дальнейшие действия

* [Установка пакета Удаленной отрисовки для Unity](install-remote-rendering-unity-package.md)
* [Руководство. Просмотр модели удаленной отрисовки](../../tutorials/unity/view-remote-models/view-remote-models.md)
