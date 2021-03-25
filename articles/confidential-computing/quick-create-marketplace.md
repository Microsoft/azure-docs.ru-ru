---
title: Краткое руководство по созданию виртуальной машины конфиденциальных вычислений Azure с помощью Marketplace
description: Сведения, которые позволяют начать развертывание с быстрого создания виртуальной машины конфиденциальных вычислений с помощью Marketplace.
author: JBCook
ms.service: virtual-machines
ms.subservice: confidential-computing
ms.workload: infrastructure
ms.topic: quickstart
ms.date: 04/06/2020
ms.author: JenCook
ms.openlocfilehash: aba23b67574fb74b7cd571dc5d4642bb8b991b93
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102566640"
---
# <a name="quickstart-deploy-an-azure-confidential-computing-vm-in-the-marketplace"></a>Краткое руководство. Развертывание виртуальной машины конфиденциальных вычислений в Marketplace

Начните работу с конфиденциальными вычислениями Azure, применив Azure Marketplace для создания виртуальной машины на базе Intel SGX. После этого вы установите пакет средств разработки Open Enclave, чтобы настроить среду разработки. 

Этот учебник рекомендуется использовать, если вы хотите быстро приступить к развертыванию виртуальной машины конфиденциальных вычислений. Виртуальные машины выполняются на специальном оборудовании и для них требуются определенные входные данные конфигурации. Предложение Marketplace, описанное в этом кратком руководстве, упрощает развертывание, позволяя ограничивать ввод данных пользователем.

Если вы заинтересованы в развертывании виртуальных машин конфиденциальных вычислений с дополнительной настраиваемой конфигурацией, следуйте [инструкциям по развертыванию виртуальной машины конфиденциальных вычислений на портале Azure](quick-create-portal.md).

## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет подписки Azure, [создайте учетную запись Azure](https://azure.microsoft.com/pricing/purchase-options/pay-as-you-go/), прежде чем начинать работу.

> [!NOTE]
> Учетные записи бесплатной пробной версии не предоставляют доступа к виртуальным машинам, которые используются в этом руководстве. Перейдите на подписку с оплатой по мере использования.

## <a name="sign-in-to-azure"></a>Вход в Azure

