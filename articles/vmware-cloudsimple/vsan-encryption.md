---
title: Решение VMware для Azure с помощью Клаудсимпле. Настройка шифрования vSAN для частного облака
description: Описание настройки функции шифрования программного обеспечения vSAN, чтобы ваше частное облако Клаудсимпле можно было работать с сервером управления ключами, работающим в виртуальной сети Azure.
author: Ajayan1008
ms.author: v-hborys
ms.date: 08/19/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: f5da05c7f3c6878b0804799360e512676b9002d3
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97899054"
---
# <a name="configure-vsan-encryption-for-cloudsimple-private-cloud"></a>Настройка шифрования vSAN для частного облака Клаудсимпле

Вы можете настроить функцию шифрования программного обеспечения vSAN, чтобы ваше частное облако Клаудсимпле можно было работать с сервером управления ключами, работающим в виртуальной сети Azure.

При использовании шифрования vSAN VMware требует использования внешнего средства сервера управления ключами (KMS), совместимого с КМИП 1,1. Вы можете использовать любой поддерживаемый сервер управления ключами, сертифицированный VMware, и доступный для Azure.

В этом руководство описано, как использовать Хитруст Кэйконтрол KMS, работающий в виртуальной сети Azure. Аналогичный подход можно использовать для любого другого сертифицированного стороннего решения KMS для vSAN.

Это решение KMS требует следующих действий:

* Установите, настройте и управляйте сертифицированным сторонним средством KMS в виртуальной сети Azure.
* Предоставьте собственные лицензии для средства KMS.
* Настройка шифрования vSAN в частном облаке и управление им с помощью стороннего средства KMS, работающего в виртуальной сети Azure.

## <a name="kms-deployment-scenario"></a>Сценарий развертывания службы KMS

Кластер серверов службы KMS работает в виртуальной сети Azure и доступен из частного облака vCenter через настроенное подключение Azure ExpressRoute.

![.. кластер/медиа/КМС в виртуальной сети Azure](media/vsan-kms-cluster.png)

## <a name="how-to-deploy-the-solution"></a>Развертывание решения

Процесс развертывания включает следующие шаги.

