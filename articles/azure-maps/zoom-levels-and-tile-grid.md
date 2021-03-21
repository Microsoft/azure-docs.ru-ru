---
title: Уровни масштаба и сетка плиток в картах Microsoft Azure
description: Узнайте, как задать уровни масштаба в Azure Maps. См. раздел Преобразование географических координат в координаты пикселей, координаты плиток и куадкэйс. Просмотр примеров кода.
author: anastasia-ms
ms.author: v-stharr
ms.date: 07/14/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.openlocfilehash: 21c2329ec58e414ebfedaa4c49d5f690f47cac72
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92913897"
---
# <a name="zoom-levels-and-tile-grid"></a>Уровни увеличения и сетка фрагментов

В службе "Карты Azure" используется сферическая система координат проекции Меркатора (EPSG: 3857). Проекция — это математическая модель, используемая для преобразования сферического земного шара в плоскую карту. Сферический проекция Меркатора растягивает карту на полюсам, чтобы создать квадратную карту. Эта проекция значительно искажает масштаб и область схемы, но имеет два важных свойства, которые перевешивают это искажение:

- Это согласованная проекция, которая означает, что она сохраняет форму относительно мелких объектов. Сохранение формы мелких объектов особенно важно при отображении аэрофотосъемки изображений. Например, мы хотим избежать искажения формы зданий. Квадратные здания должны выглядеть как квадратные, а не прямоугольные.
- Это цилиндрическая проекция. Север и Юг всегда работают вверх и вниз, а Запад и Восток всегда остаются слева и справа. 

Чтобы оптимизировать производительность извлечения и отображения карт, эта схема делится на квадратные плитки. В Azure Maps SDK используются плитки, размер которых составляет 512 x 512 пикселей для карт дороги и меньше 256 x 256 пикселей для вспомогательных изображений. Azure Maps предоставляет растровые и векторные плитки для 23 уровней масштаба с числом от 0 до 22. На нулевом уровне масштаба вся карта мира помещается на одной плитке:

:::image type="content" source="./media/zoom-levels-and-tile-grid/world0.png" alt-text="Плитка карты мира":::

На первом уровне масштаба карта мира отображается в 2 фрагментах: квадрат 2 x 2.

:::image type="content" source="./media/zoom-levels-and-tile-grid/map-2x2-tile-layout.png" alt-text="Макет плитки карты 2x2":::

Каждый дополнительный уровень масштаба четыре делит плитки предыдущего, создавая сетку с<sup>масштабом</sup><sup>2 x 2</sup> . 22-й уровень масштаба — это сетка 2<sup>22</sup> x 2<sup>22</sup>, или 4 194 304 x 4 194 304 фрагмента (всего 17 592 186 044 416 фрагментов).

Azure Maps Интерактивные элементы управления картой для веб-и Android поддерживают 25 уровней масштабирования с номерами от 0 до 24. Хотя дорожные данные будут доступны только на уровнях масштабирования в, если плитки доступны.

В следующей таблице приведен полный список значений для уровней масштаба, где размер плитки составляет 512 пикселей в квадрате широты 0:

|Уровень масштаба|м/пиксель|м/сторона фрагмента|
|--- |--- |--- |
| 0 | 156 543 | 40075017 |
| 1 | 78 271,5 | 20037508 |
| 2 | 39 135,8 | 10018754 |
| 3 | 19567,88 | 5009377,1 |
| 4 | 9783,94 | 2504688,5 |
| 5 | 4891,97 | 1252344,3 |
| 6 | 2445,98 | 626172,1 |
| 7 | 1222,99 | 313086,1 |
| 8 | 611,5 | 156 543 |
| 9 | 305,75 | 78 271,5 |
| 10 | 152,87 | 39 135,8 |
| 11 | 76,44 | 19 567,9 |
| 12 | 38,219 | 9783,94 |
| 13 | 19,109 | 4891,97 |
| 14 | 9,555 | 2445,98 |
| 15 | 4,777 | 1222,99 |
| 16 | 2,3887 | 611,496 |
| 17 | 1,1943 | 305,748 |
| 18 | 0,5972 | 152,874 |
| 19 | 0,14929 | 76,437 |
| 20 | 0,14929 | 38,2185 |
| 21 | 0,074646 | 19,10926 |
| 22 | 0,037323 | 9,55463 |
| 23 | 0,0186615 | 4,777315 |
| 24 | 0,00933075 | 2,3886575 |

## <a name="pixel-coordinates"></a>Координаты в пикселях

После выбора проекции и масштабирования для каждого уровня масштаба можно преобразовать географические координаты в координаты пикселей. Полная ширина и высота изображения схемы мира для определенного уровня масштаба вычисляется следующим образом:

```javascript
var mapWidth = tileSize * Math.pow(2, zoom);

var mapHeight = mapWidth;
```

Так как ширина и высота карт различаются на каждом уровне масштаба, то есть координаты в пикселях. Пиксель в левом верхнем углу на карте всегда имеет координаты пикселей (0, 0). Пиксель в правом нижнем углу на карте имеет координаты в пикселях *(ширина-1, высота-1)* или ссылается на уравнения в предыдущем разделе *(тилесизе \* 2 <sup>Zoom</sup>– 1, тилесизе \* 2 <sup>Zoom</sup>– 1)*. Например, при использовании 512 квадратных плиток на уровне 2 диапазон координат пикселя — от (0, 0) до (2047, 2047), как показано ниже:

:::image type="content" border="false" source="./media/zoom-levels-and-tile-grid/map-width-height.png" alt-text="Схема, показывающая размеры пикселей":::

Учитывая широту и долготу в градусах, а также уровень детализации, координаты пиксельной оси рассчитываются следующим образом:

