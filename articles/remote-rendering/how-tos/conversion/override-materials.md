---
title: Переопределение материалов во время преобразования модели
description: Описывает рабочий процесс переопределения материала во время преобразования
author: florianborn71
ms.author: flborn
ms.date: 02/13/2020
ms.topic: how-to
ms.openlocfilehash: 11bd79a1bc88d2605a20744f5a6b6536d754c100
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91576648"
---
# <a name="override-materials-during-model-conversion"></a>Переопределение материалов во время преобразования модели

Параметры материала в исходной модели используются для определения [материалов PBR](../../overview/features/pbr-materials.md) , используемых модулем подготовки отчетов.
Иногда [Преобразование по умолчанию](../../reference/material-mapping.md) не дает желаемых результатов и вам необходимо внести изменения.
При преобразовании модели для использования в удаленной отрисовке Azure можно предоставить файл переопределения материалов, чтобы настроить способ преобразования материалов на основе каждого материала.
Если файл с именем `<modelName>.MaterialOverrides.json` найден во входном контейнере рядом с входной моделью `<modelName>.<ext>` , то он будет использоваться как файл переопределения материалов.

## <a name="the-override-file-used-during-conversion"></a>Файл переопределения, используемый во время преобразования

В качестве простого примера предположим, что модель Box содержит один материал, именуемый «Default».
Кроме того, предположим, что его албедо цвет должен быть скорректирован для использования в ARR.
В этом случае `box.MaterialOverrides.json` файл можно создать следующим образом:

```json
[
    {
        "name": "Default",
        "albedoColor": {
            "r": 0.33,
            "g": 0.33,
            "b": 0.33,
            "a": 1.0
        }
    }
]
```

`box.MaterialOverrides.json`Файл помещается в контейнер ввода рядом с `box.fbx` , который указывает службе преобразования применить новые параметры.

### <a name="color-materials"></a>Цветовые материалы

Модель [цветового материала](../../overview/features/color-materials.md) описывает постоянно затененную поверхность, которая не зависит от освещения.
Цветовые материалы полезны при работе с ресурсами, созданными алгоритмами Фотограмметри, например.
В файлах переопределения материалов можно объявить материал как цветный материал, задав `unlit` для значение `true` .

```json
[
    {
        "name": "Photogrametry_mat1",
        "unlit" : true
    },
    {
        "name": "Photogrametry_mat2",
        "unlit" : true
    }
]
```

### <a name="ignore-specific-texture-maps"></a>Игнорировать определенные карты текстур

Иногда может потребоваться, чтобы процесс преобразования игнорировал определенные карты текстуры. Это может быть вызвано тем, что модель создала средство, которое создает специальные карты, неправильно распознаваемые модулем подготовки отчетов. Например, "ОпаЦитимап", который используется для определения чего-либо, кроме непрозрачности, или модели, где "Нормалмап" хранится как "Бумпмап". (В последнем случае необходимо игнорировать "Нормалмап", что приведет к тому, что преобразователь будет использовать "Бумпмап" как "Нормалмап".)

Принцип прост. Просто добавьте свойство с именем `ignoreTextureMaps` и добавьте любую карту текстуры, которую нужно игнорировать:

```json
[
    {
        "name": "Default",
        "ignoreTextureMaps": ["OpacityMap", "NormalMap"]
    }
]
```

Полный список текстурных карт, которые можно игнорировать, см. в схеме JSON ниже.

### <a name="applying-the-same-overrides-to-multiple-materials"></a>Применение одних и тех же переопределений к нескольким материалам

По умолчанию запись в файле reoverrides применяется, если его имя совпадает с именем материала.
Так как очень распространено, что одно и то же переопределение должно применяться к нескольким материалам, при необходимости можно указать регулярное выражение в качестве имени записи.
Поле `nameMatching` имеет значение по умолчанию `exact` , но его можно задать, чтобы указать `regex` , что запись должна применяться к каждому сопоставленному материалу.
Используемый синтаксис совпадает с используемым для JavaScript. В следующем примере показано переопределение, которое применяется к материалам с такими именами, как "Material2", "Material01" и "Material999".

```json
[
    {
        "name": "Material[0-9]+",
        "nameMatching": "regex",
        "albedoColor": {
            "r": 0.0,
            "g": 0.0,
            "b": 1.0,
            "a": 1.0
        }
    }
]
```

Только одна запись в файле переопределения материала применяется к одному материалу.
Если точное совпадение (т. е. отсутствует `nameMatching` или равно `exact` ) для имени материала, то выбирается эта запись.
В противном случае выбирается первая запись регулярного выражения в файле, соответствующем имени материала.

### <a name="getting-information-about-which-entries-applied"></a>Получение сведений о том, какие записи применены

[Информационный файл](get-information.md#information-about-a-converted-model-the-info-file) , записанный в контейнер выходных данных, содержит сведения о количестве переопределений переопределения и количестве переопределенных материалов.

## <a name="json-schema"></a>Схема JSON

Полная схема JSON для файлов материалов указана здесь. За исключением `unlit` и `ignoreTextureMaps` , доступные свойства являются подмножеством свойств, описанных в разделах модели [цветового материала](../../overview/features/color-materials.md) и [PBR Material](../../overview/features/pbr-materials.md) .

```json
{
    "definitions" :
    {
        "color":
        {
            "type" : "object",
            "description" : "Color as 4 components vector",
            "properties":
            {
                "r": {"type":"number"},
                "g": {"type":"number"},
                "b": {"type":"number"},
                "a": {"type":"number"}
            },
            "required": ["r", "g", "b"]
        },
        "alpha":
        {
            "type" : "object",
            "description" : "Alpha channel for color",
            "properties":
            {
                "a": {"type":"number"}
            },
            "required": ["a"]
        },
        "colorOrAlpha":
        {
            "anyOf": [
                {"$ref": "#/definitions/color"},
                {"$ref": "#/definitions/alpha"}
            ]
        },
        "listOfMaps":
        {
            "type": "array",
            "items": {
                "type": "string",
                "enum": ["AlbedoMap",
                            "EmissiveMap",
                            "NormalMap",
                            "OcclusionMap",
                            "RoughnessMap",
                            "MetalnessMap",
                            "ReflectivityMap",
                            "BumpMap",
                            "OpacityMap",
                            "DiffuseMap",
                            "SpecularMap",
                            "ShininessMap",
                            "MetallicRoughnessMap",
                            "SpecularGlossinessMap"]
            }
        }
    },
    "type" : "array",
    "description" : "List of materials to override",
    "items":
    {
        "type" : "object",
        "description" : "List of parameters to override",
        "properties":
        {
            "name": { "type" : "string"},
            "nameMatching" : { "type" : "string", "enum" : ["exact", "regex"] },
            "unlit": { "type" : "boolean" },
            "albedoColor": { "$ref": "#/definitions/colorOrAlpha" },
            "roughness": { "type": "number" },
            "metalness": { "type": "number" },
            "transparent": { "type" : "boolean" },
            "alphaClipEnabled": { "type" : "boolean" },
            "alphaClipThreshold": { "type": "number" },
            "useVertexColor": { "type" : "boolean" },
            "isDoubleSided": { "type" : "boolean" },
            "ignoreTextureMaps": { "$ref" : "#/definitions/listOfMaps" }
        },
        "required": ["name"],
        "additionalProperties" : false
    }
}
```

## <a name="next-steps"></a>Дальнейшие действия

* [Цветовые материалы](../../overview/features/color-materials.md)
* [Материалы PBR](../../overview/features/pbr-materials.md)
