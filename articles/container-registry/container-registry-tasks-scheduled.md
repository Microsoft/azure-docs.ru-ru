---
title: Руководство по планированию задачи записи контроля доступа
description: В этом руководстве описано, как запустить задачу реестра контейнеров Azure по заданному расписанию, задав один или несколько триггеров таймера.
ms.topic: article
ms.date: 11/24/2020
ms.openlocfilehash: 13a4ccac4ea97538583c1c063a6dc61e4d25686a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96030617"
---
# <a name="tutorial-run-an-acr-task-on-a-defined-schedule"></a>Учебник. Запуск задачи записи контроля доступа по определенному расписанию

В этом учебнике показано, как выполнить [задачу записи контроля](container-registry-tasks-overview.md) доступа по расписанию. Запланируйте задачу, настроив один или несколько *триггеров таймера*. Триггеры таймера можно использовать отдельно или в сочетании с другими триггерами задач.

В этом руководстве содержатся сведения о планировании задач и:

> [!div class="checklist"]
> * Создание задачи с триггером таймера
> * Управление триггерами таймера

Планирование задачи полезно в следующих сценариях:

* Запустите рабочую нагрузку контейнера для запланированных операций обслуживания. Например, запустите контейнерное приложение, чтобы удалить ненужные образы из реестра.
* Выполнение набора тестов на рабочем образе во время рабочего дня в рамках мониторинга активного сайта.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

## <a name="about-scheduling-a-task"></a>О планировании задачи

