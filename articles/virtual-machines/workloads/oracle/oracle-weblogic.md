---
title: Что такое решения для запуска сервера платформы Oracle на виртуальных машинах Azure
description: Узнайте, как запускать сервер Виртуальные машины Microsoft Azure Oracle.
author: rezar
ms.service: virtual-machines
ms.subservice: oracle
ms.collection: linux
ms.topic: article
ms.date: 03/23/2021
ms.author: rezar
ms.openlocfilehash: 59bae8bfea29ee458288751209ffc860b5b9ce9d
ms.sourcegitcommit: ac035293291c3d2962cee270b33fca3628432fac
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "104956009"
---
# <a name="what-are-solutions-for-running-oracle-weblogic-server-on-azure-virtual-machines"></a>Что такое решения для запуска Oracle WebLogic Server в Виртуальных машинах Azure?

На этой странице описаны решения для запуска сервера веб-логики Oracle (WLS) на виртуальных машинах Azure. Эти решения совместно разработаны и поддерживаются Oracle и корпорацией Майкрософт.

Вы также можете запустить WLS в службе Azure Kubernetes. Решения для этого описаны в [этой статье Майкрософт](./weblogic-aks.md).

WLS — это ведущий сервер приложений Java, на котором выполняются некоторые наиболее критически важные корпоративные приложения Java на протяжении всего мира. WLS формирует промежуточную основу для программного пакета Oracle. Oracle и корпорация Майкрософт стремится предоставить клиентам WLS возможность выбора и гибкости при выполнении рабочих нагрузок в Azure в качестве ведущей облачной платформы.

Решения Azure WLS предназначены для упрощения переноса приложений Java на виртуальные машины Azure. Эти решения делают это путем создания развернутых ресурсов для наиболее распространенных сценариев подготовки облака. Решения автоматически подготавливают ресурсы виртуальной сети, хранилища, Java, WLS и Linux. При минимальных усилиях устанавливается сервер Logic. Решения могут настроить безопасность с помощью группы безопасности сети, балансировки нагрузки с помощью шлюза приложений Azure, проверки подлинности Azure Active Directory, централизованного ведения журнала с использованием ELK и распределенного кэширования с согласованием Oracle. Вы также можете автоматически подключиться к существующей базе данных, включая Azure PostgreSQL, Azure SQL и Oracle DB в облаке Oracle или Azure. 

:::image type="content" source="media/oracle-weblogic/wls-on-azure.gif" alt-text="Вы можете использовать портал Azure для развертывания сервера серверной логической логики в Azure.":::

