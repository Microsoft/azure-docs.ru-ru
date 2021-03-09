---
title: Руководство по настройке Azure Active Directory B2C с Аркосе Labs
titleSuffix: Azure AD B2C
description: Руководство по настройке Azure Active Directory B2C с Аркосе Labs для обнаружения рискованных и мошеннических пользователей
services: active-directory-b2c
author: gargi-sinha
manager: martinco
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 02/18/2021
ms.author: gasinh
ms.subservice: B2C
ms.openlocfilehash: 46f117b13909c2d9624b88e9f5d9a62c4c646e51
ms.sourcegitcommit: 15d27661c1c03bf84d3974a675c7bd11a0e086e6
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102500298"
---
# <a name="tutorial-configure-arkose-labs-with-azure-active-directory-b2c"></a>Руководство. Настройка Аркосе Labs с помощью Azure Active Directory B2C

В этом примере руководства вы узнаете, как интегрировать проверку подлинности Azure Active Directory (AD) B2C с [Аркосе Labs](https://www.arkoselabs.com/). Аркосе Labs помогают организациям в отношении атак Bot, атак перенаправление и мошеннических учетных записей.  

## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к работе, вам потребуется:

- Подписка Azure. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).

- [Клиент Azure AD B2C](tutorial-create-tenant.md) , связанный с вашей подпиской Azure.

