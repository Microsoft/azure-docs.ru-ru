---
title: Администрирование виртуального массива с помощью Microsoft Azure StorSimple Manager | Документация Майкрософт
description: Сведения об управлении локальным виртуальным массивом StorSimple с помощью службы диспетчера устройств StorSimple на портале Azure.
services: storsimple
documentationcenter: ''
author: alkohli
manager: carmonm
editor: ''
ms.assetid: 958244a5-f9f5-455e-b7ef-71a65558872e
ms.service: storsimple
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/17/2021
ms.author: alkohli
ms.openlocfilehash: cb5b6b1a2d890fe49d78da17d549c5676557b971
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "104654261"
---
# <a name="use-the-storsimple-device-manager-service-to-administer-your-storsimple-virtual-array"></a>Администрирование виртуального массива StorSimple с помощью службы диспетчера устройств StorSimple
![Последовательность операций процесса настройки](./media/storsimple-virtual-array-manager-service-administration/manage4.png)

## <a name="overview"></a>Обзор
В этой статье описываются интерфейс службы диспетчера устройств StorSimple, способы подключения, разные доступные параметры и др. Здесь также приведены ссылки на некоторые рабочие процессы, которые можно выполнять через этот пользовательский интерфейс.

Прочитав эту статью, вы научитесь:

* Подключение к службе диспетчера устройств StorSimple
* работать с пользовательским интерфейсом службы диспетчера устройств StorSimple;
* управлять виртуальным массивом StorSimple с помощью службы диспетчера устройств StorSimple.

> [!NOTE]
> Сведения о работе с устройствами StorSimple 8000 см. в статье [Использование службы StorSimple Manager для администрирования устройства StorSimple](./storsimple-8000-manager-service-administration.md).

## <a name="connect-to-the-storsimple-device-manager-service"></a>Подключение к службе диспетчера устройств StorSimple
Служба диспетчера устройств StorSimple работает в Microsoft Azure и подключается к нескольким виртуальным массивам StorSimple. Для управления этими устройствами используется центральный портал Microsoft Azure в браузере. Чтобы подключиться к службе диспетчера устройств StorSimple, выполните следующие действия.

#### <a name="to-connect-to-the-service"></a>Подключение к службе
1. Перейдите по адресу [https://portal.azure.com](https://portal.azure.com).
2. С помощью учетной записи Майкрософт войдите на портал Microsoft Azure (расположен в правой верхней части панели).
3. Для просмотра всех диспетчеров устройств в данной подписки щелкните "Больше служб" и примените фильтр "Диспетчеры устройств StorSimple".

## <a name="use-the-storsimple-device-manager-service-to-perform-management-tasks"></a>Выполнение задач управления с помощью службы диспетчера устройств StorSimple
В следующей таблице собраны все стандартные задачи управления и сложные рабочие процессы, которые могут быть выполнены в колонке сводных данных службы диспетчера устройств StorSimple. Эти задачи организованы по колонкам, на которых они инициируются.

Чтобы получить дополнительные сведения о каждом рабочем процессе, щелкните соответствующую процедуру в таблице.

[!INCLUDE [StorSimple software warning](../../includes/storsimple-update-software-warning.md)]

#### <a name="storsimple-device-manager-workflows"></a>Рабочие процессы диспетчера устройств StorSimple
| Если вы хотите... | Статья с описанием процедуры |
| --- | --- |
| Создание службы</br>удаление службы;</br>Получение регистрационного ключа службы</br>повторное создание ключа регистрации службы. |[Развертывание службы диспетчера устройств StorSimple](storsimple-virtual-array-manage-service.md) |
| Просмотр журналов действий |[Использование колонки сводки службы StorSimple](storsimple-virtual-array-service-summary.md) |
| Отключение виртуального массива</br>Удаление виртуального массива |[Отключение или удаление виртуального массива](storsimple-virtual-array-deactivate-and-delete-device.md) |
| Аварийное восстановление и отработка отказа устройства</br>Необходимые условия для отработки отказа</br>Аварийное восстановление непрерывности бизнеса (BCDR)</br>Ошибки во время аварийного восстановления |[Аварийное восстановление и отработка отказа устройства для виртуального массива StorSimple](storsimple-virtual-array-failover-dr.md) |
| Архивация общих папок и томов</br>Создание резервной копии вручную</br>Изменение расписания резервного копирования</br>Просмотр существующих резервных копий |[Резервное копирование виртуального массива StorSimple (предварительная версия)](storsimple-virtual-array-backup.md) |
| Клонирование общих ресурсов из резервного набора данных</br>Клонирование томов из резервного набора данных</br>Восстановление на уровне отдельных объектов (только для файлового сервера) |[Клонирование из резервной копии виртуального массива StorSimple](storsimple-virtual-array-clone.md) |
| Об учетных записях хранения</br>добавление учетной записи хранения;</br>изменение учетной записи хранения;</br>Удаление учетной записи хранения |[Использование службы диспетчера StorSimple для управления учетной записью хранения (предварительная версия)](storsimple-virtual-array-manage-storage-accounts.md) |
| Получение сведений о записях контроля доступа</br>Добавление или изменение записи контроля доступа </br>Удаление записи контроля доступа |[Управление записями контроля доступа для виртуального массива StorSimple (предварительная версия) с помощью службы диспетчера StorSimple](storsimple-virtual-array-manage-acrs.md) |
| Просмотр сведений о задании |[Управление заданиями виртуального массива StorSimple](storsimple-virtual-array-manage-jobs.md) |
| Настроить параметры оповещений.</br>Получать уведомления об оповещениях</br>Управление оповещениями</br>Просмотр оповещений |[Просмотр оповещений для виртуального массива StorSimple (предварительная версия) и управление ими с помощью службы диспетчера StorSimple](storsimple-virtual-array-manage-alerts.md) |
| Изменение пароля администратора устройства |[Изменение пароля администратора для устройства виртуального массива StorSimple (предварительная версия)](storsimple-virtual-array-change-device-admin-password.md) |
| Установка обновлений программного обеспечения |[Обновление виртуального массива](storsimple-virtual-array-install-update.md) |

> [!NOTE]
> Следующие задачи выполняются только в [локальном веб-интерфейсе](storsimple-ova-web-ui-admin.md) :
> 
> * [получение ключа шифрования данных службы;](storsimple-ova-web-ui-admin.md#get-the-service-data-encryption-key)
> * [создание пакета журналов;](storsimple-ova-web-ui-admin.md#generate-a-log-package)
> * [остановка и повторный запуск виртуального массива.](storsimple-ova-web-ui-admin.md#shut-down-and-restart-your-device)
> 
> 

## <a name="next-steps"></a>Дальнейшие действия
Сведения о веб-интерфейсе и способах его использования см. в статье [Использование веб-интерфейса для администрирования виртуального массива StorSimple](storsimple-ova-web-ui-admin.md).