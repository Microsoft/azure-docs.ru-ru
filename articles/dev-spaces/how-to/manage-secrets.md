---
title: Управление секретами при работе со службой Azure Dev Space
services: azure-dev-spaces
ms.date: 12/03/2019
ms.topic: conceptual
description: Узнайте, как использовать секреты Kubernetes во время выполнения или сборки при разработке приложений с помощью Azure Dev Spaces
keywords: Docker, Kubernetes, Azure, AKS, Azure Container Service, containers
ms.custom: devx-track-js
ms.openlocfilehash: 8791480f420dfd76d5291ce82e8ebf7412a41326
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91972974"
---
# <a name="how-to-manage-secrets-when-working-with-an-azure-dev-space"></a>Управление секретами при работе со службой Azure Dev Space

[!INCLUDE [Azure Dev Spaces deprecation](../../../includes/dev-spaces-deprecation.md)]

Вашим службам могут потребоваться определенные пароли, строки подключения и другие секреты для баз данных или других защищенных служб Azure. Задавая значения этих секретов в файле конфигурации, их можно сделать доступными в коде в качестве переменных среды.  Эти файлы конфигурации должны быть обработаны с осторожностью, чтобы избежать нарушения безопасности секретов.

## <a name="storing-and-using-runtime-secrets"></a>Хранение и использование секретов времени выполнения

Azure Dev Spaces предоставляет два рекомендуемых, упрощенных варианта хранения секретов в диаграммах Helm, созданных с помощью средств клиента Azure Dev Spaces: в `values.dev.yaml` файле и встроенного непосредственно в `azds.yaml` . Не рекомендуется хранить секреты в `values.yaml` .

> [!NOTE]
> В следующих подходах показано, как хранить и использовать секреты для диаграмм Helm, создаваемых клиентскими средствами. Если вы создаете собственную диаграмму Helm, вы можете использовать диаграмму Helm непосредственно для управления секретами и их хранения.

### <a name="using-valuesdevyaml"></a>Использование Values. dev. YAML

В проекте, который уже подготовлен с Azure Dev Spaces, создайте `values.dev.yaml` файл в той же папке, что `azds.yaml` и для определения секретных ключей и значений. Пример:

```yaml
secrets:
  redis:
    port: "6380"
    host: "contosodevredis.redis.cache.windows.net"
    key: "secretkeyhere"
```

Проверьте `azds.yaml` ссылки на файл `values.dev.yaml` как необязательные с помощью `?` . Пример:

```yaml
install:
  values:
  - values.dev.yaml?
  - secrets.dev.yaml?
```

Если у вас есть дополнительные секретные файлы, их также можно добавить сюда.

Обновите или убедитесь, что служба ссылается на секреты в качестве переменных среды. Пример:

```javascript
var redisPort = process.env.REDIS_PORT
var host = process.env.REDIS_HOST
var theKey = process.env.REDIS_KEY
```
    
Запустите обновленные службы с помощью `azds up` .

```console
azds up
```
 
Используйте `kubectl` , чтобы проверить, созданы ли секреты.

```console
kubectl get secret --namespace default -o yaml 
```

> [!IMPORTANT]
> Не рекомендуется хранить секреты в системе управления версиями. При использовании git добавьте `values.dev.yaml` в файл, `.gitignore` чтобы избежать фиксации секретов в системе управления версиями.

### <a name="using-azdsyaml"></a>Использование аздс. YAML

В проекте, который уже подготовлен с Azure Dev Spaces, добавьте секретные ключи и значения с помощью синтаксиса *$PLACEHOLDER* в разделе *Configurations. Разработка. install. Set* в `azds.yaml` . Пример:

```yaml
configurations:
  develop:
    ...
    install:
      set:
        secrets:
          redis:
            port: "$REDIS_PORT"
            host: "$REDIS_HOST"
            key: "$REDIS_KEY"
```

> [!NOTE]
> Значения секрета можно вводить напрямую без использования синтаксиса *$PLACEHOLDER* в `azds.yaml` . Однако этот подход не рекомендуется, так как `azds.yaml` хранится в системе управления версиями.
     
Создайте `.env` файл в той же папке, что `azds.yaml` и для определения значений *$PLACEHOLDER* . Пример:

```
REDIS_PORT=3333
REDIS_HOST=myredishost
REDIS_KEY=myrediskey
```

> [!IMPORTANT]
> Не рекомендуется хранить секреты в системе управления версиями. При использовании git добавьте `.env` в файл, `.gitignore` чтобы избежать фиксации секретов в системе управления версиями.

Обновите или убедитесь, что служба ссылается на секреты в качестве переменных среды. Пример:

```javascript
var redisPort = process.env.REDIS_PORT
var host = process.env.REDIS_HOST
var theKey = process.env.REDIS_KEY
```
    
Запустите обновленные службы с помощью `azds up` .

```console
azds up
```
 
Используйте `kubectl` , чтобы проверить, созданы ли секреты.

```console
kubectl get secret --namespace default -o yaml 
```

## <a name="using-secrets-as-build-arguments"></a>Использование секретов в качестве аргументов сборки

В предыдущем разделе было показано, как хранить и использовать секреты для использования во время выполнения контейнера. Можно также использовать любой секрет во время сборки контейнера, например пароль для частного NuGet, с помощью `azds.yaml` .

В `azds.yaml` Задайте секреты времени сборки в *конфигурациях. Разработка. сборка. args* с помощью `<variable name>: ${secret.<secret name>.<secret key>}` синтаксиса. Пример:

```yaml
configurations:
  develop:
    build:
      dockerfile: Dockerfile.develop
      useGitIgnore: true
      args:
        BUILD_CONFIGURATION: ${BUILD_CONFIGURATION:-Debug}
        MYTOKEN: ${secret.mynugetsecret.pattoken}
```

В приведенном выше примере *минужетсекрет* — это существующий секрет, а *паттокен* — существующий ключ.

>[!NOTE]
> Имена секретов и ключи могут содержать `.` символ. Используйте `\` для экранирования `.` при передаче секретов в качестве аргументов сборки. Например, для передачи секрета с именем *foo.Bar* с ключом *токена*: `MYTOKEN: ${secret.foo\.bar.token}` . Кроме того, секреты можно оценивать с помощью префиксного и постфиксного текста. Например, `MYURL: eus-${secret.foo\.bar.token}-version1`. Кроме того, секреты, доступные в родительских и бабушкх пространствах, могут передаваться как аргументы сборки.

В Dockerfile используйте директиву *arg* , чтобы использовать секрет, а затем используйте эту же переменную позже в Dockerfile. Пример:

```dockerfile
...
ARG MYTOKEN
...
ARG NUGET_EXTERNAL_FEED_ENDPOINTS="{'endpointCredentials': [{'endpoint':'PRIVATE_NUGET_ENDPOINT', 'password':'${MYTOKEN}'}]}"
...
```

Обновите службы, выполняемые в кластере, чтобы активировать эти изменения. В командной строке запустите команду:

```
azds up
```

## <a name="next-steps"></a>Дальнейшие действия

С помощью этих методов можно безопасно подключиться к базе данных, кэшу Azure для Redis или получить защищенный доступ к службам Azure.
 