- Учетная запись [Аркосе Labs](https://www.arkoselabs.com/book-a-demo/) .

## <a name="scenario-description"></a>Описание сценария

Интеграция Аркосе Labs включает следующие компоненты:

- **Аркосе Labs** — мошенничество и мошенники для защиты от программы-роботы и других автоматизированных нарушений.

- **Azure AD B2C регистрации пользователя** — процесс регистрации, который будет использовать службу Аркосе Labs. Будет использовать пользовательский HTML и JavaScript, а также соединители API для интеграции со службой Аркосе Labs.

- **Функции Azure** — конечная точка API, размещенная в, которая работает с соединителями API. Этот API отвечает за проверку маркера сеанса Аркосе Labs на стороне сервера.

На следующей схеме показано, как Аркосе Labs интегрируется с Azure AD B2C.

![На рисунке показана схема архитектуры Аркосе Labs](media/partner-arkose-labs/arkose-labs-architecture-diagram.png)

| Шаг  | Описание |
|---|---|
|1     | Пользователь регистрируется и создает учетную запись. Когда пользователь нажмет кнопку "Отправить", появится запрос на принудительное выполнение Аркосе Labs.         |
|2     |  После того как пользователь завершит запрос, Azure AD B2C отправит состояние в Аркосе Labs для создания маркера. |
|3     |  Аркосе Labs создает маркер и отправляет его обратно в Azure AD B2C.   |
|4     |  Azure AD B2C вызывает промежуточный веб-API для передачи формы регистрации.      |
|5     |  Промежуточный веб-API отправляет форму регистрации в Аркосе Lab для проверки маркеров.    |
|6     | Аркосе Лаборатория обрабатывает и отправляет результаты проверки обратно в промежуточный веб-API.|
|7     | Промежуточный веб-API отправляет успех или неудачу в результате запроса Azure AD B2C. |
|8     | Если запрос успешно завершен, в Azure AD B2C отправляется форма регистрации, после чего Azure AD B2C завершает проверку подлинности.|

## <a name="onboard-with-arkose-labs"></a>Подключение с помощью Аркосе Labs

1. Обратитесь в [Аркосе](https://www.arkoselabs.com/book-a-demo/) и создайте учетную запись.

2. После создания учетной записи перейдите к https://dashboard.arkoselabs.com/login  

3. На панели мониторинга перейдите к разделу Параметры сайта, чтобы найти открытый ключ и закрытый ключ. Эти сведения потребуются позже для настройки Azure AD B2C. Значения открытого и закрытого ключей называются `ARKOSE_PUBLIC_KEY` и `ARKOSE_PRIVATE_KEY` в [образце кода](https://github.com/Azure-Samples/active-directory-b2c-node-sign-up-user-flow-arkose).

## <a name="integrate-with-azure-ad-b2c"></a>Интеграция с Azure AD B2C

### <a name="part-1--create-a-arkosesessiontoken-custom-attribute"></a>Часть 1. Создание настраиваемого атрибута Аркосесессионтокен

Чтобы создать настраиваемый атрибут, выполните следующие действия.  

1. Перейдите в **портал Azure**  >  **Azure AD B2C**

2. Выбор **атрибутов пользователя**

3. Выберите **Добавить**.

4. Введите **аркосесессионтокен** в качестве имени атрибута

5. Выберите **Создать**.

Дополнительные сведения о [настраиваемых атрибутах](https://docs.microsoft.com/azure/active-directory-b2c/user-flow-custom-attributes?pivots=b2c-user-flow).

### <a name="part-2---create-a-user-flow"></a>Часть 2. Создание потока пользователя

Поток пользователя может использоваться для **регистрации** и **входа в** систему или только для **регистрации**. Пользовательский поток Аркосе Labs будет отображаться только во время регистрации.

1. Инструкции по созданию потока пользователя см. в этой [статье](https://docs.microsoft.com/azure/active-directory-b2c/tutorial-create-user-flows) . При использовании существующего потока пользователя он должен иметь рекомендованный тип версии **(Предварительная версия нового поколения)** .

2. В параметрах потока пользователя перейдите к **атрибутам пользователя** и выберите утверждение **аркосесессионтокен** .

![На рисунке показано, как выбрать настраиваемые атрибуты](media/partner-arkose-labs/select-custom-attribute.png)

### <a name="part-3---configure-custom-html-javascript-and-page-layouts"></a>Часть 3. Настройка настраиваемых макетов HTML, JavaScript и страниц

Перейдите к указанному [скрипту HTML](https://github.com/Azure-Samples/active-directory-b2c-node-sign-up-user-flow-arkose/blob/main/Assets/selfAsserted.html). Файл содержит шаблон HTML с `<script>` тегами JavaScript, который выполняет три действия:

1. Загрузите сценарий Аркосе Labs, который визуализирует мини-приложение Аркосе Labs и выполняет проверку Аркосе Labs на стороне клиента.

2. Скройте `extension_ArkoseSessionToken` элемент ввода и метку, соответствующие `ArkoseSessionToken` настраиваемому атрибуту, из пользовательского интерфейса, показанного пользователю.

3. Когда пользователь завершает задачу Аркосе Labs, Аркосе Labs проверяет ответ пользователя и создает маркер. Обратный вызов `arkoseCallback` в пользовательском сценарии JavaScript задает значение `extension_ArkoseSessionToken` созданного токена. Это значение будет отправлено в конечную точку API, как описано в следующем разделе.

    Сведения о проверке на стороне клиента Аркосе Labs см. в [этой статье](https://arkoselabs.atlassian.net/wiki/spaces/DG/pages/214176229/Standard+Setup) .

Выполните описанные ниже действия, чтобы использовать пользовательский HTML и JavaScript для пользовательского потока.

1. Измените файл [selfAsserted.html](https://github.com/Azure-Samples/active-directory-b2c-node-sign-up-user-flow-arkose/blob/main/Assets/selfAsserted.html) , чтобы он `<ARKOSE_PUBLIC_KEY>` соответствовал значению, созданному для проверки на стороне клиента, и использовался для загрузки скрипта Аркосе Labs для вашей учетной записи.

2. Разместите страницу HTML на веб-конечной точке с поддержкой общего доступа к ресурсам в разных источниках (CORS). [Создайте учетную запись хранилища BLOB-объектов Azure](https://docs.microsoft.com/azure/storage/common/storage-account-create?toc=%2Fazure%2Fstorage%2Fblobs%2Ftoc.json&tabs=azure-portal) и [Настройте CORS](https://docs.microsoft.com/rest/api/storageservices/cross-origin-resource-sharing--cors--support-for-the-azure-storage-services).

  >[!NOTE]
  >Если у вас есть собственный HTML-код, скопируйте и вставьте `<script>` элементы на страницу HTML.

3. Чтобы настроить макеты страниц, выполните следующие действия.

   а. В портал Azure перейдите к **Azure AD B2C**

   b. Перейдите в раздел " **потоки пользователей** " и выберите пользовательский поток

   c. Выбор **макетов страниц**

   d. Выберите **Макет страницы регистрации локальной учетной записи**

   д) Переключить **использование содержимого настраиваемой страницы** на **"Да"**

   е) Вставьте URI, в котором пользовательский HTML-код **использует содержимое настраиваемой страницы**

   ж. Если вы используете поставщики удостоверений социальных сетей, повторите **шаги e** и **f** для макета **страницы регистрации учетной записи социальных сетей** .

   ![изображение, показывающее макеты страниц](media/partner-arkose-labs/page-layouts.png)

4. В потоке пользователя перейдите в раздел **Свойства** и выберите **включить** применение макета страницы (Предварительная версия) JavaScript. Дополнительные сведения см. в этой [статье](https://docs.microsoft.com/azure/active-directory-b2c/javascript-and-page-layout?pivots=b2c-user-flow) .

### <a name="part-4---create-and-deploy-your-api"></a>Часть 4. Создание и развертывание API

Установите [расширение "функции Azure](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions) " для Visual Studio Code.

>[!Note]
>В действиях, описанных в этом разделе, предполагается, что вы используете Visual Studio Code для развертывания функции Azure. Для развертывания можно также использовать портал Azure, терминал или командную строку или любой другой редактор кода.

#### <a name="run-the-api-locally"></a>Запуск API в локальной среде

1. Перейдите к расширению Azure в Visual Studio Code на панели навигации слева. Выберите папку **локального проекта** , представляющую локальную функцию Azure.

2. Нажмите клавишу **F5** или используйте меню **Отладка**  >  **начать отладку** для запуска отладчика и подключения к узлу функций Azure. Эта команда автоматически использует конфигурацию единственной отладки, созданную функцией Azure.

3. Расширение функции Azure автоматически создает несколько файлов для локальной разработки, устанавливает зависимости и устанавливает средства ядра функции, если они еще не установлены. Эти средства помогают в работе с отладкой.

4. Выходные данные средства ядра функции отображаются на панели **терминала** Visual Studio Code. После запуска узла **Alt + щелчок** локального URL-адреса, показанного в выходных данных, чтобы открыть браузер и запустить функцию. В обозревателе функций Azure щелкните функцию правой кнопкой мыши, чтобы просмотреть URL-адрес локально размещенной функции.

Чтобы повторно развернуть локальный экземпляр во время тестирования, повторите шаги 1 – 4.

#### <a name="add-environment-variables"></a>Добавить переменные среды

Этот пример защищает конечную точку веб-API с помощью [обычной проверки подлинности HTTP](https://tools.ietf.org/html/rfc7617).

Имя пользователя и пароль хранятся в виде переменных среды, а не в составе репозитория. Дополнительные сведения см. [ в разделеlocal.settings.jsв](https://docs.microsoft.com/azure/azure-functions/functions-run-local?tabs=macos%2Ccsharp%2Cbash#local-settings-file) файле.

1. Создание local.settings.jsдля файла в корневой папке

2. Скопируйте приведенный ниже код и вставьте его в файл:

```
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "",
    "FUNCTIONS_WORKER_RUNTIME": "node",
    "BASIC_AUTH_USERNAME": "<USERNAME>",
    "BASIC_AUTH_PASSWORD": "<PASSWORD>",
    "ARKOSE_PRIVATE_KEY": "<ARKOSE_PRIVATE_KEY>",
    "B2C_EXTENSIONS_APP_ID": "<B2C_EXTENSIONS_APP_ID>"
  }
}
```
Значения **BASIC_AUTH_USERNAME** и **BASIC_AUTH_PASSWORD** будут учетными данными, используемыми для проверки подлинности вызова API функции Azure. Выберите нужные значения.

`<ARKOSE_PRIVATE_KEY>`— Это секрет на стороне сервера, созданный в службе Аркосе Labs. Он используется для вызова [API проверки на стороне сервера Аркосе Labs](https://arkoselabs.atlassian.net/wiki/spaces/DG/pages/266214758/Server-Side+Instructions) для проверки значения, `ArkoseSessionToken` созданного внешним интерфейсом.

`<B2C_EXTENSIONS_APP_ID>`— Это идентификатор приложения, используемого Azure AD B2C для хранения настраиваемых атрибутов в каталоге. Этот идентификатор приложения можно найти, перейдя по адресу Регистрация приложений, выполнив поиск B2C-Extensions-App и скопировав идентификатор приложения (клиента) в области **Обзор** . Удалите `-` символы.

![На рисунке показан поиск по идентификатору приложения](media/partner-arkose-labs/search-app-id.png)

#### <a name="deploy-the-application-to-the-web"></a>Развертывание приложения в Интернете

1. Выполните действия, описанные в [этом](https://docs.microsoft.com/azure/javascript/tutorial-vscode-serverless-node-04) руководстве, чтобы развернуть функцию Azure в облаке. Скопируйте URL-адрес конечной точки функции Azure.

2. После развертывания выберите параметр **передать параметры** . Переменные среды будут загружены в [Параметры приложения](https://docs.microsoft.com/azure/azure-functions/functions-develop-vs-code?tabs=csharp#application-settings-in-azure) службы приложений. Эти параметры приложения также можно настраивать или [управлять с помощью портал Azure.](https://docs.microsoft.com/azure/azure-functions/functions-how-to-use-azure-function-app-settings)

Дополнительные сведения о разработке Visual Studio Code для функций Azure см. в [этой статье](https://docs.microsoft.com/azure/azure-functions/functions-develop-vs-code?tabs=csharp#republish-project-files) .

#### <a name="configure-and-enable-the-api-connector"></a>Настройка и включение соединителя API

[Создайте соединитель API](https://docs.microsoft.com/azure/active-directory-b2c/add-api-connector) и включите его для пользовательского потока. Конфигурация соединителя API должна выглядеть следующим образом:

![На рисунке показано, как настроить соединитель API](media/partner-arkose-labs/configure-api-connector.png)

- **URL-адрес конечной точки** — это URL-адрес функции, скопированный ранее при развертывании функции Azure.

- **Имя пользователя и пароль** — это имя пользователя и пароль, которые вы определили как переменные среды ранее.

Чтобы включить соединитель API, в параметрах **соединителя API** для потока пользователя выберите соединитель API, который будет вызываться в **перед созданием шага пользователя** . Это вызовет API, когда пользователь выбирает **CREATE** в потоке регистрации. API выполнит проверку значения на стороне сервера `ArkoseSessionToken` , которое было задано обратным вызовом мини-приложения Аркосе `arkoseCallback` .

![Изображение показывает включение соединителя API](media/partner-arkose-labs/enable-api-connector.png)

## <a name="test-the-user-flow"></a>Тестирование потока пользователя

1. Откройте клиент Azure AD B2C и в разделе политики выберите **потоки пользователя**.

2. Выберите ранее созданный поток пользователя.

3. Выберите **запустить поток пользователя** и выберите параметры:

   а. Приложение: выберите зарегистрированное приложение (пример — JWT)

   b. URL-адрес ответа: выберите URL-адрес перенаправления

   c. Выберите **Выполнить поток пользователя**.

4. Пройдите процесс регистрации и создайте учетную запись.

5. Выйти

6. Пройдите по потоку входа  

7. После нажатия на кнопку **"продолжить"** появится головоломка Аркосе Labs.

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Примеры кодов](https://github.com/Azure-Samples/active-directory-b2c-node-sign-up-user-flow-arkose) для пользовательского потока регистрации Azure AD B2C

- [Пользовательские политики в Azure AD B2C](https://docs.microsoft.com/azure/active-directory-b2c/custom-policy-overview)

- [Приступая к работе с пользовательскими политиками в Azure AD B2C](https://docs.microsoft.com/azure/active-directory-b2c/custom-policy-get-started?tabs=applications)
