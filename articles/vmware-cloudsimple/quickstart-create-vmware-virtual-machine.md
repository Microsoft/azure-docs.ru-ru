---
title: Краткое руководство. Использование виртуальных машин VMware в Azure
titleSuffix: Azure VMware Solution by CloudSimple
description: Узнайте, как настроить и использовать виртуальные машины VMware из портал Azure с помощью решения VMware для Azure, Клаудсимпле
author: sharaths-cs
ms.author: dikamath
ms.date: 08/14/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 4ab613c251bc43a025e0381046805ec998a04227
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "77019559"
---
# <a name="quickstart---consume-vmware-vms-on-azure"></a>Краткое руководство. Использование виртуальных машин VMware в Azure

Чтобы создать виртуальную машину на портал Azure, используйте шаблоны виртуальных машин, которые администратор Клаудсимпле включил для вашей подписки. Шаблоны виртуальных машин находятся в инфраструктуре VMware.

## <a name="cloudsimple-vm-creation-on-azure-requires-a-vm-template"></a>Для создания виртуальной машины Клаудсимпле в Azure требуется шаблон виртуальной машины

Создайте виртуальную машину в частном облаке из пользовательского интерфейса vCenter. Чтобы создать шаблон, следуйте инструкциям в статье [клонирование виртуальной машины на шаблон в веб-клиенте vSphere](https://docs.vmware.com/en/VMware-vSphere/6.7/com.vmware.vsphere.vm_admin.doc/GUID-FE6DE4DF-FAD0-4BB0-A1FD-AFE9A40F4BFE.html). Сохраните шаблон виртуальной машины в частном облаке vCenter.

## <a name="create-a-virtual-machine-in-the-azure-portal"></a>Создание виртуальной машины на портале Azure

1. Выбор пункта **Все службы**.

2. Выполните поиск **CloudSimple Virtual Machines**.

3. Нажмите кнопку **Добавить**.

    ![Создание виртуальной машины Клаудсимпле](media/create-cloudsimple-virtual-machine.png)

4. Введите Основные сведения и нажмите кнопку **Далее: размер**.

    ![Создание виртуальной машины Клаудсимпле — основы](media/create-cloudsimple-virtual-machine-basic-info.png)

    | Поле | Описание |
    | ------------ | ------------- |
    | Подписка | Подписка Azure, связанная с частным облаком.  |
    | Группа ресурсов | Группа ресурсов, которой будет назначена виртуальная машина. Можно выбрать имеющуюся группу или создать новую. |
    | Имя | Имя, идентифицирующее виртуальную машину.  |
    | Расположение | Регион Azure, в котором размещена эта виртуальная машина.  |
    | Частное облако | Клаудсимпле частное облако, в котором вы хотите создать виртуальную машину. |
    | Пул ресурсов | Сопоставленный пул ресурсов для виртуальной машины. Выберите из доступных пулов ресурсов. |
    | vSphere Template (Шаблон vSphere) | шаблон vSphere для виртуальной машины.  |
    | Имя пользователя | Имя пользователя администратора виртуальной машины (для шаблонов Windows).|
    | Пароль |  Пароль для администратора виртуальной машины (для шаблонов Windows). |
    | Подтверждение пароля | Подтвердите пароль. |

5. Выберите количество ядер и объем памяти для виртуальной машины и нажмите кнопку **Далее: конфигурации**. Установите флажок, если требуется предоставить полную виртуализацию ЦП операционной системе на виртуальной машине. Приложения, требующие аппаратной виртуализации, смогут работать на виртуальных машинах без двоичной трансляции или паравиртуализации. Дополнительные сведения см. в статье VMware <a href="https://docs.vmware.com/en/VMware-vSphere/6.5/com.vmware.vsphere.vm_admin.doc/GUID-2A98801C-68E8-47AF-99ED-00C63E4857F6.html" target="_blank">Expose VMware Hardware Assisted Virtualization</a> (Предоставление аппаратной виртуализации VMware).

    ![Создание виртуальной машины Клаудсимпле. размер](media/create-cloudsimple-virtual-machine-size.png)

6. Настройте сетевые интерфейсы и диски, как описано в следующих таблицах, и щелкните **проверить и создать**.

    ![Создание виртуальной машины Клаудсимпле. конфигурации](media/create-cloudsimple-virtual-machine-configurations.png)

    Для параметра сетевые интерфейсы щелкните **Добавить сетевой интерфейс** и настройте следующие параметры.

    | Control | Описание |
    | ------------ | ------------- |
    | Имя | Введите имя для идентификации интерфейса.  |
    | Сеть | Выберите из списка настроенной распределенной группы портов в vSphere частного облака.  |
    | Адаптер | Выберите адаптер vSphere из списка доступных типов, настроенных для виртуальной машины. Дополнительные сведения см. в статье базы знаний VMware <a href="https://kb.vmware.com/s/article/1001805" target="_blank">Выбор сетевого адаптера для виртуальной машины</a>. |
    | Power on at Boot (Включать при загрузке) | Выберите, следует ли включить оборудование сетевого адаптера при загрузке виртуальной машины. По умолчанию этот параметр имеет значение **Enable** (Включить). |

    Для параметра диски щелкните **Добавить диск** и настройте следующие параметры.

    | Элемент | Описание |
    | ------------ | ------------- |
    | Имя | Введите имя для идентификации диска.  |
    | Размер | Выберите один из доступных размеров.  |
    | SCSI Controller (Контроллер SCSI) | Выберите контроллер SCSI для диска.  |
    | Режим | Определяет, как диск участвует в моментальных снимках. Выберите один из следующих параметров. <br> -Независимый постоянный: все данные, записываемые на диск, записываются без возможности восстановления.<br> — Независимое от постоянного: изменения, записанные на диск, отбрасываются при выключении или сбросе виртуальной машины.  Режим независимого временного хранения позволяет восстановить состояние виртуальной машины, просто перезапустив ее. Дополнительные сведения см. в <a href="https://docs.vmware.com/en/VMware-vSphere/6.5/com.vmware.vsphere.vm_admin.doc/GUID-8B6174E6-36A8-42DA-ACF7-0DA4D8C5B084.html" target="_blank">документации VMware</a>.

7. После завершения проверки проверьте параметры и нажмите кнопку **создать**. Чтобы внести изменения, щелкните вкладки вверху или щелкните.

    ![Создание виртуальной машины Клаудсимпле — обзор](media/create-cloudsimple-virtual-machine-review.png)

## <a name="next-steps"></a>Дальнейшие действия

* [Просмотр списка виртуальных машин CloudSimple](azure-create-vm.md#view-list-of-cloudsimple-virtual-machines)
* [Управление виртуальной машиной Клаудсимпле из Azure](azure-manage-vm.md)
