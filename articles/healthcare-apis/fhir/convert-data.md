---
title: Преобразование данных для Azure API для FHIR
description: Используйте конечную точку $convert-data и шаблоны customize-converter, чтобы преобразовать данные в Azure API для FHIR.
services: healthcare-apis
author: ranvijaykumar
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: overview
ms.date: 01/19/2021
ms.author: ranku
ms.openlocfilehash: 2a34cfee57ecc1870c420c4c0f3c9261aa02f192
ms.sourcegitcommit: 4bda786435578ec7d6d94c72ca8642ce47ac628a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/16/2021
ms.locfileid: "103490931"
---
# <a name="how-to-convert-data-to-fhir-preview"></a>Преобразование данных для FHIR (предварительная версия)

> [!IMPORTANT]
> Эта возможность предоставляется в общедоступной предварительной версии без соглашения об уровне обслуживания. Не рекомендуем использовать ее в рабочей среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены. Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Пользовательская конечная точка $convert-data в Azure API для FHIR предназначена для преобразования данных из разных форматов в формат FHIR. В ней применяются модуль шаблонов Liquid и шаблоны из проекта [преобразователя FHIR](https://github.com/microsoft/FHIR-Converter) в качестве шаблонов по умолчанию. Вы можете изменить эти шаблоны в соответствии с конкретными потребностями. Сейчас поддерживается преобразование из HL7v2 в FHIR.

## <a name="use-the-convert-data-endpoint"></a>Использование конечной точки $convert-data

`https://<<FHIR service base URL>>/$convert-data`

$convert-data принимает в тексте запроса ресурс [Parameter](http://hl7.org/fhir/parameters.html), как описано ниже.

**Ресурс Parameter:**

| имени параметра      | Описание | Допустимые значения |
| ----------- | ----------- | ----------- |
| inputData      | Преобразуемые данные. | Допустимое значение с типом данных JSON String (строка)|
| inputDataType   | Тип входных данных. | ```HL7v2``` |
| templateCollectionReference | Ссылка на коллекцию шаблонов. Это может быть ссылка на **шаблоны по умолчанию** или на пользовательский образ шаблонов, зарегистрированный в Azure API для FHIR. Ниже описано, как правильно настроить шаблоны, разместить их в Реестре контейнеров и зарегистрировать в Azure API для FHIR.  | ```microsofthealth/fhirconverter:default```, \<RegistryServer\>/\<imageName\>@\<imageDigest\> |
| rootTemplate | Корневой шаблон, используемый при преобразовании данных. | ```ADT_A01```, ```OML_O21```, ```ORU_R01```, ```VXU_V04``` |  

> [!WARNING]
> Шаблоны по умолчанию помогут вам быстро приступить к работе. Но они могут измениться при очередном обновлении Azure API для FHIR. Чтобы обеспечить согласованное преобразование данных в разных версиях Azure API для FHIR, нужно разместить собственную копию шаблонов в Реестре контейнеров Azure, зарегистрировать их в Azure API для FHIR и применять в вызовах API, как описано ниже.

**Пример запроса:**

```json
{
    "resourceType": "Parameters",
    "parameter": [
        {
            "name": "inputData",
            "valueString": "MSH|^~\\&|SIMHOSP|SFAC|RAPP|RFAC|20200508131015||ADT^A01|517|T|2.3|||AL||44|ASCII\nEVN|A01|20200508131015|||C005^Whittingham^Sylvia^^^Dr^^^DRNBR^PRSNL^^^ORGDR|\nPID|1|3735064194^^^SIMULATOR MRN^MRN|3735064194^^^SIMULATOR MRN^MRN~2021051528^^^NHSNBR^NHSNMBR||Kinmonth^Joanna^Chelsea^^Ms^^CURRENT||19870624000000|F|||89 Transaction House^Handmaiden Street^Wembley^^FV75 4GJ^GBR^HOME||020 3614 5541^HOME|||||||||C^White - Other^^^||||||||\nPD1|||FAMILY PRACTICE^^12345|\nPV1|1|I|OtherWard^MainRoom^Bed 183^Simulated Hospital^^BED^Main Building^4|28b|||C005^Whittingham^Sylvia^^^Dr^^^DRNBR^PRSNL^^^ORGDR|||CAR|||||||||16094728916771313876^^^^visitid||||||||||||||||||||||ARRIVED|||20200508131015||"
        },
        {
            "name": "inputDataType",
            "valueString": "Hl7v2"
        },
        {
            "name": "templateCollectionReference",
            "valueString": "microsofthealth/fhirconverter:default"
        },
        {
            "name": "rootTemplate",
            "valueString": "ADT_A01"
        }
    ]
}
```

**Пример ответа:**

```json
{
  "resourceType": "Bundle",
  "type": "transaction",
  "entry": [
    {
      "fullUrl": "urn:uuid:9d697ec3-48c3-3e17-db6a-29a1765e22c6",
      "resource": {
        "resourceType": "Patient",
        "id": "9d697ec3-48c3-3e17-db6a-29a1765e22c6",
        ...
        ...
      "request": {
        "method": "PUT",
        "url": "Location/50becdb5-ff56-56c6-40a1-6d554dca80f0"
      }
    }
  ]
}
```

## <a name="customize-templates"></a>Настройка шаблонов

Для настройки шаблонов под ваши потребности можно использовать [расширение преобразователя FHIR](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-health-fhir-converter) для Visual Studio Code. Это расширение поддерживает интерактивное редактирование и упрощает скачивание опубликованных корпорацией Майкрософт шаблонов и примеров данных. Дополнительные сведения см. во встроенной документации расширения.

## <a name="host-and-use-templates"></a>Размещение и использование шаблонов

Мы настоятельно рекомендуем всегда размещать собственную копию шаблонов в Реестре контейнеров Azure. Размещение собственной копии шаблонов и применение их в операции $convert-data включает следующие четыре операции:

1. передача шаблонов в Реестр контейнеров Azure;
1. включение управляемого удостоверения в экземпляре Azure API для FHIR;
1. предоставление управляемому удостоверению Azure API для FHIR доступа к Реестру контейнеров Azure;
1. регистрация серверов Реестра контейнеров Azure в Azure API для FHIR.

### <a name="push-templates-to-azure-container-registry"></a>Отправка шаблонов в Реестр контейнеров Azure

Создав экземпляр Реестра контейнеров Azure, вы можете применить команду _FHIR Converter: Push Templates_ из [расширения преобразователя FHIR](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-health-fhir-converter), чтобы отправить измененные шаблоны в Реестр контейнеров Azure. Кроме того, для этой цели можно использовать [средство командной строки для управления шаблонами](https://github.com/microsoft/FHIR-Converter/blob/main/docs/TemplateManagementCLI.md).

### <a name="enable-managed-identity-on-azure-api-for-fhir"></a>Включение управляемого удостоверения в Azure API для FHIR

Перейдите к экземпляру службы Azure API для FHIR на портале Azure и выберите колонку **Удостоверение**.
Установите здесь состояние **Включено**, чтобы включить управляемое удостоверение в Azure API для FHIR.

![Включение управляемого удостоверения](media/convert-data/fhir-mi-enabled.png)

### <a name="provide-access-of-the-acr-to-azure-api-for-fhir"></a>Предоставление Azure API для FHIR доступа к Реестру контейнеров Azure

Перейдите к колонке "Управление доступом (IAM)" в экземпляре Реестра контейнеров Azure и щелкните _Добавление назначений ролей_.

![Назначение ролей в Реестре контейнеров Azure](media/convert-data/fhir-acr-role-assignment.png)

Предоставьте экземпляру службы Azure API для FHIR роль AcrPull.

![Добавление роли](media/convert-data/fhir-acr-role-add.png)

### <a name="register-the-acr-servers-in-azure-api-for-fhir"></a>Регистрация серверов Реестра контейнеров Azure в Azure API для FHIR.

Сервер записи контроля доступа можно зарегистрировать с помощью портал Azure или с помощью интерфейса командной строки.

#### <a name="registering-the-acr-server-using-azure-portal"></a>Регистрация сервера записи контроля доступа с помощью портал Azure
Перейдите в колонку _артефакты_ в разделе _Преобразование данных_ в API Azure для экземпляра FHIR. Вы увидите список зарегистрированных в настоящее время серверов записи контроля доступа. Щелкните _Добавить_ и выберите сервер реестра в раскрывающемся списке. Чтобы регистрация вступила в силу, необходимо нажать кнопку _сохранить_ . Применение изменений и перезапуск экземпляра может занять несколько минут.

#### <a name="registering-the-acr-server-using-cli"></a>Регистрация сервера записи контроля доступа с помощью интерфейса командной строки
Вы можете зарегистрировать в Azure API для FHIR до двадцати серверов Реестра контейнеров Azure.

При желании можно установить CLI для API здравоохранения с помощью Azure PowerShell:

```powershell
az extension add -n healthcareapis
```

Зарегистрируйте серверы Реестра контейнеров Azure в Azure API для FHIR по аналогии со следующими примерами.

##### <a name="register-a-single-acr-server"></a>Регистрация одного сервера Реестра контейнеров Azure

```powershell
az healthcareapis acr add --login-servers "fhiracr2021.azurecr.io" --resource-group fhir-test --resource-name fhirtest2021
```

##### <a name="register-multiple-acr-servers"></a>Регистрация нескольких серверов Реестра контейнеров Azure

```powershell
az healthcareapis acr add --login-servers "fhiracr2021.azurecr.io fhiracr2020.azurecr.io" --resource-group fhir-test --resource-name fhirtest2021
```

### <a name="verify"></a>Проверка

Выполните вызов к API $convert-data, указав ссылку на шаблон в параметре templateCollectionReference.

`<RegistryServer>/<imageName>@<imageDigest>`