1. [Проверка соблюдения предварительных требований](#verify-prerequisites-are-met)
2. [Портал Клаудсимпле: получение сведений об пиринга ExpressRoute](#cloudsimple-portal-obtain-expressroute-peering-information)
3. [Портал Azure: подключение виртуальной сети к частному облаку](#azure-portal-connect-your-virtual-network-to-your-private-cloud)
4. [Портал Azure. Развертывание кластера Хитруст Кэйконтрол в виртуальной сети](#azure-portal-deploy-a-hytrust-keycontrol-cluster-in-the-azure-resource-manager-in-your-virtual-network)
5. [Хитруст WebUI: Настройка сервера КМИП](#hytrust-webui-configure-the-kmip-server)
6. [Пользовательский интерфейс vCenter: Настройка шифрования vSAN для использования кластера KMS в виртуальной сети Azure](#vcenter-ui-configure-vsan-encryption-to-use-kms-cluster-in-your-azure-virtual-network)

### <a name="verify-prerequisites-are-met"></a>Проверка соблюдения предварительных требований

Перед развертыванием проверьте следующее:

* Выбранный поставщик KMS, инструмент и версия находятся в списке совместимости vSAN.
* Выбранный поставщик поддерживает версию инструмента для запуска в Azure.
* Версия Azure средства KMS совместима с КМИП 1,1.
* Azure Resource Manager и виртуальная сеть уже созданы.
* Клаудсимпле частное облако уже создано.

### <a name="cloudsimple-portal-obtain-expressroute-peering-information"></a>Портал Клаудсимпле: получение сведений об пиринга ExpressRoute

Чтобы продолжить установку, требуется ключ авторизации и URI однорангового канала для ExpressRoute и доступ к подписке Azure. Эти сведения доступны на странице подключения к виртуальной сети на портале Клаудсимпле. Инструкции см. в статье [Настройка подключения виртуальной сети к частному облаку](virtual-network-connection.md). Если у вас возникли проблемы с получением информации, отправьте [запрос в службу поддержки](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

### <a name="azure-portal-connect-your-virtual-network-to-your-private-cloud"></a>Портал Azure: подключение виртуальной сети к частному облаку

1. Создайте шлюз виртуальной сети для виртуальной сети, следуя инструкциям в разделе [Настройка шлюза виртуальной сети для ExpressRoute с помощью портал Azure](../expressroute/expressroute-howto-add-gateway-portal-resource-manager.md).
2. Свяжите виртуальную сеть с каналом ExpressRoute Клаудсимпле, следуя инструкциям в статье [подключение виртуальной сети к каналу expressroute с помощью портала](../expressroute/expressroute-howto-linkvnet-portal-resource-manager.md).
3. Используйте сведения о канале ExpressRoute Клаудсимпле, полученные в приветственном сообщении электронной почты от Клаудсимпле, чтобы связать виртуальную сеть с каналом Клаудсимпле ExpressRoute в Azure.
4. Введите ключ авторизации и URI одноранговой цепи, присвойте имя соединению и нажмите кнопку **ОК**.

![Укажите URI одноранговой цепи CS при создании виртуальной сети](media/vsan-azureportal01.png) 

### <a name="azure-portal-deploy-a-hytrust-keycontrol-cluster-in-the-azure-resource-manager-in-your-virtual-network"></a>Портал Azure. Развертывание кластера Хитруст Кэйконтрол в Azure Resource Manager в виртуальной сети

Чтобы развернуть кластер Хитруст Кэйконтрол в Azure Resource Manager в виртуальной сети, выполните следующие задачи. Дополнительные сведения см. в [документации по хитруст](https://docs.hytrust.com/DataControl/Admin_Guide-4.0/Default.htm#OLH-Files/Azure.htm%3FTocPath%3DHyTrust%2520DataControl%2520and%2520Microsoft%2520Azure%7C_____0) .

1. Создайте группу безопасности сети Azure (NSG-хитруст) с указанными правилами для входящих подключений, следуя инструкциям в документации по Хитруст.
2. Создание пары ключей SSH в Azure.
3. Разверните исходный узел Кэйконтрол из образа в Azure Marketplace.  Используйте открытый ключ созданной пары ключей и выберите **NSG-хитруст** в качестве группы безопасности сети для узла кэйконтрол.
4. Преобразуйте частный IP-адрес Кэйконтрол в статический IP-адрес.
5. Подключитесь к виртуальной машине Кэйконтрол по протоколу SSH, используя общедоступный IP-адрес и закрытый ключ ранее упомянутой пары ключей.
6. При появлении запроса в командной консоли SSH выберите, `No` чтобы задать узел в качестве начального узла кэйконтрол.
7. Добавьте дополнительные узлы Кэйконтрол, повторив шаги 3-5 этой процедуры и выбрав `Yes` вопрос о добавлении в существующий кластер.

### <a name="hytrust-webui-configure-the-kmip-server"></a>Хитруст WebUI: Настройка сервера КМИП

Перейдите в https://*public-IP*, где *public-IP* — общедоступный IP-адрес виртуальной машины узла кэйконтрол. Выполните следующие действия из [документации по хитруст](https://docs.hytrust.com/DataControl/Admin_Guide-4.0/Default.htm#OLH-Files/Azure.htm%3FTocPath%3DHyTrust%2520DataControl%2520and%2520Microsoft%2520Azure%7C_____0).

1. [Настройка сервера КМИП](https://docs.hytrust.com/DataControl/4.2/Admin_Guide-4.2/index.htm#Books/VMware-vSphere-VSAN-Encryption/configuring-kmip-server.htm%3FTocPath%3DHyTrust%2520KeyControl%2520with%2520VSAN%25C2%25A0and%2520VMware%2520vSphere%2520VM%2520Encryption%7C_____2)
2. [Создание пакета сертификатов для шифрования VMware](https://docs.hytrust.com/DataControl/4.2/Admin_Guide-4.2/index.htm#Books/VMware-vSphere-VSAN-Encryption/creating-user-for-vmcrypt.htm%3FTocPath%3DHyTrust%2520KeyControl%2520with%2520VSAN%25C2%25A0and%2520VMware%2520vSphere%2520VM%2520Encryption%7C_____3)

### <a name="vcenter-ui-configure-vsan-encryption-to-use-kms-cluster-in-your-azure-virtual-network"></a>Пользовательский интерфейс vCenter: Настройка шифрования vSAN для использования кластера KMS в виртуальной сети Azure

Следуйте инструкциям по Хитруст, чтобы [создать кластер службы KMS в vCenter](https://docs.hytrust.com/DataControl/4.2/Admin_Guide-4.2/index.htm#Books/VMware-vSphere-VSAN-Encryption/creating-KMS-Cluster.htm%3FTocPath%3DHyTrust%2520KeyControl%2520with%2520VSAN%25C2%25A0and%2520VMware%2520vSphere%2520VM%2520Encryption%7C_____4).

![Добавление сведений о кластере KMS в vCenter](media/vsan-config01.png)

В vCenter перейдите к **кластеру > настроить** и выберите **Общие** параметры для vSAN. Включите шифрование и выберите кластер KMS, который ранее был добавлен в vCenter.

![Включение шифрования vSAN и настройка кластера KMS в vCenter](media/vsan-config02.png)

## <a name="references"></a>Ссылки

### <a name="azure"></a>Azure

[Настройка шлюза виртуальной сети для ExpressRoute с помощью портала Azure](../expressroute/expressroute-howto-add-gateway-portal-resource-manager.md)

[Подключение виртуальной сети к каналу ExpressRoute с помощью портала](../expressroute/expressroute-howto-linkvnet-portal-resource-manager.md)

### <a name="hytrust"></a>хитруст

[Хитруст элемент управления и Microsoft Azure](https://docs.hytrust.com/DataControl/Admin_Guide-4.0/Default.htm#OLH-Files/Azure.htm%3FTocPath%3DHyTrust%2520DataControl%2520and%2520Microsoft%2520Azure%7C_____0)

[Настройка сервера КМПИ](https://docs.hytrust.com/DataControl/4.2/Admin_Guide-4.2/index.htm#Books/VMware-vSphere-VSAN-Encryption/configuring-kmip-server.htm%3FTocPath%3DHyTrust%2520KeyControl%2520with%2520VSAN%25C2%25A0and%2520VMware%2520vSphere%2520VM%2520Encryption%7C_____2)

[Создание пакета сертификатов для шифрования VMware](https://docs.hytrust.com/DataControl/4.2/Admin_Guide-4.2/index.htm#Books/VMware-vSphere-VSAN-Encryption/creating-user-for-vmcrypt.htm%3FTocPath%3DHyTrust%2520KeyControl%2520with%2520VSAN%25C2%25A0and%2520VMware%2520vSphere%2520VM%2520Encryption%7C_____3)

[Создание кластера KMS в vSphere](https://docs.hytrust.com/DataControl/4.2/Admin_Guide-4.2/index.htm#Books/VMware-vSphere-VSAN-Encryption/creating-KMS-Cluster.htm%3FTocPath%3DHyTrust%2520KeyControl%2520with%2520VSAN%25C2%25A0and%2520VMware%2520vSphere%2520VM%2520Encryption%7C_____4)
