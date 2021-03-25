---
title: Запись образов для решения "без кода" в Azure Перцепт Studio
description: Узнайте, как записывать образы с помощью Azure Перцепт DK в Azure Перцепт Studio для решения, не имеющего кода.
author: elqu20
ms.author: v-elqu
ms.service: azure-percept
ms.topic: how-to
ms.date: 02/12/2021
ms.custom: template-how-to
ms.openlocfilehash: d6cece6ee3079ba9f400f40026ca26ea36668710
ms.sourcegitcommit: a8ff4f9f69332eef9c75093fd56a9aae2fe65122
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105024648"
---
# <a name="capture-images-for-a-vision-project-in-azure-percept-studio"></a>Запись образов для проекта концепции в Azure Перцепт Studio

Следуйте указаниям этого раздела, чтобы записать образы с помощью SoM концепции Azure Перцепт DK для существующего проекта концепции в Azure Перцепт Studio. Если вы еще не создали концепцию проекта, см. Руководство по [созданию непрограммных концепций](./tutorial-nocode-vision.md).

## <a name="prerequisites"></a>Предварительные требования

- Azure Percept DK (DevKit).
- [Подписка Azure.](https://azure.microsoft.com/free/)
- [Выполненная настройка Azure Percept DK](./quickstart-percept-dk-set-up.md). Предполагается, что вы уже подключили DevKit к сети Wi-Fi, создали центр Интернета вещей и подключили к нему DevKit.
- [Проект концепции без кода](./tutorial-nocode-vision.md)

## <a name="capture-images"></a>Образы записи

1. Включите свой DevKit.

1. Перейдите в [Azure Percept Studio](https://go.microsoft.com/fwlink/?linkid=2135819).

1. В левой части страницы Обзор щелкните **устройства**.

    :::image type="content" source="./media/how-to-capture-images/overview-devices-inline.png" alt-text="Экран обзора Azure Percept Studio." lightbox="./media/how-to-capture-images/overview-devices.png":::

1. Выберите DevKit из списка.

    :::image type="content" source="./media/how-to-capture-images/select-device.png" alt-text="Список устройств перцепт.":::

1. На странице устройства нажмите кнопку **захватить образы для проекта**.

    :::image type="content" source="./media/how-to-capture-images/capture-images.png" alt-text="Страница устройств перцепт с доступными действиями в списке.":::

1. В окне **Capture Image (запись образа** ) выполните следующие действия.

    1. В раскрывающемся меню **проекта** выберите проект концепции, для которого требуется получить образы.

    1. Нажмите кнопку **Просмотр потока устройства** , чтобы убедиться, что камера правильно размещена SoM концепции.

    1. Щелкните **взять фотографию** , чтобы записать изображение.

    1. Кроме того, установите флажок для параметра **Автоматическая запись образа** , чтобы настроить таймер для записи образа.

        1. Выберите предпочтительную скорость работы с образами в разделе **Частота захвата**.
        1. Выберите общее число образов, которые вы хотите получать в рамках **целевого объекта**.

    :::image type="content" source="./media/how-to-capture-images/take-photo.png" alt-text="Экран захвата изображения.":::

Все изображения будут доступны в [пользовательское визуальное распознавание](https://www.customvision.ai/).

## <a name="next-steps"></a>Следующие шаги

[Протестируйте и переучить модель концепции без кода](../cognitive-services/custom-vision-service/test-your-model.md).