---
title: Ошибки и предупреждения, связанные с преобразованием рисунков Azure Maps
description: Сведения об ошибках и предупреждениях, которые могут возникнуть при использовании службы преобразования Azure Maps. Ознакомьтесь с рекомендациями по устранению ошибок и предупреждений, часть из которых сопровождается примерами.
author: anastasia-ms
ms.author: v-stharr
ms.date: 12/07/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philMea
ms.openlocfilehash: cecc19f0984ce1801d50e5cbda73e98a01e2825b
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96906222"
---
# <a name="drawing-conversion-errors-and-warnings"></a>Ошибки и предупреждения, связанные с преобразованием рисунков

[Служба преобразования Azure Maps](/rest/api/maps/conversion) позволяет преобразовывать отправленные пакеты рисунков в данные схемы. Пакеты рисунков должны соответствовать [требованиям к пакету рисунков](drawing-requirements.md). Если одно или несколько требований не выполняются, служба преобразования вернет ошибки или предупреждения. В этой статье перечислены коды ошибок и предупреждений в результате преобразования и рекомендации по их устранению. Здесь также приводятся некоторые примеры рисунков, которые могут привести к возврату этих кодов службой преобразования.

Служба преобразования будет работать правильно, если нет никаких предупреждений о преобразовании. Однако рекомендуется просмотреть все предупреждения и устранять указанные в них проблемы. Предупреждение означает, что часть преобразования была пропущена или исправлена автоматически. Если не разрешить проблему, ставшую причиной предупреждения, это может привести к ошибкам в последующих процессах.

## <a name="general-warnings"></a>Предупреждения общего характера

### <a name="geometrywarning"></a>**geometryWarning**

#### <a name="description-for-geometrywarning"></a>*Описание для geometryWarning*

Предупреждение **geometryWarning** возникает, когда рисунок содержит недопустимую сущность. Недопустимая сущность — это сущность, которая не согласуется с ограничениями для геометрии. Примеры недопустимой сущности: самопересекающийся многоугольник или незамкнутая ломаная линия в слое, поддерживающем только замкнутую геометрию.

Службе преобразования не удается создать функцию схемы из недопустимой сущности — вместо этого она игнорируется.

#### <a name="examples-for-geometrywarning"></a>*Примеры для geometryWarning*

* На двух рисунках ниже показаны примеры самопересекающихся многоугольников.

     ![Пример самопересекающегося многоугольника, например один.](./media/drawing-conversion-error-codes/geometry-warning-1.png)

     ![Пример самопересекающегося многоугольника, например два.](./media/drawing-conversion-error-codes/geometry-warning-2.png)

* Ниже приведен рисунок, на котором показана незамкнутая ломаная линия. Допустим, что слой поддерживает только замкнутую геометрию.

    ![Пример незамкнутой ломаной линии](./media/drawing-conversion-error-codes/geometry-warning-3.png)

#### <a name="how-to-fix-geometrywarning"></a>*Как устранить ошибку geometryWarning*

Проверьте **geometryWarning** для каждой сущности, чтобы убедиться, что она соответствует ограничениям на геометрию.

### <a name="unexpectedgeometryinlayer"></a>**unexpectedGeometryInLayer**

#### <a name="description-for-unexpectedgeometryinlayer"></a>*Описание для unexpectedGeometryInLayer*

Предупреждение **unexpectedGeometryInLayer** возникает, если рисунок содержит геометрию, несовместимую с ожидаемым типом геометрии для данного слоя. Если служба преобразования возвращает предупреждение **unexpectedGeometryInLayer**, она будет игнорировать эту геометрию.

#### <a name="example-for-unexpectedgeometryinlayer"></a>*Пример для unexpectedGeometryInLayer*

На рисунке ниже показана незамкнутая ломаная линия. Допустим, что слой поддерживает только замкнутую геометрию.

![Пример незамкнутой ломаной линии](./media/drawing-conversion-error-codes/geometry-warning-3.png)

#### <a name="how-to-fix-unexpectedgeometryinlayer"></a>*Как устранить ошибку unexpectedGeometryInLayer*

Проверьте все предупреждения **unexpectedGeometryInLayer** и переместите несовместимую геометрию в совместимый слой. Если она несовместима с другими слоями, ее следует удалить.

### <a name="unsupportedfeaturerepresentation"></a>**unsupportedFeatureRepresentation**

