---
title: Идентификация сущностей
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: include
ms.date: 03/11/2021
ms.author: aahi
ms.openlocfilehash: 352b81bf2dfeca1d7413e7cac131264d06c7b92e
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104599310"
---
### <a name="financial-account-identification"></a>Идентификация финансовой учетной записи

Эта категория сущностей включает в себя финансовую информацию и официальные формы идентификации.

#### <a name="category-aba-routing-number"></a>Категория: номер маршрутизации ABA

Эта категория содержит следующую сущность:

:::row:::
    :::column span="":::
        **Сущность**

        Номер маршрутизации ABA

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Номера транзитного маршрута ассоциации банка Америки (ABA).

        Чтобы получить эту категорию сущностей, добавьте `ABARoutingNumber` к `pii-categories` параметру. `ABARoutingNumber` также будет возвращен в ответе API, если он обнаружен.
      
    :::column-end:::
    :::column span="2":::
      **Поддерживаемые языки документов**

      `en`
      
   :::column-end:::
:::row-end:::


#### <a name="category-swift-code"></a>Категория: код SWIFT

Эта категория содержит следующую сущность:

:::row:::
    :::column span="":::
        **Сущность**

        Код SWIFT

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Коды SWIFT для сведений о инструкции по оплате.

        Чтобы получить эту категорию сущностей, добавьте `SWIFTCode` к `pii-categories` параметру. `SWIFTCode` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="2":::
      **Поддерживаемые языки документов**

      `en`, `es`, `fr`, `de`, `it`, `pt-pt`, `pt-br`, `ja`
      
   :::column-end:::
:::row-end:::

#### <a name="category-credit-card"></a>Категория: кредитная карта

Эта категория содержит следующую сущность:

:::row:::
    :::column span="":::
        **Сущность**

        Кредитная карта

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Номера кредитных карт. 

        Чтобы получить эту категорию сущностей, добавьте `CreditCardNumber` к `pii-categories` параметру. `CreditCardNumber` При обнаружении будет возвращено в ответе API.

    :::column-end:::
    :::column span="2":::
      **Поддерживаемые языки документов**

      `en`, `es`, `fr`, `de`, `it`, `pt-pt`, `pt-br`, `ja`, `zh-hans`, `ja`, `ko`
      
   :::column-end:::
:::row-end:::

#### <a name="category-international-banking-account-number-iban"></a>Категория: Международный номер банковского счета (IBAN) 

Эта категория содержит следующую сущность:

:::row:::
    :::column span="":::
        **Сущность**

        Кредитная карта

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Коды IBAN для сведений о инструкции по оплате.

        Чтобы получить эту категорию сущностей, добавьте `InternationlBankingAccountNumber` к `pii-categories` параметру. `InternationlBankingAccountNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="2":::
      **Поддерживаемые языки документов**

      `en`, `es`, `fr`, `de`, `it`, `pt-pt`, `pt-br`
      
   :::column-end:::
:::row-end:::

### <a name="government-and-countryregion-specific-identification"></a>Идентификация правительства и страны или региона

> [!NOTE]
> Следующие сущности, относящиеся к финансовой и стране, не возвращаются с помощью `domain=phi` параметра:
> * Номера паспорта
> * Налоговые коды

Следующие сущности сгруппированы и перечислены по странам:

#### <a name="argentina"></a>Аргентина

:::row:::
    :::column span="":::
        **Сущность**

        Номер государственного удостоверения (дни)

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Чтобы получить эту категорию сущностей, добавьте `ARNationalIdentityNumber` к `pii-categories` параметру. `ARNationalIdentityNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::
      **Поддерживаемые языки документов**

      `en`, `es`
      
   :::column-end:::
:::row-end:::


#### <a name="austria"></a>Австрия

