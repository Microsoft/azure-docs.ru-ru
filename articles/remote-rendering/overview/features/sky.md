---
title: Отражения неба
description: Узнайте, как настроить карту среды для отражения неба
author: florianborn71
ms.author: flborn
ms.date: 02/07/2020
ms.topic: article
ms.custom: devx-track-csharp
ms.openlocfilehash: 9c5ad4b21b428f38bbd4d9f7d19fa633c5161b5c
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99594186"
---
# <a name="sky-reflections"></a>Отражения неба

В Удаленной отрисовке Azure для реалистичного освещения объектов применяется текстура неба. Для приложений дополненной реальности эта текстура должна убедительно имитировать обстановку реального мира. В этой статье описано, как изменить текстуру неба.

> [!NOTE]
> Текстура неба также называется *картой среды*. Эти термины являются взаимозаменяемыми.

## <a name="object-lighting"></a>Освещение объекта

Удаленная отрисовка Azure использует *физически корректную отрисовку* для вычисления реалистичного освещения. Вы можете добавлять в сцену разные [источники света](lights.md), но хорошая текстура неба дает самый лучший результат.

На рисунках ниже показаны результаты освещения разных поверхностей только текстурой неба:

| Неровность  | 0                                        | 0,25                                          | 0,5                                          | 0,75                                          | 1                                          |
|:----------:|:----------------------------------------:|:---------------------------------------------:|:--------------------------------------------:|:---------------------------------------------:|:------------------------------------------:|
| Неметаллические  | ![Неэлектрический, грубость = 0](media/dielectric-0.png)   | ![Неэлектрический, грубость = 0,25](media/dielectric-0.25.png)  | ![Неэлектрический, грубость = 0,5](media/dielectric-0.5.png)  | ![Неэлектрический, грубость = 0,75](media/dielectric-0.75.png)  | ![Неэлектрический, грубость = 1](media/dielectric-1.png)  |
| Металлические      | ![Металл, грубость = 0](media/metallic-0.png)  | ![Металлическое, грубое = 0,25](media/metallic-0.25.png)    | ![Металл, грубость = 0,5](media/metallic-0.5.png)    | ![Металл, грубое значение = 0,75](media/metallic-0.75.png)    | ![Металл, грубость = 1](media/metallic-1.png)    |

Дополнительные сведения о модели освещения см. в главе о [материалах](../../concepts/materials.md).

> [!IMPORTANT]
> Удаленная отрисовка Azure использует текстуру неба для моделей освещения. Она не отрисовывает цвет неба в качестве фона, так как приложения дополненной реальности уже имеют надлежащий фон, то есть реальный мир.

## <a name="changing-the-sky-texture"></a>Изменение текстуры неба

Чтобы изменить карту среды, достаточно [загрузить текстуру](../../concepts/textures.md) и изменить значение `SkyReflectionSettings` для сеанса.

```cs
async void ChangeEnvironmentMap(RenderingSession session)
{
    try
    {
        Texture skyTex = await session.Connection.LoadTextureFromSasAsync(new LoadTextureFromSasOptions("builtin://VeniceSunset", TextureType.CubeMap));
        session.Connection.SkyReflectionSettings.SkyReflectionTexture = skyTex;
    }
    catch (RRException exception)
    {
        System.Console.WriteLine($"Setting sky reflection failed: {exception.Message}");
    }
}
```

```cpp
void ChangeEnvironmentMap(ApiHandle<RenderingSession> session)
{
    LoadTextureFromSasOptions params;
    params.TextureType = TextureType::CubeMap;
    params.TextureUri = "builtin://VeniceSunset";
    session->Connection()->LoadTextureFromSasAsync(params, [&](Status status, ApiHandle<Texture> res) {
        if (status == Status::OK)
        {
            ApiHandle<SkyReflectionSettings> settings = session->Connection()->GetSkyReflectionSettings();
            settings->SetSkyReflectionTexture(res);
        }
        else
        {
            printf("Texture loading failed!\n");
        }
    });
}
```

