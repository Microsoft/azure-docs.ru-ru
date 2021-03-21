---
title: Обновление агента Linux для Azure с сайта GitHub
description: Сведения об обновлении агента Linux для Azure на виртуальной машине Linux
ms.topic: article
ms.service: virtual-machines
ms.subservice: extensions
author: amjads1
ms.author: amjads
ms.collection: linux
ms.date: 08/02/2017
ms.openlocfilehash: a274435d6e0fc32bdf5b2ab04702ed971d7c5175
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102558990"
---
# <a name="how-to-update-the-azure-linux-agent-on-a-vm"></a>Как обновить агент Azure Linux на виртуальной машине

Для обновления [агента Linux для Azure](https://github.com/Azure/WALinuxAgent) на виртуальной машине Linux требуется:

- работающая виртуальная машина Linux в Azure;
- подключение к этой виртуальной машине Linux с помощью протокола SSH.

Пакет нужно всегда сначала проверять в репозитории дистрибутива Linux. Возможно, доступный пакет будет не последней версии. Тем не менее после включения автоматического обновления агент Linux будет всегда получать последнее обновление. При возникновении проблем во время установки из диспетчеров пакетов за поддержкой обратитесь к поставщику дистрибутива.

> [!NOTE]
> Дополнительные сведения см. [в статье Рекомендуемые дистрибутивы Linux в Azure](../linux/endorsed-distros.md) .

Ознакомьтесь со статьей [Minimum version support for virtual machine agents in Azure](https://support.microsoft.com/help/4049215/extensions-and-virtual-machine-agent-minimum-version-support) (Минимальная поддержка версий для агентов виртуальных машин в Azure) перед продолжением.


## <a name="ubuntu"></a>Ubuntu

Проверка текущей версии пакета

```bash
apt list --installed | grep walinuxagent
```

Обновление кэша пакета

```bash
sudo apt-get -qq update
```

Установка последней версии пакета

```bash
sudo apt-get install walinuxagent
```

Убедитесь, что включено автоматическое обновление. Сначала проверьте, включено ли автоматическое обновление:

```bash
cat /etc/waagent.conf
```

Найдите параметр AutoUpdate.Enabled. Если отображаются следующие результаты, значит эта возможность включена:

```bash
# AutoUpdate.Enabled=y
AutoUpdate.Enabled=y
```

Чтобы включить автоматическое обновление, выполните следующую команду:

```bash
sudo sed -i 's/# AutoUpdate.Enabled=n/AutoUpdate.Enabled=y/g' /etc/waagent.conf
```

Перезапуск службы вааженгт для 14,04

```bash
initctl restart walinuxagent
```

Перезапуск службы waagent для 16,04/17,04

```bash
systemctl restart walinuxagent.service
```

## <a name="red-hat--centos"></a>Red Hat или CentOS

### <a name="rhelcentos-6"></a>RHEL или CentOS 6

Проверка текущей версии пакета

```bash
sudo yum list WALinuxAgent
```

Проверка доступных обновлений

```bash
sudo yum check-update WALinuxAgent
```

Установка последней версии пакета

```bash
sudo yum install WALinuxAgent
```

Гарантия включения автоматического обновления 

Сначала проверьте, включено ли автоматическое обновление:

```bash
cat /etc/waagent.conf
```

Найдите параметр AutoUpdate.Enabled. Если отображаются следующие результаты, значит эта возможность включена:

```bash
# AutoUpdate.Enabled=y
AutoUpdate.Enabled=y
```

Чтобы включить автоматическое обновление, выполните следующую команду:

```bash
sudo sed -i 's/\# AutoUpdate.Enabled=y/AutoUpdate.Enabled=y/g' /etc/waagent.conf
```

Перезапустите службу waagent

```
sudo service waagent restart
```

## <a name="rhelcentos-7"></a>RHEL или CentOS 7

Проверка текущей версии пакета

```bash
sudo yum list WALinuxAgent
```

Проверка доступных обновлений

```bash
sudo yum check-update WALinuxAgent
```

Установка последней версии пакета

```bash
sudo yum install WALinuxAgent  
```

Убедитесь, что включено автоматическое обновление. Сначала проверьте, включено ли автоматическое обновление:

```bash
cat /etc/waagent.conf
```

Найдите параметр AutoUpdate.Enabled. Если отображаются следующие результаты, значит эта возможность включена:

```bash
# AutoUpdate.Enabled=y
AutoUpdate.Enabled=y
```

Чтобы включить автоматическое обновление, выполните следующую команду:

```bash
sudo sed -i 's/# AutoUpdate.Enabled=n/AutoUpdate.Enabled=y/g' /etc/waagent.conf
```

Перезапустите службу waagent

```bash
sudo systemctl restart waagent.service
```

## <a name="suse-sles"></a>SUSE SLES

### <a name="suse-sles-11-sp4"></a>SUSE SLES 11 SP4

Проверка текущей версии пакета

```bash
zypper info python-azure-agent
```

Проверьте доступные обновления. Проанализировав выходные данные выше, можно определить, обновлен ли пакет.

Установка последней версии пакета

```bash
sudo zypper install python-azure-agent
```

Гарантия включения автоматического обновления 

Сначала проверьте, включено ли автоматическое обновление:

```bash
cat /etc/waagent.conf
```

Найдите параметр AutoUpdate.Enabled. Если отображаются следующие результаты, значит эта возможность включена:

```bash
# AutoUpdate.Enabled=y
AutoUpdate.Enabled=y
```

Чтобы включить автоматическое обновление, выполните следующую команду:

```bash
sudo sed -i 's/# AutoUpdate.Enabled=n/AutoUpdate.Enabled=y/g' /etc/waagent.conf
```

Перезапустите службу waagent

```bash
sudo /etc/init.d/waagent restart
```

### <a name="suse-sles-12-sp2"></a>SUSE SLES 12 SP2

Проверка текущей версии пакета

```bash
zypper info python-azure-agent
```

Проверка доступных обновлений

Проанализировав выходные данные выше, можно определить, обновлен ли пакет.

Установка последней версии пакета

```bash
sudo zypper install python-azure-agent
```

Гарантия включения автоматического обновления 

Сначала проверьте, включено ли автоматическое обновление:

```bash
cat /etc/waagent.conf
```

Найдите параметр AutoUpdate.Enabled. Если отображаются следующие результаты, значит эта возможность включена:

```bash
# AutoUpdate.Enabled=y
AutoUpdate.Enabled=y
```

Чтобы включить автоматическое обновление, выполните следующую команду:

```bash
sudo sed -i 's/# AutoUpdate.Enabled=n/AutoUpdate.Enabled=y/g' /etc/waagent.conf
```

Перезапустите службу waagent

```bash
sudo systemctl restart waagent.service
```

## <a name="debian"></a>Debian

### <a name="debian-7-jesse-debian-7-stretch"></a>Debian 7 "Джесси"/Debian 7 "Stretch"

Проверка текущей версии пакета

```bash
dpkg -l | grep waagent
```

Обновление кэша пакета

```bash
sudo apt-get -qq update
```

Установка последней версии пакета

```bash
sudo apt-get install waagent
```

Включить автоматическое обновление агента в этой версии Debian отсутствует версия >= 2.0.16, поэтому для нее недоступно Автообновление. Проанализировав выходные данные команды выше, можно определить, обновлен ли пакет.

### <a name="debian-8-jessie--debian-9-stretch"></a>Debian 8 "Jessie" и Debian 9 "Stretch"

Проверка текущей версии пакета

```bash
apt list --installed | grep waagent
```

Обновление кэша пакета

```bash
sudo apt-get -qq update
```

Установка последней версии пакета

```bash
sudo apt-get install waagent
```

Убедитесь, что сначала включено автоматическое обновление, и убедитесь, что оно включено:

```bash
cat /etc/waagent.conf
```

Найдите параметр AutoUpdate.Enabled. Если отображаются следующие результаты, значит эта возможность включена:

```bash
AutoUpdate.Enabled=y
AutoUpdate.Enabled=y
```

Чтобы включить автоматическое обновление, выполните следующую команду:

```bash
sudo sed -i 's/# AutoUpdate.Enabled=n/AutoUpdate.Enabled=y/g' /etc/waagent.conf
Restart the waagent service
sudo systemctl restart walinuxagent.service
```

## <a name="oracle-linux-6-and-oracle-linux-7"></a>Oracle Linux 6 и Oracle Linux 7

При работе с Oracle Linux убедитесь, что включен репозиторий `Addons` . Откройте для редактирования файл `/etc/yum.repos.d/public-yum-ol6.repo` (Oracle Linux 6) или `/etc/yum.repos.d/public-yum-ol7.repo` (Oracle Linux) и замените в нем строку `enabled=0` на строку `enabled=1` в разделе **[ol6_addons]** или **[ol7_addons]**.

Затем введите следующие команды для установки последней версии агента Linux для Azure:

```bash
sudo yum install WALinuxAgent
```

Если вы не нашли репозиторий надстроек, можно просто добавить следующие строки в конец REPO-файла в соответствии с используемой версией Oracle Linux.

Для виртуальных машин Oracle Linux 6.

```sh
[ol6_addons]
name=Add-Ons for Oracle Linux $releasever ($basearch)
baseurl=https://public-yum.oracle.com/repo/OracleLinux/OL6/addons/x86_64
gpgkey=https://public-yum.oracle.com/RPM-GPG-KEY-oracle-ol6
gpgcheck=1
enabled=1
```

Для виртуальных машин Oracle Linux 7.

```sh
[ol7_addons]
name=Oracle Linux $releasever Add ons ($basearch)
baseurl=http://public-yum.oracle.com/repo/OracleLinux/OL7/addons/$basearch/
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-oracle
gpgcheck=1
enabled=1
```

Затем введите следующую команду:

```bash
sudo yum update WALinuxAgent
```

Обычно это все, что требуется. Но если по какой-либо причине необходимо установить его с сайта https://github.com напрямую, сделайте следующее.


## <a name="update-the-linux-agent-when-no-agent-package-exists-for-distribution"></a>Обновление агента Linux, если пакет агента для дистрибутива не существует

Установите wget (некоторые дистрибутивы не устанавливают его по умолчанию, например Red Hat, CentOS и Oracle Linux версий 6.4 и 6.5), введя `sudo yum install wget` в командной строке.

### <a name="1-download-the-latest-version"></a>1. Скачайте последнюю версию.
Откройте [выпуск агента Linux для Azure в GitHub](https://github.com/Azure/WALinuxAgent/releases) на веб-странице и узнайте номер последней версии. (Номер текущей версии можно узнать, введя `waagent --version`.)

Для версии 2.2.x или более поздней версии введите:
```bash
wget https://github.com/Azure/WALinuxAgent/archive/v2.2.x.zip
unzip v2.2.x.zip
cd WALinuxAgent-2.2.x
```

В следующей строке используется версия 2.2.0 в качестве примера:

```bash
wget https://github.com/Azure/WALinuxAgent/archive/v2.2.14.zip
unzip v2.2.14.zip  
cd WALinuxAgent-2.2.14
```

### <a name="2-install-the-azure-linux-agent"></a>2. Установка агента Linux для Azure

Для версии 2.2. x используйте: может потребоваться сначала установить пакет `setuptools` . см. [здесь](https://pypi.python.org/pypi/setuptools). Далее выполните:

```bash
sudo python setup.py install
```

Убедитесь, что включено автоматическое обновление. Сначала проверьте, включено ли автоматическое обновление:

```bash
cat /etc/waagent.conf
```

Найдите параметр AutoUpdate.Enabled. Если отображаются следующие результаты, значит эта возможность включена:

```bash
# AutoUpdate.Enabled=y
AutoUpdate.Enabled=y
```

Чтобы включить автоматическое обновление, выполните следующую команду:

```bash
sudo sed -i 's/# AutoUpdate.Enabled=n/AutoUpdate.Enabled=y/g' /etc/waagent.conf
```

### <a name="3-restart-the-waagent-service"></a>3. Перезапустите службу waagent
Для большинства дистрибутивов Linux:

```bash
sudo service waagent restart
```

Для Ubuntu используйте:

```bash
sudo service walinuxagent restart
```

Для CoreOS используйте:

```bash
sudo systemctl restart waagent
```

### <a name="4-confirm-the-azure-linux-agent-version"></a>4. Подтвердите версию агента Linux для Azure
    
```bash
waagent -version
```

Для CoreOS приведенная выше команда может не работать.

Вы увидите, что агент Linux для Azure обновлен до новой версии.

Дополнительные сведения об агенте Linux для Azure см. в [файле сведений агента Linux для Azure](https://github.com/Azure/WALinuxAgent).
