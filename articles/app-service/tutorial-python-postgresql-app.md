---
title: Руководство по развертыванию приложения Python (Django) с Postgres
description: Создайте веб-приложение Python с использованием базы данных PostgreSQL и разверните его в Azure. В этом учебнике используется платформа Django, а приложение размещается в Службе приложений Azure в Linux.
ms.devlang: python
ms.topic: tutorial
ms.date: 02/02/2021
ms.custom:
- mvc
- seodec18
- seo-python-october2019
- cli-validate
- devx-track-python
- devx-track-azurecli
ms.openlocfilehash: 882a9fb0f8d528ca21cdc8149c60b9d5bdaf1723
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107767101"
---
# <a name="tutorial-deploy-a-django-web-app-with-postgresql-in-azure-app-service"></a>Руководство по развертыванию веб-приложения Django с PostgreSQL в Службе приложений Azure

В этом учебнике показано, как развернуть веб-приложение Python [Django](https://www.djangoproject.com/) на основе данных в [Службе приложений Azure](overview.md) и подключить его к Базе данных Azure для Postgres. Служба приложений — служба веб-размещения с автоматической установкой исправлений и высоким уровнем масштабируемости.

В этом учебнике используется Azure CLI для выполнения следующих задач:

> [!div class="checklist"]
> * настройка начальной среды с помощью Python и Azure CLI;
> * создание базы данных в службе Базы данных Azure для PostgreSQL;
> * развертывание кода в Службе приложений Azure и подключение к PostgreSQL;
> * обновление кода и повторное развертывание;
> * просмотр журналов диагностики;
> * управление веб-приложением на портале Azure.

Вы также можете воспользоваться версией этого руководства для [портала Azure](/azure/developer/python/tutorial-python-postgresql-app-portal).


## <a name="1-set-up-your-initial-environment"></a>1. Настройка начальной среды

1. Подготовьте учетную запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) бесплатно.
1. Установите <a href="https://www.python.org/downloads/" target="_blank">Python 3.6 или более поздней версии</a>.
1. Установите <a href="/cli/azure/install-azure-cli" target="_blank">Azure CLI</a> версии 2.18.0 или более поздней для выполнения команд в любой оболочке для подготовки и настройки ресурсов Azure.

Откройте окно терминала и проверьте, что используется Python 3.6 или более поздней версии:

# <a name="bash"></a>[Bash](#tab/bash)

