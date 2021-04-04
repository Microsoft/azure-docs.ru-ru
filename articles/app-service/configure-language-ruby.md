---
title: Настройка приложений Ruby — Служба приложений Azure
description: Узнайте, как настроить готовый контейнер Ruby для приложения. В этой статье показаны наиболее распространенные задачи настройки.
ms.topic: quickstart
ms.date: 06/18/2020
ms.reviewer: astay; kraigb
ms.custom: mvc, seodec18, devx-track-azurecli
ms.openlocfilehash: 038d62573b491325adc60647debf17fa87e06cfe
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92743689"
---
# <a name="configure-a-linux-ruby-app-for-azure-app-service"></a>Настройка приложения Ruby в Linux для Службы приложений Azure

В этой статье объясняется, как [Служба приложений Azure](overview.md) запускает приложения Ruby в контейнере Linux, а также как вы можете настроить поведение Службы приложений при необходимости. Приложения Ruby нужно развертывать со всеми необходимыми пакетами [gem](https://rubygems.org/gems).

Это руководство содержит основные понятия и инструкции для разработчиков Ruby, которые используют встроенный контейнер Linux в Службе приложений. Если вы раньше не использовали Службу приложений Azure, сначала ознакомьтесь с [кратким руководством по Ruby](quickstart-ruby.md) и [руководством по использованию Ruby с PostgreSQL](tutorial-ruby-postgres-app.md).

## <a name="show-ruby-version"></a>Отображение сведений о версии Ruby

Чтобы отобразились сведения о текущей версии Ruby, выполните следующую команду в [Cloud Shell](https://shell.azure.com):

```azurecli-interactive
az webapp config show --resource-group <resource-group-name> --name <app-name> --query linuxFxVersion
```

Чтобы отобразились сведения обо всех поддерживаемых версиях Ruby, выполните следующую команду в [Cloud Shell](https://shell.azure.com):

```azurecli-interactive
az webapp list-runtimes --linux | grep RUBY
```

Вместо них вы можете запустить любую неподдерживаемую версию Ruby, скомпилировав собственный образ контейнера. Дополнительные сведения см. в статье об [использовании пользовательского образа Docker](tutorial-custom-container.md?pivots=container-linux).

## <a name="set-ruby-version"></a>Настройка версии Ruby

Выполните следующую команду в [Cloud Shell](https://shell.azure.com), чтобы задать для Ruby версию 2.3:

```azurecli-interactive
az webapp config set --resource-group <resource-group-name> --name <app-name> --linux-fx-version "RUBY|2.3"
```

> [!NOTE]
> Во время развертывания вы можете увидеть ошибки следующего вида:
> ```
> Your Ruby version is 2.3.3, but your Gemfile specified 2.3.1
> ```
> или диспетчер конфигурации служб
> ```
> rbenv: version `2.3.1' is not installed
> ```
> Это означает, что настроенная в проекте версия Ruby отличается от версии, которая установлена в выполняемом контейнере (в приведенном выше примере это `2.3.3`). В приведенном выше примере просмотрите *Gemfile* и *.ruby-version* и убедитесь, что версия Ruby не указана или указана та же версия, что и в используемом контейнере (в приведенном выше примере это `2.3.3`).

## <a name="access-environment-variables"></a>Доступ к переменным среды

В Службе приложений можно [задать параметры приложения](configure-common.md#configure-app-settings) вне кода приложения. Затем вы сможете получать к ним доступ, используя стандартный шаблон [ENV['\<path-name>']](https://ruby-doc.org/core-2.3.3/ENV.html). Например, для доступа к параметру приложения с именем `WEBSITE_SITE_NAME` используйте следующий код:

```ruby
ENV['WEBSITE_SITE_NAME']
```

## <a name="customize-deployment"></a>Настройка развертывания

Если вы развертываете [репозиторий Git](deploy-local-git.md) или [ZIP-пакет](deploy-zip.md) с включенными процессами сборки, механизм развертывания (Kudu) по умолчанию автоматически выполняет следующие шаги после развертывания:

1. Проверка наличия *Gemfile*.
1. Выполните `bundle clean`. 
1. Выполните `bundle install --path "vendor/bundle"`.
1. Выполнение `bundle package` для упаковки пакетов в папку vendor/cache.

### <a name="use---without-flag"></a>Использование флага --without

Чтобы выполнить `bundle install` с флагом [--without](https://bundler.io/man/bundle-install.1.html), укажите для [параметра приложения](configure-common.md#configure-app-settings) `BUNDLE_WITHOUT` список групп с разделителями запятыми. Например, приведенная ниже команда позволяет присвоить ему значение `development,test`.

```azurecli-interactive
az webapp config appsettings set --name <app-name> --resource-group <resource-group-name> --settings BUNDLE_WITHOUT="development,test"
```

Если этот параметр определен, механизм развертывания выполняет `bundle install` с `--without $BUNDLE_WITHOUT`.

### <a name="precompile-assets"></a>Предварительная компиляция ресурсов

По умолчанию предварительная компиляция ресурсов не входит в список действий после развертывания. Чтобы включить предварительную компиляцию ресурсов, присвойте [параметру приложения](configure-common.md#configure-app-settings) `ASSETS_PRECOMPILE` значение `true`. Теперь команда `bundle exec rake --trace assets:precompile` будет выполняться последней в списке действий после развертывания. Пример:

```azurecli-interactive
az webapp config appsettings set --name <app-name> --resource-group <resource-group-name> --settings ASSETS_PRECOMPILE=true
```

Дополнительную информацию см. в статье, посвященной [предоставлению статических ресурсов](#serve-static-assets).

## <a name="customize-start-up"></a>Настройка запуска

По умолчанию контейнер Ruby запускает сервер Rails в следующей последовательности (дополнительные сведения см. в [описании скрипта запуска](https://github.com/Azure-App-Service/ruby/blob/master/2.3.8/startup.sh)):

1. Создание значения [secret_key_base](https://edgeguides.rubyonrails.org/security.html#environmental-security), если его еще нет. Это значение требуется для запуска приложения в рабочем режиме.
1. Присвоение переменной среды `RAILS_ENV` значения `production`.
1. Удаление всех файлов *.pid* в каталоге *tmp/pids*, которые остались от ранее запущенного сервера Rails.
1. Проверка установки всех зависимостей. Если чего-то не хватает, следует установить пакеты из локального каталога *vendor/cache*.
1. Выполните `rails server -e $RAILS_ENV`.

Процесс загрузки можно настроить следующими способами:

- [предоставление статических ресурсов](#serve-static-assets);
- [выполнение в режиме непроизводственной среды](#run-in-non-production-mode);
- [настройка secret_key_base вручную](#set-secret_key_base-manually).

### <a name="serve-static-assets"></a>Предоставление статических ресурсов

Сервер Rails в контейнере Ruby по умолчанию работает в режиме производственной среды и [ожидает, что все ресурсы заранее скомпилированы и предоставляются веб-сервером](https://guides.rubyonrails.org/asset_pipeline.html#in-production). Чтобы предоставлять статические ресурсы из сервера Rails, следует обеспечить следующее:

- **Предварительная компиляция ресурсов** - [предварительно скомпилируйте статические ресурсы локально](https://guides.rubyonrails.org/asset_pipeline.html#local-precompilation) и вручную разверните их. Также вы можете поручить этот процесс механизму развертывания (подробнее см. статью [о предварительной компиляции ресурсов](#precompile-assets)).
- **Настройка предоставления статических файлов** — чтобы обслуживать статические ресурсы из контейнера Ruby, [присвойте параметру приложения `RAILS_SERVE_STATIC_FILES`](configure-common.md#configure-app-settings) значение `true`. Пример:

    ```azurecli-interactive
    az webapp config appsettings set --name <app-name> --resource-group <resource-group-name> --settings RAILS_SERVE_STATIC_FILES=true
    ```

### <a name="run-in-non-production-mode"></a>Выполнение в режиме непроизводственной среды

По умолчанию сервер Rails выполняется в режиме производственной среды. Чтобы переключить его, например, в режим разработки, задайте [параметру приложения](configure-common.md#configure-app-settings) `RAILS_ENV` значение `development`.

```azurecli-interactive
az webapp config appsettings set --name <app-name> --resource-group <resource-group-name> --settings RAILS_ENV="development"
```

Но этот параметр сам по себе только запускает сервер Rails в режиме разработки. То есть он будет обрабатывать запросы только от localhost и не будет доступен вне контейнера. Чтобы принимать запросы от удаленных клиентов, задайте [параметру приложения](configure-common.md#configure-app-settings) `APP_COMMAND_LINE` значение `rails server -b 0.0.0.0`. Этот параметр приложения позволяет выполнить в контейнере Ruby пользовательскую команду. Пример:

```azurecli-interactive
az webapp config appsettings set --name <app-name> --resource-group <resource-group-name> --settings APP_COMMAND_LINE="rails server -b 0.0.0.0"
```

### <a name="set-secret_key_base-manually"></a><a name="set-secret_key_base-manually"></a>настройка secret_key_base вручную.

Чтобы использовать собственное значение `secret_key_base`, а не автоматически созданное Службой приложений, присвойте желаемое значение [параметру приложения](configure-common.md#configure-app-settings) `SECRET_KEY_BASE`. Пример:

```azurecli-interactive
az webapp config appsettings set --name <app-name> --resource-group <resource-group-name> --settings SECRET_KEY_BASE="<key-base-value>"
```

## <a name="access-diagnostic-logs"></a>Доступ к журналам диагностики

[!INCLUDE [Access diagnostic logs](../../includes/app-service-web-logs-access-no-h.md)]

## <a name="open-ssh-session-in-browser"></a>Открытие сеанса SSH в браузере

[!INCLUDE [Open SSH session in browser](../../includes/app-service-web-ssh-connect-builtin-no-h.md)]

[!INCLUDE [robots933456](../../includes/app-service-web-configure-robots933456.md)]

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Руководство. Приложение Ruby с PostgreSQL](tutorial-ruby-postgres-app.md)

> [!div class="nextstepaction"]
> [Служба приложений под управлением Linux: вопросы и ответы](faq-app-service-linux.md)
