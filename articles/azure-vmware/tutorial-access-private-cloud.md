---
title: Учебник. Получение доступа к частному облаку
description: Узнайте, как получить доступ к частному облаку Решения Azure VMware
ms.topic: tutorial
ms.date: 03/13/2021
ms.openlocfilehash: f2af1cffda08bf4b9c62e63f32d36cc9bbd7024a
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103494399"
---
# <a name="tutorial-access-an-azure-vmware-solution-private-cloud"></a>Руководство по получению доступа к частному облаку Решения Azure VMware

Решение Azure VMware не поддерживает управление частным облаком с помощью локального экземпляра vCenter. Вам потребуется подключиться к экземпляру vCenter для Решения Azure VMware через инсталяционный сервер. 

В этом учебнике описано, как создать инсталляционный сервер в группе ресурсов, созданной с помощью инструкций из [предыдущего учебника](tutorial-configure-networking.md), и войти в vCenter для Решения Azure VMware. Инсталляционный сервер — это виртуальная машина Windows в той же виртуальной сети, которую вы создали.  Он предоставляет доступ к vCenter и NSX Manager. 

В этом руководстве описано следующее:

> [!div class="checklist"]
> * создание виртуальной машины Windows для доступа к vCenter для Решения Azure VMware;
> * вход в vCenter из этой виртуальной машины.

## <a name="create-a-new-windows-virtual-machine"></a>Создание виртуальной машины Windows

[!INCLUDE [create-avs-jump-box-steps](includes/create-jump-box-steps.md)]

## <a name="connect-to-the-local-vcenter-of-your-private-cloud"></a>Подключение к локальному экземпляру vCenter частного облака

1. На инсталляционном сервере войдите в клиент vSphere с помощью единого входа VMware vCenter. Используйте для этого имя пользователя облачного администратора и убедитесь, что пользовательский интерфейс отображается корректно.

1. На портале Azure выберите частное облако и щелкните **Управление** > **Удостоверение**. 

   Отобразятся URL-адреса и учетные данные пользователя в vCenter и NSX-T Manager частного облака.

   :::image type="content" source="media/tutorial-access-private-cloud/ss4-display-identity.png" alt-text="Отобразите URL-адреса и учетные данные частного облака vCenter и NSX Manager." border="true" lightbox="media/tutorial-access-private-cloud/ss4-display-identity.png":::

1. Перейдите к виртуальной машине, созданной на предыдущем этапе, и подключитесь к ней. 

   Если вам нужна помощь с подключением к виртуальной машине, ознакомьтесь со сведениями в [соответствующем разделе](../virtual-machines/windows/connect-logon.md#connect-to-the-virtual-machine).

1. На виртуальной машине Windows откройте браузер и перейдите на URL-адреса vCenter и NSX-T Manger на двух вкладках. 

1. На вкладке vCenter введите учетные данные пользователя `cloudadmin@vmcp.local`, полученные на предыдущем этапе.

   :::image type="content" source="media/tutorial-access-private-cloud/ss5-vcenter-login.png" alt-text="Войдите в экземпляр vCenter частного облака." border="true":::

   :::image type="content" source="media/tutorial-access-private-cloud/ss6-vsphere-client-home.png" alt-text="Портал vCenter." border="true":::

1. На второй вкладке браузера войдите в NSX-T Manager.

   :::image type="content" source="media/tutorial-access-private-cloud/ss10-nsx-manager-home.png" alt-text="На второй вкладке браузера войдите в NSX-T Manager." border="true":::



## <a name="next-steps"></a>Дальнейшие действия

Из этого руководства вы узнали, как выполнять такие задачи:

> [!div class="checklist"]
> * Создание виртуальной машины Windows, которая будет использоваться для подключения к vCenter.
> * Вход в vCenter с виртуальной машины.

Перейдите к следующему учебнику, где объясняется, как создать виртуальную сеть, чтобы настроить локальное управление для кластеров частного облака.

> [!div class="nextstepaction"]
> [Создание виртуальной сети](tutorial-configure-networking.md)