:::row:::
    :::column span="":::
        **Сущность**

        Австрия Identity Card

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Чтобы получить эту категорию сущностей, добавьте `ATIdentityCard` к `pii-categories` параметру. `ATIdentityCard` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::
      **Поддерживаемые языки документов**

      `de`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Налоговый идентификационный номер Австрии

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `ATTaxIdentificationNumber` к `pii-categories` параметру. `ATTaxIdentificationNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `de`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Номер налога на добавленную стоимость (VAT)

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `ATValueAddedTaxNumber` к `pii-categories` параметру. `ATValueAddedTaxNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `de`
      
   :::column-end:::
:::row-end:::



#### <a name="australia"></a>Австралия

:::row:::
    :::column span="":::
        **Сущность**

        Номер банковского счета Австралии

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Чтобы получить эту категорию сущностей, добавьте `AUDriversLicenseNumber` к `pii-categories` параметру. `AUDriversLicenseNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::
      **Поддерживаемые языки документов**

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Австралийский бизнес-номер

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `AUBusinessNumber` к `pii-categories` параметру. `AUBusinessNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Номер компании Австралии

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `AUCompanyNumber` к `pii-categories` параметру. `AUCompanyNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Лицензия на драйвер Австралии  

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `AUDriversLicense` к `pii-categories` параметру. `AUDriversLicense` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Номер медицинской учетной записи Австралии

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `AUMedicalAccountNumber` к `pii-categories` параметру. `AUMedicalAccountNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Номер паспорта для Австралии

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `ATPassportNumber` к `pii-categories` параметру. `ATPassportNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Номер файла налога Австралии

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `ATTaxIdentificationNumber` к `pii-categories` параметру. `ATTaxIdentificationNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::


#### <a name="belgium"></a>Бельгия

:::row:::
    :::column span="":::
        **Сущность**

        Бельгийский национальный номер

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Чтобы получить эту категорию сущностей, добавьте `BENationalNumber` к `pii-categories` параметру. `BENationalNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::
      **Поддерживаемые языки документов**

      `fr`, `de`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Номер налога на добавленную стоимость (VAT)

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `BEValueAddedTaxNumber` к `pii-categories` параметру. `BEValueAddedTaxNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `fr`, `de`
      
   :::column-end:::
:::row-end:::


#### <a name="brazil"></a>Бразилия 

:::row:::
    :::column span="":::
        **Сущность**

        Номер юридического лица в Бразилии (CNPJ)

        

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Чтобы получить эту категорию сущностей, добавьте `BRLegalEntityNumber` к `pii-categories` параметру. `BRLegalEntityNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::
      **Поддерживаемые языки документов**

      `pt-pt`, `pt-br`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Номер CPF в Бразилии

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `BRCPFNumber` к `pii-categories` параметру. `BRCPFNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `pt-pt`, `pt-br`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Национальная ИДЕНТИФИКАЦИОНная карта Бразилии (RG)

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `BRNationalIDRG` к `pii-categories` параметру. `BRNationalIDRG` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `pt-pt`, `pt-br`
      
   :::column-end:::
:::row-end:::

#### <a name="canada"></a>Канада

:::row:::
    :::column span="":::
        **Сущность**

        Номер банковского счета Канады

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Чтобы получить эту категорию сущностей, добавьте `CABankAccountNumber` к `pii-categories` параметру. `CABankAccountNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::
      **Поддерживаемые языки документов**

      `en`, `fr`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Номер лицензии для драйвера Канады

    :::column-end:::

    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `CADriversLicenseNumber` к `pii-categories` параметру. `CADriversLicenseNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::

    :::column span="":::

      `en`, `fr`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Номер службы работоспособности Канады

        
    :::column-end:::

    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `CAHealthServiceNumber` к `pii-categories` параметру. `CAHealthServiceNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::

    :::column span="":::

      `en`, `fr`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Номер паспорта для Канады

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `CAPassportNumber` к `pii-categories` параметру. `CAPassportNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `en`, `fr`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Личный идентификационный номер работоспособности (фин) для Канады

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `CAPersonalHealthIdentification` к `pii-categories` параметру. `CAPersonalHealthIdentification` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `en`, `fr`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Номер страхования социальных сетей (Канада)

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `CASocialInsuranceNumber` к `pii-categories` параметру. `CASocialInsuranceNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `en`, `fr`
      
   :::column-end:::
:::row-end:::

#### <a name="chile"></a>Чили 

:::row:::
    :::column span="":::
        **Сущность**

        Номер идентификационной карты для Чили

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Чтобы получить эту категорию сущностей, добавьте `CLIdentityCardNumber` к `pii-categories` параметру. `CLIdentityCardNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::
      **Поддерживаемые языки документов**

      `es`
      
   :::column-end:::
