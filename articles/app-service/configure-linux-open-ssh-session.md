---
title: Доступ по протоколу SSH для контейнеров Linux
description: Вы можете открыть сеанс SSH для контейнера Linux в службе приложений Azure. Настраиваемые контейнеры Linux поддерживаются с некоторыми изменениями пользовательского образа.
keywords: служба приложений azure, веб-приложение, linux, oss
author: msangapu-msft
ms.assetid: 66f9988f-8ffa-414a-9137-3a9b15a5573c
ms.topic: article
ms.date: 02/23/2021
ms.author: msangapu
ms.custom: seodec18
ms.openlocfilehash: 8e9dd76b60d05b9fa5e3a4aaf7ccc6663f4a969b
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101709042"
---
# <a name="open-an-ssh-session-to-a-linux-container-in-azure-app-service"></a>Открытие сеанса SSH для контейнера Linux в службе приложений Azure

[Secure Shell (SSH)](https://wikipedia.org/wiki/Secure_Shell) широко используется для удаленного выполнения административных команд из терминала командной строки. Служба приложений в Linux обеспечивает поддержку SSH в контейнере приложения. 

![SSH службы приложений Linux](./media/configure-linux-open-ssh-session/app-service-linux-ssh.png)

Вы также можете подключиться к контейнеру непосредственно из локального компьютера, где ведется разработка, используя SSH и SFTP.

## <a name="open-ssh-session-in-browser"></a>Открытие сеанса SSH в браузере

[!INCLUDE [Open SSH session in browser](../../includes/app-service-web-ssh-connect-no-h.md)]

## <a name="use-ssh-support-with-custom-docker-images"></a>Применение поддержки SSH в пользовательских образах Docker

См. раздел [Настройка SSH в пользовательском контейнере](configure-custom-container.md#enable-ssh).

## <a name="open-ssh-session-from-remote-shell"></a>Открытие сеанса SSH из удаленной оболочки

> [!NOTE]
> Эта функция сейчас доступна в режиме предварительной версии.
>

С помощью TCP-туннеля можно создать сетевое подключение между веб-приложением для контейнеров и компьютером, на котором выполняется разработка, с проверкой подлинности через WebSocket. Это позволяет открыть из необходимого клиента сеанс SSH с контейнером, запущенным в службе приложений.

Чтобы начать работу, нужно установить [Azure CLI](/cli/azure/install-azure-cli). Чтобы ознакомиться с принципами работы без установки Azure CLI, перейдите к статье [Обзор Azure Cloud Shell](../cloud-shell/overview.md). 

Откройте удаленное подключение к приложению, используя команду [az webapp remote-connection create](/cli/azure/ext/webapp/webapp/remote-connection#ext-webapp-az-webapp-remote-connection-create). Укажите _\<subscription-id>_ _\<group-name>_ и \_ \<app-name> _ для своего приложения.

```azurecli-interactive
az webapp create-remote-connection --subscription <subscription-id> --resource-group <resource-group-name> -n <app-name> &
```

> [!TIP]
> Знак `&` в конце команды предназначен исключительно для удобства при использовании Cloud Shell. Он позволяет запустить процесс в фоновом режиме и выполнять следующую команду в той же оболочке.

> [!NOTE]
> Если эта команда завершается неудачно, убедитесь, что [Удаленная отладка](https://medium.com/@auchenberg/introducing-remote-debugging-of-node-js-apps-on-azure-app-service-from-vs-code-in-public-preview-9b8d83a6e1f0) *отключена* с помощью следующей команды:
>
> ```azurecli-interactive
> az webapp config set --resource-group <resource-group-name> -n <app-name> --remote-debugging-enabled=false
> ```

В выходных данных команды содержатся сведения, необходимые для того, чтобы открыть сеанс SSH.

```output
Port 21382 is open
SSH is available { username: root, password: Docker! }
Start your favorite client and connect to port 21382
```

Откройте сеанс SSH с контейнером с помощью необходимого клиента, используя локальный порт. В следующем примере используется команда по умолчанию [ssh](https://ss64.com/bash/ssh.html).

```bash
ssh root@127.0.0.1 -p <port>
```

Если отобразится запрос, введите `yes`, чтобы продолжить установку подключения. Затем появится запрос на ввод пароля. Используйте показанный ранее `Docker!`.

<pre>
Warning: Permanently added '[127.0.0.1]:21382' (ECDSA) to the list of known hosts.
root@127.0.0.1's password:
</pre>

После выполнения проверки подлинности появится приветственный экран сеанса.

<pre>
  _____
  /  _  \ __________ _________   ____
 /  /_\  \___   /  |  \_  __ \_/ __ \
/    |    \/    /|  |  /|  | \/\  ___/
\____|__  /_____ \____/ |__|    \___  &gt;
        \/      \/                  \/
A P P   S E R V I C E   O N   L I N U X

0e690efa93e2:~#
</pre>

Вы подключены к соединителю.  

Попробуйте выполнить команду [top](https://ss64.com/bash/top.html). В списке процессов должен содержаться процесс приложения. В приведенном ниже примере выходных данных — это процесс с `PID 263`.

<pre>
Mem: 1578756K used, 127032K free, 8744K shrd, 201592K buff, 341348K cached
CPU:   3% usr   3% sys   0% nic  92% idle   0% io   0% irq   0% sirq
Load average: 0.07 0.04 0.08 4/765 45738
  PID  PPID USER     STAT   VSZ %VSZ CPU %CPU COMMAND
    1     0 root     S     1528   0%   0   0% /sbin/init
  235     1 root     S     632m  38%   0   0% PM2 v2.10.3: God Daemon (/root/.pm2)
  263   235 root     S     630m  38%   0   0% node /home/site/wwwroot/app.js
  482   291 root     S     7368   0%   0   0% sshd: root@pts/0
45513   291 root     S     7356   0%   0   0% sshd: root@pts/1
  291     1 root     S     7324   0%   0   0% /usr/sbin/sshd
  490   482 root     S     1540   0%   0   0% -ash
45539 45513 root     S     1540   0%   0   0% -ash
45678 45539 root     R     1536   0%   0   0% top
45733     1 root     Z        0   0%   0   0% [init]
45734     1 root     Z        0   0%   0   0% [init]
45735     1 root     Z        0   0%   0   0% [init]
45736     1 root     Z        0   0%   0   0% [init]
45737     1 root     Z        0   0%   0   0% [init]
45738     1 root     Z        0   0%   0   0% [init]
</pre>

## <a name="next-steps"></a>Дальнейшие действия

Если у вас возникли вопросы, опубликуйте их на [форуме Azure](/answers/topics/azure-webapps.html).

Дополнительные сведения о платформе "Веб-приложения для контейнеров" см. в статьях:

* [Общие сведения об удаленной отладке приложений Node.js в службе приложений Azure из VS Code](https://medium.com/@auchenberg/introducing-remote-debugging-of-node-js-apps-on-azure-app-service-from-vs-code-in-public-preview-9b8d83a6e1f0)
* [Краткое руководство. Запуск пользовательского контейнера в Службе приложений](quickstart-custom-container.md?pivots=container-linux)
* [Использование Ruby в службе приложений Azure на платформе Linux](quickstart-ruby.md).
* [Часто задаваемые вопросы о службе приложений Azure Веб-приложение для контейнеров](faq-app-service-linux.md)
