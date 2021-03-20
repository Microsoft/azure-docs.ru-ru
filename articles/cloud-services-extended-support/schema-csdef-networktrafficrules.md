---
title: Облачные службы Azure (Расширенная поддержка) DEF. NetworkTrafficRules Schema | Документация Майкрософт
description: Сведения о правилах сетевого трафика, связанных с облачными службами (Расширенная поддержка)
ms.topic: article
ms.service: cloud-services-extended-support
ms.date: 10/14/2020
author: gachandw
ms.author: gachandw
ms.reviewer: mimckitt
ms.custom: ''
ms.openlocfilehash: 0064794701e87419da086c458673f7ccee4f37dd
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98744795"
---
# <a name="azure-cloud-services-extended-support-definition-networktrafficrules-schema"></a>Схема NetworkTrafficRules определения облачных служб Azure (Расширенная поддержка)

Узел `NetworkTrafficRules` — это необязательный элемент в файле определения службы, который указывает, как роли взаимодействуют друг с другом. Он ограничивает множество ролей, которые могут обращаться ко внутренним конечным точкам определенной роли. `NetworkTrafficRules` не является отдельным элементом. Он объединяется с двумя или более ролями в файле определения службы.

Расширение по умолчанию для файла определения службы — CSDEF.

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

[Элемент AllowAllTraffic](#AllowAllTraffic)

[Элемент WhenSource](#WhenSource)

[Элемент FromRole](#FromRole)

##  <a name="networktrafficrules-element"></a><a name="NetworkTrafficRules"></a> NetworkTrafficRules, элемент
Элемент `NetworkTrafficRules` определяет роли, которые могут обмениваться данными с определенной конечной точкой в другой роли. Служба может содержать одно определение `NetworkTrafficRules`.

##  <a name="onlyallowtrafficto-element"></a><a name="OnlyAllowTrafficTo"></a> OnlyAllowTrafficTo, элемент
Элемент `OnlyAllowTrafficTo` описывает коллекцию целевых конечных точек и ролей, которые могут обмениваться данными с ними. Можно указать несколько узлов `OnlyAllowTrafficTo`.

##  <a name="destinations-element"></a><a name="Destinations"></a> Элемент Destinations
Элемент `Destinations` описывает коллекцию элементов RoleEndpoint, с которыми можно обмениваться данными.

##  <a name="roleendpoint-element"></a><a name="RoleEndpoint"></a> RoleEndpoint, элемент
Элемент `RoleEndpoint` описывает конечную точку в роли, с которой можно обмениваться данными. Можно указать несколько элементов `RoleEndpoint`, если в роли существует несколько конечных точек.

| attribute      | Тип     | Описание |
| -------------- | -------- | ----------- |
| `endpointName` | `string` | Обязательный. Имя конечной точки, к которой разрешается трафик.|
| `roleName`     | `string` | Обязательный. Имя веб-роли, с которой разрешается обмен данными.|

## <a name="allowalltraffic-element"></a><a name="AllowAllTraffic"></a> AllowAllTraffic, элемент
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

## <a name="see-also"></a>См. также раздел
[Схема определения облачной службы (Расширенная поддержка)](schema-csdef-file.md).




