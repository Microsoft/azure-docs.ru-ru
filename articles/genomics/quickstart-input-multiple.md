---
title: Отправка рабочего процесса с помощью нескольких входных данных
titleSuffix: Microsoft Genomics
description: В этой статье показано, как отправить рабочий процесс в службу Microsoft Genomics, если входной файл является несколькими файлами FASTQ-или BAM из одного и того же примера.
services: genomics
ms.service: genomics
author: grhuynh
manager: cgronlund
ms.author: grhuynh
ms.topic: conceptual
ms.date: 02/05/2018
ms.openlocfilehash: b426015906a8e17674123c0c3ad2fccb9c43798f
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "72248567"
---
# <a name="submit-a-workflow-using-multiple-inputs-from-the-same-sample"></a>Отправка рабочего процесса с помощью нескольких входных данных из одного примера

В этой статье показано, как отправить рабочий процесс в службу Microsoft Genomics, если входной файл является несколькими файлами FASTQ-или BAM **, поступающими из одного и того же примера**. Например, если вы запустили **тот же пример** в нескольких полосах в Sequencer, вы можете получить по паре файлов FASTQ для каждой полосы. Вместо объединения этих файлов FASTQ до выравнивания и определения вариантов, можно отправить все эти входные данные непосредственно в клиент `msgen`. Выходные данные клиента `msgen` будут представлены **одним набором** файлов, включая файлы с расширениями .bam, .bai и .vcf. 

Однако следует помнить, что **нельзя** смешивать FASTQ- и BAM-файлы в рамках одной отправки. Кроме того, вы **не можете** отправлять несколько файлов FASTQ или BAM от нескольких отдельных пользователей. 

В этой статье предполагается, что вы уже установили и запустили клиент `msgen` и знаете, как использовать службу хранилища Azure. Если вы успешно отправили рабочий процесс с помощью предоставленных примеров данных, можно приступать к работе с этой статьей. 


## <a name="multiple-bam-files"></a>Несколько BAM-файлов

### <a name="upload-your-input-files-to-azure-storage"></a>Передача входных файлов в службу хранилища Azure
Предположим, что у вас есть несколько BAM-файлов в качестве входных данных — *reads.bam*, *additional_reads.bam* и *yet_more_reads.bam* — и вы передали их в свою учетную запись хранения *myaccount* в Azure. У вас есть URL-адрес API и ключ доступа. Вы хотите разместить выходные данные на странице **https://<span></span>myaccount.blob.core <span></span>.windows <span></span>.net <span></span>/outputs <span></span>**.


### <a name="submit-your-job-to-the-msgen-client"></a>Отправка задания в клиент `msgen` 

Вы можете отправить несколько BAM-файлов, передав все их имена в аргумент --input-blob-name-1. Обратите внимание, что все файлы должны быть из одного примера, но их порядок не имеет значения. В следующих разделах представлены примеры отправки данных из командной строки в Windows и Unix, а также с помощью файла конфигурации. Разрывы строк добавлены для ясности.


Для Windows:

```
msgen submit ^
  --api-url-base <Genomics API URL> ^
  --access-key <Genomics access key> ^
  --process-args R=b37m1 ^
  --input-storage-account-name myaccount ^
  --input-storage-account-key <storage access key to "myaccount"> ^
  --input-storage-account-container inputs ^
  --input-blob-name-1 reads.bam additional_reads.bam yet_more_reads.bam ^
  --output-storage-account-name myaccount ^
  --output-storage-account-key <storage access key to "myaccount"> ^
  --output-storage-account-container outputs
```


Для Unix:

```
msgen submit \
  --api-url-base <Genomics API URL> \
  --access-key <Genomics access key> \
  --process-args R=b37m1 \
  --input-storage-account-name myaccount \
  --input-storage-account-key <storage access key to "myaccount"> \
  --input-storage-account-container inputs \
  --input-blob-name-1 reads.bam additional_reads.bam yet_more_reads.bam \
  --output-storage-account-name myaccount \
  --output-storage-account-key <storage access key to "myaccount"> \
  --output-storage-account-container outputs
```


Если вы предпочитаете использовать файл конфигурации, он должен содержать следующие строки:

