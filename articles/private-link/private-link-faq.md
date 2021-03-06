---
title: Часто задаваемые вопросы о Приватном канале Azure
description: Сведения о частной ссылке Azure.
services: private-link
author: malopMSFT
ms.service: private-link
ms.topic: conceptual
ms.date: 10/05/2019
ms.author: allensu
ms.openlocfilehash: d06e90a691389b99d8f439364203b921f49b2305
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103496479"
---
# <a name="azure-private-link-frequently-asked-questions-faq"></a>Часто задаваемые вопросы о Приватном канале Azure

## <a name="private-link"></a>Приватный канал

### <a name="what-is-azure-private-endpoint-and-azure-private-link-service"></a>Что такое частная конечная точка Azure и служба частной связи Azure?

- **[Частная конечная точка Azure](private-endpoint-overview.md)**. Частная конечная точка Azure — это сетевой интерфейс, который обеспечивает безопасное и надежное подключение к службе с помощью частной связи Azure. Частные конечные точки можно использовать для подключения к службе PaaS Azure, которая поддерживает закрытую ссылку или собственную службу частной связи.
- Служба **[частной связи Azure](private-link-service-overview.md)**: служба частной связи Azure — это служба, созданная поставщиком услуг. В настоящее время служба частной связи может быть подключена к интерфейсной IP-конфигурации Load Balancer (цен. категория "Стандартный"). 

### <a name="how-is-traffic-being-sent-when-using-private-link"></a>Как отправляется трафик при использовании частной ссылки?
Трафик отправляется частным образом с помощью магистрали Майкрософт. Он не проходит через Интернет. Частная ссылка Azure не хранит данные клиента.
 
### <a name="what-is-the-difference-between-service-endpoints-and-private-endpoints"></a>В чем разница между конечными точками службы и частными конечными точками?
- Частные конечные точки предоставляют сетевой доступ к определенным ресурсам за заданной службой, обеспечивая детальное сегментирование. Трафик может получить доступ к ресурсу службы из локальной среды без использования общедоступных конечных точек.
- Конечная точка службы остается общедоступным направляемым IP-адресом.  Частная конечная точка — это частный IP-адрес в пространстве адресов виртуальной сети, в которой настроена частная конечная точка.

### <a name="what-is-the-relationship-between-private-link-service-and-private-endpoint"></a>Какова связь между службой частной связи и частной конечной точкой?
Несколько типов ресурсов частной ссылки поддерживают доступ через закрытую конечную точку. Ресурсы включают службы Azure PaaS и собственную службу частной связи. Это связь «один ко многим». 

Служба частной связи получает подключения из нескольких частных конечных точек. Частная конечная точка подключается к одной службе частной связи.    

### <a name="do-i-need-to-disable-network-policies-for-private-link"></a>Нужно ли отключить сетевые политики для закрытой ссылки
Да. Как частная конечная точка, так и служба частной связи должны отключить сетевые политики для правильной работы. У них есть свойства, не зависящие друг от друга.

## <a name="private-endpoint"></a>Частная конечная точка 
 
### <a name="can-i-create-multiple-private-endpoints-in-same-vnet-can-they-connect-to-different-services"></a>Можно ли создать несколько частных конечных точек в одной виртуальной сети? Могут ли они подключаться к разным службам? 
Да. В одной виртуальной сети или подсетях может быть несколько частных конечных точек. Они могут подключаться к разным службам.  
 
### <a name="do-i-require-a-dedicated-subnet-for-private-endpoints"></a>Требуется ли выделенная подсеть для частных конечных точек? 
Нет. Выделенная подсеть для частных конечных точек не требуется. Вы можете выбрать IP-адрес частной конечной точки из любой подсети в виртуальной сети, в которой развернута служба.  
 
### <a name="can-a-private-endpoint-connect-to-private-link-services-across-azure-active-directory-tenants"></a>Может ли частная конечная точка подключаться к службам частной связи между клиентами Azure Active Directory? 
Да. Частные конечные точки могут подключаться к службам Private Link или Azure PaaS через клиентов Azure Active Directory. Частные конечные точки, подключающиеся по клиентам, должны утверждать запрос вручную. 
 
### <a name="can-private-endpoint-connect-to-azure-paas-resources-across-azure-regions"></a>Может ли частная конечная точка подключаться к ресурсам Azure PaaS в регионах Azure?
Да. Частные конечные точки могут подключаться к ресурсам Azure PaaS в регионах Azure.

### <a name="can-i-modify-my-private-endpoint-network-interface-nic-"></a>Можно ли изменить сетевой интерфейс (NIC) частной конечной точки?
При создании частной конечной точки назначается сетевая карта, доступная только для чтения. Этот параметр не может быть изменен и останется в течение жизненного цикла закрытой конечной точки.

### <a name="how-do-i-achieve-availability-while-using-private-endpoints-in-case-of-regional-failures-"></a>Разделы справки достичь доступности при использовании частных конечных точек в случае региональных сбоев?

