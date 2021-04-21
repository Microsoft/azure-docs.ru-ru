---
title: Руководство по Сборка и выполнение пользовательского образа в Службе приложений Azure
description: В этом пошаговом руководстве описано, как создать пользовательский образ Linux или Windows, отправить его в Реестр контейнеров Azure, а затем развернуть этот образ в Службе приложений Azure. Сведения о том, как выполнить перенос развертывания пользовательского программного обеспечения в Службу приложений в пользовательском контейнере.
ms.topic: tutorial
ms.date: 07/16/2020
ms.author: msangapu
keywords: служба приложений azure, веб-приложение, linux, windows, docker, контейнер
ms.custom: devx-track-csharp, mvc, seodec18, devx-track-python, devx-track-azurecli
zone_pivot_groups: app-service-containers-windows-linux
ms.openlocfilehash: 0770b46a60f497d3a3da772e7be13ece0526eca0
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107765707"
---
# <a name="migrate-custom-software-to-azure-app-service-using-a-custom-container"></a>Перенос пользовательского программного обеспечения в Службу приложений Azure с помощью пользовательского контейнера

::: zone pivot="container-windows"  

[Служба приложений Azure](overview.md) предоставляет предопределенные стеки приложений на платформе Windows, например ASP.NET или Node.js, выполняющиеся в IIS. Предварительно настроенная среда Windows блокирует в операционной системе возможность административного доступа, установки программного обеспечения, изменений в глобальном кэше сборок и т. д. (см. раздел [Функциональные возможности операционной системы для службы приложений Azure](operating-system-functionality.md)). Тем не менее с помощью пользовательского контейнера Windows в службе приложений можно вносить в ОС изменения, необходимые для вашего приложения. Поэтому легко перенести локальное приложение, которому требуется пользовательская ОС и конфигурация программного обеспечения. В этом руководстве показано, как перенести в службу приложений приложение ASP.NET, которое использует пользовательские шрифты, установленные в библиотеке шрифтов Windows. Вы развернете пользовательский образ Windows из Visual Studio в [Реестр контейнеров Azure](../container-registry/index.yml), а затем запустите его в службе приложений.

![Показано веб-приложение, выполняемое в контейнере Windows.](media/tutorial-custom-container/app-running.png)

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим руководством сделайте следующее:

- <a href="https://hub.docker.com/" target="_blank">зарегистрируйте учетную запись центра Docker</a>.
- <a href="https://docs.docker.com/docker-for-windows/install/" target="_blank">Установите Docker для ОС Windows</a>.
- <a href="/virtualization/windowscontainers/quick-start/quick-start-windows-10" target="_blank">Переключите Docker для запуска контейнеров Windows</a>.
- <a href="https://www.visualstudio.com/downloads/" target="_blank">Установите Visual Studio 2019</a>, а также следующие рабочие нагрузки: **ASP.NET и веб-разработка** и **разработка Azure**. Если у вас уже установлена версия Visual Studio 2019, сделайте следующее.
    - Установите последние обновления для Visual Studio, выбрав **Справка** > **Проверить наличие обновлений**.
    - Добавьте рабочие нагрузки в Visual Studio, выбрав **Инструменты** > **Get Tools and Features** (Получить инструменты и компоненты).

## <a name="set-up-the-app-locally"></a>Настройка приложения в локальной среде

### <a name="download-the-sample"></a>Скачивание примера приложения

На этом шаге вы настроите локальный проект .NET.

