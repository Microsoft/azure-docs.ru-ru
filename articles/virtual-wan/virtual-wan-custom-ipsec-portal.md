---
title: 'Настройка настраиваемой политики IPsec для виртуальной глобальной сети Azure: портал | Документация Майкрософт'
description: Узнайте, как настроить пользовательскую политику IPsec для виртуальной глобальной сети Azure с помощью портала.
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: how-to
ms.date: 10/14/2019
ms.author: cherylmc
ms.openlocfilehash: 07eae453441ea8bff81d7cdb60f9c46c08a22829
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91851185"
---
# <a name="configure-a-custom-ipsec-policy-for-virtual-wan-using-the-portal"></a>Настройка настраиваемой политики IPsec для виртуальной глобальной сети с помощью портала

Вы можете настроить пользовательскую политику IPsec для VPN-подключения виртуальной глобальной сети в портал Azure. Пользовательские политики полезны, когда нужно, чтобы обе стороны (локальный и VPN-шлюз Azure) использовали одни и те же параметры для фазы IKE 1 и фазы IKE 2.

## <a name="working-with-custom-policies"></a>Работа с пользовательскими политиками

[!INCLUDE [IPsec](../../includes/virtual-wan-ipsec-custom-include.md)]

## <a name="configure-a-policy"></a>Настройка политики

1. **Нахождение виртуального концентратора**. В браузере откройте [портал Azure](https://aka.ms/azurevirtualwanpreviewfeatures) и выполните вход с помощью учетной записи Azure. Перейдите к виртуальному ресурсу глобальной сети и найдите виртуальный концентратор, к которому подключен сайт VPN.
2. **Выберите VPN-сайт**. На странице обзора концентратора щелкните **VPN (сайт-сайт)** и выберите VPN-сайт, для которого требуется настроить настраиваемую политику IPSec.

   ![select](./media/virtual-wan-custom-ipsec-portal/locate.png)
3. **Измените VPN-подключение**. В **контекстном меню** выберите **изменить VPN-подключение**.

   ![изменение;](./media/virtual-wan-custom-ipsec-portal/contextmenu.png)
4. **Настройте параметры**. На странице **изменение VPN-подключения** измените значение параметра IPSec с default на Custom и настройте политику IPSec. Нажмите кнопку **Save** (Сохранить), чтобы сохранить настройки.

   ![Настройка и сохранение](./media/virtual-wan-custom-ipsec-portal/edit.png)

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о Виртуальной глобальной сети см. в [этой статье](virtual-wan-about.md).