```javascript
var sinLatitude = Math.sin(latitude * Math.PI/180);

var pixelX = ((longitude + 180) / 360) * tileSize * Math.pow(2, zoom);

var pixelY = (0.5 – Math.log((1 + sinLatitude) / (1 – sinLatitude)) / (4 * Math.PI)) * tileSize * Math.pow(2, zoom);
```

Значения широты и долготы считаются на WGS 84. Хотя Azure Maps использует сферическую проекцию, очень важно преобразовать все географические координаты в общую. WGS 84 — это выбранная база. Значение долготы принимается в диапазоне от-180 градусов до + 180 градусов, а значение широты должно быть обрезано до диапазона от-85,05112878 до 85,05112878. Применяя эти значения, вы помогаете избежать несовпадения в полюсам, и это гарантирует, что проецируемая схема является фигурой в квадрате.

## <a name="tile-coordinates"></a>Координаты плитки

Чтобы оптимизировать производительность при извлечении и отображении карт, визуализированная схема обрезана до плиток. Количество пикселей и число плиток на каждом уровне различаются:

```javascript
var numberOfTilesWide = Math.pow(2, zoom);

var numberOfTilesHigh = numberOfTilesWide;
```

Каждой плитке присваиваются координаты координат от (0,0) в верхнем левом углу до *(2 <sup>масштаба</sup>– 1, 2 <sup>масштаба</sup>– 1)* в правом нижнем углу. Например, на уровне масштабирования 3 Координаты плитки находятся в диапазоне от (0, 0) до (7, 7) следующим образом:

:::image type="content" border="false" source="./media/zoom-levels-and-tile-grid/map-tiles-x-y-coordinates-7x7.png" alt-text="Схема координат плитки":::

Учитывая пару координат XY в пикселях, можно легко определить координаты мозаичной диаграммы плитки, содержащей этот пиксель:

```javascript
var tileX = Math.floor(pixelX / tileSize);

var tileY = Math.floor(pixelY / tileSize);
```

Плитки вызываются по уровню масштабирования. Координаты x и y соответствуют положению плитки в сетке для этого уровня масштабирования.

При определении используемого уровня масштабирования Помните, что каждое расположение находится в фиксированной позиции на плитке. В результате количество плиток, необходимых для отображения заданной заданной территории, зависит от конкретного расположения сетки масштаба на карте мира. Например, есть две точки, расстояние между которыми составляет 900 метров. Для отображения этого маршрута *может* понадобиться только три плитки на 17-ом уровне масштабирования. Но если западная точка находится а правой части своего фрагмента, а восточная — в левой части своего, может понадобиться четыре фрагмента:

:::image type="content" border="false" source="./media/zoom-levels-and-tile-grid/zoomdemo_scaled.png" alt-text="Демонстрация увеличения масштаба":::

После выбора уровня масштаба вычисляются значения x и y. Верхняя левая плитка в каждой сетке масштабирования равна x = 0, y = 0; Нижняя правая плитка — x = 2<sup>Zoom-1</sup>, y = 2<sup>Масштаб-1</sup>.

Вот сетка масштабирования для первого уровня масштаба:

:::image type="content" border="false" source="./media/zoom-levels-and-tile-grid/api_x_y.png" alt-text="Сетка масштаба для первого уровня масштаба":::

## <a name="quadkey-indices"></a>Индексы куадкэй

Некоторые платформы сопоставлений используют `quadkey` соглашение об именовании индексирования, объединяющее Зи координаты в строку с одним измерением, именуемую `quadtree` Keys или `quadkeys` Short. Каждый `quadkey` однозначно определяет одну плитку на определенном уровне детализации, которую можно использовать в качестве ключа в общих индексах сбалансированного дерева базы данных. Azure Maps пакеты SDK поддерживают наложение слоев мозаики, использующих `quadkey` соглашение об именовании, в дополнение к другим соглашениям об именовании, как описано в документе [Добавление мозаичного слоя](map-add-tile-layer.md) .

> [!NOTE]
> `quadkeys`Соглашение об именовании работает только для одного или более уровней масштаба. Уровень масштабирования для поддержки Azure Maps SDK 0, который является одной плиткой карты для всего мира. 

Чтобы преобразовать координаты плитки в `quadkey` , биты координат Y и X чередуются, а результат интерпретируется как число с основанием 4 (с ведущими нулями) и преобразуется в строку. Например, указанные координаты координат мозаики (3, 5) на уровне 3 `quadkey` определяются следующим образом:

```
tileX = 3 = 011 (base 2)

tileY = 5 = 101 (base 2)

quadkey = 100111 (base 2) = 213 (base 4) = "213"
```

`Qquadkeys` иметь несколько интересных свойств. Во-первых, длина `quadkey` (число цифр) равна уровню масштаба соответствующей плитки. Во-вторых, элемент `quadkey` любой плитки начинается с его `quadkey` родительской плитки (содержащей плитку на предыдущем уровне). Как показано в примере ниже, плитка 2 является родителем плиток от 20 до 23:

:::image type="content" border="false" source="./media/zoom-levels-and-tile-grid/quadkey-tile-pyramid.png" alt-text="Пирамидальная с плиткой куадкэй":::

Наконец, `quadkeys` укажите одномерный ключ индекса, который обычно сохраняет сходство плиток в пространстве координат. Другими словами, две плитки с координатами, `quadkeys` расположенными поблизости, обычно имеют относительно близко друг к другу. Это важно для оптимизации производительности базы данных, поскольку соседние плитки часто запрашиваются в группах, и желательно удерживать эти плитки на одних и тех же дисковых блоках, чтобы свести к минимальному числу операций чтения с диска.

## <a name="tile-math-source-code"></a>Исходный код математической плитки

