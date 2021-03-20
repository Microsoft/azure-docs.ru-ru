---
title: Azure NetApp Files тесты производительности для Linux | Документация Майкрософт
description: Описывает тесты производительности Azure NetApp Files для Linux.
services: azure-netapp-files
documentationcenter: ''
author: b-juche
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 04/29/2020
ms.author: b-juche
ms.openlocfilehash: b763a734866dd5fed5bf0500d4d52b9324c92a79
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "82614594"
---
# <a name="azure-netapp-files-performance-benchmarks-for-linux"></a>Azure NetApp Files тесты производительности для Linux

В этой статье описываются тесты производительности Azure NetApp Files для Linux.

## <a name="linux-scale-out"></a>Горизонтальное масштабирование Linux

В этом разделе описываются показатели производительности рабочей нагрузки Linux и операций ввода-вывода в секунду.

### <a name="linux-workload-throughput"></a>Пропускная способность рабочей нагрузки Linux  

Приведенный ниже граф представляет последовательную рабочую нагрузку 64-кибибите (КИБ) и 1-Тиб рабочий набор. Он показывает, что один Azure NetApp Files том может работать между последовательными операциями записи ~ 1 600 MiB/s, а также чистые последовательные операции чтения ~ 4 500 MiB/s.  

На диаграмме показано уменьшение в 10% за раз, начиная с чистого чтения до чистого записи. В нем показано, что можно рассчитывать при использовании различных соотношений чтения и записи (100%: 0%, 90%: 10%, 80%: 20% и т. д.).

![Пропускная способность рабочей нагрузки Linux](../media/azure-netapp-files/performance-benchmarks-linux-workload-throughput.png)  

### <a name="linux-workload-iops"></a>Рабочие нагрузки Linux, операции ввода-вывода  

На следующем графике представлена случайная Рабочая нагрузка 4-кибибите (КИБ) и 1-Тиб рабочий набор. На графике показано, что Azure NetApp Files том может работать между ~ 130 000 чистых случайных операций записи и ~ 460 000 чистых случайных чтений.  

На этой диаграмме показано уменьшение в 10% за раз, начиная с чистого чтения до чистого записи. В нем показано, что можно рассчитывать при использовании различных соотношений чтения и записи (100%: 0%, 90%: 10%, 80%: 20% и т. д.).

![Рабочие нагрузки Linux, операции ввода-вывода](../media/azure-netapp-files/performance-benchmarks-linux-workload-iops.png)  

## <a name="linux-scale-up"></a>Масштабирование Linux  

Ядро Linux 5,3 позволяет выполнять одноклиентское масштабирование сети для NFS `nconnect` . Графы в этом разделе показывают результаты проверочного тестирования для параметра подключения на стороне клиента с NFSv3. Эта функция доступна в SUSE (начиная с версии SLES12SP4) и Ubuntu (начиная с выпуска 19,10). Он похож на концепцию многоканального SMB и Oracle Direct NFS.

Диаграммы сравнивают преимущества с подключенным `nconnect` томом, не подключенным к сети. На графах FIO создал рабочую нагрузку из одного экземпляра D32s_v3 в регионе Azure US-west2.

### <a name="linux-read-throughput"></a>Пропускная способность чтения Linux  

На следующих диаграммах показаны последовательные операции чтения ~ 3 500 MiB/s с `nconnect` , примерно 2.3 x не- `nconnect` .

![Пропускная способность чтения Linux](../media/azure-netapp-files/performance-benchmarks-linux-read-throughput.png)  

### <a name="linux-write-throughput"></a>Пропускная способность записи в Linux  

На следующих диаграммах показаны последовательные операции записи. Они указывают, что `nconnect` не имеет заметного преимущества для последовательных операций записи. 1 500. MiB/s — это примерно верхний предел объема последовательного тома для записи и предельный объем исходящих экземпляров D32s_v3.

![Пропускная способность записи в Linux](../media/azure-netapp-files/performance-benchmarks-linux-write-throughput.png)  

### <a name="linux-read-iops"></a>Операции ввода-вывода для чтения Linux  

На следующих диаграммах показано случайное считывание ~ 200 000 операций ввода-вывода в секунду с помощью `nconnect` , примерно 3 раза, а не `nconnect` .

![Операции ввода-вывода для чтения Linux](../media/azure-netapp-files/performance-benchmarks-linux-read-iops.png)  

### <a name="linux-write-iops"></a>Записи ввода-вывода в Linux  

На следующих диаграммах показана случайная запись ~ 135 000 операций ввода-вывода в секунду с параметром `nconnect` , примерно 3 раза, а не `nconnect` .

![Записи ввода-вывода в Linux](../media/azure-netapp-files/performance-benchmarks-linux-write-iops.png)  

## <a name="next-steps"></a>Дальнейшие действия

- [Azure NetApp Files: максимально эффективное получение из облачного хранилища](https://cloud.netapp.com/hubfs/Resources/ANF%20PERFORMANCE%20TESTING%20IN%20TEMPLATE.pdf?hsCtaTracking=f2f560e9-9d13-4814-852d-cfc9bf736c6a%7C764e9d9c-9e6b-4549-97ec-af930247f22f)
