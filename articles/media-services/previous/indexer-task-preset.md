---
title: Предустановка задачи для индексатора мультимедийных данных Azure
description: В этом разделе приводятся общие сведения о предустановленной задаче для индексатора мультимедиа служб мультимедиа Azure.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 03/10/2021
ms.author: inhenkel
ms.openlocfilehash: c054c0aa8c58894f63f8ce8432e8d7a9e1275639
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103012230"
---
# <a name="task-preset-for-azure-media-indexer"></a>Предустановка задачи для индексатора мультимедийных данных Azure

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

Индексатор мультимедийных данных Azure — это обработчик мультимедиа, который используется для выполнения следующих задач: обеспечение возможности поиска по мультимедийным файлам и содержимому, создание дорожек скрытых субтитров и ключевых слов, а также индексирование файлов ресурсов, являющихся частью вашего ресурса.

В этом разделе описывается предустановка задачи, которую необходимо передать в задание индексирования. Полный пример см. в статье [Индексирование файлов мультимедиа с помощью индексатора мультимедийных данных Azure](media-services-index-content.md).

## <a name="azure-media-indexer-configuration-xml"></a>XML-файл конфигурации индексатора мультимедийных данных Azure

В следующей таблице описаны элементы и атрибуты XML-файла конфигурации.

|Имя|Require|Описание|
|---|---|---|
|Входные данные|true|Файлы ресурсов-контейнеров, которые необходимо проиндексировать.<br/>Azure Media Indexer поддерживает следующие форматы файлов мультимедиа: MP4, MOV, WMV, MP3, M4A, WMA, AAC, WAV. <br/><br/>Можно указать одно или несколько имен файлов в атрибуте **name** или **list** элемента **input**, как показано ниже. Если индексируемый файл ресурса не указан, выбирается основной файл. Если основной файл ресурса-контейнера не задан, индексируется первый файл входного ресурса-контейнера.<br/><br/>Чтобы явно указать имя файла ресурса-контейнера, используйте запись следующего формата:<br/>```<input name="TestFile.wmv" />```<br/><br/>Вы также можете настроить индексирование сразу нескольких файлов ресурсов-контейнеров (до 10). Для этого выполните следующие действия.<br/>- Создайте текстовый файл (файл манифеста) с расширением LST.<br/>- Внесите в этот файл манифеста список всех имен файлов входного ресурса-контейнера.<br/>— Добавьте (передайте) файл манифеста в ресурс.<br/>- Укажите имя файла манифеста в атрибуте list элемента input.<br/>```<input list="input.lst">```<br/><br/>**Примечание.** Если добавить в файл манифеста более 10 файлов, задание индексирования завершится с кодом ошибки 2006.|
|метаданные|false|Метаданные для указанных файлов ресурсов.<br/>```<metadata key="..." value="..." />```<br/><br/>Вы можете указать значения (value) для предопределенных ключей (key). <br/><br/>Сейчас поддерживаются следующие ключи.<br/><br/>**title** и **description** — служат для обновления модели языка с целью повышения точности распознавания речи.<br/>```<metadata key="title" value="[Title of the media file]" /><metadata key="description" value="[Description of the media file]" />```<br/><br/>**username** и **password** — применяются для проверки подлинности при загрузке файлов из Интернета по протоколу HTTP или HTTPS.<br/>```<metadata key="username" value="[UserName]" /><metadata key="password" value="[Password]" />```<br/>Значения username (имя пользователя) и password (пароль) применяются ко всем URL-адресам мультимедийного содержимого во входном манифесте.|
|features<br/><br/> Добавлено в версии 1.2. В настоящее время поддерживается только одна функция: распознавание речи (ASR).|false|Функция распознавания речи имеет следующие ключи настройки.<br/><br/>Язык:<br/>- естественный язык, распознаваемый в файле мультимедиа;<br/>- English, Spanish (английский, испанский).<br/><br/>CaptionFormats:<br/>- список предпочтительных форматов вывода (если они есть), разделенных точкой с запятой;<br/>-TTML; WebVTT<br/><br/><br/>GenerateKeywords:<br/>- Логический флаг. Если он установлен, создается XML-файл ключевых слов.<br/>- True; False.|

## <a name="azure-media-indexer-configuration-xml-example"></a>Пример XML-файла конфигурации индексатора мультимедийных данных Azure

``` 
<?xml version="1.0" encoding="utf-8"?>  
<configuration version="2.0">  
  <input>  
    <metadata key="title" value="[Title of the media file]" />  
    <metadata key="description" value="[Description of the media file]" />  
  </input>  
  <settings>  
  </settings>  
  
  <features>  
    <feature name="ASR">    
      <settings>  
        <add key="Language" value="English"/>  
        <add key="GenerateKeywords" value ="true" />  
      </settings>  
    </feature>  
  </features>  
  
</configuration>  
```
  
## <a name="next-steps"></a>Дальнейшие действия

См. статью [Индексирование файлов мультимедиа с помощью индексатора мультимедийных данных Azure](media-services-index-content.md).

