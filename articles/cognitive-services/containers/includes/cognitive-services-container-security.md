---
title: Безопасность контейнера
titleSuffix: Azure Cognitive Services
description: Узнайте, как защитить контейнер
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 12/17/2020
ms.author: aahi
ms.openlocfilehash: 4b106ebc5606c4e5a290e12728d4e2011e80f6a2
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97861837"
---
## <a name="azure-cognitive-services-container-security"></a>Безопасность контейнера Cognitive Services Azure

Безопасность должна быть основной задачей при разработке приложений. Важность безопасности является метрикой успеха. При проектировании программного решения, включающего Cognitive Services контейнеры, крайне важно понимать ограничения и возможности, доступные вам. Дополнительные сведения о сетевой безопасности см. в статье [Настройка виртуальных сетей Azure Cognitive Services][az-security].

> [!IMPORTANT]
> По умолчанию в API контейнера Cognitive Services *нет безопасности* . Это связано с тем, что чаще всего контейнер будет работать как часть Pod, которая защищена извне сетевым мостом. Однако можно включить проверку подлинности, которая работает аналогично проверке подлинности, используемой при доступе к [облачной Cognitive Services][request-authentication].

На схеме ниже показан подход по умолчанию и **небезопасный** метод.

![Безопасность контейнера](../media/container-security.svg)

В качестве альтернативного и *безопасного* подхода потребители Cognitive Services контейнеры могут расширить контейнер с интерфейсным компонентом, сохранив конечную точку контейнера закрытой. Рассмотрим ситуацию, когда мы используем [Istio][istio] в качестве входящего шлюза. Istio поддерживает HTTPS/TLS и проверку подлинности на основе сертификата клиента. В этом сценарии интерфейс Istio предоставляет доступ к контейнеру, представляя сертификат клиента, который утверждается заранее с помощью Istio.

[Nginx][nginx] — это еще один популярный вариант в той же категории. Как Istio, так и nginx выступают в качестве сетки службы и предлагают дополнительные функции, такие как балансировка нагрузки, маршрутизация и контроль скорости.

### <a name="container-networking"></a>Работа с контейнерами в сети

Контейнеры Cognitive Services необходимы для отправки сведений о метриках в целях выставления счетов. Если не разрешить перечисление различных сетевых каналов, от которых зависит контейнер Cognitive Services, не позволит контейнеру работать.

#### <a name="allow-list-cognitive-services-domains-and-ports"></a>Разрешить список доменов и портов Cognitive Services

Узел должен разрешить **порты 443** и следующие домены:

* `*.cognitive.microsoft.com`
* `*.cognitiveservices.azure.com`

#### <a name="disable-deep-packet-inspection"></a>Отключение глубокой проверки пакетов

[Глубокая проверка пакетов](https://en.wikipedia.org/wiki/Deep_packet_inspection) (DPI) — это тип обработки данных, который подробно анализирует данные, передаваемые по компьютерной сети, и обычно выполняет действия путем блокировки, повторной маршрутизации или ведения журнала.

Отключите DPI на защищенных каналах, которые контейнеры Cognitive Services создают на серверах Майкрософт. В противном случае контейнер будет некорректно работать.

[istio]: https://istio.io/
[nginx]: https://www.nginx.com
[request-authentication]: ../../authentication.md
[az-security]: ../../cognitive-services-virtual-networks.md
