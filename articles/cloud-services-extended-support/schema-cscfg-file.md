---
title: Схема определения (файл. cscfg) облачных служб Azure (Расширенная поддержка) | Документация Майкрософт
description: Сведения, относящиеся к схеме определения для облачных служб (Расширенная поддержка)
ms.topic: article
ms.service: cloud-services-extended-support
ms.date: 10/14/2020
author: gachandw
ms.author: gachandw
ms.reviewer: mimckitt
ms.custom: ''
ms.openlocfilehash: 3aac5ca8d146f0cd74f483846d22fdf0052f86f2
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98744695"
---
# <a name="azure-cloud-services-extended-support-config-schema-cscfg-file"></a>Схема конфигурации облачных служб Azure (Расширенная поддержка) (CSCFG-файл)

Файл конфигурации службы определяет число экземпляров роли, которые развертываются для каждой роли в службе, значения всех параметров конфигурации и отпечатки всех сертификатов, связанных с ролью. Если служба является частью виртуальной сети, нужно указать сведения о конфигурации сети в файле конфигурации службы и в файле конфигурации виртуальной сети. Расширение по умолчанию для файла конфигурации службы — CSCFG.

Модель службы описывается в [схеме определения облачной службы (Расширенная поддержка)](schema-csdef-file.md).

По умолчанию файл схемы конфигурации системы диагностики Azure устанавливается в каталог `C:\Program Files\Microsoft SDKs\Windows Azure\.NET SDK\<version>\schemas`. Замените `<version>` установленной версией [пакета SDK для Azure](https://azure.microsoft.com/downloads/).

Дополнительные сведения о настройке ролей для службы см. в статье [Что такое модель облачных служб и как создать ее пакет?](../cloud-services/cloud-services-model-and-package.md)

## <a name="basic-service-configuration-schema"></a>Схема конфигурации службы "базовый"
Ниже приводится базовый формат файла конфигурации службы.

```xml
<ServiceConfiguration serviceName="<service-name>" osFamily="<osfamily-number>" osVersion="<os-version>" schemaVersion="<schema-version>">

  <Role …>
    …
  </Role>

  <NetworkConfiguration>
    …
  </NetworkConfiguration>

</ServiceConfiguration>
```

## <a name="schema-definitions"></a>Определения схем
В следующих статьях описаны схемы для элемента `ServiceConfiguration`:

- [Схема роли](schema-cscfg-role.md)
- [Схема NetworkConfiguration](schema-cscfg-networkconfiguration.md)

## <a name="service-configuration-namespace"></a>Пространство имен конфигурации службы
Для файла конфигурации диагностики используется пространство имен XML `http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceConfiguration`.

##  <a name="serviceconfiguration-element"></a><a name="ServiceConfiguration"></a> ServiceConfiguration, элемент
Элемент `ServiceConfiguration` занимает верхний уровень в файле конфигурации службы.

В таблице ниже описаны атрибуты элемента `ServiceConfiguration`. Значения всех атрибутов имеют строковый тип.

| Атрибут | Описание |
| --------- | ----------- |
|serviceName|Обязательный. Имя облачной службы. Указанное здесь имя должно соответствовать имени, указанному в файле определения службы.|
|osFamily|Необязательный параметр. Указывает гостевую ОС, которая будет выполняться в экземплярах ролей в облачной службе. Информацию о поддерживаемых выпусках гостевой ОС вы найдете в [таблице совместимости выпусков гостевых ОС Azure и пакетов SDK](../cloud-services/cloud-services-guestos-update-matrix.md).<br /><br /> Если вы не включите значение `osFamily` и не укажете конкретную версию гостевой ОС через атрибут `osVersion`, по умолчанию используется значение 1.|
|osVersion|Необязательный параметр. Указывает версию гостевой ОС, которая будет выполняться в экземплярах ролей в облачной службе. Дополнительную информацию о версиях гостевой ОС вы найдете в [таблице совместимости выпусков гостевых ОС Azure и пакетов SDK](../cloud-services/cloud-services-guestos-update-matrix.md).<br /><br /> Вы можете настроить автоматическое обновление гостевой ОС до последней версии. Для этого присвойте атрибуту `osVersion` значение `*`. Если задано значение `*`, экземпляры роли развертываются с последней версией гостевой ОС из указанного семейства ОС, и эти версии автоматически обновляются при выходе новых версий гостевой ОС.<br /><br /> Чтобы вручную указать конкретную версию, используйте `Configuration String` (строку конфигурации) из раздела **о будущих, текущих и переходных версиях гостевой ОС** в [таблице совместимости выпусков гостевых ОС Azure и пакетов SDK](../cloud-services/cloud-services-guestos-update-matrix.md).<br /><br /> Для атрибута `osVersion` по умолчанию используется значение `*`.|
|schemaVersion|Необязательный параметр. Указывает версию схемы конфигурации службы. Версия схемы позволяет Visual Studio выбрать правильные средства пакета SDK для использования при проверке схемы, если установлено одновременно несколько версий пакета SDK. Дополнительную информацию о схеме и совместимости версий вы найдете в [таблице совместимости выпусков гостевых ОС Azure и пакетов SDK](../cloud-services/cloud-services-guestos-update-matrix.md).|

Файл конфигурации службы должен содержать один элемент `ServiceConfiguration`. Элемент `ServiceConfiguration` может содержать любое количество элементов `Role` и не более одного элемента `NetworkConfiguration`.

## <a name="see-also"></a>См. также раздел

[Схема определения облачных служб Azure (Расширенная поддержка) (файл CSDEF)](schema-csdef-file.md)