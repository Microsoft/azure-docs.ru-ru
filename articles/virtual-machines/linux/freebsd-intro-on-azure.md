---
title: Введение в FreeBSD в Azure
description: Узнайте о том, как использовать виртуальные машины FreeBSD в Azure.
author: thomas1206
ms.service: virtual-machines
ms.collection: linux
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 09/13/2017
ms.author: mimckitt
ms.openlocfilehash: da51fabaaa3c02137770f0b2d9a851b1f6702980
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105044055"
---
# <a name="introduction-to-freebsd-on-azure"></a>Введение в FreeBSD в Azure
В этой статье представлен обзор запуска виртуальной машины FreeBSD в Azure.

## <a name="overview"></a>Обзор
FreeBSD для Microsoft Azure — это расширенная операционная система, которая использовалась для управления мощными современными серверами, настольными компьютерами и внедренными платформами.

Корпорация Майкрософт предоставляет образы FreeBSD на платформе Azure с предварительно настроенным [гостевым агентом виртуальной машины Azure](https://github.com/Azure/WALinuxAgent/). В настоящее время она предлагает образы FreeBSD следующих версий:

- FreeBSD 10.4 в Azure Marketplace
- FreeBSD 11.2 в Azure Marketplace
- FreeBSD 12.0 в Azure Marketplace

Этот агент отвечает за обмен данными между виртуальной машиной FreeBSD и Azure Fabric при таких операциях, как подготовка виртуальной машины к первому использованию (имя пользователя, пароль или ключ SSH, имя узла и т. д.) и выборочное включение функций расширений виртуальной машины.

Стратегия в отношении текущей и будущих версий FreeBSD — обеспечивать актуальность и предоставлять последние версии вскоре после их публикации командой технических специалистов по выпуску FreeBSD.

### <a name="create-a-freebsd-vm-through-azure-cli-on-freebsd"></a>Создайте виртуальную машину FreeBSD с помощью Azure CLI в FreeBSD.
Сначала необходимо установить [Azure CLI](/cli/azure/get-started-with-azure-cli), выполнив следующую команду на компьютере FreeBSD.

```bash 
curl -L https://aka.ms/InstallAzureCli | bash
```

Если на компьютере FreeBSD не установлена оболочка bash, выполните следующую команду перед установкой. 

```bash
sudo pkg install bash
```

Если на компьютере FreeBSD не установлен python, выполните следующие команды перед установкой. 

```bash
sudo pkg install python38
cd /usr/local/bin 
sudo rm /usr/local/bin/python 
sudo ln -s /usr/local/bin/python3.8 /usr/local/bin/python
```

Во время установки вам будет задан вопрос `Modify profile to update your $PATH and enable shell/tab completion now? (Y/n)`. Если вы ответили `y` и указали `/etc/rc.conf` в качестве `a path to an rc file to update`, может возникнуть следующая ошибка: `ERROR: [Errno 13] Permission denied`. Чтобы устранить эту проблему, нужно предоставить текущему пользователю права на запись для файла `etc/rc.conf`.

Теперь можно выполнить вход в Azure и создать виртуальную машину FreeBSD. Ниже приведен пример создания виртуальной машины FreeBSD версии 11.0. Также можно добавить параметр `--public-ip-address-dns-name` с глобальным именем DNS для созданного общедоступного IP-адреса. 

```azurecli
az login 
az group create --name myResourceGroup --location eastus
az vm create --name myFreeBSD11 \
    --resource-group myResourceGroup \
    --image MicrosoftOSTC:FreeBSD:11.0:latest \
    --admin-username azureuser \
    --generate-ssh-keys
```

Затем вы сможете выполнить вход в виртуальную машину FreeBSD с помощью IP-адреса, который приведен в выходных данных описанного выше развертывания. 

```bash
ssh azureuser@xx.xx.xx.xx -i /etc/ssh/ssh_host_rsa_key
```   

## <a name="vm-extensions-for-freebsd"></a>Расширения виртуальной машины для FreeBSD
Ниже приведены поддерживаемые во FreeBSD расширения виртуальной машины.

### <a name="vmaccess"></a>VMAccess
Возможности расширения [VMAccess](https://github.com/Azure/azure-linux-extensions/tree/master/VMAccess) :

* сброс пароля исходного пользователя sudo;
* создание пользователя sudo с указанным паролем;
* настройка заданного значения открытого ключа узла;
* сброс открытого ключа узла, указанного во время подготовки виртуальной машины, если не указан ключ узла;
* открытие порта SSH (22) и восстановление sshd_config, если для reset_ssh задано значение true;
* удаление существующего пользователя;
* проверка дисков;
* восстановление добавленного диска.

### <a name="customscript"></a>CustomScript
Возможности расширения [CustomScript](https://github.com/Azure/azure-linux-extensions/tree/master/CustomScript) :

* скачивание предоставленных пользовательских сценариев из службы хранилища Azure или общедоступного внешнего хранилища (например, Github);
* выполнение сценария точки входа;
* поддержка встроенных команд;
* автоматическое преобразование символа новой строки в стиле Windows в сценариях оболочки и Python;
* автоматическое удаление спецификации в сценариях оболочки и Python;
* защита конфиденциальных данных в commandToExecute.

> [!NOTE]
> В настоящий момент виртуальная машина FreeBSD поддерживает только CustomScript версии 1.x.  

## <a name="authentication-user-names-passwords-and-ssh-keys"></a>Аутентификация: имена пользователей, пароли и ключи SSH
При создании виртуальной машины FreeBSD с помощью портала Azure необходимо указать имя пользователя, пароль или открытый ключ SSH.
Имена пользователей для развертывания виртуальной машины FreeBSD в Azure не должны совпадать с именами системных учетных записей (UID < 100), уже присутствующих в виртуальной машине (например, root).
В настоящее время поддерживается только ключ SSH на основе RSA. Многострочный ключ SSH должен начинаться с `---- BEGIN SSH2 PUBLIC KEY ----` и заканчиваться `---- END SSH2 PUBLIC KEY ----`.

## <a name="obtaining-superuser-privileges"></a>Получение привилегий суперпользователя
Учетная запись пользователя, указанная во время развертывания экземпляра виртуальной машины в Azure, является привилегированной учетной записью. Пакет sudo установлен в опубликованном образе FreeBSD.
После входа в систему с помощью этой учетной записи пользователя вы сможете выполнять команды от учетной записи root, используя соответствующий синтаксис команд.

```
$ sudo <COMMAND>
```

При необходимости можно получить доступ к оболочке root с помощью команды `sudo -s`.

## <a name="known-issues"></a>Известные проблемы
[Гостевой агент виртуальной машины Azure](https://github.com/Azure/WALinuxAgent/) версии 2.2.2 имеет [известную проблему](https://github.com/Azure/WALinuxAgent/pull/517), вызывающую сбой подготовки виртуальной машины FreeBSD в Azure. Ошибка исправлена в [гостевом агенте виртуальной машины Azure](https://github.com/Azure/WALinuxAgent/) версии 2.2.3 и более поздних выпусках. 

## <a name="next-steps"></a>Дальнейшие действия
* Перейдите на сайт [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/thefreebsdfoundation.freebsd-12_2?tab=Overview) , чтобы создать виртуальную машину FreeBSD.
