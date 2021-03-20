---
title: YAML Reference — задачи записи контроля доступа
description: Справка по определению задач Реестра контейнеров Azure в YAML, включая свойства задач, типы шагов, свойства шагов и встроенные переменные.
ms.topic: article
ms.date: 07/08/2020
ms.openlocfilehash: 042310d29f5561c2cd77b0b9cccfc587ca4aa767
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88067589"
---
# <a name="acr-tasks-reference-yaml"></a>Справка по задачам Реестра контейнеров Azure: YAML

Определение многошаговой задачи в службе "Задачи ACR" предоставляет ориентированный на контейнеры вычислительный примитив, направленный на сборку, тестирование и исправление контейнеров. В этой статье рассматриваются команды, параметры, свойства и синтаксис для файлов YAML, определяющих многошаговые задачи.

Эта статья содержит справочные сведения о создании YAML-файлов многошаговых задач для службы "Задачи ACR". Общие сведения о задачах ACR см. в [обзоре службы "Задачи ACR"](container-registry-tasks-overview.md).

## <a name="acr-taskyaml-file-format"></a>Формат файла acr-task.yaml

Служба "Задачи ACR" поддерживает объявление многошаговой задачи в стандартном синтаксисе YAML. Шаги задачи определяются в файле YAML. Затем эту задачу можно запустить вручную, передав файл в команду AZ запись [контроля][az-acr-run] доступа. Или используйте файл для создания задачи с помощью команды AZ записи контроля доступа, [созданной][az-acr-task-create] автоматически при фиксации Git, обновлении базового образа или по расписанию. Несмотря на то, что в этой статье `acr-task.yaml` рассматривается как файл, содержащий шаги, служба "Задачи ACR" поддерживает любое допустимое имя файла с [поддерживаемым расширением](#supported-task-filename-extensions).

Примитивами `acr-task.yaml` верхнего уровня являются **свойства задачи**, **типы шагов** и **свойства шага**.

* [Свойства задачи](#task-properties) применяются для всех шагов на протяжении всего выполнения задачи. Существует несколько глобальных свойств задачи, включая следующие.
  * `version`
  * `stepTimeout`
  * `workingDirectory`
* [Типы шагов задачи](#task-step-types) представляют собой типы действий, которые могут быть выполнены в задаче. Имеются три типа шагов:
  * `build`
  * `push`
  * `cmd`
* [Свойства шага задачи](#task-step-properties) — параметры, которые применяются к отдельному шагу. Существует несколько свойств шага, в том числе:
  * `startDelay`
  * `timeout`
  * `when`
  * ...и многие другие.

Ниже приведен пример базового формата файла `acr-task.yaml`, включая некоторые общие свойства шага. Хотя он не является исчерпывающим представлением всех доступных свойств шага или использования типа шага, он предоставляет краткий обзор базового формата файла.

```yml
version: # acr-task.yaml format version.
stepTimeout: # Seconds each step may take.
steps: # A collection of image or container actions.
  - build: # Equivalent to "docker build," but in a multi-tenant environment
  - push: # Push a newly built or retagged image to a registry.
    when: # Step property that defines either parallel or dependent step execution.
  - cmd: # Executes a container, supports specifying an [ENTRYPOINT] and parameters.
    startDelay: # Step property that specifies the number of seconds to wait before starting execution.
```

### <a name="supported-task-filename-extensions"></a>Поддерживаемые расширения имен файлов задач

Для службы "Задачи ACR" зарезервированы несколько расширений имен файлов, включая `.yaml`, которые будут обрабатываться как файл задачи. Любое расширение, *отсутствующее* в следующем списке, служба "Задачи ACR" относит к Dockerfile: .yaml, .yml, .toml, .json, .sh, .bash, .zsh, .ps1, .ps, .cmd, .bat, .ts, .js, .php, .py, .rb, .lua

YAML — единственный формат файлов, который сейчас поддерживает служба "Задачи ACR". Другие расширения имен файлов зарезервированы для возможной будущей поддержки.

## <a name="run-the-sample-tasks"></a>Запуск примеров задач

В следующих разделах этой статьи приводятся ссылки на несколько примеров файлов задач. Примеры задач расположены в общедоступном репозитории GitHub [Azure-Samples/acr-tasks][acr-tasks]. Их можно запустить с помощью команды [az acr run][az-acr-run] в Azure CLI. Примеры команд выглядят приблизительно так:

```azurecli
az acr run -f build-push-hello-world.yaml https://github.com/Azure-Samples/acr-tasks.git
```

Форматирование в примерах команд предполагает, что вы уже настроили реестр по умолчанию в Azure CLI, поэтому параметр `--registry` опускается. Чтобы настроить реестр по умолчанию, воспользуйтесь командой [az configure][az-configure] с параметром `--defaults`, который принимает значение `acr=REGISTRY_NAME`.

Например, чтобы настроить Azure CLI с реестром myregistry по умолчанию, выполните следующую команду:

```azurecli
az configure --defaults acr=myregistry
```

## <a name="task-properties"></a>Свойства задачи

Свойства задачи обычно отображаются в верхней части `acr-task.yaml` файла и являются глобальными свойствами, которые применяются во время полного выполнения шагов задачи. Некоторые из этих глобальных свойств могут быть переопределены в рамках отдельного шага.

| Свойство. | Type | Необязательно | Описание | Поддерживается ли переопределение | Значение по умолчанию |
| -------- | ---- | -------- | ----------- | ------------------ | ------------- |
| `version` | строка | Да | Версия файла `acr-task.yaml`, проанализированного службой "Задачи ACR". В то время как служба "Задачи ACR" стремится поддерживать обратную совместимость, это значение позволяет службе поддерживать совместимость в пределах заданной версии. Если значение не указано, по умолчанию используется последняя версия. | Нет | None |
| `stepTimeout` | int (секунды) | Да | Максимальное число секунд выполнения шага. Если свойство задано в задаче, оно устанавливает свойство по умолчанию `timeout` для всех шагов. Если `timeout` свойство задано на шаге, оно переопределяет свойство, предоставленное задачей. | Да | 600 (10 минут) |
| `workingDirectory` | строка | Да | Рабочий каталог контейнера во время выполнения. Если свойство задано в задаче, оно устанавливает свойство по умолчанию `workingDirectory` для всех шагов. Если этот параметр указан на шаге, он переопределяет свойство, предоставленное задачей. | Да | `c:\workspace` в Windows или `/workspace` Linux |
| `env` | [строка, строка, ...] | Да |  Массив строк в `key=value` формате, который определяет переменные среды для задачи. Если свойство задано в задаче, оно устанавливает свойство по умолчанию `env` для всех шагов. Если этот параметр указан на шаге, он переопределяет все переменные среды, унаследованные от задачи. | Да | Нет |
| `secrets` | [секретный, секретный,...] | Да | Массив [секретных](#secret) объектов. | Нет | None |
| `networks` | [сеть, сеть,...] | Да | Массив [сетевых](#network) объектов. | Нет | None |
| `volumes` | [том, том,...] | Да | Массив объектов [тома](#volume) . Указывает тома с исходным содержимым для подключения к шагу. | Нет | None |

### <a name="secret"></a>secret

Секретный объект имеет следующие свойства.

| Свойство. | Type | Необязательно | Описание | Значение по умолчанию |
| -------- | ---- | -------- | ----------- | ------- |
| `id` | Строка | Нет | Идентификатор секрета. | Нет |
| `keyvault` | строка | Да | URL-адрес Azure Key Vault секрета. | Нет |
| `clientID` | строка | Да | Идентификатор клиента [управляемого удостоверения, назначаемого пользователем](container-registry-tasks-authentication-managed-identity.md) для ресурсов Azure. | Нет |

### <a name="network"></a>network

Объект Network имеет следующие свойства.

| Свойство. | Type | Необязательно | Описание | Значение по умолчанию |
| -------- | ---- | -------- | ----------- | ------- | 
| `name` | Строка | Нет | Имя сети. | Нет |
| `driver` | строка | Да | Драйвер для управления сетью. | Нет |
| `ipv6` | bool | Да | Включена ли сеть IPv6. | `false` |
| `skipCreation` | bool | Да | Указывает, следует ли пропустить создание сети. | `false` |
| `isDefault` | bool | Да | Является ли сеть сетью по умолчанию, предоставляемой реестром контейнеров Azure. | `false` |

### <a name="volume"></a>том

Объект тома имеет следующие свойства.

| Свойство. | Type | Необязательно | Описание | Значение по умолчанию |
| -------- | ---- | -------- | ----------- | ------- | 
| `name` | Строка | Нет | Имя тома для подключения. Может содержать только буквы, цифры, символы "-" и "_". | Нет |
| `secret` | Map [строка] строка | Нет | Каждый ключ на карте — это имя файла, созданного и заполняемого в томе. Каждое значение является строковой версией секрета. Значения секрета должны быть в кодировке Base64. | Нет |

## <a name="task-step-types"></a>Типы шагов задач

Служба "Задачи ACR" поддерживает три типа шагов. Каждый тип шага поддерживает несколько свойств, подробно описанных в соответствующих разделах о каждом из типов.

| Тип шага | Описание |
| --------- | ----------- |
| [`build`](#build) | Создает образ контейнера с использованием знакомого синтаксиса `docker build`. |
| [`push`](#push) | Выполняет команду `docker push` для отправки только что созданных или перемаркированных образов в реестр контейнеров. Поддерживается Реестр контейнеров Azure, другие закрытые реестры, а также общедоступный реестр Docker Hub. |
| [`cmd`](#cmd) | Запускает контейнер в качестве команды с параметрами, передаваемыми в `[ENTRYPOINT]` контейнера. `cmd`Тип шага поддерживает такие параметры, как `env` , `detach` и другие привычные `docker run` Параметры команды, позволяющие выполнять модульное и функциональное тестирование с параллельным выполнением контейнера. |

## <a name="build"></a>build;

Сборка образа контейнера. Тип шага `build` представляет мультитенантное защищенное средство запуска `docker build` в облаке в качестве примитива первого класса.

### <a name="syntax-build"></a>Синтаксис: build

```yml
version: v1.1.0
steps:
  - [build]: -t [imageName]:[tag] -f [Dockerfile] [context]
    [property]: [value]
```

Тип шага `build` поддерживает параметры, описанные в следующей таблице. Тип шага `build` также поддерживает все параметры сборки из команды [docker build](https://docs.docker.com/engine/reference/commandline/build/), такие как `--build-arg`, для определения переменных во время сборки.

| Параметр | Описание | Необязательный |
| --------- | ----------- | :-------: |
| `-t` &#124; `--image` | Определяет полное значение `image:tag` созданного образа.<br /><br />Так как образы могут использоваться для внутренних проверок задач, например функциональных тестов, не все образы требуют выполнения `push` для отправки в реестр. Но чтобы создать экземпляр образа в пределах выполнения задачи, необходимо указать имя образа для ссылки.<br /><br />В отличие от `az acr build` , выполнение задач записи контроля доступа не обеспечивает поведения принудительной отправки по умолчанию. При использовании службы "Задачи ACR" стандартный сценарий предполагает возможность создания, проверки и последующей отправки образа. Сведения о том, как при необходимости отправлять созданные образы, см. в описании команды [push](#push). | Да |
| `-f` &#124; `--file` | Позволяет указать файл Dockerfile, передаваемый в `docker build`. Если этот параметр не указан, по умолчанию принимается Dockerfile в корне контекста. Чтобы указать Dockerfile, передайте имя файла относительно корня контекста. | Да |
| `context` | Корневой каталог, передаваемый в `docker build`. В качестве корневого каталога каждой задачи задается общий каталог [workingDirectory](#task-step-properties), который включает в себя корень связанного клонированного каталога Git. | Нет |

### <a name="properties-build"></a>Свойства: build

Тип шага `build` поддерживает следующие свойства. Дополнительные сведения об этих свойствах см. в разделе [Свойства шага задачи](#task-step-properties) этой статьи.

| Свойства | Type | Обязательно |
| -------- | ---- | -------- |
| `detach` | bool | Необязательно |
| `disableWorkingDirectoryOverride` | bool | Необязательно |
| `entryPoint` | строка | Необязательно |
| `env` | [строка, строка, ...] | Необязательно |
| `expose` | [строка, строка, ...] | Необязательно |
| `id` | строка | Необязательно |
| `ignoreErrors` | bool | Необязательно |
| `isolation` | строка | Необязательно |
| `keep` | bool | Необязательно |
| `network` | объект | Необязательно |
| `ports` | [строка, строка, ...] | Необязательно |
| `pull` | bool | Необязательно |
| `repeat` | INT | Необязательно |
| `retries` | INT | Необязательно |
| `retryDelay` | int (секунды) | Необязательно |
| `secret` | объект | Необязательно |
| `startDelay` | int (секунды) | Необязательно |
| `timeout` | int (секунды) | Необязательно |
| `volumeMount` | объект | Необязательно |
| `when` | [строка, строка, ...] | Необязательно |
| `workingDirectory` | строка | Необязательно |

### <a name="examples-build"></a>Примеры: build

#### <a name="build-image---context-in-root"></a>Сборка образа — контекст в корне

```azurecli
az acr run -f build-hello-world.yaml https://github.com/AzureCR/acr-tasks-sample.git
```

<!-- SOURCE: https://github.com/Azure-Samples/acr-tasks/blob/master/build-hello-world.yaml -->
[!code-yml[task](~/acr-tasks/build-hello-world.yaml)]

#### <a name="build-image---context-in-subdirectory"></a>Сборка образа — контекст в подкаталоге

```yml
version: v1.1.0
steps:
  - build: -t $Registry/hello-world -f hello-world.dockerfile ./subDirectory
```

## <a name="push"></a>push

Отправляет командой один или несколько из только что созданных или перемаркированных образов в реестр контейнеров. Команда push поддерживается для таких закрытых реестров, как Реестр контейнеров Azure, а также общедоступный реестр Docker Hub.

### <a name="syntax-push"></a>Синтаксис: push

Тип шага `push` поддерживает коллекцию образов. Синтаксис коллекции YAML поддерживает встроенные и вложенные форматы. Отправка одного образа обычно представляется с помощью встроенного синтаксиса:

```yml
version: v1.1.0
steps:
  # Inline YAML collection syntax
  - push: ["$Registry/hello-world:$ID"]
```

Для повышения удобочитаемости используйте вложенный синтаксис при отправке нескольких образов:

```yml
version: v1.1.0
steps:
  # Nested YAML collection syntax
  - push:
    - $Registry/hello-world:$ID
    - $Registry/hello-world:latest
```

### <a name="properties-push"></a>Свойства: push

Тип шага `push` поддерживает следующие свойства. Дополнительные сведения об этих свойствах см. в разделе [Свойства шага задачи](#task-step-properties) этой статьи.

| Свойство. | Type | Обязательно |
| -------- | ---- | -------- |
| `env` | [строка, строка, ...] | Необязательно |
| `id` | строка | Необязательно |
| `ignoreErrors` | bool | Необязательно |
| `startDelay` | int (секунды) | Необязательно |
| `timeout` | int (секунды) | Необязательно |
| `when` | [строка, строка, ...] | Необязательно |

### <a name="examples-push"></a>Примеры: push

#### <a name="push-multiple-images"></a>Отправка нескольких образов командой push

```azurecli
az acr run -f build-push-hello-world.yaml https://github.com/Azure-Samples/acr-tasks.git
```

<!-- SOURCE: https://github.com/Azure-Samples/acr-tasks/blob/master/build-push-hello-world.yaml -->
[!code-yml[task](~/acr-tasks/build-push-hello-world.yaml)]

#### <a name="build-push-and-run"></a>Сборка, отправка и запуск

```azurecli
az acr run -f build-run-hello-world.yaml https://github.com/Azure-Samples/acr-tasks.git
```

<!-- SOURCE: https://github.com/Azure-Samples/acr-tasks/blob/master/build-run-hello-world.yaml -->
[!code-yml[task](~/acr-tasks/build-run-hello-world.yaml)]

## <a name="cmd"></a>cmd

Тип шага `cmd` запускает контейнер.

### <a name="syntax-cmd"></a>Синтаксис: cmd

```yml
version: v1.1.0
steps:
  - [cmd]: [containerImage]:[tag (optional)] [cmdParameters to the image]
```

### <a name="properties-cmd"></a>Свойства: cmd

Тип шага `cmd` поддерживает следующие свойства:

| Свойство. | Type | Обязательно |
| -------- | ---- | -------- |
| `detach` | bool | Необязательно |
| `disableWorkingDirectoryOverride` | bool | Необязательно |
| `entryPoint` | строка | Необязательно |
| `env` | [строка, строка, ...] | Необязательно |
| `expose` | [строка, строка, ...] | Необязательно |
| `id` | строка | Необязательно |
| `ignoreErrors` | bool | Необязательно |
| `isolation` | строка | Необязательно |
| `keep` | bool | Необязательно |
| `network` | объект | Необязательно |
| `ports` | [строка, строка, ...] | Необязательно |
| `pull` | bool | Необязательно |
| `repeat` | INT | Необязательно |
| `retries` | INT | Необязательно |
| `retryDelay` | int (секунды) | Необязательно |
| `secret` | объект | Необязательно |
| `startDelay` | int (секунды) | Необязательно |
| `timeout` | int (секунды) | Необязательно |
| `volumeMount` | объект | Необязательно |
| `when` | [строка, строка, ...] | Необязательно |
| `workingDirectory` | строка | Необязательно |

Сведения об этих свойствах см. в разделе [Свойства шага задачи](#task-step-properties) этой статьи.

### <a name="examples-cmd"></a>Примеры: cmd

#### <a name="run-hello-world-image"></a>Запуск образа hello-world

Эта команда выполняет файл задачи `hello-world.yaml`, который ссылается на образ [hello-world](https://hub.docker.com/_/hello-world/) в Docker Hub.

```azurecli
az acr run -f hello-world.yaml https://github.com/Azure-Samples/acr-tasks.git
```

<!-- SOURCE: https://github.com/Azure-Samples/acr-tasks/blob/master/hello-world.yaml -->
[!code-yml[task](~/acr-tasks/hello-world.yaml)]

#### <a name="run-bash-image-and-echo-hello-world"></a>Запуск образа bash и вывод командой echo hello world

Эта команда выполняет файл задачи `bash-echo.yaml`, который ссылается на образ [bash](https://hub.docker.com/_/bash/) в Docker Hub.

```azurecli
az acr run -f bash-echo.yaml https://github.com/Azure-Samples/acr-tasks.git
```

<!-- SOURCE: https://github.com/Azure-Samples/acr-tasks/blob/master/bash-echo.yaml -->
[!code-yml[task](~/acr-tasks/bash-echo.yaml)]

#### <a name="run-specific-bash-image-tag"></a>Тег для запуска определенного образа bash

Чтобы запустить конкретную версию образа, укажите тег в команде `cmd`.

Эта команда выполняет файл задачи `bash-echo-3.yaml`, который ссылается на образ [bash:3.0](https://hub.docker.com/_/bash/) в Docker Hub.

```azurecli
az acr run -f bash-echo-3.yaml https://github.com/Azure-Samples/acr-tasks.git
```

<!-- SOURCE: https://github.com/Azure-Samples/acr-tasks/blob/master/bash-echo-3.yaml -->
[!code-yml[task](~/acr-tasks/bash-echo-3.yaml)]

#### <a name="run-custom-images"></a>Запуск пользовательских образов

Тип шага `cmd` ссылается на образы с использованием стандартного формата `docker run`. Предполагается, что образы, которые не начинаются с указания реестра, поступают из docker.io. Предыдущий пример может быть также представлен следующим образом:

```yml
version: v1.1.0
steps:
  - cmd: docker.io/bash:3.0 echo hello world
```

Использование стандартного `docker run` соглашения о ссылках на образы `cmd` позволяет запускать образы из любого частного реестра или общедоступного центра DOCKER. Если вы ссылаетесь на образы в том же реестре, в котором выполняется задача ACR, не нужно указывать учетные данные реестра.

* Запустите образ из реестра контейнеров Azure. В следующем примере предполагается, что у вас есть реестр с именем `myregistry` и пользовательский образ `myimage:mytag` .

    ```yml
    version: v1.1.0
    steps:
        - cmd: myregistry.azurecr.io/myimage:mytag
    ```

* Обобщить ссылку реестра с помощью переменной запуска или псевдонима

    Вместо того чтобы жестко программировать имя реестра в `acr-task.yaml` файле, можно сделать его более переносимым с помощью [переменной запуска](#run-variables) или [псевдонима](#aliases). `Run.Registry`Переменная или `$Registry` псевдоним расширяется во время выполнения до имени реестра, в котором выполняется задача.

    Например, чтобы обобщить предыдущую задачу, чтобы она работала в любом реестре контейнеров Azure, сослаться на переменную $Registry в имени образа:

    ```yml
    version: v1.1.0
    steps:
      - cmd: $Registry/myimage:mytag
    ```

#### <a name="access-secret-volumes"></a>Доступ к секретным томам

`volumes`Свойство позволяет указать тома и их секретное содержимое для `build` и `cmd` шагов в задаче. Внутри каждого шага необязательное `volumeMounts` свойство перечисляет тома и соответствующие пути к контейнеру для подключения к контейнеру на этом шаге. Секреты предоставляются в виде файлов на пути подключения каждого тома.

Выполните задачу и подключите два секрета к шагу: один из них хранится в хранилище ключей и один в командной строке:

```azurecli
az acr run -f mounts-secrets.yaml --set-secret mysecret=abcdefg123456 https://github.com/Azure-Samples/acr-tasks.git
```

<!-- SOURCE: https://github.com/Azure-Samples/acr-tasks/blob/master/mounts-secrets.yaml -->
[!code-yml[task](~/acr-tasks/mounts-secrets.yaml)]

## <a name="task-step-properties"></a>Свойства шага задачи

Каждый тип шага поддерживает несколько свойств, подходящих для его типа. В следующей таблице определены все доступные свойства шагов. Не все типы шагов поддерживают все свойства. Чтобы узнать, какие из этих свойств доступны для каждого типа шага, см. разделы справки по типам шагов [cmd](#cmd), [build](#build) и [push](#push).

| Свойство. | Type | Необязательно | Описание | Значение по умолчанию |
| -------- | ---- | -------- | ----------- | ------- |
| `detach` | bool | Да | Определяет, следует ли отсоединить контейнер при запуске. | `false` |
| `disableWorkingDirectoryOverride` | bool | Да | Следует ли отключить `workingDirectory` функцию переопределения. Используйте его в сочетании с `workingDirectory` , чтобы получить полный контроль над рабочим каталогом контейнера. | `false` |
| `entryPoint` | строка | Да | Переопределяет `[ENTRYPOINT]` контейнера шага. | Нет |
| `env` | [строка, строка, ...] | Да | Массив строк в формате `key=value`, определяющий переменные среды для шага. | Нет |
| `expose` | [строка, строка, ...] | Да | Массив портов, доступных из контейнера. |  Нет |
| [`id`](#example-id) | строка | Да | Однозначно определяет шаг в рамках задачи. Другие шаги в задаче могут ссылаться на `id` шага, например для проверки зависимостей с помощью `when`.<br /><br />`id` также является именем запущенного контейнера. Процессы, запущенные в других контейнерах в задаче, могут ссылаться на `id` в качестве имени его узла DNS, или же для доступа к нему с помощью журналов docker [id], например. | `acb_step_%d`, где `%d` — это Отсчитываемый от нуля индекс шага сверху вниз в файле YAML |
| `ignoreErrors` | bool | Да | Помечать ли шаг как успешный независимо от того, произошла ли ошибка во время выполнения контейнера. | `false` |
| `isolation` | строка | Да | Уровень изоляции контейнера. | `default` |
| `keep` | bool | Да | Следует ли сохранить контейнер этого шага после выполнения. | `false` |
| `network` | object | Да | Определяет сеть, в которой выполняется контейнер. | Нет |
| `ports` | [строка, строка, ...] | Да | Массив портов, опубликованных из контейнера на узле. |  Нет |
| `pull` | bool | Да | Указывает, следует ли принудительно запрашивать контейнер перед его выполнением, чтобы предотвратить любое поведение кэширования. | `false` |
| `privileged` | bool | Да | Следует ли запускать контейнер в привилегированном режиме. | `false` |
| `repeat` | INT | Да | Число повторных попыток для повторного выполнения контейнера. | 0 |
| `retries` | INT | Да | Число повторных попыток при сбое выполнения контейнера. Повторная попытка выполняется только в том случае, если код выхода контейнера не равен нулю. | 0 |
| `retryDelay` | int (секунды) | Да | Задержка в секундах между повторными попытками выполнения контейнера. | 0 |
| `secret` | object | Да | Определяет Azure Key Vault секретного или [управляемого удостоверения для ресурсов Azure](container-registry-tasks-authentication-managed-identity.md). | Нет |
| `startDelay` | int (секунды) | Да | Количество секунд, в течение которых откладывается выполнение контейнера. | 0 |
| `timeout` | int (секунды) | Да | Максимальное число секунд, в течение которого может выполняться шаг перед завершением. | 600 |
| [`when`](#example-when) | [строка, строка, ...] | Да | Настраивает зависимость шага от одного или нескольких других шагов в пределах задачи. | Нет |
| `user` | строка | Да | Имя пользователя или UID контейнера | Нет |
| `workingDirectory` | строка | Да | Задает рабочий каталог для шага. По умолчанию служба "Задачи ACR" создает корневой каталог в качестве рабочего каталога. Но если сборка включает несколько шагов, для предыдущих и последующих шагов можно использовать общие артефакты, указав один и тот же рабочий каталог. | `c:\workspace` в Windows или `/workspace` Linux |

### <a name="volumemount"></a>волумемаунт

Объект Волумемаунт имеет следующие свойства.

| Свойство. | Type | Необязательно | Описание | Значение по умолчанию |
| -------- | ---- | -------- | ----------- | ------- | 
| `name` | Строка | Нет | Имя тома для подключения. Должно точно соответствовать имени из `volumes` Свойства. | Нет |
| `mountPath`   | строка | Нет | Абсолютный путь для подключения файлов в контейнере.  | Нет |

### <a name="examples-task-step-properties"></a>Примеры. Свойства шага задачи

#### <a name="example-id"></a>Пример: id

Создайте два образа, используя экземпляры образа функционального теста. Каждый шаг идентифицируется уникальным `id`, на который ссылаются другие шаги задачи в свойстве `when`.

```azurecli
az acr run -f when-parallel-dependent.yaml https://github.com/Azure-Samples/acr-tasks.git
```

<!-- SOURCE: https://github.com/Azure-Samples/acr-tasks/blob/master/when-parallel-dependent.yaml -->
[!code-yml[task](~/acr-tasks/when-parallel-dependent.yaml)]

#### <a name="example-when"></a>Пример: when

Свойство `when` указывает зависимость шага от других шагов в пределах задачи. Оно поддерживает два значения параметров:

* `when: ["-"]` — указывает отсутствие зависимости от других шагов. Шаг с указанием `when: ["-"]` начнет выполняться немедленно и обеспечивает параллельное выполнение шагов.
* `when: ["id1", "id2"]` — указывает, что шаг зависит от шагов с `id` id1 и `id` id2. Этот шаг не будет выполнен до тех пор, пока не будут завершены оба шага id1 и id2.

Если свойство `when` не указано в шаге, этот шаг зависит от завершения предыдущего шага в файле `acr-task.yaml`.

Последовательное выполнение шагов без свойства `when`:

```azurecli
az acr run -f when-sequential-default.yaml https://github.com/Azure-Samples/acr-tasks.git
```

<!-- SOURCE: https://github.com/Azure-Samples/acr-tasks/blob/master/when-sequential-default.yaml -->
[!code-yml[task](~/acr-tasks/when-sequential-default.yaml)]

Последовательное выполнение шагов со свойством `when`:

```azurecli
az acr run -f when-sequential-id.yaml https://github.com/Azure-Samples/acr-tasks.git
```

<!-- SOURCE: https://github.com/Azure-Samples/acr-tasks/blob/master/when-sequential-id.yaml -->
[!code-yml[task](~/acr-tasks/when-sequential-id.yaml)]

Сборка параллельных образов:

```azurecli
az acr run -f when-parallel.yaml https://github.com/Azure-Samples/acr-tasks.git
```

<!-- SOURCE: https://github.com/Azure-Samples/acr-tasks/blob/master/when-parallel.yaml -->
[!code-yml[task](~/acr-tasks/when-parallel.yaml)]

Параллельное создание образов и тестирование зависимостей:

```azurecli
az acr run -f when-parallel-dependent.yaml https://github.com/Azure-Samples/acr-tasks.git
```

<!-- SOURCE: https://github.com/Azure-Samples/acr-tasks/blob/master/when-parallel-dependent.yaml -->
[!code-yml[task](~/acr-tasks/when-parallel-dependent.yaml)]

## <a name="run-variables"></a>Переменные run

Служба "Задачи ACR" содержит набор переменных по умолчанию, доступных для шагов задачи при их выполнении. К этим переменным можно получить доступ с использованием формата `{{.Run.VariableName}}`, где `VariableName` — одна из следующих переменных:

* `Run.ID`
* `Run.SharedVolume`
* `Run.Registry`
* `Run.RegistryName`
* `Run.Date`
* `Run.OS`
* `Run.Architecture`
* `Run.Commit`
* `Run.Branch`
* `Run.TaskName`

Имена переменных обычно говорят сами за себя. Дополнительные сведения см. в описании часто используемых переменных. Начиная с версии YAML `v1.1.0` можно использовать сокращенный, предопределенный [псевдоним задачи](#aliases) вместо большинства переменных запуска. Например, вместо `{{.Run.Registry}}` Используйте `$Registry` псевдоним.

### <a name="runid"></a>Run.ID

Каждое выполнение задач, выполняемых с помощью, `az acr run` или триггеров, созданных с помощью `az acr task create` , имеет уникальный идентификатор. Идентификатор представляет выполняющуюся в текущий момент команду Run.

Обычно используется для уникальной маркировки образа:

```yml
version: v1.1.0
steps:
    - build: -t $Registry/hello-world:$ID .
```

### <a name="runsharedvolume"></a>Run. Шаредволуме

Уникальный идентификатор общего тома, доступного для всех шагов задачи. Том подключен к `c:\workspace` в Windows или `/workspace` Linux. 

### <a name="runregistry"></a>Run.Registry

Полное имя сервера реестра. Обычно используется для универсальной ссылки на реестр, в котором выполняется задача.

```yml
version: v1.1.0
steps:
  - build: -t $Registry/hello-world:$ID .
```

### <a name="runregistryname"></a>Run. Регистринаме

Имя реестра контейнеров. Обычно используется в шагах задач, не требующих полного имени сервера, например для `cmd` действий, выполняемых Azure CLI команд в реестре.

```yml
version 1.1.0
steps:
# List repositories in registry
- cmd: az login --identity
- cmd: az acr repository list --name $RegistryName
```

### <a name="rundate"></a>Run.Date

Текущее время в формате UTC, когда начался запуск.

### <a name="runcommit"></a>Выполнить. Commit

Для задачи, активируемой фиксацией в репозитории GitHub, идентификатор фиксации.

### <a name="runbranch"></a>Запустить. Branch

Для задачи, активируемой фиксацией в репозитории GitHub, это имя ветви.

## <a name="aliases"></a>Aliases

Начиная с `v1.1.0` , задачи записи контроля доступа поддерживают псевдонимы, доступные для шагов задач при их выполнении. Псевдонимы похожи на понятия псевдонимов (ярлыки команд), поддерживаемые Bash и другие командные оболочки. 

С помощью псевдонима можно запустить любую команду или группу команд (включая параметры и имена файлов), введя одно слово.

Задачи записи контроля доступа поддерживают несколько предопределенных псевдонимов, а также пользовательские псевдонимы, которые вы создаете.

### <a name="predefined-aliases"></a>Предопределенные псевдонимы

Следующие псевдонимы задач можно использовать вместо [переменных запуска](#run-variables):

| Псевдоним | Выполнить переменную |
| ----- | ------------ |
| `ID` | `Run.ID` |
| `SharedVolume` | `Run.SharedVolume` |
| `Registry` | `Run.Registry` |
| `RegistryName` | `Run.RegistryName` |
| `Date` | `Run.Date` |
| `OS` | `Run.OS` |
| `Architecture` | `Run.Architecture` |
| `Commit` | `Run.Commit` |
| `Branch` | `Run.Branch` |

В шагах задачи перед псевдонимом с помощью `$` директивы, как показано в следующем примере:

```yml
version: v1.1.0
steps:
  - build: -t $Registry/hello-world:$ID -f hello-world.dockerfile .
```

### <a name="image-aliases"></a>Псевдонимы изображений

Каждый из следующих псевдонимов указывает на стабильный образ в реестре контейнеров Microsoft (мкр). Вы можете ссылаться на каждую из них в `cmd` разделе файла задачи без использования директивы.

| Псевдоним | Образ — |
| ----- | ----- |
| `acr` | `mcr.microsoft.com/acr/acr-cli:0.1` |
| `az` | `mcr.microsoft.com/acr/azure-cli:a80af84` |
| `bash` | `mcr.microsoft.com/acr/bash:a80af84` |
| `curl` | `mcr.microsoft.com/acr/curl:a80af84` |

В следующем примере задача использует несколько псевдонимов для [очистки](container-registry-auto-purge.md) тегов изображений старше 7 дней в репозитории `samples/hello-world` в реестре выполнения:

```yml
version: v1.1.0
steps:
  - cmd: acr tag list --registry $RegistryName --repository samples/hello-world
  - cmd: acr purge --registry $RegistryName --filter samples/hello-world:.* --ago 7d
```

### <a name="custom-alias"></a>Пользовательский псевдоним

Определите пользовательский псевдоним в файле YAML и используйте его, как показано в следующем примере. Псевдоним может содержать только буквенно-цифровые символы. Директивой по умолчанию для расширения псевдонима является `$` символ.

```yml
version: v1.1.0
alias:
  values:
    repo: myrepo
steps:
  - build: -t $Registry/$repo/hello-world:$ID -f Dockerfile .
```

Можно создать ссылку на удаленный или локальный файл YAML для определений пользовательских псевдонимов. В следующем примере выполняется ссылка на файл YAML в хранилище BLOB-объектов Azure:

```yml
version: v1.1.0
alias:
  src:  # link to local or remote custom alias files
    - 'https://link/to/blob/remoteAliases.yml?readSasToken'
[...]
```

## <a name="next-steps"></a>Дальнейшие действия

Обзор многошаговых задач см. в статье [Run multi-step build, test, and patch tasks in ACR Tasks](container-registry-tasks-multi-step.md) (Выполнение многошаговых задач сборки, тестирования и исправления в решении "Задачи ACR").

Сведения об одношаговых сборках см. в статье [Автоматизация установки исправлений ОС и платформы с помощью службы "Задачи ACR"](container-registry-tasks-overview.md).



<!-- IMAGES -->

<!-- LINKS - External -->
[acr-tasks]: https://github.com/Azure-Samples/acr-tasks

<!-- LINKS - Internal -->
[az-acr-run]: /cli/azure/acr#az-acr-run
[az-acr-task-create]: /cli/azure/acr/task#az-acr-task-create
[az-configure]: /cli/azure/reference-index#az-configure
