---
title: Список поддерживаемых классификаций
description: На этой странице перечислены поддерживаемые классификации систем в Azure зрения.
author: anmuk601
ms.author: anmuk
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: reference
ms.date: 02/05/2021
ms.openlocfilehash: 60aede65f8217d46844398d0199ff7edca7f36a9
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103200805"
---
# <a name="supported-classifications-in-azure-purview"></a>Поддерживаемые классификации в Azure зрения

В этой статье перечислены поддерживаемые и определенные классификации систем в Azure зрения (Предварительная версия).


- **Порог DISTINCT Match**— общее число уникальных значений данных, которые должны быть найдены в столбце, прежде чем средство проверки запустит на нем шаблон данных. Наши правила классификации системы должны содержать по крайней мере 8 различных значений в каждом столбце, чтобы они были подвержены классификации. Система требует это значение, чтобы убедиться, что столбец содержит достаточно данных, чтобы средство проверки правильно классифицирует его. Например, столбец, содержащий несколько строк, содержащих значение 1, не будет классифицирован. Столбцы, содержащие одну строку со значением, а остальные строки имеют значения NULL, также не будут классифицироваться. При указании нескольких шаблонов это значение применяется к каждому из них.

- **Минимальное пороговое значение соответствия**: это минимальный процент совпадений значений данных в столбце, который должен быть найден сканером для применения классификации. Значение классификации системы равно 60%.


## <a name="defined-system-classifications"></a>Определенные классификации систем