#### <a name="description-for-unsupportedfeaturerepresentation"></a>*Описание для unsupportedFeatureRepresentation*

Предупреждение **unsupportedFeatureRepresentation** возникает, если рисунок содержит неподдерживаемый тип сущности.

#### <a name="example-for-unsupportedfeaturerepresentation"></a>*Пример для unsupportedFeatureRepresentation*

На рисунке ниже показан неподдерживаемый тип сущности в виде многострочного текстового объекта на слое меток.
  
![Пример многострочного текстового объекта на слое меток](./media/drawing-conversion-error-codes/multi-line.png)

#### <a name="how-to-fix-unsupportedfeaturerepresentation"></a>*Как устранить ошибку unsupportedFeatureRepresentation*

Убедитесь, что DWG-файлы содержат только поддерживаемые типы сущностей. Поддерживаемые типы перечислены в [разделе требований к графическим файлам в статье "Требования к пакету рисунков"](drawing-requirements.md#drawing-package-requirements).

### <a name="automaticrepairperformed"></a>**automaticRepairPerformed**

#### <a name="description-for-automaticrepairperformed"></a>*Описание для automaticRepairPerformed*

Предупреждение **automaticRepairPerformed** возникает, когда служба преобразования автоматически восстанавливает недопустимую геометрию.

#### <a name="examples-for-automaticrepairperformed"></a>*Примеры для automaticRepairPerformed*

* На рисунке ниже показано, как служба преобразования восстановила самопересекающийся многоугольник в допустимую геометрию.

    ![Пример восстановленного самопересекающегося многоугольника](./media/drawing-conversion-error-codes/automatic-repair-1.png)

* На рисунке ниже показано, как служба преобразования привязывает первую и последнюю вершину незамкнутой ломаной линии для создания замкнутой ломаной линии, где первая и последняя вершины находятся на расстоянии менее 1 мм друг от друга.  

    ![Пример привязанной ломаной линии](./media/drawing-conversion-error-codes/automatic-repair-2.png)

* На рисунке ниже показано, как в слое, поддерживающем только замкнутые ломаные линии, служба преобразования исправила несколько незамкнутых ломаных линий. Чтобы избежать отбрасывания незамкнутых ломаных линий, служба объединяет их в одну замкнутую линию.

    ![Пример объединения незамкнутых ломаных линий в одну замкнутую ломаную линию](./media/drawing-conversion-error-codes/automatic-repair-3.png)

#### <a name="how-to-fix-automaticrepairperformed"></a>*Как устранить ошибку automaticRepairPerformed*

Чтобы устранить предупреждение **automaticRepairPerformed**, выполните следующие действия.

1. Изучите геометрию каждого предупреждения и текст самого предупреждения.
2. Определите, правильно ли используется автоматическое восстановление.
3. Если восстановление прошло правильно, продолжайте. В противном случае перейдите в файл проекта и устраните предупреждение вручную.

>[!TIP]
>Чтобы на будущее отключить предупреждения, внесите изменения в исходный рисунок, чтобы исходный рисунок соответствовал исправленному.

## <a name="manifest-warnings"></a>Предупреждения о манифесте

### <a name="redundantattribution"></a>**redundantAttribution**

#### <a name="description-for-redundantattribution"></a>*Описание для redundantAttribution*

Предупреждение **redundantAttribution** возникает, когда манифест содержит избыточные или конфликтующие свойства объекта.

#### <a name="examples-for-redundantattribution"></a>*Примеры для redundantAttribution*

* Приведенный ниже фрагмент кода JSON содержит два или более объектов `unitProperties` с одним и тем же параметром `name`.

    ```json
    "unitProperties": [
        {
            "unitName": "L1-100",
            "categoryName": "room.office"
        },
        {
            "unitName": "L1-101",
            "categoryName": "room.office"
        },
        {
            "unitName": "L1-101",
            "categoryName": "room.office"
        }
    ]
    ```

* В следующем фрагменте кода JSON два или более объектов `zoneProperties` имеют один и тот же параметр `name`.

    ```json
     "zoneProperties": [
        {
            "zoneName": "Assembly Area 1",
            "categoryName": "zone.assembly"
        },
        {
            "zoneName": "Assembly Area 2",
            "categoryName": "zone.assembly"
        },
        {
            "zoneName": "Assembly Area 2",
            "categoryName": "zone.assembly"
        }
    ```

#### <a name="how-to-fix-redundantattribution"></a>*Как устранить ошибку redundantAttribution*

Чтобы устранить предупреждение *redundantAttribution*, удалите избыточные или конфликтующие свойства объекта.

### <a name="manifestwarning"></a>**manifestWarning**

#### <a name="description-for-manifestwarning"></a>*Описание для manifestWarning*

Предупреждение **manifestWarning** возникает, когда манифест содержит неиспользуемые во время преобразования объекты unitProperties или zoneProperties.

#### <a name="examples-for-manifestwarning"></a>*Примеры для manifestWarning*

* Манифест содержит объект `unitProperties` с параметром `unitName`, который не имеет совпадающей метки в слое `unitLabel`.

* Манифест содержит объект `zoneProperties` с параметром `zoneName`, который не имеет совпадающей метки в слое `zoneLabel`.

#### <a name="how-to-fix-manifestwarning"></a>*Как устранить ошибку manifestWarning*

Чтобы исправить предупреждение **manifestWarning**, удалите неиспользуемый объект `unitProperties` или `zoneProperties` из манифеста или добавьте к рисунку метку секции или зоны, чтобы объект свойств использовался во время преобразования.

## <a name="wall-warnings"></a>Предупреждения о стенах

### <a name="walloutsidelevel"></a>**wallOutsideLevel**

#### <a name="description-for-walloutsidelevel"></a>*Описание для wallOutsideLevel*

Предупреждение **wallOutsideLevel** возникает, когда рисунок содержит геометрию стены за пределами контура уровня.

#### <a name="example-for-walloutsidelevel"></a>*Пример для wallOutsideLevel*

* На рисунке ниже показана внутренняя стена красного цвета вне желтой границы уровня.

    ![Пример внутренней стены за пределами границы уровня](./media/drawing-conversion-error-codes/wall-outside-level.png)

* На рисунке ниже показана наружная стена красного цвета за желтой границей уровня.

    ![Пример внешней стены за пределами границы уровня](./media/drawing-conversion-error-codes/wall-outside-level-exterior.png)

#### <a name="how-to-fix-walloutsidelevel"></a>*Как устранить ошибку wallOutsideLevel*

Чтобы устранить предупреждение **wallOutsideLevel**, увеличьте геометрию уровня, чтобы она охватывала все стены. Или измените границы стены, чтобы они поместились внутри границы уровня.

## <a name="unit-warnings"></a>Предупреждения о секциях

### <a name="unitoutsidelevel"></a>**unitOutsideLevel**

#### <a name="description-for-unitoutsidelevel"></a>*Описание для unitOutsideLevel*

Предупреждение **unitOutsideLevel** возникает, когда рисунок содержит геометрию секции за пределами контура уровня.

#### <a name="example-for-unitoutsidelevel"></a>*Пример для unitOutsideLevel*

 На следующем рисунке геометрия секции (показана красным), выходит за границы уровня (показан желтым).

 ![Пример секции за пределами границы уровня](./media/drawing-conversion-error-codes/unit-outside-level.png)

#### <a name="how-to-fix-unitoutsidelevel"></a>*Как устранить ошибку unitOutsideLevel*

Чтобы исправить предупреждение **unitOutsideLevel**, растяните границу уровня, чтобы она охватывала все секции. Или измените геометрию секции, чтобы она поместилась внутри границы уровня.

### <a name="partiallyoverlappingunit"></a>**partiallyOverlappingUnit**

#### <a name="description-for-partiallyoverlappingunit"></a>*Описание для partiallyOverlappingUnit*

Предупреждение появляется **partiallyOverlappingUnit**, если рисунок содержит геометрию секции, которая частично перекрывается с другой геометрией секции. Служба преобразования отклоняет перекрывающиеся секции.

#### <a name="example-scenarios-partiallyoverlappingunit"></a>*Пример сценария partiallyOverlappingUnit*

На рисунке ниже перекрывающиеся секции выделены красным цветом. `UNIT110` и `HALLWAY` отбрасываются.

![Пример перекрывающихся секций](./media/drawing-conversion-error-codes/partially-overlapping-unit.png)

#### <a name="how-to-fix-partiallyoverlappingunit"></a>*Как устранить ошибку partiallyOverlappingUnit*

Чтобы исправить предупреждение **partiallyOverlappingUnit**, перерисуйте все частично перекрывающиеся секции, чтобы они не перекрывались между собой.

## <a name="door-warnings"></a>Предупреждения о дверях

### <a name="dooroutsidelevel"></a>**doorOutsideLevel**

#### <a name="description-for-dooroutsidelevel"></a>*Описание для doorOutsideLevel*

Предупреждение **doorOutsideLevel** возникает, когда рисунок содержит геометрию двери за пределами геометрии уровня.

#### <a name="example-for-dooroutsidelevel"></a>*Пример для doorOutsideLevel*

На рисунке ниже геометрия двери, выделенная красным цветом, пересекается с желтой границей уровня.

![Пример перекрытия двери с границей уровня](./media/drawing-conversion-error-codes/door-outside-level.png)

#### <a name="how-to-fix-dooroutsidelevel"></a>*Как устранить ошибку doorOutsideLevel*

Чтобы устранить предупреждение **doorOutsideLevel**, перерисуйте геометрию двери, чтобы она находилась внутри границ уровня.

## <a name="zone-warnings"></a>Предупреждения о зонах

### <a name="zonewarning"></a>**zoneWarning**

#### <a name="description-for-zonewarning"></a>*Описание для zoneWarning*

Предупреждение **zoneWarning** возникает, когда зона не содержит метку. Служба преобразования отклоняет зону, не имеющую меток.

#### <a name="example-for-zonewarning"></a>*Пример для zoneWarning*

На рисунке ниже показана зона, которая не содержит метку.

![Пример зоны, не содержащей метку](./media/drawing-conversion-error-codes/zone-warning.png)

#### <a name="how-to-fix-zonewarning"></a>*Как устранить ошибку zoneWarning*

Чтобы устранить предупреждение **zoneWarning**, убедитесь, что каждая зона имеет одну метку.

## <a name="label-warnings"></a>Предупреждения о метках

### <a name="labelwarning"></a>*labelWarning*

#### <a name="description-for-labelwarning"></a>*Описание для labelWarning*

Предупреждение **labelWarning** возникает, если рисунок содержит функцию неоднозначных или противоречивых меток.

Предупреждение **labelWarning** возникает по одной или нескольким из следующих причин:

* Метка устройства не находится ни в одной из секций.
* Метка зоны не находится ни в одной из зон.
* Метка зоны находится внутри двух или более зон.

#### <a name="example-for-labelwarning"></a>*Пример для labelWarning*

На рисунке ниже показана метка, которая находится внутри двух зон.

![Пример метки внутри двух зон ](./media/drawing-conversion-error-codes/label-warning.png)

#### <a name="how-to-fix-labelwarning"></a>*Как устранить ошибку labelWarning*

Чтобы устранить предупреждение **labelWarning**, убедитесь в том, что:

* Все метки секций находятся внутри секций.
* Все метки зон находятся внутри зон.
* Все метки зон находятся в одной и только одной зоне.

## <a name="drawing-package-errors"></a>Ошибки пакета рисунков

### <a name="invalidarchiveformat"></a>**invalidArchiveFormat**

#### <a name="description-for-invalidarchiveformat"></a>*Описание для invalidArchiveFormat*

Ошибка **invalidArchiveFormat** возникает, когда пакет рисунков находится в архиве недопустимого формата, например GZIP или 7-Zip. Поддерживается только формат архивов ZIP.

Ошибка **invalidArchiveFormat** также возникнет, если ZIP-архив пуст.

#### <a name="how-to-fix-invalidarchiveformat"></a>*Как устранить ошибку invalidArchiveFormat*

Чтобы исправить ошибку **invalidArchiveFormat**, убедитесь в том, что:

* Имя файла архива заканчивается на _.zip_.
* ZIP-архив содержит данные.
* ZIP-архив не удается открыть.

### <a name="invaliduserdata"></a>**invalidUserData**

#### <a name="description-for-invaliduserdata"></a>*Описание для invalidUserData*

Ошибка **invalidUserData** возникает, когда службе преобразования не удается прочитать объект пользовательских данных из хранилища.

#### <a name="example-scenario-for-invaliduserdata"></a>*Пример сценария для invalidUserData*

Предпринята попытка передать пакет рисунков с неверным параметром `udid`.

#### <a name="how-to-fix-invaliduserdata"></a>*Как устранить ошибку invalidUserData*

Чтобы исправить ошибку **invalidUserData**, убедитесь в том, что:

* Вы указали правильный идентификатор `udid` для отправленного пакета.
* Создатель Azure Maps (Предварительная версия) включен для учетной записи Azure Maps, которую вы использовали для отправки пакета Drawing.
* Запрос API к службе преобразования содержит ключ подписки для учетной записи Azure Maps, которую вы использовали для отправки пакета рисунков.

### <a name="dwgerror"></a>**dwgError**

#### <a name="description-for-dwgerror"></a>*Описание для dwgError*

Ошибка **dwgError** возникает, когда пакет рисунков содержит ошибку одного или нескольких файлов DWG в отправленном ZIP-архиве.

Ошибка **dwgError** возникает, когда пакет рисунков содержит файл DWG, который не удается открыть, поскольку он является недопустимым или поврежденным.

* DWG-файл не является допустимым рисунком в формате файла AutoCAD DWG.
* DWG-файл поврежден.
* Файл DWG указан в файле _manifest.json_, но отсутствует в ZIP-архиве.

#### <a name="how-to-fix-dwgerror"></a>*Как устранить ошибку dwgError*

Чтобы исправить **dwgError**, проверьте файл манифеста _manifest.json_ и убедитесь в том, что:

* Все DWG-файлы в ZIP-архиве являются допустимыми рисунками в формате AutoCAD DWG; откройте каждый из них в программе AutoCAD. Удалите или исправьте все недопустимые рисунки.
* DWG-файлы, указанные в файле _manifest.json_, совпадают с DWG-файлами в ZIP-архиве.

## <a name="manifest-errors"></a>Ошибки манифеста

### <a name="invalidjsonformat"></a>**invalidJsonFormat**

#### <a name="description-for-invalidjsonformat"></a>Описание для invalidJsonFormat

Ошибка **invalidJsonFormat** возникает, когда не удается прочитать файл _manifest.json_.

Не удалось прочитать файл "manifest.json" из-за ошибок форматирования или синтаксиса JSON. Дополнительные сведения о формате и синтаксисе JSON см. в разделе [Формат обмена данными JSON](https://tools.ietf.org/html/rfc7159).

#### <a name="how-to-fix-invalidjsonformat"></a>*Как устранить ошибку invalidJsonFormat*

Чтобы исправить ошибку **invalidJsonFormat**, используйте средство поиска ошибок JSON, чтобы обнаружить и устранить ошибки JSON.

### <a name="missingrequiredfield"></a>**missingRequiredField**

#### <a name="description-for-missingrequiredfield"></a>*Описание для missingRequiredField*

Ошибка **missingRequiredField** возникает, когда в файле _manifest.json_ отсутствуют необходимые данные.

#### <a name="how-to-fix-missingrequiredfield"></a>*Как устранить ошибку missingRequiredField*

Чтобы исправить ошибку **missingRequiredField**, убедитесь, что манифест содержит все необходимые свойства. Полный список обязательных объектов манифеста см. в разделе [манифеста статьи о требованиях к пакету рисунков](drawing-requirements.md#manifest-file-requirements).  

### <a name="missingmanifest"></a>**missingManifest**

#### <a name="description-for-missingmanifest"></a>*Описание для missingManifest*

Ошибка **missingManifest** возникает, когда файл _manifest.json_ отсутствует в ZIP-архиве.

Ошибка **missingManifest** возникает по одной или нескольким из следующих причин:

* Файл _manifest.json_ написан неправильно.
* Файл _manifest.json_ отсутствует.
* Файл _manifest.json_ находится не в корневом каталоге ZIP-архива.

#### <a name="how-to-fix-missingmanifest"></a>*Как устранить ошибку missingManifest*

Чтобы устранить ошибку **missingManifest**, убедитесь, что архив содержит файл с именем _manifest.json_ в корневом каталоге ZIP-архива.

### <a name="conflict"></a>**conflict**

#### <a name="description-for-conflict"></a>*Описание для conflict*

Ошибка **conflict** возникает, если файл _manifest.json_ содержит конфликтующие сведения.

#### <a name="example-scenario-for-conflict"></a>*Пример сценария для conflict*

Служба преобразования возвращает ошибку **conflict**, если несколько уровней определены с одним и тем же порядковым номером уровня. В следующем фрагменте кода JSON показаны два уровня с одинаковым порядковым номером.

```JSON
"buildingLevels":
{
    "levels": [
        {
            "levelName": "Ground",
            "ordinal": 0,
            "filename": "./Level_0.dwg"
        },
        {
            "levelName": "Parking",
            "ordinal": 0,
            "filename": "./Level_P.dwg"
        }
    ]
}
```

#### <a name="how-to-fix-conflict"></a>*Как устранить ошибку conflict*

Чтобы устранить ошибку **conflict**, проверьте файл _manifest.json_ и удалите все конфликтующие сведения.

### <a name="invalidgeoreference"></a>**invalidGeoreference**

#### <a name="description-for-invalidgeoreference"></a>*Описание для invalidGeoreference*

Ошибка **invalidGeoreference** возникает, когда файл _manifest.json_ содержит недопустимую геоссылку.

Ошибка **invalidGeoreference** возникает по одной или нескольким из следующих причин:

* Пользователь указывает на значение широты или долготы, находящиеся вне допустимого диапазона.
* Пользователь указывает значение поворота геолокации, находящееся вне допустимого диапазона.

#### <a name="example-scenario-for-invalidgeoreference"></a>*Пример сценария для invalidGeoreference*

В следующем фрагменте кода JSON значение широты превышает верхний предел.

```json
"georeference"
{
    "lat": 88.0,
    "lon": -122.132600,
    "angle": 0
},
```

#### <a name="how-to-fix-invalidgeoreference"></a>*Как устранить ошибку invalidGeoreference*

Чтобы исправить ошибку **invalidGeoreference**, убедитесь, что значения в геоссылке попадают в правильный диапазон.

>[!IMPORTANT]
>В формате GeoJSON порядок координат: долгота и широта. Если вы не используете правильный порядок, возможно, вы случайно вызываете значение широты или долготы, которое выходит за пределы диапазона.

## <a name="wall-errors"></a>Ошибки стен

### <a name="wallerror"></a>**wallError**

#### <a name="description-for-wallerror"></a>*Описание для wallError*

Ошибка **wallError** возникает, когда рисунок содержит ошибку при попытке создать функцию стены.

#### <a name="example-scenario-for-wallerror"></a>*Пример сценария для wallError*

На рисунке ниже показана функция стены, которая не пересекается ни с одной из секций.

![Пример функции "Стена", которая не пересекается ни с одной секцией](./media/drawing-conversion-error-codes/wall-error.png)

#### <a name="how-to-fix-wallerror"></a>*Как устранить ошибку wallError*

Чтобы устранить ошибку **wallError**, перерисуйте стену, чтобы она перекрывалась по крайней мере с одной секцией. Или создайте новую секцию, которая будет перекрываться со стеной.

## <a name="vertical-penetration-errors"></a>Ошибки вертикального ввода

### <a name="verticalpenetrationerror"></a>**verticalPenetrationError**

#### <a name="description-for-verticalpenetrationerror"></a>*Описание для verticalPenetrationError*

Ошибка **verticalPenetrationError** возникает, когда рисунок содержит функцию неоднозначного вертикального ввода.

Ошибка **verticalPenetrationError** возникает по одной или нескольким из следующих причин:

* Рисунок содержит вертикальный ввод без перекрывающихся областей вертикального ввода на любом из уровней выше или ниже этого.
* Пакет рисунков содержит уровень с двумя или более функциями вертикального ввода, которые накладываются на одну функцию вертикального ввода на другом уровне непосредственно над или под этим уровнем.

#### <a name="example-scenario-for-verticalpenetrationerror"></a>*Пример сценария для verticalPenetrationError*

На рисунке ниже показана область вертикального ввода без перекрывающихся областей вертикального ввода на уровнях выше или ниже этого.

![Пример вертикального ввода 1](./media/drawing-conversion-error-codes/vrt-2.png)

На следующем рисунке показана область вертикального ввода, которая перекрывается с несколькими областями вертикального ввода на смежном уровне.

![Пример вертикального ввода 2](./media/drawing-conversion-error-codes/vrt-1.png)

#### <a name="how-to-fix-verticalpenetrationerror"></a>Как устранить ошибку verticalPenetrationError

Чтобы устранить ошибку **verticalPenetrationError**, прочитайте о том, как использовать функцию вертикального ввода в статье [Требования к пакету рисунков](drawing-requirements.md).

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Использование визуализатора ошибок рисования в Azure Maps](drawing-error-visualizer.md)

> [!div class="nextstepaction"]
> [Создатель (Предварительная версия) для сопоставления помещений](creator-indoor-maps.md)