Частные конечные точки — это высокодоступные ресурсы с соглашением об уровне обслуживания 99,99% (соглашение об [уровне обслуживания для частного канала Azure)](https://azure.microsoft.com/support/legal/sla/private-link/v1_0/) Однако, поскольку они являются региональными ресурсами, любой сбой в регионе Azure может повлиять на доступность. Для обеспечения доступности в случае региональных сбоев несколько PEs, подключенных к одному целевому ресурсу, могут быть развернуты в разных регионах. Таким образом, если один из регионов выходит из строя, вы по-прежнему можете маршрутизировать трафик для сценариев восстановления через PE в другом регионе, чтобы получить доступ к целевому ресурсу. Сведения о том, как региональные сбои обрабатываются на стороне назначения службы, см. в документации по службе при отработке отказа и восстановлении. Трафик частного канала соответствует разрешению Azure DNS для конечной точки назначения. 


## <a name="private-link-service"></a>Служба "Приватный канал"
 
### <a name="what-are-the-pre-requisites-for-creating-a-private-link-service"></a>Каковы предварительные требования для создания службы частной связи? 
Она должна находиться в виртуальной сети и находиться за Load Balancer (цен. категория "Стандартный").
 
### <a name="how-can-i-scale-my-private-link-service"></a>Как можно масштабировать службу частной связи? 
Вы можете масштабировать службу частной связи несколькими способами: 
- Добавление серверных виртуальных машин в пул за Load Balancer (цен. категория "Стандартный") 
- Добавьте IP-адрес в службу частной связи. Мы допуским до 8 IP-адресов на службу частных ссылок.  
- Добавьте новую службу закрытых ссылок в Load Balancer (цен. категория "Стандартный"). Мы допуским до восьми служб частной связи на подсистему балансировки нагрузки.   

### <a name="what-is-natnetwork-address-translation-ip-configuration-used-in-private-link-service-how-can-i-scale-in-terms-of-available-ports-and-connections"></a>Конфигурация IP-адресов NAT (преобразование сетевых адресов), используемая в службе частной связи Как можно масштабироваться с точки зрения доступных портов и подключений? 

Конфигурация IP NAT обеспечивает отсутствие конфликта IP-адресов между исходным (на стороне потребителя) и адресным пространством (поставщиком услуг), предоставляя исходный NAT для трафика частной связи на стороне назначения (на стороне поставщика услуг). IP-адрес NAT будет отображаться в качестве исходного IP-адреса для всех пакетов, полученных службой, и IP-адресов назначения для всех пакетов, отправляемых службой.  IP-адрес NAT можно выбрать из любой подсети в виртуальной сети поставщика услуг. 

Каждый IP-адрес NAT обеспечивает подключение по ПРОТОКОЛу 64 к виртуальным машинам на виртуальной машине за пределами Load Balancer (цен. категория "Стандартный"). Чтобы масштабировать и добавлять подключения, можно добавить новые IP-адреса NAT или добавить дополнительные виртуальные машины за Load Balancer (цен. категория "Стандартный"). Это позволит масштабировать доступность порта и разрешить больше подключений. Подключения будут распределяться по IP-адресам NAT и виртуальным машинам за Load Balancer (цен. категория "Стандартный").

### <a name="can-i-connect-my-service-to-multiple-private-endpoints"></a>Можно ли подключить службу к нескольким частным конечным точкам?
Да. Одна служба частной связи может принимать подключения из нескольких частных конечных точек. Однако одна частная конечная точка может подключаться только к одной частной службе ссылок.  
 
### <a name="how-should-i-control-the-exposure-of-my-private-link-service"></a>Как управлять раскрытием службы частной связи?
Вы можете контролировать экспозицию с помощью конфигурации видимости в службе частной связи. Видимость поддерживает три параметра:

- **Нет** — подписки с доступом к Azure RBAC могут быть обнаружены только для тех подписок. 
-  Для доступа к службе можно использовать только утвержденные подписки с доступом к Azure RBAC. 
- **ALL — все** пользователи могут размещать службу. 
 
### <a name="can-i-create-a-private-link-service-with-basic-load-balancer"></a>Можно ли создать службу частной связи с базовой подсистемой балансировки нагрузки? 
Нет. Служба закрытых ссылок через базовую подсистему балансировки нагрузки не поддерживается.
 
### <a name="is-a-dedicated-subnet-required-for-private-link-service"></a>Требуется ли выделенная подсеть для службы частной связи? 
Нет. Для службы частной связи не требуется выделенная подсеть. Вы можете выбрать любую подсеть в виртуальной сети, в которой развернута служба.   

### <a name="im-a-service-provider-using-azure-private-link-do-i-need-to-make-sure-all-my-customers-have-unique-ip-space-and-dont-overlap-with-my-ip-space"></a>Я являюсь поставщиком услуг, используя частную ссылку Azure. Нужно ли, чтобы все мои клиенты имели уникальные IP-пространство и не перекрывались с моим IP-адресом? 
Нет. Частная ссылка Azure предоставляет эту функцию. Не обязательно включать в адресное пространство клиента неперекрывающиеся адресные пространства. 

##  <a name="next-steps"></a>Следующие шаги

- Подробнее о [Приватном канале Azure](private-link-overview.md)