:::row-end:::

#### <a name="china"></a>Китай

:::row:::
    :::column span="":::
        **Сущность**

        Номер резидентной идентификационной карточки (КНР) Китая

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Чтобы получить эту категорию сущностей, добавьте `CNResidentIdentityCardNumber` к `pii-categories` параметру. `CNResidentIdentityCardNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::
      **Поддерживаемые языки документов**

      `zh-hans`
      
   :::column-end:::
:::row-end:::


#### <a name="european-union-eu"></a>Европейский союз (ЕС)

:::row:::
    :::column span="":::
        **Сущность**

        Номер дебетовой карты ЕС

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Чтобы получить эту категорию сущностей, добавьте `EUDebitCardNumber` к `pii-categories` параметру. `EUDebitCardNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::
      **Поддерживаемые языки документов**

      `en`, `es`, `fr`, `de`, `it`, `pt-pt` 
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Номер лицензии для драйвера ЕС

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `EUDriversLicenseNumber` к `pii-categories` параметру. `EUDriversLicenseNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `en`, `es`, `fr`, `de`, `it`, `pt-pt` 
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Координаты GPU в ЕС

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `EUGPSCoordinates` к `pii-categories` параметру. `EUGPSCoordinates` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `en`, `es`, `fr`, `de`, `it`, `pt-pt` 
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Национальный идентификационный номер ЕС

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `EUNationalIdentificationNumber` к `pii-categories` параметру. `EUNationalIdentificationNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `en`, `es`, `fr`, `de`, `it`, `pt-pt` 
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Номер паспорта ЕС

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `EUPassportNumber` к `pii-categories` параметру. `EUPassportNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `en`, `es`, `fr`, `de`, `it`, `pt-pt` 
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Номер социального страхования (SSN) ЕС или эквивалентный идентификатор

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `EUSocialSecurityNumber` к `pii-categories` параметру. `EUSocialSecurityNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `en`, `es`, `fr`, `de`, `it`, `pt-pt` 
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Идентификационный номер налогоплательщика в ЕС

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `EUTaxIdentificationNumber` к `pii-categories` параметру. `EUTaxIdentificationNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `en`, `es`, `fr`, `de`, `it`, `pt-pt` 
      
   :::column-end:::
:::row-end:::

#### <a name="france"></a>Франция

:::row:::
    :::column span="":::
        **Сущность**

        Номер лицензии для драйвера Франции

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Чтобы получить эту категорию сущностей, добавьте `FRDriversLicenseNumber` к `pii-categories` параметру. `FRDriversLicenseNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::
      **Поддерживаемые языки документов**

      `fr` 
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Номер страховки работоспособности Франции

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `FRHealthInsuranceNumber` к `pii-categories` параметру. `FRHealthInsuranceNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `fr` 
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Французская Национальная ИДЕНТИФИКАЦИОНная карта (CNI)

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `FRNationalID` к `pii-categories` параметру. `FRNationalID` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `fr` 
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Номер паспорта Франции

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `FRPassportNumber` к `pii-categories` параметру. `FRPassportNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `fr` 
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Номер социального страхования Франции (ИНСИ)

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `FRSocialSecurityNumber` к `pii-categories` параметру. `FRSocialSecurityNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `fr` 
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Налоговый идентификационный номер Франции (Нумéро SPI)

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `FRTaxIdentificationNumber` к `pii-categories` параметру. `FRTaxIdentificationNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `fr` 
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Номер налога на добавленную ценность Франции (НДС)

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `FRValueAddedTaxNumber` к `pii-categories` параметру. `FRValueAddedTaxNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `fr` 
      
   :::column-end:::
:::row-end:::

#### <a name="germany"></a>Германия

:::row:::
    :::column span="":::
        **Сущность**

        Номер лицензии драйвера для Германии

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Чтобы получить эту категорию сущностей, добавьте `DEDriversLicenseNumber` к `pii-categories` параметру. `DEDriversLicenseNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::
      **Поддерживаемые языки документов**

      `de` 
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Номер карточки удостоверения Германии

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `DEIdentityCardNumber` к `pii-categories` параметру. `DEIdentityCardNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `de`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Номер паспорта для Германии

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `DEPassportNumber` к `pii-categories` параметру. `DEPassportNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `de` 
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Идентификационный номер налогоплательщика для Германии

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `DETaxIdentificationNumber` к `pii-categories` параметру. `DETaxIdentificationNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `de`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Налоговый номер добавленного значения в Германии

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Чтобы получить эту категорию сущностей, добавьте `DEValueAddedNumber` к `pii-categories` параметру. `DEValueAddedNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `de`
      
   :::column-end:::