1. Войдите на [портал Azure](https://portal.azure.com/).

1. В верхней части панели поиска введите **Конфиденциальные вычисления Azure**.

1. Выберите **Azure confidential computing (Virtual Machine)** (Конфиденциальные вычисления Azure (виртуальная машина)) в разделе **Marketplace**. 

    ![Выбор плитки Marketplace](media/quick-create-marketplace/portal-search-marketplace.png)    

1. На целевой странице развертывания конфиденциальных вычислений Azure выберите **Создать**.
 

## <a name="configure-your-virtual-machine"></a>Настройка виртуальной машины

1. На вкладке **Основы** выберите **Подписка** и **Группа ресурсов**. Чтобы развернуть виртуальную машину из этого шаблона, группа ресурсов должна быть пустой.

1. Введите или выберите следующие значения:

   * **Регион**. Выберите подходящий регион Azure.

        > [!NOTE]
        > Виртуальные машины конфиденциальных вычислений работают только на специализированном оборудовании, которое доступно не во всех регионах. Актуальный список регионов с поддержкой виртуальных машин серии DCsv2 см. [здесь](https://azure.microsoft.com/global-infrastructure/services/?products=virtual-machines).
    
    * **Выбор образа.** Выберите любой образ. Для работы с этим учебником выберите Ubuntu 18.04 (2-го поколения). В противном случае вас перенаправят на соответствующие шаги ниже. 

    * В поле **Имя виртуальной машины** введите имя новой виртуальной машины.

    * **Тип проверки подлинности.** Выберите элемент **Открытый ключ SSH**, если вы создаете виртуальную машину Linux. 

         > [!NOTE]
         > Для проверки подлинности вы можете использовать открытый ключ или пароль SSH. SSH является более безопасным. Сведения о создании ключа SSH см. в статье [Как создать и использовать пару из открытого и закрытого ключей SSH для виртуальных машин Linux в Azure](../virtual-machines/linux/mac-create-ssh-keys.md).

    * **Имя пользователя**. Введите имя администратора для виртуальной машины.

    * **Открытый ключ SSH**. Введите открытый ключ RSA, если это применимо.
    
    * **Пароль**. Введите пароль, используемый для проверки подлинности, если это применимо.
 
1. Нажмите кнопку **Далее: Параметры виртуальной машины** в нижней части экрана.

    > [!IMPORTANT]
    > Подождите, пока страница обновится. Вы *не должны* видеть сообщение: Confidential Computing DCsv2-series VMs are available in a limited number of regions (Конфиденциальные вычисления виртуальных машин DCsv2-Series доступны в ограниченном числе регионов). Если это сообщение не исчезнет, вернитесь на предыдущую страницу и выберите доступный регион для DCsv2-Series.

1. В разделе **Изменить размер** выберите виртуальную машину с возможностями конфиденциальных вычислений в средстве для выбора размера. 

    > [!TIP]
    > Здесь вы увидите размеры **DC1s_v2**, **DC2s_v2**, **DC4s_V2** и **DC8_v2**. Пока только эти размеры виртуальных машин поддерживают конфиденциальные вычисления. [Подробнее](virtual-machine-solutions.md).

1. В разделе **Тип диска ОС** выберите тип диска.

1. В разделе **Виртуальная сеть** создайте виртуальную сеть или выберите имеющуюся.

1. В разделе **Подсеть** создайте подсеть или выберите имеющуюся.

1. В разделе **Выберите общедоступные входящие порты** выберите **SSH(Linux)/RDP(Windows)** . В нашем руководстве этот шаг нужен для подключения к виртуальной машине и завершения настройки пакета SDK Open Enclave. 

1. Для работы с этим кратким руководством выключите **диагностику загрузки**. 

1. Выберите **Review + create** (Просмотреть и создать).

1. В области **Отзыв и создание** выберите **Создать**.

> [!NOTE]
> Если вы развернули виртуальную машину Linux, переходите к следующему разделу этого руководства. Если вы развернули виртуальную машину Windows, [выполните эту процедуру для подключения к виртуальной машине Windows](../virtual-machines/windows/connect-logon.md) и установите [пакет OE SDK для Windows](https://github.com/openenclave/openenclave/blob/master/docs/GettingStartedDocs/install_oe_sdk-Windows.md).


## <a name="connect-to-the-linux-vm"></a>Подключение к виртуальной машине Linux

Если вы уже используете оболочку BASH, подключитесь к виртуальной машине Azure с помощью команды **SSH**. В следующей команде замените имя пользователя и IP-адрес виртуальной машины, чтобы подключиться к виртуальной машине Linux.

```bash
ssh azureadmin@40.55.55.555
```

Общедоступный IP-адрес виртуальной машины можно получить на портале Azure в разделе "Обзор" для виртуальной машины.

![IP-адрес на портале Azure](media/quick-create-portal/public-ip-virtual-machine.png)

Если вы используете Windows и у вас нет оболочки BASH, установите клиент SSH, например PuTTY.

1. [Скачайте и установите PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/download.html).

1. Запустите PuTTY.

1. На экране настройки PuTTY введите общедоступный IP-адрес виртуальной машины.

1. Выберите **Открыть** и введите имя пользователя и пароль при появлении запросов.

Дополнительные сведения о подключении к виртуальным машинам Linux см. в статье [Создание виртуальной машины Linux с помощью портала Azure](../virtual-machines/linux/quick-create-portal.md).

> [!NOTE]
> Если вы видите оповещение системы безопасности PuTTY о том, что ключ узла сервера не кэшируется в реестре, выберите один из следующих параметров. Если вы доверяете этому узлу, выберите **Да**, чтобы добавить ключ в кэш PuTTy и продолжить подключение. Если вы хотите продолжить подключение один раз, не добавляя ключ в кэш, выберите **Нет**. Если вы не доверяете этому узлу, выберите **Отмена**, чтобы прервать подключение.

## <a name="install-the-open-enclave-sdk-oe-sdk"></a>Установка пакета SDK Open Enclave (OE SDK) <a id="Install"></a>

Выполните пошаговые инструкции по установке [OE SDK](https://github.com/openenclave/openenclave) на виртуальной машине серии DCsv2, развернутой из образа Ubuntu 18.04 LTS 2-го поколения. 

Если для виртуальной машины вы выбрали образ Ubuntu 16.04 LTS 2-го поколения, воспользуйтесь [инструкциями по установке Ubuntu 16.04](https://github.com/openenclave/openenclave/blob/master/docs/GettingStartedDocs/install_oe_sdk-Ubuntu_16.04.md). 

#### <a name="1-configure-the-intel-and-microsoft-apt-repositories"></a>1. Настройте репозитории Intel и Microsoft APT.

```bash
echo 'deb [arch=amd64] https://download.01.org/intel-sgx/sgx_repo/ubuntu bionic main' | sudo tee /etc/apt/sources.list.d/intel-sgx.list
wget -qO - https://download.01.org/intel-sgx/sgx_repo/ubuntu/intel-sgx-deb.key | sudo apt-key add -

echo "deb http://apt.llvm.org/bionic/ llvm-toolchain-bionic-7 main" | sudo tee /etc/apt/sources.list.d/llvm-toolchain-bionic-7.list
wget -qO - https://apt.llvm.org/llvm-snapshot.gpg.key | sudo apt-key add -

echo "deb [arch=amd64] https://packages.microsoft.com/ubuntu/18.04/prod bionic main" | sudo tee /etc/apt/sources.list.d/msprod.list
wget -qO - https://packages.microsoft.com/keys/microsoft.asc | sudo apt-key add -
```

#### <a name="2-install-the-intel-sgx-dcap-driver"></a>2. Установите драйвер Intel SGX DCAP.

```bash
sudo apt update
sudo apt -y install dkms
wget https://download.01.org/intel-sgx/sgx-dcap/1.4/linux/distro/ubuntuServer18.04/sgx_linux_x64_driver_1.21.bin -O sgx_linux_x64_driver.bin
chmod +x sgx_linux_x64_driver.bin
sudo ./sgx_linux_x64_driver.bin
```

> [!WARNING]
> Установите последнюю версию драйвера Intel SGX DCAP с [сайта Intel SGX](https://01.org/intel-software-guard-extensions/downloads).

#### <a name="3-install-the-intel-and-open-enclave-packages-and-dependencies"></a>3. Установите пакеты Intel и Open Enclave и зависимости для них.

```bash
sudo apt -y install clang-7 libssl-dev gdb libsgx-enclave-common libsgx-enclave-common-dev libprotobuf10 libsgx-dcap-ql libsgx-dcap-ql-dev az-dcap-client open-enclave
```

> [!NOTE] 
> Среди прочего, на этом шаге устанавливается пакет [az-dcap-client](https://github.com/microsoft/azure-dcap-client), который необходим для удаленной аттестации в Azure.

#### <a name="4-verify-the-open-enclave-sdk-install"></a>4. **Проверьте установку пакета SDK Open Enclave (OE SDK)** .

Изучите инструкции по [использованию пакета SDK Open Enclave](https://github.com/openenclave/openenclave/blob/master/docs/GettingStartedDocs/Linux_using_oe_sdk.md) на GitHub, где описаны проверка и применение установленного пакета.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если группа ресурсов, виртуальная машина и все связанные с ними ресурсы вам больше не требуются, их можно удалить. 

Выберите группу ресурсов для виртуальной машины и щелкните **Удалить**. Подтвердите имя группы ресурсов, чтобы завершить удаление ресурсов.

## <a name="next-steps"></a>Дальнейшие действия

В рамках этого краткого руководства вы развернули виртуальную машину конфиденциальных вычислений и пакет SDK Open Enclave. Дополнительные сведения о виртуальных машинах конфиденциальных вычислений в Azure см. в [этой статье](virtual-machine-solutions.md). 

Изучите возможности по созданию приложений конфиденциальных вычислений на примерах для пакета SDK Open Enclave на сайте GitHub. 

> [!div class="nextstepaction"]
> [Примеры создания приложений с помощью пакета SDK Open Enclave](https://github.com/openenclave/openenclave/blob/master/samples/README.md)