``` config.txt
api_url_base:                     <Genomics API URL>
access_key:                       <Genomics access key>
process_args:                     R=b37m1
input_storage_account_name:       myaccount
input_storage_account_key:        <storage access key to "myaccount">
input_storage_account_container:  inputs
input_blob_name_1:                reads.bam additional_reads.bam yet_more_reads.bam
output_storage_account_name:      myaccount
output_storage_account_key:       <storage access key to "myaccount">
output_storage_account_container: outputs
```

Отправьте файл `config.txt` с помощью этого вызова: `msgen submit -f config.txt`


## <a name="multiple-paired-fastq-files"></a>Несколько пар FASTQ-файлов

### <a name="upload-your-input-files-to-azure-storage"></a>Передача входных файлов в службу хранилища Azure
Предположим, что у вас есть несколько пар FASTQ-файлов в качестве входных данных — *reads_1.fq.gz* и *reads_2.fq.gz*, *additional_reads_1.fq.gz* и *additional_reads_2.fq.gz*, *yet_more_reads_1.fq.gz* и *yet_more_reads_2.fq.gz*. Вы отправили их в свою учетную запись хранения *myaccount* в Azure, и у вас есть URL-адрес API и ключ доступа. Вы хотите разместить выходные данные на странице **https://<span></span>myaccount.blob.core <span></span>.windows <span></span>.net <span></span>/outputs <span></span>**.


### <a name="submit-your-job-to-the-msgen-client"></a>Отправка задания в клиент `msgen` 

Пары FASTQ-файлов должны не только быть взяты из одного примера, но и обрабатываться вместе.  Порядок имен файлов имеет значение при передаче их в качестве аргументов в --input-blob-name-1 и --input-blob-name-2. 

В следующих разделах представлены примеры отправки данных из командной строки в Windows и Unix, а также с помощью файла конфигурации. Разрывы строк добавлены для ясности.


Для Windows:

```
msgen submit ^
  --api-url-base <Genomics API URL> ^
  --access-key <Genomics access key> ^
  --process-args R=b37m1 ^
  --input-storage-account-name myaccount ^
  --input-storage-account-key <storage access key to "myaccount"> ^
  --input-storage-account-container inputs ^
  --input-blob-name-1 reads_1.fastq.gz additional_reads_1.fastq.gz yet_more_reads_1.fastq.gz ^
  --input-blob-name-2 reads_2.fastq.gz additional_reads_2.fastq.gz yet_more_reads_2.fastq.gz ^
  --output-storage-account-name myaccount ^
  --output-storage-account-key <storage access key to "myaccount"> ^
  --output-storage-account-container outputs
```


Для Unix:

```
msgen submit \
  --api-url-base <Genomics API URL> \
  --access-key <Genomics access key> \
  --process-args R=b37m1 \
  --input-storage-account-name myaccount \
  --input-storage-account-key <storage access key to "myaccount"> \
  --input-storage-account-container inputs \
  --input-blob-name-1 reads_1.fastq.gz additional_reads_1.fastq.gz yet_more_reads_1.fastq.gz \
  --input-blob-name-2 reads_2.fastq.gz additional_reads_2.fastq.gz yet_more_reads_2.fastq.gz \
  --output-storage-account-name myaccount \
  --output-storage-account-key <storage access key to "myaccount"> \
  --output-storage-account-container outputs
```


Если вы предпочитаете использовать файл конфигурации, он должен содержать следующие строки:

```
api_url_base:                     <Genomics API URL>
access_key:                       <Genomics access key>
process_args:                     R=b37m1
input_storage_account_name:       myaccount
input_storage_account_key:        <storage access key to "myaccount">
input_storage_account_container:  inputs
input_blob_name_1:                reads_1.fq.gz additional_reads_1.fastq.gz yet_more_reads_1.fastq.gz
input_blob_name_2:                reads_2.fq.gz additional_reads_2.fastq.gz yet_more_reads_2.fastq.gz
output_storage_account_name:      myaccount
output_storage_account_key:       <storage access key to "myaccount">
output_storage_account_container: outputs
```

Отправьте файл `config.txt` с помощью этого вызова: `msgen submit -f config.txt`

## <a name="next-steps"></a>Дальнейшие действия
Из этой статьи вы узнали, как передать несколько BAM-файлов или пар FASTQ-файлов в службу хранилища Azure и как отправить рабочий процесс в службу Microsoft Genomics через клиент Python `msgen`. Дополнительные сведения об отправке рабочего процесса и других командах, которые можно использовать в службе Microsoft Genomics, см. в разделе [вопросов и ответов](frequently-asked-questions-genomics.md). 