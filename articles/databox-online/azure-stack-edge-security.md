---
title: Безопасность Azure Stack ребра Pro | Документация Майкрософт
description: Описание функций обеспечения безопасности и конфиденциальности, защищающих ваше Azure Stack устройства, службы и данные в локальной среде и в облаке.
services: Data Box Edge
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: article
ms.date: 08/21/2019
ms.author: alkohli
ms.openlocfilehash: b5c5143c7bc92487aeabc3cdc1afea5061466027
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "90903156"
---
# <a name="azure-stack-edge-pro-security-and-data-protection"></a>Azure Stack пограничной Pro безопасность и защита данных

Безопасность — это серьезная проблема при внедрении новой технологии, особенно если технология используется с конфиденциальными или собственными данными. Azure Stack пограничной Pro позволяет убедиться, что только разрешенные сущности могут просматривать, изменять или удалять данные.

В этой статье описываются функции безопасности, Azure Stack пограничной Pro, которые защищают каждый из компонентов решения и данные, хранящиеся в них.

Azure Stack пограничным Pro состоит из четырех основных компонентов, взаимодействующих друг с другом:

- **Служба пограничной службы Azure Stack, размещенная в Azure**. Ресурс управления, используемый для создания порядка устройств, настройки устройства, а затем отследить порядок до завершения.
- **Azure Stack Device Pro**. Устройство, которое вы передаете, чтобы импортировать локальные данные в Azure.
- **Клиенты и узлы, подключенные к устройству**. Клиенты в инфраструктуре, подключающиеся к устройству Azure Stack пограничной Pro и содержащие данные, которые необходимо защитить.
- **Облачное хранилище**. Расположение на облачной платформе Azure, в которой хранятся данные. Это обычно учетная запись хранения, связанная с созданным ресурсом Azure Stack ребра.

## <a name="azure-stack-edge-service-protection"></a>Защита Azure Stack пограничной службы

Служба Azure Stack ребра — это служба управления, размещенная в Azure. Служба используется для настройки устройства и управления им.

[!INCLUDE [data-box-edge-gateway-data-rest](../../includes/data-box-edge-gateway-service-protection.md)]

## <a name="azure-stack-edge-pro-device-protection"></a>Azure Stack защита устройства на границе Pro

Устройство Azure Stack пограничной Pro — это локальное устройство, которое помогает преобразовать данные, обрабатывая их локально и отправив в Azure. Устройство:

- Требуется ключ активации для доступа к службе Azure Stack ребра.
- В любое время защищается паролем устройства.
- — Это заблокированное устройство. BMC и BIOS устройств защищаются паролем. BIOS защищена ограниченным доступом пользователей.
- Включена безопасная загрузка.
- Запускает Windows Defender Device Guard. Device Guard позволяет запускать только доверенные приложения, определенные в политиках целостности кода.

### <a name="protect-the-device-via-activation-key"></a>Защита устройства с помощью ключа активации

Только разрешенное устройство Azure Stack пограничной Pro может присоединиться к службе пограничной Azure Stack, созданной в подписке Azure. Для авторизации устройства необходимо использовать ключ активации, чтобы активировать устройство с помощью службы Azure Stack ребра.

[!INCLUDE [data-box-edge-gateway-data-rest](../../includes/data-box-edge-gateway-activation-key.md)]

Дополнительные сведения см. в разделе [Получение ключа активации](azure-stack-edge-deploy-prep.md#get-the-activation-key).

### <a name="protect-the-device-via-password"></a>Защита устройства с помощью пароля

Пароли гарантируют, что только полномочные пользователи могут получить доступ к вашим данным. Azure Stack устройства пограничной Pro загружаются в заблокированном состоянии.

Вы можете:

- Подключитесь к локальному веб-ИНТЕРФЕЙСу устройства через браузер, а затем укажите пароль для входа на устройство.
- Удаленное подключение к интерфейсу PowerShell устройства через HTTP. Удаленное управление включено по умолчанию. Затем можно указать пароль устройства для входа на устройство. Дополнительные сведения см. [в статье удаленное подключение к устройству Azure Stack ребра Pro](azure-stack-edge-connect-powershell-interface.md#connect-to-the-powershell-interface).

[!INCLUDE [data-box-edge-gateway-data-rest](../../includes/data-box-edge-gateway-password-best-practices.md)]
- Используйте локальный веб-интерфейс для [изменения пароля](azure-stack-edge-manage-access-power-connectivity-mode.md#manage-device-access). Если вы измените пароль, не забудьте уведомить всех пользователей удаленного доступа, чтобы у них не было проблем при входе в систему.

## <a name="protect-your-data"></a>Защита своих данных

В этом разделе описаны функции безопасности, Azure Stack пограничной Pro, которые защищают транзитные и хранимые данные.

### <a name="protect-data-at-rest"></a>Защита хранящихся данных

[!INCLUDE [data-box-edge-gateway-data-rest](../../includes/data-box-edge-gateway-data-rest.md)]
- Шифрование BitLocker XTS-AES 256-bit используется для защиты локальных данных.


### <a name="protect-data-in-flight"></a>Защита данных в полете

[!INCLUDE [data-box-edge-gateway-data-rest](../../includes/data-box-edge-gateway-data-flight.md)]

### <a name="protect-data-via-storage-accounts"></a>Защита данных с помощью учетных записей хранения

[!INCLUDE [data-box-edge-gateway-data-rest](../../includes/data-box-edge-gateway-protect-data-storage-accounts.md)]
- Регулярно вращайте и [синхронизируйте ключи учетной записи хранения](azure-stack-edge-manage-shares.md#sync-storage-keys) , чтобы защитить учетную запись хранения от неавторизованных пользователей.

## <a name="manage-personal-information"></a>Управление личными сведениями

Служба Azure Stack ребра собирает персональные данные в следующих сценариях:

[!INCLUDE [data-box-edge-gateway-data-rest](../../includes/data-box-edge-gateway-manage-personal-data.md)]

Чтобы просмотреть список пользователей, которые могут получить доступ к общей папке или удалить ее, выполните действия, описанные в разделе [Управление общими папками на Azure Stack пограничных Pro](azure-stack-edge-manage-shares.md).

Для получения дополнительных сведений ознакомьтесь с политикой конфиденциальности Майкрософт в [центре управления безопасностью](https://www.microsoft.com/trustcenter).

## <a name="next-steps"></a>Дальнейшие действия

[Развертывание устройства Azure Stack пограничной Pro](azure-stack-edge-deploy-prep.md)