:::row-end:::

#### <a name="hong-kong"></a>Гонконг, САР

:::row:::
    :::column span="":::
        **Сущность**

        Номер карточки удостоверения Гонконг (ХКИД)

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Чтобы получить эту категорию сущностей, добавьте `HKIdentityCardNumber` к `pii-categories` параметру. `HKIdentityCardNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::
      **Поддерживаемые языки документов**

      `en`
      
   :::column-end:::
:::row-end:::

#### <a name="hungary"></a>Венгрия

:::row:::
    :::column span="":::
        **Сущность**

        Личный идентификационный номер венгерского

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Чтобы получить эту категорию сущностей, добавьте `HUPersonalIdentificationNumber` к `pii-categories` параметру. `HUPersonalIdentificationNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::
      **Поддерживаемые языки документов**

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Идентификационный номер венгерского налога

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `HUTaxIdentificationNumber` к `pii-categories` параметру. `HUTaxIdentificationNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Налоговый номер добавленного значения Венгрии

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `HUValueAddedNumber` к `pii-categories` параметру. `HUValueAddedNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::

#### <a name="india"></a>Индия

:::row:::
    :::column span="":::
        **Сущность**

        Номер постоянной учетной записи Индии (PAN)

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Чтобы получить эту категорию сущностей, добавьте `INPermanentAccount` к `pii-categories` параметру. `INPermanentAccount` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::
      **Поддерживаемые языки документов**

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Номер уникальной идентификации (Аадхаар) Индии

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Чтобы получить эту категорию сущностей, добавьте `INUniqueIdentificationNumber` к `pii-categories` параметру. `INUniqueIdentificationNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::


#### <a name="indonesia"></a>Индонезия

:::row:::
    :::column span="":::
        **Сущность**

        Номер карточки удостоверения Индонезия (КТП)

    :::column-end:::
    :::column span="2":::

        **Сведения**

        Чтобы получить эту категорию сущностей, добавьте `IDIdentityCardNumber` к `pii-categories` параметру. `IDIdentityCardNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::
      **Поддерживаемые языки документов**

      `en`
      
   :::column-end:::
:::row-end:::

#### <a name="ireland"></a>Ирландия

