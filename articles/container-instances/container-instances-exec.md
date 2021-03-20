---
title: Выполнение команд в работающем экземпляре контейнера
description: Узнайте, как выполнить команду в контейнере, который выполняется в данный момент в службе "Экземпляры контейнеров"
ms.topic: article
ms.date: 03/30/2018
ms.openlocfilehash: de48e6ac246e2b0751561b4c60bb63d88b599bdf
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "79247206"
---
# <a name="execute-a-command-in-a-running-azure-container-instance"></a>Выполнение команды в экземпляре контейнера Azure

Служба "Экземпляры контейнеров Azure" поддерживает выполнения команды в запущенном контейнере. Выполнять команды в уже запущенном контейнере особенно удобно при разработке приложений и устранении неполадок. Чаще всего эту функцию используют для запуска интерактивной оболочки, чтобы устранять проблемы в запущенном контейнере.

## <a name="run-a-command-with-azure-cli"></a>Выполнение команды с помощью Azure CLI

Чтобы выполнить команду в запущенном контейнере, введите [az container exec][az-container-exec] в [Azure CLI][azure-cli]:

```azurecli
az container exec --resource-group <group-name> --name <container-group-name> --exec-command "<command>"
```

Например, для запуска оболочки Bash в контейнер Nginx:

```azurecli
az container exec --resource-group myResourceGroup --name mynginx --exec-command "/bin/bash"
```

В приведенном ниже примере оболочка Bash запущена в запущенном контейнере Linux. Она предоставляет терминал, в котором выполняется `ls`:

```output
root@caas-83e6c883014b427f9b277a2bba3b7b5f-708716530-2qv47:/# ls
bin   dev  home  lib64  mnt  proc  run   srv  tmp  var
boot  etc  lib   media  opt  root  sbin  sys  usr
root@caas-83e6c883014b427f9b277a2bba3b7b5f-708716530-2qv47:/# exit
exit
Bye.
```

В этом примере интерфейс командной строки запущен в запущенном контейнере Nanoserver:

```azurecli
az container exec --resource-group myResourceGroup --name myiis --exec-command "cmd.exe"
```

```output
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.

C:\>dir
 Volume in drive C has no label.
 Volume Serial Number is 76E0-C852

 Directory of C:\

03/23/2018  09:13 PM    <DIR>          inetpub
11/20/2016  11:32 AM             1,894 License.txt
03/23/2018  09:13 PM    <DIR>          Program Files
07/16/2016  12:09 PM    <DIR>          Program Files (x86)
03/13/2018  08:50 PM           171,616 ServiceMonitor.exe
03/23/2018  09:13 PM    <DIR>          Users
03/23/2018  09:12 PM    <DIR>          var
03/23/2018  09:22 PM    <DIR>          Windows
               2 File(s)        173,510 bytes
               6 Dir(s)  21,171,609,600 bytes free

C:\>exit
Bye.
```

## <a name="multi-container-groups"></a>Многоконтейнерные группы

Если ваша [группа контейнеров](container-instances-container-groups.md) содержит несколько контейнеров, например контейнер приложения и расширение ведения журнала, укажите имя контейнера, в котором должна выполняться команда, с помощью `--container-name`.

Например, в группе контейнеров *mynginx* два контейнера: *nginx-app* и *logger*. Чтобы запустить оболочку в контейнере *nginx-app*:

```azurecli
az container exec --resource-group myResourceGroup --name mynginx --container-name nginx-app --exec-command "/bin/bash"
```

## <a name="restrictions"></a>Ограничения

Служба "Экземпляры контейнеров Azure" в настоящее время поддерживает запуск одного процесса с помощью команды [az container exec][az-container-exec]. При этом передать аргументы команды невозможно. Например, нельзя добавить команды, как в случае с `sh -c "echo FOO && echo BAR"`, или выполнить `echo FOO`.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о других средствах устранения неполадок и часто встречающихся проблемах развертывания см. в статье [Устранение неполадок развертывания с помощью службы "Экземпляры контейнеров Azure"](container-instances-troubleshooting.md).

<!-- LINKS - internal -->
[az-container-create]: /cli/azure/container#az-container-create
[az-container-exec]: /cli/azure/container#az-container-exec
[azure-cli]: /cli/azure