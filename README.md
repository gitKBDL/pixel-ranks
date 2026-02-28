<div align="center">
  <h1>Pixel Ranks</h1>
  <p><strong>Онлайн‑редактор пиксельных рангов в стиле Minecraft</strong></p>
  <p>Соберите бейдж ранга за пару минут: настройка слоёв, пресеты, сохранение проекта и экспорт в PNG.</p>
</div>

<p align="center">
  <img src="https://img.shields.io/badge/Vue-3-42b883?logo=vue.js&logoColor=white" alt="Vue 3">
  <img src="https://img.shields.io/badge/Vite-6-646cff?logo=vite&logoColor=white" alt="Vite 6">
  <img src="https://img.shields.io/badge/GitHub%20Pages-Ready-2f7dff" alt="GitHub Pages Ready">
  <img src="https://img.shields.io/badge/Backend-Not%20Required-1f8f6f" alt="Backend Not Required">
  <img src="https://img.shields.io/badge/License-Apache%202.0-blue" alt="Apache 2.0">
</p>

<p align="center">
  <a href="https://gitkbdl.github.io/pixel-ranks/">
    <img src="https://img.shields.io/badge/Открыть%20онлайн-Pixel%20Ranks-20c997?style=for-the-badge" alt="Открыть онлайн">
  </a>
</p>

## Что это

`Pixel Ranks` — генератор пиксельных рангов, который работает прямо в браузере.
Подходит для игровых серверов, паков, превью в Discord/Telegram и любых задач, где нужен компактный ретро‑лейбл в PNG.

## Для кого

- Для владельцев и админов игровых серверов
- Для тех, кто делает визуалы для Discord/Telegram
- Для тех, кому нужен быстрый пиксельный лейбл без сложных редакторов

## Что можно сделать

- Ввести текст и сразу увидеть результат
- Настроить фон, рамку, тень и текст по отдельности
- Включить градиенты (второй цвет показывается рядом с первым)
- Выбрать готовый пресет и быстро получить красивый стиль
- Сохранить проект и вернуться к нему позже
- Скачать PNG или скопировать PNG в буфер обмена

## Как сделать ранг (1 минута)

1. Введите текст ранга.
2. Выберите пресет или настройте цвета вручную.
3. Нажмите `Скачать PNG` или `Копировать PNG`.
4. Если хотите вернуться к макету позже, нажмите `Сохранить проект`.

## Полезно знать

- Проекты хранятся в вашем браузере (`localStorage`).
- Если очистить данные браузера, сохранённые проекты удалятся.
- Копирование PNG в буфер работает в `https` или `localhost` (ограничение браузеров).
- Кириллица автоматически транслитерируется для корректного пиксельного рендера.

## Горячие клавиши

| Комбинация | Что делает |
| --- | --- |
| `Ctrl/Cmd + S` | Сохраняет проект |
| `Ctrl/Cmd + Enter` | Скачивает PNG |

## Для разработчиков

### Локальный запуск

```bash
npm install
npm run dev
```

### Скрипты

| Команда | Назначение |
| --- | --- |
| `npm run dev` | Разработка |
| `npm run build` | Сборка в `dist/` |
| `npm run preview` | Просмотр production-сборки |

## Лицензия

[Apache License 2.0](./LICENSE)
