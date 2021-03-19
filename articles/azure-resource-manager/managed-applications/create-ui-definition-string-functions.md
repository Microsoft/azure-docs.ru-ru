---
title: Создание строковых функций определения пользовательского интерфейса
description: Описывает строковые функции, используемые при создании определений пользовательского интерфейса для управляемых приложений Azure.
author: tfitzmac
ms.topic: conceptual
ms.date: 07/13/2020
ms.author: tomfitz
ms.openlocfilehash: c662948542c36cd93f889ca045ee245c15c7bb11
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87098162"
---
# <a name="createuidefinition-string-functions"></a>CreateUiDefinition строковые функции

Эти функции используются со строками JSON.

## <a name="concat"></a>concat

Объединяет одну или несколько строк.

Например, если выходным значением `element1` является `"Contoso"`, то этот пример возвращает строку `"Demo Contoso app"`:

```json
"[concat('Demo ', steps('step1').element1, ' app')]"
```

## <a name="endswith"></a>endsWith

Определяет, заканчивается ли строка определенным значением.

Следующий пример возвращает значение true.

```json
"[endsWith('tuvwxyz', 'xyz')]"
```

## <a name="guid"></a>guid

Создает глобально уникальную строку (идентификатор GUID).

В следующем примере возвращается значение, например `"c7bc8bdc-7252-4a82-ba53-7c468679a511"` :

```json
"[guid()]"
```

## <a name="indexof"></a>indexOf

Возвращает первую точку значения в строке или-1, если не найден.

Следующий пример возвращает значение 2.

```json
"[indexOf('abcdef', 'cd')]"
```

## <a name="lastindexof"></a>lastIndexOf

Возвращает последнюю точку значения в строке или значение-1, если не найдено.

Следующий пример возвращает значение 3.

```json
"[lastIndexOf('test', 't')]"
```

## <a name="replace"></a>replace

Возвращает строку, в которой все вхождения указанной строки в текущей строке заменены другой строкой.

В следующем примере возвращается `"Contoso.com web app"`.

```json
"[replace('Contoso.net web app', '.net', '.com')]"
```

## <a name="startswith"></a>startsWith

Определяет, начинается ли строка с определенного значения.

Следующий пример возвращает значение true.

```json
"[startsWith('abcdefg', 'ab')]"
```

## <a name="substring"></a>substring

Возвращает подстроку указанной строки. Подстрока начинается с указанного индекса и имеет указанную длину.

В следующем примере возвращается `"web"`.

```json
"[substring('Contoso.com web app', 12, 3)]"
```

## <a name="tolower"></a>toLower

Возвращает строку, преобразованную в нижний регистр.

В следующем примере возвращается `"contoso"`.

```json
"[toLower('CONTOSO')]"
```

## <a name="toupper"></a>toUpper

Возвращает строку, преобразованную в верхний регистр.

В следующем примере возвращается `"CONTOSO"`.

```json
"[toUpper('contoso')]"
```

## <a name="next-steps"></a>Дальнейшие действия

* Общие сведения об Azure Resource Manager см. в [этой статье](../management/overview.md).