Обратите внимание, что выше используется вариант `LoadTextureFromSasAsync`, так как загружаются встроенные текстуры. Если загрузка производится из [связанных хранилищ BLOB-объектов](../../how-tos/create-an-account.md#link-storage-accounts), используйте вариант `LoadTextureAsync`.

## <a name="sky-texture-types"></a>Типы текстур неба

Вы можете использовать в качестве карты среды *[cubemap](https://en.wikipedia.org/wiki/Cube_mapping)* или *двухмерную текстуру*.

Все текстуры должны иметь [поддерживаемый формат текстуры](../../concepts/textures.md#supported-texture-formats). Для текстур неба не нужно предоставлять MIP-карты.

### <a name="cube-environment-maps"></a>Кубические карты среды

Для информации ниже приведен пример развернутого объекта cubemap.

![Развернутый объект cubemap](media/Cubemap-example.png)

Используйте `RenderingSession.Connection.LoadTextureAsync`/ `LoadTextureFromSasAsync` с `TextureType.CubeMap` для загрузки текстур кубической карты (cubemap).

### <a name="sphere-environment-maps"></a>Сферические карты среды

При использовании двухмерной текстуры в качестве карты среды изображение должно находиться в [ пространстве сферических координат](https://en.wikipedia.org/wiki/Spherical_coordinate_system).

![Изображение неба вычисляется в сферических координатах](media/spheremap-example.png)

Используйте `RenderingSession.Connection.LoadTextureAsync` с `TextureType.Texture2D` для загрузки сферических карт среды.

## <a name="built-in-environment-maps"></a>Встроенные карты среды

Удаленная отрисовка Azure предоставляет несколько встроенных карт среды, которые всегда доступны. Все встроенные карты среды являются кубическими (cubemap).

|Идентификатор                         | Описание                                              | Иллюстрация                                                      |
|-----------------------------------|:---------------------------------------------------------|:-----------------------------------------------------------------:|
|builtin://Autoshop                 | Разнообразные полосы света, яркое базовое освещение для закрытых помещений    | ![Скибокса автомагазина, используемая для освещения объекта](media/autoshop.png)
|builtin://BoilerRoom               | Яркое освещение для сцен в закрытых помещениях, свет от нескольких окон      | ![Боилеррум скибокс, используемый для освещения объекта](media/boiler-room.png)
|builtin://ColorfulStudio           | Окрашенный свет в закрытом помещении средней освещенности  | ![Колорфулстудио скибокс, используемый для освещения объекта](media/colorful-studio.png)
|builtin://Hangar                   | Равномерное неинтенсивное освещение в помещении большого размера                     | ![Смаллхангар скибокс, используемый для освещения объекта](media/hangar.png)
|builtin://IndustrialPipeAndValve   | Тускло освещенное закрытое помещение с контрастными участками              | ![Индустриалпипеандвалве скибокс, используемый для освещения объекта](media/industrial-pipe-and-valve.png)
|builtin://Lebombo                  | Равномерное дневное освещение в комнате, ярко освещенная область окна     | ![Лебомбо скибокс, используемый для освещения объекта](media/lebombo.png)
|builtin://SataraNight              | Темное ночное небо и земля со множеством источников света   | ![Сатаранигхт скибокс, используемый для освещения объекта](media/satara-night.png)
|builtin://SunnyVondelpark          | Яркий солнечный свет и контрастные тени                      | ![Суннивонделпарк скибокс, используемый для освещения объекта](media/sunny-vondelpark.png)
|builtin://Syferfontein             | Чистое небо, умеренное освещение от земли            | ![Сиферфонтеин скибокс, используемый для освещения объекта](media/syferfontein.png)
|builtin://TearsOfSteelBridge       | Умеренное освещение с неравномерными светлыми и затененными участками                         | ![Теарсофстилбридже скибокс, используемый для освещения объекта](media/tears-of-steel-bridge.png)
|builtin://VeniceSunset             | Вечернее закатное освещение перед сумерками                    | ![Веницесунсет скибокс, используемый для освещения объекта](media/venice-sunset.png)
|builtin://WhippleCreekRegionalPark | Освещение ярких тонов зеленого и белого цвета, затененная земля | ![Вхипплекрикрегионалпарк скибокс, используемый для освещения объекта](media/whipple-creek-regional-park.png)
|builtin://WinterRiver              | Дневное освещение с ярко и равномерно освещенной землей                 | ![Винтерривер скибокс, используемый для освещения объекта](media/winter-river.png)
|builtin://DefaultSky               | То же, что и TearsOfSteelBridge                               | ![Дефаултски скибокс, используемый для освещения объекта](media/tears-of-steel-bridge.png)

## <a name="api-documentation"></a>Документирование API

* [C# Рендерингконнектион. Скирефлектионсеттингс, свойство](/dotnet/api/microsoft.azure.remoterendering.renderingconnection.skyreflectionsettings)
* [C++ Рендерингконнектион:: Скирефлектионсеттингс ()](/cpp/api/remote-rendering/renderingconnection#skyreflectionsettings)

## <a name="next-steps"></a>Дальнейшие действия

* [Освещение](../../overview/features/lights.md)
* [Материалы](../../concepts/materials.md)
* [Текстуры](../../concepts/textures.md)
