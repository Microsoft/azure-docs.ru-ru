---
title: Сохранение состояния в службе "Сетка событий Azure" IoT Edge | Документация Майкрософт
description: Сохранение метаданных в Linux
author: VidyaKukke
manager: rajarv
ms.author: vkukke
ms.reviewer: spelluru
ms.date: 07/08/2020
ms.topic: article
ms.openlocfilehash: 92333c2528303a6fa53fa30f47def33c33235d39
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "86171488"
---
# <a name="persist-state-in-linux"></a>Сохранение состояния в Linux

Разделы и подписки, созданные в модуле сетки событий, по умолчанию хранятся в файловой системе контейнера. Без сохранения при повторном развертывании модуля все созданные метаданные будут потеряны. Чтобы сохранить данные между развертываниями и перезапусками, необходимо сохранить данные вне файловой системы контейнера.

По умолчанию сохраняются только метаданные, и события по-прежнему хранятся в памяти для повышения производительности. Чтобы включить сохраняемость событий, следуйте разделу материализованные события.

В этой статье описаны действия по развертыванию модуля службы "Сетка событий" с сохранением в развертываниях Linux.

> [!NOTE]
>Модуль "Сетка событий" работает как пользователь с низким уровнем привилегий с идентификатором UID `2000` и именем `eventgriduser` .

## <a name="persistence-via-volume-mount"></a>Сохраняемость через подключение тома

 [Тома DOCKER](https://docs.docker.com/storage/volumes/) используются для сохранения данных в разных развертываниях. Вы можете разрешить DOCKER автоматически создать именованный том как часть развертывания модуля сетки событий. Этот параметр является простейшим. Имя тома, создаваемого в разделе **BIND** , можно указать следующим образом.

```json
  {
     "HostConfig": {
          "Binds": [
                 "<your-volume-name-here>:/app/metadataDb"
           ]
      }
   }
```

>[!IMPORTANT]
>Не изменяйте вторую часть значения привязки. Он указывает на определенное расположение внутри модуля. Для модуля "Сетка событий" в Linux он должен быть **/АПП/метадатадб**.

Например, следующая конфигурация приведет к созданию тома **егметадатадбвол** , в котором будут храниться метаданные.

```json
 {
  "Env": [
    "inbound__serverAuth__tlsPolicy=strict",
    "inbound__serverAuth__serverCert__source=IoTEdge",
    "inbound__clientAuth__sasKeys__enabled=false",
    "inbound__clientAuth__clientCert__enabled=true",
    "inbound__clientAuth__clientCert__source=IoTEdge",
    "inbound__clientAuth__clientCert__allowUnknownCA=true",
    "outbound__clientAuth__clientCert__enabled=true",
    "outbound__clientAuth__clientCert__source=IoTEdge",
    "outbound__webhook__httpsOnly=true",
    "outbound__webhook__skipServerCertValidation=false",
    "outbound__webhook__allowUnknownCA=true"
  ],
  "HostConfig": {
    "Binds": [
      "egmetadataDbVol:/app/metadataDb",
      "egdataDbVol:/app/eventsDb"
    ],
    "PortBindings": {
      "4438/tcp": [
        {
          "HostPort": "4438"
        }
      ]
    }
  }
}
```

Вместо подключения тома можно создать каталог в главной системе и подключить этот каталог.

## <a name="persistence-via-host-directory-mount"></a>Сохранение с помощью подключения к каталогу узлов

Вместо тома DOCKER можно также подключить папку узла.

1. Сначала создайте пользователя с именем **евентгридусер** и идентификатором **2000** на главном компьютере, выполнив следующую команду:

    ```sh
    sudo useradd -u 2000 eventgriduser
    ```
1. Создайте каталог в файловой системе узла, выполнив следующую команду.

   ```sh
   md <your-directory-name-here>
   ```

    Например, при выполнении следующей команды будет создан каталог с именем **михостдир**.

    ```sh
    md /myhostdir
    ```
1. Затем сделайте **евентгридусер** владельцем этой папки, выполнив следующую команду.

   ```sh
   sudo chown eventgriduser:eventgriduser -hR <your-directory-name-here>
   ```

    Например,

    ```sh
    sudo chown eventgriduser:eventgriduser -hR /myhostdir
    ```
1. Используйте **привязки** для подключения каталога и повторного развертывания модуля службы "Сетка событий" из портал Azure.

    ```json
    {
         "HostConfig": {
            "Binds": [
                "<your-directory-name-here>:/app/metadataDb",
                "<your-directory-name-here>:/app/eventsDb",
             ]
         }
    }
    ```

    Например,

    ```json
    {
          "Env": [
            "inbound__serverAuth__tlsPolicy=strict",
            "inbound__serverAuth__serverCert__source=IoTEdge",
            "inbound__clientAuth__sasKeys__enabled=false",
            "inbound__clientAuth__clientCert__enabled=true",
            "inbound__clientAuth__clientCert__source=IoTEdge",
            "inbound__clientAuth__clientCert__allowUnknownCA=true",
            "outbound__clientAuth__clientCert__enabled=true",
            "outbound__clientAuth__clientCert__source=IoTEdge",
            "outbound__webhook__httpsOnly=true",
            "outbound__webhook__skipServerCertValidation=false",
            "outbound__webhook__allowUnknownCA=true"
          ],
          "HostConfig": {
                "Binds": [
                  "/myhostdir:/app/metadataDb",
                  "/myhostdir2:/app/eventsDb"
                ],
                "PortBindings": {
                      "4438/tcp": [
                        {
                          "HostPort": "4438"
                        }
                      ]
                }
          }
    }
    ```

    >[!IMPORTANT]
    >Не изменяйте вторую часть значения привязки. Он указывает на определенное расположение внутри модуля. Для модуля "Сетка событий" в Linux он должен быть **/АПП/метадатадб** и **/АПП/евентсдб**


## <a name="persist-events"></a>Сохранить события

Чтобы включить сохраняемость событий, необходимо сначала включить сохранение метаданных с помощью подключения тома или подключения к каталогу узла, используя приведенные выше разделы.

Важные моменты, которые следует учитывать при сохранении событий:

* Сохранение событий включено для каждой подписки на события и является явной присоединением тома или каталога.
* Сохраняемость событий настраивается в подписке на событие во время создания и не может быть изменена после создания подписки на события. Чтобы переключить сохраняемость событий, необходимо удалить и повторно создать подписку на события.
* Сохранение событий почти всегда выполняется медленнее, чем в операциях с памятью, однако разница в скорости сильно зависит от характеристик диска. Компромисс между скоростью и надежностью относится ко всем системам обмена сообщениями, но в целом становится заметным только в больших масштабах.

Чтобы включить сохраняемость событий в подписке на события, задайте для параметра значение `persistencePolicy` `true` :

 ```json
        {
          "properties": {
            "persistencePolicy": {
              "isPersisted": "true"
            },
            "destination": {
              "endpointType": "WebHook",
              "properties": {
                "endpointUrl": "<your-webhook-url>"
              }
            }
          }
        }
 ```
