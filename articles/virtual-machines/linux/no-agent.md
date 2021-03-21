---
title: Создание образов Linux без агента подготовки
description: Создавайте обобщенные образы Linux без агента подготовки в Azure.
author: danielsollondon
ms.service: virtual-machines
ms.subservice: imaging
ms.collection: linux
ms.topic: how-to
ms.workload: infrastructure
ms.date: 09/01/2020
ms.author: danis
ms.reviewer: cynthn
ms.openlocfilehash: c7ca147f0a5b907ee0c5c66d53a219fe75ab2179
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102551714"
---
# <a name="creating-generalized-images-without-a-provisioning-agent"></a>Создание обобщенных образов без агента подготовки

Microsoft Azure предоставляет агенты подготовки для виртуальных машин Linux в форме [walinuxagent](https://github.com/Azure/WALinuxAgent) или [Cloud-init](https://github.com/canonical/cloud-init) (рекомендуется). Но может существовать ситуация, когда вы не хотите использовать ни одно из этих приложений для агента подготовки, например:

- Дистрибутив или версия Linux не поддерживает агент Cloud-init/Linux.
- Необходимо задать определенные свойства виртуальной машины, например имя узла.

> [!NOTE] 
>
> Если не требуется задавать какие-либо свойства или выполнять подготовку, следует подумать о создании специализированного образа.

В этой статье показано, как настроить образ виртуальной машины для удовлетворения требований к платформе Azure и задать имя узла без установки агента подготовки.

## <a name="networking-and-reporting-ready"></a>Подготовка к работе в сети и отчеты

Чтобы виртуальная машина Linux получала связь с компонентами Azure, вам потребуется DHCP-клиент для получения IP-адреса узла из виртуальной сети, а также разрешения DNS и управления маршрутами. Большинство дистрибутивов поставляются с этими служебными программами готовыми. Средства, которые были протестированы в Azure дистрибутив поставщиками Linux, включают `dhclient` в себя, `network-manager` `systemd-networkd` и другие.

> [!NOTE]
>
> Сейчас создание обобщенных образов без агента подготовки поддерживает только виртуальные машины с поддержкой DHCP.

После настройки и настройки сети необходимо «подготовить отчет». Это сообщит Azure о том, что виртуальная машина успешно подготовлена.

> [!IMPORTANT]
>
> Если не сообщить о готовности в Azure, виртуальная машина будет перезагружена.

## <a name="demosample"></a>Демонстрация и выборка

В этой демонстрации показано, как можно взять существующий образ Marketplace (в данном случае это виртуальная машина Debian бустер) и удалить агент Linux (walinuxagent), но также создать самый простой процесс для передачи отчетов в Azure о том, что виртуальная машина готова.

### <a name="create-the-resource-group-and-base-vm"></a>Создайте группу ресурсов и базовую виртуальную машину.

```bash
$ az group create --location eastus --name demo1
```

Создайте базовую виртуальную машину.

```bash
$ az vm create \
    --resource-group demo1 \
    --name demo1 \
    --location eastus \
    --ssh-key-value <ssh_pub_key_path> \
    --public-ip-address-dns-name demo1 \
    --image "debian:debian-10:10:latest"
```

### <a name="remove-the-image-provisioning-agent"></a>Удаление агента подготовки образа

После подготовки виртуальной машины вы можете с помощью SSH и удалить агент Linux:

```bash
$ sudo apt purge -y waagent
$ sudo rm -rf /var/lib/waagent /etc/waagent.conf /var/log/waagent.log
```

### <a name="add-required-code-to-the-vm"></a>Добавление требуемого кода в виртуальную машину

Кроме того, в виртуальной машине, так как мы удалили агент Linux для Azure, необходимо предоставить механизм для подготовки отчетов. 

#### <a name="python-script"></a>Сценарий Python

```python
import http.client
import sys
from xml.etree import ElementTree

wireserver_ip = '168.63.129.16'
wireserver_conn = http.client.HTTPConnection(wireserver_ip)

print('Retrieving goal state from the Wireserver')
wireserver_conn.request(
    'GET',
    '/machine?comp=goalstate',
    headers={'x-ms-version': '2012-11-30'}
)

resp = wireserver_conn.getresponse()

if resp.status != 200:
    print('Unable to connect with wireserver')
    sys.exit(1)

wireserver_goalstate = resp.read().decode('utf-8')

xml_el = ElementTree.fromstring(wireserver_goalstate)

container_id = xml_el.findtext('Container/ContainerId')
instance_id = xml_el.findtext('Container/RoleInstanceList/RoleInstance/InstanceId')
print(f'ContainerId: {container_id}')
print(f'InstanceId: {instance_id}')

# Construct the XML response we need to send to Wireserver to report ready.
health = ElementTree.Element('Health')
goalstate_incarnation = ElementTree.SubElement(health, 'GoalStateIncarnation')
goalstate_incarnation.text = '1'
container = ElementTree.SubElement(health, 'Container')
container_id_el = ElementTree.SubElement(container, 'ContainerId')
container_id_el.text = container_id
role_instance_list = ElementTree.SubElement(container, 'RoleInstanceList')
role = ElementTree.SubElement(role_instance_list, 'Role')
instance_id_el = ElementTree.SubElement(role, 'InstanceId')
instance_id_el.text = instance_id
health_second = ElementTree.SubElement(role, 'Health')
state = ElementTree.SubElement(health_second, 'State')
state.text = 'Ready'

out_xml = ElementTree.tostring(
    health,
    encoding='unicode',
    method='xml'
)
print('Sending the following data to Wireserver:')
print(out_xml)

wireserver_conn.request(
    'POST',
    '/machine?comp=health',
    headers={
        'x-ms-version': '2012-11-30',
        'Content-Type': 'text/xml;charset=utf-8',
        'x-ms-agent-name': 'custom-provisioning'
    },
    body=out_xml
)

resp = wireserver_conn.getresponse()
print(f'Response: {resp.status} {resp.reason}')

wireserver_conn.close()
```

#### <a name="generic-steps-without-using-python"></a>Универсальные шаги (без использования Python)

Если на виртуальной машине нет установленного или доступного Python, можно программно воспроизвести эту логику скрипта, выполнив следующие действия:

1. Извлеките `ContainerId` и `InstanceId` , анализируя ответ от виресервер: `curl -X GET -H 'x-ms-version: 2012-11-30' http://168.63.129.16/machine?comp=goalstate` .

2. Создайте следующие XML-данные, подставляемые с помощью синтаксического анализа `ContainerId` и указанного `InstanceId` выше шага.
   ```xml
   <Health>
     <GoalStateIncarnation>1</GoalStateIncarnation>
     <Container>
       <ContainerId>CONTAINER_ID</ContainerId>
       <RoleInstanceList>
         <Role>
           <InstanceId>INSTANCE_ID</InstanceId>
           <Health>
             <State>Ready</State>
           </Health>
         </Role>
       </RoleInstanceList>
     </Container>
   </Health>
   ```

3. Опубликовать эти данные в Виресервер: `curl -X POST -H 'x-ms-version: 2012-11-30' -H "x-ms-agent-name: WALinuxAgent" -H "Content-Type: text/xml;charset=utf-8" -d "$REPORT_READY_XML" http://168.63.129.16/machine?comp=health`

### <a name="automating-running-the-code-at-first-boot"></a>Автоматизация выполнения кода при первой загрузке

В этой демонстрации используется система, которая является наиболее распространенной системой инициализации в современных Linux дистрибутивов. Таким образом, самый простой и эффективный способ гарантировать, что этот механизм готовности отчетов будет выполняться в нужное время, — это создать системную единицу службы. В этот пример можно добавить следующий файл единицы `/etc/systemd/system` (например, имя файла единицы измерения `azure-provisioning.service` ):

```bash
[Unit]
Description=Azure Provisioning

[Service]
Type=oneshot
ExecStart=/usr/bin/python3 /usr/local/azure-provisioning.py
ExecStart=/bin/bash -c "hostnamectl set-hostname $(curl \
    -H 'metadata: true' \
    'http://169.254.169.254/metadata/instance/compute/name?api-version=2019-06-01&format=text')"
ExecStart=/usr/bin/systemctl disable azure-provisioning.service

[Install]
WantedBy=multi-user.target
```

Эта системная служба выполняет три вещи для базовой подготовки:

1. Отчеты, готовые к работе в Azure (чтобы указать, что он был успешно создан).
1. Переименовывает виртуальную машину на основе имени предоставленной пользователем виртуальной машины, изменяя эти данные из [службы метаданных экземпляра Azure (IMDS)](./instance-metadata-service.md). **Примечание** . IMDS также предоставляет другие [метаданные экземпляра](./instance-metadata-service.md#access-azure-instance-metadata-service), такие как открытые ключи SSH, поэтому вы можете задать больше имени узла.
1. Отключается, чтобы он выполнялся только при первой загрузке, а не при последующих перезагрузках.

С помощью единицы в файловой системе выполните следующую команду, чтобы включить ее:

```bash
$ sudo systemctl enable azure-provisioning.service
```

Теперь виртуальная машина готова к обобщением и создана на основе образа. 

#### <a name="completing-the-preparation-of-the-image"></a>Завершение подготовки образа

Вернитесь на компьютер разработки и выполните следующую команду, чтобы подготовить образ к созданию образа из базовой виртуальной машины:

```bash
$ az vm deallocate --resource-group demo1 --name demo1
$ az vm generalize --resource-group demo1 --name demo1
```

Создайте образ на основе этой виртуальной машины:

```bash
$ az image create \
    --resource-group demo1 \
    --source demo1 \
    --location eastus \
    --name demo1img
```

Теперь мы готовы создать виртуальную машину (или несколько виртуальных машин) из образа.

```bash
$ IMAGE_ID=$(az image show -g demo1 -n demo1img --query id -o tsv)
$ az vm create \
    --resource-group demo12 \
    --name demo12 \
    --location eastus \
    --ssh-key-value <ssh_pub_key_path> \
    --public-ip-address-dns-name demo12 \
    --image "$IMAGE_ID" 
    --enable-agent false
```

> [!NOTE]
>
> Важно задать значение `--enable-agent` , `false` так как walinuxagent не существует на этой виртуальной машине, которая будет создана из образа.

Эта виртуальная машина должна быть успешно подготовлена. Войдя на новую виртуальную машину подготовки, вы сможете увидеть выходные данные готовой системной службы Reporting Service:

```bash
$ sudo journalctl -u azure-provisioning.service
-- Logs begin at Thu 2020-06-11 20:28:45 UTC, end at Thu 2020-06-11 20:31:24 UTC. --
Jun 11 20:28:49 thstringnopa systemd[1]: Starting Azure Provisioning...
Jun 11 20:28:54 thstringnopa python3[320]: Retrieving goal state from the Wireserver
Jun 11 20:28:54 thstringnopa python3[320]: ContainerId: 7b324f53-983a-43bc-b919-1775d6077608
Jun 11 20:28:54 thstringnopa python3[320]: InstanceId: fbb84507-46cd-4f4e-bd78-a2edaa9d059b._thstringnopa2
Jun 11 20:28:54 thstringnopa python3[320]: Sending the following data to Wireserver:
Jun 11 20:28:54 thstringnopa python3[320]: <Health><GoalStateIncarnation>1</GoalStateIncarnation><Container><ContainerId>7b324f53-983a-43bc-b919-1775d6077608</ContainerId><RoleInstanceList><Role><InstanceId>fbb84507-46cd-4f4e-bd78-a2edaa9d059b._thstringnopa2</InstanceId><Health><State>Ready</State></Health></Role></RoleInstanceList></Container></Health>
Jun 11 20:28:54 thstringnopa python3[320]: Response: 200 OK
Jun 11 20:28:56 thstringnopa bash[472]:   % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
Jun 11 20:28:56 thstringnopa bash[472]:                                  Dload  Upload   Total   Spent    Left  Speed
Jun 11 20:28:56 thstringnopa bash[472]: [158B blob data]
Jun 11 20:28:56 thstringnopa2 systemctl[475]: Removed /etc/systemd/system/multi-user.target.wants/azure-provisioning.service.
Jun 11 20:28:56 thstringnopa2 systemd[1]: azure-provisioning.service: Succeeded.
Jun 11 20:28:56 thstringnopa2 systemd[1]: Started Azure Provisioning.
```

## <a name="support"></a>Поддержка

Если вы реализуете собственный код подготовки или агента, вы владеете поддержкой этого кода, поддержка Майкрософт будет исследовать только проблемы, связанные с интерфейсами подготовки, которые недоступны. Мы постоянно делаем улучшения и изменения в этой области, поэтому необходимо отслеживать изменения в Cloud-init и агенте Linux для Azure для подготовки изменений API.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в разделе [Подготовка Linux](provisioning.md).
