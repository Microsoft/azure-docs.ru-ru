---
title: Общие сведения о безопасности Azure Percept
description: Дополнительные сведения о безопасности Azure Перцепт
author: elqu20
ms.author: v-elqu
ms.service: azure-percept
ms.topic: conceptual
ms.date: 02/18/2021
ms.custom: template-concept
ms.openlocfilehash: a08876cde9fac64c3a361b469049b4e33678a86f
ms.sourcegitcommit: 4b7a53cca4197db8166874831b9f93f716e38e30
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102098151"
---
# <a name="azure-percept-security-overview"></a>Общие сведения о безопасности Azure Percept

Устройства Azure Перцепт DK разработаны с корнем оборудования доверия: дополнительные встроенные средства безопасности на каждом устройстве. Он помогает защитить конфиденциальные от конфиденциальности датчики, такие как камеры и микрофоны, данные вывода, а также обеспечивает проверку подлинности и авторизацию устройств для служб Azure Перцепт Studio.

> [!NOTE]
> Azure Перцепт DK лицензируется для использования только в средах разработки и тестирования.

## <a name="devices"></a>Устройства

### <a name="azure-percept-dk"></a>Azure Percept DK

Azure Перцепт DK включает доверенный платформенный модуль (TPM) (TPM) версии 2,0, которую можно использовать для подключения устройства к службам подготовки устройств Azure с дополнительной безопасностью. TPM — это общедоступный стандарт ISO из организация TCG, и вы можете ознакомиться с дополнительными сведениями о TPM по [полной спецификации tpm 2,0](https://trustedcomputinggroup.org/resource/tpm-library-specification/) или спецификации ISO/IEC 11889. Дополнительные сведения о том, как служба DPS может безопасно подготавливать устройства, см. в статье Подготовка устройств к добавлению в [центр Интернета вещей Azure — аттестация TPM](https://docs.microsoft.com/azure/iot-dps/concepts-tpm-attestation).

### <a name="azure-percept-system-on-module-som"></a>Система Azure Перцепт в модуле (SOM)

Для защиты доступа к внедренным датчикам AI в Azure Перцепт с включенной концепцией в модуле (SOM) и в наборе SOM Azure Перцепт Audio можно включить модуль Micro Controller (МИКРОКОНТРОЛЛЕРАМИ). При каждой загрузке встроенное по МИКРОКОНТРОЛЛЕРАМИ проверяет подлинность и авторизует средство AI Accelerator с помощью служб Azure Перцепт Studio, используя архитектуру механизма композиции идентификаторов устройств (КОСТей). КОСТИ работают, нарушая загрузку в слои и создавая секреты, уникальные для каждого слоя и конфигурации на основе уникального секрета устройства (доменов обновления). Если в любой точке цепочки загружается другой код или конфигурация, секреты будут отличаться. Дополнительные сведения о КОСТях см. в [спецификации "кости для рабочих групп](https://trustedcomputinggroup.org/work-groups/dice-architectures/)". Сведения о настройке доступа к Azure Перцепт Studio и требуемым службам см. в разделе **Настройка брандмауэров для Azure ПЕРЦЕПТ DK** ниже.

Устройства Перцепт Azure используют корневое доверие оборудования для защиты встроенного по. Загрузочный диск гарантирует целостность микропрограммы между ПЗУ и загрузчиком операционной системы, что, в свою очередь, обеспечит целостность других программных компонентов, создающих цепочку доверия.

## <a name="services"></a>Службы

### <a name="iot-edge"></a>IoT Edge

Azure Перцепт DK подключается к Azure Перцепт Studio с дополнительной безопасностью и другими службами Azure, использующими протокол TLS. Azure Перцепт DK — это устройство с поддержкой Azure IoT Edge. IoT Edge среда выполнения — это набор программ, которые включают устройство в устройство IoT Edge. В совокупности компоненты среды выполнения IoT Edge позволяют устройствам IoT Edge получать код для выполнения на границе и передавать результаты. Azure Перцепт DK использует контейнеры DOCKER для изоляции рабочих нагрузок IoT Edge от операционной системы узла и приложений с поддержкой пограничных устройств. Дополнительные сведения о Azure IoT Edgeной платформе безопасности см. в статье [Диспетчер безопасности IOT Edge](https://docs.microsoft.com/azure/iot-edge/iot-edge-security-manager?view=iotedge-2018-06).

### <a name="device-update-for-iot-hub"></a>Обновление устройства для центра Интернета вещей

Обновление устройства для центра Интернета вещей обеспечивает более безопасное, масштабируемое и надежное высокопроизводительное обновление, которое приводит к устанавливать возобновляемую безопасности на устройствах Azure Перцепт. Она предоставляет широкие возможности управления и обновления соответствия с помощью аналитических сведений. Azure Перцепт DK включает в себя предварительно интегрированное решение для обновления устройств, обеспечивающее устойчивое обновление (A/B) от встроенного по к уровням операционной системы.

<!---I think the below topics need to be somewhere else, (i.e. not on the main page)
--->

## <a name="configuring-firewalls-for-azure-percept-dk"></a>Настройка брандмауэров для Azure Перцепт DK

Если для настройки сети требуется явное разрешение подключений с устройств Azure Перцепт DK, ознакомьтесь со следующим списком компонентов.

Этот контрольный список является отправной точкой для настройки правил брандмауэра:

|URL-адрес (* = подстановочный знак) |Исходящие порты TCP|    Использование|
|-------------------|------------------|---------|
|*. auth.azureperceptdk.azure.net|   443|    Проверка подлинности и авторизация SOM в Azure DK|
|*. auth.projectsantacruz.azure.net| 443|    Проверка подлинности и авторизация SOM в Azure DK|

Кроме того, проверьте список [соединений, используемых Azure IOT Edge](https://docs.microsoft.com/azure/iot-edge/production-checklist?view=iotedge-2018-06#allow-connections-from-iot-edge-devices).

<!---
## Additional Recommendations for Deployment to Production

Azure Percept DK offers a great variety of security capabilities out of the box. In addition to those powerful security features included in the current release, Microsoft also suggests the following guidelines when considering production deployments:

- Strong physical protection of the device itself
- Ensuring data at rest encryption is enabled
- Continuously monitoring the device posture and quickly responding to alerts
- Limiting the number of administrators who have access to the device
--->


## <a name="next-steps"></a>Дальнейшие действия

Сведения о доступных [моделях Azure ПЕРЦЕПТ AI](./overview-ai-models.md).
