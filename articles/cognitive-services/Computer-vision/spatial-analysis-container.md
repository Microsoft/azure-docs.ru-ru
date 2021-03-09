---
title: Как установить и запустить контейнер пространственного анализа — Компьютерное зрение
titleSuffix: Azure Cognitive Services
description: Контейнер пространственного анализа позволяет обнаруживать людей и расстояния.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: conceptual
ms.date: 01/12/2021
ms.author: aahi
ms.openlocfilehash: 9989c6ea6b75203d43c37854caef7fdcbc321779
ms.sourcegitcommit: 956dec4650e551bdede45d96507c95ecd7a01ec9
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102519033"
---
# <a name="install-and-run-the-spatial-analysis-container-preview"></a>Установка и запуск контейнера пространственного анализа (Предварительная версия)

Контейнер пространственного анализа позволяет анализировать потоковую передачу видео в реальном времени, чтобы понять пространственные отношения между людьми, их движением и взаимодействием с объектами в физических средах. Контейнеры соответствуют конкретным требованиям к безопасности и управлению данными.

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services).
* Создав подписку Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesComputerVision"  title=" Создайте ресурс компьютерное зрение "  target="_blank"> Создайте компьютерное зрение ресурс </a> для уровня "Стандартный" S1 в портал Azure, чтобы получить ключ и конечную точку. После развертывания щелкните **Перейти к ресурсам**.
    * Вам потребуется ключ и конечная точка из созданного ресурса, чтобы запустить контейнер пространственного анализа. Вы будете использовать ключ и конечную точку позже.


### <a name="spatial-analysis-container-requirements"></a>Требования к контейнеру пространственного анализа

