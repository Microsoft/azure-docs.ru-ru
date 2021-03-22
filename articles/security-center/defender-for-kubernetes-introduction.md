---
title: Преимущества и функции Azure Defender для Kubernetes
description: Сведения о преимуществах и функциях Azure Defender для Kubernetes.
author: memildin
ms.author: memildin
ms.date: 02/07/2021
ms.topic: overview
ms.service: security-center
manager: rkarlin
ms.openlocfilehash: 83d0215ebca9d60d61937cb20bb82c7ccb30aac1
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102100633"
---
# <a name="introduction-to-azure-defender-for-kubernetes"></a>Общие сведения об Azure Defender для Kubernetes

Служба Azure Kubernetes (AKS) — это управляемая служба Майкрософт для разработки, развертывания контейнерных приложений и управления ими.

Центр безопасности Azure и AKS представляют собой лучшее из ориентированных на облако решений по обеспечению безопасности Kubernetes. Они обеспечивают усиление защиты среды, защиту рабочих нагрузок и защиту во время выполнения, как описано в статье [Защита контейнеров в Центре безопасности](container-security.md).

Для обнаружения угроз в кластерах Kubernetes включите **Azure Defender для Kubernetes**.

Обнаружение угроз на уровне узла доступно для узлов AKS в Linux при включенном [Azure Defender для серверов](defender-for-servers-introduction.md) и агенте Log Analytics. Но если кластер AKS развертывается в масштабируемом наборе виртуальных машин, то агент Log Analytics пока не поддерживается.

## <a name="availability"></a>Доступность

|Аспект|Сведения|
|----|:----|
|Состояние выпуска:|Общедоступная версия|
|Цены|Плата за использование **Azure Defender для Kubernetes** начисляется по тарифам, приведенным на странице [Цены на Центр безопасности](https://azure.microsoft.com/pricing/details/security-center/).|
|Требуемые роли и разрешения|**Администратор безопасности** может отклонять предупреждения.<br>**Читатель сведений о безопасности** может просматривать результаты.|
|Облако.|![Да](./media/icons/yes-icon.png) Коммерческие облака<br>![Да](./media/icons/yes-icon.png) Национальные и независимые (US Gov, China Gov, другие правительственные облака)|
|||

## <a name="what-are-the-benefits-of-azure-defender-for-kubernetes"></a>Каковы преимущества Azure Defender для Kubernetes?

Azure Defender для Kubernetes обеспечивает **защиту от угроз на уровне кластера**, отслеживая службы, управляемые AKS, с помощью журналов, полученных Службой Azure Kubernetes (AKS).

Примерами событий безопасности, которые отслеживает Azure Defender для Kubernetes, могут служить представление панелей мониторинга Kubernetes, создание ролей с высоким уровнем привилегий и создание конфиденциальных подключений. Полный список оповещений на уровне кластера AKS см. в [справочной таблице оповещений](alerts-reference.md#alerts-akscluster).

> [!TIP]
> Вы можете имитировать оповещения для контейнеров, выполнив инструкции в [этой записи блога](https://techcommunity.microsoft.com/t5/azure-security-center/how-to-demonstrate-the-new-containers-features-in-azure-security/ba-p/1011270).

Следует также отметить, что наша международная команда специалистов по безопасности постоянно отслеживает ландшафт угроз. Они добавляют оповещение и уязвимости для конкретных контейнеров по мере их обнаружения.

>[!NOTE]
> Центр безопасности создает оповещения системы безопасности для действий и развертываний Службы Azure Kubernetes, происходящих **после** включения Azure Defender для Kubernetes.




## <a name="azure-defender-for-kubernetes---faq"></a>Вопросы и ответы по Azure Defender для Kubernetes

### <a name="can-i-still-get-aks-protections-without-the-log-analytics-agent"></a>Можно ли сейчас обеспечить защиту AKS без агента Log Analytics?

**Azure Defender для Kubernetes** обеспечивает защиту на уровне кластера. Если вы также развертываете агент Log Analytics решения **Azure Defender для серверов**, вы получите защиту от угроз для своих узлов, которая предоставляется в этом плане. Дополнительные сведения см. в статье [Общие сведения об Azure Defender для серверов](defender-for-servers-introduction.md).

Мы рекомендуем развернуть оба инструмента, чтобы обеспечить максимально полную защиту.

Если вы решите не устанавливать агенты на узлах, то получите только часть преимуществ защиты от угроз и лишь некоторые из оповещений системы безопасности. Вы будете по-прежнему получать оповещения, связанные с анализом сети и обменом данными с вредоносными серверами.

### <a name="does-aks-allow-me-to-install-custom-vm-extensions-on-my-aks-nodes"></a>Разрешает ли AKS устанавливать пользовательские расширения виртуальных машин в узлах AKS?
Чтобы средство Azure Defender отслеживало узлы AKS, на них должен работать агент Log Analytics. 

AKS — это управляемая служба, а так как агент Log analytics является расширением, управляемым корпорацией Майкрософт, его также поддерживают кластеры AKS.

### <a name="if-my-cluster-is-already-running-an-azure-monitor-for-containers-agent-do-i-need-the-log-analytics-agent-too"></a>Если в моем кластере уже выполняется Azure Monitor для агента контейнеров, мне также нужен агент Log Analytics?
Чтобы средство Azure Defender отслеживало узлы AKS, на них должен работать агент Log Analytics.

Если в кластерах уже используется агент Azure Monitor для контейнеров, вы можете установить и агент Log Analytics. Два агента могут работать параллельно без каких бы то ни было проблем.

[См. сведения об Azure Monitor для агента контейнеров](../azure-monitor/containers/container-insights-manage-agent.md).


## <a name="next-steps"></a>Дальнейшие действия

Из этой статьи вы узнали о защите Kubernetes в Центре безопасности, в том числе об Azure Defender для Kubernetes. 

> [!div class="nextstepaction"]
> [Включение Azure Defender](enable-azure-defender.md)

Связанные материалы см. в следующих статьях: 

- [Потоковая передача оповещений в решение SIEM, SOAR или решение для управления ИТ-услугами](export-to-siem.md)
- [Справочная таблица предупреждений](alerts-reference.md)
