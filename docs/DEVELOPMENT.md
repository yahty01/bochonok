# Руководство по разработке

## Настройка окружения

### Требования

- Node.js 20+
- pnpm 8+
- Аккаунт Supabase

### Первоначальная настройка

```bash
# 1. Клонирование
git clone <repo-url>
cd bochonok

# 2. Установка зависимостей
pnpm install

# 3. Настройка Supabase
# - Создайте проект на supabase.com
# - Скопируйте Project URL и Anon Key

# 4. Создание .env.local
cp .env.example .env.local
# Заполните переменные из Supabase

# 5. Запуск
pnpm dev
```

## Работа с компонентами

### Создание Server Component

```tsx
// src/app/dashboard/page.tsx
import { createClient } from '@/lib/supabase/server'

export default async function Dashboard() {
  const supabase = await createClient()
  const { data: { user } } = await supabase.auth.getUser()
  
  return <div>Hello, {user?.email}</div>
}
```

**Правила:**
- Не используйте хуки (useState, useEffect)
- Можно использовать async/await
- Прямой доступ к БД и cookies

### Создание Client Component

```tsx
// src/components/counter.tsx
'use client'

import { useState } from 'react'

export default function Counter() {
  const [count, setCount] = useState(0)
  
  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  )
}
```

**Правила:**
- Добавьте `'use client'` в начало файла
- Можно использовать хуки и браузерные API
- Нет прямого доступа к серверным ресурсам

## Работа с данными

### Чтение данных (Server Component)

```tsx
async function fetchPosts() {
  const supabase = await createClient()
  
  const { data, error } = await supabase
    .from('posts')
    .select('*')
    .order('created_at', { ascending: false })
    .limit(10)
  
  if (error) {
    console.error('Error:', error)
    return []
  }
  
  return data
}

export default async function PostsPage() {
  const posts = await fetchPosts()
  
  return (
    <div>
      {posts.map(post => (
        <article key={post.id}>{post.title}</article>
      ))}
    </div>
  )
}
```

### Создание/обновление данных (Client Component)

```tsx
'use client'

import { createClient } from '@/lib/supabase/client'
import { useState } from 'react'

export default function CreatePost() {
  const [title, setTitle] = useState('')
  const [loading, setLoading] = useState(false)
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault()
    setLoading(true)
    
    const supabase = createClient()
    const { data: { user } } = await supabase.auth.getUser()
    
    const { error } = await supabase
      .from('posts')
      .insert({ title, user_id: user?.id })
    
    if (error) {
      alert('Ошибка: ' + error.message)
    } else {
      setTitle('')
      alert('Пост создан!')
    }
    
    setLoading(false)
  }
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        value={title}
        onChange={(e) => setTitle(e.target.value)}
        placeholder="Заголовок"
      />
      <button disabled={loading}>
        {loading ? 'Сохранение...' : 'Создать'}
      </button>
    </form>
  )
}
```

## Аутентификация

Основные методы:
- `signUp()` — регистрация
- `signInWithPassword()` — вход
- `signOut()` — выход
- `auth.getUser()` — получение текущего пользователя

Подробное API см. в API.md.

## Работа с базой данных

### Создание таблицы

```sql
CREATE TABLE posts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES auth.users(id),
  title TEXT NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Включение Row Level Security
ALTER TABLE posts ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users view own posts"
  ON posts FOR SELECT
  USING (auth.uid() = user_id);
```

### Обновление типов TypeScript

```bash
supabase gen types typescript \
  --project-id YOUR_PROJECT_REF \
  > src/lib/types/database.ts
```

## Маршрутизация

### Структура файлов App Router

```
app/
├── page.tsx              # / (корневой маршрут)
├── about/
│   └── page.tsx          # /about
├── blog/
│   ├── page.tsx          # /blog
│   └── [slug]/
│       └── page.tsx      # /blog/:slug (динамический)
└── api/
    └── hello/
        └── route.ts      # /api/hello (API-эндпоинт)
```

