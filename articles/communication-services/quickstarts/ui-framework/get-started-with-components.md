---
title: Начало работы с базовыми компонентами инфраструктуры пользовательского интерфейса Служб коммуникации Azure
titleSuffix: An Azure Communication Services quickstart
description: В этом кратком руководстве описывается, как начать работу с базовыми компонентами инфраструктуры пользовательского интерфейса.
author: ddematheu2
ms.author: dademath
ms.date: 03/10/2021
ms.topic: quickstart
ms.service: azure-communication-services
ms.openlocfilehash: 336059007cfca40a74ad5a4395c6f9a59215bb58
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105825787"
---
# <a name="quickstart-get-started-with-ui-framework-base-components"></a>Краткое руководство. Начало работы с базовыми компонентами инфраструктуры пользовательского интерфейса

[!INCLUDE [Private Preview Notice](../../includes/private-preview-include.md)]

Приступите к работе со Службами коммуникации Azure, используя инфраструктуру пользовательского интерфейса для быстрой интеграции возможностей коммуникаций в свои приложения. В этом кратком руководстве описывается, как интегрировать в приложение базовые компоненты инфраструктуры пользовательского интерфейса для создания возможностей коммуникаций. 

Компоненты инфраструктуры пользовательского интерфейса бывают двух видов: базовые и составные.

