---
title: Подготовка серверов Windows Server 2003 для миграции с помощью службы "миграция Azure"
description: Узнайте, как подготовить серверы Windows Server 2003 для миграции с помощью службы "миграция Azure".
author: rahulg1190
ms.author: rahugup
ms.manager: bsiva
ms.topic: how-to
ms.date: 05/27/2020
ms.openlocfilehash: f8d3dea970d07d951467a44661e12000ba413f72
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96753759"
---
# <a name="prepare-windows-server-2003-machines-for-migration"></a>Подготовка компьютеров Windows Server 2003 к миграции

В этой статье описывается подготовка компьютеров под Windows Server 2003 для миграции в Azure. 


> [!NOTE]
> [Расширенная поддержка Windows Server 2003](/troubleshoot/azure/virtual-machines/run-win-server-2003#microsoft-windows-server-2003-end-of-support) закончилась 14 июля 2015.  Группа поддержки Azure продолжит устранение неполадок, связанных с Windows Server 2003 в Azure. Однако эта поддержка ограничена проблемами, которые не нуждаются в устранении неполадок или исправлениях на уровне операционной системы. Перенос приложений на экземпляры Azure, работающие под управлением более новой версии Windows Server, является рекомендуемым подходом к эффективному использованию гибкости и надежности облака Azure. Однако если вы по-прежнему решили перенести Windows Server 2003 в Azure, можно использовать средство миграции Azure Migration Tool, если Windows Server является виртуальной машиной, работающей в VMware или Hyper-V.


- Миграцию без агента можно использовать для переноса [виртуальных машин Hyper-V](tutorial-migrate-hyper-v.md) и [виртуальных машин VMware](tutorial-migrate-vmware.md) в Azure.
- Чтобы подключиться к виртуальным машинам Azure после миграции, на виртуальной машине Azure необходимо установить Hyper-V Integration Services. На компьютерах с Windows Server 2003 эта установка не устанавливается по умолчанию.
- Нет прямой ссылки для скачивания для установки Hyper-V Integration Services, поэтому необходимо выполнить следующие действия.
    - Для виртуальных машин Hyper-V, на которых она не установлена, можно извлечь файлы установки для Integration Services на компьютере под управлением Windows Server 2012 R2/Windows Server 2012 с ролью Hyper-V, а затем скопировать установщик на компьютер Windows Server 2003. Файлы установки недоступны на компьютерах под Windows Server 2016.
    - Для виртуальных машин VMware создайте задачу запуска, которая устанавливает Integration Services при запуске виртуальной машины Azure после миграции.


## <a name="install-on-hyper-v-vms"></a>Установка на виртуальных машинах Hyper-V

Перед миграцией проверьте, установлен ли Hyper-V Integration Services, а затем установите его при необходимости.

1. Выполните [эти инструкции](/windows-server/virtualization/hyper-v/manage/manage-hyper-v-integration-services#turn-an-integration-service-on-or-off-using-hyper-v-manager) , чтобы проверить, установлены ли они.
2. Если она не установлена, войдите на компьютер под управлением Windows Server 2012 R2/Windows Server 2012 с ролью Hyper-V.
3. Перейдите к файлу установки по адресу **C:\Windows\System32\vmguest.ISO** и подключите его.
2. Скопируйте папку установки на компьютер с Windows Server 2003 и установите Integration Services.
4. После установки можно оставить параметры по умолчанию в Integration Services. 

## <a name="install-on-vmware-vms"></a>Установка на виртуальных машинах VMware

1. Войдите на компьютер под управлением Windows Server 2012 R2/Windows Server 2012 с ролью Hyper-V.
2. Перейдите к файлу установки по адресу **C:\Windows\System32\vmguest.ISO** и подключите его.
3. Скопируйте папку установки на виртуальную машину VMware.
4. В командной строке виртуальной машины выполните команду ```gpedit.msc``` .
5. Откройте **Конфигурация компьютера**  >  **Параметры Windows**  >  **сценарии (запуск и завершение работы)**.
6. В поле **Автозагрузка**  >  **добавить**  >  **имя скрипта** введите адрес setup.exe.
7. После миграции в Azure сценарий выполняется при первом запуске виртуальной машины Azure.
8. Перезапустите виртуальную машину Azure вручную. Имеется всплывающее окно диагностики загрузки, указывающее, что требуется перезагрузка.
9. После выполнения скрипта и установки Hyper-V Integration Services на виртуальной машине Azure можно удалить скрипт из запуска.
10. После установки можно оставить параметры по умолчанию в Integration Services. 

## <a name="next-steps"></a>Дальнейшие действия

- Ознакомьтесь с требованиями к миграции для виртуальных машин [VMware](migrate-support-matrix-vmware-migration.md) и [Hyper-V](migrate-support-matrix-hyper-v-migration.md) .
- Перенос виртуальных машин [VMware](server-migrate-overview.md) и [Hyper-V](tutorial-migrate-hyper-v.md) .
