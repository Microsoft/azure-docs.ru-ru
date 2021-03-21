---
title: Функции Центра безопасности Azure в зависимости от ОС, типа компьютера и облака
description: Узнайте, какие функции Центра безопасности Azure доступны в зависимости от ОС, типа компьютера и облачного развертывания.
author: memildin
manager: rkarlin
ms.service: security-center
ms.topic: overview
ms.date: 02/16/2021
ms.author: memildin
ms.openlocfilehash: 070f35dc3efea4a3f05ec36cc3860fa589aa87e3
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102437916"
---
# <a name="feature-coverage-for-machines"></a>Поддерживаемые функции для виртуальных машин

На двух вкладках ниже перечислены функции Центра безопасности Azure, доступные для виртуальных машин и серверов Windows и Linux.

## <a name="supported-features-for-virtual-machines-and-servers"></a>Поддерживаемые функции для виртуальных машин и серверов <a name="vm-server-features"></a>

### <a name="windows-machines"></a>[**Компьютеры Windows**](#tab/features-windows)

|**Возможность**|**Виртуальные машины Azure**|**Масштабируемые наборы виртуальных машин Azure**|**Компьютеры с поддержкой Azure Arc**|**Требуется ли Azure Defender**
|----|:----:|:----:|:----:|:----:|
|[Интеграция Microsoft Defender для конечной точки](security-center-wdatp.md)|✔</br>(в поддерживаемых версиях)|✔</br>(в поддерживаемых версиях)|✔|Да|
|[Аналитика поведения виртуальных машин (и оповещения системы безопасности)](alerts-reference.md)|✔|✔|✔|Да|
|[Оповещения системы безопасности о бесфайловых угрозах](alerts-reference.md#alerts-windows)|✔|✔|✔|Да|
|[Оповещения системы безопасности об угрозах на уровне сети](other-threat-protections.md#network-layer)|✔|✔|-|Да|
|[JIT-доступ к виртуальной машине](security-center-just-in-time.md)|✔|-|-|Да|
|[Нативная оценка уязвимостей](deploy-vulnerability-assessment-vm.md#overview-of-the-integrated-vulnerability-scanner)|✔|-|✔|Да|
|[Мониторинг целостности файлов](security-center-file-integrity-monitoring.md)|✔|✔|✔|Да|
|[Адаптивные элементы управления приложениями](security-center-adaptive-application.md)|✔|-|✔|Да|
|[Карта сети](security-center-network-recommendations.md#network-map)|✔|✔|-|Да|
|[Адаптивная защита сети](security-center-adaptive-network-hardening.md)|✔|-|-|Да|
|[Соответствие нормативным требованиям: панель мониторинга и отчеты](security-center-compliance-dashboard.md)|✔|✔|✔|Да|
|Рекомендации и защита от угроз для IaaS-контейнеров Docker|-|-|-|Да|
|Оценка на предмет отсутствующих исправлений ОС|✔|✔|✔|Azure Нет<br><br>Поддержка Arc: Да|
|Оценка на предмет неправильных настроек системы безопасности|✔|✔|✔|Azure Нет<br><br>Поддержка Arc: Да|
|[Оценка защиты конечных точек](security-center-services.md#supported-endpoint-protection-solutions-)|✔|✔|✔|Azure Нет<br><br>Поддержка Arc: Да|
|Оценка шифрования дисков|✔</br>(в [поддерживаемых сценариях](../virtual-machines/windows/disk-encryption-windows.md#unsupported-scenarios))|✔|-|Нет|
|Оценка уязвимостей сторонних решений|✔|-|✔|Нет|
|[Оценка безопасности сети](security-center-network-recommendations.md)|✔|✔|-|Нет|


### <a name="linux-machines"></a>[**Компьютеры Linux**](#tab/features-linux)

|**Возможность**|**Виртуальные машины Azure**|**Масштабируемые наборы виртуальных машин Azure**|**Компьютеры с поддержкой Azure Arc**|**Требуется ли Azure Defender**
|----|:----:|:----:|:----:|:----:|
|[Интеграция Microsoft Defender для конечной точки](security-center-wdatp.md)|-|-|-|-|
|[Аналитика поведения виртуальных машин (и оповещения системы безопасности)](./azure-defender.md)|✔</br>(в поддерживаемых версиях)|✔</br>(в поддерживаемых версиях)|✔|Да|
|[Оповещения системы безопасности о бесфайловых угрозах](alerts-reference.md#alerts-windows)|-|-|-|Да|
|[Оповещения системы безопасности об угрозах на уровне сети](other-threat-protections.md#network-layer)|✔|✔|-|Да|
|[JIT-доступ к виртуальной машине](security-center-just-in-time.md)|✔|-|-|Да|
|[Нативная оценка уязвимостей](deploy-vulnerability-assessment-vm.md#overview-of-the-integrated-vulnerability-scanner)|✔|-|✔|Да|
|[Мониторинг целостности файлов](security-center-file-integrity-monitoring.md)|✔|✔|✔|Да|
|[Адаптивные элементы управления приложениями](security-center-adaptive-application.md)|✔|-|✔|Да|
|[Карта сети](security-center-network-recommendations.md#network-map)|✔|✔|-|Да|
|[Адаптивная защита сети](security-center-adaptive-network-hardening.md)|✔|-|-|Да|
|[Соответствие нормативным требованиям: панель мониторинга и отчеты](security-center-compliance-dashboard.md)|✔|✔|✔|Да|
|Рекомендации и защита от угроз для IaaS-контейнеров Docker|✔|✔|✔|Да|
|Оценка на предмет отсутствующих исправлений ОС|✔|✔|✔|Azure Нет<br><br>Поддержка Arc: Да|
|Оценка на предмет неправильных настроек системы безопасности|✔|✔|✔|Azure Нет<br><br>Поддержка Arc: Да|
|[Оценка защиты конечных точек](security-center-services.md#supported-endpoint-protection-solutions-)|-|-|-|Нет|
|Оценка шифрования дисков|✔</br>(в [поддерживаемых сценариях](../virtual-machines/windows/disk-encryption-windows.md#unsupported-scenarios))|✔|-|Нет|
|Оценка уязвимостей сторонних решений|✔|-|✔|Нет|
|[Оценка безопасности сети](security-center-network-recommendations.md)|✔|✔|-|Нет|

--- 


> [!TIP]
>Чтобы проверить работу функций, доступных только с Azure Defender, вы можете зарегистрироваться для получения 30-дневной бесплатной пробной версии этой службы. Дополнительные сведения см. на [странице с расценками](https://azure.microsoft.com/pricing/details/security-center/).


## <a name="supported-endpoint-protection-solutions"></a>Поддерживаемые решения для защиты конечных точек <a name="endpoint-supported"></a>

В таблице ниже приведена матрица условий:

 - Можно ли использовать Центр безопасности Azure для установки отдельных решений.
 - Какие решения по защите конечных точек может обнаружить Центр безопасности. Если Центр безопасности обнаружит решение, указанное в списке ниже, не следует устанавливать его.

Сведения о том, на основе чего создаются рекомендации по установке каждого из этих решений, см. в статье об [оценке и рекомендациях, связанных с защитой конечных точек](security-center-endpoint-protection.md).

| Endpoint Protection| Платформы | Установка центра безопасности | Обнаружение центра безопасности |
|------|------|-----|-----|
| Антивирусная программа Microsoft Defender| Windows Server 2016 или более поздней версии.| Нет, встроен в ОС| Да |
| System Center Endpoint Protection (антивредоносное ПО Майкрософт) | Windows Server 2012 R2, 2012, 2008 R2 (см. примечание ниже) | Через расширение | Да |
| Trend Micro — агент Deep Security | Семейство Windows Server  | Нет | Да |
| Symantec v12.1.1100+| Семейство Windows Server  | Нет | Да |
| McAfee v10+ | Семейство Windows Server  | Нет | Да |
| McAfee v10+ | Семейство Linux Server  | Нет | Да |
| Sophos, начиная с версии 9| Семейство Linux Server  | Нет | Да |

> [!NOTE]
> Чтобы служба System Center Endpoint Protection (SCEP) была доступна для обнаружения на виртуальной машине Windows Server 2008 R2, службу нужно установить после установки PowerShell 3.0 или более поздней версии.



## <a name="feature-support-in-government-clouds"></a>Поддержка компонентов и функций в облаках для государственных организаций

| Служба, компонент или функция | US Gov | China Gov |
|------|:----:|:----:|
|[JIT-доступ к виртуальной машине](security-center-just-in-time.md) (1)|✔|✔|
|[Мониторинг целостности файлов](security-center-file-integrity-monitoring.md) (1)|✔|✔|
|[Адаптивные элементы управления приложениями](security-center-adaptive-application.md) (1)|✔|✔|
|[Адаптивная защита сети](security-center-adaptive-network-hardening.md) (1)|-|-|
|[Защита узлов Docker](harden-docker-hosts.md) (1)|✔|✔|
|[Интегрированная оценка уязвимостей для компьютеров](deploy-vulnerability-assessment-vm.md) (1)|-|-|
|[Microsoft Defender для конечных точек](harden-docker-hosts.md) (1)|✔|-|
|[Подключение учетной записи AWS](quickstart-onboard-aws.md) (1)|-|-|
|[Подключение учетной записи GCP](quickstart-onboard-gcp.md) (1)|-|-|
|[непрерывный экспорт.](continuous-export.md)|✔|✔|
|[Автоматизация рабочих процессов](workflow-automation.md)|✔|✔|
|[Правила исключения рекомендаций](exempt-resource.md)|-|-|
|[Правила подавления оповещений](alerts-suppression-rules.md)|✔|✔|
|[Отправки по электронной почте уведомлений об оповещениях системы безопасности](security-center-provide-security-contact-details.md)|✔|✔|
|[Инвентаризация ресурсов](asset-inventory.md)|✔|✔|
|[Azure Defender для Службы приложений](defender-for-app-service-introduction.md)|-|-|
|[Azure Defender для службы хранилища](defender-for-storage-introduction.md)|✔|-|
|[Azure Defender для SQL](defender-for-sql-introduction.md)|✔|✔ (2)|
|[Azure Defender для Key Vault](defender-for-key-vault-introduction.md)|-|-|
|[Azure Defender для Resource Manager](defender-for-resource-manager-introduction.md)|-|-|
|[Azure Defender для DNS](defender-for-dns-introduction.md)|-|-|
|[Azure Defender для реестров контейнеров](defender-for-container-registries-introduction.md)|✔ (2)|✔ (2)|
|[Azure Defender для Kubernetes](defender-for-kubernetes-introduction.md)|✔|✔|
|[Защита рабочих нагрузок Kubernetes](kubernetes-workload-protections.md)|✔|✔|
|||

(1) Требуется **Azure Defender для серверов**.

(2) Частично.


## <a name="next-steps"></a>Дальнейшие действия

- Узнайте, как [в Центре безопасности собираются данные с помощью агента Log Analytics](security-center-enable-data-collection.md).
- Узнайте, как [Центр безопасности управляет данными и защищает их](security-center-data-security.md).
- Просмотрите [платформы, которые поддерживают Центр безопасности](security-center-os-coverage.md).