:::row:::
    :::column span="":::
        **Сущность**

        Номер личной общедоступной службы (PPS)

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Чтобы получить эту категорию сущностей, добавьте `IEPersonalPublicServiceNumber` к `pii-categories` параметру. `IEPersonalPublicServiceNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::
      **Поддерживаемые языки документов**

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::
 
        Личная общедоступная служба (PPS), номер 2

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `IEPersonalPublicServiceNumberV2` к `pii-categories` параметру. `IEPersonalPublicServiceNumberV2` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::
      **Поддерживаемые языки документов**

      `en`
      
   :::column-end:::
:::row-end:::

#### <a name="israel"></a>Израиль

:::row:::
    :::column span="":::
        **Сущность**

        Национальный идентификатор Израиля

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Чтобы получить эту категорию сущностей, добавьте `ILNationalID` к `pii-categories` параметру. `ILNationalID` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::
      **Поддерживаемые языки документов**

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Номер банковского счета Израиля

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `ILBankAccountNumber` к `pii-categories` параметру. `ILBankAccountNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::

#### <a name="italy"></a>Италия

:::row:::
    :::column span="":::
        **Сущность**

        Идентификатор лицензии драйвера Италии

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Чтобы получить эту категорию сущностей, добавьте `ITDriversLicenseNumber` к `pii-categories` параметру. `ITDriversLicenseNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::
      **Поддерживаемые языки документов**

      `it`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Финансовый код Италии

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `ITFiscalCode` к `pii-categories` параметру. `ITFiscalCode` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `it`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Налоговый номер добавленного значения Италии

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `ITValueAddedTaxNumber` к `pii-categories` параметру. `ITValueAddedTaxNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `it`
      
   :::column-end:::
:::row-end:::


#### <a name="japan"></a>Япония

:::row:::
    :::column span="":::
        **Сущность**

        Номер банковского счета для Японии

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Чтобы получить эту категорию сущностей, добавьте `JPBankAccountNumber` к `pii-categories` параметру. `JPBankAccountNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::
      **Поддерживаемые языки документов**

      `ja`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Номер лицензии для драйвера Японии

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `JPDriversLicenseNumber` к `pii-categories` параметру. `JPDriversLicenseNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `ja`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Япония "Моя цифра" (личное)

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `JPMyNumberPersonal` к `pii-categories` параметру. `JPMyNumberPersonal` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `ja`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Япония "Моя цифра" (корпоративный)

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `JPMyNumberCorporate` к `pii-categories` параметру. `JPMyNumberCorporate` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `ja`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Номер резидентной регистрации для Японии

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `ITValueAddedTaxNumber` к `pii-categories` параметру. `ITValueAddedTaxNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

     `ja`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Номер карты для Японии

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `JPResidenceCardNumber` к `pii-categories` параметру. `JPResidenceCardNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `ja`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Номер социального страхования для Японии (SIN)

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `JPSocialInsuranceNumber` к `pii-categories` параметру. `JPSocialInsuranceNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `ja`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Номер паспорта для Японии

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `JPPassportNumber` к `pii-categories` параметру. `JPPassportNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `ja`
      
   :::column-end:::
:::row-end:::

#### <a name="luxembourg"></a>Люксембург

:::row:::
    :::column span="":::
        **Сущность**

        Идентификационный номер Люксембург (естественное лицо)

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Чтобы получить эту категорию сущностей, добавьте `LUNationalIdentificationNumberNatural` к `pii-categories` параметру. `LUNationalIdentificationNumberNatural` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::
      **Поддерживаемые языки документов**

      `fr`, `de`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Идентификационный номер Люксембург (неестественные лица)

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `LUNationalIdentificationNumberNonNatural` к `pii-categories` параметру. `LUNationalIdentificationNumberNonNatural` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `fr`, `de`
      
   :::column-end:::
:::row-end:::

#### <a name="malta"></a>Мальта