Существует четыре предложения для удовлетворения различных сценариев: [один узел без сервера администрирования](https://portal.azure.com/#create/oracle.20191001-arm-oraclelinux-wls20191001-arm-oraclelinux-wls), [один узел с сервером администрирования](https://portal.azure.com/#create/oracle.20191009-arm-oraclelinux-wls-admin20191009-arm-oraclelinux-wls-admin), [кластером](https://portal.azure.com/#create/oracle.20191007-arm-oraclelinux-wls-cluster20191007-arm-oraclelinux-wls-cluster)и [динамическим кластером](https://portal.azure.com/#create/oracle.20191021-arm-oraclelinux-wls-dynamic-cluster20191021-arm-oraclelinux-wls-dynamic-cluster). Предложения предоставляются бесплатно. Эти предложения описаны и связаны ниже.

_Все предлагаемые решения подразумевают использование собственных лицензий (BYOL)_ . Предполагается, что у вас уже есть соответствующие лицензии на Oracle и они должным образом лицензированы для запуска предложений в Azure.

Предложения поддерживают диапазон версий операционной системы, Java и WLS с помощью базовых образов (например, с сервера JDK версии 10 и 11 на Oracle Linux 7,6). Эти базовые образы также доступны в Azure самостоятельно. Базовые образы подходят для клиентов, которым требуются сложные, настроенные развертывания Azure. Текущий набор базовых образов доступен [здесь](https://azuremarketplace.microsoft.com/marketplace/apps?search=WebLogic%20Server%20Base%20Image&page=1).

_Если вы заинтересованы в тесном взаимодействии с сценариями миграции с группой разработки этих предложений, нажмите кнопку " [связаться со мной](https://azuremarketplace.microsoft.com/marketplace/apps/oracle.oraclelinux-wls-cluster?tab=Overview) "_ на [странице "Обзор" предложения Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/oracle.oraclelinux-wls-cluster?tab=Overview). Руководитель программы, архитекторы и инженеры будут в ближайшее время вернуться к работе и приступить к закрытию совместной работы. Возможность совместной работы над сценарием миграции предоставляется бесплатно, пока предложения находятся под активной разработкой.

## <a name="oracle-weblogic-server-single-node"></a>Один узел Oracle WebLogic Server

[Это предложение](https://portal.azure.com/#create/oracle.20191001-arm-oraclelinux-wls20191001-arm-oraclelinux-wls) подготавливает одну виртуальную машину и устанавливает в ней WLS. Он не создает домен или не запускает сервер администрирования. Предложение для одного узла удобно использовать для сценариев с сильно настроенной конфигурацией домена.

## <a name="oracle-weblogic-server-with-admin-server"></a>Oracle WebLogic Server с сервером администрирования

[Это предложение](https://portal.azure.com/#create/oracle.20191009-arm-oraclelinux-wls-admin20191009-arm-oraclelinux-wls-admin) подготавливает одну виртуальную машину и устанавливает в ней WLS. Он создает домен и запускает сервер администрирования. Вы можете управлять доменом и начать работу с развертываниями приложений сразу же.

## <a name="oracle-weblogic-server-cluster"></a>Кластер Oracle WebLogic Server

[Это предложение](https://portal.azure.com/#create/oracle.20191007-arm-oraclelinux-wls-cluster20191007-arm-oraclelinux-wls-cluster) создает высокодоступный кластер ВИРТУАЛЬНЫХ машин WLS. Сервер администрирования и все управляемые серверы запускаются по умолчанию. Вы можете управлять кластером и начать работу с высокодоступными приложениями прямо сейчас.

## <a name="oracle-weblogic-server-dynamic-cluster"></a>Динамический кластер Oracle WebLogic Server

[Это предложение](https://portal.azure.com/#create/oracle.20191021-arm-oraclelinux-wls-dynamic-cluster20191021-arm-oraclelinux-wls-dynamic-cluster) создает высокодоступный и масштабируемый динамический кластер ВИРТУАЛЬНЫХ машин WLS. Сервер администрирования и все управляемые серверы запускаются по умолчанию.

Решения позволяют использовать широкий спектр готовых к производству архитектур развертывания с относительным удобством. Для большинства вариантов миграции можно использовать самый эффективный способ, позволяющий сосредоточиться на разработке бизнес-приложений.

:::image type="content" source="media/oracle-weblogic/weblogic-architecture-vms.png" alt-text="В Azure включены сложные развертывания сервера для серверной логической логики":::

Помимо тех, которые автоматически подготавливаются решениями, клиенты имеют полную гибкость в настройке своих развертываний. Скорее всего, на самом верху развертывания приложений клиенты будут интегрировать дополнительные ресурсы Azure с их развертываниями. Клиентам рекомендуется [подключаться к группе разработки](https://azuremarketplace.microsoft.com/marketplace/apps/oracle.oraclelinux-wls-cluster?tab=Overview) и предоставлять отзывы о дальнейших улучшениях решений.

## <a name="next-steps"></a>Дальнейшие действия

Изучите предложения в Azure.

> [!div class="nextstepaction"]
> [Один узел Oracle WebLogic Server](https://portal.azure.com/#create/oracle.20191001-arm-oraclelinux-wls20191001-arm-oraclelinux-wls)

> [!div class="nextstepaction"]
> [Oracle WebLogic Server с сервером администрирования](https://portal.azure.com/#create/oracle.20191009-arm-oraclelinux-wls-admin20191009-arm-oraclelinux-wls-admin)

> [!div class="nextstepaction"]
> [Кластер Oracle WebLogic Server](https://portal.azure.com/#create/oracle.20191007-arm-oraclelinux-wls-cluster20191007-arm-oraclelinux-wls-cluster)

> [!div class="nextstepaction"]
> [Динамический кластер Oracle WebLogic Server](https://portal.azure.com/#create/oracle.20191021-arm-oraclelinux-wls-dynamic-cluster20191021-arm-oraclelinux-wls-dynamic-cluster)
