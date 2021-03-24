---
title: Примеры для PowerShell
description: В этой статье приведены примеры Azure PowerShell для некоторых распространенных сценариев Службы приложений. Узнайте, как автоматизировать задачи развертывания и управления в Службе приложений.
tags: azure-service-management
ms.assetid: b48d1137-8c04-46e0-b430-101e07d7e470
ms.topic: sample
ms.date: 07/07/2020
ms.custom: mvc
ms.openlocfilehash: a1577d42de9a4452467a448a0de5cd5f9575a55f
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "86169431"
---
# <a name="powershell-samples-for-azure-app-service"></a>Примеры PowerShell для Службы приложений Azure

В следующей таблице приведены ссылки на сценарии PowerShell, созданные с помощью Azure PowerShell.

| Скрипт | Описание |
|-|-|
|**Создание приложения**||
| [Создание приложения с развертыванием из GitHub](./scripts/powershell-deploy-github.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Создает приложение Службы приложений, которое вытягивает код из GitHub. |
| [Создание приложения с непрерывным развертыванием из GitHub](./scripts/powershell-continuous-deployment-github.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Создает приложение Службы приложений, которое непрерывно развертывает код из GitHub. |
| [Создание приложения и развертывание кода с помощью протокола FTP](./scripts/powershell-deploy-ftp.md?toc=%2fpowershell%2fmodule%2ftoc.json) | Создает приложение Службы приложений и передает файлы из локального каталога с помощью протокола FTP. |
| [Создание приложения и развертывание кода из локального репозитория Git](./scripts/powershell-deploy-local-git.md?toc=%2fpowershell%2fmodule%2ftoc.json) | Создает приложение Службы приложений и настраивает вживление кода из локального репозитория Git. |
| [Создание приложения и развертывание кода в промежуточной среде](./scripts/powershell-deploy-staging-environment.md?toc=%2fpowershell%2fmodule%2ftoc.json) | Создает приложение Службы приложений со слотом развертывания для изменений промежуточного кода. |
|  [Создание приложения и его предоставление с использованием частной конечной точки](./scripts/powershell-deploy-private-endpoint.md?toc=%2fpowershell%2fmodule%2ftoc.json) | Создает приложение в Службе приложений с частной конечной точкой. |
|**Настройка приложения**||
| [Сопоставление личного домена с приложением](./scripts/powershell-configure-custom-domain.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Создает приложение Службы приложений и сопоставляет c ним имя личного домена. |
| [Привязка TLS/SSL-сертификата к приложению](./scripts/powershell-configure-ssl-certificate.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Создает приложение Службы приложений и привязывает к нему TLS/SSL-сертификат имени личного домена. |
|**Масштабирование приложения**||
| [Масштабирование приложения вручную](./scripts/powershell-scale-manual.md?toc=%2fpowershell%2fmodule%2ftoc.json) | Создает приложение Службы приложений и масштабирует его по двум экземплярам. |
| [Глобальное масштабирование приложения с помощью высокодоступной архитектуры](./scripts/powershell-scale-high-availability.md?toc=%2fpowershell%2fmodule%2ftoc.json) | Создает два приложения Службы приложений в двух разных географических регионах и делает их доступными через одну конечную точку с помощью диспетчера трафика Azure. |
|**Подключение приложения к ресурсам**||
| [Подключение приложения к Базе данных SQL](./scripts/powershell-connect-to-sql.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Создает приложение Службы приложений и базу данных в Базе данных SQL Azure, а затем добавляет строку подключения базы данных к параметрам приложения. |
| [Подключение приложения к учетной записи хранения](./scripts/powershell-connect-to-storage.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Создает приложение Службы приложений и учетную запись хранения, а затем добавляет строку подключения хранилища к параметрам приложения. |
|**Резервное копирование и восстановление приложения**||
| [Резервное копирование приложения](./scripts/powershell-backup-onetime.md?toc=%2fpowershell%2fmodule%2ftoc.json) | Создает приложение Службы приложений и однократно создает его резервную копию. |
| [Создание резервной копии приложения по расписанию](./scripts/powershell-backup-scheduled.md?toc=%2fpowershell%2fmodule%2ftoc.json) | Создает приложение Службы приложений и запланировано создает его резервную копию. |
| [Удаление резервной копии приложения](./scripts/powershell-backup-delete.md?toc=%2fpowershell%2fmodule%2ftoc.json) | Удаление существующей резервной копии приложения. |
| [Восстановление приложения из резервной копии](./scripts/powershell-backup-restore.md?toc=%2fpowershell%2fmodule%2ftoc.json) | Восстановление приложения из ранее созданной резервной копии. |
| [Восстановление резервной копии между подписками](./scripts/powershell-backup-restore-diff-sub.md?toc=%2fpowershell%2fmodule%2ftoc.json) | Восстановление веб-приложения из резервной копии в других подписках. |
|**Мониторинг приложения**||
| [Мониторинг приложения с помощью журналов веб-сервера](./scripts/powershell-monitor.md?toc=%2fpowershell%2fmodule%2ftoc.json) | Создает приложение Службы приложений, включает ведение журналов и скачивает их на локальный компьютер. |
| | |
