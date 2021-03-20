---
title: Отладка приложения в Eclipse
description: Повысьте надежность и производительность служб, разрабатывая их в Eclipse и локальном кластере разработки.
ms.topic: conceptual
ms.date: 11/02/2017
ms.custom: devx-track-java
ms.openlocfilehash: d321e0c10b66a15e6cb309cefe711602fa12957c
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91534116"
---
# <a name="debug-your-java-service-fabric-application-using-eclipse"></a>Отладка приложения Java Service Fabric с помощью Eclipse
> [!div class="op_single_selector"]
> * [Visual Studio и CSharp](service-fabric-debugging-your-application.md) 
> * [Eclipse и Java](service-fabric-debugging-your-application-java.md)
> 

1. Запустите кластер локальной разработки, выполнив действия, описанные в статье [Настройка среды разработки Service Fabric](service-fabric-get-started-linux.md).

2. Обновите файл entryPoint.sh службы, которую необходимо отладить, таким образом, чтобы он запускал процесс Java с параметрами удаленной отладки. Этот файл можно найти в следующем расположении: `ApplicationName\ServiceNamePkg\Code\entrypoint.sh`. Для отладки в этом примере задается порт 8001.

    ```sh
    java -Xdebug -Xrunjdwp:transport=dt_socket,address=8001,server=y,suspend=n -Djava.library.path=$LD_LIBRARY_PATH -jar myapp.jar
    ```
3. Обновите манифест приложения, задав для отлаживаемой службы число экземпляров или реплик, равное 1. Эта настройка позволяет избежать конфликтов для порта, используемого для отладки. Например, для служб без отслеживания состояния, задайте `InstanceCount="1"`, а для служб с отслеживанием состояния задайте целевые и минимальные размеры набора реплик, равные 1, следующим образом: `TargetReplicaSetSize="1" MinReplicaSetSize="1"`.

4. Разверните приложения.

5. В интегрированной среде разработки Eclipse выберите **Запустить-> Debug Configurations (Конфигурации отладки)-> Remote Java Application and input connection properties (Свойства удаленного приложения Java и входных подключений)** и установите свойства следующим образом:

   ```
   Host: ipaddress
   Port: 8001
   ```
6.  Установите точки останова на требуемых точках и запустите отладку приложения.

При сбое приложения можно также включить функцию coredumps. Выполните `ulimit -c` в оболочке. Если возвращается 0, то функция coredumps не включена. Чтобы функция coredumps работала без ограничений, выполните следующую команду: `ulimit -c unlimited`. Состояние функции можно проверить с помощью команды `ulimit -a`.  Если требуется обновить путь создания coredump, выполните следующую команду: `echo '/tmp/core_%e.%p' | sudo tee /proc/sys/kernel/core_pattern`. 

### <a name="next-steps"></a>Дальнейшие действия

* [Сбор журналов с помощью системы диагностики Azure](./service-fabric-diagnostics-event-aggregation-lad.md).
* [Локальное отслеживание и диагностика служб](service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally-linux.md).
