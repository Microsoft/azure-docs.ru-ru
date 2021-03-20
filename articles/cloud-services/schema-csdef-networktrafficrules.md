---
title: Схема DEF. NetworkTrafficRules (классическая модель) облачных служб Azure | Документация Майкрософт
description: Сведения о NetworkTrafficRules, которые ограничивают роли, которые могут получать доступ к внутренним конечным точкам роли. Он сочетается с ролями в файле определения службы.
ms.topic: article
ms.service: cloud-services
ms.date: 10/14/2020
ms.author: tagore
author: tanmaygore
ms.reviewer: mimckitt
ms.custom: ''
ms.openlocfilehash: 2c8ab53068b71652d03d03bf79a224fe5e34dff3
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98739774"
---
# <a name="azure-cloud-services-classic-definition-networktrafficrules-schema"></a>Схема NetworkTrafficRules определения облачных служб Azure (классическая модель)

> [!IMPORTANT]
> [Облачные службы Azure (Расширенная поддержка)](../cloud-services-extended-support/overview.md) — это новая модель развертывания на основе Azure Resource Manager для продукта облачных служб Azure.После этого изменения облачные службы Azure, работающие в модели развертывания на основе Service Manager Azure, были переименованы как облачные службы (классические), и все новые развертывания должны использовать [облачные службы (Расширенная поддержка)](../cloud-services-extended-support/overview.md).

Узел `NetworkTrafficRules` — это необязательный элемент в файле определения службы, который указывает, как роли взаимодействуют друг с другом. Он ограничивает множество ролей, которые могут обращаться ко внутренним конечным точкам определенной роли. `NetworkTrafficRules` не является отдельным элементом. Он объединяется с двумя или более ролями в файле определения службы.

По умолчанию определения службы хранятся в файле с расширением .csdef.

> [!NOTE]
>  Узел `NetworkTrafficRules` доступен только при использовании пакета Azure SDK версии 1.3 или выше.

## <a name="basic-service-definition-schema-for-the-network-traffic-rules"></a>Базовая схема определения службы для правил сетевого трафика
Ниже приведен базовый формат файла определения службы, содержащий определения сетевого трафика.

```xml
<ServiceDefinition …>
   <NetworkTrafficRules>
      <OnlyAllowTrafficTo>
         <Destinations>
            <RoleEndpoint endpointName="<name-of-the-endpoint>" roleName="<name-of-the-role-containing-the-endpoint>"/>
         </Destinations>
         <AllowAllTraffic/>
         <WhenSource matches="[AnyRule]">
            <FromRole roleName="<name-of-the-role-to-allow-traffic-from>"/>
         </WhenSource>
      </OnlyAllowTrafficTo>
   </NetworkTrafficRules>
</ServiceDefinition>
```

## <a name="schema-elements"></a>Элементы схемы
Узел `NetworkTrafficRules` файла определения службы содержит следующие элементы, которые подробно описаны в этой статье:

[Элемент NetworkTrafficRules](#NetworkTrafficRules)

[OnlyAllowTrafficTo, элемент](#OnlyAllowTrafficTo)

[Элемент Destinations](#Destinations)

[Элемент RoleEndpoint](#RoleEndpoint)

Элемент AllowAllTraffic

[Элемент WhenSource](#WhenSource)

[Элемент FromRole](#FromRole)

##  <a name="networktrafficrules-element"></a><a name="NetworkTrafficRules"></a> NetworkTrafficRules, элемент
Элемент `NetworkTrafficRules` определяет роли, которые могут обмениваться данными с определенной конечной точкой в другой роли. Служба может содержать одно определение `NetworkTrafficRules`.

##  <a name="onlyallowtrafficto-element"></a><a name="OnlyAllowTrafficTo"></a> Элемент OnlyAllowTrafficTo
Элемент `OnlyAllowTrafficTo` описывает коллекцию целевых конечных точек и ролей, которые могут обмениваться данными с ними. Можно указать несколько узлов `OnlyAllowTrafficTo`.

##  <a name="destinations-element"></a><a name="Destinations"></a> Элемент Destinations
Элемент `Destinations` описывает коллекцию элементов RoleEndpoint, с которыми можно обмениваться данными.

##  <a name="roleendpoint-element"></a><a name="RoleEndpoint"></a> RoleEndpoint, элемент
Элемент `RoleEndpoint` описывает конечную точку в роли, с которой можно обмениваться данными. Можно указать несколько элементов `RoleEndpoint`, если в роли существует несколько конечных точек.

| attribute      | Тип     | Описание |
| -------------- | -------- | ----------- |
| `endpointName` | `string` | Обязательный. Имя конечной точки, к которой разрешается трафик.|
| `roleName`     | `string` | Обязательный. Имя веб-роли, с которой разрешается обмен данными.|

## <a name="allowalltraffic-element"></a>Элемент AllowAllTraffic
Элемент `AllowAllTraffic` — это правило, которое разрешает всем ролям обмениваться данными с конечными точками, определенными в узле `Destinations`.

##  <a name="whensource-element"></a><a name="WhenSource"></a> WhenSource, элемент
Элемент `WhenSource` описывает коллекцию ролей, которые могут обмениваться данными с конечными точками, определенными в узле `Destinations`.

| attribute | Тип     | Описание |
| --------- | -------- | ----------- |
| `matches` | `string` | Обязательный. Указывает правило, применяемое при разрешении обмена данными. Сейчас единственное допустимое значение — `AnyRule`.|
  
##  <a name="fromrole-element"></a><a name="FromRole"></a> FromRole, элемент
Элемент `FromRole` указывает роли, которые могут обмениваться данными с конечными точками, определенными в узле `Destinations`. Можно указать несколько элементов `FromRole`, если существует несколько ролей, которые могут обмениваться данными с конечными точками.

| attribute  | Тип     | Описание |
| ---------- | -------- | ----------- |
| `roleName` | `string` | Обязательный. Имя роли, из которой разрешается обмен данными.|

## <a name="see-also"></a>См. также:
[Схема определения облачных служб (классических)](schema-csdef-file.md).




