<div align="center">
  <h1>Pixel Ranks</h1>
  <p><strong>Локальный редактор пиксельных рангов в стиле Minecraft</strong></p>
  <p>Быстро собрать красивый бейдж, сохранить проект и выгрузить PNG за пару кликов.</p>
</div>

<p align="center">
  <img src="https://img.shields.io/badge/Vue-3-42b883?logo=vue.js&logoColor=white" alt="Vue 3">
  <img src="https://img.shields.io/badge/Vite-6-646cff?logo=vite&logoColor=white" alt="Vite 6">
  <img src="https://img.shields.io/badge/Local--First-Yes-2f7dff" alt="Local First">
  <img src="https://img.shields.io/badge/License-Apache%202.0-blue" alt="Apache 2.0">
</p>
<p align="center">
  <a href="https://gitkbdl.github.io/pixel-ranks/">
    <img src="https://img.shields.io/badge/Открыть%20онлайн-Pixel%20Ranks-20c997?style=for-the-badge" alt="Открыть онлайн">
  </a>
</p>

## О проекте

`Pixel Ranks` — это удобный генератор пиксельных рангов, где весь рабочий процесс происходит локально в браузере.
Подходит для серверов, паков, превью в Discord/Telegram и любых задач, где нужен компактный ретро-лейбл в PNG.

## Возможности

- Живой предпросмотр текста ранга
- Отдельные настройки для фона, рамки, тени и текста
- Градиенты на каждом слое (второй цвет показывается рядом с первым)
- Базовые пресеты с автоматически включаемыми градиентами
- Локальные проекты: сохранить, открыть, обновить, удалить, искать
- Экспорт PNG и копирование PNG в буфер обмена
- Полностью русский интерфейс

## Как пользоваться

1. Введите текст ранга.
2. Настройте цвета или выберите пресет.
3. Сохраните проект (по желанию).
4. Нажмите `Скачать PNG` или `Копировать PNG`.

## Горячие клавиши

| Комбинация | Действие |
| --- | --- |
| `Ctrl/Cmd + S` | Сохранить проект |
| `Ctrl/Cmd + Enter` | Скачать PNG |

## Быстрый старт

```bash
npm install
npm run dev
```

После запуска откройте адрес из терминала (обычно `http://localhost:5173`).

## Скрипты

```bash
npm run dev      # локальная разработка
npm run build    # production-сборка в /dist
npm run preview  # локальный просмотр production-сборки
```

## Технические заметки

- Приложение работает полностью локально: без сервера, без БД, без внешнего API.
- Проекты сохраняются в `localStorage` браузера.
- Для кириллицы применяется транслитерация в ASCII, чтобы текст корректно рендерился пиксельным шрифтом.
- Копирование PNG в буфер обмена работает только в `https` или `localhost` (ограничение браузеров).

## Деплой на GitHub Pages

В проекте уже настроен базовый путь:

```js
base: '/pixel-ranks/'
```

Это значение находится в `vite.config.js` и подходит для публикации репозитория `pixel-ranks` в GitHub Pages.

## Лицензия

[Apache License 2.0](./LICENSE)
