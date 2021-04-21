---
title: 'Синхронизация Azure AD Connect: справочник по функциям | Документация Майкрософт'
description: Общие сведения о выражениях декларативной подготовки в Azure AD Connect Sync
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: 4f525ca0-be0e-4a2e-8da1-09b6b567ed5f
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: reference
ms.date: 07/12/2017
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 7454733233f1fd487d774d52a6f46187354ae05c
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98246712"
---
# <a name="azure-ad-connect-sync-functions-reference"></a>Синхронизация Azure AD Connect: справочник по функциям
В службе Azure AD Connect функции используются для обработки значения атрибута во время синхронизации.  
Синтаксис функций выражается в следующем формате:   
`<output type> FunctionName(<input type> <position name>, ..)`

Если функция перегружена и принимает несколько вариантов синтаксиса, перечисляются все допустимые варианты синтаксиса.  
Функции являются строго типизированными и контролируют, чтобы переданный тип соответствовал задокументированному типу.  
Если тип не совпадает, выводится сообщение об ошибке.

Типы выражаются с помощью следующего синтаксиса:

* **bin** — двоичный;
* **bool** — логический;
* **dt** — дата и время в формате UTC;
* **enum** — перечисление известных констант;
* **exp** — выражение, результат вычисления которого должен иметь логический тип;
* **mvbin** — двоичное значение с несколькими значениями.
* **mvstr** — строка с несколькими значениями.
* **mvref** — ссылка с несколькими значениями.
* **num** — числовой;
* **ref** — ссылка.
* **str** — строка.
* **var** — вариант практически любого другого типа;
* **void** — значение не возвращается.

Функции типов **mvbin**, **mvstr** и **mvref** работают только в атрибутах с несколькими значениями. Функции типов **bin**, **str** и **ref** работают с однозначными и многозначными атрибутами.

## <a name="functions-reference"></a>Справочник по функциям

