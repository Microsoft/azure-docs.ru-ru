---
title: 'Azure бастиона: Просмотр сеанса виртуальной машины: во весь экран'
description: Узнайте, как изменить представление виртуальной машины на полноэкранное и обратно в браузере для подключения RDP или SSH в Azure бастиона.
services: bastion
author: cherylmc
ms.service: bastion
ms.topic: how-to
ms.date: 02/03/2020
ms.author: cherylmc
ms.openlocfilehash: 7db85c5e1d5f67c30adf08969c51bb4c57838fb5
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92079145"
---
# <a name="change-to-full-screen-view-for-a-vm-session-azure-bastion"></a>Переход в полноэкранный режим для сеанса виртуальной машины: Azure бастиона

Эта статья поможет вам изменить представление виртуальной машины на полноэкранное и обратно в браузере. Перед началом работы с виртуальной машиной убедитесь, что выполнены действия по [созданию узла бастиона](./tutorial-create-host-portal.md). Затем подключитесь к виртуальной машине, которую вы хотите использовать, с помощью [RDP](bastion-connect-vm-rdp.md) или [SSH](bastion-connect-vm-ssh.md).

## <a name="launch-the-clipboard-tool"></a>Запуск средства буфера обмена

В ходе удаленного сеанса откройте палитру средств доступа к буферу обмена бастиона, выбрав две стрелки, расположенные в левом центре сеанса.

![средства](./media/bastion-vm-manage/left.png)

## <a name="select-full-screen"></a>Выберите "во весь экран"

Нажмите кнопку **полноэкранного** режима, чтобы перевести сеанс в полноэкранный режим. После переключения сеанс повторно инициализируется до полного экрана.

![полноэкранный режим](./media/bastion-vm-manage/full-screen.png)
 
## <a name="next-steps"></a>Дальнейшие действия

Ознакомьтесь с [часто задаваемыми вопросами о Бастионе Azure](bastion-faq.md).
Сведения о [копировании и вставке](bastion-vm-copy-paste.md) на виртуальную машину Azure и из нее.
