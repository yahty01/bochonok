# Чеклист разработчика

## Первоначальная настройка

- [ ] Клонировать репозиторий
- [ ] Установить зависимости: `pnpm install`
- [ ] Создать проект на [supabase.com](https://supabase.com)
- [ ] Скопировать `.env.example` в `.env.local`
- [ ] Заполнить `NEXT_PUBLIC_SUPABASE_URL` и `NEXT_PUBLIC_SUPABASE_ANON_KEY`
- [ ] Запустить: `pnpm dev`

## Создание новой таблицы

- [ ] Открыть Supabase SQL Editor
- [ ] Создать таблицу с нужными полями
- [ ] Включить RLS: `ALTER TABLE table_name ENABLE ROW LEVEL SECURITY;`
- [ ] Создать политики доступа (SELECT, INSERT, UPDATE, DELETE)
- [ ] Обновить типы: `supabase gen types typescript --project-id ID > src/lib/types/database.ts`

## Создание новой страницы

- [ ] Создать файл `src/app/[route]/page.tsx`
- [ ] Определить как Server или Client Component
- [ ] Добавить получение данных (если нужно)
- [ ] Добавить стили через Tailwind
- [ ] Протестировать в браузере

## Работа с данными

**Чтение (Server Component):**
- [ ] Импортировать `createClient` из `@/lib/supabase/server`
- [ ] Вызвать `await createClient()`
- [ ] Выполнить запрос `.from('table').select('*')`
- [ ] Обработать ошибки

**Запись (Client Component):**
- [ ] Добавить `'use client'` в начало файла
- [ ] Импортировать `createClient` из `@/lib/supabase/client`
- [ ] Вызвать `createClient()`
- [ ] Выполнить `.insert()` или `.update()`
- [ ] Добавить `router.refresh()` после мутации

## Перед коммитом

- [ ] Запустить `pnpm lint`
- [ ] Проверить TypeScript ошибки
- [ ] Убедиться что RLS политики работают
- [ ] Протестировать функционал
- [ ] Написать понятный commit message

## Перед деплоем

- [ ] Запустить `pnpm build`
- [ ] Проверить production сборку
- [ ] Настроить переменные окружения на платформе
- [ ] Проверить что все ключи Supabase корректны
- [ ] Добавить production URL в Supabase Auth Settings

## Типичные проблемы

**"Invalid API key"**
→ Проверьте `.env.local`, перезапустите сервер

**"Cookies can only be modified in Server Action"**
→ Используйте правильный клиент (server/browser)

**Данные не обновляются**
→ Добавьте `router.refresh()` после мутации

**RLS блокирует запросы**
→ Проверьте политики, используйте `auth.uid() = user_id`
