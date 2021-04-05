---
title: Краткое руководство. Ваш первый запрос Ruby
description: В этом кратком руководстве показано, как включить пакет Resource Graph для Ruby и выполнить первый запрос.
ms.date: 01/27/2021
ms.topic: quickstart
ms.openlocfilehash: 41769359eaa1f930d34b0a182f35da38a6f5109d
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98920044"
---
# <a name="quickstart-run-your-first-resource-graph-query-using-ruby"></a>Краткое руководство. Первый запрос Resource Graph с помощью Ruby

При использовании Azure Resource Graph в первую очередь необходимо проверить, установлены ли необходимые пакеты для Ruby. В этом кратком руководстве показано, как добавить пакеты в установку Ruby.

В результате вы добавите пакеты в установку Ruby и выполните запрос Resource Graph.

## <a name="prerequisites"></a>Предварительные требования

- Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.
- Субъект-служба Azure, включая _clientId_ и _clientSecret_.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="create-the-resource-graph-project"></a>Создание проекта Resource Graph

Чтобы можно было отправлять запросы с помощью Ruby в Resource Graph, необходимо добавить соответствующий пакет в `Gemfile`. Этот пакет работает везде, где можно использовать Ruby, в том числе в [Azure Cloud Shell](https://shell.azure.com), [Bash в Windows 10](/windows/wsl/install-win10) или в локальной среде.

1. Убедитесь, что установлена последняя версия Ruby (не ниже **2.7.1**). Если она еще не установлена, скачайте ее на сайте [Ruby-Lang.org](https://www.ruby-lang.org/en/downloads/).

1. В выбранной среде Ruby инициализируйте пакет в новой папке проекта:

   ```bash
   # Initialize a bundle to create a new Gemfile
   bundle init
   ```

1. Обновите `Gemfile` с помощью пакетов, необходимых для Azure Resource Graph. Результаты должны выглядеть примерно как в приведенном примере.

   ```file
   # frozen_string_literal: true

   source "https://rubygems.org"

   git_source(:github) {|repo_name| "https://github.com/#{repo_name}" }

   # gem "rails"
   gem 'azure_mgmt_resourcegraph', '~> 0.17.2'
   ```

1. В папке проекта запустите `bundle install`. Убедитесь, что пакеты установлены с помощью `bundle list`.

1. В той же папке проекта создайте `argQuery.rb` с помощью следующего кода и сохраните обновленный файл:

   ```ruby
   #!/usr/bin/env ruby

   require 'azure_mgmt_resourcegraph'
   ARG = Azure::ResourceGraph::Profiles::Latest::Mgmt

   # Get arguments and set options
   options = {
       tenant_id: ARGV[0],
       client_id: ARGV[1],
       client_secret: ARGV[2],
       subscription_id: ARGV[3]
   }

   # Create Resource Graph client from options
   argClient = ARG::Client.new(options)

   # Create Resource Graph QueryRequest for subscription with query
   query_request = ARGModels::QueryRequest.new.tap do |qr|
       qr.subscriptions = [ARGV[3]]
       qr.query = ARGV[4]
   end

   # Get the resources from Resource Graph
   response = argClient.resources(query_request)

   # Convert data to JSON and output
   puts response.data.to_json
   ```

## <a name="run-your-first-resource-graph-query"></a>Выполните первый запрос график ресурсов

Когда сценарий Ruby сохранен и готов к использованию, можно выполнить простой запрос Resource Graph. Запрос возвращает первые пять ресурсов Azure с указанными значениями **Имя** и **Тип ресурса** для каждого ресурса.

В каждом вызове к `argQuery` есть переменные, которые необходимо заменить собственными значениями:

- `{tenantId}` — замените своим идентификатором клиента.
- `{clientId}` — замените идентификатором клиента своего субъекта-службы.
- `{clientSecret}` — замените секретом клиента своего субъекта-службы.
- `{subscriptionId}` — замените это значение идентификатором своей подписки.

1. Перейдите в папку проекта, в которой были созданы файлы `Gemfile` и `argClient.rb`.

1. Запустите ваш первый запрос графика ресурсов Azure, используя пакеты и метод `resources`:

   ```bash
   ruby argQuery.rb "{tenantId}" "{clientId}" "{clientSecret}" "{subscriptionId}" "Resources | project name, type | limit 5"
   ```

   > [!NOTE]
   > Так как этот пример запроса не меняет сортировку, например `order by`, выполнение этого запроса несколько раз может получить различные наборы ресурсов для каждого запроса.

1. Измените значение последнего параметра на `argQuery.rb` и измените запрос, чтобы выполнить сортировку (`order by`) по свойству **Name**:

   ```bash
   ruby argQuery.rb "{tenantId}" "{clientId}" "{clientSecret}" "{subscriptionId}" "Resources | project name, type | limit 5 | order by name asc"
   ```

   > [!NOTE]
   > Как и с первым запросом, выполнение этого запроса несколько раз может получить различные наборы ресурсов для каждого запроса. Важен порядок команд запроса. В этом примере `order by` следует после `limit`. Эта последовательность команд сначала ограничивает результаты запроса, а затем упорядочивает их.

1. Измените значение последнего параметра на `argQuery.rb`, чтобы сначала выполнить сортировку (`order by`) по свойству **Name**, а затем ограничить вывод (`limit`) первыми пятью результатами:

   ```bash
   ruby argQuery.rb "{tenantId}" "{clientId}" "{clientSecret}" "{subscriptionId}" "Resources | project name, type | order by name asc | limit 5"
   ```

Повторное выполнение последней версии запроса, если ничего не изменяется в самой среде, возвращает стабильные результаты, отсортированные по свойству **Name** и ограниченные пятью первыми результатами.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы хотите удалить установленные пакеты из среды Ruby, это можно сделать с помощью следующей команды.

```bash
# Remove the installed gems from the Ruby environment
gem uninstall azure_mgmt_resourcegraph
```

> [!NOTE]
> Пакет `azure_mgmt_resourcegraph` имеет зависимости, такие как `ms_rest` и `ms_rest_azure`, которые также можно установить в зависимости от среды. Эти пакеты можно удалить, если они больше не нужны.

## <a name="next-steps"></a>Дальнейшие действия

При работе с этим кратким руководством вы добавили пакеты Resource Graph в среду Ruby и выполнили первый запрос. Чтобы узнать больше о языке Resource Graph, перейдите на страницу сведений о языке запросов.

> [!div class="nextstepaction"]
> [Получите более подробную информацию о языке запросов](./concepts/query-language.md).
