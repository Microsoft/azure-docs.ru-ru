---
title: Создание элементов определения пользовательского интерфейса
description: Сведения об элементах, используемых при создании определений пользовательского интерфейса для портала Azure.
author: tfitzmac
ms.topic: conceptual
ms.date: 10/27/2020
ms.author: tomfitz
ms.openlocfilehash: 3e8bdd85c112d90fd5e10f6e65e48ca6b3f71b6e
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94410748"
---
# <a name="createuidefinition-elements"></a>Элементы CreateUiDefinition

В этой статье описывается схема и свойства всех поддерживаемых элементов CreateUiDefinition.

## <a name="schema"></a>схема

Для большинства элементов схема выглядит следующим образом:

```json
{
  "name": "element1",
  "type": "Microsoft.Common.TextBox",
  "label": "Some text box",
  "defaultValue": "my value",
  "toolTip": "Provide a descriptive name.",
  "constraints": {},
  "options": {},
  "visible": true
}
```

| Свойство | Обязательно | Описание |
| -------- | -------- | ----------- |
| name | Да | Внутренний идентификатор для ссылки на конкретный экземпляр элемента. Чаще всего имя элемента используется в `outputs`, где значения выходных данных указанных элементов сопоставляются с параметрами шаблона. Его также можно использовать для привязки выходного значения элемента к `defaultValue` другого элемента. |
| тип | Да | Элемент управления пользовательского интерфейса для обработки элемента. Список поддерживаемых типов см. в разделе [Элементы](#elements). |
| метка | Да | Отображаемый текст элемента. Некоторые типы элементов содержат несколько меток, поэтому значение может быть объектом, содержащим несколько строк. |
| defaultValue | Нет | Значение элемента по умолчанию. Некоторые типы элементов поддерживают сложные значения по умолчанию, поэтому значение может быть объектом. |
| toolTip | Нет | Текст для отображения в подсказке элемента. Аналогично `label` некоторые элементы поддерживают несколько строк с подсказками. С помощью синтаксиса Markdown можно внедрить встроенные ссылки.
| constraints | Нет | Одно или несколько свойств, используемых для настройки поведения проверки элемента. Поддерживаемые свойства для constraints зависят от типа элемента. Некоторые типы элементов не поддерживают настройку поведения проверки и поэтому не содержат свойство constraints. |
| options | Нет | Дополнительные свойства, позволяющие настроить поведение элемента. Аналогично `constraints` поддерживаемые свойства зависят от типа элемента. |
| visible | Нет | Указывает, отображается ли элемент. Если задано значение `true`, элемент и применимые дочерние элементы будут отображены. Значение по умолчанию — `true`. Используйте [логические функции](create-uidefinition-functions.md#logical-functions), чтобы динамически управлять значением свойства.

## <a name="elements"></a>Элементы

Документация для каждого элемента содержит примеры пользовательского интерфейса, схемы, примечания о поведении элемента (обычно в отношении проверки и поддерживаемой настройки) и пример выходных данных.

- [Microsoft. Common. CheckBox](microsoft-common-checkbox.md)
- [Элемент пользовательского интерфейса Microsoft.Common.DropDown](microsoft-common-dropdown.md)
- [Microsoft. Common. Едитаблегрид](microsoft-common-editablegrid.md)
- [Элемент пользовательского интерфейса Microsoft.Common.FileUpload](microsoft-common-fileupload.md)
- [Microsoft.Common.InfoBox](microsoft-common-infobox.md)
- [Элемент пользовательского интерфейса Microsoft.Common.OptionsGroup](microsoft-common-optionsgroup.md)
- [Элемент пользовательского интерфейса Microsoft.Common.PasswordBox](microsoft-common-passwordbox.md)
- [Элемент пользовательского интерфейса Microsoft.Common.Section](microsoft-common-section.md)
- [Microsoft. Common. СервицепринЦипалселектор](microsoft-common-serviceprincipalselector.md)
- [Microsoft. Common. Slider](microsoft-common-slider.md)
- [Microsoft. Common. Тагсбиресаурце](microsoft-common-tagsbyresource.md)
- [Microsoft.Common.TextBlock](microsoft-common-textblock.md)
- [Элемент пользовательского интерфейса Microsoft.Common.TextBox](microsoft-common-textbox.md)
- [Элемент пользовательского интерфейса Microsoft.Compute.CredentialsCombo](microsoft-compute-credentialscombo.md)
- [Элемент пользовательского интерфейса Microsoft.Compute.SizeSelector](microsoft-compute-sizeselector.md)
- [Элемент пользовательского интерфейса Microsoft.Compute.UserNameTextBox](microsoft-compute-usernametextbox.md)
- [Microsoft. KeyVault. Кэйваултцертификатеселектор](microsoft-keyvault-keyvaultcertificateselector.md)
- [Microsoft. ManagedIdentity. Идентитиселектор](microsoft-managedidentity-identityselector.md)
- [Элемент пользовательского интерфейса Microsoft.Network.PublicIpAddressCombo](microsoft-network-publicipaddresscombo.md)
- [Элемент пользовательского интерфейса Microsoft.Network.VirtualNetworkCombo](microsoft-network-virtualnetworkcombo.md)
- [Microsoft. Solutions. Армапиконтрол](microsoft-solutions-armapicontrol.md)
- [Microsoft. Solutions. Ресаурцеселектор](microsoft-solutions-resourceselector.md)
- [Элемент пользовательского интерфейса Microsoft.Storage.MultiStorageAccountCombo](microsoft-storage-multistorageaccountcombo.md)
- [Элемент пользовательского интерфейса Microsoft.Storage.StorageAccountSelector](microsoft-storage-storageaccountselector.md)
- [Microsoft. Storage. Сторажеблобселектор](microsoft-storage-storageblobselector.md)

## <a name="next-steps"></a>Дальнейшие действия

Общие сведения о создании определений пользовательского интерфейса см. в статье [Начало работы с CreateUiDefinition](create-uidefinition-overview.md).
