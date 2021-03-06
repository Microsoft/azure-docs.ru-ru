---
title: Поддержка удаленной работы брандмауэра Azure
description: В этой статье показано, как брандмауэр Azure может поддерживать требования к удаленной работе.
services: firewall
author: vhorne
ms.service: firewall
ms.topic: conceptual
ms.date: 05/04/2020
ms.author: victorh
ms.openlocfilehash: 3c0e2033ee559af38a6816bdfa611eea86b14dea
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94658321"
---
# <a name="azure-firewall-remote-work-support"></a>Поддержка удаленной работы брандмауэра Azure

Брандмауэр Azure — это управляемая облачная служба безопасности сети, которая защищает ресурсы виртуальной сети Azure. Это высокодоступная служба с полным отслеживанием состояния и неограниченными возможностями облачного масштабирования.

## <a name="virtual-desktop-infrastructure-vdi-deployment-support"></a>Поддержка развертывания инфраструктуры виртуальных рабочих столов (VDI)

Работа с домашними политиками требует от многих ИТ организаций решать фундаментальные изменения в емкости, сети, безопасности и управлении. Сотрудники не защищаются многоуровневой политикой безопасности, связанной с локальными службами, при работе из дома. Развертывания инфраструктуры виртуальных рабочих столов (VDI) в Azure помогут организациям быстро реагировать на эту смену среды. Однако необходим способ защиты входящего и исходящего доступа к Интернету в этих развертываниях VDI. Вы можете использовать [правила ДНаТ](rule-processing.md) брандмауэра Azure вместе с возможностями фильтрации на основе [аналитики угроз](threat-intel.md) для защиты развертываний VDI.

## <a name="azure-windows-virtual-desktop-support"></a>Поддержка виртуальных рабочих столов Windows в Azure

Виртуальный рабочий стол Windows — это комплексная служба виртуализации для настольных систем и приложений, работающая в Azure. Это единственная инфраструктура виртуальных рабочих столов (VDI), которая обеспечивает упрощенное управление, многосеансовую Windows 10, оптимизацию для Microsoft 365 приложений для предприятия и поддержку сред службы удаленных рабочих столов (RDS). Вы можете развернуть и масштабировать настольные компьютеры и приложения Windows в Azure за считаные минуты, а также получить встроенные функции обеспечения безопасности и соответствия требованиям. Виртуальному рабочему столу Windows не требуется открывать входящий доступ к виртуальной сети. Однако необходимо разрешить набор исходящих сетевых подключений для виртуальных машин виртуальных рабочих столов Windows, работающих в виртуальной сети. Дополнительные сведения см. в статье [Использование Брандмауэра Azure для защиты развертываний виртуального рабочего стола Windows](protect-windows-virtual-desktop.md).

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о [виртуальных рабочих столах Windows](../virtual-desktop/index.yml).