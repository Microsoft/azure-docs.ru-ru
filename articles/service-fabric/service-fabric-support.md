---
title: Сведения о вариантах поддержки Azure Service Fabric
description: Поддерживаемые версии кластера Service Fabric и ссылки для отправки запросов в службу поддержки.
author: pkcsf
ms.topic: troubleshooting
ms.date: 8/24/2018
ms.author: pkc
ms.openlocfilehash: 977cd79de629670cef526f072340f8897fa6446e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92316500"
---
# <a name="azure-service-fabric-support-options"></a>Варианты поддержки Azure Service Fabric

Мы создали ряд вариантов запроса на поддержку для удовлетворения потребностей управления кластерами Service Fabric и рабочими нагрузками приложений. В зависимости от срочности необходимой поддержки и серьезности проблемы вы можете выбрать подходящий вариант.

## <a name="report-production-issues-or-request-paid-support-for-azure"></a>Сообщение о проблемах рабочей среды или запрос платной поддержки для Azure

Чтобы сообщить о проблемах, связанных с кластером Service Fabric, работающим в Azure, откройте запрос в службу поддержки [на портал Azure](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview) или на [портале службы поддержки Майкрософт](https://support.microsoft.com/oas/default.aspx?prid=16146).

См. также:
 
- [Поддержка корпорации Майкрософт для клиентов Azure](https://azure.microsoft.com/support/plans/?b=16.44).
- [Поддержка Premier](https://support.microsoft.com/en-us/premier).

> [!Note]
> Кластеры, работающие на уровне бронзовой надежности или кластере с одним узлом, позволяют выполнять только тестовые рабочие нагрузки. При возникновении проблем с кластером с бронзовым уровнем надежности или кластером с одним узлом техническая поддержка Майкрософт поможет устранить неполадку, но анализ первопричин проводить не будет. Дополнительные сведения см. в статье [характеристики надежности кластера](./service-fabric-cluster-capacity.md#reliability-characteristics-of-the-cluster).
>
> Дополнительные сведения о том, что требуется для пригодного для промышленной эксплуатации кластера, см. в статье [Контрольный список готовности рабочей среды](./service-fabric-production-readiness-checklist.md).

<a id="getlivesitesupportonprem"></a>

## <a name="report-production-issues-or-request-paid-support-for-standalone-service-fabric-clusters"></a>Сообщение о проблемах рабочей среды или запрос платной поддержки для изолированных кластеров Service Fabric

Чтобы сообщить о проблемах, связанных с Service Fabric кластерами, работающими локально или в других облаках, вы можете открыть билет для профессиональной поддержки на [портале службы поддержки Майкрософт](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview).

См. также:

- [Professional Support Options](https://support.microsoft.com/en-us/gp/offerprophone?wa=wsignin1.0) (Варианты профессиональной поддержки).
- [Поддержка Premier](https://support.microsoft.com/en-us/premier).

## <a name="report-azure-service-fabric-issues"></a>Сообщение о проблемах Azure Service Fabric

Мы настроили репозиторий GitHub для сообщения о проблемах Service Fabric.  Кроме того, мы активно наблюдаем за приведенными ниже форумами.

### <a name="github-repo"></a>Репозиторий GitHub 

Сообщите о проблемах Service Fabric Azure на сайте [Service Fabric GitHub](https://github.com/microsoft/service-fabric/issues). Этот репозиторий предназначен для составления отчетов и отслеживания проблем, а также для создания небольших запросов функций, связанных с Service Fabric Azure. **Не используйте этот носитель для сообщения о проблемах в реальном** рабочем месте.

### <a name="stackoverflow-and-msdn-forums"></a>Сайт StackOverflow и форум MSDN

[Тег Service Fabric на StackOverflow][stackoverflow] и [форум Service Fabric в MSDN][msdn-forum] лучше использовать для получения общих вопросов о том, как работает платформа, и о том, как ее можно использовать для выполнения определенных задач.

### <a name="azure-feedback-forum"></a>Форум отзывов и предложений по Azure

На [форуме обратной связи Azure для Service Fabric][uservoice-forum] — лучшее место для отправки значительных идей с функциями продукта. Мы рассмотрим самые популярные запросы и рассматривайте их для нашего среднего и долгосрочного планирования. Мы рекомендуем вам обращаться с предложениями в службу поддержки в этом сообществе.

## <a name="service-fabric-preview-versions---unsupported-for-production-use"></a>Предварительные версии Service Fabric не поддерживают работу в рабочей среде.

Иногда мы создаем специальные предварительные выпуски, содержащие существенные изменения функций, для которых мы хотели бы проделать опрос с ранними отзывами. Предварительные версии следует использовать только в изолированных тестовых средах, которые не обслуживают рабочие нагрузки. На производственном кластере всегда должна быть запущена поддерживаемая стабильная версия Service Fabric. Мы не предлагаем платный вариант поддержки для этих предварительных выпусков.

Предварительная версия всегда начинается с номера основной и вспомогательной версии, равного 255. Например, если отображается Service Fabric версии 255.255.5703.949, эта версия находится на этапе предварительной версии и предназначена только для использования в тестовых кластерах. О выходе этих предварительных выпусков также объявляется в [блоге группы Service Fabric](https://techcommunity.microsoft.com/t5/azure-service-fabric/bg-p/Service-Fabric) и приводятся сведения о функциях, включенных в их состав. Чтобы задать вопросы или отправить отзыв, воспользуйтесь одной из возможностей в разделе [Сообщение о проблемах Azure Service Fabric](#report-azure-service-fabric-issues).

## <a name="next-steps"></a>Дальнейшие действия

[Поддерживаемые версии Service Fabric](service-fabric-versions.md)

<!--references-->
[Microsoft Q&A question page]: /answers/topics/azure-service-fabric.html
[stackoverflow]: https://stackoverflow.com/questions/tagged/azure-service-fabric
[uservoice-forum]: https://feedback.azure.com/forums/293901-service-fabric
[acom-docs]: ./index.yml
[sample-repos]: /samples/browse/?products=azure
[msdn-forum]: https://social.msdn.microsoft.com/forums/azure/en-US/home?category=windowsazureplatform