Azure зрения классифицирует данные по [регулярному выражению](https://wikipedia.org/wiki/Regular_expression) и [фильтру раскрытия](https://wikipedia.org/wiki/Bloom_filter). В следующих списках описывается формат, шаблон и ключевые слова для определенных системных классификаций Azure зрения.

Каждое имя классификации имеет префикс MICROSOFT.

## <a name="bloom-filter-classifications"></a>Классификации фильтров раскрытия

## <a name="city-country-and-place"></a>Город, страна и место

Фильтры «город», «страна» и «место» были подготовлены с использованием лучших наборов данных, доступных для подготовки.

## <a name="person"></a>Человек

Фильтр Person раскрытия был подготовлен с помощью следующих двух наборов данных.

- [2010. Перепись данных США за последние имена (записи 162-K)](https://www.census.gov/topics/population/genealogy/data/2010_surnames.html)
- [Популярные наименования с малышами (от SSN) с использованием всех лет 1880-2019 (98-K записей)](https://www.ssa.gov/oact/babynames/limits.html)

## <a name="regex-classifications"></a>Классификации регулярных выражений

## <a name="aba-routing"></a>Маршрутизация ABA

### <a name="format"></a>Формат

Девять цифр, которые могут быть в форматированном или неформатированном шаблоне

### <a name="pattern"></a>Шаблон

Формате

- четыре цифры, начинающиеся с 0, 1, 2, 3, 6, 7 или 8.
- дефис
- четыре цифры
- дефис
- цифра

Неформатированный: девять последовательных цифр, начинающиеся с 0, 1, 2, 3, 6, 7 или 8.

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_aba_routing"></a>Keyword_aba_routing

```
amba number
aba #
aba
abarouting#
abaroutingnumber
americanbankassociationrouting#
americanbankassociationroutingnumber
bank routing#
bankroutingnumber
routing #
routing no
routing number
routing transit number
routing#
RTN
```

## <a name="argentina-national-identity-dni-number"></a>Номер государственного удостоверения (дни)

### <a name="format"></a>Формат

Восемь цифр с периодами или без них

### <a name="pattern"></a>Шаблон

Восемь цифр:

- две цифры
- Необязательный период
- три цифры
- Необязательный период
- три цифры

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_argentina_national_id"></a>Keyword_argentina_national_id

```
Argentina National Identity number
cedula
cédula
dni
documento nacional de identidad
documento número
documento numero
registro nacional de las personas
rnp
```

## <a name="australia-bank-account-number"></a>Номер банковского счета Австралии

### <a name="format"></a>Формат

От шести до 10 цифр с номером филиала банка или без него

### <a name="pattern"></a>Шаблон

Номер счета составляет от шести до 10 цифр.

Номер ветви банковского состояния Австралии:

- три цифры
- дефис
- три цифры

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_australia_bank_account_number"></a>Keyword_australia_bank_account_number

```
swift bank code
correspondent bank
base currency
usa account
holder address
bank address
information account
fund transfers
bank charges
bank details
banking information
full names
iaea
```

## <a name="australia-drivers-license-number"></a>Номер лицензии для драйвера Австралии

### <a name="format"></a>Формат
Девять букв и цифр

### <a name="pattern"></a>Шаблон
девять букв и цифр:

- две цифры или буквы (без учета регистра)
- две цифры
- пять цифр или букв (без учета регистра)

OR

- от одной до двух необязательных букв (без учета регистра)
- от 4 до девяти цифр

OR

- девять цифр или букв (без учета регистра)

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_australia_drivers_license_number"></a>Keyword_australia_drivers_license_number

```
international driving permits
australian automobile association
international driving permit
DriverLicence
DriverLicences
Driver Lic
Driver License
Driver Licenses
DriversLic
DriversLicence
DriversLicences
Drivers Lic
Drivers Lics
Drivers License
Drivers Licenses
Driver'Lic
Driver'Lics
Driver'License
Driver'Licenses
Driver' Lic
Driver' Lics
Driver' License
Driver' Licenses
Driver'sLic
Driver'sLics
Driver'sLicence
Driver'sLicences
Driver's Lic
Driver's Lics
Driver's License
Driver's Licenses
DriverLic#
DriverLics#
DriverLicence#
DriverLicences#
Driver Lic#
Driver Lics#
Driver License#
Driver Licenses#
DriversLic#
DriversLics#
DriversLicence#
DriversLicences#
Drivers Lic#
Drivers Lics#
Drivers License#
Drivers Licenses#
Driver'Lic#
Driver'Lics#
Driver'License#
Driver'Licenses#
Driver' Lic#
Driver' Lics#
Driver' License#
Driver' Licenses#
Driver'sLic#
Driver'sLics#
Driver'sLicence#
Driver'sLicences#
Driver's Lic#
Driver's Lics#
Driver's License#
Driver's Licenses#
```

#### <a name="keyword_australia_drivers_license_number_exclusions"></a>Keyword_australia_drivers_license_number_exclusions

```
aaa
DriverLicense
DriverLicenses
Driver License
Driver Licenses
DriversLicense
DriversLicenses
Drivers License
Drivers Licenses
Driver'License
Driver'Licenses
Driver' License
Driver' Licenses
Driver'sLicense
Driver'sLicenses
Driver's License
Driver's Licenses
DriverLicense#
DriverLicenses#
Driver License#
Driver Licenses#
DriversLicense#
DriversLicenses#
Drivers License#
Drivers Licenses#
Driver'License#
Driver'Licenses#
Driver' License#
Driver' Licenses#
Driver'sLicense#
Driver'sLicenses#
Driver's License#
Driver's Licenses#
```

## <a name="australian-medicare-number"></a>Австралийский Медикаре номер

### <a name="format"></a>Формат

10-11 цифр

### <a name="pattern"></a>Шаблон

10-11 цифр:

- Первая цифра находится в диапазоне 2-6
- девятая цифра является контрольной цифрой
- Десятая цифра — это цифра проблемы
- Одиннадцатая цифра (необязательно) — это отдельный номер

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_australia_medicare_number"></a>Keyword_Australia_Medicare_Number

```
bank account details
medicare payments
mortgage account
bank payments
information branch
credit card loan
department of human services
local service
medicare
```

## <a name="australia-passport-number"></a>Номер паспорта для Австралии

### <a name="format"></a>Формат

Буква, за которой следуют семь цифр

### <a name="pattern"></a>Шаблон

Буква (без учета регистра), за которой следуют семь цифр

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_passport"></a>Паспорт ключевого слова \_

```
Passport Number
Passport No
Passport #
Passport#
PassportID
Passport no
passport number
パスポート
パスポート番号
パスポートのNum
パスポート＃
Numéro de passeport
Passeport n °
Passeport Non
Passeport #
Passeport#
PasseportNon
Passeportn °
```

#### <a name="keyword_australia_passport_number"></a>Зарезервированное слово \_ \_ номер паспорта для Австралии \_

```
passport
passport details
immigration and citizenship
commonwealth of australia
department of immigration
residential address
department of immigration and citizenship
visa
national identity card
passport number
travel document
issuing authority
```

## <a name="australia-tax-file-number"></a>Номер файла налога Австралии

### <a name="format"></a>Формат

от восьми до девяти цифр

### <a name="pattern"></a>Шаблон

от восьми до девяти цифр, как правило, представляются пробелы следующим образом:

- три цифры
- дополнительное пространство
- три цифры
- дополнительное пространство
- две-три цифры, где последняя цифра является контрольной цифрой

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_australia_tax_file_number"></a>Ключевое слово " \_ \_ номер налогового \_ файла" Австралии \_

```
australian business number
marginal tax rate
medicare levy
portfolio number
service veterans
withholding tax
individual tax return
tax file number
tfn
```

## <a name="belgium-national-number"></a>Бельгийский национальный номер

### <a name="format"></a>Формат

11 цифр плюс необязательные разделители

### <a name="pattern"></a>Шаблон

11 цифр плюс разделители:

- шесть цифр и две необязательные периоды в формате YY. ММ. ДД — Дата рождения
- Необязательный разделитель из точки, тире, пробела
- три последовательных цифры (нечетные для мужчин, даже для женщин;)
- Необязательный разделитель из точки, тире, пробела
- две контрольные цифры

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_belgium_national_number"></a>Ключевое слово \_ Бельгии — \_ Национальный \_ номер

```
be lasting aantal
bnn#
bnn
carte d'identité
identifiant national
identifiantnational#
identificatie
identification
identifikation
identifikationsnummer
identifizierung
identité
identiteit
identiteitskaart
identity
inscription
national number
national register
national number#
national number
nif#
nif
numéro d'assuré
numéro de registre national
numéro de sécurité
numéro d'identification
numéro d'immatriculation
numéro national
numéronational#
personal ID number
personalausweis
personalidnumber#
registratie
registration
registrationsnumme
registrierung
social security number
ssn#
ssn
steuernummer
tax ID
tax identification no
tax identification number
tax no#
tax no
tax number
tax registration number
tax ID#
tax ID no#
tax ID number#
tax no#
tax number#
tax number
tin ID
tin no
tin#
```

## <a name="brazil-cpf-number"></a>Номер CPF в Бразилии

### <a name="format"></a>Формат

11 цифр, которые содержат контрольную цифру и могут быть форматированы или неформатированы

### <a name="pattern"></a>Шаблон

Формате

- три цифры
- точка
- три цифры
- точка
- три цифры
- дефис
- две цифры, которые являются контрольными цифрами

Неформатированный вид:

- 11 цифр, где две последние цифры — проверка цифр

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_brazil_cpf"></a>По ключевому слову \_ CPF в Бразилии \_

```
CPF
Identification
Registration
Revenue
Cadastro de Pessoas Físicas
Imposto
Identificação
Inscrição
Receita
```

## <a name="brazil-legal-entity-number-cnpj"></a>Номер юридического лица в Бразилии (CNPJ)

### <a name="format"></a>Формат

14 цифр, включающих регистрационный номер, номер филиала и контрольные цифры, а также разделители

### <a name="pattern"></a>Шаблон

14 цифр плюс разделители:

- две цифры
- точка
- три цифры
- точка
- три цифры (первые восемь цифр — номер регистрации).
- косая черта вперед
- номер ветви из четырех цифр
- дефис
- две цифры, которые являются контрольными цифрами

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_brazil_cnpj"></a>Ключевое слово \_ Бразилии \_ CNPJ

```
CNPJ
CNPJ/MF
CNPJ-MF
National Registry of Legal Entities
Taxpayers Registry
Legal entity
Legal entities
Registration Status
Business
Company
CNPJ
Cadastro Nacional da Pessoa Jurídica
Cadastro Geral de Contribuintes
CGC
Pessoa jurídica
Pessoas jurídicas
Situação cadastral
Inscrição
Empresa
```

## <a name="brazil-national-identification-card-rg"></a>Национальная идентификационная карта Бразилии (RG)

### <a name="format"></a>Формат

:::no-loc text="Registro Geral (old format): Nine digits":::

:::no-loc text="Registro de Identidade (RIC) (new format): 11 digits":::

### <a name="pattern"></a>Шаблон

:::no-loc text="Registro Geral (old format):":::

- две цифры
- точка
- три цифры
- точка
- три цифры
- дефис
- одна цифра, которая является контрольной цифрой

:::no-loc text="Registro de Identidade (RIC) (new format):":::

- 10 знака
- дефис
- одна цифра, которая является контрольной цифрой

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_brazil_rg"></a>Ключевое слово \_ Бразилии \_ RG

```
Cédula de identidade
identity card
national ID
número de rregistro
registro de Iidentidade
registro geral
RG (this keyword is case-sensitive)
RIC (this keyword is case-sensitive)
```

## <a name="canada-bank-account-number"></a>Номер банковского счета Канады

### <a name="format"></a>Формат

7 или 12 цифр

### <a name="pattern"></a>Шаблон

Номер банковского счета Канады составляет 7 или 12 цифр.

Номер транзитного банковского счета для Канады:

- пять цифр
- дефис
- три цифры или
- нуль &quot; 0&quot;
- восемь цифр

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_canada_bank_account_number"></a>\_ \_ \_ Номер банковского счета \_ по ключевому слову Канада

```
canada savings bonds
canada revenue agency
canadian financial institution
direct deposit form
canadian citizen
legal representative
notary public
commissioner for oaths
child care benefit
universal child care
canada child tax benefit
income tax benefit
harmonized sales tax
social insurance number
income tax refund
child tax benefit
territorial payments
institution number
deposit request
banking information
direct deposit
```

## <a name="canada-drivers-license-number"></a>Номер лицензии для драйвера Канады

### <a name="format"></a>Формат

Зависит от Республики

### <a name="pattern"></a>Шаблон

Различные шаблоны, охватывающие Альберта, Британская Колумбия, Манитоба, New Нью Брунсвик, Ньюфаундленд/Лабрадор, Новая Шотландия, Онтарио, Prince Эдвард, Квебек и Саскачеван

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_province_name_drivers_license_name"></a>Ключевое слово \_ [ \_ название Республики] \_ \_ имя лицензии для драйверов \_

- Сокращенное название Республики, например AB
- Название Республики, например Альберта

#### <a name="keyword_canada_drivers_license"></a>Лицензия на использование ключевых слов для \_ \_ драйверов Канады \_

```
DL
DLS
CDL
CDLS
DriverLic
DriverLics
DriverLicense
DriverLicenses
DriverLicence
DriverLicences
Driver Lic
Driver Lics
Driver License
Driver Licenses
Driver License
Driver Licenses
DriversLic
DriversLics
DriversLicence
DriversLicences
DriversLicense
DriversLicenses
Drivers Lic
Drivers Lics
Drivers License
Drivers Licenses
Drivers License
Drivers Licenses
Driver'Lic
Driver'Lics
Driver'License
Driver'Licenses
Driver'License
Driver'Licenses
Driver' Lic
Driver' Lics
Driver' License
Driver' Licenses
Driver' License
Driver' Licenses
Driver'sLic
Driver'sLics
Driver'sLicense
Driver'sLicenses
Driver'sLicence
Driver'sLicences
Driver's Lic
Driver's Lics
Driver's License
Driver's Licenses
Driver's License
Driver's Licenses
Permis de Conduire
ID
IDs
ID card number
ID card numbers
ID card #
ID card #s
ID card
ID cards
ID card
identification number
identification numbers
identification #
identification #s
identification card
identification cards
identification
DL#
DLS#
CDL#
CDLS#
DriverLic#
DriverLics#
DriverLicense#
DriverLicenses#
DriverLicence#
DriverLicences#
Driver Lic#
Driver Lics#
Driver License#
Driver Licenses#
Driver License#
Driver Licenses#
DriversLic#
DriversLics#
DriversLicense#
DriversLicenses#
DriversLicence#
DriversLicences#
Drivers Lic#
Drivers Lics#
Drivers License#
Drivers Licenses#
Drivers License#
Drivers Licenses#
Driver'Lic#
Driver'Lics#
Driver'License#
Driver'Licenses#
Driver'License#
Driver'Licenses#
Driver' Lic#
Driver' Lics#
Driver' License#
Driver' Licenses#
Driver' License#
Driver' Licenses#
Driver'sLic#
Driver'sLics#
Driver'sLicense#
Driver'sLicenses#
Driver'sLicence#
Driver'sLicences#
Driver's Lic#
Driver's Lics#
Driver's License#
Driver's Licenses#
Driver's License#
Driver's Licenses#
Permis de Conduire#
ID#
IDs#
ID card#
ID cards#
ID card#
identification card#
identification cards#
identification#
```

## <a name="canada-health-service-number"></a>Номер службы работоспособности Канады

### <a name="format"></a>Формат

10 знака

### <a name="pattern"></a>Шаблон

10 знака

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_canada_health_service_number"></a>Ключевое слово " \_ \_ \_ номер службы работоспособности Канады" \_

```
personal health number
patient information
health services
specialty services
automobile accident
patient hospital
psychiatrist
workers compensation
disability
```

## <a name="canada-passport-number"></a>Номер паспорта для Канады

### <a name="format"></a>Формат

две прописные буквы, за которыми следуют шесть цифр

### <a name="pattern"></a>Шаблон

две прописные буквы, за которыми следуют шесть цифр

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_canada_passport_number"></a>Ключевое слово \_ Канада \_ \_ номер паспорта

```
canadian citizenship
canadian passport
passport application
passport photos
certified translator
canadian citizens
processing times
renewal application
```

#### <a name="keyword_passport"></a>Паспорт ключевого слова \_

```
Passport Number
Passport No
Passport #
Passport#
PassportID
Passport no
passport number
パスポート
パスポート番号
パスポートのNum
パスポート＃
Numéro de passeport
Passeport n °
Passeport Non
Passeport #
Passeport#
PasseportNon
Passeportn °
```

## <a name="canada-personal-health-identification-number-phin"></a>Личный идентификационный номер работоспособности (фин) для Канады

### <a name="format"></a>Формат

девять цифр

### <a name="pattern"></a>Шаблон

девять цифр

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_canada_phin"></a>Ключевое слово \_ Канада \_ Фин

```
social insurance number
health information act
income tax information
manitoba health
health registration
prescription purchases
benefit eligibility
personal health
power of attorney
registration number
personal health number
practitioner referral
wellness professional
patient referral
health and wellness
```

#### <a name="keyword_canada_provinces"></a>Зарезервированное слово, \_ \_ провинции

```
Nunavut
Quebec
Northwest Territories
Ontario
British Columbia
Alberta
Saskatchewan
Manitoba
Yukon
Newfoundland and Labrador
New Brunswick
Nova Scotia
Prince Edward Island
Canada
```

## <a name="canada-social-insurance-number"></a>Номер страхования социальных сетей (Канада)

### <a name="format"></a>Формат

девять цифр с необязательными дефисами или пробелами

### <a name="pattern"></a>Шаблон

Формате

- три цифры
- дефис или пробел
- три цифры
- дефис или пробел
- три цифры

Неформатированные: девять цифр

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_sin"></a>Ключевое слово \_ sin

```
sin
social insurance
numero d'assurance sociale
sins
ssn
ssns
social security
numero d'assurance social
national identification number
national ID
sin#
soc ins
social ins
```

#### <a name="keyword_sin_collaborative"></a>Ключевое слово \_ sin для \_ совместной работы

```
driver's license
drivers license
driver's license
drivers license
DOB
Birthdate
Birthday
Date of Birth
```

## <a name="chile-identity-card-number"></a>Номер идентификационной карты для Чили

### <a name="format"></a>Формат

от семи до восьми цифр и разделителей проверять цифру или букву

### <a name="pattern"></a>Шаблон

от семи до восьми цифр плюс разделители:

- от одной до двух цифр
- Необязательный период
- три цифры
- Необязательный период
- три цифры
- тире
- одна цифра или буква (без учета регистра), которая является контрольной цифрой

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_chile_id_card"></a>\_ \_ Идентификационная \_ карта с ключевым словом Чили

```
cédula de identidad
identificación
national identification
national identification number
national ID
número de identificación nacional
rol único nacional
rol único tributario
RUN
RUT
tarjeta de identificación
Rol Unico Nacional
Rol Unico Tributario
RUN#
RUT#
nationaluniqueroleID#
nacional identidad
número identificación
identidad número
numero identificacion
identidad numero
Chilean identity no.
Chilean identity number
Chilean identity #
Unique Tax Registry
Unique Tributary Role
Unique Tax Role
Unique Tributary Number
Unique National Number
Unique National Role
National unique role
Chile identity no.
Chile identity number
Chile identity #
```

## <a name="china-resident-identity-card-prc-number"></a>Китай — номер резидентной идентификационной карточки (КНР)

### <a name="format"></a>Формат

18 цифр

### <a name="pattern"></a>Шаблон

18 цифр:

- шесть цифр, которые являются кодом адреса
- восемь цифр в формате ГГГГММДД, которые являются датой рождения
- три цифры, которые являются кодом заказа
- одна цифра, которая является контрольной цифрой

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_china_resident_id"></a>Зарезервированное слово \_ китайского \_ резидентного \_ идентификатора

```
Resident Identity Card
PRC
National Identification Card
身份证
居民身份证
居民身份证
鉴定
身分證
居民身份證
鑑定
```

## <a name="credit-card-number"></a>Номер кредитной карты

### <a name="format"></a>Формат

от 14 до 16 цифр, которые могут быть форматированы или неформатированы (дддддддддддддддд) и которые должны пройти тест Луна.

### <a name="pattern"></a>Шаблон

Сложная и надежная схема, которая находит карты из всех крупных торговых марок, включая Visa, MasterCard, Discover Card, ЖКБ, Америка Express, подарочные карты и Динер карты.

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_cc_verification"></a>Проверка ключевого слова \_ CC \_

```
card verification
card identification number
cvn
cid
cvc2
cvv2
pin block
security code
security number
security no
issue number
issue no
cryptogramme
numéro de sécurité
numero de securite
kreditkartenprüfnummer
kreditkartenprufnummer
prüfziffer
prufziffer
sicherheits Kode
sicherheitscode
sicherheitsnummer
verfalldatum
codice di verifica
cod. sicurezza
cod sicurezza
n autorizzazione
código
codigo
cod. seg
cod seg
código de segurança
codigo de seguranca
codigo de segurança
código de seguranca
cód. segurança
cod. seguranca
cod. segurança
cód. seguranca
cód segurança
cod seguranca
cod segurança
cód seguranca
número de verificação
numero de verificacao
ablauf
gültig bis
gültigkeitsdatum
gultig bis
gultigkeitsdatum
scadenza
data scad
fecha de expiracion
fecha de venc
vencimiento
válido hasta
valido hasta
vto
data de expiração
data de expiracao
data em que expira
validade
valor
vencimento
transaction
transaction number
reference number
セキュリティコード
セキュリティ コード
セキュリティナンバー
セキュリティ ナンバー
セキュリティ番号
```

#### <a name="keyword_cc_name"></a>\_Имя для копии ключевого слова \_

```
amex
Americans express
Americans express
americano espresso
Visa
Mastercard
master card
mc
master cards
master cards
diner's Club
diners club
diners club
discover
discover card
discover card
discover cards
JCB
BrandSmart
japanese card bureau
carte blanche
carteblanche
credit card
cc#
cc#:
expiration date
exp date
expiry date
date d'expiration
date d'exp
date expiration
bank card
bank card
card number
card num
card number
card numbers
card numbers
credit card
credit cards
credit cards
ccn
card holder
cardholder
card holders
cardholders
check card
check card
check cards
check cards
debit card
debit card
debit cards
debit cards
atm card
atmcard
atm cards
atmcards
enroute
en route
card type
Cardmember Acct
cardmember account
Card no
Corporate Card
Corporate cards
Type of card
card account number
card member account
Cardmember Acct.
card no.
card no
card number
carte bancaire
carte de crédit
carte de credit
numéro de carte
numero de carte
nº de la carte
nº de carte
kreditkarte
karte
karteninhaber
karteninhabers
kreditkarteninhaber
kreditkarteninstitut
kreditkartentyp
eigentümername
kartennr
kartennummer
kreditkartennummer
kreditkarten-nummer
carta di credito
carta credito
n. carta
n carta
nr. carta
nr carta
numero carta
numero della carta
numero di carta
tarjeta credito
tarjeta de credito
tarjeta crédito
tarjeta de crédito
tarjeta de atm
tarjeta atm
tarjeta debito
tarjeta de debito
tarjeta débito
tarjeta de débito
nº de tarjeta
no. de tarjeta
no de tarjeta
numero de tarjeta
número de tarjeta
tarjeta no
tarjetahabiente
cartão de crédito
cartão de credito
cartao de crédito
cartao de credito
cartão de débito
cartao de débito
cartão de debito
cartao de debito
débito automático
debito automatico
número do cartão
numero do cartão
número do cartao
numero do cartao
número de cartão
numero de cartão
número de cartao
numero de cartao
nº do cartão
nº do cartao
nº. do cartão
no do cartão
no do cartao
no. do cartão
no. do cartao
クレジットカード番号
クレジットカードナンバー
クレジットカード＃
クレジットカード
クレジット
クレカ
カード番号
カードナンバー
カード＃
アメックス
アメリカンエクスプレス
アメリカン エクスプレス
Visaカード
Visa カード
マスターカード
マスター カード
マスター
ダイナースクラブ
ダイナース クラブ
ダイナース
有効期限
期限
キャッシュカード
キャッシュ カード
カード名義人
カードの名義人
カードの名義
デビット カード
デビットカード
```

## <a name="croatia-drivers-license-number"></a>Номер лицензии драйвера Хорватия

Эта сущность типа конфиденциальной информации доступна только в типе конфиденциальной информации с номером лицензии для драйвера ЕС.

### <a name="format"></a>Формат

восемь цифр без пробелов и разделителей

### <a name="pattern"></a>Шаблон

восемь цифр

### <a name="keywords"></a>Ключевые слова

#### <a name="keywords_eu_drivers_license_number"></a>\_ \_ Номер лицензии драйвера ЕС \_ по \_ ключевому слову

```
driverlic
driverlics
driver license
driver licenses
driverlicence
driverlicences
driver lic
driver lics
driver license
driver licenses
driver license
driver licenses
driverslic
driverslics
driverslicence
driverslicences
drivers license
drivers licenses
drivers lic
drivers lics
drivers license
drivers licenses
drivers license
drivers licenses
driver'lic
driver'lics
driver'license
driver'licenses
driver'license
driver'licenses
driver' lic
driver' lics
driver' license
driver' licenses
driver' license
driver' licenses
driver'slic
driver'slics
driver'license
driver'licenses
driver'license
driver'licenses
driver's lic
driver's lics
driver's license
driver's licenses
driver's license
driver's licenses
dl#
dls#
driverlic#
driverlics#
driver license#
driver licenses#
driverlicence#
driverlicences#
driver lic#
driver lics#
driver license#
driver licenses#
driver licenses#
driverslic#
driverslics#
drivers license#
drivers licenses#
driverslicence#
driverslicences#
drivers lic#
drivers lics#
drivers license#
drivers licenses#
drivers license#
drivers licenses#
driver'lic#
driver'lics#
driver'license#
driver'licenses#
driver'license#
driver'licenses#
driver' lic#
driver' lics#
driver' license#
driver' licenses#
driver' license#
driver' licenses#
driver'slic#
driver'slics#
driver'license#
driver'licenses#
driver'license#
driver'licenses#
driver's lic#
driver's lics#
driver's license#
driver's licenses#
driver's license#
driver's licenses#
driving license
driving license
dlno#
driv lic
driv licen
driv license
driv licenses
driv license
driv licenses
driver licen
drivers licen
driver's licen
driving lic
driving licen
driving licenses
driving license
driving licenses
driving permit
dl no
dlno
dl number
```

#### <a name="keywords_croatia_eu_drivers_license_number"></a>Ключевые \_ слова \_ Хорватия \_ \_ номер лицензии для драйвера ЕС \_

```
vozačka dozvola
vozačke dozvole
```

## <a name="croatia-identity-card-number"></a>Номер идентификационной карты (Хорватия)

Эта сущность типа конфиденциальной информации включается в тип конфиденциальной информации о национальном номере страны ЕС и доступна в виде изолированной сущности типа конфиденциальной информации.

### <a name="format"></a>Формат

девять цифр

### <a name="pattern"></a>Шаблон

девять последовательных цифр

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_croatia_id_card"></a>Ключевое слово \_ Хорватия \_ ID \_ карточка

```
majstorski broj građana
master citizen number
nacionalni identifikacijski broj
national identification number
oib#
oib
osobna iskaznica
osobni ID
osobni identifikacijski broj
personal identification number
porezni broj
porezni identifikacijski broj
tax ID
tax identification no
tax identification number
tax no#
tax no
tax number
tax registration number
tax ID#
tax ID no#
tax ID number#
tax no#
tax number#
tax number
tin ID
tin no
tin#
```

## <a name="croatia-personal-identification-oib-number"></a>Номер личной идентификации (OIB) (Хорватия)

### <a name="format"></a>Формат

11 цифр

### <a name="pattern"></a>Шаблон

11 цифр:

- 10 знака
- Последняя цифра является контрольной цифрой

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_croatia_oib_number"></a>Ключевое слово \_ Хорватия \_ OIB \_ Number

```
majstorski broj građana
master citizen number
nacionalni identifikacijski broj
national identification number
oib#
oib
osobna iskaznica
osobni ID
osobni identifikacijski broj
personal identification number
porezni broj
porezni identifikacijski broj
tax ID
tax identification no
tax identification number
tax no#
tax no
tax number
tax registration number
tax ID#
tax ID no#
tax Id number#
tax no#
tax number#
tax number
tin ID
tin no
tin#
```

## <a name="denmark-personal-identification-number"></a>Дания личного идентификационного номера

### <a name="format"></a>Формат

10 цифр, содержащих дефис

### <a name="pattern"></a>Шаблон

10 цифр:

- шесть цифр в формате ДДММИИ, которые являются датой рождения
- дефис
- четыре цифры, в которых конечная цифра является контрольной цифрой

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_denmark_id"></a>Ключевое слово \_ Дании \_ ID

```
centrale person register
civilt registreringssystem
cpr
cpr#
gesundheitskarte nummer
gesundheitsversicherungkarte nummer
health card
health insurance card number
health insurance number
identification number
identifikationsnummer
identifikationsnummer#
identity number
krankenkassennummer
national ID#
national number#
national number
personalidnumber#
personalidentityno#
personal ID number
personnummer
personnummer#
reisekrankenversicherungskartenummer
rejsesygesikringskort
ssn
ssn#
skat ID
skat kode
skat nummer
skattenummer
social security number
sundhedsforsikringskort
sundhedsforsikringsnummer
sundhedskort
sundhedskortnummer
sygesikring
sygesikringkortnummer
tax code
travel health insurance card
uniqueidentityno#
tax number
tax registration number
tax ID
tax identification number
tax ID#
tax number#
tax no
tax no#
tax number
tax identification no
tin#
tax ID no#
tax ID number#
tax no#
tin ID
tin no
cpr.nr
cprnr
cprnummer
personnr
person register
sygesikringsbevis
sygesikringsbevisnr
sygesikringsbevisnummer
sygesikringskort
sygesikringskortnr
sygesikringskortnummer
sygesikringsnr
sygesikringsnummer
```

## <a name="eu-debit-card-number"></a>Номер дебетовой карты ЕС

### <a name="format"></a>Формат

16 цифр

### <a name="pattern"></a>Шаблон

Сложный и надежный шаблон

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_eu_debit_card"></a>\_ \_ Дебетовая карта ЕС по ключевому слову \_

```
account number
card number
card no.
security number
cc#
```

#### <a name="keyword_card_terms_dict"></a>Ключевое слово " \_ условия на карточке" \_ \_ словарь

```
acct nbr
acct num
acct no
Americans express
Americans express
americano espresso
amex
atm card
atm cards
atm kaart
atmcard
atmcards
atmkaart
atmkaarten
ban contact
bank card
bankkaart
card holder
card holders
card num
card number
card numbers
card type
cardano numerico
cardholder
cardholders
card number
card numbers
carta bianca
carta credito
carta di credito
cartao de credito
cartao de crédito
cartao de debito
cartao de débito
carte bancaire
carte blanche
carte bleue
carte de credit
carte de crédit
carte di credito
carteblanche
cartão de credito
cartão de crédito
cartão de debito
cartão de débito
cb
ccn
check card
check cards
check card
check cards
chequekaart
cirrus
cirrus-edc-maestro
controlekaart
controlekaarten
credit card
credit cards
credit card
credit cards
debetkaart
debetkaarten
debit card
debit cards
debit card
debit cards
debito automatico
diners club
diners club
discover
discover card
discover cards
discover card
discover cards
débito automático
edc
eigentümername
european debit card
hoofdkaart
hoofdkaarten
in viaggio
japanese card bureau
japanse kaartdienst
jcb
kaart
kaart num
kaartaantal
kaartaantallen
kaarthouder
kaarthouders
karte
karteninhaber
karteninhabers
kartennr
kartennummer
kreditkarte
kreditkarten-nummer
kreditkarteninhaber
kreditkarteninstitut
kreditkartennummer
kreditkartentyp
maestro
master card
master cards
Mastercard
master cards
mc
mister cash
n carta
carta
no de tarjeta
no do cartao
no do cartão
no. de tarjeta
no. do cartao
no. do cartão
nr carta
nr. carta
numeri di scheda
numero carta
numero de cartao
numero de carte
numero de cartão
numero de tarjeta
numero della carta
numero di carta
numero di scheda
numero do cartao
numero do cartão
numéro de carte
nº carta
nº de carte
nº de la carte
nº de tarjeta
nº do cartao
nº do cartão
nº. do cartão
número de cartao
número de cartão
número de tarjeta
número do cartao
scheda dell'assegno
scheda dell'atmosfera
scheda dell'atmosfera
scheda della banca
scheda di controllo
scheda di debito
scheda matrice
schede dell'atmosfera
schede di controllo
schede di debito
schede matrici
scoprono la scheda
scoprono le schede
solo
supporti di scheda
supporto di scheda
switch
tarjeta atm
tarjeta credito
tarjeta de atm
tarjeta de credito
tarjeta de debito
tarjeta debito
tarjeta no
tarjetahabiente
tipo della scheda
ufficio giapponese della
scheda
v pay
v-pay
visa
visa plus
visa electron
visto
visum
vpay
```

#### <a name="keyword_card_security_terms_dict"></a>Ключевые \_ слова \_ безопасности \_ карточки \_ словарь

```
card identification number
card verification
cardi la verifica
cid
cod seg
cod seguranca
cod segurança
cod sicurezza
cod. seg
cod. seguranca
cod. segurança
cod. sicurezza
codice di sicurezza
codice di verifica
codigo
codigo de seguranca
codigo de segurança
crittogramma
cryptogram
cryptogramme
cv2
cvc
cvc2
cvn
cvv
cvv2
cód seguranca
cód segurança
cód. seguranca
cód. segurança
código
código de seguranca
código de segurança
de kaart controle
geeft nr uit
issue no
issue number
kaartidentificatienummer
kreditkartenprufnummer
kreditkartenprüfnummer
kwestieaantal
no. dell'edizione
no. di sicurezza
numero de securite
numero de verificacao
numero dell'edizione
numero di identificazione della
scheda
numero di sicurezza
numero van veiligheid
numéro de sécurité
nº autorizzazione
número de verificação
perno il blocco
pin block
prufziffer
prüfziffer
security code
security no
security number
sicherheits kode
sicherheitscode
sicherheitsnummer
speldblok
veiligheid nr
veiligheidsaantal
veiligheidscode
veiligheidsnummer
verfalldatum
```

#### <a name="keyword_card_expiration_terms_dict"></a>\_ \_ Условия истечения срока действия карточки ключевого слова \_ \_ словарь

```
ablauf
data de expiracao
data de expiração
data del exp
data di exp
data di scadenza
data em que expira
data scad
data scadenza
date de validité
datum afloop
datum van exp
de afloop
espira
espira
exp date
exp datum
expiration
expire
expires
expiry
fecha de expiracion
fecha de venc
gultig bis
gultigkeitsdatum
gültig bis
gültigkeitsdatum
la scadenza
scadenza
valable
validade
valido hasta
valor
venc
vencimento
vencimiento
verloopt
vervaldag
vervaldatum
vto
válido hasta
```

## <a name="eu-drivers-license-number"></a>Номер лицензии для драйвера ЕС

Это сущности в типе конфиденциальной информации номера лицензии драйвера ЕС.

- Австрия
- Бельгия
- Болгария
- Хорватия
- Кипр
- Чешский
- Дания
- Эстония
- Финляндия
- Франция
- Германия
- Греция
- Венгрия
- Ирландия
- Италия
- Латвия
- Литва
- Люксембург
- Мальта
- Нидерланды
- Польша
- Португалия
- Румыния
- Словакия
- Словения
- Испания
- Швеция
- Великобритании

## <a name="eu-national-identification-number"></a>Национальный идентификационный номер ЕС

Это сущности в типе конфиденциальной информации, определяемой национальным идентификационным номером ЕС.

- Австрия
- Бельгия
- Болгария
- Хорватия
- Кипр
- Чешский
- Дания
- Эстония
- Финляндия
- Франция
- Германия
- Греция
- Венгрия
- Ирландия
- Италия
- Латвия
- Литва
- Люксембург
- Мальта
- Нидерланды
- Польша
- Португалия
- Румыния
- Словакия
- Словения
- Испания
- Великобритании

## <a name="eu-passport-number"></a>Номер паспорта ЕС

Эти сущности в номере ЕС Passport конфиденциальной информации Типесесе являются сущностями в пакете номеров паспорта ЕС.

- Австрия
- Бельгия
- Болгария
- Хорватия
- Кипр
- Чешский
- Дания
- Эстония
- Финляндия
- Франция
- Германия
- Греция
- Венгрия
- Ирландия
- Италия
- Латвия
- Литва
- Люксембург
- Мальта
- Нидерланды
- Польша
- Португалия
- Румыния
- Словакия
- Словения
- Испания
- Швеция
- Великобритании

## <a name="eu-social-security-number-or-equivalent-identification"></a>Номер социального страхования ЕС или эквивалентная идентификация

Это сущности, которые относятся к идентификатору социального страхования ЕС или эквивалентному типу конфиденциальной информации.

- Австрия
- Бельгия
- Хорватия
- Чешский
- Дания
- Финляндия
- Франция
- Германия
- Греция
- Венгрия
- Португалия
- Испания
- Швеция

## <a name="eu-tax-identification-number"></a>Идентификационный номер налогоплательщика ЕС

Эти сущности находятся в типе конфиденциальной информации налогового номера ЕС.

- Австрия
- Бельгия
- Болгария
- Хорватия
- Кипр
- Чешский
- Дания
- Эстония
- Финляндия
- Франция
- Германия
- Греция
- Венгрия
- Ирландия
- Италия
- Латвия
- Литва
- Люксембург
- Мальта
- Нидерланды
- Польша
- Португалия
- Румыния
- Словакия
- Словения
- Испания
- Швеция
- Великобритании



## <a name="finland-national-id"></a>Национальный идентификатор Финляндии

### <a name="format"></a>Формат

шесть цифр плюс символ, указывающий на столетие плюс три цифры плюс цифру чека

### <a name="pattern"></a>Шаблон

Шаблон должен включать все следующие:

- шесть цифр в формате ДДММИИ, которые являются датой рождения
- Метка века ("-", "+" или "a")
- личный идентификационный номер из трех цифр
- цифра или буква (без учета регистра), которая является контрольной цифрой

### <a name="keywords"></a>Ключевые слова

```
ainutlaatuinen henkilökohtainen tunnus
henkilökohtainen tunnus
henkilötunnus
henkilötunnusnumero#
henkilötunnusnumero
hetu
ID no
ID number
identification number
identiteetti numero
identity number
ID number
kansallinen henkilötunnus
kansallisen henkilökortin
national ID card
national ID no.
personal ID
personal identity code
personalidnumber#
personbeteckning
personnummer
social security number
sosiaaliturvatunnus
tax ID
tax identification no
tax identification number
tax no#
tax no
tax number
tax registration number
tax ID#
tax ID no#
tax ID number#
tax no#
tax number#
tax number
tin ID
tin no
tin#
tunnistenumero
tunnus numero
tunnusluku
tunnusnumero
verokortti
veronumero
verotunniste
verotunnus
```

## <a name="finland-passport-number"></a>Номер паспорта для Финляндии

Эта сущность типа конфиденциальной информации доступна в типе конфиденциальной информации паспорта ЕС и доступна в виде изолированной сущности типа конфиденциальной информации.

### <a name="format"></a>Формат

сочетание девяти букв и цифр

### <a name="pattern"></a>Шаблон

сочетание девяти букв и цифр:

- две буквы (без учета регистра)
- семь цифр

### <a name="keywords"></a>Ключевые слова

#### <a name="keywords_eu_passport_number_common"></a>Ключевые слова. \_ \_ \_ \_ Общий номер паспорта ЕС

```
passport#
passport #
passport ID
passports
passport no
passport no
passport number
passport number
passport numbers
passport numbers
```

#### <a name="keyword_finland_passport_number"></a>\_ \_ Номер паспорта выслова Финляндии \_

```
suomalainen passi
passin numero
passin numero.#
passin numero#
passin numero.
passi#
passi number
```

## <a name="france-drivers-license-number"></a>Номер лицензии для драйвера Франции

Эта сущность типа конфиденциальной информации доступна в типе конфиденциальной информации номера лицензии драйвера ЕС и доступна в виде изолированной сущности типа конфиденциальной информации.

### <a name="format"></a>Формат

12 цифр

### <a name="pattern"></a>Шаблон

12 цифр с проверкой на похожие шаблоны, такие как французские телефонные номера

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_french_drivers_license"></a>\_Лицензия на французские \_ драйверы \_

```
driverlic
driverlics
driver license
driver licenses
driverlicence
driverlicences
driver lic
driver lics
driver license
driver licenses
driver license
driver licenses
driverslic
driverslics
driverslicence
driverslicences
drivers license
drivers licenses
drivers lic
drivers lics
drivers license
drivers licenses
drivers license
drivers licenses
driver'lic
driver'lics
driver'license
driver'licenses
driver'license
driver'licenses
driver' lic
driver' lics
driver' license
driver' licenses
driver' license
driver' licenses
driver'slic
driver'slics
driver'license
driver'licenses
driver'license
driver'licenses
driver's lic
driver's lics
driver's license
driver's licenses
driver's license
driver's licenses
dl#
dls#
driverlic#
driverlics#
driver license#
driver licenses#
driverlicence#
driverlicences#
driver lic#
driver lics#
driver license#
driver licenses#
driver licenses#
driverslic#
driverslics#
drivers license#
drivers licenses#
driverslicence#
driverslicences#
drivers lic#
drivers lics#
drivers license#
drivers licenses#
drivers license#
drivers licenses#
driver'lic#
driver'lics#
driver'license#
driver'licenses#
driver'license#
driver'licenses#
driver' lic#
driver' lics#
driver' license#
driver' licenses#
driver' license#
driver' licenses#
driver'slic#
driver'slics#
driver'license#
driver'licenses#
driver'license#
driver'licenses#
driver's lic#
driver's lics#
driver's license#
driver's licenses#
driver's license#
driver's licenses#
driving license
driving license
dlno#
driv lic
driv licen
driv license
driv licenses
driv license
driv licenses
driver licen
drivers licen
driver's licen
driving lic
driving licen
driving licenses
driving license
driving licenses
driving permit
dl no
dlno
dl number
permis de conduire
license number
license number
license numbers
license numbers
numéros de license
```

## <a name="france-national-id-card-cni"></a>Французская Национальная ИДЕНТИФИКАЦИОНная карта (CNI)

### <a name="format"></a>Формат

12 цифр

### <a name="pattern"></a>Шаблон

12 цифр

### <a name="keywords"></a>Ключевые слова

#### <a name="keywords_france_eu_national_id_card"></a>Ключевые слова \_ Франция \_ ЕС \_ Национальный \_ Идентификационная \_ карта

```
card number
carte nationale d'identité
carte nationale d'idenite no
cni#
cni
compte bancaire
national identification number
national identity
nationalidno#
numéro d'assurance maladie
numéro de carte vitale
```

## <a name="france-passport-number"></a>Номер паспорта Франции

Эта сущность типа конфиденциальной информации доступна в типе конфиденциальной информации паспорта ЕС и доступна в виде изолированной сущности типа конфиденциальной информации.

### <a name="format"></a>Формат

девять цифр и букв

### <a name="pattern"></a>Шаблон

девять цифр и букв:

- две цифры
- две буквы (без учета регистра)
- пять цифр

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_passport"></a>Паспорт ключевого слова \_

```
Passport Number
Passport No
Passport #
Passport#
PassportID
Passport no
passport number
パスポート
パスポート番号
パスポートのNum
パスポート＃
Numéro de passeport
Passeport n °
Passeport Non
Passeport #
Passeport#
PasseportNon
Passeportn °
```

## <a name="france-social-security-number-insee-or-equivalent-identification"></a>Номер социального страхования (ИНСИ) Франции или эквивалентная идентификация

Эта сущность типа конфиденциальной информации включена в номер социального страхования ЕС и эквивалентный тип конфиденциальной информации и доступна в виде изолированной сущности типа конфиденциальной информации.

### <a name="format"></a>Формат

15 знака

### <a name="pattern"></a>Шаблон

Должен соответствовать одному из двух шаблонов:

- 13 цифр, за которыми следует пробел, за которым следуют две цифры или
- 15 последовательных цифр

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_fr_insee"></a>Ключевое слово \_ fr \_ ИНСИ

```
insee
securité sociale
securite sociale
national ID
national identification
numéro d'identité
no d'identité
no. d'identité
numero d'identite
no d'identite
no. d'identite
social security number
social security code
social insurance number
le numéro d'identification nationale
d'identité nationale
numéro de sécurité sociale
le code de la sécurité sociale
numéro d'assurance sociale
numéro de sécu
code sécu
```

## <a name="germany-drivers-license-number"></a>Номер лицензии для драйвера Германии

Эта сущность типа конфиденциальной информации включена в тип конфиденциальной информации номера лицензии драйвера ЕС и доступна в виде изолированной сущности типа конфиденциальной информации.

### <a name="format"></a>Формат

сочетание 11 цифр и букв

### <a name="pattern"></a>Шаблон

11 цифр и букв (без учета регистра):

- цифра или буква
- две цифры
- шесть цифр или букв
- цифра
- цифра или буква

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_german_drivers_license_number"></a>Ключевое слово для \_ немецких \_ драйверов \_ \_ номер лицензии

```
ausstellungsdatum
ausstellungsort
ausstellende behöde
ausstellende behorde
ausstellende behoerde
führerschein
fuhrerschein
fuehrerschein
führerscheinnummer
fuhrerscheinnummer
fuehrerscheinnummer
führerschein-
fuhrerschein-
fuehrerschein-
führerscheinnummernr
fuhrerscheinnummernr
fuehrerscheinnummernr
führerscheinnummerklasse
fuhrerscheinnummerklasse
fuehrerscheinnummerklasse
nr-führerschein
nr-fuhrerschein
nr-fuehrerschein
no-führerschein
no-fuhrerschein
no-fuehrerschein
n-führerschein
n-fuhrerschein
n-fuehrerschein
permis de conduire
driverlic
driverlics
driver license
driver licenses
driverlicence
driverlicences
driver lic
driver lics
driver license
driver licenses
driver license
driver licenses
driverslic
driverslics
driverslicence
driverslicences
drivers license
drivers licenses
drivers lic
drivers lics
drivers license
drivers licenses
drivers license
drivers licenses
driver'lic
driver'lics
driver'license
driver'licenses
driver'license
driver'licenses
driver' lic
driver' lics
driver' license
driver' licenses
driver' license
driver' licenses
driver'slic
driver'slics
driver'license
driver'licenses
driver'license
driver'licenses
driver's lic
driver's lics
driver's license
driver's licenses
driver's license
driver's licenses
dl#
dls#
driverlic#
driverlics#
driver license#
driver licenses#
driverlicence#
driverlicences#
driver lic#
driver lics#
driver license#
driver licenses#
driver licenses#
driverslic#
driverslics#
drivers license#
drivers licenses#
driverslicence#
driverslicences#
drivers lic#
drivers lics#
drivers license#
drivers licenses#
drivers license#
drivers licenses#
driver'lic#
driver'lics#
driver'license#
driver'licenses#
driver'license#
driver'licenses#
driver' lic#
driver' lics#
driver' license#
driver' licenses#
driver' license#
driver' licenses#
driver'slic#
driver'slics#
driver'license#
driver'licenses#
driver'license#
driver'licenses#
driver's lic#
driver's lics#
driver's license#
driver's licenses#
driver's license#
driver's licenses#
driving license
driving license
dlno#
driv lic
driv licen
driv license
driv licenses
driv license
driv licenses
driver licen
drivers licen
driver's licen
driving lic
driving licen
driving licenses
driving license
driving licenses
driving permit
dlno
```

## <a name="germany-identity-card-number"></a>Номер карточки удостоверения Германии

### <a name="format"></a>Формат

с 1 ноября 2010: девять букв и цифр

с 1 апреля 1987 до 31 октября 2010:10 цифр

### <a name="pattern"></a>Шаблон

с 1 ноября 2010:

- одна буква (без учета регистра)
- восемь цифр

с 1 апреля 1987 до 31 октября 2010:

- 10 знака

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_germany_id_card"></a>Ключевое слово \_ Германии \_ ID \_ Card

```
ausweis
gpid
identification
identifikation
identifizierungsnummer
identity card
identity number
id-nummer
personal ID
personalausweis
persönliche ID nummer
persönliche identifikationsnummer
persönliche-id-nummer
```

## <a name="germany-passport-number"></a>Номер паспорта для Германии

Эта сущность типа конфиденциальной информации включена в тип конфиденциальной информации паспорта ЕС и доступна в виде изолированной сущности типа конфиденциальной информации.

### <a name="format"></a>Формат

10 цифр или букв

### <a name="pattern"></a>Шаблон

Шаблон должен включать все следующие:

- первым символом является цифра или буква в этом наборе (C, F, G, H, J, K)
- три цифры
- пять цифр или букв из этого набора (C,-H, J-N, P, R, T, V-Z)
- цифра

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_german_passport"></a>Ключевое слово \_ немецкая \_ Служба Passport

```
reisepasse
reisepassnummer
No-Reisepass
Nr-Reisepass
Reisepass-Nr
Passnummer
reisepässe
passeport no.
passeport no
```

## <a name="greece-national-id-card"></a>Греция National ID Card

### <a name="format"></a>Формат

Сочетание букв и цифр 7-8, плюс тире

### <a name="pattern"></a>Шаблон

Семь букв и цифр (старый формат):

- Одна буква (любая буква греческого алфавита)
- Тире
- Шесть цифр

Восемь букв и цифр (новый формат):

- Две буквы, чей символ верхнего регистра встречаются как в греческом, так и в Латинской алфавите (АБЕЗИКМНОПТИКС)
- Тире
- Шесть цифр

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_greece_id_card"></a>Ключевое слово \_ Греция \_ ID \_ Card

```
greek ID
greek national ID
greek personal ID card
greek police ID
identity card
tautotita
ταυτότητα
ταυτότητας
```

## <a name="hong-kong-identity-card-hkid-number"></a>Номер карточки удостоверения Гонконг (ХКИД)

### <a name="format"></a>Формат

Сочетание букв и цифр 8-9, плюс необязательные круглые скобки вокруг последнего символа

### <a name="pattern"></a>Шаблон

Сочетание 8-9 букв:

- 1-2 буквы (без учета регистра)
- Шесть цифр
- Последний символ (любая цифра или буква A), который является цифрой чека и может быть заключен в круглые скобки.

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_hong_kong_id_card"></a>Ключевое слово \_ Гонконг \_ \_ ID \_ Card

```
hkid
Hong Kong identity card
HKIDC
ID card
identity card
hk identity card
Hong Kong ID
香港身份證
香港永久性居民身份證
身份證
身份証
身分證
身分証
香港身份証
香港身分證
香港身分証
香港身份證
香港居民身份證
香港居民身份証
香港居民身分證
香港居民身分証
香港永久性居民身份証
香港永久性居民身分證
香港永久性居民身分証
香港永久性居民身份證
香港非永久性居民身份證
香港非永久性居民身份証
香港非永久性居民身分證
香港非永久性居民身分証
香港特別行政區永久性居民身份證
香港特別行政區永久性居民身份証
香港特別行政區永久性居民身分證
香港特別行政區永久性居民身分証
香港特別行政區非永久性居民身份證
香港特別行政區非永久性居民身份証
香港特別行政區非永久性居民身分證
香港特別行政區非永久性居民身分証
```

## <a name="ip-address"></a>IP-адрес

### <a name="format"></a>Формат

#### <a name="ipv4"></a>IPv4

Сложный шаблон, в котором учетные записи для отформатированных (периодов) и неформатированных (без точек) версий IPv4-адресов

#### <a name="ipv6"></a>Протокол

Сложный шаблон, в котором учетные записи для форматированных номеров IPv6 (включая двоеточия)

### <a name="pattern"></a>Шаблон

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_ipaddress"></a>IP-адрес ключевого слова \_

```
IP (this keyword is case-sensitive)
ip address
ip addresses
internet protocol
IP-כתובת ה
```

## <a name="india-permanent-account-number-pan"></a>Номер постоянной учетной записи Индии (PAN)

### <a name="format"></a>Формат

10 букв или цифр

### <a name="pattern"></a>Шаблон

10 букв или цифр:

- Три буквы (без учета регистра)
- Буква в C, P, H, F, A, T, B, L, J, G (без учета регистра)
- Буква
- четыре цифры;
- Буква (без учета регистра)

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_india_permanent_account_number"></a>Ключевое слово " \_ \_ Постоянный \_ номер учетной записи Индии" \_

```
Permanent Account Number
PAN
```

## <a name="india-unique-identification-aadhaar-number"></a>Номер уникальной идентификации (Аадхаар) Индии

### <a name="format"></a>Формат

12 цифр, содержащих необязательные пробелы или тире

### <a name="pattern"></a>Шаблон

12 цифр:

- Цифра, не равная 0 или 1
- Три цифры
- Необязательный пробел или тире
- четыре цифры;
- Необязательный пробел или тире
- Последняя цифра, которая является контрольной цифрой

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_india_aadhar"></a>Ключевое слово \_ Индии \_ Аадхар

```
aadhaar
aadhar
aadhar#
uid
आधार
uidai
```

## <a name="indonesia-identity-card-ktp-number"></a>Номер карточки удостоверения Индонезия (КТП)

### <a name="format"></a>Формат

16 цифр, содержащих необязательные периоды

### <a name="pattern"></a>Шаблон

16 цифр:

- Код района из двух цифр
- Точка (необязательно)
- Реженци или код города с двумя цифрами
- 2-значный код подрайона
- Точка (необязательно)
- Шесть цифр в формате ДДММИИ, которые являются датой рождения
- Точка (необязательно)
- четыре цифры;

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_indonesia_id_card"></a>\_ \_ Карточка идентификатора Индонезия \_

```
KTP
Kartu Tanda Penduduk
Nomor Induk Kependudukan
```

## <a name="international-banking-account-number-iban"></a>Международный номер банковского счета (IBAN)

### <a name="format"></a>Формат

Код страны (две буквы) плюс цифры (две цифры) плюс :::no-loc text="bban"::: число (до 30 символов)

### <a name="pattern"></a>Шаблон

Шаблон должен включать все следующие:

- Двухбуквенный код страны
- Две контрольные цифры (за которыми следует дополнительное пространство)
- 1-7. группы из четырех букв или цифр (могут быть разделены пробелами)
- 1-3 букв или цифр

Формат каждой страны немного отличается. Тип конфиденциальной информации IBAN охватывает следующие 60 стран:

```
ad, ae, al, at, az, ba, be, bg, bh, ch, cr, cy, cz, de, dk, do, ee, es, fi, fo, fr, gb, ge, gi, gl, gr, hr, hu, that is, il, is, it, kw, kz, lb, li, lt, lu, lv, mc, md, me, mk, mr, mt, mu, nl, no, pl, pt, ro, rs, sa, se, si, sk, sm, tn, tr, vg
```

### <a name="keywords"></a>Ключевые слова

Нет

## <a name="ireland-personal-public-service-pps-number"></a>Номер личной общедоступной службы (PPS)

### <a name="format"></a>Формат

Старый формат (до 31 декабря 2012):

- семь цифр, за которыми следует 1-2 букв

Новый формат (1 января 2013 и после):

- семь цифр, за которыми следует две буквы

### <a name="pattern"></a>Шаблон

Старый формат (до 31 декабря 2012):

- семь цифр
- от одной до двух букв (без учета регистра)

Новый формат (1 января 2013 и после):

- семь цифр
- буква (без учета регистра), которая является цифрой алфавитной проверки
- Необязательная буква в диапазоне A-I или &quot; W&quot;

### <a name="keywords"></a>Ключевые слова

#### <a name="keywords_ireland_eu_national_id_card"></a>Ключевые слова \_ Ирландии \_ ЕС \_ Национальная \_ Идентификационная \_ карта

```
client identity service
identification number
personal ID number
personal public service number
personal service no
phearsanta seirbhíse poiblí
pps no
pps number
pps num
pps service no
ppsn
ppsno#
ppsno
psp
public service no
publicserviceno#
publicserviceno
revenue and social insurance number
rsi no
rsi number
rsin
seirbhís aitheantais cliant
uimh
uimhir aitheantais chánach
uimhir aitheantais phearsanta
uimhir phearsanta seirbhíse poiblí
tax ID
tax identification no
tax identification number
tax no#
tax no
tax number
tax registration number
tax ID#
tax ID no#
tax ID number#
tax no#
tax number#
tax number
tin ID
tin no
tin#
```

## <a name="israel-bank-account-number"></a>Номер банковского счета Израиля

### <a name="format"></a>Формат

13 цифр

### <a name="pattern"></a>Шаблон

Формате

- две цифры
- тире
- три цифры
- тире
- восемь цифр

Неформатированный вид:

- 13 последовательных цифр

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_israel_bank_account_number"></a>Ключевое слово \_ Израиля \_ \_ номер банковского счета \_

```
Bank Account Number
Bank Account
Account Number
מספר חשבון בנק
```

## <a name="israel-national-identification-number"></a>Идентификационный номер Израиля (национальный)

### <a name="format"></a>Формат

девять цифр

### <a name="pattern"></a>Шаблон

девять последовательных цифр

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_israel_national_id"></a>Ключевое слово \_ Израиля \_ идентификатор национального алфавита \_

```
מספר זהות
מספר זיה וי
מספר זיהוי ישר אלי
זהותישר אלית
هو ية اسرائيل ية عدد
هوية إسرائ يلية
رقم الهوية
عدد هوية فريدة من نوعها
ID number#
ID number
identity no
identity number#
identity number
israeliidentitynumber
personal ID
unique ID
```

## <a name="italy-drivers-license-number"></a>Номер лицензии для драйвера Италии

Эта сущность типа конфиденциальной информации включена в тип конфиденциальной информации номера лицензии драйвера ЕС и доступна в виде изолированной сущности типа конфиденциальной информации.

### <a name="format"></a>Формат

сочетание 10 букв и цифр

### <a name="pattern"></a>Шаблон

сочетание 10 букв и цифр:

- одна буква (без учета регистра)
- буква &quot; A &quot; или &quot; V &quot; (без учета регистра)
- семь цифр
- одна буква (без учета регистра)

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_italy_drivers_license_number"></a>Ключевое слово \_ Италии \_ драйверы \_ \_ номер лицензии

```
numero di patente
patente di guida
patente guida
patenti di guida
patenti guida
```

## <a name="japan-bank-account-number"></a>Номер банковского счета для Японии

### <a name="format"></a>Формат

семь или восемь цифр

### <a name="pattern"></a>Шаблон

номер банковского счета:

- семь или восемь цифр
- код ветви банковского счета:
- четыре цифры
- пробел или тире (необязательно)
- три цифры

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_jp_bank_account"></a>Ключевое слово \_ JP — \_ банковский \_ счет

```
Checking Account Number
Checking Account
Checking Account #
Checking Acct Number
Checking Acct #
Checking Acct No.
Checking Account No.
Bank Account Number
Bank Account
Bank Account #
Bank Acct Number
Bank Acct #
Bank Acct No.
Bank Account No.
Savings Account Number
Savings Account
Savings Account #
Savings Acct Number
Savings Acct #
Savings Acct No.
Savings Account No.
Debit Account Number
Debit Account
Debit Account #
Debit Acct Number
Debit Acct #
Debit Acct No.
Debit Account No.
口座番号
銀行口座
銀行口座番号
総合口座
普通預金口座
普通口座
当座預金口座
当座口座
預金口座
振替口座
銀行
バンク
```

#### <a name="keyword_jp_bank_branch_code"></a>Ключевое слово \_ JP \_ \_ код филиала банка \_

```
支店番号
支店コード
店番号
```

## <a name="japan-drivers-license-number"></a>Номер лицензии для драйвера Японии

### <a name="format"></a>Формат

12 цифр

### <a name="pattern"></a>Шаблон

12 последовательных цифр

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_jp_drivers_license_number"></a>Ключевое слово \_ JP \_ Drivers \_ \_ номер лицензии

```
driver license
drivers license
driver'license
drivers licenses
driver'licenses
driver licenses
dl#
dls#
lic#
lics#
運転免許証
運転免許
免許証
免許
運転免許証番号
運転免許番号
免許証番号
免許番号
運転免許証ナンバー
運転免許ナンバー
免許証ナンバー
運転免許証no
運転免許no
免許証no
免許no
運転経歴証明書番号
運転経歴証明書
運転免許証No.
運転免許No.
免許証No.
免許No.
運転免許証#
運転免許#
免許証#
免許#
```

## <a name="japan-passport-number"></a>Номер паспорта для Японии

### <a name="format"></a>Формат

две буквы, за которыми следуют семь цифр

### <a name="pattern"></a>Шаблон

две буквы (без учета регистра), за которыми следуют семь цифр

#### <a name="keyword_jp_passport"></a>Ключевое слово \_ JP \_ Passport

```
Passport
Passport Number
Passport No.
Passport #
パスポート
パスポート番号
パスポートナンバー
パスポート＃
パスポート#
パスポートNo.
旅券番号
旅券番号＃
旅券番号♯
旅券ナンバー
```

## <a name="japan-residence-card-number"></a>Номер карты для Японии

### <a name="format"></a>Формат

12 букв и цифр

### <a name="pattern"></a>Шаблон

12 букв и цифр:

- две буквы (без учета регистра)
- восемь цифр
- две буквы (без учета регистра)

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_jp_residence_card_number"></a>Ключевое слово \_ JP \_ \_ номер карточки проживания \_

```
Residence card number
Residence card no
Residence card #
在留カード番号
在留カード
在留番号
```

## <a name="japan-resident-registration-number"></a>Япон. номер резидентной регистрации

### <a name="format"></a>Формат

11 цифр

### <a name="pattern"></a>Шаблон

11 последовательных цифр

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_jp_resident_registration_number"></a>Ключевое слово \_ JP \_ номер резидентной \_ регистрации \_

```
Resident Registration Number
Residents Basic Registry Number
Resident Registration No.
Resident Register No.
Residents Basic Registry No.
Basic Resident Register No.
外国人登録証明書番号
証明書番号
登録番号
外国人登録証
```

## <a name="japan-social-insurance-number-sin"></a>Номер социального страхования для Японии (SIN)

### <a name="format"></a>Формат

7-12 цифр

### <a name="pattern"></a>Шаблон

7-12 цифр:

- четыре цифры
- дефис (необязательно)
- шесть цифр или
- 7-12 последовательных цифр

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_jp_sin"></a>Ключевое слово \_ JP \_ sin

```
Social Insurance No.
Social Insurance Num
Social Insurance Number
健康保険被保険者番号
健保番号
基礎年金番号
雇用保険被保険者番号
雇用保険番号
保険証番号
社会保険番号
社会保険No.
社会保険
介護保険
介護保険被保険者番号
健康保険被保険者整理番号
雇用保険被保険者整理番号
厚生年金
厚生年金被保険者整理番号
```

## <a name="malaysia-identification-card-number"></a>Номер идентификационной карточки Малайзии

### <a name="format"></a>Формат

12 цифр, содержащих необязательные дефисы

### <a name="pattern"></a>Шаблон

12 цифр:

- шесть цифр в формате YYMMDD, которые являются датой рождения
- тире (необязательно)
- 2-буквенный код места рождения
- тире (необязательно)
- три случайные цифры
- код пола из одной цифры

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_malaysia_id_card_number"></a>\_ \_ \_ Номер карточки по ключевому слову Малайзии \_

```
digital application card
i/c
i/c no
ic
ic no
ID card
identification Card
identity card
k/p
k/p no
kad akuan diri
kad aplikasi digital
kad pengenalan malaysia
kp
kp no
mykad
mykas
mykid
mypr
mytentera
malaysia identity card
malaysian identity card
nric
personal identification card
```

## <a name="netherlands-citizens-service-bsn-number"></a>Номер службы (BSN) для Нидерландов

### <a name="format"></a>Формат

8-девять цифр, содержащих необязательные пробелы

### <a name="pattern"></a>Шаблон

8 девяти цифр:

- три цифры
- пробел (необязательно)
- три цифры
- пробел (необязательно)
- две-три цифры

### <a name="keywords"></a>Ключевые слова

#### <a name="keywords_netherlands_eu_national_id_card"></a>Ключевые слова \_ Нидерландов — \_ \_ Национальный \_ идентификационный номер страны ЕС \_

```
bsn#
bsn
burgerservicenummer
citizen service number
person number
personal number
personal numeric code
person-related number
persoonlijk nummer
persoonlijke numerieke code
persoonsgebonden
persoonsnummer
sociaal-fiscaal nummer
social-fiscal number
sofi
sofinummer
uniek identificatienummer
uniek identiteitsnummer
unique identification number
unique identity number
uniqueidentityno#
```

## <a name="new-zealand-ministry-of-health-number"></a>Министерство номера работоспособности Новой Зеландии

### <a name="format"></a>Формат

три буквы: пробел (необязательно) и четыре цифры

### <a name="pattern"></a>Шаблон

- три буквы (без учета регистра), за исключением "I" и "O"
- пробел (необязательно)
- четыре цифры

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_nz_terms"></a>Условия использования ключевого слова \_ NZ \_

```
NHI
New Zealand
Health
treatment
National Health Index Number
nhi number
nhi no.
NHI#
National Health Index No.
National Health Index Id
National Health Index #
```

## <a name="norway-identification-number"></a>Идентификационный номер Норвегии

### <a name="format"></a>Формат

11 цифр

### <a name="pattern"></a>Шаблон

11 цифр:

- шесть цифр в формате ДДММИИ, которые являются датой рождения
- индивидуальный номер из трех цифр
- две контрольные цифры

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_norway_id_number"></a>Ключевое слово \_ Норвегии с \_ идентификатором \_

```
Personal identification number
Norwegian ID Number
ID Number
Identification
Personnummer
Fødselsnummer
```

## <a name="philippines-unified-multi-purpose-identification-number"></a>Однозначный унифицированный идентификационный номер Филиппины

### <a name="format"></a>Формат

12 цифр, разделенных дефисами

### <a name="pattern"></a>Шаблон

12 цифр:

- четыре цифры
- дефис
- семь цифр
- дефис
- одна цифра

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_philippines_id"></a>Ключевое слово \_ Филиппины \_ ID

```
Unified Multi-Purpose ID
UMID
Identity Card
Pinag-isang Multi-Layunin ID
```

## <a name="poland-identity-card"></a>Идентификационная карта для Польша

### <a name="format"></a>Формат

три буквы и шесть цифр

### <a name="pattern"></a>Шаблон

три буквы (без учета регистра), за которыми следуют шесть цифр

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_poland_national_id_passport_number"></a>Ключевое слово \_ Польша \_ \_ \_ номер удостоверения паспорта \_

```
Dowód osobisty
Numer dowodu osobistego
Nazwa i numer dowodu osobistego
Nazwa i nr dowodu osobistego
Nazwa i nr dowodu tożsamości
Dowód Tożsamości
dow. os.
```

## <a name="poland-national-id-pesel"></a>Польша Национальный идентификатор (ПЕСЕЛ)

### <a name="format"></a>Формат

11 цифр

### <a name="pattern"></a>Шаблон

- 6 цифр, представляющих дату рождения в формате YYMMDD
- 4 цифры
- 1 контрольная цифра

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_pesel_identification_number"></a>\_ \_ Идентификационный номер песел ключевого слова \_

```
dowód osobisty
dowódosobisty
niepowtarzalny numer
niepowtarzalnynumer
nr.-pesel
nr-pesel
numer identyfikacyjny
pesel
tożsamości narodowej
```

## <a name="poland-passport-number"></a>Номер службы "Польша Passport"

Эта сущность типа конфиденциальной информации включена в тип конфиденциальной информации паспорта ЕС и доступна в виде изолированной сущности типа конфиденциальной информации.

### <a name="format"></a>Формат

две буквы и семь цифр

### <a name="pattern"></a>Шаблон

Две буквы (без учета регистра), за которыми следуют семь цифр

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_poland_national_id_passport_number"></a>Ключевое слово \_ Польша \_ \_ \_ номер удостоверения паспорта \_

```
Numer paszportu
Nr. Paszportu
Paszport
```

## <a name="portugal-citizen-card-number"></a>Номер карточки гражданского по Португалия

### <a name="format"></a>Формат

восемь цифр

### <a name="pattern"></a>Шаблон

восемь цифр

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_portugal_citizen_card"></a>Карточка собственного ключевого слова \_ Португалия \_ \_

```
bilhete de identidade
cartão de cidadão
citizen card
document number
documento de identificação
ID number
identification no
identification number
identity card no
identity card number
national ID card
nic
número bi de portugal
número de identificação civil
número de identificação fiscal
número do documento
portugal bi number
```

## <a name="portugal-drivers-license-number"></a>Номер лицензии драйвера Португалия

Эта сущность типа конфиденциальной информации доступна только в типе конфиденциальной информации с номером лицензии для драйвера ЕС.

### <a name="format"></a>Формат

два шаблона — две буквы, за которыми следуют 5-8 цифр со специальными символами

### <a name="pattern"></a>Шаблон

Шаблон 1: две буквы, за которыми следует 5/6, со специальными символами:

- Две буквы (без учета регистра)
- дефис;
- Пять или шесть цифр
- Пробел
- Одна цифра

Шаблон 2. одна буква, за которой следуют 6/8 цифр со специальными символами:

- Одна буква (без учета регистра)
- дефис;
- Шесть или восемь цифр
- Пробел
- Одна цифра

### <a name="keywords"></a>Ключевые слова

#### <a name="keywords_eu_drivers_license_number"></a>\_ \_ Номер лицензии драйвера ЕС \_ по \_ ключевому слову

```
driverlic
driverlics
driver license
driver licenses
driverlicence
driverlicences
driver lic
driver lics
driver license
driver licenses
driver license
driver licenses
driverslic
driverslics
driverslicence
driverslicences
drivers license
drivers licenses
drivers lic
drivers lics
drivers license
drivers licenses
drivers license
drivers licenses
driver'lic
driver'lics
driver'license
driver'licenses
driver'license
driver'licenses
driver' lic
driver' lics
driver' license
driver' licenses
driver' license
driver' licenses
driver'slic
driver'slics
driver'license
driver'licenses
driver'license
driver'licenses
driver's lic
driver's lics
driver's license
driver's licenses
driver's license
driver's licenses
dl#
dls#
driverlic#
driverlics#
driver license#
driver licenses#
driverlicence#
driverlicences#
driver lic#
driver lics#
driver license#
driver licenses#
driver licenses#
driverslic#
driverslics#
drivers license#
drivers licenses#
driverslicence#
driverslicences#
drivers lic#
drivers lics#
drivers license#
drivers licenses#
drivers license#
drivers licenses#
driver'lic#
driver'lics#
driver'license#
driver'licenses#
driver'license#
driver'licenses#
driver' lic#
driver' lics#
driver' license#
driver' licenses#
driver' license#
driver' licenses#
driver'slic#
driver'slics#
driver'license#
driver'licenses#
driver'license#
driver'licenses#
driver's lic#
driver's lics#
driver's license#
driver's licenses#
driver's license#
driver's licenses#
driving license
driving license
dlno#
driv lic
driv licen
driv license
driv licenses
driv license
driv licenses
driver licen
drivers licen
driver's licen
driving lic
driving licen
driving licenses
driving license
driving licenses
driving permit
dl no
dlno
dl number
```

#### <a name="keywords_portugal_eu_drivers_license_number"></a>Ключевые \_ слова \_ \_ , \_ номер лицензии для драйвера ЕС \_

```
carteira de motorista
carteira motorista
carteira de habilitação
carteira habilitação
número de licença
número licença
permissão de condução
permissão condução
Licença condução Portugal
carta de condução
```

## <a name="saudi-arabia-national-id"></a>Национальный идентификатор Саудовская Аравия

### <a name="format"></a>Формат

10 знака

### <a name="pattern"></a>Шаблон

10 последовательных цифр

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_saudi_arabia_national_id"></a>Ключевое слово \_ Саудовская \_ Аравия \_ National \_ ID

```
Identification Card
I card number
ID number
الوطنية الهوية بطاقة رقم
```

## <a name="singapore-national-registration-identity-card-nric-number"></a>(Сингапур) номер национальной регистрационной карточки (НРИК)

### <a name="format"></a>Формат

девять букв и цифр

### <a name="pattern"></a>Шаблон

- девять букв и цифр:
- буква &quot; F &quot; , &quot; G &quot; , &quot; S &quot; или &quot; T &quot; (без учета регистра)
- семь цифр
- цифра алфавитной проверки

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_singapore_nric"></a>Ключевое слово \_ Сингапур \_ нрик

```
National Registration Identity Card
Identity Card Number
NRIC
IC
Foreign Identification Number
FIN
身份证
身份證
```

## <a name="south-africa-identification-number"></a>Идентификационный номер Южной Африки

### <a name="format"></a>Формат

13 цифр, которые могут содержать пробелы

### <a name="pattern"></a>Шаблон

13 цифр:

- шесть цифр в формате YYMMDD, которые являются датой рождения
- четыре цифры
- однозначный индикатор граждана
- цифра &quot; 8 &quot; или &quot; 9&quot;
- одна цифра, которая является цифрой контрольной суммы

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_south_africa_identification_number"></a>\_ \_ \_ Идентификационный номер Южно-Африканского зарезервированного слова \_

```
Identity card
ID
Identification
```

## <a name="south-korea-resident-registration-number"></a>Номер резидентной регистрации Южной Кореи

### <a name="format"></a>Формат

13 цифр, содержащих дефис

### <a name="pattern"></a>Шаблон

13 цифр:

- шесть цифр в формате YYMMDD, которые являются датой рождения
- дефис
- одна цифра, определяемая столетием и пол
- 4-значный код региона рождения
- одна цифра, используемая для различения людей, для которых предыдущие числа идентичны
- контрольная цифра.

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_south_korea_resident_number"></a>\_ \_ \_ Резидентный номер Южная Корея по ключевому слову \_

```
National ID card
Citizen's Registration Number
Jumin deungnok beonho
RRN
주민등록번호
```

## <a name="spain-social-security-number-ssn"></a>Номер социального страхования (SSN) Испания

Эта сущность типа конфиденциальной информации включена в номер социального страхования ЕС или эквивалентный тип конфиденциальной информации и доступна в виде изолированной сущности типа конфиденциальной информации.

### <a name="format"></a>Формат

11-12 цифр

### <a name="pattern"></a>Шаблон

11-12 цифр:

- две цифры
- косая черта (необязательно)
- от семи до восьми цифр
- косая черта (необязательно)
- две цифры

### <a name="keywords"></a>Ключевые слова

Нет

## <a name="sweden-national-id"></a>Швеция Национальный идентификатор

### <a name="format"></a>Формат

10 или 12 цифр и необязательный разделитель

### <a name="pattern"></a>Шаблон

10 или 12 цифр и необязательный разделитель:

- две цифры (необязательно)
- 6 цифр в формате даты YYMMDD
- разделитель &quot; - &quot; или &quot; + &quot; (необязательно)
- четыре цифры

### <a name="keywords"></a>Ключевые слова

#### <a name="keywords_swedish_national_identifier"></a>Ключевые \_ слова \_ шведский \_ идентификатор национального языка

```
ID no
ID number
ID#
identification no
identification number
identifikationsnumret#
identifikationsnumret
identitetshandling
identity document
identity no
identity number
id-nummer
personal ID
personnummer#
personnummer
skatteidentifikationsnummer
```

## <a name="sweden-passport-number"></a>Номер паспорта для Швеции

Эта сущность типа конфиденциальной информации включена в тип конфиденциальной информации паспорта ЕС и доступна в виде изолированной сущности типа конфиденциальной информации.

### <a name="format"></a>Формат

восемь цифр

### <a name="pattern"></a>Шаблон

восемь последовательных цифр

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_sweden_passport"></a>Ключевое слово \_ Швеция \_ Passport

```
visa requirements
Alien Registration Card
Schengen visas
Schengen visa
Visa Processing
Visa Type
Single Entry
Multiple Entries
G3 Processing Fees
```

#### <a name="keyword_passport"></a>Паспорт ключевого слова \_

```
Passport Number
Passport No
Passport #
Passport#
PassportID
Passport no
passport number
パスポート
パスポート番号
パスポートのNum
パスポート＃
Numéro de passeport
Passeport n °
Passeport Non
Passeport #
Passeport#
PasseportNon
Passeportn °
```

## <a name="swift-code"></a>Код SWIFT

### <a name="format"></a>Формат

четыре буквы, за которыми следует 5-31 букв или цифр

### <a name="pattern"></a>Шаблон

четыре буквы, за которыми следует 5-31 букв или цифр:

- код банка из четырех букв (без учета регистра)
- дополнительное пространство
- 4-28. буквы или цифры (Базовый номер банковского счета (ББАН))
- дополнительное пространство
- от одной до трех букв или цифр (остаток от ББАН)

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_swift"></a>Ключевое слово \_ SWIFT

```
international organization for standardization 9362
iso 9362
iso9362
swift#
swift code
swift number
swiftroutingnumber
swift code
swift number #
swift routing number
bic number
bic code
bic #
bic#
bank identifier code
Organization internationale de normalization 9362
rapide #
code SWIFT
le numéro de swift
swift numéro d'acheminement
le numéro BIC
#BIC
code identificateur de banque
SWIFTコード
SWIFT番号
BIC番号
BICコード
SWIFT コード
SWIFT 番号
BIC 番号
BIC コード
金融機関識別コード
金融機関コード
銀行コード
```

## <a name="taiwan-national-identification-number"></a>Национальный идентификационный номер Тайваня

### <a name="format"></a>Формат

одна буква (на английском языке), за которой следуют девять цифр

### <a name="pattern"></a>Шаблон

одна буква (на английском языке), за которой следует девять цифр:

- одна буква (на английском языке, без учета регистра)
- цифра &quot; 1 &quot; или &quot; 2&quot;
- восемь цифр

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_taiwan_national_id"></a>Ключевое слово \_ Тайвань \_ National \_ ID

```
身份證字號
身份證
身份證號碼
身份證號
身分證字號
身分證
身分證號碼
身份證號
身分證統一編號
國民身分證統一編號
簽名
蓋章
簽名或蓋章
簽章
```

## <a name="taiwan-passport-number"></a>Номер паспорта для Тайваня

### <a name="format"></a>Формат

- номер биометрической службы Passport: девять цифр
- номер паспорта биометрической метрики: девять цифр

### <a name="pattern"></a>Шаблон

номер биометрической службы Passport:

- символ &quot; 3&quot;
- восемь цифр

Биометрический номер паспорта:

- девять цифр

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_taiwan_passport"></a>Ключевое слово \_ Тайвань \_ Passport

```
ROC passport number
Passport number
Passport no
Passport Num
Passport #
护照
中華民國護照
Zhōnghuá Mínguó hùzhào
```

## <a name="taiwan-resident-certificate-arctarc-number"></a>Номер резидентного сертификата (ARC/ТАРК)

### <a name="format"></a>Формат

10 букв и цифр

### <a name="pattern"></a>Шаблон

10 букв и цифр:

- две буквы (без учета регистра)
- восемь цифр

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_taiwan_resident_certificate"></a>\_ \_ Резидентный сертификат Тайваня по ключевому слову \_

```
Resident Certificate
Resident Cert
Resident Cert.
Identification card
Alien Resident Certificate
ARC
Taiwan Area Resident Certificate
TARC
居留證
外僑居留證
台灣地區居留證
```

## <a name="uk-drivers-license-number"></a>Великобритании номер лицензии драйвера

Эта сущность типа конфиденциальной информации включена в тип конфиденциальной информации номера лицензии драйвера ЕС и доступна в виде изолированной сущности типа конфиденциальной информации.

### <a name="format"></a>Формат

Сочетание из 18 букв и цифр в указанном формате

### <a name="pattern"></a>Шаблон

18 букв и цифр:

- пять букв (без учета регистра) или цифра &quot; 9 &quot; вместо буквы
- одна цифра
- пять цифр в формате даты ММДДИ для даты рождения (седьмой символ увеличивается на 50, если драйвер — женщина, то есть 51 – 62 вместо 01 – 12).
- две буквы (без учета регистра) или цифра &quot; 9 &quot; вместо буквы
- пять цифр

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_uk_drivers_license"></a>\_Лицензия для \_ драйверов \_ Великобритании

```
DVLA
light vans
quad bikes
motor cars
125cc
sidecar
tricycles
motorcycles
photo card license
learner drivers
license holder
license holders
driving licenses
driving license
dual control car
```

## <a name="uk-electoral-roll-number"></a>Великобритании номер рулона предпочтений избирателей

### <a name="format"></a>Формат

две буквы, за которыми следует 1-4 цифр

### <a name="pattern"></a>Шаблон

две буквы (без учета регистра), за которыми следуют номера 1-4

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_uk_electoral"></a>Ключевое слово \_ UK \_ предпочтений избирателей

- Совет предварительного утверждения
- Форма предварительного утверждения
- предпочтений избирателей регистр
- предпочтений избирателей

## <a name="uk-national-health-service-number"></a>Великобритании номер службы национальной работоспособности

### <a name="format"></a>Формат

10-17 цифр, разделенных пробелами

### <a name="pattern"></a>Шаблон

10-17 цифр:

- либо три, либо 10 цифр
- пробел
- три цифры
- пробел
- четыре цифры

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_uk_nhs_number"></a>Ключевое слово \_ Великобритании \_ отделения \_ Number

```
national health service
nhs
health services authority
health authority
```

#### <a name="keyword_uk_nhs_number1"></a>Ключевое слово \_ UK \_ отделения \_ Число1

```
patient ID
patient identification
patient no
patient number
```

#### <a name="keyword_uk_nhs_number_dob"></a>Ключевое слово \_ UK \_ отделения \_ Number \_ DOB

```
GP
DOB
D.O.B
Date of Birth
Birth Date
```

## <a name="uk-national-insurance-number-nino"></a>Великобритании номер национальной страховки (Нино)

Эта сущность типа конфиденциальной информации включается в тип конфиденциальной информации о национальном номере страны ЕС и доступна в виде изолированной сущности типа конфиденциальной информации.

### <a name="format"></a>Формат

семь символов или девять символов, разделенных пробелами или тире

### <a name="pattern"></a>Шаблон

два возможных шаблона:

- две буквы (допустимые Нинос используют только определенные символы в этом префиксе, которые проверяются в этом шаблоне, без учета регистра).
- шесть цифр
- "A", "B", "C" или "d" (как в префиксе, в суффиксе допускаются только некоторые символы; регистр не учитывается)

OR

- две буквы
- пробел или тире
- две цифры
- пробел или тире
- две цифры
- пробел или тире
- две цифры
- пробел или тире
- "A", "B", "C" или "d"


### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_uk_nino"></a>Ключевое слово \_ UK \_ Нино

```
national insurance number
national insurance contributions
protection act
insurance
social security number
insurance application
medical application
social insurance
medical attention
social security
Great Britain
NI Number
NI No.
NI #
NI#
insurance#
insurance number
national insurance#
nationalinsurancenumber
```

## <a name="uk-unique-taxpayer-reference-number"></a>Великобритании Уникальный номер ссылки налогоплательщика

Этот тип конфиденциальной информации доступен только для использования в:

- политики защиты от потери данных
- политики соответствия для взаимодействия
- Управление информацией
- Управление записями
- Microsoft Cloud App Security

### <a name="format"></a>Формат

10 цифр без пробелов и разделителей

### <a name="pattern"></a>Шаблон

10 знака

### <a name="keywords"></a>Ключевые слова

#### <a name="keywords_uk_eu_tax_file_number"></a>Ключевые \_ слова \_ \_ номер налогового \_ файла Великобритании ЕС \_

```
tax number
tax file
tax ID
tax identification no
tax identification number
tax no#
tax no
tax registration number
tax ID#
tax ID no#
tax ID number#
tax no#
tax number#
tax number
tin ID
tin no
tin#
```

## <a name="us-bank-account-number"></a>Номер банковского счета США

### <a name="format"></a>Формат

6-17 цифр

### <a name="pattern"></a>Шаблон

6-17 последовательных цифр

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_usa_bank_account"></a>Ключевое слово \_ США \_ банковским \_ счетом

```
Checking Account Number
Checking Account
Checking Account #
Checking Acct Number
Checking Acct #
Checking Acct No.
Checking Account No.
Bank Account Number
Bank Account #
Bank Acct Number
Bank Acct #
Bank Acct No.
Bank Account No.
Savings Account Number
Savings Account.
Savings Account #
Savings Acct Number
Savings Acct #
Savings Acct No.
Savings Account No.
Debit Account Number
Debit Account
Debit Account #
Debit Acct Number
Debit Acct #
Debit Acct No.
Debit Account No.
```

## <a name="us-drivers-license-number"></a>Номер лицензии для драйвера США

### <a name="format"></a>Формат

Зависит от состояния

### <a name="pattern"></a>Шаблон

зависит от состояния, например Нью-Йорк:

- девять цифр, отформатированных как DDD, будут соответствовать.
- девять цифр, например ддддддддд, не совпадают.

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_us_drivers_license_abbreviations"></a>Ключевые слова для \_ драйверов US. \_ \_ \_ сокращенные названия лицензий

```
DL
DLS
CDL
CDLS
ID
IDs
DL#
DLS#
CDL#
CDLS#
ID#
IDs#
ID number
ID numbers
LIC
LIC#
```

#### <a name="keyword_us_drivers_license"></a>Лицензия на ключевые слова \_ US \_ Driver \_

```
DriverLic
DriverLics
DriverLicense
DriverLicenses
Driver Lic
Driver Lics
Driver License
Driver Licenses
DriversLic
DriversLics
DriversLicense
DriversLicenses
Drivers Lic
Drivers Lics
Drivers License
Drivers Licenses
Driver'Lic
Driver'Lics
Driver'License
Driver'Licenses
Driver' Lic
Driver' Lics
Driver' License
Driver' Licenses
Driver'sLic
Driver'sLics
Driver'sLicense
Driver'sLicenses
Driver's Lic
Driver's Lics
Driver's License
Driver's Licenses
identification number
identification numbers
identification #
ID card
ID cards
identification card
identification cards
DriverLic#
DriverLics#
DriverLicense#
DriverLicenses#
Driver Lic#
Driver Lics#
Driver License#
Driver Licenses#
DriversLic#
DriversLics#
DriversLicense#
DriversLicenses#
Drivers Lic#
Drivers Lics#
Drivers License#
Drivers Licenses#
Driver'Lic#
Driver'Lics#
Driver'License#
Driver'Licenses#
Driver' Lic#
Driver' Lics#
Driver' License#
Driver' Licenses#
Driver'sLic#
Driver'sLics#
Driver'sLicense#
Driver'sLicenses#
Driver's Lic#
Driver's Lics#
Driver's License#
Driver's Licenses#
ID card#
ID cards#
identification card#
identification cards#
```

#### <a name="keyword_state_name_drivers_license_name"></a>Ключевое слово \_ [State \_ name] \_ драйверы \_ \_ имя лицензии

- сокращение штата (например, Нью &quot; &quot; -Йорк)
- имя состояния (например, Нью &quot; -Йорк &quot; )

## <a name="us-individual-taxpayer-identification-number-itin"></a>Индивидуальный идентификационный номер налогоплательщика (ИТИН) США

### <a name="format"></a>Формат

девять цифр, которые начинаются с &quot; 9 &quot; и содержат &quot; 7 &quot; или 8 &quot; в &quot; качестве четвертой цифры, при необходимости отформатированные пробелами или тире

### <a name="pattern"></a>Шаблон

формате

- цифра &quot; 9&quot;
- две цифры
- пробел или тире
- &quot;7 &quot; или &quot; 8&quot;
- цифра
- пробел или тире
- четыре цифры

неформатированного

- цифра &quot; 9&quot;
- две цифры
- &quot;7 &quot; или &quot; 8&quot;
- пять цифр

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_itin"></a>Ключевое слово \_ Итин

```
taxpayer
tax ID
tax identification
itin
i.t.i.n.
ssn
tin
social security
tax payer
itins
tax ID
individual taxpayer
```

## <a name="us-social-security-number-ssn"></a>Номер социального страхования США (SSN)

### <a name="format"></a>Формат

девять цифр, которые могут быть в форматированном или неформатированном шаблоне

>[!Note]
> Если выдается до середины 2011, SSN имеет строгое форматирование, где определенные части числа должны находиться в пределах определенных диапазонов (но нет контрольной суммы).
>

### <a name="pattern"></a>Шаблон

четыре функции ищут в службах SSNs четыре разных шаблона:

- Функция Func \_ ssn находит в службах. 2011 строгое форматирование с тире или пробелами (ddd-dd-dddd или ddd dd dddd).
- \_Неформатированный \_ ssn находит SSNs с предварительным 2011 строгим форматированием, не отформатированным как девять последовательных цифр (ддддддддд)
- Функция Func с \_ произвольным \_ форматом \_ находит пост-2011 SSNs, отформатированные с помощью дефисов и пробелов (ddd-dd-dddd или ddd dd dddd).
- Функция Func \_ случайного \_ неформатированного \_ страхового номера находит пост-2011 SSNs, которые имеют неформатированную девять последовательных цифр (ддддддддд)

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_ssn"></a>Ключевое слово \_ ssn

```
SSA Number
social security number
social security #
social security#
social security no
Social Security#
Soc Sec
SSN
SSNS
SSN#
SS#
SSID
```

## <a name="us--uk-passport-number"></a>США И ВЕЛИКОБРИТАНИЯ номер паспорта

Великобритания сущность с конфиденциальными данными о номере паспорта доступна в типе конфиденциальной информации ЕС Passport Number и доступна в виде изолированной сущности типа конфиденциальной информации.

### <a name="format"></a>Формат

девять цифр

### <a name="pattern"></a>Шаблон

девять последовательных цифр

### <a name="keywords"></a>Ключевые слова

#### <a name="keyword_passport"></a>Паспорт ключевого слова \_

```
Passport Number
Passport No
Passport #
Passport#
PassportID
Passport no
passport number
パスポート
パスポート番号
パスポートのNum
パスポート＃
Numéro de passeport
Passeport n °
Passeport Non
Passeport #
Passeport#
PasseportNon
Passeportn °
```
