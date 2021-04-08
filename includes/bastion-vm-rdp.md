---
title: включить файл
description: включить файл
services: bastion
author: cherylmc
ms.service: bastion
ms.topic: include
ms.date: 10/21/2020
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 083ab61d5a20bfb8e38747ae0694b1176c0a0fd1
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92521545"
---
1. Откройте [портал Azure](https://portal.azure.com). Перейдите к виртуальной машине, к которой необходимо подключиться, и выберите **Подключить**. В раскрывающемся списке выберите **Бастион**.

   :::image type="content" source="./media/bastion-vm-rdp/connect-vm.png" alt-text="Выбор элемента &quot;Бастион&quot;":::

1. Когда вы выберете этот элемент в раскрывающемся списке, появится боковая панель с тремя вкладками: RDP, SSH и "Бастион". Так как бастион был подготовлен для виртуальной сети, вкладка "Бастион" активна по умолчанию. Выберите **Использовать бастион**.

   :::image type="content" source="./media/bastion-vm-rdp/select-use-bastion.png" alt-text="Выбор элемента &quot;Использовать бастион&quot;":::

1. На странице **Подключиться с помощью Бастиона Azure** введите имя пользователя и пароль для виртуальной машины, а затем выберите **Подключить**.

   :::image type="content" source="./media/bastion-vm-rdp/connect-vm-host.png" alt-text="Подключить":::

1. Подключение по RDP к этой виртуальной машине через Бастион будет открываться непосредственно на портале Azure (через HTML5) с использованием порта 443 и службы "Бастион".

   :::image type="content" source="./media/bastion-vm-rdp/connection.png" alt-text="Подключение через порт 443":::
