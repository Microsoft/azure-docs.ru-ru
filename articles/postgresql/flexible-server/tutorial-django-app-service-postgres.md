---
title: Руководство по Развертывание приложения Django с помощью Службы приложений и Гибкого сервера Базы данных Azure для PostgreSQL (предварительная версия) в виртуальной сети
description: Развертывание приложения Django с помощью Службы приложений и Гибкого сервера Базы данных Azure для PostgreSQL (предварительная версия) в виртуальной сети
author: mksuni
ms.author: sumuth
ms.service: postgresql
ms.devlang: azurecli
ms.topic: tutorial
ms.date: 09/22/2020
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 84fd0c368bcf39af1bae90dc83336ffdb8489768
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107791523"
---
# <a name="tutorial-deploy-django-app-with-app-service-and-azure-database-for-postgresql---flexible-server-preview"></a>Руководство по Развертывание приложения Django с помощь Службы приложений и Гибкого сервера Базы данных Azure для PostgreSQL (предварительная версия)

> [!IMPORTANT]
> Гибкий сервер Базы данных Azure для PostgreSQL предоставляется в режиме предварительной версии

Из этого руководства вы узнаете, как развернуть приложение Django в Azure с помощью Службы приложений и Базы данных Azure для PostgreSQL (Гибкий сервер) в виртуальной сети.

## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.

Для этой статьи требуется запустить локально Azure CLI версии 2.0 или более поздней. Чтобы узнать, какая установлена версия, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0](/cli/azure/install-azure-cli).

Вам потребуется выполнить вход в учетную запись с помощью команды [az login](/cli/azure/authenticate-azure-cli). Запишите свойство **id** из выходных данных команды для соответствующего имени подписки.

```azurecli
az login
```

Если вы используете несколько подписок, выберите соответствующую, в которой за ресурс будет взиматься плата. Выберите конкретный идентификатор подписки вашей учетной записи, выполнив команду [az account set](/cli/azure/account). Подставьте свойство **subscription ID** из выходных данных **az login** для своей подписки в заполнитель для идентификатора подписки.

```azurecli
az account set --subscription <subscription id>
```
## <a name="clone-or-download-the-sample-app"></a>Клонирование или скачивание примера приложения

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

