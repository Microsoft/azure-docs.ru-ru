---
title: Тестирование безопасности Azure v2 — безопасность конечных точек
description: Безопасность конечных точек системы безопасности Azure версии 2
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 02/22/2021
ms.author: mbaldwin
ms.custom: security-benchmark
ms.openlocfilehash: 48b22ba913370b27cd01319a14a2a627d7589ce4
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102051520"
---
# <a name="security-control-v2-endpoint-security"></a>Управление безопасностью v2: безопасность конечных точек

Безопасность конечных точек охватывает элементы управления при обнаружении и отклике конечных точек. Сюда входит использование функции обнаружения конечных точек и реагирования (ЕДР) и службы защиты от вредоносных программ для конечных точек в средах Azure.

Чтобы просмотреть подходящую встроенную политику Azure, ознакомьтесь с [подробными сведениями о встроенной инициативе по обеспечению безопасности в Azure: безопасность конечных точек.](../../governance/policy/samples/azure-security-benchmark.md#endpoint-security)

## <a name="es-1-use-endpoint-detection-and-response-edr"></a>ES-1. Использование обнаружения конечных точек и ответа (ЕДР)

| Идентификатор Azure | ИДЕНТИФИКАТОРы элементов управления CIS v 7.1 | Директива NIST SP 800-53 R4 ID (s) |
|--|--|--|--|
| ES-1 | 8.1 | SI-2, SI-3, SC-3 |

Включите возможности обнаружения конечных точек и реагирования (ЕДР) для серверов и клиентов, а также Интегрируйте процессы SIEM и операций безопасности.

Защитник Майкрософт для конечной точки предоставляет возможности ЕДР в составе корпоративной платформы безопасности, чтобы предотвратить, обнаруживать, исследовать и реагировать на сложные угрозы.

- [Обзор защитника Майкрософт для конечных точек](/windows/security/threat-protection/microsoft-defender-atp/microsoft-defender-advanced-threat-protection)

- [Защитник Майкрософт для конечных точек для серверов Windows](/windows/security/threat-protection/microsoft-defender-atp/configure-server-endpoints)

- [Защитник Майкрософт для конечных точек для серверов, отличных от Windows](/windows/security/threat-protection/microsoft-defender-atp/configure-endpoints-non-windows)

**Ответственность**: Customer

**Заинтересованные лица по безопасности клиентов** (дополнительные [сведения](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)):

- [Безопасность инфраструктуры и конечных точек](/azure/cloud-adoption-framework/organize/cloud-security)

- [Аналитика угроз](/azure/cloud-adoption-framework/organize/cloud-security-threat-intelligence)

- [Управление соответствием требованиям безопасности](/azure/cloud-adoption-framework/organize/cloud-security-compliance-management)

- [Управление состоянием](/azure/cloud-adoption-framework/organize/cloud-security-compliance-management)

## <a name="es-2-use-centrally-managed-modern-anti-malware-software"></a>ES-2. Использование централизованно управляемого современного программного обеспечения для защиты от вредоносных программ

| Идентификатор Azure | ИДЕНТИФИКАТОРы элементов управления CIS v 7.1 | Директива NIST SP 800-53 R4 ID (s) |
|--|--|--|--|
| ES-2 | 8.1 | SI-2, SI-3, SC-3 |

Использование централизованно управляемого решения для защиты от вредоносных программ на конечной точке с поддержкой реального времени и периодического сканирования

Центр безопасности Azure может автоматически обнаруживать использование ряда популярных решений для защиты от вредоносных программ для виртуальных машин, а также сообщать о состоянии запуска Endpoint Protection и делать рекомендации. 

Антивредоносное по Майкрософт для облачных служб Azure — это антивредоносное по по умолчанию для виртуальных машин Windows (ВМ). Для виртуальных машин Linux используйте стороннее решение для защиты от вредоносных программ. Кроме того, для обнаружения вредоносных программ, отправленных в учетные записи хранения Azure, можно использовать обнаружение угроз центра безопасности Azure для служб данных. 

- [Настройка антивредоносного по Майкрософт для облачных служб и виртуальных машин](../fundamentals/antimalware.md)

- [Поддерживаемые решения для защиты конечных точек](../../security-center/security-center-services.md?tabs=features-windows#supported-endpoint-protection-solutions-)

**Ответственность**: Customer

**Заинтересованные лица по безопасности клиентов** (дополнительные [сведения](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)):

- [Безопасность инфраструктуры и конечных точек](/azure/cloud-adoption-framework/organize/cloud-security)

- [Аналитика угроз](/azure/cloud-adoption-framework/organize/cloud-security-threat-intelligence)

- [Управление соответствием требованиям безопасности](/azure/cloud-adoption-framework/organize/cloud-security-compliance-management)

- [Управление состоянием](/azure/cloud-adoption-framework/organize/cloud-security-compliance-management)

## <a name="es-3-ensure-anti-malware-software-and-signatures-are-updated"></a>ES-3: обеспечение обновления программного обеспечения и подписей для защиты от вредоносных программ

| Идентификатор Azure | ИДЕНТИФИКАТОРы элементов управления CIS v 7.1 | Директива NIST SP 800-53 R4 ID (s) |
|--|--|--|--|
| ES-3 | 8.2 | SI-2, SI-3 |

Регулярное обновление подписей для защиты от вредоносных программ.

Следуйте рекомендациям в центре безопасности Azure: "вычисление & приложений", чтобы обеспечить актуальность всех конечных точек с последними сигнатурами. Антивредоносная программа Майкрософт автоматически устанавливает последние сигнатуры и обновления антивирусного по по умолчанию. Для Linux убедитесь, что подписи обновлены в решении для защиты от вредоносных программ стороннего производителя.

- [Развертывание антивредоносного по Майкрософт для облачных служб и виртуальных машин Azure](../fundamentals/antimalware.md)

**Ответственность**: Customer

**Заинтересованные лица по безопасности клиентов** (дополнительные [сведения](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)):

- [Безопасность инфраструктуры и конечных точек](/azure/cloud-adoption-framework/organize/cloud-security)

- [Аналитика угроз](/azure/cloud-adoption-framework/organize/cloud-security-threat-intelligence)

- [Управление соответствием требованиям безопасности](/azure/cloud-adoption-framework/organize/cloud-security-compliance-management)

- [Управление состоянием](/azure/cloud-adoption-framework/organize/cloud-security-compliance-management)

- [Оценка и рекомендации по защите конечных точек в центре безопасности Azure](../../security-center/security-center-endpoint-protection.md)