В следующем примере кода показано, как реализовать функции, описанные в этом документе. Эти функции можно легко перевести на другие языки программирования по мере необходимости.

#### <a name="c"></a>[C#](#tab/csharp)

```csharp
using System;
using System.Text;

namespace AzureMaps
{
    /// <summary>
    /// Tile System math for the Spherical Mercator projection coordinate system (EPSG:3857)
    /// </summary>
    public static class TileMath
    {
        //Earth radius in meters.
        private const double EarthRadius = 6378137;

        private const double MinLatitude = -85.05112878;
        private const double MaxLatitude = 85.05112878;
        private const double MinLongitude = -180;
        private const double MaxLongitude = 180;

        /// <summary>
        /// Clips a number to the specified minimum and maximum values.
        /// </summary>
        /// <param name="n">The number to clip.</param>
        /// <param name="minValue">Minimum allowable value.</param>
        /// <param name="maxValue">Maximum allowable value.</param>
        /// <returns>The clipped value.</returns>
        private static double Clip(double n, double minValue, double maxValue)
        {
            return Math.Min(Math.Max(n, minValue), maxValue);
        }

        /// <summary>
        /// Calculates width and height of the map in pixels at a specific zoom level from -180 degrees to 180 degrees.
        /// </summary>
        /// <param name="zoom">Zoom Level to calculate width at</param>
        /// <param name="tileSize">The size of the tiles in the tile pyramid.</param>
        /// <returns>Width and height of the map in pixels</returns>
        public static double MapSize(double zoom, int tileSize)
        {
            return Math.Ceiling(tileSize * Math.Pow(2, zoom));
        }

        /// <summary>
        /// Calculates the Ground resolution at a specific degree of latitude in meters per pixel.
        /// </summary>
        /// <param name="latitude">Degree of latitude to calculate resolution at</param>
        /// <param name="zoom">Zoom level to calculate resolution at</param>
        /// <param name="tileSize">The size of the tiles in the tile pyramid.</param>
        /// <returns>Ground resolution in meters per pixels</returns>
        public static double GroundResolution(double latitude, double zoom, int tileSize)
        {
            latitude = Clip(latitude, MinLatitude, MaxLatitude);
            return Math.Cos(latitude * Math.PI / 180) * 2 * Math.PI * EarthRadius / MapSize(zoom, tileSize);
        }

        /// <summary>
        /// Determines the map scale at a specified latitude, level of detail, and screen resolution.
        /// </summary>
        /// <param name="latitude">Latitude (in degrees) at which to measure the map scale.</param>
        /// <param name="zoom">Level of detail, from 1 (lowest detail) to 23 (highest detail).</param>
        /// <param name="screenDpi">Resolution of the screen, in dots per inch.</param>
        /// <param name="tileSize">The size of the tiles in the tile pyramid.</param>
        /// <returns>The map scale, expressed as the denominator N of the ratio 1 : N.</returns>
        public static double MapScale(double latitude, double zoom, int screenDpi, int tileSize)
        {
            return GroundResolution(latitude, zoom, tileSize) * screenDpi / 0.0254;
        }

        /// <summary>
        /// Global Converts a Pixel coordinate into a geospatial coordinate at a specified zoom level. 
        /// Global Pixel coordinates are relative to the top left corner of the map (90, -180)
        /// </summary>
        /// <param name="pixel">Pixel coordinates in the format of [x, y].</param>  
        /// <param name="zoom">Zoom level</param>
        /// <param name="tileSize">The size of the tiles in the tile pyramid.</param>
        /// <returns>A position value in the format [longitude, latitude].</returns>
        public static double[] GlobalPixelToPosition(double[] pixel, double zoom, int tileSize)
        {
            var mapSize = MapSize(zoom, tileSize);

            var x = (Clip(pixel[0], 0, mapSize - 1) / mapSize) - 0.5;
            var y = 0.5 - (Clip(pixel[1], 0, mapSize - 1) / mapSize);

            return new double[] {
                360 * x,    //Longitude
                90 - 360 * Math.Atan(Math.Exp(-y * 2 * Math.PI)) / Math.PI  //Latitude
            };
        }

        /// <summary>
        /// Converts a point from latitude/longitude WGS-84 coordinates (in degrees) into pixel XY coordinates at a specified level of detail.
        /// </summary>
        /// <param name="position">Position coordinate in the format [longitude, latitude]</param>
        /// <param name="zoom">Zoom level.</param>
        /// <param name="tileSize">The size of the tiles in the tile pyramid.</param> 
        /// <returns>A global pixel coordinate.</returns>
        public static double[] PositionToGlobalPixel(double[] position, int zoom, int tileSize)
        {
            var latitude = Clip(position[1], MinLatitude, MaxLatitude);
            var longitude = Clip(position[0], MinLongitude, MaxLongitude);

            var x = (longitude + 180) / 360;
            var sinLatitude = Math.Sin(latitude * Math.PI / 180);
            var y = 0.5 - Math.Log((1 + sinLatitude) / (1 - sinLatitude)) / (4 * Math.PI);

            var mapSize = MapSize(zoom, tileSize);

            return new double[] {
                 Clip(x * mapSize + 0.5, 0, mapSize - 1),
                 Clip(y * mapSize + 0.5, 0, mapSize - 1)
            };
        }

        /// <summary>
        /// Converts pixel XY coordinates into tile XY coordinates of the tile containing the specified pixel.
        /// </summary>
        /// <param name="pixel">Pixel coordinates in the format of [x, y].</param>  
        /// <param name="tileSize">The size of the tiles in the tile pyramid.</param>
        /// <param name="tileX">Output parameter receiving the tile X coordinate.</param>
        /// <param name="tileY">Output parameter receiving the tile Y coordinate.</param>
        public static void GlobalPixelToTileXY(double[] pixel, int tileSize, out int tileX, out int tileY)
        {
            tileX = (int)(pixel[0] / tileSize);
            tileY = (int)(pixel[1] / tileSize);
        }

        /// <summary>
        /// Performs a scale transform on a global pixel value from one zoom level to another.
        /// </summary>
        /// <param name="pixel">Pixel coordinates in the format of [x, y].</param>  
        /// <param name="oldZoom">The zoom level in which the input global pixel value is from.</param>  
        /// <returns>A scale pixel coordinate.</returns>
        public static double[] ScaleGlobalPixel(double[] pixel, double oldZoom, double newZoom)
        {
            var scale = Math.Pow(2, oldZoom - newZoom);

            return new double[] { pixel[0] * scale, pixel[1] * scale };
        }

        /// <summary>
        /// Performs a scale transform on a set of global pixel values from one zoom level to another.
        /// </summary>
        /// <param name="pixels">A set of global pixel value from the old zoom level. Points are in the format [x,y].</param>
        /// <param name="oldZoom">The zoom level in which the input global pixel values is from.</param>
        /// <param name="newZoom">The new zoom level in which the output global pixel values should be aligned with.</param>
        /// <returns>A set of global pixel values that has been scaled for the new zoom level.</returns>
        public static double[][] ScaleGlobalPixels(double[][] pixels, double oldZoom, double newZoom)
        {
            var scale = Math.Pow(2, oldZoom - newZoom);

            var output = new System.Collections.Generic.List<double[]>();
            foreach (var p in pixels)
            {
                output.Add(new double[] { p[0] * scale, p[1] * scale });
            }

            return output.ToArray();
        }

        /// <summary>
        /// Converts tile XY coordinates into a global pixel XY coordinates of the upper-left pixel of the specified tile.
        /// </summary>
        /// <param name="tileX">Tile X coordinate.</param>
        /// <param name="tileY">Tile Y coordinate.</param>
        /// <param name="tileSize">The size of the tiles in the tile pyramid.</param>
        /// <param name="pixelX">Output parameter receiving the X coordinate of the point, in pixels.</param>  
        /// <param name="pixelY">Output parameter receiving the Y coordinate of the point, in pixels.</param>  
        public static double[] TileXYToGlobalPixel(int tileX, int tileY, int tileSize)
        {
            return new double[] { tileX * tileSize, tileY * tileSize };
        }

        /// <summary>
        /// Converts tile XY coordinates into a quadkey at a specified level of detail.
        /// </summary>
        /// <param name="tileX">Tile X coordinate.</param>
        /// <param name="tileY">Tile Y coordinate.</param>
        /// <param name="zoom">Zoom level</param>
        /// <returns>A string containing the quadkey.</returns>
        public static string TileXYToQuadKey(int tileX, int tileY, int zoom)
        {
            var quadKey = new StringBuilder();
            for (int i = zoom; i > 0; i--)
            {
                char digit = '0';
                int mask = 1 << (i - 1);
                if ((tileX & mask) != 0)
                {
                    digit++;
                }
                if ((tileY & mask) != 0)
                {
                    digit++;
                    digit++;
                }
                quadKey.Append(digit);
            }
            return quadKey.ToString();
        }

        /// <summary>
        /// Converts a quadkey into tile XY coordinates.
        /// </summary>
        /// <param name="quadKey">Quadkey of the tile.</param>
        /// <param name="tileX">Output parameter receiving the tile X coordinate.</param>
        /// <param name="tileY">Output parameter receiving the tile Y coordinate.</param>
        /// <param name="zoom">Output parameter receiving the zoom level.</param>
        public static void QuadKeyToTileXY(string quadKey, out int tileX, out int tileY, out int zoom)
        {
            tileX = tileY = 0;
            zoom = quadKey.Length;
            for (int i = zoom; i > 0; i--)
            {
                int mask = 1 << (i - 1);
                switch (quadKey[zoom - i])
                {
                    case '0':
                        break;

                    case '1':
                        tileX |= mask;
                        break;

                    case '2':
                        tileY |= mask;
                        break;

                    case '3':
                        tileX |= mask;
                        tileY |= mask;
                        break;

                    default:
                        throw new ArgumentException("Invalid QuadKey digit sequence.");
                }
            }
        }

        /// <summary>
        /// Calculates the XY tile coordinates that a coordinate falls into for a specific zoom level.
        /// </summary>
        /// <param name="position">Position coordinate in the format [longitude, latitude]</param>
        /// <param name="zoom">Zoom level</param>
        /// <param name="tileSize">The size of the tiles in the tile pyramid.</param>
        /// <param name="tileX">Output parameter receiving the tile X position.</param>
        /// <param name="tileY">Output parameter receiving the tile Y position.</param>
        public static void PositionToTileXY(double[] position, int zoom, int tileSize, out int tileX, out int tileY)
        {
            var latitude = Clip(position[1], MinLatitude, MaxLatitude);
            var longitude = Clip(position[0], MinLongitude, MaxLongitude);

            var x = (longitude + 180) / 360;
            var sinLatitude = Math.Sin(latitude * Math.PI / 180);
            var y = 0.5 - Math.Log((1 + sinLatitude) / (1 - sinLatitude)) / (4 * Math.PI);

            //tileSize needed in calculations as in rare cases the multiplying/rounding/dividing can make the difference of a pixel which can result in a completely different tile. 
            var mapSize = MapSize(zoom, tileSize);
            tileX = (int)Math.Floor(Clip(x * mapSize + 0.5, 0, mapSize - 1) / tileSize);
            tileY = (int)Math.Floor(Clip(y * mapSize + 0.5, 0, mapSize - 1) / tileSize);
        }

        /// <summary>
        /// Calculates the tile quadkey strings that are within a specified viewport.
        /// </summary>
        /// <param name="position">Position coordinate in the format [longitude, latitude]</param>
        /// <param name="zoom">Zoom level</param>
        /// <param name="width">The width of the map viewport in pixels.</param>
        /// <param name="height">The height of the map viewport in pixels.</param>
        /// <param name="tileSize">The size of the tiles in the tile pyramid.</param>
        /// <returns>A list of quadkey strings that are within the specified viewport.</returns>
        public static string[] GetQuadkeysInView(double[] position, int zoom, int width, int height, int tileSize)
        {
            var p = PositionToGlobalPixel(position, zoom, tileSize);

            var top = p[1] - height * 0.5;
            var left = p[0] - width * 0.5;

            var bottom = p[1] + height * 0.5;
            var right = p[0] + width * 0.5;

            var tl = GlobalPixelToPosition(new double[] { left, top }, zoom, tileSize);
            var br = GlobalPixelToPosition(new double[] { right, bottom }, zoom, tileSize);

            //Boudning box in the format: [west, south, east, north];
            var bounds = new double[] { tl[0], br[1], br[0], tl[1] };

            return GetQuadkeysInBoundingBox(bounds, zoom, tileSize);
        }

        /// <summary>
        /// Calculates the tile quadkey strings that are within a bounding box at a specific zoom level.
        /// </summary>
        /// <param name="bounds">A bounding box defined as an array of numbers in the format of [west, south, east, north].</param>
        /// <param name="zoom">Zoom level to calculate tiles for.</param>
        /// <param name="tileSize">The size of the tiles in the tile pyramid.</param>
        /// <returns>A list of quadkey strings.</returns>
        public static string[] GetQuadkeysInBoundingBox(double[] bounds, int zoom, int tileSize)
        {
            var keys = new System.Collections.Generic.List<string>();

            if (bounds != null && bounds.Length >= 4)
            {
                PositionToTileXY(new double[] { bounds[3], bounds[0] }, zoom, tileSize, out int tlX, out int tlY);
                PositionToTileXY(new double[] { bounds[1], bounds[2] }, zoom, tileSize, out int brX, out int brY);

                for (int x = tlX; x <= brX; x++)
                {
                    for (int y = tlY; y <= brY; y++)
                    {
                        keys.Add(TileXYToQuadKey(x, y, zoom));
                    }
                }
            }

            return keys.ToArray();
        }

        /// <summary>
        /// Calculates the bounding box of a tile.
        /// </summary>
        /// <param name="tileX">Tile X coordinate</param>
        /// <param name="tileY">Tile Y coordinate</param>
        /// <param name="zoom">Zoom level</param>
        /// <param name="tileSize">The size of the tiles in the tile pyramid.</param>
        /// <returns>A bounding box of the tile defined as an array of numbers in the format of [west, south, east, north].</returns>
        public static double[] TileXYToBoundingBox(int tileX, int tileY, double zoom, int tileSize)
        {
            //Top left corner pixel coordinates
            var x1 = (double)(tileX * tileSize);
            var y1 = (double)(tileY * tileSize);

            //Bottom right corner pixel coordinates
            var x2 = (double)(x1 + tileSize);
            var y2 = (double)(y1 + tileSize);

            var nw = GlobalPixelToPosition(new double[] { x1, y1 }, zoom, tileSize);
            var se = GlobalPixelToPosition(new double[] { x2, y2 }, zoom, tileSize);

            return new double[] { nw[0], se[1], se[0], nw[1] };
        }

        /// <summary>
        /// Calculates the best map view (center, zoom) for a bounding box on a map.
        /// </summary>
        /// <param name="bounds">A bounding box defined as an array of numbers in the format of [west, south, east, north].</param>
        /// <param name="mapWidth">Map width in pixels.</param>
        /// <param name="mapHeight">Map height in pixels.</param>
        /// <param name="padding">Width in pixels to use to create a buffer around the map. This is to keep markers from being cut off on the edge</param>
        /// <param name="tileSize">The size of the tiles in the tile pyramid.</param>
        /// <param name="latitude">Output parameter receiving the center latitude coordinate.</param>
        /// <param name="longitude">Output parameter receiving the center longitude coordinate.</param>
        /// <param name="zoom">Output parameter receiving the zoom level</param>
        public static void BestMapView(double[] bounds, double mapWidth, double mapHeight, int padding, int tileSize, out double centerLat, out double centerLon, out double zoom)
        {
            if (bounds == null || bounds.Length < 4)
            {
                centerLat = 0;
                centerLon = 0;
                zoom = 1;
                return;
            }

            double boundsDeltaX;

            //Check if east value is greater than west value which would indicate that bounding box crosses the antimeridian.
            if (bounds[2] > bounds[0])
            {
                boundsDeltaX = bounds[2] - bounds[0];
                centerLon = (bounds[2] + bounds[0]) / 2;
            }
            else
            {
                boundsDeltaX = 360 - (bounds[0] - bounds[2]);
                centerLon = ((bounds[2] + bounds[0]) / 2 + 360) % 360 - 180;
            }

            var ry1 = Math.Log((Math.Sin(bounds[1] * Math.PI / 180) + 1) / Math.Cos(bounds[1] * Math.PI / 180));
            var ry2 = Math.Log((Math.Sin(bounds[3] * Math.PI / 180) + 1) / Math.Cos(bounds[3] * Math.PI / 180));
            var ryc = (ry1 + ry2) / 2;

            centerLat = Math.Atan(Math.Sinh(ryc)) * 180 / Math.PI;

            var resolutionHorizontal = boundsDeltaX / (mapWidth - padding * 2);

            var vy0 = Math.Log(Math.Tan(Math.PI * (0.25 + centerLat / 360)));
            var vy1 = Math.Log(Math.Tan(Math.PI * (0.25 + bounds[3] / 360)));
            var zoomFactorPowered = (mapHeight * 0.5 - padding) / (40.7436654315252 * (vy1 - vy0));
            var resolutionVertical = 360.0 / (zoomFactorPowered * tileSize);

            var resolution = Math.Max(resolutionHorizontal, resolutionVertical);

            zoom = Math.Log(360 / (resolution * tileSize), 2);
        }
    }
}
```