```bash
python3 --version
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```cmd
py -3 --version
```

# <a name="cmd"></a>[Cmd](#tab/cmd)

```cmd
py -3 --version
```

---

Убедитесь, что у вас есть Azure CLI версии 2.18.0 или выше.

```azurecli
az --version
```

Если необходимо выполнить обновление, воспользуйтесь командой `az upgrade` (требуется версия 2.11 или выше) или см. статью <a href="/cli/azure/install-azure-cli" target="_blank">Установка Azure CLI</a>.

Теперь войдите в Azure с помощью CLI.

```azurecli
az login
```

Эта команда открывает окно браузера для ввода учетных данных. После выполнения команда отображает выходные данные JSON с информацией о подписках.

Войдя, вы сможете выполнять в Azure CLI команды Azure для работы с ресурсами в подписке.

Возникли проблемы? [Сообщите нам об этом.](https://aka.ms/DjangoCLITutorialHelp)

## <a name="2-clone-or-download-the-sample-app"></a>2. Клонирование или скачивание примера приложения

# <a name="git-clone"></a>[Git clone](#tab/clone)

Клонируйте репозиторий примера.

```terminal
git clone https://github.com/Azure-Samples/djangoapp
```

Затем перейдите в эту папку:

```terminal
cd djangoapp
```

# <a name="download"></a>[Загрузить](#tab/download)

Откройте страницу [https://github.com/Azure-Samples/djangoapp](https://github.com/Azure-Samples/djangoapp), затем выберите действие **Clone** (Клонировать) и **Download ZIP** (Скачать ZIP-файл). 

Распакуйте ZIP-файл в папку с именем *djangoapp*. 

Затем откройте окно терминала в этой папке *djangoapp*.

---

Пример djangoapp содержит управляемое данными приложение Django для опросов, которое вы можете получить, выполнив [инструкции по созданию первого приложения](https://docs.djangoproject.com/en/3.1/intro/tutorial01/) из документации по Django. Готовое приложение предоставляется здесь для удобства.

Пример также изменяется для использования в рабочей среде, такой как Служба приложений.

- Параметры рабочей среды находятся в файле *azuresite/production.py*. Параметры разработки хранятся в файле *azuresite/settings.py*.
- Приложение использует параметры рабочей среде, если для задана переменная среды `WEBSITE_HOSTNAME`. Служба приложений Azure автоматически присваивает этой переменной URL-адрес веб-приложения, например `msdocs-django.azurewebsites.net`.

Эти производственные параметры связаны с настройкой Django для запуска в любой рабочей среде и не относятся к Службе приложений. Дополнительные сведения см. в [контрольном списке развертывания Django](https://docs.djangoproject.com/en/3.1/howto/deployment/checklist/). Дополнительные сведения о некоторых изменениях см. в разделе [Параметры рабочей среды для Django в Azure](configure-language-python.md#production-settings-for-django-apps).

Возникли проблемы? [Сообщите нам об этом.](https://aka.ms/DjangoCLITutorialHelp)

## <a name="3-create-postgres-database-in-azure"></a>3. Создание базы данных Postgres в Azure

<!-- > [!NOTE]
> Before you create an Azure Database for PostgreSQL server, check which [compute generation](../postgresql/concepts-pricing-tiers.md#compute-generations-and-vcores) is available in your region. -->

Установите расширение `db-up` для Azure CLI, выполнив следующую команду:

```azurecli
az extension add --name db-up
```

Если команда `az` не распознана, убедитесь, что установлен Azure CLI, как описано в разделе [Настройка начальной среды](#1-set-up-your-initial-environment).

Затем создайте базу данных Postgres в Azure с помощью команды [`az postgres up`](/cli/azure/ext/db-up/postgres#ext-db-up-az-postgres-up):

```azurecli
az postgres up --resource-group DjangoPostgres-tutorial-rg --location westus2 --sku-name B_Gen5_1 --server-name <postgres-server-name> --database-name pollsdb --admin-user <admin-username> --admin-password <admin-password> --ssl-enforcement Enabled
```

- **Замените** *\<postgres-server-name>* **уникальным именем в Azure** (конечная точка сервера станет `https://<postgres-server-name>.postgres.database.azure.com`). В качестве хорошего шаблона можно будет использовать сочетание названия компании и другого уникального значения.
- Для значений *\<admin-username>* и *\<admin-password>* укажите учетные данные, чтобы создать пользователя с правами администратора для сервера Postgres. Не используйте для имени администратора такие варианты: *azure_superuser*, *azure_pg_admin*, *admin*, *administrator*, *root*, *guest* или *public*. Оно не может начинаться с *pg_* . Пароль должен содержать **от 8 до 128 знаков** из таких трех категорий: прописные латинские буквы, строчные латинские буквы, цифры (0–9) и небуквенно-цифровые символы (например, !, #, %). Пароль не может содержать имя пользователя.
- Не используйте символ `$` в имени пользователя или пароле. Позже вы создадите переменные среды с этими значениями, где символу `$` отведена особая роль в контейнере Linux, используемом для запуска приложений Python.
- [Ценовая категория](../postgresql/concepts-pricing-tiers.md) B_Gen5_1 (Базовый, 5-е поколение, 1 ядро), используемая здесь, является самой дешевой. Для баз данных рабочей среды не указывайте аргумент `--sku-name`, чтобы вместо этого использовать уровень GP_Gen5_2 (общего назначения, 5-е поколение, 2 ядра).

С помощью этой команды выполняются следующие действия (может занять несколько минут):

- Создается [группа ресурсов](../azure-resource-manager/management/overview.md#terminology) с именем `DjangoPostgres-tutorial-rg`, если она еще не существует.
- Создается сервер Postgres с именем, указанным в аргументе `--server-name`.
- Создается учетная запись администратора с помощью аргументов `--admin-user` и `--admin-password`. Эти аргументы можно упустить, чтобы команда могла создавать уникальные учетные данные.
- Создается база данных `pollsdb` с именем, указанным в аргументе `--database-name`.
- Разрешается доступ с локального IP-адреса.
- Разрешается доступ из служб Azure.
- Создается пользователь базы данных с доступом к базе данных `pollsdb`.

Все эти шаги можно выполнять отдельно с помощью других команд `az postgres` и `psql`, но `az postgres up` выполняет все эти действия сразу.

После выполнения команды выводится объект JSON, который содержит различные строки подключения для базы данных вместе с URL-адресом сервера, созданным именем пользователя (например, joyfulKoala@msdocs-djangodb-12345) и паролем GUID. **Скопируйте имя пользователя и пароль во временный текстовый файл**, так как они еще понадобятся при работе с этим руководством.

<!-- not all locations support az postgres up -->
> [!TIP]
> Для параметра `-l <location-name>` можно указать любой из [регионов Azure](https://azure.microsoft.com/global-infrastructure/regions/). Список регионов, доступных для вашей подписки, можно получить с помощью команды [`az account list-locations`](/cli/azure/account#az_account_list_locations). Для рабочих приложений разместите базу данных и приложение в одном месте.

Возникли проблемы? [Сообщите нам об этом.](https://aka.ms/DjangoCLITutorialHelp)

## <a name="4-deploy-the-code-to-azure-app-service"></a>4. Развертывание кода в Службе приложений Azure

В этом разделе вы создадите узел приложения в приложении Службы приложений, подключите это приложение к базе данных Postgres, а затем развернете код на этом узле.

### <a name="41-create-the-app-service-app"></a>4.1. Создание приложения Службы приложений

В терминале убедитесь, что вы находитесь в папке репозитория *djangoapp*, в котором содержится код приложения.

Создайте приложение Службы приложений (хост-процесс) с помощью команды [`az webapp up`](/cli/azure/webapp#az_webapp_up):

```azurecli
az webapp up --resource-group DjangoPostgres-tutorial-rg --location westus2 --plan DjangoPostgres-tutorial-plan --sku B1 --name <app-name>
```
<!-- without --sku creates PremiumV2 plan -->

- Для аргумента `--location` используйте то же расположение, что и для базы данных в предыдущем разделе.
- **Замените** *\<app-name>* уникальным именем в Azure (конечная точка сервера — `https://<app-name>.azurewebsites.net`). Для *\<app-name>* допускаются символы `A`-`Z`, `0`-`9` и `-`. Рекомендуется использовать сочетание названия компании и идентификатора приложения.

С помощью этой команды выполняются следующие действия (может занять несколько минут):

<!-- - Create the resource group if it doesn't exist. `--resource-group` is optional. -->
<!-- No it doesn't. az webapp up doesn't respect --resource-group -->
- Создается [группа ресурсов](../azure-resource-manager/management/overview.md#terminology), если она еще не существует. (В этой команде используется та же группа ресурсов, в которой ранее была создана база данных.)
- Создается [план службы приложений](overview-hosting-plans.md) *DjangoPostgres-tutorial-plan* ценовой категории "Базовый" (B1), если он еще не существует. Параметры `--plan` и `--sku` являются необязательными.
- Создается приложение Службы приложений, если оно еще не существует.
- Для приложения включается ведение журнала по умолчанию, если оно еще не включено.
- Передается репозиторий через развертывание на основе ZIP-файла с включенной автоматизацией сборки.
- Кэшируются общие параметры, такие как имя группы ресурсов и план службы приложений, в файл *.azure/config*. Поэтому не нужно указывать все эти параметры с помощью последующих команд. Например, чтобы повторно развернуть приложение после внесения изменений, можно просто еще раз запустить `az webapp up` без параметров. Команды, поступающие из расширений CLI, например `az postgres up`, не используют кэш, поэтому необходимо указать здесь группу ресурсов и расположение при первом использовании команды `az webapp up`.

После успешного развертывания команда создает выходные данные JSON, как показано в следующем примере:

![Пример выходных данных команды az webapp up](./media/tutorial-python-postgresql-app/az-webapp-up-output.png)

Возникли проблемы? Сначала изучите [руководство по устранению неполадок](configure-language-python.md#troubleshooting). Если проблему решить не удается, [свяжитесь с нами](https://aka.ms/DjangoCLITutorialHelp).

### <a name="42-configure-environment-variables-to-connect-the-database"></a>4.2. Настройка переменных среды для подключения к базе данных

Теперь, когда код развернут в Службе приложений, необходимо подключить приложение к базе данных Postgres в Azure.

Код приложения должен находить сведения о базе данных в четырех переменных среды с именами `DBHOST`, `DBNAME`, `DBUSER` и `DBPASS`.

Чтобы задать в Службе приложений переменные среды, создайте параметры приложения с помощью следующей команды [az webapp config appsettings set](/cli/azure/webapp/config/appsettings#az_webapp_config_appsettings_set).

```azurecli
az webapp config appsettings set --settings DBHOST="<postgres-server-name>" DBNAME="pollsdb" DBUSER="<username>" DBPASS="<password>"
```

- Замените *\<postgres-server-name>* именем, которое использовали ранее с помощью команды `az postgres up`. Код в файле *azuresite/production.py* автоматически добавляет `.postgres.database.azure.com` для создания полного URL-адреса сервера Postgres.
- Замените *\<username>* и *\<password>* учетными данными администратора, которые использовали с предыдущей командой `az postgres up`, или теми, которые были созданы для вас с помощью команды `az postgres up`. Код в файле *azuresite/production.py* автоматически формирует полное имя пользователя Postgres из `DBUSER` и `DBHOST`, поэтому не включайте часть `@server`. (Как было указано ранее, не используйте символ `$` в этих значениях, так как ему отведена особая роль в переменных среды Linux.)
- Имена группы ресурсов и приложения извлекаются из кэшированных значений в файле *.azure/config*.

В коде Python можно обращаться к этим параметрам как к переменным среды с помощью таких инструкций, как `os.environ.get('DBHOST')`. Дополнительную информацию см. в разделе [Доступ к переменным среды](configure-language-python.md#access-environment-variables).

Возникли проблемы? Сначала изучите [руководство по устранению неполадок](configure-language-python.md#troubleshooting). Если проблему решить не удается, [свяжитесь с нами](https://aka.ms/DjangoCLITutorialHelp).

### <a name="43-run-django-database-migrations"></a>4.3. Перенос базы данных Django

Перенос баз данных Django гарантирует, что схема в базе данных PostgreSQL в Azure соответствует схемам, описанным в вашем коде.

1. Откройте **в браузере** сеанс SSH. Для этого перейдите по указанному ниже URL-адресу и выполните вход с использованием данных учетной записи Azure (а не учетных данных сервера базы данных).

    ```
    https://<app-name>.scm.azurewebsites.net/webssh/host
    ```

    Замените `<app-name>` именем, которое использовали ранее в команде `az webapp up`.

    Можно также подключиться к сеансу SSH с помощью команды [`az webapp ssh`](/cli/azure/webapp#az_webapp_ssh). В Windows для выполнения этой команды требуется Azure CLI 2.18.0 или более поздней версии.

    Если не удается подключиться к сеансу SSH, значит, не запустилось само приложение. [Проверьте журналы диагностики](#6-stream-diagnostic-logs), чтобы получить подробные сведения. Например, если вы еще не создали необходимые параметры приложения в предыдущем разделе, в журналах будет указано `KeyError: 'DBNAME'`.

1. Выполните следующие команды в сеансе SSH (можно вставить команды с помощью сочетания клавиш **CTRL**+**SHIFT**+**V**):

    ```bash
    # Change to the app folder
    cd $APP_PATH
    
    # Activate the venv
    source /antenv/bin/activate

    # Install requirements
    pip install -r requirements.txt

    # Run database migrations
    python manage.py migrate

    # Create the super user (follow prompts)
    python manage.py createsuperuser
    ```

    При возникновении ошибок, связанных с подключением к базе данных, проверьте значения параметров приложения, созданных при работе с предыдущим разделом.

1. Команда `createsuperuser` запросит учетные данные суперпользователя. В рамках этого учебника используйте имя пользователя по умолчанию `root`, нажмите **ВВОД**, чтобы адрес электронной почты оставался пустым, а затем введите `Pollsdb1` в качестве пароля.

1. Если отображается сообщение-ошибка о том, что база данных заблокирована, убедитесь, что в предыдущем разделе вы выполнили команду `az webapp settings`. Без этих параметров команда переноса не сможет взаимодействовать с базой данных, что приведет к ошибке.

Возникли проблемы? Сначала изучите [руководство по устранению неполадок](configure-language-python.md#troubleshooting). Если проблему решить не удается, [свяжитесь с нами](https://aka.ms/DjangoCLITutorialHelp).
    
### <a name="44-create-a-poll-question-in-the-app"></a>4.4. Создание вопроса опроса в приложении

1. В браузере откройте URL-адрес `http://<app-name>.azurewebsites.net`. В приложении должно отобразиться сообщение Polls app (Опрашивает приложение) или No polls are available (Доступных опросов нет), так как в базе данных нет конкретных опросов.

    Если отображается сообщение об ошибке приложения, скорее всего, вы не создали необходимые параметры на предыдущем шаге [Настройка переменных среды для подключения к базе данных](#42-configure-environment-variables-to-connect-the-database) или значения этих параметров содержат ошибки. Выполните команду `az webapp config appsettings list`, чтобы проверить параметры. Кроме того, можно [проверить журналы диагностики](#6-stream-diagnostic-logs), чтобы увидеть конкретные ошибки при запуске приложения. Например, если вы не создали параметры, в журналах отобразится ошибка `KeyError: 'DBNAME'`.

    После обновления параметров для исправления ошибок дайте приложению минуту для перезапуска, а затем обновите страницу браузера.

1. Перейдите по адресу `http://<app-name>.azurewebsites.net/admin`. Выполните вход с использованием учетных данных суперпользователя Django из предыдущего раздела (`root` и `Pollsdb1`). В разделе **Polls** (Опросы) щелкните **Add** (Добавить) возле поля **Questions** (Вопросы) и создайте вопрос с несколькими вариантами ответа.

1. Еще раз перейдите по адресу `http://<app-name>.azurewebsites.net`, чтобы убедиться, что пользователь видит вопросы. Ответьте на вопросы произвольным образом, чтобы наполнить базу данных.

**Поздравляем!** Вы запустили веб-приложение Python Django в Службе приложений Azure для Linux с активной базой данных Postgres.

Возникли проблемы? [Сообщите нам об этом.](https://aka.ms/DjangoCLITutorialHelp)

> [!NOTE]
> Служба приложений определяет проекты Django по файлу *wsgi.py* в каждой подпапке, которая создается в `manage.py startproject` по умолчанию. Обнаружив такой файл, Служба приложений загружает веб-приложение Django. Дополнительные сведения см. в статье о [настройке встроенного образа Python](configure-language-python.md).

## <a name="5-make-code-changes-and-redeploy"></a>5. Внесение изменений в код и его повторное развертывание

В этом разделе вы внесете локальные изменения в приложение и повторно развернете код в Службе приложений. В процессе вы настроите виртуальную среду Python, которая поддерживает текущую работу.

### <a name="51-run-the-app-locally"></a>5.1. Запуск приложения на локальном компьютере

Выполните следующие команды в окне терминала. При создании суперпользователя обязательно следуйте указаниям на экране:

# <a name="bash"></a>[bash](#tab/bash)

```bash
# Configure the Python virtual environment
python3 -m venv venv
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt
# Run Django migrations
python manage.py migrate
# Create Django superuser (follow prompts)
python manage.py createsuperuser
# Run the dev server
python manage.py runserver
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```powershell
# Configure the Python virtual environment
py -3 -m venv venv
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned -Force
venv\scripts\activate

# Install dependencies
pip install -r requirements.txt
# Run Django migrations
python manage.py migrate
# Create Django superuser (follow prompts)
python manage.py createsuperuser
# Run the dev server
python manage.py runserver
```

# <a name="cmd"></a>[CMD](#tab/cmd)

```cmd
:: Configure the Python virtual environment
py -3 -m venv venv
venv\scripts\activate

:: Install dependencies
pip install -r requirements.txt
:: Run Django migrations
python manage.py migrate
:: Create Django superuser (follow prompts)
python manage.py createsuperuser
:: Run the dev server
python manage.py runserver
```
---

После полной загрузки веб-приложения сервер разработки Django предоставит URL-адрес локального приложения в сообщении Starting development server at http://127.0.0.1:8000/. Quit the server with CTRL-BREAK. (Запуск сервера разработки по адресу http://127.0.0.1:8000/. Выйдите из сервера с помощью клавиш CTRL+BREAK).

![Пример выходных данных сервера разработки Django](./media/tutorial-python-postgresql-app/django-dev-server-output.png)

Локально протестируйте приложение, выполнив следующие действия:

1. Перейдите в браузере по адресу `http://localhost:8000`. Должно отобразиться сообщение No polls are available (Доступных опросов нет). 

1. Перейдите к `http:///localhost:8000/admin` и войдите под именем администратора, который был создан ранее. В разделе **Polls** (Опросы) снова щелкните **Add** (Добавить) возле поля **Questions** (Вопросы) и создайте вопрос с несколькими вариантами ответа. 

1. Снова перейдите по адресу *http:\//localhost:8000* и ответьте на вопрос, чтобы проверить приложение. 

1. Остановите сервер Django, нажав клавиши **CTRL**+**C**.

При локальном запуске приложение использует локальную базу данных Sqlite3 и не влияет на рабочую базу данных. При необходимости можно также использовать локальную базу данных PostgreSQL, чтобы лучше имитировать рабочую среду.

Возникли проблемы? [Сообщите нам об этом.](https://aka.ms/DjangoCLITutorialHelp)

### <a name="52-update-the-app"></a>5.2. Обновление приложения

В файле `polls/models.py` найдите строку, которая начинается с `choice_text` и измените значение параметра `max_length` на 100:

```python
# Find this line of code and set max_length to 100 instead of 200
choice_text = models.CharField(max_length=100)
```

Так как вы изменили модель данных, создайте миграцию Django и перенесите базу данных:

```
python manage.py makemigrations
python manage.py migrate
```

Снова запустите сервер разработки с `python manage.py runserver` и протестируйте приложение по адресу *http:\//localhost:8000/admin*:

Снова остановите веб-сервер Django с помощью комбинации клавиш **CTRL**+**C**.

Возникли проблемы? Сначала изучите [руководство по устранению неполадок](configure-language-python.md#troubleshooting). Если проблему решить не удается, [свяжитесь с нами](https://aka.ms/DjangoCLITutorialHelp).

### <a name="53-redeploy-the-code-to-azure"></a>5.3. Повторное развертывание кода в Azure

Выполните следующую команду в корне репозитория:

```azurecli
az webapp up
```

Эта команда использует параметры, кэшированные в файле *.azure/config*. Так как Служба приложений обнаружит, что приложение уже существует, она просто повторно развернет в нем код.

Возникли проблемы? Сначала изучите [руководство по устранению неполадок](configure-language-python.md#troubleshooting). Если проблему решить не удается, [свяжитесь с нами](https://aka.ms/DjangoCLITutorialHelp).

### <a name="54-rerun-migrations-in-azure"></a>5.4. Повторное выполнение миграций в Azure

Поскольку вы внесли изменения в модель данных, все миграции базы данных нужно повторить в Службе приложений.

Еще раз откройте в браузере сеанс SSH, перейдя по адресу `https://<app-name>.scm.azurewebsites.net/webssh/host`. Затем выполните следующие команды:

```
cd $APP_PATH
source /antenv/bin/activate
pip install -r requirements.txt
python manage.py migrate
```

Возникли проблемы? Сначала изучите [руководство по устранению неполадок](configure-language-python.md#troubleshooting). Если проблему решить не удается, [свяжитесь с нами](https://aka.ms/DjangoCLITutorialHelp).

### <a name="55-review-app-in-production"></a>5.5. Проверка приложения в рабочей среде

Перейдите по адресу `http://<app-name>.azurewebsites.net` и снова проверьте приложение в рабочей среде. (Так как вы изменили только длину поля базы данных, это изменение заметно лишь при попытке ввести более длинный ответ при создании вопроса.)

Возникли проблемы? Сначала изучите [руководство по устранению неполадок](configure-language-python.md#troubleshooting). Если проблему решить не удается, [свяжитесь с нами](https://aka.ms/DjangoCLITutorialHelp).

## <a name="6-stream-diagnostic-logs"></a>6. Потоковая передача журналов диагностики

Вы можете получить доступ к журналам консоли, созданным в контейнере, в котором размещено приложение в Azure.

Выполните следующую команду Azure CLI, чтобы увидеть поток журнала. Эта команда использует параметры, кэшированные в файле *.azure/config*.

```azurecli
az webapp log tail
```

Если журналы консоли не отображаются, проверьте еще раз через 30 секунд.

Чтобы остановить потоковую передачу журналов, нажмите клавиши **CTRL**+**C**.

Возникли проблемы? [Сообщите нам об этом.](https://aka.ms/DjangoCLITutorialHelp)

> [!NOTE]
> Вы также можете проверить файлы журнала в браузере на странице `https://<app-name>.scm.azurewebsites.net/api/logs/docker`.
>
> Команда `az webapp up` позволяет включить ведение журналов по умолчанию. Чтобы не ухудшать производительность, через некоторое время ведение журнала отключается, но включается повторно при каждом запуске `az webapp up`. Чтобы отключить его вручную, выполните следующую команду:
>
> ```azurecli
> az webapp log config --docker-container-logging filesystem
> ```

## <a name="7-manage-your-app-in-the-azure-portal"></a>7. Управление приложением с помощью портала Azure

На [портале Azure](https://portal.azure.com) выполните поиск приложения по имени и выберите его.

![Переход к приложению Python (Django) на портале Azure](./media/tutorial-python-postgresql-app/navigate-to-django-app-in-app-services-in-the-azure-portal.png)

По умолчанию на портале отображается страница **Обзор** приложения с общим представлением данных о производительности. Вы можете выполнять здесь базовые задачи управления, например просмотр, остановку, перезагрузку и удаление. На вкладках в левой части страницы отображаются различные страницы конфигурации, которые можно открыть.

![Управление приложением Python (Django) на странице "Обзор" на портале Azure](./media/tutorial-python-postgresql-app/manage-django-app-in-app-services-in-the-azure-portal.png)

Возникли проблемы? Сначала изучите [руководство по устранению неполадок](configure-language-python.md#troubleshooting). Если проблему решить не удается, [свяжитесь с нами](https://aka.ms/DjangoCLITutorialHelp).

## <a name="8-clean-up-resources"></a>8. Очистка ресурсов

Если вы хотите сохранить приложение или продолжить работу с другими учебниками, перейдите к разделу с [дальнейшими действиями](#next-steps). В противном случае, чтобы избежать постоянных расходов, вы можете удалить группу ресурсов, созданную для этого учебника:

```azurecli
az group delete --no-wait
```

Эта команда использует имя группы ресурсов, кэшированное в файле *.azure/config*. При удалении группы ресурсов будут также освобождены и удалены все ресурсы, содержащиеся в ней.

Удаление всех ресурсов может занять некоторое время. Аргумент `--no-wait` позволяет команде выполнять возврат без задержки.

Возникли проблемы? [Сообщите нам об этом.](https://aka.ms/DjangoCLITutorialHelp)

## <a name="next-steps"></a>Дальнейшие действия

Сведения о сопоставлении настраиваемого DNS-имени с вашим приложением:

> [!div class="nextstepaction"]
> [Руководство. Сопоставление настраиваемого DNS-имени с приложением](app-service-web-tutorial-custom-domain.md)

Узнайте, как Служба приложений запускает приложение Python:

> [!div class="nextstepaction"]
> [Настройка приложения Python](configure-language-python.md)
