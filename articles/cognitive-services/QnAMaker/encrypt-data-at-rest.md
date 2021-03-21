---
title: QnA Maker шифрование неактивных данных
titleSuffix: Azure Cognitive Services
description: Корпорация Майкрософт предлагает ключи шифрования, управляемые корпорацией Майкрософт, а также позволяет управлять Cognitive Services подписками с помощью собственных ключей, называемых управляемыми клиентом ключами (CMK). В этой статье описывается шифрование неактивных данных для QnA Maker и включение и управление CMK.
author: erindormier
manager: venkyv
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: conceptual
ms.date: 11/09/2020
ms.author: egeaney
ms.openlocfilehash: 19dc0f3a676d5373b28e4b7055050477c426f847
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100524532"
---
# <a name="qna-maker-encryption-of-data-at-rest"></a>QnA Maker шифрование неактивных данных

QnA Maker автоматически шифрует данные при сохранении в облаке, помогая обеспечить безопасность и соответствие требованиям Организации.

## <a name="about-encryption-key-management"></a>Об управлении ключами шифрования

По умолчанию в подписке используются ключи шифрования, управляемые корпорацией Майкрософт. Кроме того, существует возможность управления подпиской с помощью собственных ключей, которые называются ключами, управляемыми клиентом (CMK). CMK обеспечивает большую гибкость при создании, повороте, отключении и отмене контроля доступа. Они также дают возможность выполнять аудит ключей шифрования, используемых для защиты ваших данных. Если для вашей подписки настроен CMK, то предоставляется двойное шифрование, которое обеспечивает второй уровень защиты, позволяя управлять ключом шифрования с помощью Azure Key Vault.

# <a name="qna-maker-ga-stable-release"></a>[Общедоступная версия QnA Maker (стабильный выпуск)](#tab/v1)

QnA Maker использует поддержку CMK из службы поиска Azure. Настройка [CMK в поиске Azure с помощью Azure Key Vault](../../search/search-security-manage-encryption-keys.md). Этот экземпляр Azure должен быть связан со службой QnA Maker, чтобы сделать ее CMK включенной.

# <a name="qna-maker-managed-preview-release"></a>[Управляемая служба QnA Maker (предварительный выпуск)](#tab/v2)

QnA Maker использует [поддержку CMK из службы поиска Azure](../../search/search-security-manage-encryption-keys.md)и автоматически связывает предоставленный CMK для шифрования данных, хранящихся в индексе службы поиска Azure.

---

> [!IMPORTANT]
> Ресурс службы поиска Azure должен быть создан после января 2019 и не может быть в бесплатном (общем) уровне. В портал Azure не поддерживается настройка ключей, управляемых клиентом.

## <a name="enable-customer-managed-keys"></a>Активация управляемых клиентом ключей

Служба QnA Maker использует CMK из службы поиска Azure. Чтобы включить ключей CMK, выполните следующие действия.

# <a name="qna-maker-ga-stable-release"></a>[Общедоступная версия QnA Maker (стабильный выпуск)](#tab/v1)

1. Создайте новый экземпляр службы поиска Azure и включите необходимые компоненты, указанные в [разделе Предварительные требования к управляемым клиентом ключам для Azure когнитивный Поиск](../../search/search-security-manage-encryption-keys.md#prerequisites).

   ![Просмотреть параметры шифрования 1](../media/cognitive-services-encryption/qna-encryption-1.png)

2. При создании QnA Maker ресурса он автоматически связывается с экземпляром службы поиска Azure. Этот экземпляр нельзя использовать с CMK. Чтобы использовать CMK, необходимо связать только что созданный экземпляр поиска Azure, созданный на шаге 1. В частности, необходимо обновить `AzureSearchAdminKey` и `AzureSearchName` в ресурсе QnA Maker.

   ![Просмотреть параметры шифрования 2](../media/cognitive-services-encryption/qna-encryption-2.png)

3. Затем создайте новый параметр приложения:
   * **Имя**: задайте значение. `CustomerManagedEncryptionKeyUrl`
   * **Значение**: используйте значение, полученное на шаге 1 при создании экземпляра службы поиска Azure.

   ![Просмотреть параметры шифрования 3](../media/cognitive-services-encryption/qna-encryption-3.png)

4. По завершении перезапустите среду выполнения. Теперь служба QnA Maker включена CMK.

# <a name="qna-maker-managed-preview-release"></a>[Управляемая служба QnA Maker (предварительный выпуск)](#tab/v2)

1.  Перейдите на вкладку **Шифрование** службы QnA Maker Managed (Предварительная версия).
2.  Выберите параметр " **управляемые ключи клиента** ". Укажите сведения о [ключах, управляемых клиентом](../../storage/common/customer-managed-keys-configure-key-vault.md?tabs=portal) , и щелкните **сохранить**.

     :::image type="content" source="../media/cognitive-services-encryption/qnamaker-v2-encryption-cmk.png" alt-text="Параметр CMK управляемого (Предварительная версия) QnA Maker" lightbox="../media/cognitive-services-encryption/qnamaker-v2-encryption-cmk.png":::

3.  При успешном сохранении CMK будет использоваться для шифрования данных, хранящихся в индексе службы поиска Azure.

> [!IMPORTANT]
> Перед созданием баз знаний рекомендуется задать CMK в новой службе Azure Когнитивный поиск. Если задать CMK в службе QnA Maker с существующими базами знаний, доступ к ним может быть потерян. Узнайте больше о [работе с зашифрованным содержимым](../../search/search-security-manage-encryption-keys.md#work-with-encrypted-content) в службе поиска в Azure.

> [!NOTE]
> Чтобы запросить возможность использования ключей, управляемых клиентом, заполните и отправьте [Cognitive Services Customer-Managed форму запроса ключа](https://aka.ms/cogsvc-cmk).

---

## <a name="regional-availability"></a>Доступность в регионах

Ключи, управляемые клиентом, доступны во всех регионах поиска Azure.

## <a name="encryption-of-data-in-transit"></a>Шифрование передаваемых данных

QnA Maker портал работает в браузере пользователя. Каждое действие активирует прямой вызов соответствующего API-интерфейса службы. Таким образом, QnA Maker соответствует данным при передаче.
Однако, поскольку служба QnA Maker Portal размещена в западной части США, она по-прежнему не идеально подходит для клиентов, не являющихся клиентами США. 

## <a name="next-steps"></a>Дальнейшие действия

* [Шифрование в поиске Azure с помощью ключей CMK в Azure Key Vault](../../search/search-security-manage-encryption-keys.md)
* [Шифрование неактивных данных](../../security/fundamentals/encryption-atrest.md)
* [Дополнительные сведения об Azure Key Vault](../../key-vault/general/overview.md)