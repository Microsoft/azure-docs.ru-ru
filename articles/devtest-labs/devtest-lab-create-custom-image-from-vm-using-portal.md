---
title: Создание пользовательского образа Azure DevTest Labs из виртуальной машины | Документация Майкрософт
description: Сведения о создании пользовательского образа в Azure DevTest Labs из подготовленной виртуальной машины с использованием портала Azure
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: ad45ed6eb7f97e14ec0ca0bb89efb2967c90fc16
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87277033"
---
# <a name="create-a-custom-image-from-a-vm"></a>Создание пользовательского образа из виртуальной машины

[!INCLUDE [devtest-lab-custom-image-definition](../../includes/devtest-lab-custom-image-definition.md)]

## <a name="step-by-step-instructions"></a>Пошаговые инструкции

Вы можете создать пользовательский образ из подготовленной виртуальной машины, а затем использовать его для создания идентичных виртуальных машин. Ниже описано, как создать пользовательский образ на основе виртуальной машины.

1. Войдите на [портал Azure](https://go.microsoft.com/fwlink/p/?LinkID=525040).

1. Выберите **все службы**, а затем в списке выберите **DevTest Labs** .

1. Из списка лабораторий выберите нужную лабораторию.  

1. На главной странице лаборатории выберите **Мои виртуальные машины**.
 
1. На панели **Мои виртуальные машины** выберите виртуальную машину, на основе которой будет создан пользовательский образ.

1. На панели управления виртуальной машиной выберите **создать пользовательский образ** в разделе **операции**.

    :::image type="content" source="./media/devtest-lab-create-template/create-custom-image.png" alt-text="Пункт меню &quot;Создание пользовательского образа&quot;":::
1. На панели **Пользовательский образ** введите имя и описание нового пользовательского образа. Эти данные отображаются в списке базовых образов при создании виртуальной машины. Пользовательский образ будет включать диск с ОС и все диски данных, прикрепленные к виртуальной машине.

    :::image type="content" source="./media/devtest-lab-create-template/create-custom-image-blade.png" alt-text="Создание настраиваемой страницы образа":::
1. Укажите, была ли выполнена на виртуальной машине программа sysprep. Если программа sysprep не была запущена на виртуальной машине, укажите, нужно ли ее запускать при создании пользовательского образа.
1. Нажмите кнопку **ОК** , чтобы создать пользовательский образ.

    Через несколько минут пользовательский образ создается и сохраняется в учетной записи хранения лаборатории. Когда пользователь лаборатории создает новую виртуальную машину, этот образ доступен в списке базовых.

    :::image type="content" source="./media/devtest-lab-create-template/custom-image-available-as-base.png" alt-text="пользовательский образ доступен в списке базовых образов":::

## <a name="related-blog-posts"></a>Связанные записи в блогах

- [Custom images or formulas? (Пользовательские изображения или формулы?)](./devtest-lab-faq.md#blog-post)
- [Copying Custom Images between Azure DevTest Labs (Копирование пользовательских образов между лабораториями для разработки и тестирования Azure)](https://www.visualstudiogeeks.com/blog/DevOps/How-To-Move-CustomImages-VHD-Between-AzureDevTestLabs#copying-custom-images-between-azure-devtest-labs)

## <a name="next-steps"></a>Дальнейшие действия

- [Добавление виртуальной машины с артефактами в лабораторию в Azure DevTest Labs](devtest-lab-add-vm.md)