- [Загрузите пример проекта](https://github.com/Azure-Samples/custom-font-win-container/archive/master.zip).
- Извлеките (распакуйте) содержимое файла *custom-font-win-container.zip*.

Пример проекта содержит простое приложение ASP.NET, которое использует пользовательский шрифт, устанавливаемый в библиотеку шрифтов Windows. Устанавливать шрифты не обязательно, но это пример приложения, интегрированного с базовой ОС. Чтобы перенести такое приложение в службу приложений, переработайте код, чтобы удалить интеграцию, либо перенесите его "как есть" в пользовательском контейнере Windows.

### <a name="install-the-font"></a>Установка шрифта

В проводнике Windows перейдите в папку _custom-font-win контейнер master/CustomFontSample_, щелкните правой кнопкой мыши файл _FrederickatheGreat Regular.ttf_ и выберите **Установить**.

Этот шрифт находится в свободном доступе на веб-сайте [Google Fonts](https://fonts.google.com/specimen/Fredericka+the+Great).

### <a name="run-the-app"></a>Запустите приложение

Откройте файл *custom-font-win-container/CustomFontSample.sln* в Visual Studio. 

Введите `Ctrl+F5`, чтобы запустить приложение без отладки. Это приложение откроется в браузере по умолчанию. 

:::image type="content" source="media/tutorial-custom-container/local-app-in-browser.png" alt-text="Снимок экрана: приложение, отображаемое в браузере по умолчанию.":::

Так как приложение использует установленный шрифт, оно не может работать в песочнице службы приложений. Тем не менее его можно развернуть с помощью контейнера Windows, так как шрифт можно установить в контейнер Windows.

### <a name="configure-windows-container"></a>Настройка контейнера Windows

В обозревателе решений щелкните правой кнопкой мыши проект **CustomFontSample** и выберите **Добавить** > **Container Orchestration Support** (Поддержка оркестрации контейнеров).

:::image type="content" source="media/tutorial-custom-container/enable-container-orchestration.png" alt-text="Снимок экрана: окно &quot;Обозреватель решений&quot;, в котором отображаются выбранный проект CustomFontSample и элементы меню &quot;Добавить&quot; > &quot;Поддержка оркестратора контейнеров&quot;.":::

Выберите **Docker Compose** > **ОК**.

Теперь проект настроен для запуска в контейнере Windows. Объект _Dockerfile_ добавлен в проект **CustomFontSample**, а проект **docker-compose** добавлен в решение. 

В обозревателе решений откройте **Dockerfile**.

Необходимо использовать [поддерживаемый родительский образ](configure-custom-container.md#supported-parent-images). Измените родительский образ, заменив строку `FROM` приведенным ниже кодом.

```dockerfile
FROM mcr.microsoft.com/dotnet/framework/aspnet:4.7.2-windowsservercore-ltsc2019
```

В конце файла добавьте приведенную ниже строку, после чего сохраните файл.

```dockerfile
RUN ${source:-obj/Docker/publish/InstallFont.ps1}
```

Файл _InstallFont.ps1_ можно найти в проекте **CustomFontSample**. Это простой сценарий, который устанавливает шрифт. Более сложную версию этого сценария можно найти в [Центре сценариев](https://gallery.technet.microsoft.com/scriptcenter/fb742f92-e594-4d0c-8b79-27564c575133).

> [!NOTE]
> Чтобы локально проверить контейнер Windows, убедитесь, что на локальном компьютере запущен Docker.
>

## <a name="publish-to-azure-container-registry"></a>Публикация в Реестре контейнеров Azure

В [Реестре контейнеров Azure](../container-registry/index.yml) можно хранить образы для развертывания контейнеров. Можно настроить службу приложений для использования образов, размещенных в Реестре контейнеров Azure.

### <a name="open-publish-wizard"></a>Открытие мастера публикации

Щелкните правой кнопкой мыши проект **CustomFontSample** в обозревателе решений и выберите **Опубликовать**.

:::image type="content" source="media/tutorial-custom-container/open-publish-wizard.png" alt-text="Снимок экрана: окно &quot;Обозреватель решений&quot;, в котором выделены проект CustomFontSample и пункт меню &quot;Опубликовать&quot;.":::

### <a name="create-registry-and-publish"></a>Создание реестра и публикация

В мастере публикации выберите **Реестр контейнеров** > **Создать реестр контейнеров Azure** > **Опубликовать**.

:::image type="content" source="media/tutorial-custom-container/create-registry.png" alt-text="Снимок экрана: мастер публикации, в котором выделены такие элементы: &quot;Реестр контейнеров&quot;, &quot;Создать реестр контейнеров Azure&quot; и кнопка &quot;Опубликовать&quot;.":::

### <a name="sign-in-with-azure-account"></a>Вход с учетной записью Azure

В диалоговом окне **Создать реестр контейнеров Azure** выберите **Добавить учетную запись** и выполните вход в подписку Azure. Если вы уже вошли в систему, выберите учетную запись, содержащую необходимую подписку, в раскрывающемся меню.

![Вход в Azure](./media/tutorial-custom-container/add-an-account.png)

### <a name="configure-the-registry"></a>Настройка реестра

Настройте новый реестр контейнеров в соответствии с предлагаемыми значениями в следующей таблице. По завершении нажмите кнопку **Создать**.

| Параметр  | Рекомендуемое значение | Дополнительные сведения |
| ----------------- | ------------ | ----|
|**DNS-префикс**| Оставьте имя созданного реестра или измените его, указав другое уникальное имя. |  |
|**Группа ресурсов**| Щелкните **Создать**, введите имя **myResourceGroup** и нажмите кнопку **ОК**. |  |
|**SKU**| Basic | [Ценовые категории](https://azure.microsoft.com/pricing/details/container-registry/)|
|**Расположение реестра**| Западная Европа | |

![Настройка Реестра контейнеров Azure](./media/tutorial-custom-container/configure-registry.png)

Откроется окно терминала, в котором отображается ход развертывания образа. Дождитесь завершения развертывания.

## <a name="sign-in-to-azure"></a>Вход в Azure

Войдите на портал Azure по адресу https://portal.azure.com.

## <a name="create-a-web-app"></a>Создание веб-приложения

В меню слева выберите **Создать ресурс** > **Веб** > **Веб-приложение для контейнеров**.

### <a name="configure-app-basics"></a>Настройка основных сведений приложения

На вкладке **Основные сведения** настройте параметры в соответствии со следующей таблицей, а затем щелкните **Next: Docker** (Далее: Docker).

| Параметр  | Рекомендуемое значение | Дополнительные сведения |
| ----------------- | ------------ | ----|
|**Подписка**| Убедитесь, что отображается правильная подписка. |  |
|**Группа ресурсов**| Выберите **Создать**, введите **myResourceGroup** и щелкните **ОК**. |  |
|**имя**;| Введите уникальное имя. | URL-адрес веб-приложения: `http://<app-name>.azurewebsites.net`, где `<app-name>` — имя приложения. |
|**Опубликовать**| Контейнер Docker | |
|**Операционная система**| Windows | |
|**Регион**| Западная Европа | |
|**План Windows**| Выберите **Создать**, введите **myAppServicePlan** и щелкните **ОК**. | |

Вкладка **Основные сведения** должна выглядеть следующим образом:

![Показана вкладка "Основные сведения", которая используется для настройки веб-приложения.](media/tutorial-custom-container/configure-app-basics.png)

### <a name="configure-windows-container"></a>Настройка контейнера Windows

На вкладке **Docker** настройте свой пользовательский контейнер Windows, как показано в следующей таблице, и выберите **Просмотр и создание**.

| Параметр  | Рекомендуемое значение |
| ----------------- | ------------ |
|**Источник образа**| Реестр контейнеров Azure |
|**Реестр**| Выберите [созданный ранее реестр](#publish-to-azure-container-registry). |
|**Изображение**| customfontsample |
|**Тег**| последняя |

### <a name="complete-app-creation"></a>Завершение создания приложения

Нажмите кнопку **Создать** и подождите, пока Azure создаст необходимые ресурсы.

## <a name="browse-to-the-web-app"></a>Переход к веб-приложению

По завершении операции Azure отображается окно уведомления.

![Показано сообщение о том, что операция Azure завершена.](media/tutorial-custom-container/portal-create-finished.png)

1. Щелкните **Перейти к ресурсу**.

2. На странице приложения щелкните ссылку в разделе **URL-адрес**.

В браузере откроется следующая страница:

![Показана новая страница браузера для веб-приложения.](media/tutorial-custom-container/app-starting.png)

Подождите несколько минут и попробуйте еще раз, пока не откроется домашняя страница с тем самым красивым шрифтом.

![Показана домашняя страница со шрифтом, который вы настроили.](media/tutorial-custom-container/app-running.png)

**Поздравляем!** Вы перенесли приложение ASP.NET в Службу приложений Azure в контейнере Windows.

## <a name="see-container-start-up-logs"></a>Просмотр журналов запуска контейнера

Загрузка контейнера Windows может занять некоторое время. Чтобы просмотреть ход выполнения, перейдите по следующему URL-адресу, заменив *\<app-name>* именем приложения.
```
https://<app-name>.scm.azurewebsites.net/api/logstream
```

Потоковые журналы выглядят следующим образом:

```
14/09/2018 23:16:19.889 INFO - Site: fonts-win-container - Creating container for image: customfontsample20180914115836.azurecr.io/customfontsample:latest.
14/09/2018 23:16:19.928 INFO - Site: fonts-win-container - Create container for image: customfontsample20180914115836.azurecr.io/customfontsample:latest succeeded. Container Id 329ecfedbe370f1d99857da7352a7633366b878607994ff1334461e44e6f5418
14/09/2018 23:17:23.405 INFO - Site: fonts-win-container - Start container succeeded. Container: 329ecfedbe370f1d99857da7352a7633366b878607994ff1334461e44e6f5418
14/09/2018 23:17:28.637 INFO - Site: fonts-win-container - Container ready
14/09/2018 23:17:28.637 INFO - Site: fonts-win-container - Configuring container
14/09/2018 23:18:03.823 INFO - Site: fonts-win-container - Container ready
14/09/2018 23:18:03.823 INFO - Site: fonts-win-container - Container start-up and configuration completed successfully
```

::: zone-end

::: zone pivot="container-linux"

Служба приложений Azure использует технологию контейнеров Docker для размещения встроенных и пользовательских образов. Чтобы просмотреть список встроенных образов, выполните команду Azure CLI [az webapp list-runtimes --linux](/cli/azure/webapp#az_webapp_list_runtimes). Если эти образы не соответствуют вашим требованиям, вы можете создать и развернуть пользовательский образ.

В этом руководстве описано следующее:

> [!div class="checklist"]
> * создание пользовательского образа, если ни один из встроенных не соответствует вашим требованиям;
> * отправка пользовательского образа в частный реестр контейнеров в Azure;
> * Запуск настраиваемого образа в Службе приложений.
> * Настройка переменных среды
> * Обновление и повторное развертывание образа.
> * Доступ к журналам диагностики
> * Подключение контейнера с помощью SSH.

Выполнение инструкций из этого руководства связано с небольшими расходами для учетной записи Azure за создание реестра контейнеров, и, возможно, за размещение контейнера на срок более месяца.

## <a name="set-up-your-initial-environment"></a>Настройка начальной среды

- Подготовьте учетную запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) бесплатно.
- Установите [Docker](https://docs.docker.com/get-started/#setup) для создания образов Docker. Для установки Docker может потребоваться перезагрузка компьютера.
[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]
- Для работы с этим учебником требуется Azure CLI версии 2.0.80 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

После установки Docker или запуска Azure Cloud Shell откройте окно терминала и убедитесь, что Docker установлен:

```bash
docker --version
```

## <a name="clone-or-download-the-sample-app"></a>Клонирование или скачивание примера приложения

Чтобы получить пример для работы с этим руководством, его можно клонировать с помощью Git или скачать.

### <a name="clone-with-git"></a>Клонирование с помощью Git

Клонируйте репозиторий примера.

```terminal
git clone https://github.com/Azure-Samples/docker-django-webapp-linux.git --config core.autocrlf=input
```

Не забудьте включить аргумент `--config core.autocrlf=input`, чтобы обеспечить правильное завершение строк в файлах, которые используются в контейнере Linux.

Затем перейдите в эту папку:

```terminal
cd docker-django-webapp-linux
```

### <a name="download-from-github"></a>Загрузка с GitHub

Вместо клонирования с помощью Git вы можете открыть страницу [https://github.com/Azure-Samples/docker-django-webapp-linux](https://github.com/Azure-Samples/docker-django-webapp-linux), а затем выбрать действие **Clone** (Клонировать) и **Download ZIP** (Скачать ZIP-файл). 

Распакуйте ZIP-файл в папку с именем *docker-django-webapp-linux*. 

Затем откройте окно терминала в этой папке *docker-django-webapp-linux*.

## <a name="optional-examine-the-docker-file"></a>(Необязательно) Анализ файл Docker

В нашем примере файл с именем _Dockerfile_ описывает образ Docker и содержит инструкции по настройке.

```Dockerfile
FROM tiangolo/uwsgi-nginx-flask:python3.6

RUN mkdir /code
WORKDIR /code
ADD requirements.txt /code/
RUN pip install -r requirements.txt --no-cache-dir
ADD . /code/

# ssh
ENV SSH_PASSWD "root:Docker!"
RUN apt-get update \
        && apt-get install -y --no-install-recommends dialog \
        && apt-get update \
    && apt-get install -y --no-install-recommends openssh-server \
    && echo "$SSH_PASSWD" | chpasswd 

COPY sshd_config /etc/ssh/
COPY init.sh /usr/local/bin/
    
RUN chmod u+x /usr/local/bin/init.sh
EXPOSE 8000 2222

#CMD ["python", "/code/manage.py", "runserver", "0.0.0.0:8000"]
ENTRYPOINT ["init.sh"]
```

* Первая группа команд устанавливает в среде обязательные для приложения компоненты.
* Вторая группа команд создает сервер [SSH](https://www.ssh.com/ssh/protocol/) для безопасного взаимодействия между контейнером и узлом.
* В последней строке файла (`ENTRYPOINT ["init.sh"]`) вызывается `init.sh` для запуска службы SSH и сервера Python.

## <a name="build-and-test-the-image-locally"></a>Сборка и тестирование образа в локальной среде

> [!NOTE]
> Docker Hub имеет [квоты на число анонимных вытягиваний с одного IP-адреса и на число аутентифицированных вытягиваний от одного бесплатного пользователя (см. сведения о **передаче данных**)](https://www.docker.com/pricing). Если вы заметили, что число вытягиваний из Docker Hub ограничено, попробуйте выполнить команду `docker login`, если вы еще не вошли в систему.
> 

1. Запустите сборку образа с помощью следующей команды.

    ```bash
    docker build --tag appsvc-tutorial-custom-image .
    ```
    
1. Убедитесь, что сборка работает, запустив контейнер Docker на локальном компьютере.

    ```bash
    docker run -p 8000:8000 appsvc-tutorial-custom-image
    ```
    
    Эта команда [`docker run`](https://docs.docker.com/engine/reference/commandline/run/) задает порт с помощью аргумента `-p`, за которым следует имя образа. 
    
    > [!TIP]
    > Если вы работаете в среде Windows и видите ошибку вида *standard_init_linux.go:211: exec user process caused "no such file or directory"* , значит в файле *init.sh* для завершения строк используется CR-LF, а не ожидаемый вариант LF. Эта ошибка возникает, если вы клонировали пример репозитория с помощью Git, но не включили параметр `--config core.autocrlf=input`. В этом случае повторите клонирование репозитория с правильным аргументом --config. Вы также можете получить аналогичную ошибку, если редактировали файл *init.sh* и сохранили его с использованием CR-LF для завершения. В этом случае сохраните его снова, используя LF.

1. Перейдите по адресу `http://localhost:8000` и убедитесь, что веб-приложение и контейнер функционируют правильно.

    ![Локальное тестирование веб-приложения](./media/app-service-linux-using-custom-docker-image/app-service-linux-browse-local.png)

## <a name="create-a-resource-group"></a>Создание группы ресурсов

В этом и последующих разделах показано, как подготовить в Azure ресурсы, затем отправить в них образ и развернуть контейнер в Службе приложений Azure. Для начала создайте группу ресурсов, в которой будут собраны все эти ресурсы.

Выполните команду [az group create](/cli/azure/group#az_group_create), чтобы создать группу ресурсов.

```azurecli-interactive
az group create --name AppSvc-DockerTutorial-rg --location westus2
```

Вы можете изменить значение `--location`, чтобы указать близкий к вам регион.

## <a name="push-the-image-to-azure-container-registry"></a>Принудительная отправка образов в Реестр контейнеров Azure

В этом разделе вы отправите образ в Реестр контейнеров Azure, откуда Служба приложений сможет его развернуть.

1. Выполните команду [`az acr create`](/cli/azure/acr#az_acr_create), чтобы создать Реестр контейнеров Azure.

    ```azurecli-interactive
    az acr create --name <registry-name> --resource-group AppSvc-DockerTutorial-rg --sku Basic --admin-enabled true
    ```
    
    Замените `<registry-name>` понятным именем реестра. Это имя может содержать только буквы и цифры и должно быть уникальным в пределах Azure.

1. Выполните команду [`az acr show`](/cli/azure/acr#az_acr_show), чтобы получить учетные данные для этого реестра.

    ```azurecli-interactive
    az acr credential show --resource-group AppSvc-DockerTutorial-rg --name <registry-name>
    ```
    
    Выходные данные этой команды содержат два пароля и имя пользователя для реестра в формате JSON.
    
1. Выполните команду `docker login` для входа в реестр контейнеров:

    ```bash
    docker login <registry-name>.azurecr.io --username <registry-username>
    ```
    
    Замените `<registry-name>` и `<registry-username>` значениями из предыдущих шагов. При появлении запроса введите один из паролей из предыдущего шага.

    Используйте одно и то же имя реестра во всех остальных шагах этого раздела.

1. Войдя, присвойте локальному образу Docker тег для реестра.

    ```bash
   docker tag appsvc-tutorial-custom-image <registry-name>.azurecr.io/appsvc-tutorial-custom-image:latest
    ```    

1. С помощью команды `docker push` отправьте образ в реестр.

    ```bash
    docker push <registry-name>.azurecr.io/appsvc-tutorial-custom-image:latest
    ```

    Первая отправка образа может занять несколько минут, так как она включает базовый образ. Все последующие отправки обычно выполняются быстрее.

    Пока вы ждете завершения, вы можете выполнить шаги, описанные в следующем разделе, то есть настроить развертывание из реестра в Службе приложений.

1. Чтобы проверить успешность принудительной отправки, выполните команду `az acr repository list`.

    ```azurecli-interactive
    az acr repository list -n <registry-name>
    ```
    
    В выходных данных должно отобразиться имя отправленного образа.


## <a name="configure-app-service-to-deploy-the-image-from-the-registry"></a>Настройка развертывания образа из реестра в Службе приложений

Чтобы развернуть контейнер в Службе приложений Azure, сначала создайте веб-приложение в Службе приложений, а затем подключите это веб-приложение к реестру контейнеров. При запуске веб-приложения Служба приложений автоматически извлечет этот образ из реестра.

1. Создайте план Службы приложений с помощью команды [`az appservice plan create`](/cli/azure/appservice/plan#az_appservice_plan_create).

    ```azurecli-interactive
    az appservice plan create --name AppSvc-DockerTutorial-plan --resource-group AppSvc-DockerTutorial-rg --is-linux
    ```

    План Службы приложений соответствует виртуальной машине, в которой размещено веб-приложение. По умолчанию предыдущая команда использует бюджетную [ценовую категорию B1](https://azure.microsoft.com/pricing/details/app-service/linux/), которая предоставляется в течение первого месяца бесплатно. Уровень можно изменить с помощью параметра `--sku`.

1. Создайте веб-приложение с помощью команды [`az webpp create`](/cli/azure/webapp#az_webapp_create).

    ```azurecli-interactive
    az webapp create --resource-group AppSvc-DockerTutorial-rg --plan AppSvc-DockerTutorial-plan --name <app-name> --deployment-container-image-name <registry-name>.azurecr.io/appsvc-tutorial-custom-image:latest
    ```
    
    Замените здесь `<app-name>` именем веб-приложения, которое должно быть уникальным в пределах Azure. Также замените `<registry-name>` именем реестра из предыдущего раздела.

1. Используйте [`az webapp config appsettings set`](/cli/azure/webapp/config/appsettings#az_webapp_config_appsettings_set), чтобы задать переменную среды `WEBSITES_PORT`, которую ожидает код приложения. 

    ```azurecli-interactive
    az webapp config appsettings set --resource-group AppSvc-DockerTutorial-rg --name <app-name> --settings WEBSITES_PORT=8000
    ```

    Замените `<app-name>` значением, которое вы использовали на предыдущем шаге.
    
    Дополнительные сведения об этой переменной среды см. в [файле сведений в репозитории этого примера](https://github.com/Azure-Samples/docker-django-webapp-linux) на сайте GitHub.

1. Включите [управляемое удостоверение](./overview-managed-identity.md) для веб-приложения, выполнив команду [`az webapp identity assign`](/cli/azure/webapp/identity#az_webapp_identity-assign).

    ```azurecli-interactive
    az webapp identity assign --resource-group AppSvc-DockerTutorial-rg --name <app-name> --query principalId --output tsv
    ```

    Замените `<app-name>` значением, которое вы использовали на предыдущем шаге. Отфильтровав выходные данные этой команды с помощью аргументов `--query` и `--output`, вы найдете субъект-службу назначенного удостоверения, который вам вскоре потребуется.

    Управляемое удостоверение позволяет предоставить веб-приложению разрешения на доступ к другим ресурсам Azure, не создавая специальные учетные данные.

1. Получите с помощью команды [`az account show`](/cli/azure/account#az_account_show) идентификатор подписки, который вам понадобится на следующем шаге.

    ```azurecli-interactive
    az account show --query id --output tsv
    ``` 

1. Предоставьте веб-приложению разрешение на доступ к реестру контейнеров.

    ```azurecli-interactive
    az role assignment create --assignee <principal-id> --scope /subscriptions/<subscription-id>/resourceGroups/AppSvc-DockerTutorial-rg/providers/Microsoft.ContainerRegistry/registries/<registry-name> --role "AcrPull"
    ```

    Измените следующие значения:
    - `<principal-id>` на идентификатор субъекта-службы, полученный командой `az webapp identity assign`;
    - `<registry-name>` на имя реестра контейнеров;
    - `<subscription-id>` на идентификатор подписки, полученный командой `az account show`.

Дополнительные сведения об этих разрешениях см. в статье [Общие сведения об управлении доступом на основе ролей (RBAC) для ресурсов Azure](../role-based-access-control/overview.md). 

## <a name="deploy-the-image-and-test-the-app"></a>Развертывание образа и тестирование приложения

Вы сможете выполнить эти шаги, когда образ будет передан в реестр контейнеров, а Служба приложений полностью подготовлена.

1. С помощью команды [`az webapp config container set`](/cli/azure/webapp/config/container#az_webapp_config_container_set) укажите реестр контейнеров и образ, который нужно развернуть для веб-приложения.

    ```azurecli-interactive
    az webapp config container set --name <app-name> --resource-group AppSvc-DockerTutorial-rg --docker-custom-image-name <registry-name>.azurecr.io/appsvc-tutorial-custom-image:latest --docker-registry-server-url https://<registry-name>.azurecr.io
    ```
    
    Замените `<app_name>` именем веб-приложения, а два вхождения `<registry-name>` — именем реестра. 

    - При использовании реестра, отличающегося от Docker Hub (как в представленном примере), значение `--docker-registry-server-url` должно быть отформатировано, чтобы содержать `https://` и полное доменное имя реестра.
    - Сообщение No credential was provided to access Azure Container Registry. Trying to look up... (Учетные данные для доступа к Реестру контейнеров Azure не предоставлены. Выполняется поиск...) означает, что Azure использует для проверки подлинности в реестре контейнеров управляемое удостоверение, а не запрашивает имя и пароль.
    - Если вы встретите ошибку AttributeError: 'NoneType' object has no attribute 'reserved' (AttributeError: объект NoneType не имеет атрибута reserved), проверьте правильность `<app-name>`.

    > [!TIP]
    > Вы можете в любой момент получить параметры контейнера для веб-приложения с помощью команды `az webapp config container show --name <app-name> --resource-group AppSvc-DockerTutorial-rg`. Этот образ задается в свойстве `DOCKER_CUSTOM_IMAGE_NAME`. Когда веб-приложение развертывается через шаблоны Azure DevOps или Azure Resource Manager, этот же образ может отображаться и в свойстве с именем `LinuxFxVersion`. Оба этих свойства имеют одинаковое назначение. Если они оба присутствуют в конфигурации веб-приложения, приоритет отдается `LinuxFxVersion`.

1. Когда команда `az webapp config container set` будет выполнена, веб-приложение должно работать в контейнере в Службе приложений.

    Чтобы проверить приложение, перейдите по адресу `http://<app-name>.azurewebsites.net`, заменив `<app-name>` именем веб-приложения. При первом входе ответ от приложения может поступить с некоторой задержкой, так как Служба приложений должна получить из реестра полный образ. Если время ожидания браузера истечет, просто обновите страницу. После получения начального образа все последующие тесты выполняются намного быстрее.

    ![Успешный тест веб-приложения в Azure](./media/app-service-linux-using-custom-docker-image/app-service-linux-browse-azure.png)

## <a name="modify-the-app-code-and-redeploy"></a>Изменение кода и повторное развертывание приложения

В этом разделе вы внесете изменения в код веб-приложения, перестроите контейнер и отправите его в реестр. Затем Служба приложений автоматически извлечет обновленный образ из реестра и обновит работающее веб-приложение.

1. В локальной папке *docker-django-webapp-linux* откройте файл *app/templates/app/index.html*.

1. Измените первый элемент HTML в соответствии со следующим кодом.

    ```html
    <nav class="navbar navbar-inverse navbar-fixed-top">
      <div class="container">
        <div class="navbar-header">
          <a class="navbar-brand" href="#">Azure App Service - Updated Here!</a>
        </div>
      </div>
    </nav>
    ```
    
1. Сохраните изменения.

1. Перейдите в папку *docker-django-webapp-linux* и перестройте образ.

    ```bash
    docker build --tag appsvc-tutorial-custom-image .
    ```

1. В теге этого образа укажите новый тег номера версии (v1.0.1).

    ```bash
    docker tag appsvc-tutorial-custom-image <registry-name>.azurecr.io/appsvc-tutorial-custom-image:latest
    ```

    Замените `<registry-name>` именем реестра.

1. Отправьте образ в реестр:

    ```bash
    docker push <registry-name>.azurecr.io/appsvc-tutorial-custom-image:latest
    ```

1. Перезапустите веб-приложение:

    ```azurecli-interactive
    az webapp restart --name <app_name> --resource-group AppSvc-DockerTutorial-rg
    ```

    Замените `<app_name>` именем веб-приложения. После перезагрузки Служба приложений извлечет обновленный образ из реестра контейнеров.

1. Проверьте успешность развертывания обновления, перейдя по адресу `http://<app-name>.azurewebsites.net`.

## <a name="access-diagnostic-logs"></a>Доступ к журналам диагностики

1. Включите ведение журнала для контейнера.

    ```azurecli-interactive
    az webapp log config --name <app-name> --resource-group AppSvc-DockerTutorial-rg --docker-container-logging filesystem
    ```
    
1. Включите потоковую передачу журналов.

    ```azurecli-interactive
    az webapp log tail --name <app-name> --resource-group AppSvc-DockerTutorial-rg
    ```
    
    Если журналы консоли не отображаются, проверьте еще раз через 30 секунд.

    Вы также можете проверить файлы журнала в браузере на странице `https://<app-name>.scm.azurewebsites.net/api/logs/docker`.

1. Чтобы остановить потоковую передачу журналов, нажмите клавиши **CTRL**+**C**.

## <a name="connect-to-the-container-using-ssh"></a>Подключение контейнера с помощью SSH.

SSH обеспечивает безопасный обмен данными между клиентом и контейнером. Чтобы включить SSH-подключение к контейнеру, для него необходимо настроить настраиваемый образ. После запуска контейнера можно открыть SSH-подключение к нему.

### <a name="configure-the-container-for-ssh"></a>Настройка SSH для контейнера

Пример приложения в этом руководстве уже имеет *Dockerfile* с необходимой конфигурацией, которая устанавливает SSH-сервер и задает учетные данные для входа. Этот раздел не содержит практических действий. Чтобы подключиться к контейнеру, перейдите к следующему разделу.

```Dockerfile
ENV SSH_PASSWD "root:Docker!"
RUN apt-get update \
        && apt-get install -y --no-install-recommends dialog \
        && apt-get update \
  && apt-get install -y --no-install-recommends openssh-server \
  && echo "$SSH_PASSWD" | chpasswd 
```

> [!NOTE]
> Эта конфигурация не допускает внешние подключения к контейнеру. SSH доступен только на сайте Kudu и на сайте SCM. Сайты Kudu и SCM выполняют аутентификацию с вашей учетной записью Azure.

Также этот *Dockerfile* копирует файл *sshd_config* в папку */etc/ssh/* и предоставляет порт 2222 в контейнере.

```Dockerfile
COPY sshd_config /etc/ssh/

# ...

EXPOSE 8000 2222
```

Внутренний порт 2222 доступен контейнерам внутри мостовой сети виртуальной частной сети. 

Наконец, начальный сценарий *init.sh* запускает SSH-сервер.

```bash
#!/bin/bash
service ssh start
```

### <a name="open-ssh-connection-to-container"></a>Открытие SSH-подключения к контейнеру

1. Чтобы установить подключение, перейдите по адресу `https://<app-name>.scm.azurewebsites.net/webssh/host` и войдите с учетной записью Azure. Замените `<app-name>` именем веб-приложения.

1. Войдя, вы попадете на информационную страницу веб-приложения. Выберите **SSH** в верхней части страницы, чтобы открыть оболочку и выполнить команды.

    Например, здесь можно изучить выполняемые процессы с помощью команды `top`.
    
## <a name="clean-up-resources"></a>Очистка ресурсов

За использование ресурсов, созданных с помощью этой статьи, может взиматься плата. Чтобы очистить ресурсы, достаточно удалить только группу ресурсов, которая их содержит.

```azurecli
az group delete --name AppSvc-DockerTutorial-rg
```

::: zone-end

## <a name="next-steps"></a>Дальнейшие действия

Вы научились выполнять следующие задачи:

> [!div class="checklist"]
> * Развертывание настраиваемого образа в частном реестре контейнеров.
> * Развертывание и запуск пользовательского образа в Службе приложений
::: zone pivot="container-linux"
> * Обновление и повторное развертывание образа.
::: zone-end
> * Доступ к журналам диагностики
::: zone pivot="container-linux"
> * Подключение контейнера с помощью SSH.
::: zone-end

Перейдите к следующему руководству, в котором описано, как сопоставлять пользовательские DNS-имена с приложением.

> [!div class="nextstepaction"]
> [Руководство. Сопоставление настраиваемого DNS-имени с приложением](app-service-web-tutorial-custom-domain.md)

Также ознакомьтесь с другими ресурсами:

> [!div class="nextstepaction"]
> [Настройка пользовательского контейнера](configure-custom-container.md)

::: zone pivot="container-linux"
> [!div class="nextstepaction"]
> [Руководство. по приложению WordPress с несколькими контейнерами](tutorial-multi-container-app.md)
::: zone-end
