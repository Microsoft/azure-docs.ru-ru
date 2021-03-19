---
title: Элемент пользовательского интерфейса FileUpload
description: Сведения об элементе пользовательского интерфейса Microsoft.Common.FileUpload для портала Azure. Позволяет пользователям отправлять файлы при развертывании управляемого приложения.
author: tfitzmac
ms.topic: conceptual
ms.date: 09/05/2018
ms.author: tomfitz
ms.openlocfilehash: c4eb3972be85ac7e291380f64127c20424b0d0f0
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87004292"
---
# <a name="microsoftcommonfileupload-ui-element"></a>Элемент пользовательского интерфейса Microsoft.Common.FileUpload

Элемент управления, который позволяет пользователю указать один или несколько файлов для отправки.

## <a name="ui-sample"></a>Пример элемента пользовательского интерфейса

![Элемент пользовательского интерфейса Microsoft.Common.FileUpload](./media/managed-application-elements/microsoft-common-fileupload.png)

## <a name="schema"></a>схема

```json
{
  "name": "element1",
  "type": "Microsoft.Common.FileUpload",
  "label": "Some file upload",
  "toolTip": "",
  "constraints": {
    "required": true,
    "accept": ".doc,.docx,.xml,application/msword"
  },
  "options": {
    "multiple": false,
    "uploadMode": "file",
    "openMode": "text",
    "encoding": "UTF-8"
  },
  "visible": true
}
```

## <a name="sample-output"></a>Пример выходных данных

Если options.multiple имеет значение false и options.uploadMode — file, содержимое файла в выходных данных будет представлено в виде строки JSON:

```json
"Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua."
```

Если options.multiple имеет значение true и options.uploadMode — file, содержимое файла в выходных данных будет представлено в виде массива JSON:

```json
[
  "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.",
  "Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat.",
  "Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur.",
  "Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum."
]
```

Если options.multiple имеет значение false и uploadMode — url, выходные данные будут содержать ULR-адрес в виде строки JSON:

```json
"https://myaccount.blob.core.windows.net/pictures/profile.jpg?sv=2013-08-15&st=2013-08-16&se=2013-08-17&sr=c&sp=r&rscd=file;%20attachment&rsct=binary &sig=YWJjZGVmZw%3d%3d&sig=a39%2BYozJhGp6miujGymjRpN8tsrQfLo9Z3i8IRyIpnQ%3d"
```

Если options.multiple имеет значение true и uploadMode — url, выходные данные будут содержать список ULR-адресов в виде массива JSON:
```json
[
  "https://myaccount.blob.core.windows.net/pictures/profile1.jpg?sv=2013-08-15&st=2013-08-16&se=2013-08-17&sr=c&sp=r&rscd=file;%20attachment&rsct=binary &sig=YWJjZGVmZw%3d%3d&sig=a39%2BYozJhGp6miujGymjRpN8tsrQfLo9Z3i8IRyIpnQ%3d",
  "https://myaccount.blob.core.windows.net/pictures/profile2.jpg?sv=2013-08-15&st=2013-08-16&se=2013-08-17&sr=c&sp=r&rscd=file;%20attachment&rsct=binary &sig=YWJjZGVmZw%3d%3d&sig=a39%2BYozJhGp6miujGymjRpN8tsrQfLo9Z3i8IRyIpnQ%3d",
  "https://myaccount.blob.core.windows.net/pictures/profile3.jpg?sv=2013-08-15&st=2013-08-16&se=2013-08-17&sr=c&sp=r&rscd=file;%20attachment&rsct=binary &sig=YWJjZGVmZw%3d%3d&sig=a39%2BYozJhGp6miujGymjRpN8tsrQfLo9Z3i8IRyIpnQ%3d"
]
```

При тестировании определения CreateUiDefinition некоторые браузеры (например, Google Chrome) обрезают ULR-адреса, созданные элементом Microsoft.Common.FileUpload, в консоли браузера. Чтобы скопировать полные URL-адреса, вам может потребоваться щелкнуть правой кнопкой мыши отдельные ссылки.

## <a name="remarks"></a>Комментарии

- `constraints.accept` — указывает типы файлов, которые отображаются в файловом диалоговом окне браузера. Сведения о допустимых значениях см. в [спецификации HTML5](https://html.spec.whatwg.org/multipage/input.html#attr-input-accept). Значение по умолчанию — **null**.
- Если `options.multiple` имеет значение **true**, пользователь может выбрать несколько файлов в файловом диалоговом окне браузера. Значение по умолчанию — **false**.
- Этот элемент поддерживает отправку файлов в двух режимах, которые задаются в параметре `options.uploadMode`. Если указан **file**, выходные данные предоставляют содержимое файла в виде большого двоичного объекта. Если указан **url**, файл передается во временное расположение и выходные данные содержат URL-адрес большого двоичного объекта. Временные большие двоичные объекты очищаются через 24 часа. Значение по умолчанию — **File**.
- Отправленный файл защищен. Выходной URL-адрес включает в себя [маркер SAS](../../storage/common/storage-sas-overview.md?toc=/azure/storage/blobs/toc.json) для доступа к файлу во время развертывания.
- Значение `options.openMode` определяет режим, в котором открывается файл. Если предполагается, что файл содержит обычный текст, укажите **text**. В противном случае укажите **binary**. Значение по умолчанию — **Text**.
- Если `options.uploadMode` имеет значение **file**, а `options.openMode` имеет значение **binary**, то выходные данные закодированы в формате Base64.
- `options.encoding` — указывает кодирование, которое используется при чтении файла. Значение по умолчанию — **UTF-8**, которое используется только если для параметра `options.openMode` задано значение **text**.

## <a name="next-steps"></a>Дальнейшие действия

* Общие сведения о создании определений пользовательского интерфейса см. в статье [Начало работы с CreateUiDefinition](create-uidefinition-overview.md).
* Дополнительные сведения об общих свойствах элементов пользовательского интерфейса см. в статье [Элементы CreateUiDefinition](create-uidefinition-elements.md).