:::row:::
    :::column span="":::
        **Сущность**

        Номер карты идентификации Мальта

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Чтобы получить эту категорию сущностей, добавьте `MTIdentityCardNumber` к `pii-categories` параметру. `MTIdentityCardNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::
      **Поддерживаемые языки документов**

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Идентификационный номер налогоплательщика Мальта

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `MTTaxIDNumber` к `pii-categories` параметру. `MTTaxIDNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::


#### <a name="new-zealand"></a>Новая Зеландия

:::row:::
    :::column span="":::
        **Сущность**

        Номер банковского счета Новой Зеландии

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Чтобы получить эту категорию сущностей, добавьте `NZBankAccountNumber` к `pii-categories` параметру. `NZBankAccountNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::
      **Поддерживаемые языки документов**

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Номер лицензии для драйвера Новой Зеландии

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `NZDriversLicenseNumber` к `pii-categories` параметру. `NZDriversLicenseNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Номер дохода для Новой Зеландии

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `NZInlandRevenueNumber` к `pii-categories` параметру. `NZInlandRevenueNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Министерство номера работоспособности Новой Зеландии

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `NZMinistryOfHealthNumber` к `pii-categories` параметру. `NZMinistryOfHealthNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

       Номер социального Велфареа Новой Зеландии

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `NZSocialWelfareNumber` к `pii-categories` параметру. `NZSocialWelfareNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::


#### <a name="philippines"></a>Филиппины

:::row:::
    :::column span="":::
        **Сущность**

        Идентификатор единой унифицированной многозначной идентификации Филиппины

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Чтобы получить эту категорию сущностей, добавьте `PHUnifiedMultiPurposeIDNumber` к `pii-categories` параметру. `PHUnifiedMultiPurposeIDNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::
      **Поддерживаемые языки документов**

      `en`
      
   :::column-end:::
:::row-end:::

#### <a name="portugal"></a>Португалия 

:::row:::
    :::column span="":::
        **Сущность**

        Номер карточки гражданского по Португалия

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Чтобы получить эту категорию сущностей, добавьте `PTCitizenCardNumber` к `pii-categories` параметру. `PTCitizenCardNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::
      **Поддерживаемые языки документов**

      `pt-pt`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

       Идентификационный номер налогоплательщика (Португалия)

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `PTTaxIdentificationNumber` к `pii-categories` параметру. `PTTaxIdentificationNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `pt-pt`
      
   :::column-end:::
:::row-end:::

#### <a name="singapore"></a>Сингапур

:::row:::
    :::column span="":::
        **Сущность**

        (НРИК) номер национальной регистрационной карты (Сингапур)

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Чтобы получить эту категорию сущностей, добавьте `PTTaxIdentificationNumber` к `pii-categories` параметру. `PTTaxIdentificationNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::
      **Поддерживаемые языки документов**

      `en`, `zh-hans`
      
   :::column-end:::
:::row-end:::


#### <a name="south-africa"></a>Южно-Африканская Республика

:::row:::
    :::column span="":::
        **Сущность**

        Идентификационный номер Южной Африки

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Чтобы получить эту категорию сущностей, добавьте `ZAIdentificationNumber` к `pii-categories` параметру. `ZAIdentificationNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::
      **Поддерживаемые языки документов**

      `en`
      
   :::column-end:::
:::row-end:::


#### <a name="south-korea"></a>Южная Корея