Пример djangoapp содержит управляемое данными приложение Django для опросов, которое вы можете получить, выполнив [инструкции по созданию первого приложения](https://docs.djangoproject.com/en/2.1/intro/tutorial01/) из документации по Django. Готовое приложение предоставляется здесь для удобства.

Пример также изменяется для использования в рабочей среде, такой как Служба приложений.

- Параметры рабочей среды находятся в файле *azuresite/production.py*. Сведения о разработке хранятся в файле *azuresite/settings.py*.
- Приложение использует параметры для работы в производственной среде, если для переменной среды `DJANGO_ENV` задано значение production. В учебнике эта переменная среды создается позже вместе с другими переменными, используемыми для настройки базы данных PostgreSQL.

Эти изменения связаны с настройкой Django для запуска в любой рабочей среде и не относятся к Службе приложений. Дополнительные сведения см. в [контрольном списке развертывания Django](https://docs.djangoproject.com/en/2.1/howto/deployment/checklist/).

## <a name="create-a-postgresql-flexible-server-in-a-new-virtual-network"></a>Создание гибкого сервера PostgreSQL в новой виртуальной сети

Создайте гибкий сервер с частным доступом и базу данных в виртуальной сети с помощью следующей команды:
```azurecli
# Create Flexible server in a VNET

az postgres flexible-server create --resource-group myresourcegroup --location westus2

```

С помощью этой команды выполняются следующие действия (может занять несколько минут):

- Создайте группу ресурсов, если она еще не существует.
- Создает имя сервера, если оно не предоставлено.
- Создает виртуальную сеть для нового сервера PostgreSQL. **Запишите имя виртуальной сети и имя подсети**, которые созданы для этого сервера, так как веб-приложение нужно будет добавить в ту же виртуальную сеть.
- Создает имя пользователя и пароль администратора для этого сервера, если они не предоставлены. **Запишите имя пользователя и пароль**, которые понадобятся на следующем шаге.
- Создайте базу данных ```postgres```, которую можно использовать для разработки. Вы можете выполнить команду [**psql** для подключения к базе данных](quickstart-create-server-portal.md#connect-to-the-postgresql-database-using-psql), чтобы создать другую базу данных.

> [!NOTE]
> Если вы не указали пароль, запишите тот, который будет создан для вас автоматически. Если вы забудете пароль, вам придется сбросить его с помощью команды ``` az postgres flexible-server update```.


## <a name="deploy-the-code-to-azure-app-service"></a>Развертывание кода в Службе приложений Azure

В этом разделе вы создадите узел приложения в приложении Службы приложений, подключите это приложение к базе данных Postgres, а затем развернете код на этом узле.


### <a name="create-the-app-service-web-app-in-a-virtual-network"></a>Создание веб-приложения Службы приложений в виртуальной сети

В терминале убедитесь, что вы находитесь в корне репозитория (`djangoapp`), в котором находится код приложения.

Создайте приложение Службы приложений (хост-процесс) с помощью команды [`az webapp up`](/cli/azure/webapp#az_webapp_up):

```azurecli

# Create a web app
az webapp up --resource-group myresourcegroup --location westus2 --plan DjangoPostgres-tutorial-plan --sku B1 --name <app-name>

# Enable VNET integration for web app.
# Replace <vnet-name> and <subnet-name> with the virtual network and subnet name that the flexible server is using.

az webapp vnet-integration add -g myresourcegroup -n  mywebapp --vnet <vnet-name> --subnet <subnet-name>

# Configure database information as environment variables
# Use the postgres server name , database name , username , password for the database created in the previous steps

az webapp config appsettings set --settings DJANGO_ENV="production" DBHOST="<postgres-server-name>.postgres.database.azure.com" DBNAME="postgres" DBUSER="<username>" DBPASS="<password>"
```
- Для аргумента `--location` используйте то же расположение, что и для базы данных в предыдущем разделе.
- Замените *\<app-name>* уникальным именем в Azure (конечная точка сервера — `https://\<app-name>.azurewebsites.net`). Для *\<app-name>* допускаются символы `A`-`Z`, `0`-`9` и `-`. Рекомендуется использовать сочетание названия компании и идентификатора приложения.
- Создается [план службы приложений](../../app-service/overview-hosting-plans.md) *DjangoPostgres-tutorial-plan* ценовой категории "Базовый" (B1), если он еще не существует. Параметры `--plan` и `--sku` являются необязательными.
- Создается приложение Службы приложений, если оно еще не существует.
- Для приложения включается ведение журнала по умолчанию, если оно еще не включено.
- Передается репозиторий через развертывание на основе ZIP-файла с включенной автоматизацией сборки.
- Команда **az webapp vnet-integration** добавляет веб-приложение в ту виртуальную сеть, где расположен сервер postgres.
- Код приложения должен находить сведения о базе данных в переменных среды. Чтобы задать в Службе приложений переменные среды, вы создаете параметры приложения с помощью команды [az webapp config appsettings set](/cli/azure/webapp/config/appsettings#az_webapp_config_appsettings_set).

> [!TIP]
> Многие команды Azure CLI кэшируют общие параметры, такие как имя группы ресурсов и план службы приложений, в файл *.azure/config*. Поэтому не нужно указывать все эти параметры с помощью последующих команд. Например, чтобы повторно развернуть приложение после внесения изменений, можно просто еще раз запустить `az webapp up` без параметров.

### <a name="run-django-database-migrations"></a>Перенос базы данных Django

Перенос баз данных Django гарантирует, что схема в базе данных PostgreSQL в Azure соответствует схемам, описанным в вашем коде.

1. Откройте в браузере сеанс SSH, перейдя по адресу *https://\<app-name>.scm.azurewebsites.net/webssh/host*, и выполните вход с использованием учетных данных учетной записи Azure (а не учетных данных сервера базы данных).

1. Выполните следующие команды в сеансе SSH (можно вставить команды с помощью сочетания клавиш **CTRL**+**SHIFT**+**V**):

    ```bash
    cd site/wwwroot

    # Activate default virtual environment in App Service container
    source /antenv/bin/activate
    # Install packages
    pip install -r requirements.txt
    # Run database migrations
    python manage.py migrate
    # Create the super user (follow prompts)
    python manage.py createsuperuser
    ```

1. Команда `createsuperuser` запросит учетные данные суперпользователя. В рамках этого учебника используйте имя пользователя по умолчанию `root`, нажмите **ВВОД**, чтобы адрес электронной почты оставался пустым, а затем введите `postgres1` в качестве пароля.

### <a name="create-a-poll-question-in-the-app"></a>Создание вопроса опроса в приложении

1. В браузере откройте URL-адрес *http:\//\<app-name>.azurewebsites.net*. В приложении должно отобразиться сообщение No polls are available (Доступных опросов нет), так как в базе данных нет отдельных опросов.

1. Перейдите к *http:\//\<app-name>.azurewebsites.net/admin*. Выполните вход с использованием учетных данных суперпользователя из предыдущего раздела (`root` и `postgres1`). В разделе **Polls** (Опросы) щелкните **Add** (Добавить) возле поля **Questions** (Вопросы) и создайте вопрос с несколькими вариантами ответа.

1. Еще раз перейдите по адресу *http:\//\<app-name>.azurewebsites.net/* , чтобы убедиться, что пользователь видит вопросы. Ответьте на вопросы, если хотите наполнить базу данных.

**Поздравляем!** Вы запустили веб-приложение Python Django в Службе приложений Azure для Linux с активной базой данных Postgres.

> [!NOTE]
> Служба приложений определяет проекты Django по файлу *wsgi.py* в каждой подпапке, которая создается в `manage.py startproject` по умолчанию. Обнаружив такой файл, Служба приложений загружает веб-приложение Django. Дополнительные сведения см. в статье о [настройке встроенного образа Python](../../app-service/configure-language-python.md).

## <a name="make-code-changes-and-redeploy"></a>Внесение изменений в код и его повторное развертывание

В этом разделе вы внесете локальные изменения в приложение и повторно развернете код в Службе приложений. В процессе вы настроите виртуальную среду Python, которая поддерживает текущую работу.

### <a name="run-the-app-locally"></a>Локальный запуск приложения

Выполните следующие команды в окне терминала. При создании суперпользователя обязательно следуйте указаниям на экране:

```bash
# Configure the Python virtual environment
python3 -m venv venv
source venv/bin/activate

# Install packages
pip install -r requirements.txt
# Run Django migrations
python manage.py migrate
# Create Django superuser (follow prompts)
python manage.py createsuperuser
# Run the dev server
python manage.py runserver
```
После полной загрузки веб-приложения сервер разработки Django предоставит URL-адрес локального приложения в сообщении Starting development server at http://127.0.0.1:8000/. Quit the server with CTRL-BREAK. (Запуск сервера разработки по адресу http://127.0.0.1:8000/. Выйдите из сервера с помощью клавиш CTRL+BREAK).

:::image type="content" source="./media/tutorial-django-app-service-postgres/django-dev-server-output.png" alt-text="Пример выходных данных сервера разработки Django":::

Локально протестируйте приложение, выполнив следующие действия:

1. Перейдите в браузере по адресу *http:\//localhost:8000*. Должно отобразиться сообщение No polls are available (Доступных опросов нет).

1. Перейдите по адресу *http:\//localhost:8000/admin* и войдите под именем администратора, который был создан ранее. В разделе **Polls** (Опросы) снова щелкните **Add** (Добавить) возле поля **Questions** (Вопросы) и создайте вопрос с несколькими вариантами ответа.

1. Снова перейдите по адресу *http:\//localhost:8000* и ответьте на вопрос, чтобы проверить приложение.

1. Остановите сервер Django, нажав клавиши **CTRL**+**C**.

При локальном запуске приложение использует локальную базу данных Sqlite3 и не влияет на рабочую базу данных. При необходимости можно также использовать локальную базу данных PostgreSQL, чтобы лучше имитировать рабочую среду.



### <a name="update-the-app"></a>Обновление приложения

В файле `polls/models.py` найдите строку, которая начинается с `choice_text` и измените значение параметра `max_length` на 100:

```python
# Find this lie of code and set max_length to 100 instead of 200
choice_text = models.CharField(max_length=100)
```

Так как вы изменили модель данных, создайте миграцию Django и перенесите базу данных:

```
python manage.py makemigrations
python manage.py migrate
```

Снова запустите сервер разработки с `python manage.py runserver` и протестируйте приложение по адресу *http:\//localhost:8000/admin*:

Снова остановите веб-сервер Django с помощью комбинации клавиш **CTRL**+**C**.


### <a name="redeploy-the-code-to-azure"></a>Повторное развертывание кода в Azure

Выполните следующую команду в корне репозитория:

```azurecli
az webapp up
```

Эта команда использует параметры, кэшированные в файле *.azure/config*. Так как Служба приложений обнаружит, что приложение уже существует, она просто повторно развернет в нем код.



### <a name="rerun-migrations-in-azure"></a>Повторное выполнение миграций в Azure

Поскольку вы внесли изменения в модель данных, все миграции базы данных нужно повторить в Службе приложений.

Еще раз откройте в браузере сеанс SSH, перейдя по адресу *https://\<app-name>.scm.azurewebsites.net/webssh/host*. Затем выполните следующие команды:

```
cd site/wwwroot

# Activate default virtual environment in App Service container
source /antenv/bin/activate
# Run database migrations
python manage.py migrate
```

### <a name="review-app-in-production"></a>Проверка приложения в рабочей среде

Перейдите по адресу *http:\//\<app-name>.azurewebsites.net* и снова проверьте приложение в рабочей среде. (Поскольку вы изменили только длину поля базы данных, это изменение заметно только при попытке ввести более длинный ответ при создании вопроса.)

> [!TIP]
> Вы можете использовать [django-storages](https://django-storages.readthedocs.io/en/latest/backends/azure.html) для хранения статических ресурсов и ресурсов мультимедиа в хранилище Azure. Вы можете использовать Azure CDN для упаковки статических файлов в пакеты формата GZIP.


## <a name="manage-your-app-in-the-azure-portal"></a>Управление приложением с помощью портала Azure

На [портале Azure](https://portal.azure.com) выполните поиск приложения по имени и выберите его.

:::image type="content" source="./media/tutorial-django-app-service-postgres/navigate-to-django-app-in-app-services-in-the-azure-portal.png" alt-text="Переход к приложению Python (Django) на портале Azure":::

По умолчанию на портале отображается страница **Обзор** приложения с общим представлением данных о производительности. Вы можете выполнять здесь базовые задачи управления, например просмотр, остановку, перезагрузку и удаление. На вкладках в левой части страницы отображаются различные страницы конфигурации, которые можно открыть.

:::image type="content" source="./media/tutorial-django-app-service-postgres/manage-django-app-in-app-services-in-the-azure-portal.png" alt-text="Управление приложением Python (Django) на странице &quot;Обзор&quot; на портале Azure":::


## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы хотите сохранить приложение или продолжить работу со следующим учебником, перейдите к [дальнейшим действиям](#next-steps). В противном случае, чтобы избежать постоянных расходов, вы можете удалить группу ресурсов, созданную для этого учебника:

```azurecli
az group delete -g myresourcegroup
```

Эта команда использует имя группы ресурсов, кэшированное в файле *.azure/config*. При удалении группы ресурсов будут также освобождены и удалены все ресурсы, содержащиеся в ней.

## <a name="next-steps"></a>Дальнейшие действия

Сведения о сопоставлении настраиваемого DNS-имени с вашим приложением:

> [!div class="nextstepaction"]
> [Руководство. Сопоставление настраиваемого DNS-имени с приложением](../../app-service/app-service-web-tutorial-custom-domain.md)

Узнайте, как Служба приложений запускает приложение Python:

> [!div class="nextstepaction"]
> [Настройка приложения Python](../../app-service/configure-language-python.md)
