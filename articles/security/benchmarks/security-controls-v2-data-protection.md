---
title: Тестирование безопасности Azure v2 — защита данных
description: Защита данных с учетом производительности Azure с производительностью 2
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 02/22/2021
ms.author: mbaldwin
ms.custom: security-benchmark
ms.openlocfilehash: 50358eed580bbd83f25386feb0068a252060672b
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102037121"
---
# <a name="security-control-v2-data-protection"></a>Управление безопасностью v2: защита данных

Защита данных охватывает управление защитой данных при хранении, передачей и через полномочные механизмы доступа. Сюда входит обнаружение, классификация, защита и мониторинг ресурсов конфиденциальных данных с помощью контроля доступа, шифрования и ведения журнала в Azure.

Чтобы просмотреть подходящую встроенную политику Azure, ознакомьтесь со статьей [сведения о встроенной инициативе по обеспечению безопасности в Azure: защита данных](../../governance/policy/samples/azure-security-benchmark.md#data-protection)

## <a name="dp-1-discovery-classify-and-label-sensitive-data"></a>DP-1: обнаружение, классификация и пометка конфиденциальных данных

| Идентификатор Azure | ИДЕНТИФИКАТОРы элементов управления CIS v 7.1 | Директива NIST SP 800-53 R4 ID (s) |
|--|--|--|--|
| DP-1 | 13,1, 14,5, 14,7 | SC-28 |

Выявлять, классифицировать и помечать конфиденциальные данные, чтобы можно было разрабатывать соответствующие элементы управления для обеспечения хранения, обработки и передачи конфиденциальных данных в технологические системы Организации.

Используйте службу Azure Information Protection (и связанное с ней средство сканирования) для обработки конфиденциальной информации в документах Office в Azure, локальной среде, Office 365 и в других расположениях.

Вы можете использовать службу Azure SQL Information Protection, которая может помочь классифицировать и пометить информацию, хранящуюся в Базах данных SQL Azure.

- [Добавление тегов к конфиденциальной информации с помощью Azure Information Protection](/azure/information-protection/what-is-information-protection) 

- [Реализация обнаружения данных Azure SQL](../../azure-sql/database/data-discovery-and-classification-overview.md)

**Ответственность**: Совмещаемая блокировка

**Заинтересованные лица по безопасности клиентов** (дополнительные [сведения](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)):

- [Безопасность приложений и DevOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops)

- [Безопасность данных](/azure/cloud-adoption-framework/organize/cloud-security-data-security) 

- [Безопасность инфраструктуры и конечных точек](/azure/cloud-adoption-framework/organize/cloud-security-infrastructure-endpoint)

## <a name="dp-2-protect-sensitive-data"></a>DP-2: защита конфиденциальных данных

| Идентификатор Azure | ИДЕНТИФИКАТОРы элементов управления CIS v 7.1 | Директива NIST SP 800-53 R4 ID (s) |
|--|--|--|--|
| DP-2 | 13,2, 2,10 | SC-7, AC-4 |

Защитите конфиденциальные данные, запретив доступ с помощью управления доступом на основе ролей Azure (Azure RBAC), управления доступом на основе сети и определенных элементов управления в службах Azure (например, шифрование в SQL и других базах данных). 

Чтобы гарантировать постоянное управление доступом, все его типы должны быть согласованы с вашей корпоративной стратегией сегментации. Корпоративная стратегия сегментации также должна содержать информацию о расположении конфиденциальных или критически важных для бизнеса данных и систем.

Для базовой платформы, управляемой корпорацией Майкрософт, корпорация Майкрософт считает все содержимое клиента конфиденциальным и защищает клиентов от потери данных и раскрытия информации. Чтобы обеспечить безопасность данных клиентов в Azure, корпорация Майкрософт реализовала несколько элементов управления и возможностей защиты данных по умолчанию.

- [Управление доступом Azure на основе ролей (Azure RBAC)](../../role-based-access-control/overview.md)

- [Общие сведения о защите данных клиентов в Azure](../fundamentals/protection-customer-data.md)

**Ответственность**: Совмещаемая блокировка

**Заинтересованные лица по безопасности клиентов** (дополнительные [сведения](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)):

- [Безопасность приложений и DevOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops) 

- [Безопасность данных](/azure/cloud-adoption-framework/organize/cloud-security-data-security)

- [Безопасность инфраструктуры и конечных точек](/azure/cloud-adoption-framework/organize/cloud-security-infrastructure-endpoint)

## <a name="dp-3-monitor-for-unauthorized-transfer-of-sensitive-data"></a>DP-3: отслеживание несанкционированного переноса конфиденциальных данных

| Идентификатор Azure | ИДЕНТИФИКАТОРы элементов управления CIS v 7.1 | Директива NIST SP 800-53 R4 ID (s) |
|--|--|--|--|
| DP-3 | 13,3 | AC-4, SI-4 |

Наблюдение за несанкционированным переносом данных в расположения за пределами корпоративной видимости и контроля. Обычно этот процесс предусматривает мониторинг за аномальными действиями (большими или необычными передачами данных), которые могут указывать на несанкционированную кражу данных. 

Расширенная защита от угроз (ATP) для службы хранилища Azure и Azure SQL ATP может оповещать об аномальных передачах информации, что может указывать на несанкционированную передачу конфиденциальной информации. 

Azure Information Protection (AIP) предоставляет возможности мониторинга сведений, которые были классифицированы и помечены. 

Если необходимо обеспечить соответствие политики защиты от потери данных (DLP), можно использовать решение DLP на основе узла, чтобы принудительно применять выявляющие и (или) профилактические элементы управления для предотвращения кражи данных.

- [Azure Defender для SQL](../../azure-sql/database/azure-defender-for-sql.md)

- [Azure Defender для службы хранилища](../../storage/common/azure-defender-storage-configure.md?tabs=azure-security-center)

**Ответственность**: Совмещаемая блокировка

**Заинтересованные лица по безопасности клиентов** (дополнительные [сведения](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)):

- [Операции безопасности](/azure/cloud-adoption-framework/organize/cloud-security) 

- [Безопасность приложений и DevOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops) 

- [Безопасность инфраструктуры и конечных точек](/azure/cloud-adoption-framework/organize/cloud-security-infrastructure-endpoint)

## <a name="dp-4-encrypt-sensitive-information-in-transit"></a>DP-4: шифрование конфиденциальной информации во время передачи

| Идентификатор Azure | ИДЕНТИФИКАТОРы элементов управления CIS v 7.1 | Директива NIST SP 800-53 R4 ID (s) |
|--|--|--|--|
| DP-4 | 14,4 | SC-8 |

Для дополнения элементов управления доступом данные в передаче должны быть защищены от атак с использованием аппаратного контроллера управления (например, для записи трафика) с помощью шифрования, чтобы злоумышленники не могли легко считывать или изменять данные.

Хотя это необязательно для трафика в частных сетях, это важно для трафика во внешних и общедоступных сетях. Для трафика HTTP убедитесь, что все клиенты, подключающиеся к ресурсам Azure, могут согласовать TLS v 1.2 или более поздней версии. Для удаленного управления используйте SSH (для Linux) или RDP/TLS (для Windows) вместо незашифрованного протокола. Устаревшие версии и протоколы SSL, TLS и SSH, а слабые шифры должны быть отключены.

По умолчанию Azure обеспечивает шифрование данных при передаче между центрами обработки данных Azure.

- [Общие сведения о шифровании при передаче с помощью Azure](../fundamentals/encryption-overview.md#encryption-of-data-in-transit)

- [Сведения о безопасности TLS](/security/engineering/solving-tls1-problem)

- [Двойное шифрование для данных Azure при передаче](../fundamentals/double-encryption.md#data-in-transit)

**Ответственность**: Совмещаемая блокировка

**Заинтересованные лица по безопасности клиентов** (дополнительные [сведения](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)):

- [Архитектура безопасности](/azure/cloud-adoption-framework/organize/cloud-security-architecture) 

- [Безопасность инфраструктуры и конечных точек](/azure/cloud-adoption-framework/organize/cloud-security-infrastructure-endpoint)

- [Безопасность приложений и DevOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops) 

- [Безопасность данных](/azure/cloud-adoption-framework/organize/cloud-security-data-security)

## <a name="dp-5-encrypt-sensitive-data-at-rest"></a>DP-5: шифрование конфиденциальных неактивных данных

| Идентификатор Azure | ИДЕНТИФИКАТОРы элементов управления CIS v 7.1 | Директива NIST SP 800-53 R4 ID (s) |
|--|--|--|--|
| DP-5 | 14,8 | SC-28, SC-12 |

Для дополнения элементов управления доступом неактивных данных следует защищать от атак с использованием аппаратного контроллера управления (например, доступ к базовому хранилищу) с помощью шифрования. Это гарантирует, что злоумышленники не смогут легко считывать или изменять данные. 

По умолчанию Azure обеспечивает шифрование неактивных данных. Для высокочувствительных данных вы можете реализовать дополнительные возможности шифрования при хранении во всех ресурсах Azure, где это возможно. Azure управляет ключами шифрования по умолчанию, но Azure предоставляет возможности для управления собственными ключами (управляемыми клиентом ключами) для определенных служб Azure.

- [Общие сведения о шифровании неактивных данных в Azure](../fundamentals/encryption-atrest.md#encryption-at-rest-in-microsoft-cloud-services)

- [Настройка ключей шифрования, управляемых клиентом](../../storage/common/customer-managed-keys-configure-key-vault.md)

- [Модель шифрования и таблица управления ключами](../fundamentals/encryption-models.md)

- [Двойное шифрование неактивных данных в Azure](../fundamentals/double-encryption.md#data-at-rest)

**Ответственность**: Совмещаемая блокировка

**Заинтересованные лица по безопасности клиентов** (дополнительные [сведения](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)):

- [Архитектура безопасности](/azure/cloud-adoption-framework/organize/cloud-security-architecture) 

- [Безопасность инфраструктуры и конечных точек](/azure/cloud-adoption-framework/organize/cloud-security-infrastructure-endpoint)

- [Безопасность приложений и DevOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops)

- [Безопасность данных](/azure/cloud-adoption-framework/organize/cloud-security-data-security)