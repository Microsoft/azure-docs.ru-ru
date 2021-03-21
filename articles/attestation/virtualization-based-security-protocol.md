---
title: Протокол безопасности на основе виртуализации (VBS) для аттестации Azure
description: Протокол аттестации VBS
services: attestation
author: msmbaldwin
ms.service: attestation
ms.topic: reference
ms.date: 07/20/2020
ms.author: mbaldwin
ms.openlocfilehash: 38012c5b4bb9338c1200d9583256193ee8402c98
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99507919"
---
# <a name="trusted-platform-module-tpm-and-virtualization-based-securityvbs-enclave-attestation-protocol"></a>Протокол аттестации анклава (доверенный платформенный модуль) и безопасность на основе виртуализации (VBS) доверенный платформенный модуль (TPM) 

Microsoft Azure аттестации для обеспечения надежной гарантии безопасности полагается на проверку цепочки доверия от корня доверия (TPM) до запуска низкоуровневой оболочки и защищенного ядра. Чтобы добиться этой аттестации Azure, необходимо подтвердить состояние загрузки компьютера, прежде чем можно будет установить отношения доверия в Secure анклава. Двоичные файлы операционной системы, гипервизора и защищенного ядра должны быть подписаны правильными официальными центрами Майкрософт и настроены безопасным образом. После привязки отношения доверия между доверенный платформенный модуль (TPM) (TPM) и работоспособностью гипервизора мы можем доверять безопасности на основе виртуализации (VBS) анклава Идкс в измеряемом журнале загрузки. Это позволяет проверить, что пара ключей была создана в анклава и Mint отчет аттестации, связывающий отношение доверия с этим ключом и содержащий другие утверждения, такие как свойства уровня безопасности и аттестации загрузки. 

VBS енклавес требует наличия TPM, чтобы обеспечить измерение для проверки безопасности. VBS енклавес подтверждается конечной точкой TPM с добавлением к объекту Request в протоколе. 

## <a name="protocol-messages"></a>Сообщения протокола

### <a name="init-message"></a>Сообщение о инициализации

#### <a name="direction"></a>Направление

Клиент-> аттестация Azure

#### <a name="payload"></a>Payload

```
{ 
  "type": "aikcert" 
} 
```

"Type" (строка ASCII): представляет тип запрошенной аттестации. В настоящее время поддерживается только "аикцерт".

### <a name="challenge-message"></a>Сообщение запроса

#### <a name="direction"></a>Направление

Аттестация Azure — > клиент

#### <a name="payload"></a>Payload

```
{ 
  "challenge": "<BASE64URL(CHALLENGE)>", 
  "service_context": "<BASE64URL(SERVICECONTEXT)>" 
} 
```

**запрос** (BASE64URL (октет)): случайное значение, выданное службой.

**service_context** (BASE64URL (октет)): непрозрачный контекст, созданный службой.


### <a name="request-message"></a>Сообщение запроса

#### <a name="direction"></a>Направление

Клиент-> аттестация Azure 

#### <a name="payload"></a>Payload

```
{
  "request": "<JWS>"
}
```

**запрос** (ЖВС): запрос состоит из структуры JSON Web Signature (ЖВС). Ниже показаны полезные данные защищенного заголовка ЖВС и ЖВС. Как и в любой структуре ЖВС, окончательное значение состоит из следующих компонентов:

BASE64URL (UTF8 (защищенный заголовок ЖВС)) | | '.' ||

BASE64URL (полезные данные ЖВС) | | '.' ||

BASE64URL (сигнатура ЖВС)

##### <a name="jws-protected-header"></a>ЖВС, защищенный заголовок

```
{
  "alg": "PS256",
  "typ": "attReq"
  // no "kid" parameter as the key specified by attest_key MUST sign this JWS to prove possession.
}
```

##### <a name="jws-payload"></a>Полезные данные ЖВС

Полезные данные ЖВС могут иметь тип "базовый" или "VBS". Базовый используется, когда свидетельство аттестации не включает данные VBS. 

