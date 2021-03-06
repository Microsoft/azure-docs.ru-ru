---
title: Настройка параметров приложения для Статических веб-приложений Azure
description: Сведения о настройке параметров приложения для Статических веб-приложений Azure
services: static-web-apps
author: burkeholland
ms.service: static-web-apps
ms.topic: how-to
ms.date: 05/08/2020
ms.author: buhollan
ms.custom: devx-track-js
ms.openlocfilehash: 2886cb6433c4f8a65eb0661eded0ddd08248e137
ms.sourcegitcommit: f7eda3db606407f94c6dc6c3316e0651ee5ca37c
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102219188"
---
# <a name="configure-application-settings-for-azure-static-web-apps-preview"></a>Настройка параметров приложения для предварительной версии Статических веб-приложений Azure

Параметры приложения сохраняют параметры конфигурации для значений, которые могут измениться, например строки подключения к базе данных. Добавление параметров приложения позволяет изменять входные данные конфигурации в приложении без необходимости изменения кода приложения.

Параметры приложения:

- шифруются при хранении;
- копируются в [промежуточную](review-publish-pull-requests.md) и рабочую среды.

Параметры приложения иногда также называют переменными среды.

> [!IMPORTANT]
> Параметры приложения, описанные в этой статье, применимы только к API серверной части Статического веб-приложения Azure.
>
> Сведения об использовании переменных сред с интерфейсным веб-приложением см. в документации по [платформе JavaScript](#javascript-frameworks-and-libraries) или [генератору статических сайтов](#static-site-generators).

## <a name="prerequisites"></a>Предварительные требования

- Приложение Статических веб-приложений Azure
- [Azure CLI](/cli/azure/install-azure-cli)

## <a name="types-of-application-settings"></a>Типы параметров приложения

Обычно в приложении Статических веб-приложений Azure есть два аспекта. Первый — это веб-приложение или статическое содержимое, представленное с помощью HTML, CSS, JavaScript и образов. Второй — это API серверной части на базе приложения "Функции Azure".

В этой статье показано, как управлять параметрами приложения для API серверной части в Функциях Azure.

Параметры приложения, описанные в этой статье, нельзя использовать в Статических веб-приложениях. Также нельзя ссылаться на эти параметры. Однако многие интерфейсные платформы и генераторы статических сайтов позволяют использовать переменные среды во время разработки. Во время сборки эти переменные заменяются значениями в созданном коде HTML или JavaScript. Так как данные в HTML и JavaScript легко обнаруживаются посетителями сайта, старайтесь не добавлять конфиденциальную информацию в интерфейсное приложение. Параметры, содержащие конфиденциальные данные, лучше размещать в API-части приложения.

Сведения о том, как использовать переменные среды с платформой или библиотекой JavaScript, см. в следующих статьях:

### <a name="javascript-frameworks-and-libraries"></a>Платформы и библиотеки JavaScript

- [Angular](https://angular.io/guide/build#configuring-application-environments)
- [React](https://create-react-app.dev/docs/adding-custom-environment-variables/)
- [Svelte](https://linguinecode.com/post/how-to-add-environment-variables-to-your-svelte-js-app)
- [Vue](https://cli.vuejs.org/guide/mode-and-env.html)

### <a name="static-site-generators"></a>Генераторы статических сайтов

- [Gatsby](https://www.gatsbyjs.org/docs/environment-variables/);
- [Hugo](https://gohugo.io/getting-started/configuration/);
- [Jekyll](https://jekyllrb.com/docs/configuration/environments/).

## <a name="about-api-app-settings"></a>Сведения о параметрах приложения API

API в Статических веб-приложениях Azure работают на основе Функций Azure, что позволяет определять параметры приложения в файле _local.settings.json_. Этот файл определяет параметры приложения в свойстве `Values` конфигурации.

В следующем примере файла _local.settings.json_ показано, как добавить значение для `DATABASE_CONNECTION_STRING`.

```json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "",
    "FUNCTIONS_WORKER_RUNTIME": "node",
    "DATABASE_CONNECTION_STRING": "<YOUR_DATABASE_CONNECTION_STRING>"
  }
}
```

На параметры, определенные в свойстве `Values`, можно ссылаться из кода как на переменные среды, доступные из объекта `process.env`.

```js
const connectionString = process.env.DATABASE_CONNECTION_STRING;
```

Файл `local.settings.json` не отслеживается репозиторием GitHub, так как конфиденциальная информация, например строки подключения к базам данных, часто включена в состав файла. Так как локальные параметры остаются на компьютере, необходимо вручную отправить параметры в Azure.

Как правило, отправка параметров выполняется редко и не требуется при каждой сборке.

## <a name="uploading-application-settings"></a>Отправка параметров приложения

Параметры приложения можно настроить с помощью портала Azure или Azure CLI.

### <a name="using-the-azure-portal"></a>Использование портала Azure

Портал Azure предоставляет интерфейс для создания, обновления и удаления параметров приложения.

1. Перейдите на [портал Azure](https://portal.azure.com).

1. В строке поиска найдите и выберите **Статические веб-приложения**.

1. На боковой панели щелкните параметр **Конфигурация**.

1. Выберите среду, к которой необходимо применить параметры приложения. Промежуточные среды создаются автоматически при создании запроса на вытягивание и переносятся в рабочую среду после объединения запроса на вытягивание. Вы можете задать параметры приложения для каждой среды.

1. Щелкните **Добавить кнопку**, чтобы добавить новый параметр приложения.

    :::image type="content" source="media/application-settings/configuration.png" alt-text="Представление конфигурации Статических веб-приложений Azure":::

1. Введите **имя** и **значение**.

1. Нажмите кнопку **ОК**.

1. Выберите команду **Сохранить**.

### <a name="using-the-azure-cli"></a>Использование Azure CLI

Вы можете использовать команду `az rest` для выполнения пакетной отправки параметров в Azure. Команда принимает параметры приложения в виде объектов JSON в родительском свойстве с именем `properties`.

Самый простой способ создать JSON-файл с соответствующими значениями — создать измененную версию файла _local.settings.json_.

1. Чтобы не предоставлять к новому файлу с конфиденциальными данными открытый доступ, добавьте следующую запись в файл _.gitignore_.

   ```bash
   local.settings*.json
   ```

2. Затем создайте копию файла _local.settings.json_ и назовите ее _local.settings.properties.json_.

3. В новом файле удалите все данные, за исключением параметров приложения, и переименуйте `Values` в `properties`.

   Теперь файл должен выглядеть как в показанном ниже примере:

   ```json
   {
     "properties": {
       "DATABASE_CONNECTION_STRING": "<YOUR_DATABASE_CONNECTION_STRING>"
     }
   }
   ```

Для выполнения отправки для команды Azure CLI требуется ряд значений, характерных для вашей учетной записи. В окне _Обзор_ ресурса Статических веб-приложений можно получить доступ к следующим сведениям:

1. Имя статического сайта.
2. Имя группы ресурсов
3. Идентификатор подписки

:::image type="content" source="media/application-settings/overview.png" alt-text="Обзор Статических веб-приложений Azure":::

4. В терминале или в командной строке выполните указанную ниже команду. Не забудьте заменить заполнители `<YOUR_STATIC_SITE_NAME>`, `<YOUR_RESOURCE_GROUP_NAME>` и `<YOUR_SUBSCRIPTION_ID>` значениями из окна _Обзор_.

   ```bash
   az rest --method put --headers "Content-Type=application/json" --uri "/subscriptions/<YOUR_SUBSCRIPTION_ID>/resourceGroups/<YOUR_RESOURCE_GROUP_NAME>/providers/Microsoft.Web/staticSites/<YOUR_STATIC_SITE_NAME>/config/functionappsettings?api-version=2019-12-01-preview" --body @local.settings.properties.json
   ```

> [!IMPORTANT]
> Файл local.settings.properties.json должен находиться в том же каталоге, в котором выполняется эта команда. Присвойте этому файлу любое удобное имя. Имя не имеет значения.

### <a name="view-application-settings-with-the-azure-cli"></a>Просмотр параметров приложения с помощью Azure CLI

Параметры приложения доступны для просмотра с помощью Azure CLI.

- В терминале или в командной строке выполните указанную ниже команду. Обязательно замените заполнители `<YOUR_SUBSCRIPTION_ID>`, `<YOUR_RESOURCE_GROUP_NAME>`, `<YOUR_STATIC_SITE_NAME>` вашими значениями.

   ```bash
   az rest --method post --uri "/subscriptions/<YOUR_SUBSCRIPTION_ID>/resourceGroups/<YOUR_RESOURCE_GROUP_NAME>/providers/Microsoft.Web/staticSites/<YOUR_STATIC_SITE_NAME>/listFunctionAppSettings?api-version=2019-12-01-preview"
   ```

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Настройка для локальной разработки](local-development.md)
