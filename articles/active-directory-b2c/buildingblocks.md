---
title: BuildingBlocks
titleSuffix: Azure AD B2C
description: Сведения об указании элемента BuildingBlocks настраиваемой политики в Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 12/10/2019
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: a42c1d06051c283f0e911c4cd166884ddd060f45
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100633280"
---
# <a name="buildingblocks"></a>BuildingBlocks

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Элемент **BuildingBlocks** добавляется внутри элемента [TrustFrameworkPolicy](trustframeworkpolicy.md).

```xml
<TrustFrameworkPolicy
  xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance"
  xmlns:xsd="https://www.w3.org/2001/XMLSchema"
  xmlns="http://schemas.microsoft.com/online/cpim/schemas/2013/06"
  PolicySchemaVersion="0.3.0.0"
  TenantId="mytenant.onmicrosoft.com"
  PolicyId="B2C_1A_TrustFrameworkBase"
  PublicPolicyUri="http://mytenant.onmicrosoft.com/B2C_1A_TrustFrameworkBase">

  <BuildingBlocks>
    <ClaimsSchema>
      ...
    </ClaimsSchema>
    <Predicates>
    ...
    </Predicates>
    <PredicateValidations>
    ...
    </PredicateValidations>
    <ClaimsTransformations>
      ...
    </ClaimsTransformations>
    <ContentDefinitions>
      ...
    </ContentDefinitions>
    <Localization>
      ...
    </Localization>
    <DisplayControls>
      ...
    </DisplayControls>
 </BuildingBlocks>
```

Элемент **BuildingBlocks** содержит следующие элементы, которые необходимо указывать в заданном порядке:

- Элемент [ClaimsSchema](claimsschema.md) определяет типы утверждений, на которые можно ссылаться в рамках политики. Схема утверждений — это область, в которой объявляются типы утверждений. Тип утверждения аналогичен переменной во многих языках программирования. Вы можете использовать тип утверждения для сбора данных от пользователя приложения, получения утверждений от поставщиков удостоверений социальных сетей, отправки и получения данных из настраиваемого интерфейса REST API или хранения внутренних данных, используемых для настраиваемой политики.

- [Элементы Predicates и PredicateValidationReference](predicates.md) позволяют выполнять процесс проверки, чтобы обеспечить ввод в утверждение только правильно сформированных данных.

- Элемент [ClaimsTransformations](claimstransformations.md) содержит список преобразований утверждений, которые могут применяться в политике.  Преобразование утверждений преобразует одно утверждение в другое. В преобразовании утверждений необходимо задать метод преобразования, например:
  - Изменение регистра строкового утверждения. Например, изменение регистра строки с нижнего на верхний.
  - Если в результате сравнения двух утверждений возвращается значение true, утверждения совпадают. В противном случае возвращается значение false.
  - Создание строкового утверждения на основе предоставленного параметра в политике.
  - Создание случайной строки с помощью генератора случайных чисел.
  - Форматирование утверждения в соответствии с указанным форматом строки. Это преобразование использует метод C# `String.Format`.

- Инпутвалидатион — этот элемент позволяет выполнять логические агрегаты, аналогичные *и* и, *или*.

- Элемент [ContentDefinitions](contentdefinitions.md) содержит URL-адреса шаблонов HTML5 для использования в пути взаимодействия пользователя. В настраиваемой политике определение содержимого определяет универсальный код ресурса (URI) страницы HTML5, используемой для данного шага пути взаимодействия пользователя. например для входа или регистрации, сброса пароля или страниц ошибок. Внешний вид можно изменить с помощью переопределения LoadUri для файла HTML5. Вы также можете создать новые определения содержимого в соответствии с потребностями. Этот элемент может содержать ссылку на локализованные ресурсы с использованием идентификатора локализации.

- Элемент [Localization](localization.md) предусматривает поддержку нескольких языков. Поддержка локализации в политиках позволяет настроить список поддерживаемых языков в политике и выбрать язык по умолчанию. Также поддерживаются языковые строки и коллекции.

- [Дисплайконтролс](display-controls.md) — определяет элементы управления, отображаемые на странице. Элементы управления "Отображение" имеют специальные функции и взаимодействуют с техническими профилями проверки серверной части. 