- **Базовые компоненты** представляют собой отдельные возможности коммуникаций. Они являются основными блоками, которые можно использовать для создания сложных возможностей коммуникаций. 
- **Составные компоненты** являются готовыми возможностями для распространенных сценариев коммуникаций, созданными с помощью **базовых компонентов** в качестве основных блоков и упакованных для простой интеграции в приложения.

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
- [Node.js](https://nodejs.org/) версий Active LTS и Maintenance LTS (рекомендуется Node 12).
- Активный ресурс Служб коммуникации. [Создайте ресурс Служб коммуникации.](./../create-communication-resource.md)
- Маркер доступа пользователя для создания экземпляра клиента вызова. Узнайте, как [создать маркер доступа пользователя и обеспечить управление им](./../access-tokens.md).

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

//For Private Preview install tarball

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
| Поставщик| Поставщик пользовательского интерфейса Fluent, позволяющий разработчикам изменять базовые компоненты пользовательского интерфейса Fluent.|
| CallingProvider| Вызывающий поставщик для создания экземпляра вызова. Требуется для добавления дополнительных компонентов.|
| ChatProvider | Поставщик чата для создания экземпляра потока чата. Требуется для добавления дополнительных компонентов.|
| MediaGallery | Базовый компонент, который показывает участников вызова и их потоки удаленного видео. |
| MediaControls | Базовый компонент для управления вызовом, в том числе звуком, видео, демонстрацией экрана. |
| ChatThread | Базовый компонент, отображающий поток чата с индикаторами ввода, уведомлениями о прочтении и т. д. |
| SendBox | Базовый компонент, с помощью которого пользователь может вводить сообщения, которые будут отправлены в присоединенный поток.|

## <a name="initialize-calling-and-chat-providers-using-azure-communication-services-credentials"></a>Инициализация поставщиков вызовов и чата с помощью учетных данных Служб коммуникации Azure

Перейдите в папку `src` в `my-app` и найдите файл `app.js`. Здесь мы удалим приведенный ниже код для инициализации наших поставщиков вызовов и чата. Эти поставщики отвечают за поддержание контекста возможностей вызовов и чата. Вы можете выбрать, какой из них следует использовать, в зависимости от типа создаваемой возможности коммуникации. При необходимости вы также можете использовать оба компонента. Чтобы инициализировать компоненты, вам потребуется маркер доступа, полученный от Служб коммуникации Azure. Дополнительные сведения о получении маркеров доступа см. в статье [Создание маркеров доступа и управление ими](./../access-tokens.md).

> [!NOTE]
> Компоненты не создают маркеры доступа, идентификаторы групп или идентификаторы потоков. Эти элементы предоставляются службами, которые выполняют все необходимые действия для создания этих идентификаторов и их передачи в клиентское приложение. Дополнительные сведения см. в разделе: [Архитектура "клиент — сервер"](./../../concepts/client-and-server-architecture.md).
> 
> Пример: Поставщик чата ожидает, что `userId`, связанный с `token`, который используется для его инициализации, уже присоединен к предоставляемому `threadId`. Если маркер не был присоединен к идентификатору потока, работа поставщика чата завершится ошибкой. Дополнительные сведения о чате см. в статье: [Добавление чата в приложение](./../chat/get-started.md).

Мы будем использовать тему пользовательского интерфейса Fluent, чтобы настроить внешний вид и возможности приложения:

`App.js`
```javascript

import {CallingProvider, ChatProvider} from "@azure/acs-ui-sdk"
import { mergeThemes, teamsTheme } from '@fluentui/react-northstar';
import { Provider } from '@fluentui/react-northstar/dist/commonjs/components/Provider/Provider';
import { svgIconStyles } from '@fluentui/react-northstar/dist/es/themes/teams/components/SvgIcon/svgIconStyles';
import { svgIconVariables } from '@fluentui/react-northstar/dist/es/themes/teams/components/SvgIcon/svgIconVariables';
import * as siteVariables from '@fluentui/react-northstar/dist/es/themes/teams/siteVariables';

const iconTheme = {
  componentStyles: {
    SvgIcon: svgIconStyles
  },
  componentVariables: {
    SvgIcon: svgIconVariables
  },
  siteVariables
};

function App(props) {

  return (
    <Provider theme={mergeThemes(iconTheme, teamsTheme)}>
        <CallingProvider
        displayName={/*Insert Display Name to be used for the user*/}
        groupId={/*Insert GUID for group call to be joined*/}
        token={/*Insert the Azure Communication Services access token*/}
        refreshTokenCallback={/*Optional, Insert refresh token call back function*/}
        >
            // Add Calling Components Here
        </CallingProvider>

        {/*Note: Make sure that the userId associated to the token has been added to the provided threadId*/}

        <ChatProvider
        token={/*Insert the Azure Communication Services access token*/}
        displayName={/*Insert Display Name to be used for the user*/}
        threadId={/*Insert id for group chat thread to be joined*/}
        endpointUrl={/*Insert the environment URL for the Azure Resource used*/}
        refreshTokenCallback={/*Optional, Insert refresh token call back function*/}
        >
            //  Add Chat Components Here
        </ChatProvider>
    </Provider>
  );
}

export default App;

```

После инициализации этот поставщик позволяет вам создать собственный макет с помощью базовых компонентов инфраструктуры пользовательского интерфейса и любой дополнительной логики макета. Поставщик отвечает за инициализацию всей базовой логики и надлежащее подключение разных компонентов. Далее мы будем использовать различные базовые компоненты, предоставляемые инфраструктурой пользовательского интерфейса, для создания возможностей коммуникации. Вы можете настроить макет этих компонентов и добавить любые другие настраиваемые компоненты, которые должны отображаться вместе с ними. 

## <a name="build-ui-framework-calling-component-experiences"></a>Создание возможностей компонентов вызовов для инфраструктуры пользовательского интерфейса

Для вызовов мы используем компоненты `MediaGallery` и `MediaControls`. Дополнительные сведения о них см. в статье [Возможности инфраструктуры пользовательского интерфейса](./../../concepts/ui-framework/ui-sdk-features.md). Чтобы начать, в папке `src` создайте файл с именем `CallingComponents.js`. Здесь мы инициализируем компонент функции, который будет включать наши базовые компоненты, чтобы затем импортировать их в `app.js`. Вы можете создать дополнительный макет и стилизацию для компонентов. 

`CallingComponents.js`
```javascript

import {MediaGallery, MediaControls, MapToCallConfigurationProps, connectFuncsToContext} from "@azure/acs-ui-sdk"

function CallingComponents(props) {

  if (props.isCallInitialized) {props.joinCall()}

  return (
    <div style = {{height: '35rem', width: '30rem', float: 'left'}}>
        <MediaGallery/>
        <MediaControls/>
    </div>
  );
}

export default connectFuncsToContext(CallingComponents, MapToCallConfigurationProps);

```

В конце этого файла мы экспортировали вызывающие компоненты с помощью метода `connectFuncsToContext` из инфраструктуры пользовательского интерфейса, чтобы подключить вызывающие компоненты пользовательского интерфейса к базовому состоянию с помощью функции сопоставления `MapToCallingSetupProps`. Этот метод заполняет все элементы компонента, которые мы затем используем для проверки состояния и присоединения к вызову. С помощью свойства `isCallInitialized` мы проверим, готов ли `CallAgent`, а затем воспользуемся методом `joinCall` для присоединения. Инфраструктура пользовательского интерфейса поддерживает пользовательские функции сопоставления, которые используются в сценариях, где разработчики хотят управлять способом отправки данных в компоненты.

## <a name="build-ui-framework-chat-component-experiences"></a>Создание возможностей компонентов чата для инфраструктуры пользовательского интерфейса

Для чата мы будем использовать компоненты `ChatThread` и `SendBox`. Дополнительные сведения об этих компонентах см. в статье [Возможности инфраструктуры пользовательского интерфейса](./../../concepts/ui-framework/ui-sdk-features.md). Чтобы начать, в папке `src` создайте файл с именем `ChatComponents.js`. Здесь мы инициализируем компонент функции, который будет включать наши базовые компоненты, чтобы затем импортировать их в `app.js`.

`ChatComponents.js`
```javascript

import {ChatThread, SendBox} from '@azure/acs-ui-sdk'

function ChatComponents() {

  return (
    <div style = {{height: '35rem', width: '30rem', float: 'left'}}>
        <ChatThread />
        <SendBox />
    </div >
  );
}

export default ChatComponents;

```

## <a name="add-calling-and-chat-components-to-the-main-application"></a>Добавление компонентов вызовов и чата в основное приложение

В файле `app.js` мы теперь добавим компоненты к `CallingProvider` и `ChatProvider`, как показано ниже.

`App.js`
```javascript

import ChatComponents from './ChatComponents';
import CallingComponents from './CallingComponents';

<Provider ... >
    <CallingProvider .... >
        <CallingComponents/>
    </CallingProvider>

    <ChatProvider .... >
        <ChatComponents />
    </ChatProvider>
</Provider>

```

## <a name="run-quickstart"></a>Запуск кода

Чтобы выполнить приведенный выше код, воспользуйтесь следующей командой:

```console

npm run start   

```

Чтобы полностью проверить возможности, вам потребуется второй клиент с функциями вызовов и чата, который присоединится к вызову и потоку чата. Возможные варианты см. в статьях [Начало работы с примером функции группового вызова](./../../samples/calling-hero-sample.md) и [Начало работы с главным примером функции чата](./../../samples/chat-hero-sample.md).

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы хотите очистить и удалить подписку на Службы коммуникации, вы можете удалить ресурс или группу ресурсов. При этом удаляются все ресурсы, связанные с ней. См. сведения об [очистке ресурсов](../create-communication-resource.md#clean-up-resources).

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Попробовать составные компоненты инфраструктуры пользовательского интерфейса](./get-started-with-composites.md)

Дополнительные сведения см. в следующих ресурсах:
- [Общие сведения об инфраструктуре пользовательского интерфейса](../../concepts/ui-framework/ui-sdk-overview.md)
- [Возможности инфраструктуры пользовательского интерфейса](./../../concepts/ui-framework/ui-sdk-features.md)
