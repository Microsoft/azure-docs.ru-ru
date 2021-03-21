---
title: Настройка интерфейсных платформ с помощью предварительной версии статических веб-приложений Azure
description: Параметры для популярных интерфейсных платформ, необходимых для статических веб-приложений Azure
services: static-web-apps
author: craigshoemaker
ms.service: static-web-apps
ms.topic: conceptual
ms.date: 07/18/2020
ms.author: cshoe
ms.openlocfilehash: 14564b0591ef0146131b3f9324556b613e25daac
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97901238"
---
# <a name="configure-front-end-frameworks-and-libraries-with-azure-static-web-apps-preview"></a>Настройка интерфейсных платформ и библиотек с помощью предварительной версии статических веб-приложений Azure

Статические веб-приложения Azure требуют наличия соответствующих значений конфигурации в [файле конфигурации сборки](github-actions-workflow.md) для интерфейсной платформы или библиотеки.

## <a name="configuration"></a>Конфигурация

В следующей таблице перечислены параметры для ряда платформ и библиотек<sup>1</sup>.

Цель столбцов таблицы объясняется следующими элементами:

- **Расположение выходных данных**: содержит значение `output_location` , которое является [папкой для построенных версий файлов приложения](github-actions-workflow.md#build-and-deploy).

- **Настраиваемая команда сборки**: Если для платформы требуется команда, отличающаяся от `npm run build` или `npm run azure:build` , можно определить [пользовательскую команду сборки](github-actions-workflow.md#custom-build-commands).

| Инфраструктура | Расположение артефактов приложения | Настраиваемая команда сборки |
|--|--|--|
| [Alpine.js](https://github.com/alpinejs/alpine/) | `/` | н/д <sup>2</sup> |
| [Angular](https://angular.io/) | `dist/<APP_NAME>` | `npm run build -- --prod` |
| [Угловой универсальный](https://angular.io/guide/universal) | `dist/<APP_NAME>/browser` | `npm run prerender` |
| [аурелиа](https://aurelia.io/) | `dist` | Н/Д |
| [Backbone.js](https://backbonejs.org/) | `/` | Н/Д |
| [Blazor](https://dotnet.microsoft.com/apps/aspnet/web-apps/blazor) | `wwwroot` | Н/Д |
| [Ember](https://emberjs.com/) | `dist` | Н/Д |
| [Flutter](https://flutter.dev/) | `build/web` | `flutter build web` |
| [Framework7](https://framework7.io/) | `www` | `npm run build-prod` |
| [глиммер](https://glimmerjs.com/) | `dist` | Н/Д |
| [HTML](https://developer.mozilla.org/docs/Web/HTML) | `/` | Н/Д |
| [хиперапп](https://hyperapp.dev/) | `/` | Н/Д |
| [JavaScript](https://developer.mozilla.org/docs/Web/javascript) | `/` | Н/Д |
| [jQuery](https://jquery.com/) | `/` | Н/Д |
| [KnockoutJS](https://knockoutjs.com/) | `dist` | Н/Д |
| [лителемент](https://lit-element.polymer-project.org/) | `dist` | Н/Д |
| [марко](https://markojs.com/) | `public` | Н/Д |
| [Метеор](https://www.meteor.com/) | `bundle` | Н/Д |
| [мисрил](https://mithril.js.org/) | `dist` | Н/Д |
| [Polymer](https://www.polymer-project.org/) | `build/default` | Н/Д |
| [преакт](https://preactjs.com/) | `build` | Н/Д |
| [React](https://reactjs.org/) | `build` | Н/Д |
| [Трафарета](https://stenciljs.com/) | `www` | Н/Д |
| [Svelte](https://svelte.dev/) | `public` | Н/Д |
| [Three.js](https://threejs.org/) | `/` | Н/Д |
| [TypeScript](https://www.typescriptlang.org/) | `dist` | Н/Д |
| [Vue.js](https://vuejs.org/) | `dist` | Н/Д |

<sup>1</sup> приведенная выше таблица не является исчерпывающим списком платформ и библиотек, которые работают со статическими веб-приложениями Azure.

<sup>2</sup> неприменимо

## <a name="next-steps"></a>Дальнейшие действия

- [Конфигурация сборки и рабочего процесса](github-actions-workflow.md)
