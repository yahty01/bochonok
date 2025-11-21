# Архитектура проекта

## Общая структура

```
src/
├── app/                    # Next.js App Router
│   ├── auth/callback/      # OAuth callback
│   ├── login/              # Страница входа
│   ├── layout.tsx          # Корневой layout
│   ├── page.tsx            # Главная страница
│   └── globals.css         # Глобальные стили
├── components/             # React-компоненты
│   └── auth/               # Компоненты аутентификации
├── lib/                    # Утилиты и хелперы
│   ├── supabase/           # Клиенты Supabase
│   └── types/              # TypeScript типы
└── middleware.ts           # Middleware для защиты маршрутов
```

## Архитектурные паттерны

### Next.js App Router

Проект использует **App Router** (Next.js 13+) с поддержкой:

- **Server Components** — серверный рендеринг по умолчанию
- **Client Components** — интерактивность (с директивой `'use client'`)
- **Server Actions** — серверные функции без API-эндпоинтов

### Разделение Server/Client компонентов

#### Server Components (по умолчанию)

```tsx
// src/app/page.tsx
export default async function Page() {
  const data = await fetchData()  // Выполняется на сервере
  return <div>{data}</div>
}
```

**Преимущества:**
- Прямой доступ к БД
- Нет отправки JS на клиент
- Безопасная работа с секретами

#### Client Components

```tsx
// src/components/auth/login-form.tsx
'use client'

export default function LoginForm() {
  const [state, setState] = useState()  // Требует клиентский код
  return <form>...</form>
}
```

**Используются для:**
- Состояние (useState, useReducer)
- Эффекты (useEffect)
- Обработчики событий
- Браузерные API

## Supabase интеграция

### Два типа клиентов

#### Server Client

```typescript
// src/lib/supabase/server.ts
export async function createClient() {
  const cookieStore = await cookies()
  return createServerClient(url, key, { cookies: {...} })
}
```

**Применение:** Server Components, API Routes, middleware

#### Browser Client

```typescript
// src/lib/supabase/client.ts
export function createClient() {
  return createBrowserClient(url, key)
}
```

**Применение:** Client Components

### Аутентификация

```
1. Пользователь → LoginForm (Client Component)
2. LoginForm → Supabase Auth API
3. Supabase → JWT токен в cookie
4. Middleware → проверка токена
5. Server Component → доступ к данным пользователя
```

#### Middleware защита

Middleware проверяет JWT токен на каждом запросе и автоматически обновляет его. Неавторизованные пользователи перенаправляются на `/login`.

## База данных

### Схема

```sql
-- Таблица профилей
CREATE TABLE profiles (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES auth.users(id),
  full_name TEXT,
  avatar_url TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### Row Level Security (RLS)

```sql
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users view own profile"
  ON profiles FOR SELECT
  USING (auth.uid() = user_id);
```

## TypeScript типизация

### Автогенерация типов БД

```typescript
// src/lib/types/database.ts
export type Database = {
  public: {
    Tables: {
      profiles: {
        Row: {
          id: string
          user_id: string | null
          full_name: string | null
          avatar_url: string | null
          created_at: string | null
        }
        Insert: { /* опциональные поля */ }
        Update: { /* опциональные поля */ }
      }
    }
  }
}
```

Типы генерируются через Supabase CLI (см. DEVELOPMENT.md).

## Стайлинг

### Tailwind CSS 4

```tsx
// Утилитарный подход
<div className="container mx-auto px-4 py-8">
  <h1 className="text-4xl font-bold text-gray-900 mb-2">
    Заголовок
  </h1>
</div>
```

### Глобальные стили

```css
/* src/app/globals.css */
@tailwind base;
@tailwind components;
@tailwind utilities;
```

## Паттерны работы с данными

- **Server Components** — прямые запросы к БД через `await createClient()`
- **Client Components** — мутации данных, подписки на изменения
- **Используйте `router.refresh()`** после изменений для обновления Server Components

Примеры см. в DEVELOPMENT.md и API.md.

## Производительность

- **Кэширование** — fetch по умолчанию кэшируется, используйте `cache: 'no-store'` для динамических данных
- **Оптимизация изображений** — компонент `next/image` автоматически оптимизирует изображения
- **Code splitting** — динамический импорт через `next/dynamic`

## Безопасность

### Переменные окружения

- `NEXT_PUBLIC_*` — доступны в браузере
- Без префикса — только на сервере

### RLS политики

Все таблицы защищены RLS-политиками, ограничивающими доступ на уровне строк.

### JWT токены

- Автоматическое обновление через middleware
- Хранение в httpOnly cookies
- Проверка на каждом запросе
