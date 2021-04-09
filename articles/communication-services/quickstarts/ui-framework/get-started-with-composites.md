---
title: Начало работы с составными компонентами пакета SDK инфраструктуры пользовательского интерфейса Служб коммуникации Azure
titleSuffix: An Azure Communication Services quickstart
description: В этом кратком руководстве описывается, как начать работу с составными компонентами инфраструктуры пользовательского интерфейса
author: ddematheu2
ms.author: dademath
ms.date: 03/10/2021
ms.topic: quickstart
ms.service: azure-communication-services
ms.openlocfilehash: 7d6b0d01ec3de1cb4ce91b1c81cd8a0ccd73b1d1
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105825804"
---
# <a name="quickstart-get-started-with-ui-framework-composite-components"></a>Краткое руководство. Начало работы с составными компонентами инфраструктуры пользовательского интерфейса

[!INCLUDE [Private Preview Notice](../../includes/private-preview-include.md)]

Приступите к работе со Службами коммуникации Azure, используя инфраструктуру пользовательского интерфейса для быстрой интеграции возможностей коммуникаций в свои приложения. В этом кратком руководстве описывается, как интегрировать в приложение составные компоненты инфраструктуры пользовательского интерфейса, чтобы обеспечить применение функций коммуникаций.

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
- [Node.js](https://nodejs.org/) версий Active LTS и Maintenance LTS (рекомендуется Node 12).
- Активный ресурс Служб коммуникации. [Создайте ресурс Служб коммуникации.](./../create-communication-resource.md)
- Маркер доступа пользователя для создания экземпляра составного компонента вызова. Узнайте, как [создать маркер доступа пользователя и обеспечить управление им](./../access-tokens.md).

## <a name="setting-up"></a>Настройка

Для инфраструктуры пользовательского интерфейса требуется настроить среду React. Дальше описывается, как это сделать. Если у вас уже есть приложение React, вы можете пропустить этот раздел.

### <a name="set-up-react-app"></a>Настройка приложения React

Для работы с этим кратким руководством мы будем использовать шаблон create-react-app. Дополнительные сведения см. в разделе: [Начало работы с React](https://reactjs.org/docs/create-a-new-react-app.html)

```console

npx create-react-app my-app

cd my-app

```

Завершив этот процесс, вы должны увидеть полностью развернутое приложение в папке `my-app`. Для работы с этим кратким руководством мы изменим файлы в папке `src`.

### <a name="install-the-package"></a>Установка пакета

Используйте команду `npm install`, чтобы установить пакет SDK Служб коммуникации Azure для реализации вызовов на JavaScript. Переместите предоставленный архив tarball (закрытая предварительная версия) в каталог my-app.

```console

//Private Preview install tarball

npm install --save ./{path for tarball}

```

Параметр `--save` указывает библиотеку как зависимость в файле пакета **package.json**.

### <a name="run-create-react-app"></a>Выполнение Create React App

Проверим установку Create React App, выполнив следующую команду:

```console

npm run start 

```

## <a name="object-model"></a>Объектная модель

Следующие классы и интерфейсы реализуют некоторые основные функции пакета SDK пользовательского интерфейса Служб коммуникации Azure:

| Имя                                  | Описание                                                  |
| ------------------------------------- | ------------------------------------------------------------ |
| GroupCall | Составной компонент, который отображает процесс группового вызова с коллекцией участников и элементами управления. |
| GroupChat | Составной компонент, который отображает процесс группового чата с беседой чата и входными данными |


## <a name="initialize-group-call-and-group-chat-composite-components"></a>Инициализация составных компонентов группового вызова и группового чата

Перейдите в папку `src` в `my-app` и найдите файл `app.js`. Здесь мы удалим следующий код, чтобы инициализировать составные компоненты для группового чата и вызова. Вы можете выбрать, какой из них следует использовать, в зависимости от типа создаваемой возможности коммуникации. При необходимости вы также можете использовать оба компонента. Чтобы инициализировать компоненты, вам потребуется маркер доступа из Служб коммуникации Azure. Дополнительные сведения о получении маркеров доступа см. в статье [Создание пользовательских маркеров доступа и управление ими](./../access-tokens.md).

> [!NOTE]
> Компоненты не создают маркеры доступа, идентификаторы групп или бесед. Эти элементы предоставляются службами, которые выполняют все необходимые действия для создания этих идентификаторов и их передачи в клиентское приложение. Дополнительные сведения см. в разделе: [Архитектура "клиент — сервер"](./../../concepts/client-and-server-architecture.md).
> 
> Пример: Составной компонент группового чата ожидает, что `userId`, связанный с `token`, который используется для его инициализации, уже присоединен к предоставляемому `threadId`. Если маркер не был присоединен к идентификатору беседы, работа составного компонента группового чата завершится ошибкой. Дополнительные сведения о чате см. в статье: [Начало работы с чатом](./../chat/get-started.md)


`App.js`
```javascript

import {GroupCall, GroupChat} from "@azure/acs-ui-sdk"

function App(){

    return(<>
        {/* Example styling provided, developers can provide their own styling to position and resize components */}
        <div style={{height: "35rem", width: "50rem", float: "left"}}>
            <GroupCall
                displayName={DISPLAY_NAME} //Required, Display name for the user entering the call
                token={TOKEN} // Required, Azure Communication Services access token retrieved from authentication service
                refreshTokenCallback={CALLBACK} //Optional, Callback to refresh the token in case it expires
                groupId={GROUPID} //Required, Id for group call that will be joined. (GUID)
                onEndCall = { () => {
                    //Optional, Action to be performed when the call ends
                }}
            />
        </div>

        {/*Note: Make sure that the userId associated to the token has been added to the provided threadId*/}
        {/* Example styling provided, developers can provide their own styling to position and resize components */}
        <div style={{height: "35rem", width: "30rem", float: "left"}}>
            <GroupChat 
                displayName={DISPLAY_NAME} //Required, Display name for the user entering the call
                token={TOKEN} // Required, Azure Communication Services access token retrieved from authentication service
                threadId={THREADID} //Required, Id for group chat thread that will be joined.
                endpointUrl={ENDPOINT_URL} //Required, URL for Azure endpoint being used for Azure Communication Services
                onRenderAvatar = { (acsId) => {
                    //Optional, function to override the avatar image on the chat thread. Function receives one parameters for the Azure Communication Services Identity. Must return a React element.
                }}
                refreshToken = { () => {
                    //Optional, function to refresh the access token in case it expires
                }}
                options = {{
                    //Optional, options to define chat behavior
                    sendBoxMaxLength: number | undefined //Optional, Limit the max send box length based on viewport size change.
                }}
            />
        </div>
    </>);
}

export default App;

```

## <a name="run-quickstart"></a>Выполнение кода

Чтобы выполнить приведенный выше код, используйте следующую команду:

```console

npm run start 

```

Чтобы полностью проверить возможности, вам потребуется второй клиент с функциями вызовов и чата, который присоединится к вызову и потоку чата. Возможные варианты см. в статьях [Начало работы с примером функции группового вызова](./../../samples/calling-hero-sample.md) и [Начало работы с главным примером функции чата](./../../samples/chat-hero-sample.md).

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы хотите очистить и удалить подписку на Службы коммуникации, вы можете удалить ресурс или группу ресурсов. При этом удаляются все ресурсы, связанные с ней. См. сведения об [очистке ресурсов](../create-communication-resource.md#clean-up-resources).

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Начало работы с базовыми компонентами инфраструктуры пользовательского интерфейса](./get-started-with-components.md)

Дополнительные сведения см. в следующих ресурсах:
- [Общие сведения об инфраструктуре пользовательского интерфейса](../../concepts/ui-framework/ui-sdk-overview.md)
- [Возможности инфраструктуры пользовательского интерфейса](./../../concepts/ui-framework/ui-sdk-features.md)