:::row:::
    :::column span="":::
        **Сущность**

        Номер резидентной регистрации Южной Кореи

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Чтобы получить эту категорию сущностей, добавьте `KRResidentRegistrationNumber` к `pii-categories` параметру. `KRResidentRegistrationNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::
      **Поддерживаемые языки документов**

      `ko`
      
   :::column-end:::
:::row-end:::

#### <a name="spain"></a>Испания

:::row:::
    :::column span="":::
        **Сущность**

        Испания дни

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Чтобы получить эту категорию сущностей, добавьте `ESDNI` к `pii-categories` параметру. `ESDNI` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::
      **Поддерживаемые языки документов**

      `es`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Номер социального страхования (SSN) Испания

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `ESSocialSecurityNumber` к `pii-categories` параметру. `ESSocialSecurityNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `es`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Налоговый идентификационный номер Испании

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `ESTaxIdentificationNumber` к `pii-categories` параметру. `ESTaxIdentificationNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `es`
      
   :::column-end:::
:::row-end:::
 
#### <a name="switzerland"></a>Швейцария

:::row:::
    :::column span="":::
        **Сущность**

        Швейцарский номер социального страхования АХВ

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Чтобы получить эту категорию сущностей, добавьте `CHSocialSecurityNumber` к `pii-categories` параметру. `CHSocialSecurityNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::
      **Поддерживаемые языки документов**

      `fr`, `de`, `it`
      
   :::column-end:::
:::row-end:::


#### <a name="taiwan"></a>Тайвань 

:::row:::
    :::column span="":::
        **Сущность**

        Тайваньская национальная ИДЕНТИФИКАЦИя

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Чтобы получить эту категорию сущностей, добавьте `TWNationalID` к `pii-categories` параметру. `TWNationalID` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::
      **Поддерживаемые языки документов**

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

       Сертификат в формате Тайваня (ARC/ТАРК)

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `TWResidentCertificate` к `pii-categories` параметру. `TWResidentCertificate` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Номер паспорта для Тайваня

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `TWPassportNumber` к `pii-categories` параметру. `TWPassportNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::

#### <a name="united-kingdom"></a>Соединенное Королевство

:::row:::
    :::column span="":::
        **Сущность**

        Великобритании Номер лицензии драйвера

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Чтобы получить эту категорию сущностей, добавьте `UKDriversLicenseNumber` к `pii-categories` параметру. `UKDriversLicenseNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::
      **Поддерживаемые языки документов**

      `en`
      
    :::column-end:::
    
:::row-end:::
:::row:::
    :::column span="":::

       Великобритании Номер рулона предпочтений избирателей

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `UKNationalInsuranceNumber` к `pii-categories` параметру. `UKNationalInsuranceNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

       Великобритании Номер национального служба работоспособности (отделения)

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `UKNationalHealthNumber` к `pii-categories` параметру. `UKNationalHealthNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

       Великобритании Номер национальной страховки (Нино)

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `UKNationalInsuranceNumber` к `pii-categories` параметру. `UKNationalInsuranceNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

       Великобритании или номер паспорта США

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `USUKPassportNumber` к `pii-categories` параметру. `USUKPassportNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

       Великобритании Уникальный номер ссылки налогоплательщика

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `UKUniqueTaxpayerNumber` к `pii-categories` параметру. `UKUniqueTaxpayerNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::


#### <a name="united-states"></a>США

:::row:::
    :::column span="":::
        **Сущность**

        Номер социального страхования США (SSN)

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Чтобы получить эту категорию сущностей, добавьте `USSocialSecurityNumber` к `pii-categories` параметру. `USSocialSecurityNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::
      **Поддерживаемые языки документов**

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

       Номер лицензии для драйвера США

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `USDriversLicenseNumber` к `pii-categories` параметру. `USDriversLicenseNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

       США или Великобритания Номер паспорта

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `USUKPassportNumber` к `pii-categories` параметру. `USUKPassportNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

       Индивидуальный идентификационный номер налогоплательщика (ИТИН) США

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `USIndividualTaxpayerIdentification` к `pii-categories` параметру. `USIndividualTaxpayerIdentification` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

       Номер агентства для принудительного применения наркотиков США (ДЕА)

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `DrugEnforcementAgencyNumber` к `pii-categories` параметру. `DrugEnforcementAgencyNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

       Номер банковского счета США

    :::column-end:::
    :::column span="2":::

        Чтобы получить эту категорию сущностей, добавьте `USBankAccountNumber` к `pii-categories` параметру. `USBankAccountNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `en`
      
   :::column-end:::
:::row-end:::
