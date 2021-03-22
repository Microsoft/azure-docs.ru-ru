---
title: Преимущества и функции Azure Defender для Key Vault
description: Сведения о преимуществах и функциях Azure Defender для Key Vault.
author: memildin
ms.author: memildin
ms.date: 9/22/2020
ms.topic: overview
ms.service: security-center
ms.custom: references_regions
manager: rkarlin
ms.openlocfilehash: f127a24bec5567a3868ae77cb8a52f0af2a19603
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102100684"
---
# <a name="introduction-to-azure-defender-for-key-vault"></a>Общие сведения об Azure Defender для Key Vault

Azure Key Vault — это облачная служба, которая обеспечивает защиту ключей шифрования и секретов (например, сертификатов, строк подключения и паролей). 

Включите **Azure Defender для Key Vault** для реализации расширенной системы защиты от угроз для Azure Key Vault, предоставляющей дополнительный уровень аналитики безопасности. 

## <a name="availability"></a>Доступность

|Аспект|Сведения|
|----|:----|
|Состояние выпуска:|Общедоступная версия|
|Цены|Плата за использование **Azure Defender для Key Vault** начисляется по тарифам, приведенным на странице [Цены на Центр безопасности](https://azure.microsoft.com/pricing/details/security-center/).|
|Облако.|![Да](./media/icons/yes-icon.png) Коммерческие облака<br>![Нет](./media/icons/no-icon.png) Национальные и независимые (US Gov, China Gov, другие правительственные облака).|
|||

## <a name="what-are-the-benefits-of-azure-defender-for-key-vault"></a>Каковы преимущества Azure Defender для Key Vault?

Azure Defender выявляет необычные и потенциально опасные попытки получить доступ к учетным записям Key Vault или воспользоваться ими. Этот уровень защиты позволяет устранять угрозы даже при отсутствии экспертных знаний по безопасности и избавляет от необходимости управлять сторонними системами мониторинга безопасности.  

В случае аномальных действий Azure Defender отображает оповещения и при необходимости отправляет их по электронной почте соответствующим членам вашей организации. Эти оповещения содержат сведения о подозрительных действиях и рекомендации о том, как определить причину угроз и устранить сами угрозы. 

## <a name="azure-defender-for-key-vault-alerts"></a>Оповещения Azure Defender для Key Vault
Если вы получите оповещение от Azure Defender для Key Vault, мы рекомендуем проанализировать его и отреагировать на это оповещение, как описано в статье [Реагирование на оповещения Azure Defender для Key Vault](defender-for-key-vault-usage.md). Azure Defender для Key Vault защищает приложения и учетные данные, но даже если вы знакомы с приложением или пользователем, активировавшим это оповещение, изучите причины возникновения для каждого оповещения.

Оповещения отображаются на странице **Безопасность** в Key Vault, на панели мониторинга Azure Defender и на странице оповещений Центра безопасности.

:::image type="content" source="./media/defender-for-key-vault-intro/key-vault-security-page.png" alt-text="Страница &quot;Безопасность&quot; в Azure Key Vault":::


> [!TIP]
> Вы можете имитировать оповещения Azure Defender для Key Vault, выполнив инструкции из записи блога [Проверка обнаружения угроз Azure Key Vault в Центре безопасности Azure](https://techcommunity.microsoft.com/t5/azure-security-center/validating-azure-key-vault-threat-detection-in-azure-security/ba-p/1220336).


## <a name="next-steps"></a>Дальнейшие действия

Из этой статьи вы узнали об Azure Defender для Key Vault.

Связанные материалы см. в следующих статьях: 

- [Оповещения системы безопасности Key Vault](alerts-reference.md#alerts-azurekv) — раздел, в котором приведена справочная таблица со всеми оповещениями о Key Vault от Центра безопасности Azure.
- [Непрерывный экспорт данных Центра безопасности](continuous-export.md)
- [Отключение оповещений от Azure Defender](alerts-suppression-rules.md)