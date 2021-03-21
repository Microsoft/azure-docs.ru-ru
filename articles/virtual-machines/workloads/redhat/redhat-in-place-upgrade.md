---
title: Обновление на месте образов Red Hat Enterprise Linux в Azure
description: Узнайте, как выполнить обновление на месте с образов Red Hat Enterprise 7. x до последней версии 8. x.
author: mathapli
ms.service: virtual-machines
ms.subservice: redhat
ms.collection: linux
ms.topic: article
ms.date: 04/16/2020
ms.author: alsin
ms.openlocfilehash: 1be0904cc640eff5af7a77bba3abd6aa062991a8
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101676077"
---
# <a name="red-hat-enterprise-linux-in-place-upgrades"></a>Red Hat Enterprise Linux обновления на месте

В этой статье приводятся инструкции по обновлению на месте с Red Hat Enterprise Linux (RHEL) 7 до Red Hat Enterprise Linux 8. В инструкциях используется `leapp` средство в Azure. Во время обновления на месте существующая операционная система RHEL 7 заменяется версией RHEL 8.

>[!Note] 
> Предложения SQL Server на Red Hat Enterprise Linux не поддерживают обновления на месте в Azure.

## <a name="what-to-expect-during-the-upgrade"></a>Что следует предполагать во время обновления
Во время обновления система перезапускается несколько раз. Последний перезапуск обновляет виртуальную машину до последней версии RHEL 8. 

Процесс обновления может занять от 20 минут до 2 часов. Общее время зависит от нескольких факторов, таких как размер виртуальной машины и число пакетов, установленных в системе.

## <a name="preparations"></a>Подготовке
Red Hat и Azure рекомендуют использовать обновление на месте, чтобы перевести систему в следующую основную версию. 

Прежде чем начать обновление, учитывайте следующие моменты. 

>[!Important] 
> Создайте моментальный снимок образа перед началом обновления.

* Убедитесь, что вы используете последнюю версию RHEL 7. В настоящее время последняя версия — RHEL 7,9. Если вы используете Заблокированную версию и не можете выполнить обновление до RHEL 7,9, выполните следующие [действия, чтобы перейти к репозиторию, не ЕУС (поддержка расширенных обновлений)](./redhat-rhui.md#switch-a-rhel-7x-vm-back-to-non-eus-remove-a-version-lock).

* Выполните следующую команду, чтобы проверить обновление и убедиться, что оно успешно завершено. Команда должна создать файл */вар/лог/леапп/leapp-report.txt* . В этом файле объясняется процесс, что происходит, а также указывается, возможно ли обновление.

    >[!NOTE]
    > Используйте учетную запись root для выполнения команд в этой статье. 

    ```bash
    leapp preupgrade --no-rhsm
    ```
* Убедитесь, что последовательная консоль работает. Эта консоль будет использоваться для мониторинга в процессе обновления.

* Включите корневой доступ SSH в *каталог/etc/ssh/sshd_config*:
    1. Откройте файл *каталог/etc/ssh/sshd_config*.
    1. Найдите `#PermitRootLogin yes`.
    1. Удалите знак решетки ( `#` ), чтобы раскомментировать строку.

## <a name="upgrade-steps"></a>Действия по обновлению

Внимательно выполните эти действия. Рекомендуется выполнить обновление на тестовом компьютере, прежде чем использовать его в рабочих экземплярах.

1. Выполните `yum` обновление, чтобы получить последние пакеты клиентов.
    ```bash
    yum update -y
    ```

1. Установите `leapp-client-package`.
    ```bash
    yum install leapp-rhui-azure
    ```
    
1. На [портале Red Hat](https://access.redhat.com/articles/3664871)получите файл *леапп-Дата. tar. gz* , содержащий *repomap.csv* и *pes-events.js*. Извлеките файл *леапп-Дата. tar. gz* .
    1. Скачайте файл *леапп-Дата. tar. gz* .
    1. Извлеките содержимое и удалите файл. Используйте следующую команду:
    ```bash
    tar -xzf leapp-data12.tar.gz -C /etc/leapp/files && rm leapp-data12.tar.gz
    ```

1. Добавьте `answers` файл для `leapp` .
    ```bash
    leapp answer --section remove_pam_pkcs11_module_check.confirm=True --add
    ``` 

1. Запустите обновление.
    ```bash
    leapp upgrade --no-rhsm
    ```
1.  После `leapp upgrade` успешного завершения команды вручную перезагрузите систему, чтобы завершить процесс. Система недоступна, так как она перезапускается несколько раз. Отслеживайте процесс с помощью последовательной консоли.

1.  Убедитесь, что обновление успешно завершено.
    ```bash
    uname -a && cat /etc/redhat-release
    ```

1. После завершения обновления удалите корневой доступ SSH.
    1. Откройте файл *каталог/etc/ssh/sshd_config*.
    1. Найдите `#PermitRootLogin yes`.
    1. Добавьте знак решетки ( `#` ), чтобы закомментировать строку.

1. Перезапустите службу SSHD, чтобы применить изменения.
    ```bash
    systemctl restart sshd
    ```
## <a name="common-problems"></a>Распространенные проблемы

Следующие ошибки обычно происходят в случае `leapp preupgrade` сбоя процесса или `leapp upgrade` сбоя процесса.

* **Ошибка**: для следующих отключенных шаблонов подключаемых модулей совпадения не найдены.

    ```plaintext
    STDERR:
    No matches found for the following disabled plugin patterns: subscription-manager
    Warning: Packages marked by Leapp for upgrade not found in repositories metadata: gpg-pubkey
    ```

    **Решение**. Отключите подключаемый модуль диспетчера подписки. Отключите его, изменив файл */ЕТК/Юм/плугинконф.д/субскриптион-Манажер.конф* и изменив `enabled` на `enabled=0` .

    Эта ошибка возникает `yum` , если подключаемый модуль диспетчера подписки, который включен, не используется для `PAYG` виртуальных машин.

* **Ошибка**: возможные проблемы с удаленным входом с помощью root.

    Эта ошибка может появиться в случае `leapp preupgrade` сбоя:

    ```structured-text
    ============================================================
                         UPGRADE INHIBITED
    ============================================================
    
    Upgrade has been inhibited due to the following problems:
        1. Inhibitor: Possible problems with remote login using root account
    Consult the pre-upgrade report for details and possible remediation.
    
    ============================================================
                         UPGRADE INHIBITED
    ============================================================
    ```
    **Решение**. Включите корневой доступ в */etc/sshd_config*.

    Эта ошибка возникает, когда доступ root SSH не включен в */etc/sshd_config*. Дополнительные сведения см. в разделе " [подготовительные](#preparations) действия" этой статьи. 


## <a name="next-steps"></a>Дальнейшие действия
* Дополнительные сведения о [образах Red Hat в Azure](./redhat-images.md).
* Дополнительные сведения о [инфраструктуре обновления Red Hat](./redhat-rhui.md).
* Дополнительные сведения о [предложении RHEL BYOS](./byos.md).
* Дополнительные сведения о процессах обновления Red Hat на месте см. в разделе [обновление с RHEL 7 до RHEL 8](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html-single/upgrading_from_rhel_7_to_rhel_8/index) в документации по Red Hat.
* Дополнительные сведения о политиках поддержки Red Hat для всех версий RHEL см. в разделе [Red Hat Enterprise Linux жизненный цикл](https://access.redhat.com/support/policy/updates/errata) в документации по Red Hat.