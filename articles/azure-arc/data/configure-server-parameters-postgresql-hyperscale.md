---
title: Настройка параметров сервера postgres Engine для группы PostgreSQL Scale Server в службе "Дуга" в Azure
titleSuffix: Azure Arc enabled data services
description: Настройка параметров сервера postgres Engine для группы PostgreSQL Scale Server в службе "Дуга" в Azure
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: cdbddfc84b3f71576cfd0299f2babec859b4ef1f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92311059"
---
# <a name="set-the-database-engine-settings-for-azure-arc-enabled-postgresql-hyperscale"></a>Настройка параметров ядра СУБД для PostgreSQL с поддержкой Azure Arc

В этом документе описаны шаги задания параметров ядра СУБД группы серверов PostgreSQL с гипермасштабированием настраиваемых значений (не по умолчанию). Дополнительные сведения о том, какие параметры ядра СУБД можно задать и их значения по умолчанию, см. в документации по PostgreSQL [здесь](https://www.postgresql.org/docs/current/runtime-config.html).

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

> [!NOTE]
> Предварительный просмотр не поддерживает задание следующих параметров: 
>
> - `archive_command`
> - `archive_timeout`
> - `log_directory`
> - `log_file_mode`
> - `log_filename`
> - `restore_command`
> - `shared_preload_libraries`
> - `synchronous_commit`
> - `ssl`
> - `wal_level`

## <a name="syntax"></a>Синтаксис

Общий формат команды для настройки параметров ядра СУБД:

```console
azdata arc postgres server edit -n <server group name>, [{--engine-settings, -e}] [{--replace-engine-settings, --re}] {'<parameter name>=<parameter value>, ...'}
```

## <a name="show-current-custom-values"></a>Отображение текущих пользовательских значений

### <a name="with-azure-data-cli-azdata-command"></a>С помощью [!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)] команды

```console
azdata arc postgres server show -n <server group name>
```

Пример:

```console
azdata arc postgres server show -n postgres01
```

Эта команда возвращает спецификацию группы серверов, в которой отображаются заданные параметры. Если раздел енгине\сеттингс отсутствует, это означает, что все параметры выполняются по значению по умолчанию:

```console
 "
...
engine": {
      "settings": {
        "default": {
          "autovacuum_vacuum_threshold": "65"
        }
      }
    },
...
```

### <a name="with-kubectl-command"></a>С помощью команды kubectl

Выполните следующие действия.

1. Получение типа определения пользовательского ресурса для группы серверов

   Выполните команду:

   ```console
   azdata arc postgres server show -n <server group name>
   ```

   Пример:

   ```console
   azdata arc postgres server show -n postgres01
   ```

   Эта команда возвращает спецификацию группы серверов, в которой отображаются заданные параметры. Если раздел енгине\сеттингс отсутствует, это означает, что все параметры выполняются по значению по умолчанию:

   ```output
   > {
     >"apiVersion": "arcdata.microsoft.com/v1alpha1",
     >"**kind**": "**postgresql-12**",
     >"metadata": {
       >"creationTimestamp": "2020-08-25T14:32:23Z",
       >"generation": 1,
       >"name": "postgres01",
       >"namespace": "arc",  
   ```

   В результатах вывода найдите поле `kind` и отложите значение, например: `postgresql-12` .

2. Опишите настраиваемый ресурс Kubernetes, соответствующий вашей группе серверов 

   Общий формат команды:

   ```console
   kubectl describe <kind of the custom resource> <server group name> -n <namespace name>
   ```

   Пример:

   ```console
   kubectl describe postgresql-12 postgres01
   ```

   Если для параметров подсистемы заданы пользовательские значения, они возвращают их. Пример:

   ```output
   Engine:
   ...
    Settings:
      Default:
        autovacuum_vacuum_threshold:  65
   ```

   Если вы не настроили пользовательские значения для каких бы то ни было параметров ядра, раздел параметров ядра в `resultset` будет пустым следующим образом:

   ```output
   Engine:
   ...
       Settings:
         Default:
   ```

## <a name="set-custom-values-for-engine-settings"></a>Задание пользовательских значений для параметров подсистемы

Приведенные ниже команды задают параметры узла координатора, а рабочие узлы PostgreSQL масштабируются одними и теми же значениями. Пока не удается задать параметры для каждой роли в группе серверов. То есть пока невозможно настроить заданный параметр для конкретного узла координатора и другого значения для рабочих узлов.

### <a name="set-a-single-parameter"></a>Задание одного параметра

```console
azdata arc server edit -n <server group name> -e <parameter name>=<parameter value>
```

Пример:

```console
azdata arc postgres server edit -n postgres01 -e shared_buffers=8MB
```

### <a name="set-multiple-parameters-with-a-single-command"></a>Задание нескольких параметров с помощью одной команды

```console
azdata arc postgres server edit -n <server group name> -e '<parameter name>=<parameter value>, <parameter name>=<parameter value>,...'
```

Пример:

```console
azdata arc postgres server edit -n postgres01 -e 'shared_buffers=8MB, max_connections=50'
```

### <a name="reset-a-parameter-to-its-default-value"></a>Сброс параметра к значению по умолчанию

Чтобы сбросить параметр до значения по умолчанию, задайте его без указания значения. 

Пример:

```console
azdata arc postgres server edit -n postgres01 -e shared_buffers=
```

### <a name="reset-all-parameters-to-their-default-values"></a>Сброс всех параметров до значений по умолчанию

```console
azdata arc postgres server edit -n <server group name> -e '' -re
```

Пример:

```console
azdata arc postgres server edit -n postgres01 -e '' -re
```

## <a name="special-considerations"></a>Специальные рекомендации

### <a name="set-a-parameter-which-value-contains-a-comma-space-or-special-character"></a>Задайте параметр, значение которого содержит запятую, пробел или специальный символ.

```console
azdata arc postgres server edit -n <server group name> -e '<parameter name>="<parameter value>"'
```

Пример:

```console
azdata arc postgres server edit -n postgres01 -e 'custom_variable_classes = "plpgsql,plperl"'
```

### <a name="pass-an-environment-variable-in-a-parameter-value"></a>Передача переменной среды в значение параметра

Переменная среды должна быть заключена в "" ", поэтому она не разрешается до ее установки.

Пример: 

```console
azdata arc postgres server edit -n postgres01 -e 'search_path = "$user"'
```

## <a name="next-steps"></a>Дальнейшие действия
- Дополнительные сведения о [масштабировании (Добавление рабочих узлов)](scale-out-postgresql-hyperscale-server-group.md) в группу серверов
- Дополнительные сведения о [масштабировании (увеличение или уменьшение объема памяти и виртуальных ядер)](scale-up-down-postgresql-hyperscale-server-group-using-cli.md) для группы серверов
