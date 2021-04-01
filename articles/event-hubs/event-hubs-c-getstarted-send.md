---
title: Краткое руководство. Отправка событий с помощью C в Центрах событий Azure
description: В этой статье описано, как создать приложение C, которое отправляет сообщения в Центры событий Azure.
ms.topic: quickstart
ms.date: 06/23/2020
ms.openlocfilehash: bfe1ca1a45f7b33d7431aed13446d8d72f79fb90
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "85315668"
---
# <a name="quickstart-send-events-to-azure-event-hubs-using-c"></a>Краткое руководство. Отправка событий в Центры событий Azure с помощью C

## <a name="introduction"></a>Введение
Центры событий Azure — это платформа потоковой передачи больших данных и служба приема событий, принимающая и обрабатывающая миллионы событий в секунду. Центры событий могут обрабатывать и сохранять события, данные и телеметрию, созданные распределенным программным обеспечением и устройствами. Данные, отправляемые в концентратор событий, можно преобразовывать и сохранять с помощью любого поставщика аналитики в реальном времени, а также с помощью адаптеров пакетной обработки или хранения. Подробный обзор Центров событий см. в статьях [Что такое Центры событий Azure?](event-hubs-about.md) и [Обзор функций Центров событий](event-hubs-features.md).

В этом руководстве описано, как отправлять события в концентратор событий с помощью консольного приложения на языке C. 

## <a name="prerequisites"></a>Предварительные требования
Для работы с этим учебником требуется:

* Среда разработки C. В этом учебнике предполагается, что применяется стек gcc на виртуальной машине Azure Linux с Ubuntu 14.04.
* [Microsoft Visual Studio](https://www.visualstudio.com/).
* **Создайте пространство имен Центров событий и концентратор событий**. Вы можете использовать [портал Azure](https://portal.azure.com) для создания пространства имен типа Центров событий и получать учетные данные для управления, требуемые приложению для взаимодействия с концентратором событий. Чтобы создать пространство имен и концентратор событий, выполните инструкции из [этой статьи](event-hubs-create.md). Получите значение ключа доступа для концентратора событий, следуя инструкциям в разделе [Получение строки подключения на портале](event-hubs-get-connection-string.md#get-connection-string-from-the-portal). Используйте ключ доступа в коде, который вы напишете далее в рамках этого руководства. Имя ключа по умолчанию: **RootManageSharedAccessKey**.

## <a name="write-code-to-send-messages-to-event-hubs"></a>Написание кода для отправки сообщений в Центры событий
Из этого раздела вы узнаете, как написать приложение для отправки событий в концентратор на языке C. В коде используется библиотека Proton AMQP из [проекта Apache Qpid](https://qpid.apache.org/). Эта процедура аналогична использованию очередей и разделов службы "Служебная шина" с AMQP на C, как показано [в этом примере](https://code.msdn.microsoft.com/Using-Apache-Qpid-Proton-C-afd76504). Дополнительную информацию см. в [документации по Qpid Proton](https://qpid.apache.org/proton/index.html).

1. Чтобы установить Qpid Proton, следуйте инструкциям для своей среды на [странице Qpid AMQP Messenger](https://qpid.apache.org/proton/messenger.html).
2. Для компиляции библиотеки Proton установите следующие пакеты.
   
    ```shell
    sudo apt-get install build-essential cmake uuid-dev openssl libssl-dev
    ```
3. Скачайте [библиотеку Qpid Proton](https://qpid.apache.org/proton/index.html) и извлеките ее, например:
   
    ```shell
    wget https://archive.apache.org/dist/qpid/proton/0.7/qpid-proton-0.7.tar.gz
    tar xvfz qpid-proton-0.7.tar.gz
    ```
4. Создайте каталог построения, скомпилируйте и установите:
   
    ```shell
    cd qpid-proton-0.7
    mkdir build
    cd build
    cmake -DCMAKE_INSTALL_PREFIX=/usr ..
    sudo make install
    ```
5. В рабочем каталоге создайте новый файл с именем **sender.c** со следующим кодом. Обязательно замените значения для ключа и имени, имени концентратора событий и пространства имен SAS. Также необходимо заменить версию ключа **SendRule** , закодированную как URL-адрес, созданную ранее. Выполнить кодировку как URL-адрес можно [здесь](https://www.w3schools.com/tags/ref_urlencode.asp).
   
    ```c
    #include "proton/message.h"
    #include "proton/messenger.h"
   
    #include <getopt.h>
    #include <proton/util.h>
    #include <sys/time.h>
    #include <stddef.h>
    #include <stdio.h>
    #include <string.h>
    #include <unistd.h>
    #include <stdlib.h>
   
    #define check(messenger)                                                     \
      {                                                                          \
        if(pn_messenger_errno(messenger))                                        \
        {                                                                        \
          printf("check\n");                                                     \
          die(__FILE__, __LINE__, pn_error_text(pn_messenger_error(messenger))); \
        }                                                                        \
      }
   
    pn_timestamp_t time_now(void)
    {
      struct timeval now;
      if (gettimeofday(&now, NULL)) pn_fatal("gettimeofday failed\n");
      return ((pn_timestamp_t)now.tv_sec) * 1000 + (now.tv_usec / 1000);
    }  
   
    void die(const char *file, int line, const char *message)
    {
      printf("Dead\n");
      fprintf(stderr, "%s:%i: %s\n", file, line, message);
      exit(1);
    }
   
    int sendMessage(pn_messenger_t * messenger) {
        char * address = (char *) "amqps://{SAS Key Name}:{SAS key}@{namespace name}.servicebus.windows.net/{event hub name}";
        char * msgtext = (char *) "Hello from C!";
   
        pn_message_t * message;
        pn_data_t * body;
        message = pn_message();
   
        pn_message_set_address(message, address);
        pn_message_set_content_type(message, (char*) "application/octect-stream");
        pn_message_set_inferred(message, true);
   
        body = pn_message_body(message);
        pn_data_put_binary(body, pn_bytes(strlen(msgtext), msgtext));
   
        pn_messenger_put(messenger, message);
        check(messenger);
        pn_messenger_send(messenger, 1);
        check(messenger);
   
        pn_message_free(message);
    }
   
    int main(int argc, char** argv) {
        printf("Press Ctrl-C to stop the sender process\n");
   
        pn_messenger_t *messenger = pn_messenger(NULL);
        pn_messenger_set_outgoing_window(messenger, 1);
        pn_messenger_start(messenger);
   
        while(true) {
            sendMessage(messenger);
            printf("Sent message\n");
            sleep(1);
        }
   
        // release messenger resources
        pn_messenger_stop(messenger);
        pn_messenger_free(messenger);
   
        return 0;
    }
    ```
6. Скомпилируйте файл при условии **gcc**:
   
    ```
    gcc sender.c -o sender -lqpid-proton
    ```

    > [!NOTE]
    > В этом коде для периода отправки используется значение 1. Это обеспечивают максимальную скорость принудительной отправки сообщений. Рекомендуем сгруппировать сообщения в приложении, чтобы увеличить пропускную способность. Информацию об использовании библиотеки Qpid Proton в этой и других средах, а также на платформах, для которых предоставляются привязки (сейчас это Perl, PHP, Python и Ruby), см. на [странице Qpid AMQP Messenger](https://qpid.apache.org/proton/messenger.html).

Запустите приложение для отправки сообщений в концентратор событий. 

Поздравляем! Теперь вы можете отправлять сообщения в концентратор событий.

## <a name="next-steps"></a>Дальнейшие действия
Ознакомьтесь со следующими статьями:

- [EventProcessorHost](event-hubs-event-processor-host.md)
- [Функции и терминология в Центрах событий Azure](event-hubs-features.md).


<!-- Images. -->
[21]: ./media/event-hubs-c-ephcs-getstarted/run-csharp-ephcs1.png
[24]: ./media/event-hubs-c-ephcs-getstarted/receive-eph-c.png
