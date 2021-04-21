---
title: Краткое руководство. Первый запрос на JavaScript
description: В этом кратком руководстве показано, как включить библиотеку Resource Graph для JavaScript и выполнить первый запрос.
ms.date: 01/27/2021
ms.topic: quickstart
ms.custom:
- devx-track-js
- mode-api
ms.openlocfilehash: ef93378acaf2c92cf4f7761345cea326af63d300
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/16/2021
ms.locfileid: "107533047"
---
# <a name="quickstart-run-your-first-resource-graph-query-using-javascript"></a>Краткое руководство. Выполнение первого запроса Resource Graph с помощью JavaScript

В этом кратком руководстве показано, как добавить библиотеки в установку JavaScript. При использовании Azure Resource Graph в первую очередь необходимо инициализировать приложение JavaScript с необходимыми библиотеками.

По завершении в установку JavaScript будут добавлены библиотеки и будет выполнен первый запрос Resource Graph.

## <a name="prerequisites"></a>Предварительные требования

- **Подписка Azure**: Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.

- **Node.js**. [Node.js](https://nodejs.org/) 12 или более поздней версии.

## <a name="application-initialization"></a>Инициализация приложений

Чтобы обеспечить отправку запросов JavaScript к Azure Resource Graph, необходимо настроить среду. Эта настройка подойдет для всех сред, где можно использовать JavaScript, в том числе [Bash в Windows 10](/windows/wsl/install-win10).

1. Инициализируйте новый проект Node.js, выполнив следующую команду.

   ```bash
   npm init -y
   ```

1. Добавьте ссылку на модуль yargs.

   ```bash
   npm install yargs
   ```

1. Добавьте ссылку на модуль Azure Resource Graph.

   ```bash
   npm install @azure/arm-resourcegraph
   ```

1. Добавьте ссылку на библиотеку аутентификации Azure.

   ```bash
   npm install @azure/ms-rest-nodeauth
   ```

   > [!NOTE]
   > Убедитесь в том, что в _package.json_ `@azure/arm-resourcegraph` имеет версию **2.0.0** или выше, а `@azure/ms-rest-nodeauth` — версию **3.0.3** или выше.

## <a name="query-the-resource-graph"></a>Отправка запроса в Resource Graph

1. Создайте файл с именем _index.js_ и введите следующий код.

   ```javascript
   const argv = require("yargs").argv;
   const authenticator = require("@azure/ms-rest-nodeauth");
   const resourceGraph = require("@azure/arm-resourcegraph");

   if (argv.query && argv.subs) {
       const subscriptionList = argv.subs.split(",");

       const query = async () => {
          const credentials = await authenticator.interactiveLogin();
          const client = new resourceGraph.ResourceGraphClient(credentials);
          const result = await client.resources(
             {
                 query: argv.query,
                 subscriptions: subscriptionList,
             },
             { resultFormat: "table" }
          );
          console.log("Records: " + result.totalRecords);
          console.log(result.data);
       };

       query();
   }
   ```

1. Введите следующую команду в окне терминала.

   ```bash
   node index.js --query "Resources | project name, type | limit 5" --subs <YOUR_SUBSCRIPTION_ID_LIST>
   ```

   Обязательно замените заполнитель `<YOUR_SUBSCRIPTION_ID_LIST>` списком идентификаторов подписок Azure с разделителями-запятыми.

   > [!NOTE]
   > Так как этот пример запроса не содержит модификатор сортировки, такой как `order by`, повторное выполнение запроса может каждый раз возвращать разные наборы ресурсов.

1. Измените значение первого параметра на `index.js` и измените запрос, чтобы выполнить сортировку (`order by`) по свойству **Name**. Замените `<YOUR_SUBSCRIPTION_ID_LIST>` идентификатором своей подписки.

   ```bash
   node index.js --query "Resources | project name, type | limit 5 | order by name asc" --subs "<YOUR_SUBSCRIPTION_ID_LIST>"
   ```

   Когда скрипт попытается выполнить аутентификацию, в окне терминала отобразится примерно такое сообщение:

   > Чтобы войти, откройте в браузере страницу https://microsoft.com/devicelogin. Введите код FGB56WJUGK для аутентификации.

   Когда вы пройдете аутентификацию в браузере, скрипт продолжит выполнение.

   > [!NOTE]
   > Как и с первым запросом, выполнение этого запроса несколько раз может получить различные наборы ресурсов для каждого запроса. Важен порядок команд запроса. В этом примере `order by` следует после `limit`. Эта последовательность команд сначала ограничивает результаты запроса, а затем упорядочивает их.

1. Измените значение первого параметра на `index.js`, чтобы сначала выполнить сортировку (`order by`) по свойству **Name**, а затем ограничить вывод (`limit`) пятью первыми результатами. Замените `<YOUR_SUBSCRIPTION_ID_LIST>` идентификатором своей подписки.

   ```bash
   node index.js --query "Resources | project name, type | order by name asc | limit 5" --subs "<YOUR_SUBSCRIPTION_ID_LIST>"
   ```

Повторное выполнение последней версии запроса, если ничего не изменяется в самой среде, возвращает стабильные результаты, отсортированные по свойству **Name** и ограниченные пятью первыми результатами.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы хотите удалить установленные библиотеки из приложения, выполните следующую команду.

```bash
npm uninstall @azure/arm-resourcegraph @azure/ms-rest-nodeauth yargs
```

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве показано, как добавить библиотеки Resource Graph в среду JavaScript и выполнить первый запрос. Чтобы узнать больше о языке Resource Graph, перейдите на страницу сведений о языке запросов.

> [!div class="nextstepaction"]
> [Получите более подробную информацию о языке запросов](./concepts/query-language.md).
