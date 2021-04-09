---
title: Создание и слияние CSR в Azure Key Vault
description: Узнайте, как выполнить создание и слияние CSR в Azure Key Vault.
services: key-vault
author: msmbaldwin
manager: rkarlin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: certificates
ms.topic: tutorial
ms.date: 06/17/2020
ms.author: sebansal
ms.openlocfilehash: aa631f4c505200c2c8abc67d4e22ffbab23e015c
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98789032"
---
# <a name="create-and-merge-a-csr-in-key-vault"></a>Создание и слияние CSR в Key Vault

Azure Key Vault поддерживает хранение цифровых сертификатов, выданных любым центром сертификации (ЦС). Он поддерживает создание запроса на подписывание сертификата (CSR) с помощью пары закрытого и открытого ключей. CSR может быть подписан любым ЦС (внутренним ЦС предприятия или внешним общедоступным ЦС). CSR — это сообщение, которое отправляется в центр сертификации, чтобы запросить цифровой сертификат.

Общую информацию о сертификатах см. в статье [Сведения о сертификатах Azure Key Vault](./about-certificates.md).

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="add-certificates-in-key-vault-issued-by-partnered-cas"></a>Добавление в Key Vault сертификата, выданного ЦС со статусом партнера

Key Vault пользуется услугами следующих центров сертификации для упрощения процесса создания сертификатов.

