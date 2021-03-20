---
title: Проверки Service Fabric Azure
description: Как смоделировать проверку на актуальность в Azure Service Fabric с помощью файлов манифеста приложения и службы.
ms.topic: conceptual
author: tugup
ms.author: tugup
ms.date: 3/12/2020
ms.openlocfilehash: 07a1b836ca7ea79244e303f54654dfcaa6e5fcb9
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "82137592"
---
# <a name="liveness-probe"></a>Проверка актуальности
Начиная с версии 7,1 Azure Service Fabric поддерживает механизм проверки актуальности для [контейнерных][containers-introduction-link] приложений. Проверка на актуальность помогает сообщить о жизни контейнерного приложения, которое будет перезапущено, если оно не ответит быстро.
В этой статье приводятся общие сведения об определении проверки актуальности с помощью файлов манифеста.

Прежде чем продолжить работу с этой статьей, ознакомьтесь с [моделью Service Fabric приложения][application-model-link] и [моделью размещения Service Fabric][hosting-model-link].

> [!NOTE]
> Проверка актуальности поддерживается только для контейнеров в режиме сети NAT.

## <a name="semantics"></a>Семантика
Можно указать только одну проверку актуальности на контейнер и управлять его поведением с помощью следующих полей:

* `initialDelaySeconds`: Начальная задержка в секундах, с которой начинается выполнение пробы после запуска контейнера. Поддерживаемое значение — **int**. Значение по умолчанию равно 0, а минимальное значение — 0.

* `timeoutSeconds`: Период в секундах, по истечении которого пробный зонд считается неудачным, если он не был успешно завершен. Поддерживаемое значение — **int**. Значение по умолчанию — 1, а минимальное — 1.

* `periodSeconds`: Период в секундах, указывающий частоту зонда. Поддерживаемое значение — **int**. Значение по умолчанию — 10, а минимальное — 1.

* `failureThreshold`: При достижении этого значения контейнер будет перезапущен. Поддерживаемое значение — **int**. Значение по умолчанию — 3, а минимальное — 1.

* `successThreshold`: В случае сбоя, чтобы проверка считалась успешной, она должна успешно выполняться для этого значения. Поддерживаемое значение — **int**. Значение по умолчанию — 1, а минимальное — 1.

В любой момент времени в один контейнер может быть один зонд. Если проба не закончится в течение времени, заданного в **timeoutSeconds**, подождите и подсчитайте время до **фаилуресрешолд**. 

Кроме того, Service Fabric будет создавать следующие [отчеты о работоспособности][health-introduction-link] пробы в **DeployedServicePackage**:

* `OK`: Зонд проходит проверку на значение, заданное в **сукцесссрешолд**.

* `Error`: Зонд **failureCount**  ==   **фаилуресрешолд** перед перезапуском контейнера.

* `Warning`: 
    * Происходит сбой пробы и **failureCount**  <  **фаилуресрешолд**. Этот отчет о работоспособности остается до тех пор, пока **failureCount** не достигнет значения, установленного в **фаилуресрешолд** или **сукцесссрешолд**.
    * При успешном выполнении после сбоя предупреждение остается, но при этом обновляются последовательные успехи.

## <a name="specifying-a-liveness-probe"></a>Указание проверки на актуальность

Зонд можно указать в файле ApplicationManifest.xml в разделе **ServiceManifestImport**.

Проба может быть для любого из следующих элементов:

* HTTP
* TCP
* Exec 

### <a name="http-probe"></a>проба HTTP

Для проверки HTTP Service Fabric отправляет HTTP-запрос по указанному порту и указанному пути. Код возврата, который больше или равен 200 и меньше 400, указывает на успешное выполнение.

Ниже приведен пример указания HTTP-пробы.

```xml
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="Stateless1Pkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides />
    <Policies>
      <CodePackagePolicy CodePackageRef="Code">
        <Probes>
          <Probe Type="Liveness" FailureThreshold="5" SuccessThreshold="2" InitialDelaySeconds="10" PeriodSeconds="30" TimeoutSeconds="20">
            <HttpGet Path="/" Port="8081" Scheme="http">
              <HttpHeader Name="Foo" Value="Val"/>
              <HttpHeader Name="Bar" Value="val1"/>
            </HttpGet>
          </Probe>
        </Probes>
      </CodePackagePolicy>
    </Policies>
  </ServiceManifestImport>
```

Зонд HTTP имеет дополнительные свойства, которые можно задать.

* `path`: Путь, используемый в HTTP-запросе.

* `port`: Порт, используемый для зондов. Это свойство является обязательным. Диапазон — от 1 до 65535.

* `scheme`: Схема, используемая для подключения к пакету кода. Если это свойство имеет значение HTTPS, проверка сертификата пропускается. Значение по умолчанию — HTTP.

* `httpHeader`: Заголовки, заданные в запросе. Можно указать несколько заголовков.

* `host`: IP-адрес узла для подключения.

### <a name="tcp-probe"></a>проба TCP.

Для проверки TCP Service Fabric попытается открыть сокет в контейнере с помощью указанного порта. Если он может установить соединение, проба считается успешной. Ниже приведен пример указания пробы, использующей TCP-сокет.

```xml
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="Stateless1Pkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides />
    <Policies>
      <CodePackagePolicy CodePackageRef="Code">
        <Probes>
          <Probe Type="Liveness" FailureThreshold="5" SuccessThreshold="2" InitialDelaySeconds="10" PeriodSeconds="30" TimeoutSeconds="20">
            <TcpSocket Port="8081"/>
          </Probe>
        </Probes>
      </CodePackagePolicy>
    </Policies>
  </ServiceManifestImport>
```

### <a name="exec-probe"></a>Проверка Exec

Эта проба выдаст команду **exec** в контейнер и дождитесь завершения команды.

> [!NOTE]
> Команда **exec** принимает строку с разделителями-запятыми. Команда в следующем примере будет работать для контейнера Linux.
> При попытке проверки контейнера Windows используйте **cmd**.

```xml
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="Stateless1Pkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides />
    <Policies>
      <CodePackagePolicy CodePackageRef="Code">
        <Probes>
          <Probe Type="Liveness" FailureThreshold="5" SuccessThreshold="2" InitialDelaySeconds="10" PeriodSeconds="30" TimeoutSeconds="20">
            <Exec>
              <Command>ping,-c,2,localhost</Command>
            </Exec>
          </Probe>        
       </Probes>
      </CodePackagePolicy>
    </Policies>
  </ServiceManifestImport>
```

## <a name="next-steps"></a>Дальнейшие действия
Дополнительные сведения см. в следующей статье:
* [Service Fabric и контейнеры][containers-introduction-link]

<!-- Links -->
[containers-introduction-link]: service-fabric-containers-overview.md
[health-introduction-link]: service-fabric-health-introduction.md
[application-model-link]: service-fabric-application-model.md
[hosting-model-link]: service-fabric-hosting-model.md

