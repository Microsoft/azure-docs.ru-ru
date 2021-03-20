---
title: Azure Service Fabric CLI — sfctl
description: Сведения о sfctl, интерфейсе командной строки Azure Service Fabric. Содержит список команд и подгрупп.
author: jeffj6123
ms.topic: reference
ms.date: 1/16/2020
ms.author: jejarry
ms.openlocfilehash: fc317345155a6807a20d342e2cefd0701b20f180
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "86257059"
---
# <a name="sfctl"></a>sfctl
Команды для управления кластерами и сущностями Service Fabric. Эта версия совместима со средой выполнения Service Fabric 7,0.

В командах используется шаблон "существительное — глагол". Дополнительные сведения см. в подгруппах.

## <a name="subgroups"></a>Подгруппы
|Подгруппа|Описание|
| --- | --- |
| [приложение](service-fabric-sfctl-application.md) | Создание и удаление приложений и типов приложений, а также управление ими. |
| [chaos](service-fabric-sfctl-chaos.md) | Запуск и остановка службы хаотического тестирования, а также вывод отчета о ее работе. |
| [кластеризован](service-fabric-sfctl-cluster.md) | Выбор кластеров Service Fabric, управление ими и их эксплуатация. |
| [создания](service-fabric-sfctl-compose.md) | Создание, удаление приложений Docker Compose и управление ими. |
| [container](service-fabric-sfctl-container.md) | Выполнение связанных с контейнером команд в узле кластера. |
| [events](service-fabric-sfctl-events.md) | Получение событий из хранилища событий (если служба Евентсторе уже установлена). |
| [is](service-fabric-sfctl-is.md) | Запрос службы инфраструктуры и отправка команд для нее. |
| [mesh](service-fabric-sfctl-mesh.md) | Управление приложениями в службе "Сетка Service Fabric" и их удаление. |
| [узел](service-fabric-sfctl-node.md) | Управление узлами, которые образуют кластер. |
| [диска](service-fabric-sfctl-partition.md) | Запрос секций для любой службы и управление ими. |
| [property](service-fabric-sfctl-property.md) | Хранение и запрос свойств с именами Service Fabric. |
| [член](service-fabric-sfctl-replica.md) | Управление репликами, принадлежащими секциям службы. |
| [15000](service-fabric-sfctl-rpm.md) | Запрос службы диспетчера восстановления и отправка команд для нее. |
| [sa-cluster](service-fabric-sfctl-sa-cluster.md) | Управление автономными кластерами Service Fabric. |
| [служеб](service-fabric-sfctl-service.md) | Создание и удаление служб, типов служб и пакетов службы, а также управление ими. |
| [параметры](service-fabric-sfctl-settings.md) | Настройка локальных параметров этого экземпляра sfctl. |
| [хранение](service-fabric-sfctl-store.md) | Выполнение базовых операций на уровне файла с хранилищем образов кластера. |

## <a name="next-steps"></a>Дальнейшие действия
- [Настройте](service-fabric-cli.md) Service Fabric CLI.
- Узнайте, как использовать интерфейс командной строки Service Fabric, с помощью [примеров сценариев](./scripts/sfctl-upgrade-application.md).
