---
title: Эффект Френеля
description: Страница описания функции для френелюного материала
author: jumeder
ms.author: jumeder
ms.date: 11/09/2020
ms.topic: article
ms.custom: devx-track-csharp
ms.openlocfilehash: f63cd3b50642c3cf531387b4446992b6f15116f2
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99594118"
---
# <a name="fresnel-effect"></a>Эффект Френеля

Функция материалов Френелю Effect является нефизическим нерегламентированным результатом. Эта функция основана на физическом наблюдении за объектами, которые становятся более отражающими на этих углы. Само отражение Френелю уже физически включено в [модель-материал PBR](../../overview/features/pbr-materials.md) , используемую в удаленной отрисовке Azure. В отличие от этого, функция материала Френелю Effect — это лишь аддитивный цвет, без зависимости от [огней](../../overview/features/lights.md) или « [небесной» среды](../../overview/features/sky.md).

Френелюный результат дает затрагиваемым объектам цветные светлые рамки вокруг их краев. Сведения о настройке эффектов и примеры результатов подготовки к просмотру можно найти в следующих разделах.

## <a name="enabling-the-fresnel-effect"></a>Включение френелюного влияния

Чтобы использовать функцию Френелю Effects, ее необходимо включить в рассматриваемых материалах. Его можно включить, установив бит Фреснелеффект [пбрматериалфеатурес](/dotnet/api/microsoft.azure.remoterendering.pbrmaterialfeatures) на [материале PBR](../../overview/features/pbr-materials.md). Тот же шаблон применяется к [колорматериалфеатурес](/dotnet/api/microsoft.azure.remoterendering) и [цветному материалу](../../overview/features/color-materials.md). Пример использования см. в разделе примеры кода.

После включения френелюный результат будет немедленно виден. По умолчанию в качестве «-белого» будет использоваться белый (1, 1, 1, 1) и экспонента 1. Эти параметры можно настроить с помощью методов задания параметров ниже.

## <a name="customizing-the-effect-appearance"></a>Настройка внешнего вида эффектов

В настоящее время Френелю можно настроить для каждого материала, используя следующие свойства:

| Свойство Material | Type | Пояснение |
|-------------------|------|-------------|
| фреснелеффектколор | Color4 | Цвет, который добавляется максимум в качестве френелюа. Альфа-канал в данный момент игнорируется. |
| фреснелеффектекспонент | FLOAT | Разворота френелюа. Диапазон от 0,01 (распределение по всему объекту) до 10 (только наиболее граЦинг углы). |

На практике различные параметры цвета и экспоненты будут выглядеть следующим образом:

![Примеры эффектов Френелю](./media/fresnel-effect-examples.png)

Показатель степени Френелю эффектов постепенно увеличивается от 1 до 10 для каждой строки цвета. Это позволяет постепенно зафренелю себя на краях просматриваемых объектов. На эффект Френелю также не влияет прозрачность, как видно в следующем примере:

![Примеры прозрачности Френелю эффектов](./media/fresnel-effect-transparent-examples.png)

Как видно, объекты по диагонали полностью прозрачны, но Френелю «неправильность» остается. Этот результат имитирует физическое Френелю в этом отношении, который также имеется на снимках экрана.

## <a name="code-samples"></a>Примеры кода

В следующих примерах кода показано, как включить и настроить френелюный результат для типа и [материала PBR](../../overview/features/pbr-materials.md) [:](../../overview/features/color-materials.md)

```cs
    void SetFresnelEffect(RenderingSession session, Material material)
    {
        if (material.MaterialSubType == MaterialType.Pbr)
        {
            var pbrMaterial = material as PbrMaterial;
            pbrMaterial.PbrFlags |= PbrMaterialFeatures.FresnelEffect;
            pbrMaterial.FresnelEffectColor = new Color4(1.0f, 0.5f, 0.1f, 1.0f);
            pbrMaterial.FresnelEffectExponent = 3.141592f;
        }
        else if (material.MaterialSubType == MaterialType.Color)
        {
            var colorMaterial = material as ColorMaterial;
            colorMaterial.ColorFlags |= ColorMaterialFeatures.FresnelEffect;
            colorMaterial.FresnelEffectColor = new Color4(0.25f, 1.0f, 0.25f, 1.0f);
            colorMaterial.FresnelEffectExponent = 7.654321f;
        }
    }
```

```cpp
void SetFresnelEffect(ApiHandle<RenderingSession> session, ApiHandle<Material> material)
{
    if (material->GetMaterialSubType() == MaterialType::Pbr)
    {
        auto pbrMaterial = material.as<PbrMaterial>();
        auto featureFlags = PbrMaterialFeatures((int32_t)pbrMaterial->GetPbrFlags() | (int32_t)PbrMaterialFeatures::FresnelEffect);
        pbrMaterial->SetPbrFlags(featureFlags);
        pbrMaterial->SetFresnelEffectColor(Color4{ 1.f, 0.5f, 0.1f, 1.f });
        pbrMaterial->SetFresnelEffectExponent(3.141592f);

    }
    else if (material->GetMaterialSubType() == MaterialType::Color)
    {
        auto colorMaterial = material.as<ColorMaterial>();
        auto featureFlags = ColorMaterialFeatures((int32_t)colorMaterial->GetColorFlags() | (int32_t)ColorMaterialFeatures::FresnelEffect);
        colorMaterial->SetColorFlags(featureFlags);
        colorMaterial->SetFresnelEffectColor(Color4{ 0.25f, 1.f, 0.25f, 1.f });
        colorMaterial->SetFresnelEffectExponent(7.654321f);
    }
}
```

## <a name="api-documentation"></a>Документирование API

* [Пбрматериалфеатурес C#](/dotnet/api/microsoft.azure.remoterendering.pbrmaterialfeatures)
* [Пбрматериалфеатурес C++](/cpp/api/remote-rendering/pbrmaterialfeatures)
* [Колорматериалфеатурес C#](/dotnet/api/microsoft.azure.remoterendering.colormaterialfeatures)
* [Колорматериалфеатурес C++](/cpp/api/remote-rendering/colormaterialfeatures)

## <a name="next-steps"></a>Дальнейшие действия

* [Материалы](../../concepts/materials.md)
* [Материалы по PBR](../../overview/features/pbr-materials.md)
* [Цветовые материалы](../../overview/features/color-materials.md)