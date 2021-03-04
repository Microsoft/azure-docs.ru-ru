---
title: Контрольный уровень безопасности Azure v2 — Сетевая безопасность
description: Безопасность сети, производительность системы безопасности Azure версии 2
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 02/22/2021
ms.author: mbaldwin
ms.custom: security-benchmark
ms.openlocfilehash: b47f833a3b47dae145aa18eb6eda0c6bd95f6689
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102042919"
---
# <a name="security-control-v2-network-security"></a>Управление безопасностью версии 2: Сетевая безопасность

Безопасность сети охватывает элементы управления для защиты и защиты сетей Azure. Сюда входит защита виртуальных сетей, установка частных подключений, предотвращение и устранение внешних атак, а также защита DNS.

Чтобы просмотреть подходящую встроенную политику Azure, ознакомьтесь с [подробными сведениями о встроенной инициативе по обеспечению безопасности в Azure: Сетевая безопасность](../../governance/policy/samples/azure-security-benchmark.md#network-security)

## <a name="ns-1-implement-security-for-internal-traffic"></a>NS-1: реализация безопасности для внутреннего трафика

| Идентификатор Azure | ИДЕНТИФИКАТОРы элементов управления CIS v 7.1 | Директива NIST SP 800-53 R4 ID (s) |
|--|--|--|--|
| NS-1 | 9,2, 9,4, 14,1, 14,2, 14,3 | AC-4, CA-3, SC-7 |

Убедитесь, что все виртуальные сети Azure следуют принципу сегментации предприятия, который соответствует бизнес-рискам. Любая система, которая может повлечь за собой повышенный риск для Организации, должна быть изолирована в собственной виртуальной сети и достаточно защищена с помощью группы безопасности сети (NSG) или брандмауэра Azure.

В зависимости от ваших приложений и стратегии сегментации предприятия, ограничьте или разрешите трафик между внутренними ресурсами на основе правил группы безопасности сети. Для определенных четко определенных приложений (например, 3-уровневого приложения) это может быть высокозащищенным подходом «Deny по умолчанию, разрешить исключение». Это может плохо масштабироваться, если несколько приложений и конечных точек взаимодействуют друг с другом. Вы также можете использовать брандмауэр Azure в тех случаях, когда централизованное управление требуется для большого количества сегментов предприятия или периферийных серверов (в топологии звезды).

Используйте Адаптивное усиление защиты сети в центре безопасности Azure, чтобы рекомендовать конфигурации групп безопасности сети, ограничивающие порты и исходные IP-адреса в соответствии со ссылками на правила внешнего сетевого трафика.

Используйте метку Azure для обнаружения использования устаревших небезопасных протоколов, таких как SSL/TLSv1, SMBv1, LM/аутентификация ntlmv1, wDigest, неподписанные привязки LDAP и слабые шифры в Kerberos.

- [Создание группы безопасности сети с правилами безопасности](../../virtual-network/tutorial-filter-network-traffic.md)

- [Развертывание и настройка брандмауэра Azure](../../firewall/tutorial-firewall-deploy-portal.md)

- [Адаптивное усиление защиты сети в центре безопасности Azure](../../security-center/security-center-adaptive-network-hardening.md)

- [Книга незащищенных протоколов Azure Sentinel](../../sentinel/quickstart-get-visibility.md#use-built-in-workbooks)

**Ответственность**: Customer

**Заинтересованные лица по безопасности клиентов** (дополнительные [сведения](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)):

- [Архитектура безопасности](/azure/cloud-adoption-framework/organize/cloud-security-architecture)

- [Управление состоянием](/azure/cloud-adoption-framework/organize/cloud-security-posture-management)

- [Безопасность приложений и DevOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops)

## <a name="ns-2-connect-private-networks-together"></a>NS-2. Совместное подключение частных сетей

| Идентификатор Azure | ИДЕНТИФИКАТОРы элементов управления CIS v 7.1 | Директива NIST SP 800-53 R4 ID (s) |
|--|--|--|--|
| NS-2 | Н/Д | ЦС-3, AC-17, MA-4 |

Используйте Azure ExpressRoute или виртуальную частную сеть Azure (VPN) для создания частных подключений между центрами обработки данных Azure и локальной инфраструктурой в среде совместного размещения. Подключения ExpressRoute не проходят через общедоступный Интернет и обеспечивают повышенную надежность, скорость и задержку, чем обычные подключения к Интернету. Для VPN-подключения типа "точка — сеть" и VPN-подключения типа "сеть — сеть" можно подключить локальные устройства или сети к виртуальной сети, используя любое сочетание параметров VPN и Azure ExpressRoute.

Для совместного подключения нескольких виртуальных сетей в Azure используйте пиринг виртуальных сетей или частную ссылку. Сетевой трафик между одноранговыми виртуальными сетями является частным и хранится в магистральной сети Azure.

- [Что такое модели подключения ExpressRoute](../../expressroute/expressroute-connectivity-models.md)

- [Общие сведения об Azure VPN](../../vpn-gateway/vpn-gateway-about-vpngateways.md)

- [Пиринг виртуальной сети](../../virtual-network/virtual-network-peering-overview.md)

- [Приватный канал Azure](../../private-link/private-link-service-overview.md)

**Ответственность**: Customer

**Заинтересованные лица по безопасности клиентов** (дополнительные [сведения](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)):

- [Архитектура безопасности](/azure/cloud-adoption-framework/organize/cloud-security-architecture)

- [Управление состоянием](/azure/cloud-adoption-framework/organize/cloud-security-posture-management)

- [Безопасность приложений и DevOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops)

## <a name="ns-3-establish-private-network-access-to-azure-services"></a>NS-3: установка доступа к частной сети для служб Azure

| Идентификатор Azure | ИДЕНТИФИКАТОРы элементов управления CIS v 7.1 | Директива NIST SP 800-53 R4 ID (s) |
|--|--|--|--|
| NS-3 | 14,1 | AC-4, CA-3, SC-7 |

Используйте частную связь Azure, чтобы включить частный доступ к службам Azure из ваших виртуальных сетей, не переключаясь в Интернет. В ситуациях, когда частная ссылка Azure пока недоступна, используйте конечные точки службы виртуальной сети Azure. Конечные точки службы виртуальной сети Azure обеспечивают безопасный доступ к службам с помощью оптимизированного маршрута через магистральную сеть Azure.

Частный доступ — это дополнительная детальная мера защиты в дополнение к проверке подлинности и безопасности трафика, предоставляемых службами Azure.

- [Общие сведения о частной ссылке Azure](../../private-link/private-link-overview.md)

- [Общие сведения о конечных точках службы виртуальной сети](../../virtual-network/virtual-network-service-endpoints-overview.md)

**Ответственность**: Customer

**Заинтересованные лица по безопасности клиентов** (дополнительные [сведения](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)):

- [Архитектура безопасности](/azure/cloud-adoption-framework/organize/cloud-security-architecture)

- [Управление состоянием](/azure/cloud-adoption-framework/organize/cloud-security-posture-management)

- [Безопасность приложений и DevOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops) 

## <a name="ns-4-protect-applications-and-services-from-external-network-attacks"></a>NS-4: Защита приложений и служб от внешних сетевых атак

| Идентификатор Azure | ИДЕНТИФИКАТОРы элементов управления CIS v 7.1 | Директива NIST SP 800-53 R4 ID (s) |
|--|--|--|--|
| NS-4 | 9,5, 12,3, 12,9 | SC-5, SC-7 |

Защитите ресурсы Azure от атак из внешних сетей, в том числе от распределенных атак типа "отказ в обслуживании" (от атак DDoS), атак, зависящих от приложения, а также незапрошенных и потенциально вредоносных интернет-трафика. Azure включает в себя собственные возможности для этого:
-   Используйте брандмауэр Azure для защиты приложений и служб от потенциально вредоносного трафика из Интернета и других внешних расположений. 

-   Используйте возможности брандмауэра веб-приложения (WAF) в шлюзе приложений Azure, в передней дверце Azure и сети доставки содержимого (CDN) Azure для защиты приложений, служб и интерфейсов API от атак уровня приложения.

-   Защитите свои ресурсы от атак от атак DDoS, включив стандартную защиту от атак DDoS в виртуальных сетях Azure.
-   Используйте центр безопасности Azure для обнаружения рисков с несоответствующими настройками, связанными с указанным выше.

- [Документация по брандмауэру Azure](../../firewall/index.yml)

- [Развертывание WAF Azure](../../web-application-firewall/overview.md)

- [Управление защитой от атак DDoS Azure уровня "Стандартный" с помощью портала Azure](../../ddos-protection/manage-ddos-protection.md)

**Ответственность**: Customer

**Заинтересованные лица по безопасности клиентов** (дополнительные [сведения](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)):

- [Архитектура безопасности](/azure/cloud-adoption-framework/organize/cloud-security-architecture)

- [Управление состоянием](/azure/cloud-adoption-framework/organize/cloud-security-posture-management)

- [Безопасность приложений и DevOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops)

## <a name="ns-5-deploy-intrusion-detectionintrusion-prevention-systems-idsips"></a>NS-5: развертывание систем обнаружения вторжений и предотвращения вторжений (ИДЕНТИФИКАТОРы и IP-адреса)

| Идентификатор Azure | ИДЕНТИФИКАТОРы элементов управления CIS v 7.1 | Директива NIST SP 800-53 R4 ID (s) |
|--|--|--|--|
| NS-5 | 12,6, 12,7 | SI-4 |

Используйте фильтрацию на основе интеллектуального анализа угроз Azure для оповещения о передаче и/или блокировке трафика между известными вредоносными IP-адресами и доменами. IP-адреса и домены также передаются из канала Microsoft Threat Intelligence. Если требуется проверка полезной нагрузки, вы можете развернуть сторонние средства обнаружения вторжений или системы предотвращения вторжения (ИДЕНТИФИКАТОРы и IP-адреса) из Azure Marketplace с возможностями проверки полезной нагрузки. Кроме того, вы можете использовать ИДЕНТИФИКАТОРы или IP-адреса на основе узла или решение для обнаружения конечных точек и ответа (ЕДР) в сочетании с сетевыми ИДЕНТИФИКАТОРами или IP-адресами.

Примечание. Если у вас есть нормативная информация или другие требования для использования ИДЕНТИФИКАТОРов и IP-адресов, убедитесь, что она всегда настроена на предоставление высококачественных оповещений для решения SIEM. 

- [Развертывание брандмауэра Azure](../../firewall/tutorial-firewall-deploy-portal.md)

- [Azure Marketplace включает сторонние возможности идентификации.](https://azuremarketplace.microsoft.com/marketplace?search=IDS)

- [Защитник Майкрософт для возможностей конечной точки](/windows/security/threat-protection/microsoft-defender-atp/overview-endpoint-detection-response)

**Ответственность**: Customer

**Заинтересованные лица по безопасности клиентов** (дополнительные [сведения](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)):

- [Архитектура безопасности](/azure/cloud-adoption-framework/organize/cloud-security-architecture)

- [Управление состоянием](/azure/cloud-adoption-framework/organize/cloud-security-posture-management)

- [Безопасность приложений и DevOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops)

## <a name="ns-6-simplify-network-security-rules"></a>NS-6: Упростите правила безопасности сети.

| Идентификатор Azure | ИДЕНТИФИКАТОРы элементов управления CIS v 7.1 | Директива NIST SP 800-53 R4 ID (s) |
|--|--|--|--|
| NS-6 | 1.5 | IA-4 |

Упростите правила безопасности сети, используя теги службы и группы безопасности приложений (ASG).

Используйте теги службы виртуальной сети для определения элементов управления доступом к сети для групп безопасности сети или брандмауэра Azure. Теги служб можно использовать вместо определенных IP-адресов при создании правил безопасности. Указав имя тега службы в поле источника или назначения правила, можно разрешить или запретить трафик для соответствующей службы. Корпорация Майкрософт управляет префиксами адресов, входящих в тег службы, и автоматически обновляет этот тег при изменении адресов.

Группы безопасности приложений также можно использовать для упрощения сложной настройки безопасности. Вместо того чтобы определять политику на основе явных IP-адресов в группах безопасности сети, группы безопасности приложений позволяют настроить сетевую безопасность как естественное расширение структуры приложения, что позволяет группировать виртуальные машины и определять политики безопасности сети на основе этих групп.

- [Общие сведения и использование тегов служб](../../virtual-network/service-tags-overview.md)

- [Общие сведения и использование групп безопасности приложений](../../virtual-network/network-security-groups-overview.md#application-security-groups)

**Ответственность**: Customer

**Заинтересованные лица по безопасности клиентов** (дополнительные [сведения](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)):

- [Архитектура безопасности](/azure/cloud-adoption-framework/organize/cloud-security-architecture)

- [Управление состоянием](/azure/cloud-adoption-framework/organize/cloud-security-posture-management)

- [Безопасность приложений и DevOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops)

## <a name="ns-7-secure-domain-name-service-dns"></a>NS-7: служба защиты доменных имен (DNS)

| Идентификатор Azure | ИДЕНТИФИКАТОРы элементов управления CIS v 7.1 | Директива NIST SP 800-53 R4 ID (s) |
|--|--|--|--|
| NS-7 | Н/Д | SC-20, SC-21 |

Следуйте рекомендациям по обеспечению безопасности DNS, чтобы устранить распространенные атаки, такие как висячие DNS, атаки несанкционированного доступа к DNS, неустранение и подмена DNS и т. д.

Если в качестве полномочной службы DNS используется Azure DNS, убедитесь, что зоны и записи DNS защищены от случайного или вредоносного изменения с помощью Azure RBAC и блокировок ресурсов.

- [Обзор Azure DNS](../../dns/dns-overview.md)

- [Рекомендации по развертыванию системы защиты доменных имен (DNS)](https://csrc.nist.gov/publications/detail/sp/800-81/2/final)

- [Предотвращение висячих записей DNS и избежание поддоменного перенаправление](../fundamentals/subdomain-takeover.md)

**Ответственность**: Customer

**Заинтересованные лица по безопасности клиентов** (дополнительные [сведения](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)):

- [Архитектура безопасности](/azure/cloud-adoption-framework/organize/cloud-security-architecture)

- [Управление состоянием](/azure/cloud-adoption-framework/organize/cloud-security-posture-management)

- [Безопасность приложений и DevOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops)