Пример только TPM: 

``` 
{ 
  "att_type": "basic", 
  "att_data": { 
    "rp_id": "<URL>", 
    "rp_data": "<BASE64URL(RPCUSTOMDATA)>", 
    "challenge": "<BASE64URL(CHALLENGE)>", 

    "tpm_att_data": { 
      "srtm_boot_log": "<BASE64URL(SRTMBOOTLOG)>", 
      "srtm_resume_log": "<BASE64URL(SRTMRESUMELOG)>", 
      "drtm_boot_log": "<BASE64URL(DRTMBOOTLOG)>", 
      "drtm_resume_log": "<BASE64URL(DRTMRESUMELOG)>", 
      "aik_cert": "<BASE64URL(AIKCERTIFICATE)>", 

      // aik_pub is represented as a JSON Web Key (JWK) object (RFC 7517). 

      "aik_pub": { 
        "kty": "RSA", 
        "n": "<Base64urlUInt(MODULUS)>", 
        "e": "<Base64urlUInt(EXPONENT)>" 
      }, 
      "current_claim": "<BASE64URL(CURRENTCLAIM)>", 
      "boot_claim": "<BASE64URL(BOOTCLAIM)>" 
    }, 

    // attest_key is represented as a JSON Web Key (JWK) object (RFC 7517). 

    "attest_key": { 
      "kty": "RSA", 
      "n": "<Base64urlUInt(MODULUS)>", 
      "e": "<Base64urlUInt(EXPONENT)>" 
    }, 
    "custom_claims": [ 
      { 
        "name": "<name>", 
        "value": "<value>", 
        "value_type": "<value_type>" 
      }, 
      { 
        "name": "<name>", 
        "value": "<value>", 
        "value_type": "<value_type>" 
      } 
    ], 
    "service_context": "<BASE64URL(SERVICECONTEXT)>" 
  } 
} 
```

Пример TPM + VBS анклава: 

``` 
{ 
  "att_type": "vbs", 
  "att_data": { 
    "report_signed": { 
      "rp_id": "<URL>", 
      "rp_data": "<BASE64URL(RPCUSTOMDATA)>", 
      "challenge": "<BASE64URL(CHALLENGE)>", 
      "tpm_att_data": { 
        "srtm_boot_log": "<BASE64URL(SRTMBOOTLOG)>", 
        "srtm_resume_log": "<BASE64URL(SRTMRESUMELOG)>", 
        "drtm_boot_log": "<BASE64URL(DRTMBOOTLOG)>", 
        "drtm_resume_log": "<BASE64URL(DRTMRESUMELOG)>", 
        "aik_cert": "<BASE64URL(AIKCERTIFICATE)>", 

        // aik_pub is represented as a JSON Web Key (JWK) object (RFC 7517). 

        "aik_pub": { 
          "kty": "RSA", 
          "n": "<Base64urlUInt(MODULUS)>", 
          "e": "<Base64urlUInt(EXPONENT)>" 
        }, 
        "current_claim": "<BASE64URL(CURRENTCLAIM)>", 
        "boot_claim": "<BASE64URL(BOOTCLAIM)>" 
      }, 

      // attest_key is represented as a JSON Web Key (JWK) object (RFC 7517). 

      "attest_key": { 
        "kty": "RSA", 
        "n": "<Base64urlUInt(MODULUS)>", 
        "e": "<Base64urlUInt(EXPONENT)>" 
      }, 
      "custom_claims": [ 
        { 
          "name": "<name>", 
          "value": "<value>", 
          "value_type": "<value_type>" 
        }, 
        { 
          "name": "<name>", 
          "value": "<value>", 
          "value_type": "<value_type>" 
        } 
      ], 
      "service_context": "<BASE64URL(SERVICECONTEXT)>" 
    }, 
    "vsm_report": "<BASE64URL(REPORT)>" 
  } 
} 
``` 