|Поставщик|Тип сертификата|Настройка конфигурации  
|--------------|----------------------|------------------|  
|DigiCert;|Key Vault предоставляет OV или EV SSL-сертификаты в DigiCert| [Руководство по интеграции](./how-to-integrate-certificate-authority.md)
|GlobalSign;|Key Vault предоставляет OV или EV SSL-сертификаты в GlobalSign| [Руководство по интеграции](https://support.globalsign.com/digital-certificates/digital-certificate-installation/generating-and-importing-certificate-microsoft-azure-key-vault)

## <a name="add-certificates-in-key-vault-issued-by-non-partnered-cas"></a>Добавление в Key Vault сертификата, выданного ЦС без статуса партнера

Выполните приведенные далее действия, чтобы добавить сертификат из ЦС без статуса партнера Key Vault (например, GoDaddy не является доверенным ЦС для Key Vault).

# <a name="portal"></a>[Портал](#tab/azure-portal)

1. Перейдите в хранилище ключей, в которое необходимо добавить сертификат.
1. На странице свойств выберите **Сертификаты**.
1. Откройте вкладку **Создать или импортировать**.
1. На экране **Создание сертификата** выберите следующие значения:
    - **Метод создания сертификата**: Сформировать.
    - **Имя сертификата**: ContosoManualCSRCertificate.
    - **Тип центра сертификации (ЦС):** сертификат, выданный неинтегрированным ЦС.
    - **Тема:** `"CN=www.contosoHRApp.com"`.
     > [!NOTE]
     > Если вы используете относительное различающееся имя (RDN), содержащее запятую (,) в значении, заключите значение, содержащее специальный символ, в двойные кавычки. 
     >
     > Пример записи в поле **Тема**: `DC=Contoso,OU="Docs,Contoso",CN=www.contosoHRApp.com`.
     >
     > В этом примере RDN `OU` содержит значение с запятой в имени. Выходными данными для `OU` является **Docs, Contoso**.
1. При необходимости выберите другие значения, а затем щелкните **Создать**, чтобы добавить сертификат в список **Сертификаты**.

    ![Снимок экрана: свойства сертификата](../media/certificates/create-csr-merge-csr/create-certificate.png)  

1. В списке **Сертификаты** выберите новый сертификат. Текущее состояние сертификата — **отключено**, так как он еще не выдан ЦС.
1. На вкладке **Операции с сертификатом** выберите **Скачать CSR**.

   ![Снимок экрана: выделенная кнопка "Скачать CSR".](../media/certificates/create-csr-merge-csr/download-csr.png)

1. Отправьте в ЦС запрос на подписывание файла CSR (.csr).
1. После подписания запроса выберите **Выполнить слияние подписанного запроса** на вкладке **Операции с сертификатом**, чтобы добавить подписанный сертификат в Key Vault.

Теперь для запроса на сертификат успешно выполнено слияние.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Создайте политику сертификата. Так как у ЦС, выбранного в этом сценарии, нет статуса партнера, поле **Имя издателя** имеет значение **Неизвестно**, а Key Vault не регистрирует и не обновляет сертификат.

   ```azure-powershell
   $policy = New-AzKeyVaultCertificatePolicy -SubjectName "CN=www.contosoHRApp.com" -ValidityInMonths 1  -IssuerName Unknown
   ```
     > [!NOTE]
     > Если вы используете относительное различающееся имя (RDN), содержащее запятую (,) в значении, используйте одинарные кавычки для полного значения или набора значений. Кроме того, заключите значение, содержащее специальный символ, в двойные кавычки. 
     >
     >Пример записи в поле **Название темы**: `$policy = New-AzKeyVaultCertificatePolicy -SubjectName 'OU="Docs,Contoso",DC=Contoso,CN=www.contosoHRApp.com' -ValidityInMonths 1  -IssuerName Unknown`. В этом примере значение `OU` можно растолковать как **Docs, Contoso**. Этот формат используется для всех значений, содержащих запятую.
     > 
     > В этом примере RDN `OU` содержит значение с запятой в имени. Выходными данными для `OU` является **Docs, Contoso**.

1. Создайте CSR.

   ```azure-powershell
   $csr = Add-AzKeyVaultCertificate -VaultName ContosoKV -Name ContosoManualCSRCertificate -CertificatePolicy $policy
   $csr.CertificateSigningRequest
   ```

1. Отправьте в ЦС запрос на подписывание файла CSR. `$csr.CertificateSigningRequest` — CSR для сертификата в кодировке Base64. Вы можете передать этот большой двоичный объект на веб-сайт издателя, принимающий запросы на сертификаты. Этот шаг зависит от ЦС. Изучите соответствующие рекомендации по использованию ЦС, чтобы узнать больше. Кроме того, вы можете использовать такие средства, как certreq или openssl, чтобы подписать CSR и завершить создание сертификата.

1. Выполните слияние подписанного запроса в Key Vault. После подписания запроса на сертификат его можно объединить с исходной парой закрытого и открытого ключей, созданной в Azure Key Vault.

    ```azure-powershell-interactive
    Import-AzKeyVaultCertificate -VaultName ContosoKV -Name ContosoManualCSRCertificate -FilePath C:\test\OutputCertificateFile.cer
    ```

Теперь для запроса на сертификат успешно выполнено слияние.

---

## <a name="add-more-information-to-the-csr"></a>Добавление дополнительных сведений в CSR-файл

Если вы хотите добавить дополнительные сведения при создании CSR, определите их в поле **Имя темы**. Возможно, вы захотите добавить следующие сведения:
- Страна или регион
- город или расположение;
- область, республика, край, округ;
- План
- Подразделение

Пример

   ```azure-powershell
   SubjectName="CN = docs.microsoft.com, OU = Microsoft Corporation, O = Microsoft Corporation, L = Redmond, S = WA, C = US"
   ```

> [!NOTE]
> Если вы запрашиваете сертификат проверки домена с дополнительными сведениями, ЦС может отклонить запрос, если проверить все сведения в запросе невозможно. Дополнительные сведения могут быть более подходящими, если вы запрашиваете сертификат проверки организации.

## <a name="faqs"></a>Часто задаваемые вопросы

- Как отслеживать CSR или управлять им?

     См. статью [Мониторинг и администрирование процесса создания сертификатов](./create-certificate-scenarios.md).

- Что делать, если возникает ошибка типа **The public key of the end-entity certificate in the specified X.509 certificate content does not match the public part of the specified private key. Please check if certificate is valid.** (Открытый ключ сертификата конечного субъекта в указанном содержимом сертификата X.509 не соответствует открытой части указанного закрытого ключа. Проверьте, действителен ли сертификат.)?

     Эта ошибка возникает, если не выполняется слияние подписанного CSR с тем же инициированным запросом CSR. Каждый создаваемый CSR имеет закрытый ключ, который должен соответствовать ключу запроса при слиянии подписанного запроса.

- При слиянии CSR объединяется ли вся цепочка?

     Да, вся цепочка будет объединена, если пользователь предоставит файл .p7b для слияния.

- Что делать, если сертификат выдается в отключенном состоянии на портале Azure?

     Просмотрите сообщение об ошибке для такого сертификата на вкладке **Операции с сертификатом**.

- Что делать, если возникает ошибка типа **The subject name provided is not a valid X500 name.** (Указанное имя субъекта не является допустимым именем X500.)?

     Эта ошибка может возникать, если поле **Название темы** содержит специальные знаки. Дополнительные сведения см. на портале Azure и в инструкциях PowerShell.

---

## <a name="next-steps"></a>Дальнейшие действия

- [Аутентификация, запросы и ответы](../general/authentication-requests-and-responses.md)
- [Руководство разработчика Azure Key Vault](../general/developers-guide.md)
- [Справочник по REST API для Azure Key Vault](/rest/api/keyvault)
- [Хранилища: создание или обновление](/rest/api/keyvault/vaults/createorupdate)
- [Хранилища — политика доступа для обновления](/rest/api/keyvault/vaults/updateaccesspolicy)