Чтобы запустить контейнер пространственного анализа, требуется устройство вычислений с [графическим процессором NVIDIA Tesla T4](https://www.nvidia.com/en-us/data-center/tesla-t4/). Рекомендуется использовать [Azure Stack ребра](https://azure.microsoft.com/products/azure-stack/edge/) с ускорением GPU, но контейнер работает на любом другом компьютере, который соответствует минимальным требованиям. Мы будем называть это устройство главным компьютером.

#### <a name="azure-stack-edge-device"></a>[Устройство Azure Stack Edge](#tab/azure-stack-edge)

Azure Stack ребро — это решение "оборудование как услуга" и "пограничные вычислительные устройства с поддержкой искусственного интеллекта" с возможностями перемещения данных сети. Подробные инструкции по подготовке и настройке см. в [документации по Azure Stack](../../databox-online/azure-stack-edge-deploy-prep.md).

#### <a name="desktop-machine"></a>[Настольный компьютер](#tab/desktop-machine)

#### <a name="minimum-hardware-requirements"></a>Минимальные требования к оборудованию

* 4 ГБ системной оперативной памяти
* 4 ГБ ОЗУ GPU
* 8 основных ЦП
* 1 графический процессор NVIDIA Tesla T4
* 20 ГБ дискового пространства

#### <a name="recommended-hardware"></a>Рекомендуемое оборудование

* 32 ГБ системной памяти
* 16 ГБ ОЗУ GPU
* 8 основных ЦП
* 2 видеопроцессоры NVIDIA Tesla T4 GPU
* 50 ГБ пространства SSD

В этой статье будут скачаны и установлены следующие пакеты программного обеспечения. Главный компьютер должен иметь возможность выполнять следующие действия (см. инструкции ниже).

* [Графические драйверы NVIDIA](https://docs.nvidia.com/datacenter/tesla/tesla-installation-notes/index.html) и [набор средств NVIDIA CUDA Toolkit](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html)
* Конфигурации для [NVIDIA MPS](https://docs.nvidia.com/deploy/pdf/CUDA_Multi_Process_Service_Overview.pdf) (Многопроцессная служба).
* [DOCKER CE](https://docs.docker.com/install/linux/docker-ce/ubuntu/#install-docker-engine---community-1) и [NVIDIA-Docker2](https://github.com/NVIDIA/nvidia-docker) 
* Среда выполнения [Azure IOT Edge](../../iot-edge/how-to-install-iot-edge.md) .

#### <a name="azure-vm-with-gpu"></a>[Виртуальная машина Azure с графическим процессором](#tab/virtual-machine)
В нашем примере мы будем использовать [виртуальную машину серии NC](../../virtual-machines/nc-series.md?bc=%2fazure%2fvirtual-machines%2flinux%2fbreadcrumb%2ftoc.json&toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) с одним K80 GPU.

---

| Требование | Описание |
|--|--|
| Камера | Контейнер пространственного анализа не привязан к определенной торговой марке камеры. Устройство камеры должно: Real-Time поддерживать кодирование протокола 264 (RTSP) и H., быть доступным для главного компьютера и поддерживать потоковую передачу при разрешении 15FPS и 1080p. |
| ОС Linux | На главном компьютере должен быть установлен [Ubuntu Desktop 18,04 LTS](http://releases.ubuntu.com/18.04/) .  |


## <a name="request-approval-to-run-the-container"></a>Запросить утверждение для запуска контейнера

Заполните [форму запроса](https://aka.ms/csgate) и отправьте ее, чтобы запросить утверждение для запуска контейнера.

В форме нужно указать сведения о себе, компании и пользовательском сценарии, для которого будет использоваться контейнер. После отправки формы команда Azure Cognitive Services будет просматривать ее и отправлять вам решение.

> [!IMPORTANT]
> * В форме необходимо использовать адрес электронной почты, связанный с ИДЕНТИФИКАТОРом подписки Azure.
> * Ресурс Компьютерное зрение, используемый для запуска контейнера, должен быть создан с утвержденным ИДЕНТИФИКАТОРом подписки Azure.

После утверждения вы сможете запустить контейнер после его загрузки из реестра контейнеров (Майкрософт) (мкр), как описано далее в этой статье.

Вы не сможете запустить контейнер, если подписка Azure не утверждена.

## <a name="set-up-the-host-computer"></a>Настройка главного компьютера

Рекомендуется использовать устройство Azure Stack пограничной для главного компьютера. Щелкните **Настольный компьютер** , если вы настраиваете другое устройство или **виртуальную машину** при использовании виртуальной машины.

#### <a name="azure-stack-edge-device"></a>[Устройство Azure Stack Edge](#tab/azure-stack-edge)

### <a name="configure-compute-on-the-azure-stack-edge-portal"></a>Настройка вычислений на портале Azure Stack пограничных устройств 
 
Пространственный анализ использует функции вычислений Azure Stack пограничных приложений для запуска решения искусственного интеллекта. Чтобы включить функции вычислений, убедитесь в том, что: 

* Вы [подключили и активировали](../../databox-online/azure-stack-edge-deploy-connect-setup-activate.md) устройство Azure Stack пограничным. 
* Для доступа к устройству используется клиентская система Windows с PowerShell 5,0 или более поздней версии.  
* Чтобы развернуть кластер Kubernetes, необходимо настроить устройство Azure Stack погранично через **локальный пользовательский интерфейс** на [портал Azure](https://portal.azure.com/): 
  1. Включите функцию вычислений на устройстве Azure Stackного периметра. Чтобы включить вычисление, перейдите на страницу " **Вычисление** " в веб-интерфейсе устройства. 
  2. Выберите сетевой интерфейс, который требуется включить для вычислений, а затем нажмите кнопку **включить**. На этом сетевом интерфейсе будет создан виртуальный коммутатор на устройстве.
  3. Оставьте IP-адреса узла тестирования Kubernetes и IP-адреса внешних служб Kubernetes пустыми.
  4. Нажмите кнопку **Применить**. Эта операция может занять около двух минут. 

![настройка вычислений;](media/spatial-analysis/configure-compute.png)

### <a name="set-up-an-edge-compute-role-and-create-an-iot-hub-resource"></a>Настройка роли вычислений ребра и создание ресурса центра Интернета вещей

В [портал Azure](https://portal.azure.com/)перейдите к ресурсу Azure Stackного периметра. На странице **Обзор** или в списке переходов нажмите кнопку **получить Начало работы** . На плитке  **Настройка вычислений ребра**   щелкните **настроить**. 

![Ссылка](media/spatial-analysis/configure-edge-compute-tile.png)

На странице **Настройка вычислений ребра**   выберите существующий центр Интернета вещей или создайте новый. По умолчанию для создания ресурса центра Интернета вещей используется ценовая категория "Стандартный" (S1). Чтобы использовать ресурс центра Интернета вещей уровня Free, создайте его и выберите. Ресурс центра Интернета вещей использует ту же подписку и группу ресурсов, которая используется ресурсом Azure Stack ребра. 

Нажмите кнопку **Создать**. Создание ресурса центра Интернета вещей может занять несколько минут. После создания ресурса центра Интернета вещей на плитке **Настройка COMPUTE ребра** будет отображаться новая конфигурация. Чтобы убедиться, что роль вычислений ребра настроена, выберите **Просмотреть конфигурацию** на плитке **Настройка вычислений**   .

При настройке роли вычислений Edge на устройстве Edge создается два устройства — устройство Интернета вещей и устройство IoT Edge. Оба устройства можно просмотреть в ресурсе Центра Интернета вещей. Среда выполнения Azure IoT Edge будет уже запущена на устройстве IoT Edge.

> [!NOTE]
> * В настоящее время для IoT Edge устройств поддерживается только платформа Linux. Дополнительные сведения об устранении неполадок Azure Stack пограничном устройстве см. в статье [ведение журнала и устранение неполадок](spatial-analysis-logging.md) .
> * Дополнительные сведения о настройке IoT Edge устройства для взаимодействия через прокси-сервер см. в разделе [Настройка устройства IOT Edge для взаимодействия через прокси-сервер](../../iot-edge/how-to-configure-proxy-support.md#azure-portal) .

###  <a name="enable-mps-on-azure-stack-edge"></a>Включение MPS на Azure Stack границе 

1. Запустите сеанс Windows PowerShell от имени администратора. 

2. Убедитесь, что на вашем клиенте запущена служба служба удаленного управления Windows. В терминале PowerShell используйте следующую команду: 
    
    ```powershell
    winrm quickconfig
    ```
    
    Если вы видите предупреждения об исключении брандмауэра, проверьте тип подключения к сети и просмотрите документацию по [Служба удаленного управления Windows](/windows/win32/winrm/installation-and-configuration-for-windows-remote-management) .

3. Назначьте переменную IP-адресу устройства. 
    
    ```powershell
    $ip = "<device-IP-address>" 
    ```
    
4. Чтобы добавить IP-адрес устройства в список доверенных узлов клиента, используйте следующую команду: 
    
    ```powershell
    Set-Item WSMan:\localhost\Client\TrustedHosts $ip -Concatenate -Force 
    ```

5. Запустите сеанс Windows PowerShell на устройстве. 

    ```powershell
    Enter-PSSession -ComputerName $ip -Credential $ip\EdgeUser -ConfigurationName Minishell 
    ```

6. При появлении запроса введите пароль. Используйте тот же пароль, который используется для входа в локальный веб-интерфейс. По умолчанию локальный веб-интерфейс имеет пароль `Password1` .

Введите `Start-HcsGpuMPS` , чтобы запустить службу MPS на устройстве. 

Дополнительные сведения об устранении неполадок Azure Stack пограничном устройстве см. [в разделе Устранение неполадок устройства Azure Stack пограничных устройств](spatial-analysis-logging.md#troubleshooting-the-azure-stack-edge-device) . 

#### <a name="desktop-machine"></a>[Настольный компьютер](#tab/desktop-machine)

Выполните эти инструкции, если главный компьютер не является Azure Stackным устройством.

#### <a name="install-nvidia-cuda-toolkit-and-nvidia-graphics-drivers-on-the-host-computer"></a>Установка комплекта NVIDIA CUDA Toolkit и графических драйверов NVIDIA на главном компьютере

Используйте следующий скрипт Bash для установки требуемых графических драйверов NVIDIA и CUDA Toolkit.

```bash
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/cuda-ubuntu1804.pin
sudo mv cuda-ubuntu1804.pin /etc/apt/preferences.d/cuda-repository-pin-600
sudo apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/7fa2af80.pub
sudo add-apt-repository "deb http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/ /"
sudo apt-get update
sudo apt-get -y install cuda
```

Перезагрузите компьютер и выполните следующую команду.

```bash
nvidia-smi
```

Вы должны увидеть следующие выходные данные.

![Выходные данные драйвера NVIDIA](media/spatial-analysis/nvidia-driver-output.png)

### <a name="install-docker-ce-and-nvidia-docker2-on-the-host-computer"></a>Установка DOCKER CE и NVIDIA-docker2 на главном компьютере

Установите DOCKER CE на главном компьютере.

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io
```

Установите программный пакет *NVIDIA-DOCKER-2* .

```bash
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
sudo apt-get update
sudo apt-get install -y docker-ce nvidia-docker2
sudo systemctl restart docker
```

## <a name="enable-nvidia-mps-on-the-host-computer"></a>Включение системы NVIDIA MPS на главном компьютере

> [!TIP]
> * Не устанавливайте MPS, если функция вычислений GPU меньше 7. x (до Волта). См. раздел [Совместимость CUDA](https://docs.nvidia.com/deploy/cuda-compatibility/index.html#support-title) для справки. 
> * Запустите инструкции MPS из окна терминала на главном компьютере. Не внутри экземпляра контейнера DOCKER.

Для лучшей производительности и использования настройте GPU главного компьютера для [службы многопроцессной обработки NVIDIA (MPS)](https://docs.nvidia.com/deploy/pdf/CUDA_Multi_Process_Service_Overview.pdf). Запустите инструкции MPS из окна терминала на главном компьютере.

```bash
# Set GPU(s) compute mode to EXCLUSIVE_PROCESS
sudo nvidia-smi --compute-mode=EXCLUSIVE_PROCESS

# Cronjob for setting GPU(s) compute mode to EXCLUSIVE_PROCESS on boot
echo "SHELL=/bin/bash" > /tmp/nvidia-mps-cronjob
echo "PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin" >> /tmp/nvidia-mps-cronjob
echo "@reboot   root    nvidia-smi --compute-mode=EXCLUSIVE_PROCESS" >> /tmp/nvidia-mps-cronjob

sudo chown root:root /tmp/nvidia-mps-cronjob
sudo mv /tmp/nvidia-mps-cronjob /etc/cron.d/

# Service entry for automatically starting MPS control daemon
echo "[Unit]" > /tmp/nvidia-mps.service
echo "Description=NVIDIA MPS control service" >> /tmp/nvidia-mps.service
echo "After=cron.service" >> /tmp/nvidia-mps.service
echo "" >> /tmp/nvidia-mps.service
echo "[Service]" >> /tmp/nvidia-mps.service
echo "Restart=on-failure" >> /tmp/nvidia-mps.service
echo "ExecStart=/usr/bin/nvidia-cuda-mps-control -f" >> /tmp/nvidia-mps.service
echo "" >> /tmp/nvidia-mps.service
echo "[Install]" >> /tmp/nvidia-mps.service
echo "WantedBy=multi-user.target" >> /tmp/nvidia-mps.service

sudo chown root:root /tmp/nvidia-mps.service
sudo mv /tmp/nvidia-mps.service /etc/systemd/system/

sudo systemctl --now enable nvidia-mps.service
```

## <a name="configure-azure-iot-edge-on-the-host-computer"></a>Настройка Azure IoT Edge на главном компьютере

Чтобы развернуть контейнер пространственного анализа на главном компьютере, создайте экземпляр службы [центра Интернета вещей Azure](../../iot-hub/iot-hub-create-through-portal.md) с помощью стандартной ценовой категории (S1) или бесплатной (F0). 

Используйте Azure CLI, чтобы создать экземпляр центра Интернета вещей Azure. Замените необходимые параметры. Кроме того, центр Интернета вещей Azure можно создать на [портал Azure](https://portal.azure.com/).

```bash
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```
```bash
sudo az login
```
```bash
sudo az account set --subscription "<name or ID of Azure Subscription>"
```
```bash
sudo az group create --name "<resource-group-name>" --location "<your-region>"
```
См. раздел [Поддержка регионов](https://azure.microsoft.com/global-infrastructure/services/?products=cognitive-services) для доступных регионов.
```bash
sudo az iot hub create --name "<iothub-group-name>" --sku S1 --resource-group "<resource-group-name>"
```
```bash
sudo az iot hub device-identity create --hub-name "<iothub-name>" --device-id "<device-name>" --edge-enabled
```

Необходимо установить [Azure IOT Edge](../../iot-edge/how-to-install-iot-edge.md) версии 1.0.9. Чтобы скачать правильную версию, выполните следующие действия.

Ubuntu Server 18.04:
```bash
curl https://packages.microsoft.com/config/ubuntu/18.04/multiarch/prod.list > ./microsoft-prod.list
```

Скопируйте созданный список.
```bash
sudo cp ./microsoft-prod.list /etc/apt/sources.list.d/
```

Установите открытый ключ Microsoft GPG.

```bash
curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg
```
```bash
sudo cp ./microsoft.gpg /etc/apt/trusted.gpg.d/
```

Обновите списки пакетов на устройстве.

```bash
sudo apt-get update
```

Установите выпуск 1.0.9:

```bash
sudo apt-get install iotedge=1.0.9* libiothsm-std=1.0.9*
```

Затем зарегистрируйте главный компьютер в качестве устройства IoT Edge в экземпляре центра Интернета вещей, используя [строку подключения](../../iot-edge/how-to-register-device.md?view=iotedge-2018-06).

Необходимо подключить устройство IoT Edge к центру Интернета вещей Azure. Необходимо скопировать строку подключения с IoT Edgeного устройства, созданного ранее. Кроме того, можно выполнить приведенную ниже команду в Azure CLI.

```bash
sudo az iot hub device-identity show-connection-string --device-id my-edge-device --hub-name test-iot-hub-123
```

На главном компьютере, открытом  `/etc/iotedge/config.yaml` для редактирования. Замените на `ADD DEVICE CONNECTION STRING HERE` строку подключения. Сохраните файл и закройте его. Выполните эту команду, чтобы перезапустить службу IoT Edge на главном компьютере.

```bash
sudo systemctl restart iotedge
```

Разверните контейнер пространственного анализа как модуль IoT на главном компьютере либо из [портал Azure](../../iot-edge/how-to-deploy-modules-portal.md) , либо из [Azure CLI](../cognitive-services-apis-create-account-cli.md?tabs=windows). Если вы используете портал, укажите в URI образа расположение реестра контейнеров Azure. 

Выполните следующие действия, чтобы развернуть контейнер с помощью Azure CLI.

#### <a name="azure-vm-with-gpu"></a>[Виртуальная машина Azure с графическим процессором](#tab/virtual-machine)

Виртуальную машину Azure с графическим процессором можно также использовать для выполнения пространственного анализа. В приведенном ниже примере будет использоваться виртуальная машина [серии NC](../../virtual-machines/nc-series.md?bc=%2fazure%2fvirtual-machines%2flinux%2fbreadcrumb%2ftoc.json&toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) с одним K80 GPU.

#### <a name="create-the-vm"></a>Создание виртуальной машины

Откройте мастер [создания виртуальной машины](https://ms.portal.azure.com/#create/Microsoft.VirtualMachine) на портал Azure.

Присвойте виртуальной машине имя и выберите регион (США) Западная часть США 2. Не забудьте установить значение `Availability Options` "не требуется избыточность инфраструктуры". На рисунке ниже приведена полная конфигурация и следующий шаг для справки по поиску нужного размера виртуальной машины. 

:::image type="content" source="media/spatial-analysis/virtual-machine-instance-details.png" alt-text="Сведения о конфигурации виртуальной машины." lightbox="media/spatial-analysis/virtual-machine-instance-details.png":::

Чтобы найти размер виртуальной машины, выберите "Просмотреть все размеры", а затем просмотрите список "размеры виртуальных машин хранилища без категории" Премиум ", как показано ниже.

:::image type="content" source="media/spatial-analysis/virtual-machine-sizes.png" alt-text="Размеры виртуальных машин." lightbox="media/spatial-analysis/virtual-machine-sizes.png":::

Затем выберите либо **Nc6** , либо **NC6_Promo**.

:::image type="content" source="media/spatial-analysis/promotional-selection.png" alt-text="выбор рекламы" lightbox="media/spatial-analysis/promotional-selection.png":::

Затем создайте виртуальную машину. После создания перейдите к ресурсу виртуальной машины в портал Azure и выберите в `Extensions` левой области. Появится окно расширения со всеми доступными расширениями. Выберите `NVIDIA GPU Driver Extension` , нажмите кнопку Создать и завершите работу мастера.

После успешного применения расширения перейдите на главную страницу виртуальной машины в портал Azure и нажмите кнопку `Connect` . Доступ к виртуальной машине можно получить по протоколу SSH или RDP. Протокол RDP будет полезен, так как он позволит просматривать окно визуализатора (см. описание далее). Настройте доступ по протоколу RDP, выполнив следующие [действия](../../virtual-machines/linux/use-remote-desktop.md) и открыв подключение к удаленному рабочему столу виртуальной машины.

### <a name="verify-graphics-drivers-are-installed"></a>Проверка установки графических драйверов

Выполните следующую команду, чтобы убедиться, что графические драйверы успешно установлены. 

```bash
nvidia-smi
```

Вы должны увидеть следующие выходные данные.

![Выходные данные драйвера NVIDIA](media/spatial-analysis/nvidia-driver-output.png)

### <a name="install-docker-ce-and-nvidia-docker2-on-the-vm"></a>Установка DOCKER CE и NVIDIA-docker2 на виртуальной машине

Выполните следующие команды по одному, чтобы установить DOCKER CE и NVIDIA-docker2 на виртуальной машине.

Установите DOCKER CE на главном компьютере.

```bash
sudo apt-get update
```
```bash
sudo apt-get install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common
```
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```
```bash
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```
```bash
sudo apt-get update
```
```bash
sudo apt-get install -y docker-ce docker-ce-cli containerd.io
```


Установите программный пакет *NVIDIA-DOCKER-2* .

```bash
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
```
```bash
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
```
```bash
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
```
```bash
sudo apt-get update
```
```bash
sudo apt-get install -y docker-ce nvidia-docker2
```
```bash
sudo systemctl restart docker
```

После настройки и настройки виртуальной машины выполните следующие действия, чтобы настроить Azure IoT Edge. 

## <a name="configure-azure-iot-edge-on-the-vm"></a>Настройка Azure IoT Edge на виртуальной машине

Чтобы развернуть контейнер пространственного анализа на виртуальной машине, создайте экземпляр службы [центра Интернета вещей Azure](../../iot-hub/iot-hub-create-through-portal.md) с помощью стандартной ценовой категории (S1) или бесплатной (F0).

Используйте Azure CLI, чтобы создать экземпляр центра Интернета вещей Azure. Замените необходимые параметры. Кроме того, центр Интернета вещей Azure можно создать на [портал Azure](https://portal.azure.com/).

```bash
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```
```bash
sudo az login
```
```bash
sudo az account set --subscription "<name or ID of Azure Subscription>"
```
```bash
sudo az group create --name "<resource-group-name>" --location "<your-region>"
```
См. раздел [Поддержка регионов](https://azure.microsoft.com/global-infrastructure/services/?products=cognitive-services) для доступных регионов.
```bash
sudo az iot hub create --name "<iothub-group-name>" --sku S1 --resource-group "<resource-group-name>"
```
```bash
sudo az iot hub device-identity create --hub-name "<iothub-name>" --device-id "<device-name>" --edge-enabled
```

Необходимо установить [Azure IOT Edge](../../iot-edge/how-to-install-iot-edge.md) версии 1.0.9. Чтобы скачать правильную версию, выполните следующие действия.

Ubuntu Server 18.04:
```bash
curl https://packages.microsoft.com/config/ubuntu/18.04/multiarch/prod.list > ./microsoft-prod.list
```

Скопируйте созданный список.
```bash
sudo cp ./microsoft-prod.list /etc/apt/sources.list.d/
```

Установите открытый ключ Microsoft GPG.

```bash
curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg
```
```bash
sudo cp ./microsoft.gpg /etc/apt/trusted.gpg.d/
```

Обновите списки пакетов на устройстве.

```bash
sudo apt-get update
```

Установите выпуск 1.0.9:

```bash
sudo apt-get install iotedge=1.0.9* libiothsm-std=1.0.9*
```

Затем зарегистрируйте виртуальную машину в качестве устройства IoT Edge в экземпляре центра Интернета вещей, используя [строку подключения](../../iot-edge/how-to-register-device.md?view=iotedge-2018-06).

Необходимо подключить устройство IoT Edge к центру Интернета вещей Azure. Необходимо скопировать строку подключения с IoT Edgeного устройства, созданного ранее. Кроме того, можно выполнить приведенную ниже команду в Azure CLI.

```bash
sudo az iot hub device-identity show-connection-string --device-id my-edge-device --hub-name test-iot-hub-123
```

На виртуальной машине, открытой  `/etc/iotedge/config.yaml` для редактирования. Замените на `ADD DEVICE CONNECTION STRING HERE` строку подключения. Сохраните файл и закройте его. Выполните эту команду, чтобы перезапустить службу IoT Edge на виртуальной машине.

```bash
sudo systemctl restart iotedge
```

Разверните контейнер пространственного анализа как модуль IoT на виртуальной машине либо из [портал Azure](../../iot-edge/how-to-deploy-modules-portal.md) , либо из [Azure CLI](../cognitive-services-apis-create-account-cli.md?tabs=windows). Если вы используете портал, укажите в URI образа расположение реестра контейнеров Azure. 

Выполните следующие действия, чтобы развернуть контейнер с помощью Azure CLI.

---

### <a name="iot-deployment-manifest"></a>Манифест развертывания IoT

Чтобы упростить развертывание контейнера на нескольких узловых компьютерах, можно создать файл манифеста развертывания, чтобы указать параметры создания контейнера и переменные среды. Пример манифеста развертывания [для Azure Stack](https://go.microsoft.com/fwlink/?linkid=2142179), [других настольных компьютеров](https://go.microsoft.com/fwlink/?linkid=2152270)и [ВИРТУАЛЬНОЙ машины Azure с помощью GPU](https://go.microsoft.com/fwlink/?linkid=2152189) можно найти на сайте GitHub.

В следующей таблице показаны различные переменные среды, используемые модулем IoT Edge. Их также можно задать в манифесте развертывания, связанном выше, с помощью `env` атрибута в `spatialanalysis` :

| Имя параметра | Значение | Описание|
|---------|---------|---------|
| ARCHON_LOG_LEVEL | Контактные Verbose | Уровень ведения журнала выберите одно из двух значений|
| ARCHON_SHARED_BUFFER_LIMIT | 377487360 | Не изменять|
| ARCHON_PERF_MARKER| false| Задайте для этого параметра значение true для ведения журнала производительности, в противном случае — значение false.| 
| ARCHON_NODES_LOG_LEVEL | Контактные Verbose | Уровень ведения журнала выберите одно из двух значений|
| OMP_WAIT_POLICY | ПАССИВНЫЙ | Не изменять|
| QT_X11_NO_MITSHM | 1 | Не изменять|
| APIKEY | Ваш ключ API| Собирайте это значение из портал Azure ресурса Компьютерное зрение. Его можно найти в разделе **ключ и конечная точка** для ресурса. |
| ВЫСТАВЛЕНИЕ СЧЕТОВ | URI конечной точки| Собирайте это значение из портал Azure ресурса Компьютерное зрение. Его можно найти в разделе **ключ и конечная точка** для ресурса.|
| Лицензионное соглашение | гласит | Это значение должно быть задано для *принятия* в контейнер для запуска. |
| МОНИТОР | : 1 | Это значение должно совпадать с выходными данными `echo $DISPLAY` на главном компьютере. Azure Stack пограничных устройств не отображаются. Этот параметр неприменим|
| ARCHON_GRAPH_READY_TIMEOUT | 600 | Добавьте эту переменную среды, если GPU **не** является T4 или NVIDIA 2080 Ti|
| ORT_TENSORRT_ENGINE_CACHE_ENABLE | 0 | Добавьте эту переменную среды, если GPU **не** является T4 или NVIDIA 2080 Ti|
| KEY_ENV | Ключ шифрования ASE | Добавьте эту переменную среды, если Video_URL является замаскированной строкой |
| IV_ENV | Вектор инициализации | Добавьте эту переменную среды, если Video_URL является замаскированной строкой|

> [!IMPORTANT]
> Для запуска контейнера необходимо указать параметры `Eula`, `Billing` и `ApiKey`. В противном случае контейнер не запустится.  Дополнительные сведения см. в [разделе о выставлении счетов](#billing).

После обновления манифеста развертывания для [Azure Stack пограничных устройств](https://go.microsoft.com/fwlink/?linkid=2142179), [настольного компьютера](https://go.microsoft.com/fwlink/?linkid=2152270) или [виртуальной машины Azure с графическим процессором](https://go.microsoft.com/fwlink/?linkid=2152189) с собственными параметрами и выбором операций можно использовать приведенную ниже команду [Azure CLI](../cognitive-services-apis-create-account-cli.md?tabs=windows) для развертывания контейнера на главном компьютере в качестве модуля IOT Edge.

```azurecli
sudo az login
sudo az extension add --name azure-iot
sudo az iot edge set-modules --hub-name "<iothub-name>" --device-id "<device-name>" --content DeploymentManifest.json --subscription "<name or ID of Azure Subscription>"
```

|Параметр  |Описание  |
|---------|---------|
| `--hub-name` | Имя центра Интернета вещей Azure. |
| `--content` | Имя файла развертывания. |
| `--target-condition` | Имя устройства IoT Edge для главного компьютера. |
| `-–subscription` | Идентификатор или имя подписки. |

Эта команда запустит развертывание. Перейдите на страницу экземпляра центра Интернета вещей Azure на портал Azure, чтобы просмотреть состояние развертывания. Состояние может отображаться как *417 — конфигурация развертывания устройства не устанавливается* до тех пор, пока устройство не закончит загрузку образов контейнеров и не начнет работать.

## <a name="validate-that-the-deployment-is-successful"></a>Проверка успешности развертывания

Проверить это можно несколькими способами. Нахождение *состояния среды выполнения* в **параметрах модуля IOT Edge** для модуля пространственного анализа в экземпляре центра интернета вещей Azure на портал Azure. Убедитесь, что **требуемое значение** и **Сообщаемое значение** для *состояния среды выполнения* *выполняется.*

![Пример проверки развертывания](./media/spatial-analysis/deployment-verification.png)

После завершения развертывания и запуска контейнера **главный компьютер** начнет отправлять события в центр Интернета вещей Azure. Если вы использовали `.debug` версию операций, вы увидите окно визуализатора для каждой камеры, настроенной в манифесте развертывания. Теперь можно определить строки и зоны, которые необходимо отслеживать в манифесте развертывания, и следовать инструкциям по повторному развертыванию. 

## <a name="configure-the-operations-performed-by-spatial-analysis"></a>Настройка операций, выполняемых пространственным анализом

Для настройки контейнера на использование подключенных камер необходимо использовать [операции пространственного анализа](spatial-analysis-operations.md) , настроить операции и многое другое. Для каждого настроенного устройства камеры операции пространственного анализа будут создавать выходной поток сообщений JSON, отправленный в ваш экземпляр центра Интернета вещей Azure.

## <a name="redeploy-or-delete-the-deployment"></a>Повторное развертывание или удаление развертывания

Если необходимо обновить развертывание, необходимо убедиться в том, что предыдущие развертывания успешно развернуты, или необходимо удалить IoT Edge незавершенные развертывания устройств. В противном случае эти развертывания будут продолжены, и система останется в неисправном состоянии. Можно использовать портал Azure или [Azure CLI](../cognitive-services-apis-create-account-cli.md?tabs=windows).

## <a name="use-the-output-generated-by-the-container"></a>Использование выходных данных, созданных контейнером

Если вы хотите начать использование выходных данных, созданных контейнером, см. следующие статьи:

*   Используйте пакет SDK концентратора событий Azure для выбранного языка программирования, чтобы подключиться к конечной точке центра Интернета вещей Azure и получить события. Дополнительные сведения см. в [статье чтение сообщений с устройства в облако из встроенной конечной точки](../../iot-hub/iot-hub-devguide-messages-read-builtin.md) . 
*   Настройте маршрутизацию сообщений в центре Интернета вещей Azure, чтобы отправить события в другие конечные точки или сохранить события в хранилище BLOB-объектов Azure и т. д. Дополнительные сведения см. в статье [Маршрутизация сообщений центра Интернета вещей](../../iot-hub/iot-hub-devguide-messages-d2c.md) . 

## <a name="running-spatial-analysis-with-a-recorded-video-file"></a>Выполнение пространственного анализа с помощью записанного видео-файла

Пространственный анализ можно использовать с записанными или динамическими видео. Чтобы использовать пространственный анализ для записанного видео, попробуйте записать видеофайл и сохранить его как MP4-файл. Создайте учетную запись хранения BLOB-объектов в Azure или используйте существующую. Затем обновите следующие параметры хранилища BLOB-объектов в портал Azure:
    1. Изменить **безопасное перемещение требуется** для **отключения**
    2. Изменение **разрешения общего доступа к BLOB-объекту** для **включения**

Перейдите к разделу **контейнера** и либо создайте новый контейнер, либо используйте существующий. Затем отправьте видеофайл в контейнер. Разверните параметры файла для отправленного файла и выберите **создать SAS**. Не забудьте задать **дату истечения срока действия** , достаточную для покрытия периода тестирования. Задайте для параметра **Разрешенные протоколы** значение *http* (*HTTPS* не поддерживается).

Щелкните **создать маркер SAS и URL-адрес** и скопируйте URL-адрес SAS большого двоичного объекта. Замените значение, начинающееся `https` с, `http` и проверьте URL-адрес в браузере, поддерживающем воспроизведение видео.

Замените `VIDEO_URL` в манифесте развертывания для [Azure Stack пограничных устройств](https://go.microsoft.com/fwlink/?linkid=2142179), [настольного компьютера](https://go.microsoft.com/fwlink/?linkid=2152270)или [виртуальной машины Azure с помощью GPU](https://go.microsoft.com/fwlink/?linkid=2152189) с созданным URL-адресом для всех диаграмм. Задайте `VIDEO_IS_LIVE` для значение `false` и повторно разверните контейнер пространственного анализа с обновленным манифестом. См. пример ниже.

Модуль пространственного анализа начнет использовать видеофайл и будет постоянно автоматически воспроизводить его.


```json
"zonecrossing": {
    "operationId" : "cognitiveservices.vision.spatialanalysis-personcrossingpolygon",
    "version": 1,
    "enabled": true,
    "parameters": {
        "VIDEO_URL": "Replace http url here",
        "VIDEO_SOURCE_ID": "personcountgraph",
        "VIDEO_IS_LIVE": false,
      "VIDEO_DECODE_GPU_INDEX": 0,
        "DETECTOR_NODE_CONFIG": "{ \"gpu_index\": 0, \"do_calibration\": true }",
        "SPACEANALYTICS_CONFIG": "{\"zones\":[{\"name\":\"queue\",\"polygon\":[[0.3,0.3],[0.3,0.9],[0.6,0.9],[0.6,0.3],[0.3,0.3]], \"events\": [{\"type\": \"zonecrossing\", \"config\": {\"threshold\": 16.0, \"focus\": \"footprint\"}}]}]}"
    }
   },

```

## <a name="troubleshooting"></a>Диагностика

Если при запуске или запуске контейнера возникли проблемы, см. раздел [телеметрии и устранение неполадок](spatial-analysis-logging.md) для решения распространенных проблем. В этой статье также содержатся сведения о создании и сборе журналов и сборе сведений о работоспособности системы.

## <a name="billing"></a>Выставление счетов

Контейнер пространственного анализа отправляет сведения о выставлении счетов в Azure с помощью Компьютерное зрение ресурса в учетной записи Azure. Использование пространственного анализа в общедоступной предварительной версии сейчас является бесплатным. 

Контейнеры Cognitive Services Azure не лицензированы для запуска без подключения к конечной точке контроля использования и выставления счетов. Вам необходимо разрешить контейнерам непрерывную передачу данных для выставления счетов в конечную точку выставления счетов. Cognitive Services контейнеры не отправляют в корпорацию Майкрософт данные о клиентах, например проанализируемые видео или изображения.


## <a name="summary"></a>Итоги

В этой статье вы узнали основные понятия и рабочий процесс по скачиванию, установке и запуску контейнера пространственного анализа. В разделе "Сводка" сделайте следующее.

* Пространственный анализ — это контейнер Linux для DOCKER.
* Образы контейнеров загружаются из реестра контейнеров Майкрософт.
* Образы контейнеров запускаются как модули IoT в Azure IoT Edge.
* Как настроить контейнер и развернуть его на размещающем компьютере.

## <a name="next-steps"></a>Дальнейшие действия

* [Развертывание веб-приложения инвентаризации людей](spatial-analysis-web-app.md)
* [Настройка операций пространственного анализа](spatial-analysis-operations.md)
* [Ведение журнала и устранение неполадок](spatial-analysis-logging.md)
* [Путеводитель по размещению камеры](spatial-analysis-camera-placement.md)
* [Рекомендации по размещению зон и строк](spatial-analysis-zone-line-placement.md)