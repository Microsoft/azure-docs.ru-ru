---
title: Потоковая передача журналов приложения в Azure Spring Cloud в реальном времени
description: Как мгновенно просматривать журналы приложений с помощью потоковой передачи журналов
author: MikeDodaro
ms.author: barbkess
ms.service: spring-cloud
ms.topic: how-to
ms.date: 01/14/2019
ms.custom: devx-track-java, devx-track-azurecli
ms.openlocfilehash: 1eeb291c7a058efd8905e95ebf1ea14fed046691
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104878290"
---
# <a name="stream-azure-spring-cloud-app-logs-in-real-time"></a>Потоковая передача журналов приложения в Azure Spring Cloud в реальном времени

**Эта статья применима к:** ✔️ Java ✔️ C#

Azure Веснного облака позволяет выполнять потоковую передачу журналов в Azure CLI для получения журналов консоли приложений в режиме реального времени для устранения неполадок. Вы также можете [анализировать журналы и метрики с помощью параметров диагностики](./diagnostic-services.md).

## <a name="prerequisites"></a>Предварительные требования

* Установите [расширение Azure CLI](/cli/azure/install-azure-cli) для пружинного облака, минимальной версии 0.2.0.
* Экземпляр **Azure веснного облака** с выполняющимся приложением, например [пружинное приложение Cloud](./spring-cloud-quickstart.md).

> [!NOTE]
>  Расширение интерфейса командной строки ASC обновляется с версии 0.2.0 до 0.2.1. Это изменение влияет на синтаксис команды для потоковой передачи журнала: `az spring-cloud app log tail` , который заменяется на: `az spring-cloud app logs` . Команда: `az spring-cloud app log tail` будет нерекомендуемой в будущих выпусках. Если вы используете версию 0.2.0, можно выполнить обновление до 0.2.1. Сначала удалите старую версию с помощью команды: `az extension remove -n spring-cloud` .  Затем установите 0.2.1 с помощью команды: `az extension add -n spring-cloud` .

## <a name="use-cli-to-tail-logs"></a>Использование интерфейса командной строки для заключительного фрагмента журналов

Чтобы избежать повторного указания имени группы ресурсов и экземпляра службы, задайте имя группы ресурсов по умолчанию и имя кластера.
```azurecli
az configure --defaults group=<service group name>
az configure --defaults spring-cloud=<service instance name>
```
В следующих примерах группа ресурсов и имя службы будут пропущены в командах.

### <a name="tail-log-for-app-with-single-instance"></a>Заключительный фрагмент журнала для приложения с одним экземпляром
Если приложение с именем auth-Service имеет только один экземпляр, журнал экземпляра приложения можно просмотреть с помощью следующей команды:
```azurecli
az spring-cloud app logs -n auth-service
```
Будут возвращены журналы:
```output
...
2020-01-15 01:54:40.481  INFO [auth-service,,,] 1 --- [main] o.apache.catalina.core.StandardService  : Starting service [Tomcat]
2020-01-15 01:54:40.482  INFO [auth-service,,,] 1 --- [main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.22]
2020-01-15 01:54:40.760  INFO [auth-service,,,] 1 --- [main] o.a.c.c.C.[Tomcat].[localhost].[/uaa]  : Initializing Spring embedded WebApplicationContext
2020-01-15 01:54:40.760  INFO [auth-service,,,] 1 --- [main] o.s.web.context.ContextLoader  : Root WebApplicationContext: initialization completed in 7203 ms

...
```

### <a name="tail-log-for-app-with-multiple-instances"></a>Заключительный фрагмент журнала для приложения с несколькими экземплярами
Если для приложения существует несколько экземпляров `auth-service` , журнал экземпляра можно просмотреть с помощью `-i/--instance` параметра. 

Сначала можно получить имена экземпляров приложения с помощью следующей команды.

```azurecli
az spring-cloud app show -n auth-service --query properties.activeDeployment.properties.instances -o table
```
С результатами:

```output
Name                                         Status    DiscoveryStatus
-------------------------------------------  --------  -----------------
auth-service-default-12-75cc4577fc-pw7hb  Running   UP
auth-service-default-12-75cc4577fc-8nt4m  Running   UP
auth-service-default-12-75cc4577fc-n25mh  Running   UP
``` 
Затем можно выполнить потоковую передачу журналов экземпляра приложения с `-i/--instance` параметром:

```azurecli
az spring-cloud app logs -n auth-service -i auth-service-default-12-75cc4577fc-pw7hb
```

Кроме того, можно получить сведения об экземплярах приложения из портал Azure.  Выбрав **приложения** в левой области навигации в облачной службе Azure "Весна", выберите **экземпляры приложения**.

### <a name="continuously-stream-new-logs"></a>Непрерывный поток новых журналов
По умолчанию `az spring-cloud ap log tail` выводит только существующие журналы, переданные в консоль приложения, а затем завершает работу. Если вы хотите создать потоковую передачу новых журналов, добавьте-f (--следовать):  

```azurecli
az spring-cloud app logs -n auth-service -f
``` 
Чтобы проверить все поддерживаемые параметры ведения журнала, выполните следующие действия.
```azurecli
az spring-cloud app logs -h 
```

## <a name="next-steps"></a>Дальнейшие действия
* [Краткое руководство. Мониторинг приложений Azure Spring Cloud с помощью журналов, метрик и трассировки](spring-cloud-quickstart-logs-metrics-tracing.md)
* [Анализ журналов и метрик с помощью параметров диагностики](./diagnostic-services.md)