**rp_id** (стрингорури): идентификатор проверяющей стороны. Используется службой в вычислении утверждения идентификатора компьютера

**rp_data** (BASE64URL (октеты)): непрозрачные данные, переданные проверяющей стороной. Это обычно используется проверяющей стороной как nonce, чтобы гарантировать актуальность отчета.

**запрос** (BASE64URL (октет)): случайное значение, выданное службой

**tpm_att_data**: данные аттестации, связанные с TPM

- **srtm_boot_log (BASE64URL (октеты))**: Журналы загрузки СРТМ, полученные функцией Tbsi_Get_TCG_Log_Ex с типом журнала = TBS_TCGLOG_SRTM_BOOT

- **srtm_resume_log (BASE64URL (октеты))**: СРТМ возобновляет журнал как полученный функцией Tbsi_Get_TCG_Log_Ex с типом журнала = TBS_TCGLOG_SRTM_RESUME

- **drtm_boot_log (BASE64URL (октеты))**: Журналы загрузки ДРТМ, полученные функцией Tbsi_Get_TCG_Log_Ex с типом журнала = TBS_TCGLOG_DRTM_BOOT

- **drtm_resume_log (BASE64URL (октеты))**: ДРТМ возобновляет журнал как полученный функцией Tbsi_Get_TCG_Log_Ex с типом журнала = TBS_TCGLOG_DRTM_RESUME

- **aik_cert (BASE64URL (октет))**: сертификат X. 509 для AIK, возвращенный функцией нкриптжетпроперти со свойством = NCRYPT_CERTIFICATE_PROPERTY

- **aik_pub**: открытая часть пакета AIK представлена как объект JSON Web Key (ЖВК) (RFC 7517).

- **current_claim (BASE64URL (октет))**: утверждение аттестации для текущего состояния PCR, возвращенное функцией Нкрипткреатеклаим с двклаимтипе = NCRYPT_CLAIM_PLATFORM и параметром, NCRYPTBUFFER_TPM_PLATFORM_CLAIM_PCR_MASK установить для включения всех PCRS. Запрос, отправленный службой, должен также использоваться при вычислении этого утверждения.

- **boot_claim (BASE64URL (октет))**: утверждение аттестации для состояния PCR при загрузке, возвращенное функцией Нкрипткреатеклаим с двклаимтипе = NCRYPT_CLAIM_PLATFORM и параметром, NCRYPTBUFFER_TPM_PLATFORM_CLAIM_PCR_MASK задать включение всех PCRS

**vsm_report**   (BASE64URL (ОКТЕТы)): отчет о аттестации VBS анклава, возвращенный функцией Енклавежетаттестатионрепорт. Параметр Енклаведата должен быть хэшем SHA-512 в значении report_signed (включая открывающую и закрывающую фигурные скобки). Входная хэш-функция имеет кодировку UTF8 (report_signed)

**attest_key**: открытая часть ключа анклава, представленная в виде объекта JSON Web Key (ЖВК) (RFC 7517).

**custom_claims**: массив настраиваемых утверждений анклава, отправленных в службу, которая может быть оценена политикой. Утверждение

- **Name (строка)**: имя утверждения. Это имя будет добавлено к URL-адресу, определенному службой аттестации (во избежание конфликтов), а Сцепленная строка становится типом утверждения, которое можно использовать в политике.

- **value (строка)**: значение утверждения.

- **value_type (строка)**: тип данных для значения утверждения

**service_context** (BASE64URL (октет)): непрозрачный контекст, созданный службой.

### <a name="report-message"></a>Сообщение отчета

#### <a name="direction"></a>Направление

Аттестация Azure — > клиент

#### <a name="payload"></a>Payload

```
{
  "report": "<JWT>"
}
```

**Report** (JWT): отчет аттестации в формате JSON Web Token (JWT) (RFC 7519).

## <a name="next-steps"></a>Дальнейшие действия

- [Рабочий процесс аттестации Azure](workflow.md)
