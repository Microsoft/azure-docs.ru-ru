---
title: Руководство по приложению Ruby в Linux на основе Postgres
description: Узнайте, как создать приложение Ruby в Linux, работающее в Службе приложений Azure, с подключением к базе данных PostgreSQL в Azure. В этом руководстве используется Rails.
ms.devlang: ruby
ms.topic: tutorial
ms.date: 06/18/2020
ms.custom: mvc, cli-validate, seodec18, devx-track-azurecli
ms.openlocfilehash: 63194ab87e0f2228b8585e962394aa1ebfff48d6
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107767280"
---
# <a name="build-a-ruby-and-postgres-app-in-azure-app-service-on-linux"></a>Разработка приложения на основе Ruby и Postgres в Службе приложений Azure в Linux

[Служба приложений Azure](overview.md) — это служба веб-размещения с самостоятельной установкой исправлений и высоким уровнем масштабируемости. В этом руководстве показано, как создать приложение Ruby и подключить его к базе данных PostgreSQL. По завершении вы получите приложение [Ruby on Rails](https://rubyonrails.org/), работающее в службе приложений под управлением Linux.

:::image type="content" source="./media/tutorial-ruby-postgres-app/complete-checkbox-published.png" alt-text="Снимок экрана: пример приложения Ruby on Rails с названием Tasks.":::

В этом руководстве описано следующее:

> [!div class="checklist"]
> * Создание базы данных PostgreSQL в Azure
> * Подключение приложения Ruby on Rails к PostgreSQL
> * Развертывание приложения в Azure
> * Обновление модели данных и повторное развертывание приложения.
> * Потоковая передача журналов диагностики из Azure.
> * Управление приложением на портале Azure.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим руководством сделайте следующее:

- [установите Git](https://git-scm.com/);
- [установите Ruby 2.6](https://www.ruby-lang.org/en/documentation/installation/);
- [установите Ruby on Rails 5.1](https://guides.rubyonrails.org/v5.1/getting_started.html);
- [установите и запустите PostgreSQL](https://www.postgresql.org/download/);

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

## <a name="prepare-local-postgres"></a>Подготовка локальной базы данных Postgres

На этом шаге вы создадите пример базы данных на локальном сервере Postgres.

### <a name="connect-to-local-postgres-server"></a>Подключение к локальному серверу Postgres

Откройте окно терминала и выполните команду `psql` для подключения к локальному серверу Postgres.

```bash
sudo -u postgres psql
```

Если подключение установлено, это означает, что база данных Postgres запущена. В противном случае запустите локальную базу данных Postgres, выполнив инструкции по [скачиванию ядра PostgreSQL](https://www.postgresql.org/download/).

Введите `\q`, чтобы выйти из клиента Postgres. 

С помощью своего имени пользователя для входа в Linux создайте пользователя Postgres, который сможет создавать базы данных, выполняя следующую команду:

```bash
sudo -u postgres createuser -d <signed-in-user>
```

<a name="step2"></a>

## <a name="create-a-ruby-on-rails-app-locally"></a>Создание приложения Ruby on Rails локально
На этом шаге вы создадите пример приложения Ruby on Rails, настроите его подключение к базе данных и запустите это приложение в локальной среде. 

### <a name="clone-the-sample"></a>Клонирования репозитория

Откройте окно терминала и c помощью команды `cd` перейдите в рабочий каталог.

Выполните команду ниже, чтобы клонировать репозиторий с примером.

```bash
git clone https://github.com/Azure-Samples/rubyrails-tasks.git
```

С помощью команды `cd` перейдите в клонированный каталог. Установите необходимые пакеты.

```bash
cd rubyrails-tasks
bundle install --path vendor/bundle
```

### <a name="run-the-sample-locally"></a>Локальный запуск примера

Выполните [перенос Rails](https://guides.rubyonrails.org/active_record_migrations.html#running-migrations), чтобы создать таблицы, необходимые для приложения. Чтобы узнать, какие таблицы создаются при переносе, просмотрите каталог _db/migrate_ в репозитории Git.

```bash
rake db:create
rake db:migrate
```

Запустите приложение.

```bash
rails server
```

Откройте браузер и перейдите по адресу `http://localhost:3000`. Добавьте несколько задач на странице.

![Успешное подключение Ruby on Rails к Postgres](./media/tutorial-ruby-postgres-app/postgres-connect-success.png)

Чтобы остановить сервер Rails, введите `Ctrl + C` в окне терминала.

## <a name="create-postgres-in-azure"></a>Создание Postgres в Azure

На этом этапе вы создадите базу данных Postgres в [Базе данных Azure для PostgreSQL](../postgresql/index.yml). Позже вы настроите приложение Ruby on Rails для подключения к этой базе данных.

### <a name="create-a-resource-group"></a>Создание группы ресурсов

[!INCLUDE [Create resource group](../../includes/app-service-web-create-resource-group-linux-no-h.md)] 

## <a name="create-postgres-database-in-azure"></a>Создание базы данных Postgres в Azure

<!-- > [!NOTE]
> Before you create an Azure Database for PostgreSQL server, check which [compute generation](../postgresql/concepts-pricing-tiers.md#compute-generations-and-vcores) is available in your region. If your region doesn't support Gen4 hardware, change *--sku-name* in the following command line to a value that's supported in your region, such as B_Gen4_1.  -->

В этом разделе описано, как создать сервер Базы данных Azure для PostgreSQL и саму базу данных. Для начала установите расширение `db-up`, выполнив следующую команду:

```azurecli
az extension add --name db-up
```

Создайте базу данных Postgres в Azure с помощью команды [`az postgres up`](/cli/azure/ext/db-up/postgres#ext-db-up-az-postgres-up), как показано в приведенном ниже примере. Замените заполнитель *\<postgresql-name>* *уникальным* именем (конечная точка сервера — *https://\<postgresql-name>.postgres.database.azure.com*). Для значений *\<admin-username>* и *\<admin-password>* укажите учетные данные, чтобы создать пользователя с правами администратора для сервера Postgres.

<!-- Issue: without --location -->
```azurecli
az postgres up --resource-group myResourceGroup --location westeurope --server-name <postgresql-name> --database-name sampledb --admin-user <admin-username> --admin-password <admin-password> --ssl-enforcement Enabled
```

Эта команда может выполняться довольно долго, так как осуществляются такие задачи:

- Создается [группа ресурсов](../azure-resource-manager/management/overview.md#terminology) с именем `myResourceGroup`, если она еще не существует. Каждый ресурс Azure должен находиться в одной из них. Аргумент `--resource-group` является необязательным.
- Создается сервер Postgres и пользователь с правами администратора.
- Создается база данных `sampledb`.
- Разрешается доступ с локального IP-адреса.
- Разрешается доступ из служб Azure.
- Создается пользователь базы данных с доступом к базе данных `sampledb`.

Все эти шаги можно выполнять отдельно с помощью других команд `az postgres` и `psql`, но `az postgres up` выполняет все эти действия за один шаг.

Когда команда будет выполнена, найдите в выходных данных строки, которые начинаются с `Ran Database Query:`. Здесь отображаются сведения о созданном пользователе базы данных: имя пользователя `root` и пароль `Sampledb1`. Они пригодятся вам позже для подключения приложения к базе данных.

<!-- not all locations support az postgres up -->
> [!TIP]
> Для параметра `--location <location-name>` можно указать любой из [регионов Azure](https://azure.microsoft.com/global-infrastructure/regions/). Список регионов, доступных для вашей подписки, можно получить с помощью команды [`az account list-locations`](/cli/azure/account#az_account_list_locations). Для рабочих приложений разместите базу данных и приложение в одном месте.

## <a name="connect-app-to-azure-postgres"></a>Подключение приложения к Azure Postgres

На этом шаге вы подключите приложение Ruby on Rails к базе данных Postgres, созданной в Базе данных Azure для PostgreSQL.

<a name="devconfig"></a>

### <a name="configure-the-database-connection"></a>Настройка подключения к базе данных

В репозитории откройте файл _config/database.yml_. В нижней части файла замените рабочие переменные следующим кодом. 

```txt
production:
  <<: *default
  host: <%= ENV['DB_HOST'] %>
  database: <%= ENV['DB_DATABASE'] %>
  username: <%= ENV['DB_USERNAME'] %>
  password: <%= ENV['DB_PASSWORD'] %>
```

Сохраните изменения.

### <a name="test-the-application-locally"></a>Локальное тестирование приложения

Вернитесь в окно терминала на локальном компьютере и установите следующие переменные среды:

```bash
export DB_HOST=<postgres-server-name>.postgres.database.azure.com
export DB_DATABASE=sampledb 
export DB_USERNAME=root@<postgres-server-name>
export DB_PASSWORD=Sampledb1
```

Выполните перенос базы данных Rails с настроенными рабочими значениями для создания таблиц в базе данных Postgres Базы данных Azure для PostgreSQL.

```bash
rake db:migrate RAILS_ENV=production
```

При использовании в рабочей среде приложению Rails нужны предкомпилированные ресурсы. Создайте необходимые ресурсы, выполнив следующую команду:

```bash
rake assets:precompile
```

В рабочей среде Rails также используются секреты для управления безопасностью. Создайте секретный ключ.

```bash
rails secret
```

Сохраните секретный ключ в соответствующих переменных, используемых в рабочей среде Rails. Для удобства используйте один ключ для обеих переменных.

```bash
export RAILS_MASTER_KEY=<output-of-rails-secret>
export SECRET_KEY_BASE=<output-of-rails-secret>
```

Включите обработку файлов JavaScript и CSS в рабочей среде Rails.

```bash
export RAILS_SERVE_STATIC_FILES=true
```

Запустите пример приложения в рабочей среде.

```bash
rails server -e production
```

Перейдите на страницу `http://localhost:3000`. Если страница загрузилась без ошибок, это значит, что приложение Ruby on Rails подключается к базе данных Postgres в Azure.

Добавьте несколько задач на странице.

![Приложение Ruby on Rails подключилось к Базе данных Azure для PostgreSQL](./media/tutorial-ruby-postgres-app/azure-postgres-connect-success.png)

Чтобы остановить сервер Rails, введите `Ctrl + C` в окне терминала.

### <a name="commit-your-changes"></a>Фиксация изменений

Чтобы зафиксировать изменения, выполните следующие команды Git:

```bash
git add .
git commit -m "database.yml updates"
```

Ваше приложение готово к развертыванию.

## <a name="deploy-to-azure"></a>Развертывание в Azure

На этом этапе вы развернете приложение Rails, подключенное к базе данных Postgres, в Службе приложений Azure.

### <a name="configure-a-deployment-user"></a>Настойка пользователя развертывания

[!INCLUDE [Configure deployment user](../../includes/configure-deployment-user-no-h.md)]

### <a name="create-an-app-service-plan"></a>Создание плана службы приложений

[!INCLUDE [Create app service plan no h](../../includes/app-service-web-create-app-service-plan-linux-no-h.md)]

### <a name="create-a-web-app"></a>Создание веб-приложения

[!INCLUDE [Create web app](../../includes/app-service-web-create-web-app-ruby-linux-no-h.md)] 

### <a name="configure-database-settings"></a>Настройка параметров базы данных

В службе приложений переменные среды задаются в качестве _параметров приложения_ с помощью команды [`az webapp config appsettings set`](/cli/azure/webapp/config/appsettings#az_webapp_config_appsettings_set) в Cloud Shell.

Команда Cloud Shell ниже позволяет настроить параметры приложения `DB_HOST`, `DB_DATABASE`, `DB_USERNAME` и `DB_PASSWORD`. Замените заполнители _&lt;appname>_ и _&lt;postgres-server-name>_ .

```azurecli-interactive
az webapp config appsettings set --name <app-name> --resource-group myResourceGroup --settings DB_HOST="<postgres-server-name>.postgres.database.azure.com" DB_DATABASE="sampledb" DB_USERNAME="root@<postgres-server-name>" DB_PASSWORD="Sampledb1"
```

### <a name="configure-rails-environment-variables"></a>Настройка переменных среды Rails

В окне терминала на локальном компьютере [создайте новый секрет](configure-language-ruby.md#set-secret_key_base-manually) для рабочей среды Rails в Azure.

```bash
rails secret
```

Настройте переменные, необходимые для рабочей среды Rails.

В следующей команде Cloud Shell замените два заполнителя _&lt;output-of-rails-secret>_ новым секретным ключом, созданным в окне терминала на локальном компьютере.

```azurecli-interactive
az webapp config appsettings set --name <app-name> --resource-group myResourceGroup --settings RAILS_MASTER_KEY="<output-of-rails-secret>" SECRET_KEY_BASE="<output-of-rails-secret>" RAILS_SERVE_STATIC_FILES="true" ASSETS_PRECOMPILE="true"
```

Свойство со значением `ASSETS_PRECOMPILE="true"` указывает контейнеру Ruby по умолчанию предварительно компилировать ресурсы во время каждого развертывания Git. См. подробнее о [предварительно компилируемых ресурсах](configure-language-ruby.md#precompile-assets) и [обслуживании статических ресурсов](configure-language-ruby.md#serve-static-assets).

### <a name="push-to-azure-from-git"></a>Публикация в Azure из Git

В окне терминала на локальном компьютере добавьте удаленное приложение Azure в локальный репозиторий Git.

```bash
git remote add azure <paste-copied-url-here>
```

Выполните публикацию в удаленную службу приложений Azure, чтобы развернуть приложение Ruby on Rails. После этого введите пароль, указанный ранее в процессе создания пользователя развертывания.

```bash
git push azure main
```

Во время развертывания служба приложений Azure будет взаимодействовать с Git.

<pre>
Counting objects: 3, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 291 bytes | 0 bytes/s, done.
Total 3 (delta 2), reused 0 (delta 0)
remote: Updating branch 'main'.
remote: Updating submodules.
remote: Preparing deployment for commit id 'a5e076db9c'.
remote: Running custom deployment command...
remote: Running deployment command...
...
&lt; Output has been truncated for readability &gt;
</pre>

### <a name="browse-to-the-azure-app"></a>Переход к приложению Azure

Перейдите по адресу `http://<app-name>.azurewebsites.net` и добавьте несколько задач в список.

:::image type="content" source="./media/tutorial-ruby-postgres-app/ruby-postgres-in-azure.png" alt-text="Снимок экрана: пример приложения Azure с названием Tasks, в котором показаны добавленные в список задачи.":::

Вы запустили управляемое данными приложение Ruby on Rails в службе приложений Azure.

## <a name="update-model-locally-and-redeploy"></a>Локальное обновление и повторное развертывание модели

На этом шаге вы внесете некоторые изменения в модель данных `task` и веб-приложение, а затем опубликуете обновления в Azure.

Для сценария с задачами необходимо изменить приложение, чтобы можно было пометить задачу как выполненную.

### <a name="add-a-column"></a>Добавление столбца

В окне терминала перейдите к корневой папке репозитория Git.

Создайте миграцию, при которой столбец логических значений `Done` добавляется в таблицу `Tasks`:

```bash
rails generate migration AddDoneToTasks Done:boolean
```

Эта команда создает файл миграции в каталоге _db/migrate_.


В окне терминала выполните перенос базы данных Rails, чтобы внести изменения в локальную базу данных.

```bash
rake db:migrate
```

### <a name="update-application-logic"></a>Обновление логики приложения

Откройте файл *app/controllers/tasks_controller.rb*. В конце файла найдите следующую строку:

```rb
params.require(:task).permit(:Description)
```

Измените эту строку, добавив в нее новый параметр `Done`.

```rb
params.require(:task).permit(:Description, :Done)
```

### <a name="update-the-views"></a>Обновление представлений

Откройте файл *app/views/tasks/_form.html.erb*, который является формой редактирования.

Найдите строку `<%=f.error_span(:Description) %>` и вставьте непосредственно под ней следующий код:

```erb
<%= f.label :Done, :class => 'control-label col-lg-2' %>
<div class="col-lg-10">
  <%= f.check_box :Done, :class => 'form-control' %>
</div>
```

Откройте файл *app/views/tasks/show.html.erb*, который является страницей представления с одной записью. 

Найдите строку `<dd><%= @task.Description %></dd>` и вставьте непосредственно под ней следующий код:

```erb
  <dt><strong><%= model_class.human_attribute_name(:Done) %>:</strong></dt>
  <dd><%= check_box "task", "Done", {:checked => @task.Done, :disabled => true}%></dd>
```

Откройте файл *app/views/tasks/index.html.erb*, который является страницей индекса для всех записей.

Найдите строку `<th><%= model_class.human_attribute_name(:Description) %></th>` и вставьте непосредственно под ней следующий код:

```erb
<th><%= model_class.human_attribute_name(:Done) %></th>
```

В этом же файле найдите строку `<td><%= task.Description %></td>` и вставьте непосредственно под ней следующий код:

```erb
<td><%= check_box "task", "Done", {:checked => task.Done, :disabled => true} %></td>
```

### <a name="test-the-changes-locally"></a>Локальная проверка изменений

В окне терминала на локальном компьютере запустите сервер Rails.

```bash
rails server
```

Чтобы увидеть, как изменится состояние задачи, перейдите по адресу `http://localhost:3000` и добавьте или измените элемент.

![Добавлен флажок для задачи](./media/tutorial-ruby-postgres-app/complete-checkbox.png)

Чтобы остановить сервер Rails, введите `Ctrl + C` в окне терминала.

### <a name="publish-changes-to-azure"></a>Публикация изменений в Azure

В окне терминала выполните перенос базы данных Rails для рабочей среды, чтобы внести изменения в базу данных Azure.

```bash
rake db:migrate RAILS_ENV=production
```

Зафиксируйте все изменения в Git, а затем отправьте изменения кода в Azure.

```bash
git add .
git commit -m "added complete checkbox"
git push azure main
```

После выполнения команды `git push` перейдите в приложение Azure и проверьте новые функции.

![Изменения модели и базы данных, опубликованные в Azure](media/tutorial-ruby-postgres-app/complete-checkbox-published.png)

Добавленные задачи сохраняются в базе данных. Изменения в схеме данных не влияют на имеющиеся данные.

## <a name="stream-diagnostic-logs"></a>Потоковая передача журналов диагностики

[!INCLUDE [Access diagnostic logs](../../includes/app-service-web-logs-access-no-h.md)]

## <a name="manage-the-azure-app"></a>Управление приложением Azure

Перейдите на [портал Azure](https://portal.azure.com), чтобы управлять созданным приложением.

В меню слева щелкните **Службы приложений**, а затем — имя своего приложения Azure.

![Переход к приложению Azure на портале](./media/tutorial-php-mysql-app/access-portal.png)

Отобразится страница обзора вашего приложения. Вы можете выполнять базовые задачи управления: обзор, завершение, запуск, перезагрузку и удаление.

В меню слева доступны страницы для настройки приложения.

![Страница службы приложений на портале Azure](./media/tutorial-php-mysql-app/web-app-blade.png)

[!INCLUDE [cli-samples-clean-up](../../includes/cli-samples-clean-up.md)]

<a name="next"></a>

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы узнали, как выполнять следующие задачи:

> [!div class="checklist"]
> * Создание базы данных Postgres в Azure
> * Подключение приложения Ruby on Rails к Postgres
> * Развертывание приложения в Azure
> * Обновление модели данных и повторное развертывание приложения.
> * Потоковая передача журналов диагностики из Azure.
> * Управление приложением на портале Azure.

Перейдите к следующему руководству, чтобы научиться сопоставлять пользовательские DNS-имена с приложением.

> [!div class="nextstepaction"]
> [Руководство. Сопоставление настраиваемого DNS-имени с приложением](app-service-web-tutorial-custom-domain.md)

Также ознакомьтесь с другими ресурсами:

> [!div class="nextstepaction"]
> [Настройка приложения Ruby](configure-language-ruby.md)