* **Сертификат**
  * [CertExtensionOids](#certextensionoids)
  * [CertFormat](#certformat)
  * [CertFriendlyName](#certfriendlyname)
  * [CertHashString](#certhashstring)
  * [CertIssuer](#certissuer)
  * [CertIssuerDN](#certissuerdn)
  * [CertIssuerOid](#certissueroid)
  * [CertKeyAlgorithm](#certkeyalgorithm)
  * [CertKeyAlgorithmParams](#certkeyalgorithmparams)
  * [CertNameInfo](#certnameinfo)
  * [CertNotAfter](#certnotafter)
  * [CertNotBefore](#certnotbefore)
  * [CertPublicKeyOid](#certpublickeyoid)
  * [CertPublicKeyParametersOid](#certpublickeyparametersoid)
  * [CertSerialNumber](#certserialnumber)
  * [CertSignatureAlgorithmOid](#certsignaturealgorithmoid)
  * [CertSubject](#certsubject)
  * [CertSubjectNameDN](#certsubjectnamedn)
  * [CertSubjectNameOid](#certsubjectnameoid)
  * [CertThumbprint](#certthumbprint)
  * [CertVersion](#certversion)
  * [IsCert](#iscert)
* **Преобразование**
  * [CBool](#cbool)
  * [CDate](#cdate)
  * [CGuid](#cguid)
  * [ConvertFromBase64](#convertfrombase64)
  * [ConvertToBase64](#converttobase64)
  * [ConvertFromUTF8Hex](#convertfromutf8hex)
  * [ConvertToUTF8Hex](#converttoutf8hex)
  * [CNum](#cnum)
  * [CRef](#cref)
  * [CStr](#cstr)
  * [StringFromGuid](#stringfromguid)
  * [StringFromSid](#stringfromsid)
* **Дата и время**
  * [DateAdd](#dateadd)
  * [DateFromNum](#datefromnum)
  * [FormatDateTime](#formatdatetime)
  * [Сейчас](#now)
  * [NumFromDate](#numfromdate)
* **Каталог**
  * [DNComponent](#dncomponent)
  * [DNComponentRev](#dncomponentrev)
  * [EscapeDNComponent](#escapedncomponent)
* **Ознакомительная версия**
  * [IsBitSet](#isbitset)
  * [IsDate](#isdate)
  * [IsEmpty](#isempty)
  * [IsGuid](#isguid)
  * [IsNull](#isnull)
  * [IsNullOrEmpty](#isnullorempty)
  * [IsNumeric](#isnumeric)
  * [IsPresent](#ispresent)
  * [IsString](#isstring)
* **Math**
  * [BitAnd](#bitand)
  * [BitOr](#bitor)
  * [RandomNum](#randomnum)
* **Функции с несколькими значениями**
  * [Содержит](#contains)
  * [Количество](#count)
  * [Элемент](#item)
  * [ItemOrNull](#itemornull)
  * [Join](#join)
  * [RemoveDuplicates](#removeduplicates)
  * [Split](#split)
* **Program Flow**
  * [Ошибка](#error)
  * [IIF](#iif)
  * [Select](#select)
  * [Параметр](#switch)
  * [Where](#where)
  * [With](#with)
* **Text**
  * [GUID](#guid)
  * [InStr](#instr)
  * [InStrRev](#instrrev)
  * [LCase](#lcase)
  * [Слева](#left)
  * [Len](#len)
  * [LTrim](#ltrim)
  * [Mid](#mid)
  * [PadLeft](#padleft)
  * [PadRight](#padright)
  * [PCase](#pcase)
  * [Заменить](#replace)
  * [ReplaceChars](#replacechars)
  * [Right](#right)
  * [RTrim](#rtrim)
  * [Trim](#trim) (Усечь)
  * [UCase](#ucase)
  * [Word](#word)

---
### <a name="bitand"></a>BitAnd
**Описание.**  
 Функция BitAnd задает конкретное значение для битов.

**Синтаксис**  
`num BitAnd(num value1, num value2)`

* value1, value2: числовые значения, которые должны быть соединены оператором AND.

**Примечания:**  
 Эта функция преобразует оба параметра в двоичное представление и устанавливает для бита следующие значения:

* 0 — если один или оба соответствующих бита в *value1* и *value2* равны 0;
* 1 — если оба соответствующих бита равны 1.

Другими словами, функция возвращает значение 0 всегда, за исключением случаев, когда соответствующие биты обоих параметров равны 1.

**Пример**.  
`BitAnd(&HF, &HF7)`  
 Возвращает 7, так как шестнадцатеричные F и F7 преобразовываются в это значение.

---
### <a name="bitor"></a>BitOr
**Описание.**  
 Функция BitOr задает конкретное значение для битов.

**Синтаксис**  
`num BitOr(num value1, num value2)`

* value1, value2: числовые значения, которые должны быть соединены оператором OR.

**Примечания:**  
 Эта функция преобразует оба параметра в двоичное представление и устанавливает для бита значение 1, если один или оба соответствующих бита в маске и флаге равны 1, и устанавливает значение 0, если оба соответствующих бита равны 0. Другими словами, функция возвращает значение 1 всегда, за исключением случаев, когда соответствующие биты обоих параметров равны 0.

---
### <a name="cbool"></a>CBool
**Описание.**  
 Функция CBool возвращает логическое значение на основе вычисленного выражения.

**Синтаксис**  
`bool CBool(exp Expression)`

**Примечания:**  
Если выражение преобразуется в ненулевое значение, функция CBool возвращает значение True, в противном случае она возвращает значение False.

**Пример**.  
`CBool([attrib1] = [attrib2])`  

Возвращает значение True, если оба атрибута имеют одинаковое значение.

---
### <a name="cdate"></a>CDate
**Описание.**  
 Функция CDate возвращает из строки значение даты и времени в формате UTC. DateTime не является собственным типом атрибута синхронизации, но используется некоторыми функциями.

**Синтаксис**  
`dt CDate(str value)`

* Value: строка с датой, временем и (необязательно) часовым поясом.

**Примечания:**  
 Строка всегда возвращается в формате UTC.

**Пример**.  
`CDate([employeeStartTime])`  
 Возвращает объект DateTime с учетом времени начала работы сотрудника.

`CDate("2013-01-10 4:00 PM -8")`  
 Возвращает объект DateTime, представляющий значение 2013-01-11 12:00 AM.


---
### <a name="certextensionoids"></a>CertExtensionOids
**Описание.**  
Возвращает значения Oid всех критических расширений объекта сертификата.

**Синтаксис**  
`mvstr CertExtensionOids(binary certificateRawData)`  
*   certificateRawData: представление сертификата X.509 в виде массива байтов. Массив байтов может представлять данные X.509 в двоичной кодировке (DER) или кодировке Base64.

---
### <a name="certformat"></a>CertFormat
**Описание.**  
Возвращает имя формата данного сертификата X.509v3.

**Синтаксис**  
`str CertFormat(binary certificateRawData)`  
*   certificateRawData: представление сертификата X.509 в виде массива байтов. Массив байтов может представлять данные X.509 в двоичной кодировке (DER) или кодировке Base64.

---
### <a name="certfriendlyname"></a>CertFriendlyName
**Описание.**  
Возвращает связанный с сертификатом псевдоним.

**Синтаксис**  
`str CertFriendlyName(binary certificateRawData)`  
*   certificateRawData: представление сертификата X.509 в виде массива байтов. Массив байтов может представлять данные X.509 в двоичной кодировке (DER) или кодировке Base64.

---
### <a name="certhashstring"></a>CertHashString
**Описание.**  
Возвращает хэш-значение SHA1 для сертификата X.509v3 в виде шестнадцатеричной строки.

**Синтаксис**  
`str CertHashString(binary certificateRawData)`  
*   certificateRawData: представление сертификата X.509 в виде массива байтов. Массив байтов может представлять данные X.509 в двоичной кодировке (DER) или кодировке Base64.

---
### <a name="certissuer"></a>CertIssuer
**Описание.**  
Возвращает имя центра сертификации, выдавшего сертификат X.509v3.

**Синтаксис**  
`str CertIssuer(binary certificateRawData)`  
*   certificateRawData: представление сертификата X.509 в виде массива байтов. Массив байтов может представлять данные X.509 в двоичной кодировке (DER) или кодировке Base64.

---
### <a name="certissuerdn"></a>CertIssuerDN
**Описание.**  
Возвращает различающееся имя издателя сертификата.

**Синтаксис**  
`str CertIssuerDN(binary certificateRawData)`  
*   certificateRawData: представление сертификата X.509 в виде массива байтов. Массив байтов может представлять данные X.509 в двоичной кодировке (DER) или кодировке Base64.

---
### <a name="certissueroid"></a>CertIssuerOid
**Описание.**  
Возвращает идентификатор Oid издателя сертификата.

**Синтаксис**  
`str CertIssuerOid(binary certificateRawData)`  
*   certificateRawData: представление сертификата X.509 в виде массива байтов. Массив байтов может представлять данные X.509 в двоичной кодировке (DER) или кодировке Base64.

---
### <a name="certkeyalgorithm"></a>CertKeyAlgorithm
**Описание.**  
Возвращает сведения об алгоритме ключа для сертификата X.509v3 в виде строки.

**Синтаксис**  
`str CertKeyAlgorithm(binary certificateRawData)`  
*   certificateRawData: представление сертификата X.509 в виде массива байтов. Массив байтов может представлять данные X.509 в двоичной кодировке (DER) или кодировке Base64.

---
### <a name="certkeyalgorithmparams"></a>CertKeyAlgorithmParams
**Описание.**  
Возвращает параметры алгоритма ключа для сертификата X.509v3 в виде шестнадцатеричной строки.

**Синтаксис**  
`str CertKeyAlgorithm(binary certificateRawData)`  
*   certificateRawData: представление сертификата X.509 в виде массива байтов. Массив байтов может представлять данные X.509 в двоичной кодировке (DER) или кодировке Base64.

---
### <a name="certnameinfo"></a>CertNameInfo
**Описание.**  
Возвращает имена субъекта и издателя сертификата.

**Синтаксис**  
`str CertNameInfo(binary certificateRawData, str x509NameType, bool includesIssuerName)`  
*   certificateRawData: представление сертификата X.509 в виде массива байтов. Массив байтов может представлять данные X.509 в двоичной кодировке (DER) или кодировке Base64.
*   X509NameType: значение X509NameType субъекта.
*   includesIssuerName: значение true для включения имени поставщика, в противном случае — значение false.

---
### <a name="certnotafter"></a>CertNotAfter
**Описание.**  
Возвращает дату в формате местного времени, после которой сертификат является недействительным.

**Синтаксис**  
`dt CertNotAfter(binary certificateRawData)`  
*   certificateRawData: представление сертификата X.509 в виде массива байтов. Массив байтов может представлять данные X.509 в двоичной кодировке (DER) или кодировке Base64.

---
### <a name="certnotbefore"></a>CertNotBefore
**Описание.**  
Возвращает дату в формате местного времени, при которой сертификат становится действительным.

**Синтаксис**  
`dt CertNotBefore(binary certificateRawData)`  
*   certificateRawData: представление сертификата X.509 в виде массива байтов. Массив байтов может представлять данные X.509 в двоичной кодировке (DER) или кодировке Base64.

---
### <a name="certpublickeyoid"></a>CertPublicKeyOid
**Описание.**  
Возвращает идентификатор Oid открытого ключа для сертификата X.509v3.

**Синтаксис**  
`str CertKeyAlgorithm(binary certificateRawData)`  
*   certificateRawData: представление сертификата X.509 в виде массива байтов. Массив байтов может представлять данные X.509 в двоичной кодировке (DER) или кодировке Base64.

---
### <a name="certpublickeyparametersoid"></a>CertPublicKeyParametersOid
**Описание.**  
Возвращает идентификатор Oid параметров открытого ключа для сертификата X.509v3.

**Синтаксис**  
`str CertPublicKeyParametersOid(binary certificateRawData)`  
*   certificateRawData: представление сертификата X.509 в виде массива байтов. Массив байтов может представлять данные X.509 в двоичной кодировке (DER) или кодировке Base64.

---
### <a name="certserialnumber"></a>CertSerialNumber
**Описание.**  
Возвращает серийный номер сертификата X.509v3.

**Синтаксис**  
`str CertSerialNumber(binary certificateRawData)`  
*   certificateRawData: представление сертификата X.509 в виде массива байтов. Массив байтов может представлять данные X.509 в двоичной кодировке (DER) или кодировке Base64.

---
### <a name="certsignaturealgorithmoid"></a>CertSignatureAlgorithmOid
**Описание.**  
Возвращает идентификатор Oid алгоритма, который применялся для создания подписи сертификата.

**Синтаксис**  
`str CertSignatureAlgorithmOid(binary certificateRawData)`  
*   certificateRawData: представление сертификата X.509 в виде массива байтов. Массив байтов может представлять данные X.509 в двоичной кодировке (DER) или кодировке Base64.

---
### <a name="certsubject"></a>CertSubject
**Описание.**  
Получает различающееся имя субъекта от сертификата.

**Синтаксис**  
`str CertSubject(binary certificateRawData)`  
*   certificateRawData: представление сертификата X.509 в виде массива байтов. Массив байтов может представлять данные X.509 в двоичной кодировке (DER) или кодировке Base64.

---
### <a name="certsubjectnamedn"></a>CertSubjectNameDN
**Описание.**  
Возвращает различающееся имя субъекта из сертификата.

**Синтаксис**  
`str CertSubjectNameDN(binary certificateRawData)`  
*   certificateRawData: представление сертификата X.509 в виде массива байтов. Массив байтов может представлять данные X.509 в двоичной кодировке (DER) или кодировке Base64.

---
### <a name="certsubjectnameoid"></a>CertSubjectNameOid
**Описание.**  
Возвращает идентификатор Oid имени субъекта из сертификата.

**Синтаксис**  
`str CertSubjectNameOid(binary certificateRawData)`  
*   certificateRawData: представление сертификата X.509 в виде массива байтов. Массив байтов может представлять данные X.509 в двоичной кодировке (DER) или кодировке Base64.

---
### <a name="certthumbprint"></a>CertThumbprint
**Описание.**  
Возвращает отпечаток сертификата.

**Синтаксис**  
`str CertThumbprint(binary certificateRawData)`  
*   certificateRawData: представление сертификата X.509 в виде массива байтов. Массив байтов может представлять данные X.509 в двоичной кодировке (DER) или кодировке Base64.

---
### <a name="certversion"></a>CertVersion
**Описание.**  
Возвращает версию формата X.509 сертификата.

**Синтаксис**  
`str CertThumbprint(binary certificateRawData)`  
*   certificateRawData: представление сертификата X.509 в виде массива байтов. Массив байтов может представлять данные X.509 в двоичной кодировке (DER) или кодировке Base64.

---
### <a name="cguid"></a>CGuid
**Описание.**  
 Функция CGuid преобразует строковое представление идентификатора GUID в двоичное представление.

**Синтаксис**  
`bin CGuid(str GUID)`

* Строка отформатирована по следующему шаблону: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx или {xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx}

---
### <a name="contains"></a>Содержит
**Описание.**  
 Функция Contains находит строку внутри атрибута с несколькими значениями.

**Синтаксис**  
`num Contains (mvstring attribute, str search)` — с учетом регистра  
`num Contains (mvstring attribute, str search, enum Casetype)`  
`num Contains (mvref attribute, str search)` — с учетом регистра

* attribute: атрибут с несколькими значениями для поиска.
* search: строка для поиска в атрибуте.
* Casetype: без учета регистра или с учетом регистра.

Возвращает индекс в атрибут с несколькими значениями, в котором найдена строка. Если строка не найдена, возвращается 0.

**Примечания:**  
 Для атрибутов со строкой с несколькими значениями search будет искать подстроки в значениях.  
Чтобы считаться соответствием в ссылочном атрибуте, искомая строка должна точно соответствовать значению.

**Пример**.  
`IIF(Contains([proxyAddresses],"SMTP:")>0,[proxyAddresses],Error("No primary SMTP address found."))`  
 Если атрибут proxyAddresses имеет основной адрес электронной почты (обозначается буквами в верхнем регистре SMTP:), возвращается атрибут proxyAddress, в противном случае возвращается сообщение об ошибке.

---
### <a name="convertfrombase64"></a>ConvertFromBase64
**Описание.**  
 Функция ConvertFromBase64 преобразует значение в кодировке Base64 в обычную строку.

**Синтаксис**  
`str ConvertFromBase64(str source)` — здесь предполагается кодирование Юникодом.  
`str ConvertFromBase64(str source, enum Encoding)`

* source: строка в кодировке Base64  
* Кодировка: Юникод, ASCII, UTF8

**Пример**  
`ConvertFromBase64("SABlAGwAbABvACAAdwBvAHIAbABkACEA")`  
`ConvertFromBase64("SGVsbG8gd29ybGQh", UTF8)`

Оба примера возвращают сообщение "*Hello world!*".

---
### <a name="convertfromutf8hex"></a>ConvertFromUTF8Hex
**Описание.**  
 Функция ConvertFromUTF8Hex преобразует значение в шестнадцатеричной кодировке UTF8 в обычную строку.

**Синтаксис**  
`str ConvertFromUTF8Hex(str source)`

* source: строка в двухбайтовой кодировке UTF8.

**Примечания:**  
Эта функция отличается от функции ConvertFromBase64([],UTF8) тем, что ее результат является понятным для атрибута различающегося имени.  
Azure Active Directory использует этот формат в качестве различающегося имени.

**Пример**.  
`ConvertFromUTF8Hex("48656C6C6F20776F726C6421")`  
Возвращает сообщение *Hello world!*.

---
### <a name="converttobase64"></a>ConvertToBase64
**Описание.**  
Функция ConvertToBase64 преобразует строку в строку Юникода в кодировке base64.  
Преобразует значение массива целых чисел в эквивалентное строковое представление в кодировке знаков base64.

**Синтаксис**  
`str ConvertToBase64(str source)`

**Пример**.  
`ConvertToBase64("Hello world!")`  
 Возвращает SABlAGwAbABvACAAdwBvAHIAbABkACEA.

---
### <a name="converttoutf8hex"></a>ConvertToUTF8Hex
**Описание.**  
Функция ConvertToUTF8Hex преобразует строку в значение в шестнадцатеричной кодировке UTF8.

**Синтаксис**  
`str ConvertToUTF8Hex(str source)`

**Примечания:**  
 Azure Active Directory использует формат вывода этой функции в качестве формата атрибута различающегося имени.

**Пример**.  
`ConvertToUTF8Hex("Hello world!")`  
 Возвращает 48656C6C6F20776F726C6421.

---
### <a name="count"></a>Count
**Описание.**  
 Функция Count возвращает количество элементов в атрибуте с несколькими значениями.

**Синтаксис**  
`num Count(mvstr attribute)`

---
### <a name="cnum"></a>CNum
**Описание.**  
 Функция CNum принимает строку и возвращает числовой тип данных.

**Синтаксис**  
`num CNum(str value)`

---
### <a name="cref"></a>CRef
**Описание.**  
 Преобразует строку в ссылочный атрибут.

**Синтаксис**  
`ref CRef(str value)`

**Пример**.  
`CRef("CN=LC Services,CN=Microsoft,CN=lcspool01,CN=Pools,CN=RTC Service," & %Forest.LDAP%)`

---
### <a name="cstr"></a>CStr
**Описание.**  
 Функция CStr преобразует данные в строковый тип данных.

**Синтаксис**  
`str CStr(num value)`  
`str CStr(ref value)`  
`str CStr(bool value)`  

* value: может быть числовым значением, ссылочным атрибутом или логическим значением.

**Пример**.  
`CStr([dn])`  
 Может вернуть cn=Joe,dc=contoso,dc=com.

---
### <a name="dateadd"></a>DateAdd
**Описание.**  
 Возвращает значение Date, содержащее дату, к которой добавлен указанный временной интервал.

**Синтаксис**  
`dt DateAdd(str interval, num value, dt date)`

* interval: строковое выражение, определяющее интервал времени, который нужно добавить. Строка должна иметь одно из следующих значений:
  * yyyy год
  * q квартал
  * m месяц
  * y день года
  * d день
  * w день недели
  * ww неделя
  * h час
  * n минута
  * s секунда
* value: количество единиц измерения, которое нужно добавить. Это значение может быть положительным (для получения дат в будущем) или отрицательным (для получения дат в прошлом).
* date: значение DateTime, представляющее собой дату, к которой добавлен интервал.

**Пример**.  
`DateAdd("m", 3, CDate("2001-01-01"))`  
 Добавляет три месяца и возвращает значение DateTime 2001-04-01.

---
### <a name="datefromnum"></a>DateFromNum
**Описание.**  
 Функция DateFromNum преобразует значение даты в формате AD в тип DateTime.

**Синтаксис**  
`dt DateFromNum(num value)`

**Пример**.  
`DateFromNum([lastLogonTimestamp])`  
`DateFromNum(129699324000000000)`  
 Возвращает значение DateTime, равное 2012-01-01 23:00:00.

---
### <a name="dncomponent"></a>DNComponent
**Описание.**  
 Функция DNComponent возвращает значение указанного компонента различающегося имени начиная с левого края.

**Синтаксис**  
`str DNComponent(ref dn, num ComponentNumber)`

* dn: ссылочный атрибут для интерпретации.
* ComponentNumber: компонент в различающемся имени для возврата.

**Пример**.  
`DNComponent(CRef([dn]),1)`  
 Если значение различающегося имени равно cn=Joe,ou=…, функция возвращает Joe.

---
### <a name="dncomponentrev"></a>DNComponentRev
**Описание.**  
 Функция DNComponentRev возвращает значение указанного компонента различающегося имени начиная с правого края (с конца).

**Синтаксис**  
`str DNComponentRev(ref dn, num ComponentNumber)`  
`str DNComponentRev(ref dn, num ComponentNumber, enum Options)`

* dn: ссылочный атрибут для интерпретации.
* ComponentNumber: компонент в различающемся имени для возврата.
* Options: DC — игнорировать все компоненты с "dc=".

**Пример**.  
Если различающееся имя — cn=Joe,ou=Atlanta,ou=GA,ou=US, dc=contoso,dc=com, то  
`DNComponentRev(CRef([dn]),3)`  
`DNComponentRev(CRef([dn]),1,"DC")`  
 оба вернут US.

---
### <a name="error"></a>Ошибка
**Описание.**  
 Функция Error используется для возвращения пользовательских ошибок.

**Синтаксис**  
`void Error(str ErrorMessage)`

**Пример**.  
`IIF(IsPresent([accountName]),[accountName],Error("AccountName is required"))`  
 Если атрибут accountName отсутствует, в объекте возникает ошибка.

---
### <a name="escapedncomponent"></a>EscapeDNComponent
**Описание.**  
 Функция EscapeDNComponent принимает один компонент различающегося имени и экранирует его таким образом, чтобы он мог быть представлен в виде LDAP.

**Синтаксис**  
`str EscapeDNComponent(str value)`

**Пример**.  
`EscapeDNComponent("cn=" & [displayName]) & "," & %ForestLDAP%)`  
 Убедитесь, что объект может быть создан в каталоге LDAP, даже если атрибут displayName содержит символы, которые необходимо экранировать в LDAP.

---
### <a name="formatdatetime"></a>FormatDateTime
**Описание.**  
 Функция FormatDateTime используется для форматирования значения DateTime в строку в указанном формате.

**Синтаксис**  
`str FormatDateTime(dt value, str format)`

* value: значение в формате DateTime
* format: строка, представляющая формат для преобразования.

**Примечания:**  
Возможные значения для формата представлены здесь: [Форматы даты и времени, определенные пользователем (функция FORMAT)](/dax/custom-date-and-time-formats-for-the-format-function).

**Пример**.  

`FormatDateTime(CDate("12/25/2007"),"yyyy-MM-dd")`  
 выдает результат 2007-12-25.

`FormatDateTime(DateFromNum([pwdLastSet]),"yyyyMMddHHmmss.0Z")`  
 может вернуть результат 20140905081453.0Z.

---
### <a name="guid"></a>Guid
**Описание.**  
Функция Guid создает случайный идентификатор GUID.

**Синтаксис**  
`str Guid()`

---
### <a name="iif"></a>IIF
**Описание.**  
Функция IIF возвращает одно из набора возможных значений на основе заданного условия.

**Синтаксис**  
`var IIF(exp condition, var valueIfTrue, var valueIfFalse)`

* condition: любое значение или выражение, которое может принять значение true или false.
* valueIfTrue: значение, возвращаемое, если условие принимает значение true.
* valueIfFalse: значение, возвращаемое, если условие принимает значение false.

**Пример**.  
`IIF([employeeType]="Intern","t-" & [alias],[alias])`  
  Возвращает псевдоним пользователя с префиксом t-, если пользователь является интерном, в противном случае возвращает псевдоним пользователя как есть.

---
### <a name="instr"></a>InStr
**Описание.**  
 Функция InStr находит первое вхождение подстроки в строке.

**Синтаксис**  

`num InStr(str stringcheck, str stringmatch)`  
`num InStr(str stringcheck, str stringmatch, num start)`  
`num InStr(str stringcheck, str stringmatch, num start, enum compare)`

* stringcheck: строка для поиска
* stringmatch: строка, которая должна быть найдена
* start: начальная позиция для поиска подстроки
* compare: vbTextCompare или vbBinaryCompare.

**Примечания:**  
 Возвращает позицию, в которой найдена подстрока, или значение 0, если подстрока не найдена.

**Пример**.  
`InStr("The quick brown fox","quick")`  
 Принимает значение 5.

`InStr("repEated","e",3,vbBinaryCompare)`  
 Принимает значение 7.

---
### <a name="instrrev"></a>InStrRev
**Описание.**  
 Функция InStrRev находит последнее вхождение подстроки в строке.

**Синтаксис**  
`num InstrRev(str stringcheck, str stringmatch)`  
`num InstrRev(str stringcheck, str stringmatch, num start)`  
`num InstrRev(str stringcheck, str stringmatch, num start, enum compare)`

* stringcheck: строка для поиска
* stringmatch: строка, которая должна быть найдена
* start: начальная позиция для поиска подстроки
* compare: vbTextCompare или vbBinaryCompare.

**Примечания:**  
 Возвращает позицию, в которой найдена подстрока, или значение 0, если подстрока не найдена.

**Пример**.  
`InStrRev("abbcdbbbef","bb")`  
 Возвращает значение 7.

---
### <a name="isbitset"></a>IsBitSet
**Описание.**  
 Функция IsBitSet проверяет, задан ли бит.

**Синтаксис**  
`bool IsBitSet(num value, num flag)`

* value: числовое значение, которое представляет собой evaluated.flag, т. е. числовое значение, которое имеет бит для вычисления.

**Пример**.  
`IsBitSet(&HF,4)`  
 Возвращает значение True, так как для бита 4 задано шестнадцатеричное значение F.

---
### <a name="isdate"></a>IsDate
**Описание.**  
 Функция IsDate возвращает значение True, если выражение может быть числовым значением типа DateTime.

**Синтаксис**  
`bool IsDate(var Expression)`

**Примечания:**  
 Используется для определения, будет ли функция CDate() выполнена успешно.

---
### <a name="iscert"></a>IsCert
**Описание.**  
Возвращает значение true, если необработанные данные могут быть сериализованы в объект сертификата .NET X509Certificate2.

**Синтаксис**  
`bool CertThumbprint(binary certificateRawData)`  
*   certificateRawData: представление сертификата X.509 в виде массива байтов. Массив байтов может представлять данные X.509 в двоичной кодировке (DER) или кодировке Base64.
---
### <a name="isempty"></a>IsEmpty
**Описание.**  
 Функция IsEmpty возвращает значение True, если атрибут присутствует в CS или MV, но результатом вычисления является пустая строка.

**Синтаксис**  
`bool IsEmpty(var Expression)`

---
### <a name="isguid"></a>IsGuid
**Описание.**  
 Функция IsGuid возвращает значение True, если строку можно преобразовать в идентификатор GUID.

**Синтаксис**  
`bool IsGuid(str GUID)`

**Примечания:**  
 GUID представляет собой строку, созданную по одному из этих шаблонов: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx или {xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx}.

Используется для определения, будет ли функция CGuid() выполнена успешно.

**Пример**.  
`IIF(IsGuid([strAttribute]),CGuid([strAttribute]),NULL)`  
 Если StrAttribute имеет формат GUID, возвращается двоичное представление идентификатора, если нет — возвращается значение NULL.

---
### <a name="isnull"></a>IsNull
**Описание.**  
 Функция IsNull возвращает значение True, если выражение принимает значение NULL.

**Синтаксис**  
`bool IsNull(var Expression)`

**Примечания:**  
 Значение NULL для атрибута выражается в отсутствии атрибута.

**Пример**.  
`IsNull([displayName])`  
 Возвращает значение True, если атрибут не присутствует в CS или MV.

---
### <a name="isnullorempty"></a>IsNullOrEmpty
**Описание.**  
 Функция IsNullOrEmpty возвращает значение True, если выражение имеет значение NULL или является пустой строкой.

**Синтаксис**  
`bool IsNullOrEmpty(var Expression)`

**Примечания:**  
Для атрибута это будет значение True, если атрибут отсутствует или же присутствует, но является пустой строкой.  
Обратной функцией этой функции является IsPresent.

**Пример**.  
`IsNullOrEmpty([displayName])`  
 Возвращает значение True, если атрибут не присутствует или является пустой строкой в CS или MV.

---
### <a name="isnumeric"></a>IsNumeric
**Описание.**  
 Функция IsNumeric возвращает логическое значение, указывающее, можно ли вычислить выражение как числовой тип.

**Синтаксис**  
`bool IsNumeric(var Expression)`

**Примечания:**  
 Используется для определения, сможет ли функция CNum() успешно проанализировать выражение.

---
### <a name="isstring"></a>IsString
**Описание.**  
 Функция IsString возвращает значение True, если выражение может принимать значение строкового типа.

**Синтаксис**  
`bool IsString(var expression)`

**Примечания:**  
 Используется для определения, сможет ли функция CStr() успешно проанализировать выражение.

---
### <a name="ispresent"></a>IsPresent
**Описание.**  
Функция IsPresent возвращает значение True, если результат выражения — строка, которая не имеет значение NULL и не является пустой.

**Синтаксис**  
`bool IsPresent(var expression)`

**Примечания:**  
 Обратной функцией этой функции является IsNullOrEmpty.

**Пример**.  
`Switch(IsPresent([directManager]),[directManager], IsPresent([skiplevelManager]),[skiplevelManager], IsPresent([director]),[director])`

---
### <a name="item"></a>Элемент
**Описание.**  
 Функция Item возвращает один элемент из строки или атрибута с несколькими значениями.

**Синтаксис**  
`var Item(mvstr attribute, num index)`

* attribute: атрибут с несколькими значениями
* index: индекс элемента в строке с несколькими значениями.

**Примечания:**  
Функцию Item удобно использовать с функцией Contains, так как последняя возвращает индекс элемента в атрибуте с несколькими значениями.

Вызывает ошибку, если индекс выходит за допустимые пределы.

**Пример**.  
`Mid(Item([proxyAddresses],Contains([proxyAddresses], "SMTP:")),6)`  
 Возвращает основной адрес электронной почты.

---
### <a name="itemornull"></a>ItemOrNull
**Описание.**  
 Функция ItemOrNull возвращает один элемент из строки или атрибута с несколькими значениями.

**Синтаксис**  
`var ItemOrNull(mvstr attribute, num index)`

* attribute: атрибут с несколькими значениями
* index: индекс элемента в строке с несколькими значениями.

**Примечания:**  
 Функция ItemOrNull используется вместе с функцией Contains, так как последняя функция возвращает индекс элементу в атрибуте с несколькими значениями.

Возвращает значение Null, если индекс выходит за допустимые пределы.

---
### <a name="join"></a>Join
**Описание.**  
 Функция Join принимает строку с несколькими значениями и возвращает строку с одним значением с указанным разделителем между элементами.

**Синтаксис**  
`str Join(mvstr attribute)`  
`str Join(mvstr attribute, str Delimiter)`

* attribute: атрибут с несколькими значениями, содержащий строки для объединения.
* delimiter: строка, используемая для разделения подстрок в составе возвращаемой строки. Если этот аргумент опущен, используется знак пробела (" "). Если разделитель является строкой нулевой длины («») или Nothing, все элементы списка объединяются без разделителей.

**Замечания**  
 Join и Split выполняют похожие функции. Функция Join принимает массив строк и объединяет их, используя строку-разделитель, чтобы вернуть одну строку. Функция Split принимает строку и разделяет ее по разделителям, чтобы вернуть массив строк. Эти функции различаются тем, что Join может объединять строки с любой строкой-разделителем, а Split может разделять строки с помощью одного символа-разделителя.

**Пример**.  
`Join([proxyAddresses],",")`  
Может вернуть: "SMTP:john.doe@contoso.com,smtp:jd@contoso.com"

---
### <a name="lcase"></a>LCase
**Описание.**  
 Функция LCase преобразует все символы в строке в нижний регистр.

**Синтаксис**  
`str LCase(str value)`

**Пример**.  
`LCase("TeSt")`  
 Возвращает значение Test.

---
### <a name="left"></a>Левый
**Описание.**  
 Функция Left возвращает указанное количество символов из левой части строки.

**Синтаксис**  
`str Left(str string, num NumChars)`

* string: строка, из которой возвращаются символы.
* NumChars: число, определяющее количество символов для возврата из начала строки (слева).

**Примечания:**  
 Строка, содержащая первые numChars символов в строке:

* если numChars = 0, возвращается пустая строка;
* если numChars < 0, возвращается введенная строка;
* если строка имеет значение Null, возвращается пустая строка.

Если строка содержит меньше символов, чем указано в numChars, возвращается строка без изменений (т. е. содержащая все символы в параметре 1).

**Пример**.  
`Left("John Doe", 3)`  
 Возвращает значение Joh.

---
### <a name="len"></a>Len
**Описание.**  
 Функция Len возвращает количество символов в строке.

**Синтаксис**  
`num Len(str value)`

**Пример**.  
`Len("John Doe")`  
 Возвращает значение 8.

---
### <a name="ltrim"></a>LTrim
**Описание.**  
 Функция LTrim удаляет пробелы в начале строки.

**Синтаксис**  
`str LTrim(str value)`

**Пример**.  
`LTrim(" Test ")`  
 Возвращает значение Test.

---
### <a name="mid"></a>Mid
**Описание.**  
 Функция Mid возвращает указанное количество символов из указанной позиции в строке.

**Синтаксис**  
`str Mid(str string, num start, num NumChars)`

* string: строка, из которой возвращаются символы.
* start: число, определяющее начальную позицию в строке для возврата символов из нее.
* NumChars: число, определяющее количество символов для возврата из позиции строки.

**Примечания:**  
 Возвращает символы numChars, начиная с позиции start в строке.  
Строка, содержащая numChars символов начиная с позиции start в строке:

* если numChars = 0, возвращается пустая строка;
* если numChars < 0, возвращается введенная строка;
* если start > длины строки, возвращается строка ввода;
* если start < = 0, возвращается введенная строка;
* если строка имеет значение Null, возвращается пустая строка.

Если в строке не осталось numChar символов начиная с позиции start, возвращается столько символов, сколько может быть возвращено.

**Пример**.  
`Mid("John Doe", 3, 5)`  
 Возвращает значение hn Do.

`Mid("John Doe", 6, 999)`  
 Возвращает значение Doe.

---
### <a name="now"></a>Сейчас
**Описание.**  
 Функция Now возвращает значение DateTime, обозначающее текущую дату и время, установленные на компьютере.

**Синтаксис**  
`dt Now()`

---
### <a name="numfromdate"></a>NumFromDate
**Описание.**  
 Функция NumFromDate возвращает дату в формате даты AD.

**Синтаксис**  
`num NumFromDate(dt value)`

**Пример**.  
`NumFromDate(CDate("2012-01-01 23:00:00"))`  
 Возвращает значение 129699324000000000.

---
### <a name="padleft"></a>PadLeft
**Описание.**  
 Функция PadLeft заполняет строку слева до заданной длины с помощью предоставленного знака заполнения.

**Синтаксис**  
`str PadLeft(str string, num length, str padCharacter)`

* string: строка для заполнения.
* length: целое число, представляющее требуемую длину строки.
* padCharacter: строка, состоящая из одного символа, используемого в качестве символа заполнения.

**Примечания:**

* Если длина строки меньше length, значение padCharacter многократно добавляется в начало (слева) строки, пока длина строки не будет равна length.
* Значение padCharacter может быть символом пробела, но оно не может иметь значение Null.
* Если длина строки равна или больше length, строка возвращается без изменений.
* Если длина строка больше или равна length, строка также возвращается без изменений.
* Если длина строки меньше length, возвращается новая строка требуемой длины, содержащая строку, дополненную padCharacter.
* Если строка имеет значение Null, функция возвращает пустую строку.

**Пример**.  
`PadLeft("User", 10, "0")`  
 Возвращает значение 000000User.

---
### <a name="padright"></a>PadRight
**Описание.**  
 Функция PadRight заполняет строку справа до заданной длины с помощью предоставленного знака заполнения.

**Синтаксис**  
`str PadRight(str string, num length, str padCharacter)`

* string: строка для заполнения.
* length: целое число, представляющее требуемую длину строки.
* padCharacter: строка, состоящая из одного символа, используемого в качестве символа заполнения.

**Примечания:**

* Если длина строки меньше length, значение padCharacter многократно добавляется в конец (справа) строки, пока длина строки не будет равна length.
* Значение padCharacter может быть символом пробела, но оно не может иметь значение Null.
* Если длина строки равна или больше length, строка возвращается без изменений.
* Если длина строка больше или равна length, строка также возвращается без изменений.
* Если длина строки меньше length, возвращается новая строка требуемой длины, содержащая строку, дополненную padCharacter.
* Если строка имеет значение Null, функция возвращает пустую строку.

**Пример**.  
`PadRight("User", 10, "0")`  
 Возвращает значение User000000.

---
### <a name="pcase"></a>PCase
**Описание.**  
 Функция PCase преобразует в верхний регистр первый символ каждого слова, отделенного пробелами, а все остальные символы преобразуются в нижний регистр.

**Синтаксис**  
`String PCase(string)`

**Примечания:**

* Эта функция в настоящее время не обеспечивает правильное применение регистра для преобразования слов, написанных полностью прописными буквами, таких как акронимы.

**Пример**.  
`PCase("TEsT")`  
 Возвращает значение Test.

`PCase(LCase("TEST"))`  
 Возвращает значение Test.

---
### <a name="randomnum"></a>RandomNum
**Описание.**  
 Функция RandomNum возвращает случайное число в заданном интервале.

**Синтаксис**  
`num RandomNum(num start, num end)`

* start: число, определяющее нижний предел случайного значения, которое нужно создать.
* end: число, определяющее верхний предел случайного значения для создания.

**Пример**.  
`Random(100,999)`  
 Может вернуть значение 734.

---
### <a name="removeduplicates"></a>RemoveDuplicates
**Описание.**  
 Функция RemoveDuplicates принимает строку с несколькими значениями и проверяет, чтобы каждое значение было уникальным.

**Синтаксис**  
`mvstr RemoveDuplicates(mvstr attribute)`

**Пример**.  
`RemoveDuplicates([proxyAddresses])`  
 Возвращает очищенный атрибут proxyAddress, в котором удалены все повторяющиеся значения.

---
### <a name="replace"></a>Заменить
**Описание.**  
 Функция Replace заменяет все экземпляры строки на другую строку.

**Синтаксис**  
`str Replace(str string, str OldValue, str NewValue)`

* string: строка, в которой заменяются значения.
* OldValue: строка для поиска и замены.
* NewValue: строка для замены.

**Примечания:**  
 Функция распознает следующие специальные моникеры:

* \n — новая строка;
* \r — возврат каретки;
* \t – символ табуляции.

**Пример**.  
`Replace([address],"\r\n",", ")`  
 Заменяет CRLF запятой и пробелом, может вернуть значение One Microsoft Way, Redmond, WA, USA.

---
### <a name="replacechars"></a>ReplaceChars
**Описание.**  
 Функция ReplaceChars заменяет все экземпляры символов, найденных в строке ReplacePattern.

**Синтаксис**  
`str ReplaceChars(str string, str ReplacePattern)`

* string: строка, в которой заменяются значения.
* ReplacePattern: строка, содержащая словарь с символами для замены.

Формат строки: {source1}: {target1}, {source2}: {target2}, {sourceN}, {targetN}, где source — это символ для поиска, а target — строка для замены.

**Примечания:**

* Функция принимает каждое вхождение определенных источников и заменяет их целевыми объектами.
* Источник должен состоять из ровно одного символа (в кодировке Юникод).
* Источник не может быть пустым или длиннее одного символа (ошибка разбора).
* Цель может содержать несколько знаков, например ö:oe, β:ss.
* Целевой объект может быть пустым, это будет означать, что символ должен быть удален.
* В источнике учитывается регистр, который должен точно совпадать.
* Символы «,» (запятая) и «:» (двоеточие) являются зарезервированными символами и не могут быть заменены с помощью этой функции.
* Пробелы и другие разделители в строке ReplacePattern игнорируются.

**Пример**.  
`%ReplaceString% = ’:,Å:A,Ä:A,Ö:O,å:a,ä:a,ö,o`

`ReplaceChars("Räksmörgås",%ReplaceString%)`  
 Возвращает значение Raksmorgas.

`ReplaceChars("O’Neil",%ReplaceString%)`  
Возвращает значение ONeil. Апостроф определен как символ для удаления.

---
### <a name="right"></a>Правый
**Описание.**  
 Функция Right возвращает указанное количество символов из правой части (в конце) строки.

**Синтаксис**  
`str Right(str string, num NumChars)`

* string: строка, из которой возвращаются символы.
* NumChars: число, определяющее количество символов для возврата из конца строки (справа).

**Примечания:**  
 Символы NumChars возвращаются из последней позиции строки.

Строка, содержащая последние numChars символов в строке:

* если numChars = 0, возвращается пустая строка;
* если numChars < 0, возвращается введенная строка;
* если строка имеет значение Null, возвращается пустая строка.

Если строка содержит меньше символов, чем указано в numChars, возвращается строка без изменений.

**Пример**.  
`Right("John Doe", 3)`  
 Возвращает значение Doe.

---
### <a name="rtrim"></a>RTrim
**Описание.**  
 Функция RTrim удаляет пробелы в конце строки.

**Синтаксис**  
`str RTrim(str value)`

**Пример**.  
`RTrim(" Test ")`  
 Возвращает значение Test.

---
### <a name="select"></a>Выберите пункт
**Описание.**  
Обработка всех значений атрибута с несколькими значениями (или выходного значения выражения) в соответствии с указанной функцией.

**Синтаксис**  
`mvattr Select(variable item, mvattr attribute, func function)`  
`mvattr Select(variable item, exp expression, func function)`

* item: представляет элемент атрибута с несколькими значениями
* attribute: атрибут с несколькими значениями
* expression: выражение, возвращающее коллекцию значений
* condition: любая функция, которая может обработать элемент атрибута

**Примеры**  
`Select($item,[otherPhone],Replace($item,"-",""))`  
Возвращает все значения атрибута otherPhone с несколькими значениями после удаления дефисов (-).

---
### <a name="split"></a>Разделение
**Описание.**  
 Функция Split принимает строку с разделителями и преобразует ее в строку с несколькими значениями.

**Синтаксис**  
`mvstr Split(str value, str delimiter)`  
`mvstr Split(str value, str delimiter, num limit)`

* value: строка с символом разделителя для разделения.
* delimiter: один символ, используемый в качестве разделителя.
* limit: максимальное количество значений, которые могут быть возвращены.

**Пример**.  
`Split("SMTP:john.doe@contoso.com,smtp:jd@contoso.com",",")`  
 Возвращает строку с несколькими значениями и двумя элементами, которые используются для атрибута proxyAddress.

---
### <a name="stringfromguid"></a>StringFromGuid
**Описание.**  
 Функция StringFromGuid принимает двоичный идентификатор GUID и преобразует его в строку.

**Синтаксис**  
`str StringFromGuid(bin GUID)`

---
### <a name="stringfromsid"></a>StringFromSid
**Описание.**  
 Функция StringFromSid преобразует массив байтов с идентификатором безопасности в строку.

**Синтаксис**  
`str StringFromSid(bin ObjectSID)`  

---
### <a name="switch"></a>Коммутатор
**Описание.**  
 Функция Switch используется для возвращения одного значения на основе вычисленных условий.

**Синтаксис**  
`var Switch(exp expr1, var value1[, exp expr2, var value … [, exp expr, var valueN]])`

* expr: выражение типа Variant, которое нужно вычислить.
* value: значение, которое должно быть возвращено, если соответствующее выражение имеет значение True.

**Примечания:**  
 Список аргументов функции Switch состоит из пар выражений и значений. Выражения вычисляются слева направо, и возвращается значение, связанное с первым выражением, которое получает значение True. Если части неправильно распределены по парам, возникает ошибка среды выполнения.

Например, если expr1 имеет значение True, функция Switch возвращает value1. Если expr-1 имеет значение False, а expr-2 — True функция Switch возвращает значение value-2 и т. д.

Функция Switch возвращает Nothing при таких условиях:

* ни одно из выражений не имеет значение True;
* первое значение True имеет соответствующее значение, равное NULL.

Функция Switch вычисляет все выражения, хотя возвращает только одно из них. Поэтому необходимо следить за нежелательными побочными эффектами. Например, если в результате вычисления выражения получится деление на ноль, возвращается ошибка.

Значение также может быть функцией Error, которая возвращает настраиваемую строку.

**Пример**.  
`Switch([city] = "London", "English", [city] = "Rome", "Italian", [city] = "Paris", "French", True, Error("Unknown city"))`  
 Возвращает язык, на котором говорят в некоторых крупных городах, или ошибку.

---
### <a name="trim"></a>Trim
**Описание.**  
 Функция Trim удаляет пробелы в начале и конце строки.

**Синтаксис**  
`str Trim(str value)`  

**Пример**.  
`Trim(" Test ")`  
 Возвращает значение Test.

`Trim([proxyAddresses])`  
 Удаляет начальные и конечные пробелы для каждого значения в атрибуте proxyAddress.

---
### <a name="ucase"></a>UCase
**Описание.**  
 Функция UCase преобразует все символы в строке в верхний регистр.

**Синтаксис**  
`str UCase(str string)`

**Пример**.  
`UCase("TeSt")`  
 Возвращает значение Test.

---
### <a name="where"></a>Where

**Описание.**  
Возвращает подмножество значений атрибута с несколькими значениями (или выходного значения выражения) в соответствии с указанным условием.

**Синтаксис**  
`mvattr Where(variable item, mvattr attribute, exp condition)`  
`mvattr Where(variable item, exp expression, exp condition)`  
* item: представляет элемент атрибута с несколькими значениями
* attribute: атрибут с несколькими значениями
* condition: любое выражение, которое может принимать значение true или false
* expression: выражение, возвращающее коллекцию значений

**Пример**.  
`Where($item,[userCertificate],CertNotAfter($item)>Now())`  
Возврат значений сертификата в атрибуте userCertificate с несколькими значениями, у которых не истек срок действия.

---
### <a name="with"></a>With
**Описание.**  
Функция With дает возможность упростить сложное выражение: часть этого выражения, которая встречается в выражении один или несколько раз, можно представить в виде переменной.

**Синтаксис:** 
`With(var variable, exp subExpression, exp complexExpression)`  
* variable: представляет часть выражения.
* subExpression: часть выражения, представленная переменной.
* complexExpression: сложное выражение.

**Пример**.  
`With($unExpiredCerts,Where($item,[userCertificate],CertNotAfter($item)>Now()),IIF(Count($unExpiredCerts)>0,$unExpiredCerts,NULL))`  
Функционально эквивалентно следующему:  
`IIF (Count(Where($item,[userCertificate],CertNotAfter($item)>Now()))>0, Where($item,[userCertificate],CertNotAfter($item)>Now()),NULL)`  
Здесь возвращаются только значения сертификата с неистекшим сроком действия в атрибуте userCertificate.


---
### <a name="word"></a>Word
**Описание.**  
 Функция Word возвращает слово, содержащееся внутри строки, в зависимости от параметров, описывающих разделители и количество слов для возврата.

**Синтаксис**  
`str Word(str string, num WordNumber, str delimiters)`

* string: строка, из которой возвращаются слова.
* WordNumber: число, определяющее номер слова, которое должно быть возвращено.
* delimiters: строка, представляющая разделители, которые должны использоваться для идентификации слов.

**Примечания:**  
 Каждая строка символов в строке, разделенной одним из символов-разделителей, определяется как слово:

* если number < 1, возвращается пустая строка;
* если string имеет значение Null, возвращается пустая строка.

Если строка содержит меньше слов, чем number, или строка не содержит слов, идентифицируемых с помощью разделителей, возвращается пустая строка.

**Пример**.  
`Word("The quick brown fox",3," ")`  
 Возвращает значение "brown".

`Word("This,string!has&many separators",3,",!&#")`  
 Вернет значение "has".

## <a name="additional-resources"></a>Дополнительные ресурсы
* [Знакомство с выражениями декларативной подготовки](concept-azure-ad-connect-sync-declarative-provisioning-expressions.md)
* [Azure AD Connect Sync: настройка параметров синхронизации](how-to-connect-sync-whatis.md)
* [Интеграция локальных удостоверений с Azure Active Directory](whatis-hybrid-identity.md)
