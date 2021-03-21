---
title: Создание группы PostgreSQL Scale Server с помощью средств Kubernetes
description: Создание группы PostgreSQL Scale Server с помощью средств Kubernetes
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: fd1b74d33793c06e586a92cc8b2e8d2d36f4827a
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102519968"
---
# <a name="create-a-postgresql-hyperscale-server-group-using-kubernetes-tools"></a>Создание группы PostgreSQL Scale Server с помощью средств Kubernetes

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="prerequisites"></a>Предварительные условия

Вы уже создали [контроллер данных Arc Azure](./create-data-controller.md).

Чтобы создать PostgreSQL группу серверов с помощью средств Kubernetes, необходимо установить средства Kubernetes.  Примеры в этой статье будут использовать `kubectl` , но аналогичные подходы можно использовать с другими средствами Kubernetes, такими как панель мониторинга Kubernetes, `oc` или `helm` Если вы знакомы с этими инструментами и Kubernetes YAML/JSON.

[Установка средства kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

## <a name="overview"></a>Обзор

Чтобы создать группу серверов PostgreSQL, необходимо создать секрет Kubernetes для безопасного хранения имени входа администратора postgres и пароля, а также пользовательский ресурс группы PostgreSQL Scale Server, основанный на определениях настраиваемых ресурсов PostgreSQL-12 или PostgreSQL-11.

## <a name="create-a-yaml-file"></a>Создание файла YAML

Вы можете использовать файл [шаблона YAML](https://raw.githubusercontent.com/microsoft/azure_arc/main/arc_data_services/deploy/yaml/postgresql.yaml) в качестве отправной точки для создания собственного пользовательского файла YAML PostgreSQL.  Скачайте этот файл на локальный компьютер и откройте его в текстовом редакторе.  Полезно использовать текстовый редактор, например [VS Code](https://code.visualstudio.com/download) , поддерживающий выделение синтаксиса и linting для файлов YAML.

Это пример файла YAML:

```yaml
apiVersion: v1
data:
  password: <your base64 encoded password>
kind: Secret
metadata:
  name: pg1-login-secret
type: Opaque
---
apiVersion: arcdata.microsoft.com/v1alpha1
kind: postgresql-12
metadata:
  generation: 1
  name: pg1
spec:
  engine:
    extensions:
    - name: citus
  scale:
    shards: 3
  scheduling:
    default:
      resources:
        limits:
          cpu: "4"
          memory: 4Gi
        requests:
          cpu: "1"
          memory: 2Gi
  service:
    type: LoadBalancer
  storage:
    backups:
      className: default
      size: 5Gi
    data:
      className: default
      size: 5Gi
    logs:
      className: default
      size: 1Gi
```

### <a name="customizing-the-login-and-password"></a>Настройка имени входа и пароля.
Секрет Kubernetes хранится в виде строки в кодировке Base64 — одна для имени пользователя, а другая — для пароля.  Вам потребуется выполнить кодирование имени входа и пароля администратора в Base64 и поместить их в заполнитель расположение по адресу `data.password` и `data.username` .  Не включайте `<` символы и, `>` указанные в шаблоне.

Вы можете использовать онлайн-инструмент для кодирования требуемого имени пользователя и пароля в Base64 или использовать встроенные средства CLI в зависимости от платформы.

PowerShell

```console
[Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes('<your string to encode here>'))

#Example
#[Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes('example'))

```

Linux/macOS

```console
echo -n '<your string to encode here>' | base64

#Example
# echo -n 'example' | base64
```

### <a name="customizing-the-name"></a>Настройка имени

Шаблон имеет значение "группы PG1" для атрибута Name.  Его можно изменить, но должны быть символами, которые следуют стандартам DNS-имен.  Необходимо также изменить имя секрета на Match.  Например, если изменить имя группы серверов PostgreSQL Scale на группы pg2, необходимо изменить имя секрета с "группы PG1-login-Secret" на "группы pg2-login-Secret".

### <a name="customizing-the-engine-version"></a>Настройка версии подсистемы

Вы можете изменить версию подсистемы так, чтобы она была PostgreSQL-11 или PostgreSQL-12, изменив `kind` атрибут.

### <a name="customizing-the-resource-requirements"></a>Настройка требований к ресурсам

При необходимости можно изменить требования к ресурсам для ОЗУ и ядер и запросов.  

> [!NOTE]
> Дополнительные сведения см. в статье [Управление ресурсами Kubernetes](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#resource-units-in-kubernetes).

Требования к ограничениям ресурсов и запросам:
- В целях выставления счетов **требуется** значение ограничения ядер.
- Остальные запросы и ограничения ресурсов являются необязательными.
- Ограничение количества ядер и запрос должны быть положительным целым числом, если оно указано.
- При указании запроса на ядро требуется минимум 1 ядро, если оно указано.
- Формат значения памяти соответствует нотации Kubernetes.  

### <a name="customizing-service-type"></a>Настройка типа службы

При необходимости тип службы можно изменить на Нодепорт.  Будет назначен случайный номер порта.

### <a name="customizing-storage"></a>Настройка хранилища

Классы хранения можно настроить для хранения в соответствии с вашей средой.  Если вы не уверены, какие классы хранилища доступны, можно выполнить команду, `kubectl get storageclass` чтобы просмотреть их.  Шаблон имеет значение по умолчанию "default".  Это означает, что существует класс хранения _с именем_ Default, не имеющий класса _хранения, используемого_ по умолчанию.  При необходимости можно также изменить размер хранилища.  Дополнительные сведения о [конфигурации хранилища](./storage-configuration.md)см. в этой статье.

## <a name="creating-the-postgresql-hyperscale-server-group"></a>Создание группы серверов PostgreSQL Scale

Теперь, когда вы настроили файл YAML PostgreSQL для группы серверов, можно создать группу серверов PostgreSQL Scale, выполнив следующую команду:

```console
kubectl create -n <your target namespace> -f <path to your yaml file>

#Example
#kubectl create -n arc -f C:\arc-data-services\postgres.yaml
```


## <a name="monitoring-the-creation-status"></a>Наблюдение за состоянием создания

Создание группы серверов PostgreSQL Scale займет несколько минут. Вы можете отслеживать ход выполнения в другом окне терминала с помощью следующих команд:

> [!NOTE]
>  В приведенных ниже примерах команд предполагается, что вы создали PostgreSQL группу серверов с именем "группы PG1" и пространством имен Kubernetes с именем "Arc".  Если вы использовали другое имя пространства имен или PostgreSQL, можно заменить "Arc" и "группы PG1" своими именами.

```console
kubectl get postgresql-12/pg1 --namespace arc
```

```console
kubectl get pods --namespace arc
```

Вы также можете проверить состояние создания любого конкретного модуля, выполнив команду, как показано ниже.  Это особенно полезно для устранения любых проблем.

```console
kubectl describe po/<pod name> --namespace arc

#Example:
#kubectl describe po/pg1-0 --namespace arc
```

## <a name="troubleshooting-creation-problems"></a>Устранение неполадок при создании

Если у вас возникли роняли с созданием, ознакомьтесь с [руководством по устранению неполадок](troubleshoot-guide.md).