### Динамические маршруты

```tsx
// app/blog/[slug]/page.tsx
export default async function BlogPost({ 
  params 
}: { 
  params: { slug: string } 
}) {
  const { slug } = params
  
  return <div>Post: {slug}</div>
}
```

### Навигация

```tsx
import Link from 'next/link'
import { useRouter } from 'next/navigation'

// Декларативная навигация
<Link href="/about">О нас</Link>

// Программная навигация
const router = useRouter()
router.push('/dashboard')
```

## Стилизация

### Tailwind классы

```tsx
<div className="container mx-auto px-4 py-8">
  <h1 className="text-3xl font-bold mb-4">
    Заголовок
  </h1>
  <button className="bg-blue-600 text-white px-4 py-2 rounded hover:bg-blue-700">
    Кнопка
  </button>
</div>
```

### Условная стилизация

```tsx
<div className={`
  p-4 rounded
  ${isActive ? 'bg-blue-500' : 'bg-gray-300'}
`}>
  Контент
</div>
```

## Обработка ошибок

### Try-catch в Server Component

```tsx
export default async function Page() {
  try {
    const data = await fetchData()
    return <div>{data}</div>
  } catch (error) {
    return <div>Ошибка загрузки данных</div>
  }
}
```

### Error boundaries

```tsx
// app/error.tsx
'use client'

export default function Error({ 
  error, 
  reset 
}: {
  error: Error
  reset: () => void
}) {
  return (
    <div>
      <h2>Что-то пошло не так!</h2>
      <button onClick={reset}>Попробовать снова</button>
    </div>
  )
}
```

## Оптимизация

### Lazy loading компонентов

```tsx
import dynamic from 'next/dynamic'

const HeavyComponent = dynamic(() => import('./HeavyComponent'), {
  loading: () => <div>Загрузка...</div>,
  ssr: false  // Отключить SSR для компонента
})
```

### Оптимизация изображений

```tsx
import Image from 'next/image'

<Image src="/photo.jpg" alt="Описание" width={800} height={600} />
```

## Тестирование

### Настройка ESLint

```bash
pnpm lint              # Проверка
pnpm lint --fix        # Автоисправление
```

### Рекомендуемые расширения VS Code

```json
{
  "recommendations": [
    "dbaeumer.vscode-eslint",
    "bradlc.vscode-tailwindcss",
    "esbenp.prettier-vscode"
  ]
}
```

## Отладка

### Логирование на сервере

```tsx
// Server Component
export default async function Page() {
  console.log('Это выведется в терминале, а не в браузере')
  return <div>Page</div>
}
```

### Логирование на клиенте

```tsx
'use client'

export default function Component() {
  console.log('Это выведется в консоли браузера')
  return <div>Component</div>
}
```

### React DevTools

- Установите расширение React Developer Tools
- Используйте для инспекции компонентов и props

## Git workflow

```bash
# Создание feature-ветки
git checkout -b feature/new-feature

# Коммит изменений
git add .
git commit -m "feat: добавлена новая функция"

# Push в удаленный репозиторий
git push origin feature/new-feature

# Создание Pull Request на GitHub
```

## Деплой

### Vercel (рекомендуется)

```bash
# Установка Vercel CLI
npm i -g vercel

# Деплой
vercel

# Production деплой
vercel --prod
```

**Настройка переменных окружения в Vercel:**
1. Settings → Environment Variables
2. Добавьте все переменные из `.env.local`

## Частые проблемы

### Ошибка: "Invalid API key"

**Решение:** Проверьте правильность ключей в `.env.local` и перезапустите сервер.

### Ошибка: "Cookies can only be modified in a Server Action"

**Решение:** Используйте Server Client в Server Components, а Browser Client — в Client Components.

### Данные не обновляются после мутации

**Решение:** Используйте `router.refresh()` после изменения данных.

```tsx
const router = useRouter()

await updateData()
router.refresh()  // Обновит Server Components
```
