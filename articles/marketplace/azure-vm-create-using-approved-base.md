---
title: Создание предложения виртуальной машины Azure с помощью утвержденной базы Azure Marketplace
description: Узнайте, как создать предложение виртуальной машины из утвержденной базы.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
author: emuench
ms.author: krsh
ms.date: 03/10/2021
ms.openlocfilehash: 94d21cb82290e59ebb003969a566c1bfc877713e
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103200448"
---
# <a name="how-to-create-a-virtual-machine-using-an-approved-base"></a>Создание виртуальной машины с помощью утвержденной базы

В этой статье описывается, как создать виртуальную машину, содержащую предварительно настроенную операционную систему, с помощью Azure. Если это не совместимо с вашим решением, можно [создать и настроить локальную виртуальную машину](azure-vm-create-using-own-image.md) с помощью утвержденной операционной системы.

> [!NOTE]
> Перед началом этой процедуры ознакомьтесь с [техническими требованиями](marketplace-virtual-machines.md#technical-requirements) для предложений виртуальных машин Azure, включая требования к виртуальному жесткому диску (VHD).

## <a name="select-an-approved-base-image"></a>Выберите утвержденный базовый образ

Выберите в качестве основы один из следующих образов Windows или Linux.

### <a name="windows"></a>Windows

- [Windows Server](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/microsoftwindowsserver.windowsserver?tab=Overview)
- SQL Server [2019](https://azuremarketplace.microsoft.com/marketplace/apps/microsoftsqlserver.sql2019-ws2019?tab=Overview), [2014](https://azuremarketplace.microsoft.com/marketplace/apps/microsoftsqlserver.sql2014sp3-ws2012r2?tab=Overview), [2012](https://azuremarketplace.microsoft.com/marketplace/apps/microsoftsqlserver.sql2012sp4-ws2012r2?tab=Overview)

### <a name="linux"></a>Linux

Azure предлагает широкий диапазон утвержденных дистрибутивов Linux. Текущий список см. в статье [Дистрибутивы Linux, рекомендованные для использования в Azure](../virtual-machines/linux/endorsed-distros.md).

## <a name="create-vm-on-the-azure-portal"></a>Создание виртуальной машины на портал Azure

1. Войдите на [портал Azure](https://ms.portal.azure.com/).
2. Выберите **Виртуальные машины**.
3. Выберите **+ Добавить** , чтобы открыть экран **Создание виртуальной машины** .
4. Выберите изображение из раскрывающегося списка или выберите **Просмотреть все общедоступные и частные образы** , чтобы найти или просмотреть все доступные образы виртуальных машин.
5. Чтобы создать виртуальную машину **Gen 2** , перейдите на вкладку **Дополнительно** и выберите параметр **Gen 2** .

    :::image type="content" source="media/create-vm/vm-gen-option.png" alt-text="Выберите Gen 1 или Gen 2.":::

6. Выберите размер виртуальной машины для развертывания.

    :::image type="content" source="media/create-vm/create-virtual-machine-sizes.png" alt-text="Выберите рекомендуемый размер виртуальной машины для выбранного образа.":::

7. Укажите другие необходимые сведения для создания виртуальной машины.
8. Выберите **Просмотр и создание**, чтобы проверить выбранные параметры. Когда появится сообщение **Проверка пройдена** , выберите  **создать**.

Azure начнет подготовку указанной вами виртуальной машины. Чтобы отслеживать ход выполнения, выберите вкладку **виртуальные машины** в меню слева. После создания состояние виртуальной машины изменится на **работает**.

## <a name="configure-the-vm"></a>Настройка виртуальной машины

В этом разделе описано обновление и подготовка виртуальной машины Azure, а также определение ее размера. Эти шаги нужны для того, чтобы подготовить виртуальную машину к развертыванию в Azure Marketplace.

### <a name="connect-to-your-vm"></a>Подключение к виртуальной машине

Обратитесь к следующей документации, чтобы подключиться к виртуальной машине [Windows](../virtual-machines/windows/connect-logon.md) или [Linux](../virtual-machines/linux/ssh-from-windows.md#connect-to-your-vm) .

### <a name="install-the-most-current-updates"></a>Установка самых свежих обновлений

[!INCLUDE [Discussion of most current updates](includes/most-current-updates.md)]

### <a name="perform-additional-security-checks"></a>Выполнение дополнительных проверок безопасности

[!INCLUDE [Discussion of addition security checks](includes/additional-security-checks.md)]

### <a name="perform-custom-configuration-and-scheduled-tasks"></a>Выполнение пользовательской настройки и запланированные задачи

[!INCLUDE [Discussion of custom configuration and scheduled tasks](includes/custom-config.md)]

[!INCLUDE [Discussion of addition security checks](includes/size-connect-generalize.md)]

## <a name="next-steps"></a>Следующие шаги

- Рекомендуемый следующий шаг: [Проверьте образ виртуальной машины](azure-vm-image-test.md) , чтобы убедиться, что он соответствует требованиям к публикации Azure Marketplace. Это необязательно.
- Если вы не хотите протестировать образ виртуальной машины, войдите в [Центр партнеров](https://partner.microsoft.com/) , чтобы опубликовать образ.
- Если вы столкнулись с трудностями при создании нового виртуального жесткого диска на основе Azure, см. раздел [вопросы и ответы о виртуальной машине для Azure Marketplace](azure-vm-create-faq.md).
