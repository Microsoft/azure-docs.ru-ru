---
title: Приступая к работе с элементом управления картой Android | Microsoft Azure Maps
description: Познакомьтесь с Azure Maps пакет SDK для Android. См. статью создание проекта в Android Studio, установка пакета SDK и создание интерактивной схемы.
author: rbrundritt
ms.author: richbrun
ms.date: 2/26/2021
ms.topic: how-to
ms.service: azure-maps
services: azure-maps
manager: cpendle
zone_pivot_groups: azure-maps-android
ms.openlocfilehash: 5888a5f34ef65fc1015b6e73af1d03368a8329b2
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102098525"
---
# <a name="getting-started-with-azure-maps-android-sdk"></a>Приступая к работе с пакетом SDK для Android для Azure Maps

Пакет SDK для Android для Azure Maps — это библиотека векторных карт для Android. В этой статье описывается, как установить пакет SDK для Android для Azure Maps и загрузить карту.

## <a name="prerequisites"></a>Предварительные требования

Не забудьте выполнить действия, описанные в разделе [Краткое руководство. Создание документа приложения Android](quick-android-map.md) .

## <a name="localizing-the-map"></a>Локализация карты

Пакет SDK для Android для Azure Maps предоставляет три разных способа настройки языка и регионального представления для карты. В следующем коде показано, как задать для языка значение французский (fr-FR), а для региона — значение Auto.

Первый вариант — передать сведения о языке и региональном представлении в класс `AzureMaps`, используя статические методы `setLanguage` и `setView` глобально. Это позволит задать язык и региональное представление по умолчанию для всех элементов управления Azure Maps, загруженных в приложение.

::: zone pivot="programming-language-java-android"

```java
static {
    //Set your Azure Maps Key.
    AzureMaps.setSubscriptionKey("<Your Azure Maps Key>");

    //Alternatively use Azure Active Directory authenticate.
    //AzureMaps.setAadProperties("<Your aad clientId>", "<Your aad AppId>", "<Your aad Tenant>");

    //Set the language to be used by Azure Maps.
    AzureMaps.setLanguage("fr-FR");

    //Set the regional view to be used by Azure Maps.
    AzureMaps.setView("Auto");
}
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
companion object {
    init {
        //Set your Azure Maps Key.
        AzureMaps.setSubscriptionKey("<Your Azure Maps Key>");

        //Alternatively use Azure Active Directory authenticate.
        //AzureMaps.setAadProperties("<Your aad clientId>", "<Your aad AppId>", "<Your aad Tenant>");
    
        //Set the language to be used by Azure Maps.
        AzureMaps.setLanguage("fr-FR");
    
        //Set the regional view to be used by Azure Maps.
        AzureMaps.setView("Auto");
    }
}
```

::: zone-end

Второй вариант — передать сведения о языке и представлении в XML-коде элемента управления карты.

```XML
<com.microsoft.azure.maps.mapcontrol.MapControl
    android:id="@+id/myMap"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:mapcontrol_language="fr-FR"
    app:mapcontrol_view="Auto"
    />
```

Третий вариант — задать язык и региональное представление карты программным образом с помощью метода `setStyle` карт. Это можно сделать в любое время, чтобы изменить язык и региональное представление для карты.

::: zone pivot="programming-language-java-android"

```java
mapControl.onReady(map -> {
    map.setStyle(
        language("fr-FR"),
        view("Auto")
    );
});
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
mapControl.onReady(OnReady { map: AzureMap ->
    map.setStyle(
        language("fr-FR"),
        view("Auto")
    )
})
```

::: zone-end

Ниже приведен пример Azure Maps с языком, для которого задано значение "fr-FR", а для регионального представления задано значение "Авто".

![Изображение карты Azure Maps с надписями на французском языке](media/how-to-use-android-map-control-library/android-localization.png)

Полный список поддерживаемых языков и региональных представлений приведен [здесь](supported-languages.md).

## <a name="navigating-the-map"></a>Перемещение по карте

Существует несколько способов, с помощью которых можно увеличить, сдвинуть, повернуть и наклонить карту. Ниже описаны различные способы перемещения по карте.

**Изменение масштаба карты**

* Коснитесь карты двумя пальцами и сведите их, чтобы уменьшить карту, или разведите пальцы, чтобы увеличить ее.
* Дважды коснитесь карты, чтобы увеличить ее масштаб на один уровень.
* Дважды коснитесь двумя пальцами, чтобы увеличить карту на один уровень.
* Коснитесь карты дважды, и при втором касании, удерживая палец на карте, проведите вверх, чтобы увеличить карту, либо проведите вниз, чтобы уменьшить ее.

**Сдвиг карты**

* Коснитесь карты и проведите в любом направлении.

**Поворот карты**

* Коснитесь карты двумя пальцами и вращайте ее.

**Наклон карты**

* Коснитесь карты двумя пальцами и проведите ими вверх или вниз.

## <a name="azure-government-cloud-support"></a>Облачная поддержка Azure для государственных организаций

Пакет SDK для Android Azure Maps поддерживает облако Azure для государственных организаций. Доступ к Azure Maps пакет SDK для Android осуществляется из того же репозитория Maven. Для подключения к облачной версии Azure Maps платформы Azure для государственных организаций необходимо выполнить следующие задачи.

В том же месте, где указаны Azure Maps сведения о проверке подлинности, добавьте следующую строку кода, чтобы сообщить карте об использовании облачного домена Azure Maps государственных организаций.

::: zone pivot="programming-language-java-android"

```java
AzureMaps.setDomain("atlas.azure.us");
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
AzureMaps.setDomain("atlas.azure.us")
```

::: zone-end

При проверке подлинности карт и служб обязательно используйте сведения о проверке подлинности Azure Maps на облачной платформе Azure для государственных организаций.

## <a name="next-steps"></a>Дальнейшие действия

Узнайте, как добавить данные наложения на карту:

> [!div class="nextstepaction"]
> [Управление аутентификацией в Azure Maps](how-to-manage-authentication.md)

> [!div class="nextstepaction"]
> [Изменение стилей карт Android](set-android-map-styles.md)

> [!div class="nextstepaction"]
> [Добавление слоя символов](how-to-add-symbol-to-android-map.md)

> [!div class="nextstepaction"]
> [Добавление слоя линий](android-map-add-line-layer.md)

> [!div class="nextstepaction"]
> [Добавление слоя многоугольников](how-to-add-shapes-to-android-map.md)
