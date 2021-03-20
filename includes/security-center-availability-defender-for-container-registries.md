---
author: memildin
ms.author: memildin
manager: rkarlin
ms.date: 11/22/2020
ms.topic: include
ms.openlocfilehash: 2112cde42ee00b78a82962ee46f53110068977c0
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98187099"
---
## <a name="availability"></a>Доступность

|Аспект|Сведения|
|----|:----|
|Состояние выпуска:|Общедоступная версия (GA)|
|Цены|Плата за использование **Azure Defender для реестров контейнеров** начисляется по тарифам, приведенным на [странице с ценами](../articles/security-center/security-center-pricing.md).|
|Поддерживаемые реестры и образы:|Образы Linux в реестрах ACR, доступные из общедоступного Интернета через оболочку.|
|Неподдерживаемые реестры и образы:|Образы Windows<br>"Частные" реестры.<br>Реестры с доступом, ограниченным брандмауэром, конечной точкой службы или частными конечными точками, например Приватным каналом Azure.<br>Сверхминималистические образы, например [временные образы Docker](https://hub.docker.com/_/scratch/) или "бездистрибутивные" образы, которые содержат только приложение и его зависимости для среды выполнения без диспетчера пакетов, оболочки или ОС.|
|Требуемые роли и разрешения|**Читатель сведений о безопасности**, а также [роли и разрешения Реестра контейнеров Azure](../articles/container-registry/container-registry-roles.md).|
|Облако.|:::image type="icon" source="../articles/security-center/media/icons/yes-icon.png" border="false"::: Коммерческие облака<br>:::image type="icon" source="../articles/security-center/media/icons/yes-icon.png" border="false"::: US Gov и China Gov — в настоящее время поддерживается только функция проверки или принудительной отправки. Подробные сведения см. в статье [Когда проверяются образы?](../articles/security-center/defender-for-container-registries-introduction.md#when-are-images-scanned)|
|||