* **Триггер с помощью выражения cron** . триггер таймера для задачи использует *выражение cron*. Выражение представляет собой строку с пятью полями, задающими минуты, часы, дни, месяцы и дни недели для запуска задачи. Поддерживаются частоты в один раз в минуту.

  Например, выражение `"0 12 * * Mon-Fri"` запускает задачу в 12:00 по Гринвичу для каждого дня недели. [Дополнительные сведения](#cron-expressions) см. Далее в этой статье.
* **Несколько триггеров таймера** — Добавление нескольких таймеров в задачу разрешено при условии, что расписания различаются.
    * Укажите несколько триггеров таймера при создании задачи или добавьте их позже.
    * При необходимости задайте имя триггеров для упрощения управления, или задачи записи контроля доступа будут предоставлять имена триггеров по умолчанию.
    * Если расписания таймера перекрываются одновременно, задачи записи контроля доступа запускают задачу в запланированное время для каждого таймера.
* **Другие триггеры задач** . в задаче, активируемом с помощью таймера, можно также включить триггеры на основе изменений [исходного кода](container-registry-tutorial-build-task.md) или [базовых образов](container-registry-tutorial-base-image-update.md). Как и другие задачи контроля учетных записей, можно также [вручную запустить][az-acr-task-run] запланированную задачу.

## <a name="create-a-task-with-a-timer-trigger"></a>Создание задачи с триггером таймера

### <a name="task-command"></a>Командная строка задачи

Сначала заполните следующую переменную среды оболочки значением, подходящим для вашей среды. Этот шаг не является обязательным, но он упрощает выполнение многолинейных команд Azure CLI в этом руководстве. Если вы не заполните переменную среды, необходимо вручную заменить каждое значение в любом месте в примерах команд.

[![Внедрение запуска](https://shell.azure.com/images/launchcloudshell.png "Запуск Azure Cloud Shell")](https://shell.azure.com)

```console
ACR_NAME=<registry-name>        # The name of your Azure container registry
```

При создании задачи с помощью команды [AZ контроля доступа Task Create][az-acr-task-create] можно дополнительно добавить триггер таймера. Добавьте `--schedule` параметр и передайте выражение cron для таймера.

В качестве простого примера Следующая задача запускает запуск `hello-world` образа из реестра контейнеров Майкрософт каждый день в 21:00 UTC. Задача выполняется без контекста исходного кода.

```azurecli
az acr task create \
  --name timertask \
  --registry $ACR_NAME \
  --cmd mcr.microsoft.com/hello-world \
  --schedule "0 21 * * *" \
  --context /dev/null
```

Выполните команду [az acr task show][az-acr-task-show], чтобы убедиться, что триггер таймера настроен. По умолчанию также включен базовый триггер обновления образа.

```azurecli
az acr task show --name timertask --registry $ACR_NAME --output table
```

```output
NAME      PLATFORM    STATUS    SOURCE REPOSITORY       TRIGGERS
--------  ----------  --------  -------------------     -----------------
timertask linux       Enabled                           BASE_IMAGE, TIMER
```

## <a name="trigger-the-task"></a>Активация задачи

Запустите задачу вручную с помощью команды [AZ запись контроля][az-acr-task-run] доступа, чтобы убедиться, что она правильно настроена:

```azurecli
az acr task run --name timertask --registry $ACR_NAME
```

Если контейнер выполняется успешно, выходные данные похожи на приведенные ниже. В сокращенном примере выходных данных показаны только основные шаги.

```output
Queued a run with ID: cf2a
Waiting for an agent...
2020/11/20 21:03:36 Using acb_vol_2ca23c46-a9ac-4224-b0c6-9fde44eb42d2 as the home volume
2020/11/20 21:03:36 Creating Docker network: acb_default_network, driver: 'bridge'
[...]
2020/11/20 21:03:38 Launching container with name: acb_step_0

Hello from Docker!
This message shows that your installation appears to be working correctly.
[...]
```

По истечении запланированного времени выполните команду [AZ контроля доступа Task List-runs][az-acr-task-list-runs] , чтобы убедиться, что таймер активировал задачу ожидаемым образом:

```azurecli
az acr task list-runs --name timertask --registry $ACR_NAME --output table
```

При успешном выполнении таймера выходные данные должны выглядеть следующим образом:

```output
RUN ID    TASK       PLATFORM    STATUS     TRIGGER    STARTED               DURATION
--------  ---------  ----------  ---------  ---------  --------------------  ----------
ca15      timertask  linux       Succeeded  Timer      2020-11-20T21:00:23Z  00:00:06
ca14      timertask  linux       Succeeded  Manual     2020-11-20T20:53:35Z  00:00:06
```

## <a name="manage-timer-triggers"></a>Управление триггерами таймера

Используйте команды [AZ запись контроля][az-acr-task-timer] доступа для управления триггерами таймера для задачи контроля учетных записей.

### <a name="add-or-update-a-timer-trigger"></a>Добавление или обновление триггера таймера

После создания задачи можно добавить триггер таймера с помощью команды AZ контроля доступа к [таймеру добавления][az-acr-task-timer-add] . В следующем примере добавляется имя триггера таймера *timer2* в *тимертаск* , созданное ранее. Этот таймер активирует задачу каждый день в 10:30 UTC.

```azurecli
az acr task timer add \
  --name timertask \
  --registry $ACR_NAME \
  --timer-name timer2 \
  --schedule "30 10 * * *"
```

Обновите расписание существующего триггера или измените его состояние с помощью команды AZ запись контроля доступа для [обновления таймера][az-acr-task-timer-update] . Например, обновите триггер с именем *timer2* , чтобы активировать задачу в 11:30 UTC:

```azurecli
az acr task timer update \
  --name timertask \
  --registry $ACR_NAME \
  --timer-name timer2 \
  --schedule "30 11 * * *"
```

### <a name="list-timer-triggers"></a>Список триггеров таймера

Команда [AZ запись контроля доступа Task List][az-acr-task-timer-list] показывает триггеры таймера, настроенные для задачи:

```azurecli
az acr task timer list --name timertask --registry $ACR_NAME
```

Выходные данные примера:

```JSON
[
  {
    "name": "timer2",
    "schedule": "30 11 * * *",
    "status": "Enabled"
  },
  {
    "name": "t1",
    "schedule": "0 21 * * *",
    "status": "Enabled"
  }
]
```

### <a name="remove-a-timer-trigger"></a>Удаление триггера таймера

Используйте команду [AZ запись контроля][az-acr-task-timer-remove] доступа для удаления триггера таймера из задачи. В следующем примере удаляется триггер *timer2* из *тимертаск*:

```azurecli
az acr task timer remove \
  --name timertask \
  --registry $ACR_NAME \
  --timer-name timer2
```

## <a name="cron-expressions"></a>Выражения cron

Задачи записи контроля доступа используют библиотеку [нкронтаб](https://github.com/atifaziz/NCrontab) для интерпретации выражений cron. Поддерживаемые выражения в задачах контроля доступа имеют пять обязательных полей, разделенных пробелами:

`{minute} {hour} {day} {month} {day-of-week}`

Часовым поясом, используемым с выражениями cron, является время в формате UTC. Часы представлены в 24-часовом формате.

> [!NOTE]
> Задачи записи контроля доступа не поддерживают `{second}` `{year}` поле или в выражениях cron. При копировании выражения cron, используемого в другой системе, обязательно удалите эти поля, если они используются.

Каждое поле может принимать значение одного из следующих типов:

|Тип  |Пример  |Когда активируется  |
|---------|---------|---------|
|Определенное значение |<nobr>`"5 * * * *"`</nobr>|Каждый час за 5 минут после часа|
|Все значения (`*`)|<nobr>`"* 5 * * *"`</nobr>|каждую минуту начала часа, начиная с 5:00 UTC (60 раз в день)|
|Диапазон (оператор `-`)|<nobr>`"0 1-3 * * *"`</nobr>|3 раза в день, 1:00, 2:00 и 3:00 UTC|
|Набор значений (оператор `,`)|<nobr>`"20,30,40 * * * *"`</nobr>|3 раза в час, 20 минут, 30 минут и 40 минут после часа|
|Значение интервала (оператор `/`)|<nobr>`"*/10 * * * *"`</nobr>|6 раз в час, 10 минут, 20 минут и т. д. за час

[!INCLUDE [functions-cron-expressions-months-days](../../includes/functions-cron-expressions-months-days.md)]

### <a name="cron-examples"></a>Примеры cron

|Пример|Когда активируется  |
|---------|---------|
|`"*/5 * * * *"`|через каждые пять минут|
|`"0 * * * *"`|через каждый час|
|`"0 */2 * * *"`|через каждые 2 часа|
|`"0 9-17 * * *"`|Каждый час с 9:00 до 17:00 UTC|
|`"30 9 * * *"`|в 9:30 UTC каждый день|
|`"30 9 * * 1-5"`|в 9:30 UTC каждый будний день|
|`"30 9 * Jan Mon"`|в 9:30 UTC каждый понедельник в январе|

## <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы удалить все ресурсы, созданные в этой серии руководств, включая реестр контейнеров или реестров, экземпляр контейнера, хранилище ключей и субъект-службу, выполните следующие команды:

```azurecli
az group delete --resource-group $RES_GROUP
az ad sp delete --id http://$ACR_NAME-pull
```

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы узнали, как создавать задачи реестра контейнеров Azure, которые автоматически активируются таймером. 

Пример использования запланированной задачи для очистки репозиториев в реестре см. в статье [Автоматическое удаление образов из реестра контейнеров Azure](container-registry-auto-purge.md).

Примеры задач, активируемых фиксациями исходного кода или обновлениями базовых образов, см. в других статьях [руководства по задачам записи контроля](container-registry-tutorial-quick-task.md)доступа.



<!-- LINKS - External -->
[task-examples]: https://github.com/Azure-Samples/acr-tasks


<!-- LINKS - Internal -->
[az-acr-task-create]: /cli/azure/acr/task#az-acr-task-create
[az-acr-task-show]: /cli/azure/acr/task#az-acr-task-show
[az-acr-task-list-runs]: /cli/azure/acr/task#az-acr-task-list-runs
[az-acr-task-timer]: /cli/azure/acr/task/timer
[az-acr-task-timer-add]: /cli/azure/acr/task/timer#az-acr-task-timer-add
[az-acr-task-timer-remove]: /cli/azure/acr/task/timer#az-acr-task-timer-remove
[az-acr-task-timer-list]: /cli/azure/acr/task/timer#az-acr-task-timer-list
[az-acr-task-timer-update]: /cli/azure/acr/task/timer#az-acr-task-timer-update
[az-acr-task-run]: /cli/azure/acr/task#az-acr-task-run
[az-acr-task]: /cli/azure/acr/task
[azure-cli-install]: /cli/azure/install-azure-cli