#### <a name="typescript"></a>[TypeScript](#tab/typescript)

```typescript
module AzureMaps {

    /** Tile System math for the Spherical Mercator projection coordinate system (EPSG:3857) */
    export class TileMath {
        //Earth radius in meters.
        private static EarthRadius = 6378137;

        private static MinLatitude = -85.05112878;
        private static MaxLatitude = 85.05112878;
        private static MinLongitude = -180;
        private static MaxLongitude = 180;

        /**
         * Clips a number to the specified minimum and maximum values.
         * @param n The number to clip.
         * @param minValue Minimum allowable value.
         * @param maxValue Maximum allowable value.
         * @returns The clipped value.
         */
        private static Clip(n: number, minValue: number, maxValue: number): number {
            return Math.min(Math.max(n, minValue), maxValue);
        }

        /**
         * Calculates width and height of the map in pixels at a specific zoom level from -180 degrees to 180 degrees.
         * @param zoom Zoom Level to calculate width at.
         * @param tileSize The size of the tiles in the tile pyramid.
         * @returns Width and height of the map in pixels.
         */
        public static MapSize(zoom: number, tileSize: number): number {
            return Math.ceil(tileSize * Math.pow(2, zoom));
        }

        /**
         * Calculates the Ground resolution at a specific degree of latitude in the meters per pixel.
         * @param latitude Degree of latitude to calculate resolution at.
         * @param zoom Zoom level.
         * @param tileSize The size of the tiles in the tile pyramid.
         * @returns Ground resolution in meters per pixels.
         */
        public static GroundResolution(latitude: number, zoom: number, tileSize: number): number {
            latitude = this.Clip(latitude, this.MinLatitude, this.MaxLatitude);
            return Math.cos(latitude * Math.PI / 180) * 2 * Math.PI * this.EarthRadius / this.MapSize(zoom, tileSize);
        }

        /**
         * Determines the map scale at a specified latitude, level of detail, and screen resolution.
         * @param latitude Latitude (in degrees) at which to measure the map scale.
         * @param zoom Zoom level.
         * @param screenDpi Resolution of the screen, in dots per inch.
         * @param tileSize The size of the tiles in the tile pyramid.
         * @returns The map scale, expressed as the denominator N of the ratio 1 : N.
         */
        public static MapScale(latitude: number, zoom: number, screenDpi: number, tileSize: number): number {
            return this.GroundResolution(latitude, zoom, tileSize) * screenDpi / 0.0254;
        }

        /**
         * Global Converts a Pixel coordinate into a geospatial coordinate at a specified zoom level.
         * Global Pixel coordinates are relative to the top left corner of the map (90, -180).
         * @param pixel Pixel coordinates in the format of [x, y].
         * @param zoom Zoom level.
         * @param tileSize The size of the tiles in the tile pyramid.
         * @returns A position value in the format [longitude, latitude].
         */
        public static GlobalPixelToPosition(pixel: number[], zoom: number, tileSize: number): number[] {
            var mapSize = this.MapSize(zoom, tileSize);

            var x = (this.Clip(pixel[0], 0, mapSize - 1) / mapSize) - 0.5;
            var y = 0.5 - (this.Clip(pixel[1], 0, mapSize - 1) / mapSize);

            return [
                360 * x,    //Longitude
                90 - 360 * Math.atan(Math.exp(-y * 2 * Math.PI)) / Math.PI  //Latitude
            ];
        }

        /**
         * Converts a point from latitude/longitude WGS-84 coordinates (in degrees) into pixel XY coordinates at a specified level of detail.
         * @param position Position coordinate in the format [longitude, latitude].
         * @param zoom Zoom level.
         * @param tileSize The size of the tiles in the tile pyramid.
         * @returns A pixel coordinate 
         */
        public static PositionToGlobalPixel(position: number[], zoom: number, tileSize: number): number[] {
            var latitude = this.Clip(position[1], this.MinLatitude, this.MaxLatitude);
            var longitude = this.Clip(position[0], this.MinLongitude, this.MaxLongitude);

            var x = (longitude + 180) / 360;
            var sinLatitude = Math.sin(latitude * Math.PI / 180);
            var y = 0.5 - Math.log((1 + sinLatitude) / (1 - sinLatitude)) / (4 * Math.PI);

            var mapSize = this.MapSize(zoom, tileSize);

            return [
                this.Clip(x * mapSize + 0.5, 0, mapSize - 1),
                this.Clip(y * mapSize + 0.5, 0, mapSize - 1)
            ];
        }

        /**
         * Converts pixel XY coordinates into tile XY coordinates of the tile containing the specified pixel.
         * @param pixel Pixel coordinates in the format of [x, y].
         * @param tileSize The size of the tiles in the tile pyramid.
         * @returns Tile XY coordinates.
         */
        public static GlobalPixelToTileXY(pixel: number[], tileSize: number): { tileX: number, tileY: number } {
            return {
                tileX: Math.round(pixel[0] / tileSize),
                tileY: Math.round(pixel[1] / tileSize)
            };
        }

        /**
         * Performs a scale transform on a global pixel value from one zoom level to another.
         * @param pixel Pixel coordinates in the format of [x, y].
         * @param oldZoom The zoom level in which the input global pixel value is from.
         * @param newZoom The new zoom level in which the output global pixel value should be aligned with.
         */
        public static ScaleGlobalPixel(pixel: number[], oldZoom: number, newZoom: number): number[] {
            var scale = Math.pow(2, oldZoom - newZoom);

            return [pixel[0] * scale, pixel[1] * scale];
        }

        /// <summary>
        /// Performs a scale transform on a set of global pixel values from one zoom level to another.
        /// </summary>
        /// <param name="points">A set of global pixel value from the old zoom level. Points are in the format [x,y].</param>
        /// <param name="oldZoom">The zoom level in which the input global pixel values is from.</param>
        /// <param name="newZoom">The new zoom level in which the output global pixel values should be aligned with.</param>
        /// <returns>A set of global pixel values that has been scaled for the new zoom level.</returns>
        public static ScaleGlobalPixels(pixels: number[][], oldZoom: number, newZoom: number): number[][] {
            var scale = Math.pow(2, oldZoom - newZoom);

            var output: number[][] = [];
            for (var i = 0, len = pixels.length; i < len; i++) {
                output.push([pixels[i][0] * scale, pixels[i][1] * scale]);
            }

            return output;
        }

        /**
         * Converts tile XY coordinates into a global pixel XY coordinates of the upper-left pixel of the specified tile.
         * @param tileX Tile X coordinate.
         * @param tileY Tile Y coordinate.
         * @param tileSize The size of the tiles in the tile pyramid.
         * @returns Pixel coordinates in the format of [x, y].
         */
        public static TileXYToGlobalPixel(tileX, tileY, tileSize): number[] {
            return [tileX * tileSize, tileY * tileSize];
        }

        /**
         * Converts tile XY coordinates into a quadkey at a specified level of detail.
         * @param tileX Tile X coordinate.
         * @param tileY Tile Y coordinate.
         * @param zoom Zoom level.
         * @returns A string containing the quadkey.
         */
        public static TileXYToQuadKey(tileX: number, tileY: number, zoom: number): string {
            var quadKey: number[] = [];
            for (var i = zoom; i > 0; i--) {
                var digit = 0;
                var mask = 1 << (i - 1);

                if ((tileX & mask) != 0) {
                    digit++;
                }

                if ((tileY & mask) != 0) {
                    digit += 2
                }

                quadKey.push(digit);
            }
            return quadKey.join('');
        }

        /**
         * Converts a quadkey into tile XY coordinates.
         * @param quadKey Quadkey of the tile.
         * @returns Tile XY cocorindates and zoom level for the specified quadkey.
         */
        public static QuadKeyToTileXY(quadKey: string): { tileX: number, tileY: number, zoom: number } {
            var tileX = 0;
            var tileY = 0;
            var zoom = quadKey.length;

            for (var i = zoom; i > 0; i--) {
                var mask = 1 << (i - 1);
                switch (quadKey[zoom - i]) {
                    case '0':
                        break;

                    case '1':
                        tileX |= mask;
                        break;

                    case '2':
                        tileY |= mask;
                        break;

                    case '3':
                        tileX |= mask;
                        tileY |= mask;
                        break;

                    default:
                        throw "Invalid QuadKey digit sequence.";
                }
            }

            return {
                tileX: tileX,
                tileY: tileY,
                zoom: zoom
            };
        }

        /**
         * Calculates the XY tile coordinates that a coordinate falls into for a specific zoom level.
         * @param position Position coordinate in the format [longitude, latitude].
         * @param zoom Zoom level.
         * @param tileSize The size of the tiles in the tile pyramid.
         * @returns Tiel XY coordinates.
         */
        public static PositionToTileXY(position: number[], zoom: number, tileSize: number): { tileX: number, tileY: number } {
            var latitude = this.Clip(position[1], this.MinLatitude, this.MaxLatitude);
            var longitude = this.Clip(position[0], this.MinLongitude, this.MaxLongitude);

            var x = (longitude + 180) / 360;
            var sinLatitude = Math.sin(latitude * Math.PI / 180);
            var y = 0.5 - Math.log((1 + sinLatitude) / (1 - sinLatitude)) / (4 * Math.PI);

            //tileSize needed in calculations as in rare cases the multiplying/rounding/dividing can make the difference of a pixel which can result in a completely different tile. 
            var mapSize = this.MapSize(zoom, tileSize);

            return {
                tileX: Math.floor(this.Clip(x * mapSize + 0.5, 0, mapSize - 1) / tileSize),
                tileY: Math.floor(this.Clip(y * mapSize + 0.5, 0, mapSize - 1) / tileSize)
            };
        }

        /**
         * Calculates the tile quadkey strings that are within a specified viewport.
         * @param position Position coordinate in the format [longitude, latitude].
         * @param zoom Zoom level.
         * @param width The width of the map viewport in pixels.
         * @param height The height of the map viewport in pixels.
         * @param tileSize The size of the tiles in the tile pyramid.
         * @returns A list of quadkey strings that are within the specified viewport.
         */
        public static GetQuadkeysInView(position: number[], zoom: number, width: number, height: number, tileSize: number): string[] {
            var p = this.PositionToGlobalPixel(position, zoom, tileSize);

            var top = p[1] - height * 0.5;
            var left = p[0] - width * 0.5;

            var bottom = p[1] + height * 0.5;
            var right = p[0] + width * 0.5;

            var tl = this.GlobalPixelToPosition([left, top], zoom, tileSize);
            var br = this.GlobalPixelToPosition([right, bottom], zoom, tileSize);

            //Boudning box in the format: [west, south, east, north];
            var bounds = [tl[0], br[1], br[0], tl[1]];

            return this.GetQuadkeysInBoundingBox(bounds, zoom, tileSize);
        }

        /**
         * Calculates the tile quadkey strings that are within a bounding box at a specific zoom level.
         * @param bounds A bounding box defined as an array of numbers in the format of [west, south, east, north].
         * @param zoom Zoom level to calculate tiles for.
         * @param tileSize The size of the tiles in the tile pyramid.
         * @returns A list of quadkey strings.
         */
        public static GetQuadkeysInBoundingBox(bounds: number[], zoom: number, tileSize: number): string[] {
            var keys: string[] = [];

            if (bounds != null && bounds.length >= 4) {
                var tl = this.PositionToTileXY([bounds[0], bounds[3]], zoom, tileSize);
                var br = this.PositionToTileXY([bounds[2], bounds[1]], zoom, tileSize);

                for (var x = tl[0]; x <= br[0]; x++) {
                    for (var y = tl[1]; y <= br[1]; y++) {
                        keys.push(this.TileXYToQuadKey(x, y, zoom));
                    }
                }
            }

            return keys;
        }

        /**
         * Calculates the bounding box of a tile.
         * @param tileX Tile X coordinate.
         * @param tileY Tile Y coordinate.
         * @param zoom Zoom level.
         * @param tileSize The size of the tiles in the tile pyramid.
         * @returns A bounding box of the tile defined as an array of numbers in the format of [west, south, east, north].
         */
        public static TileXYToBoundingBox(tileX: number, tileY: number, zoom: number, tileSize: number): number[] {
            //Top left corner pixel coordinates
            var x1 = tileX * tileSize;
            var y1 = tileY * tileSize;

            //Bottom right corner pixel coordinates
            var x2 = x1 + tileSize;
            var y2 = y1 + tileSize;

            var nw = this.GlobalPixelToPosition([x1, y1], zoom, tileSize);
            var se = this.GlobalPixelToPosition([x2, y2], zoom, tileSize);

            return [nw[0], se[1], se[0], nw[1]];
        }

        /**
         * Calculates the best map view (center, zoom) for a bounding box on a map.
         * @param bounds A bounding box defined as an array of numbers in the format of [west, south, east, north]. 
         * @param mapWidth Map width in pixels.
         * @param mapHeight Map height in pixels.
         * @param padding Width in pixels to use to create a buffer around the map. This is to keep markers from being cut off on the edge.
         * @param tileSize The size of the tiles in the tile pyramid.
         * @returns The center and zoom level to best position the map view over the provided bounding box.
         */
        public static BestMapView(bounds: number[], mapWidth: number, mapHeight: number, padding: number, tileSize: number): { center: number[], zoom: number } {
            if (bounds == null || bounds.length < 4) {
                return {
                    center: [0, 0],
                    zoom: 1
                };
            }

            var boundsDeltaX: number;
            var centerLat: number;
            var centerLon: number;

            //Check if east value is greater than west value which would indicate that bounding box crosses the antimeridian.
            if (bounds[2] > bounds[0]) {
                boundsDeltaX = bounds[2] - bounds[0];
                centerLon = (bounds[2] + bounds[0]) / 2;
            }
            else {
                boundsDeltaX = 360 - (bounds[0] - bounds[2]);
                centerLon = ((bounds[2] + bounds[0]) / 2 + 360) % 360 - 180;
            }

            var ry1 = Math.log((Math.sin(bounds[1] * Math.PI / 180) + 1) / Math.cos(bounds[1] * Math.PI / 180));
            var ry2 = Math.log((Math.sin(bounds[3] * Math.PI / 180) + 1) / Math.cos(bounds[3] * Math.PI / 180));
            var ryc = (ry1 + ry2) / 2;

            centerLat = Math.atan(Math.sinh(ryc)) * 180 / Math.PI;

            var resolutionHorizontal = boundsDeltaX / (mapWidth - padding * 2);

            var vy0 = Math.log(Math.tan(Math.PI * (0.25 + centerLat / 360)));
            var vy1 = Math.log(Math.tan(Math.PI * (0.25 + bounds[3] / 360)));
            var zoomFactorPowered = (mapHeight * 0.5 - padding) / (40.7436654315252 * (vy1 - vy0));
            var resolutionVertical = 360.0 / (zoomFactorPowered * tileSize);

            var resolution = Math.max(resolutionHorizontal, resolutionVertical);

            var zoom = Math.log2(360 / (resolution * tileSize));

            return {
                center: [centerLon, centerLat],
                zoom: zoom
            };
        }
    }
}
```

* * *

> [!NOTE]
> Интерактивные элементы управления картой в Azure Maps пакете SDK имеют вспомогательные функции для преобразования геопространственных положений и точек просмотра. 
> - [Веб-пакет SDK: вычисление пикселей и позиций в карте](/javascript/api/azure-maps-control/atlas.map#pixelstopositions-pixel---)

## <a name="next-steps"></a>Дальнейшие действия

Прямой доступ к плиткам карт из Azure Maps служб RESTFUL:

> [!div class="nextstepaction"]
> [Получение плиток карт](/rest/api/maps/render/getmaptile)

> [!div class="nextstepaction"]
> [Получение плиток потока трафика](/rest/api/maps/traffic/gettrafficflowtile)

> [!div class="nextstepaction"]
> [Получение плиток "инцидент трафика"](/rest/api/maps/traffic/gettrafficincidenttile)

Дополнительные сведения о геопространственных понятиях:

> [!div class="nextstepaction"]
> [Глоссарий Azure Maps](glossary.md)