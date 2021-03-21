---
title: Управление обслуживанием для виртуальных машин Azure с помощью портал Azure
description: Узнайте, как управлять применением обслуживания к виртуальным машинам Azure с помощью управления обслуживанием и портал Azure.
author: cynthn
ms.service: virtual-machines
ms.subservice: maintenance-control
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 04/22/2020
ms.author: cynthn
ms.openlocfilehash: 0dea30723a3ebe1598d6304ac3c98bfe1b55b466
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102563019"
---
# <a name="control-updates-with-maintenance-control-and-the-azure-portal"></a>Управление обновлениями с помощью управления обслуживанием и портал Azure

Управление обслуживанием позволяет решить, когда следует применять обновления к изолированным виртуальным машинам и выделенным узлам Azure. В этом разделе рассматриваются параметры портал Azure для управления обслуживанием. Дополнительные сведения о преимуществах использования управления обслуживанием, его ограничений и других параметров управления см. [в разделе Управление обновлениями платформы с помощью управления обслуживанием](maintenance-control.md).

## <a name="create-a-maintenance-configuration"></a>Создайте конфигурацию обслуживания.

1. Войдите на портал Azure.

1. Выполните поиск **конфигураций обслуживания**.

   ![Снимок экрана, показывающий, как открыть конфигурации обслуживания](media/virtual-machines-maintenance-control-portal/maintenance-configurations-search.png)

1. Нажмите кнопку **Добавить**.

   ![Снимок экрана, показывающий, как добавить конфигурацию обслуживания](media/virtual-machines-maintenance-control-portal/maintenance-configurations-add.png)

1. Выберите подписку и группу ресурсов, укажите имя конфигурации и выберите регион. Щелкните **Далее**.

   ![Снимок экрана, показывающий основы настройки обслуживания](media/virtual-machines-maintenance-control-portal/maintenance-configurations-basics.png)

1. Добавьте теги и значения. Щелкните **Далее**.

   ![Снимок экрана, показывающий, как добавить теги в конфигурацию обслуживания](media/virtual-machines-maintenance-control-portal/maintenance-configurations-tags.png)

1. Проверьте сводные данные. Нажмите кнопку **Создать**.

   ![Снимок экрана, показывающий, как создать конфигурацию обслуживания](media/virtual-machines-maintenance-control-portal/maintenance-configurations-create.png)

1. После завершения развертывания щелкните **"переход к ресурсу"**.

   ![Снимок экрана, показывающий развертывание конфигурации обслуживания завершено](media/virtual-machines-maintenance-control-portal/maintenance-configurations-deployment-complete.png)

## <a name="assign-the-configuration"></a>Назначение конфигурации

На странице сведения в конфигурации обслуживания щелкните назначения, а затем выберите **назначить ресурс**. 

![Снимок экрана, показывающий назначение ресурса](media/virtual-machines-maintenance-control-portal/maintenance-configurations-add-assignment.png)

Выберите ресурсы, которым должна быть назначена Конфигурация обслуживания, и нажмите кнопку **ОК**. В столбце **тип** указывается, является ли ресурс изолированной виртуальной машиной или выделенным узлом Azure. Для назначения конфигурации виртуальная машина должна быть запущена. При попытке назначить конфигурацию виртуальной машине, которая остановлена, возникает ошибка. 

<!---Shantanu to add details about the error case--->

![Снимок экрана, показывающий, как выбрать ресурс](media/virtual-machines-maintenance-control-portal/maintenance-configurations-select-resource.png)

## <a name="check-configuration"></a>Проверить конфигурацию

Можно проверить, правильно ли применена конфигурация, или убедиться, что конфигурация обслуживания в настоящее время назначена с помощью **конфигураций обслуживания**. В столбце **тип** указывается, назначена ли конфигурация изолированной виртуальной машине или выделенному узлу Azure. 

![Снимок экрана, показывающий, как проверить конфигурацию обслуживания](media/virtual-machines-maintenance-control-portal/maintenance-configurations-host-type.png)

Можно также проверить конфигурацию конкретной виртуальной машины на странице ее свойств. Щелкните **обслуживание** , чтобы просмотреть конфигурацию, назначенную этой виртуальной машине.

![Снимок экрана, показывающий, как проверить обслуживание узла](media/virtual-machines-maintenance-control-portal/maintenance-configurations-check-config.png)

## <a name="check-for-pending-updates"></a>Проверить наличие ожидающих обновлений

Есть также два способа проверить, ожидают ли обновления для конфигурации обслуживания. В области **конфигурации обслуживания** на странице сведения о конфигурации щелкните **назначения** и проверьте **состояние обслуживания**.

![Снимок экрана, показывающий, как проверить ожидающие обновления](media/virtual-machines-maintenance-control-portal/maintenance-configurations-pending.png)

Вы также можете проверить конкретный узел, используя **виртуальные машины** или свойства выделенного узла. 

![Снимок экрана, на котором показано выбранное состояние обслуживания.](media/virtual-machines-maintenance-control-portal/maintenance-configurations-pending-vm.png)

## <a name="apply-updates"></a>Применение обновлений

Вы можете применить ожидающие обновления по запросу с помощью **виртуальных машин**. На странице сведения о виртуальной машине щелкните **обслуживание** и щелкните **Применить обслуживание**.

![Снимок экрана, показывающий, как применить ожидающие обновления](media/virtual-machines-maintenance-control-portal/maintenance-configurations-apply-updates-now.png)

## <a name="check-the-status-of-applying-updates"></a>Проверка состояния применения обновлений 

Вы можете проверить ход выполнения обновлений для конфигурации в **конфигурациях обслуживания** или с помощью **виртуальных машин**. На странице сведения о виртуальной машине щелкните **обслуживание**. В следующем примере **состояние обслуживания** показывает, что **ожидается** обновление.

![Снимок экрана, показывающий, как проверить состояние ожидающих обновлений](media/virtual-machines-maintenance-control-portal/maintenance-configurations-status.png)

## <a name="delete-a-maintenance-configuration"></a>Удаление конфигурации обслуживания

Чтобы удалить конфигурацию, откройте сведения о конфигурации и нажмите кнопку **Удалить**.

![Снимок экрана, показывающий, как удалить конфигурацию.](media/virtual-machines-maintenance-control-portal/maintenance-configurations-delete.png)


## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в разделе [обслуживание и обновления](maintenance-and-updates.md).
