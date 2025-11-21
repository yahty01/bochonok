# API и интеграции

## Supabase API

### Инициализация клиента

#### Server-side

```typescript
import { createClient } from '@/lib/supabase/server'

const supabase = await createClient()
```

#### Client-side

```typescript
import { createClient } from '@/lib/supabase/client'

const supabase = createClient()
```

## Database API

### Select

```typescript
// Получить все записи
const { data, error } = await supabase
  .from('posts')
  .select('*')

// С фильтрацией
const { data, error } = await supabase
  .from('posts')
  .select('*')
  .eq('user_id', userId)
  .gt('created_at', '2024-01-01')

// С лимитом и сортировкой
const { data, error } = await supabase
  .from('posts')
  .select('*')
  .order('created_at', { ascending: false })
  .limit(10)

// С join (foreign key)
const { data, error } = await supabase
  .from('posts')
  .select(`
    *,
    profiles (
      full_name,
      avatar_url
    )
  `)

// Single row
const { data, error } = await supabase
  .from('profiles')
  .select('*')
  .eq('user_id', userId)
  .single()
```

### Insert

```typescript
// Одна запись
const { data, error } = await supabase
  .from('posts')
  .insert({
    title: 'New Post',
    content: 'Content here',
    user_id: userId
  })
  .select()

// Множественная вставка
const { data, error } = await supabase
  .from('posts')
  .insert([
    { title: 'Post 1', user_id: userId },
    { title: 'Post 2', user_id: userId }
  ])
  .select()
```

### Update

```typescript
// По ID
const { data, error } = await supabase
  .from('posts')
  .update({ title: 'Updated Title' })
  .eq('id', postId)
  .select()

// Множественное обновление
const { data, error } = await supabase
  .from('posts')
  .update({ status: 'published' })
  .eq('user_id', userId)
  .select()
```

### Delete

```typescript
// По ID
const { error } = await supabase
  .from('posts')
  .delete()
  .eq('id', postId)

// С условием
const { error } = await supabase
  .from('posts')
  .delete()
  .eq('user_id', userId)
  .lt('created_at', '2023-01-01')
```

### Фильтры

```typescript
// Равно
.eq('column', 'value')

// Не равно
.neq('column', 'value')

// Больше/меньше
.gt('column', 10)
.lt('column', 100)
.gte('column', 10)
.lte('column', 100)

// Как (LIKE)
.like('column', '%pattern%')
.ilike('column', '%pattern%')  // case-insensitive

// В списке
.in('column', [1, 2, 3])

// NULL проверка
.is('column', null)

// Комбинация фильтров (AND)
.eq('status', 'active')
.eq('user_id', userId)

// OR фильтры
.or('status.eq.active,status.eq.pending')
```

## Auth API

### Регистрация

```typescript
const { data, error } = await supabase.auth.signUp({
  email: 'user@example.com',
  password: 'password123',
  options: {
    data: {
      full_name: 'John Doe'
    }
  }
})

if (error) {
  console.error('Sign up error:', error.message)
}
```

### Вход

```typescript
// Email/Password
const { data, error } = await supabase.auth.signInWithPassword({
  email: 'user@example.com',
  password: 'password123'
})

// Magic Link
const { error } = await supabase.auth.signInWithOtp({
  email: 'user@example.com'
})
```

### Выход

```typescript
const { error } = await supabase.auth.signOut()
```

### Получение текущего пользователя

```typescript
const { data: { user } } = await supabase.auth.getUser()
```

### Обновление пользователя

```typescript
const { data, error } = await supabase.auth.updateUser({
  email: 'new@example.com',
  password: 'newpassword',
  data: {
    full_name: 'Updated Name'
  }
})
```

### Сброс пароля

```typescript
// Отправка ссылки
const { error } = await supabase.auth.resetPasswordForEmail(
  'user@example.com',
  {
    redirectTo: 'https://example.com/reset-password'
  }
)

// Обновление пароля (после перехода по ссылке)
const { data, error } = await supabase.auth.updateUser({
  password: 'newpassword'
})
```

### Подписка на изменения авторизации

```typescript
'use client'

useEffect(() => {
  const { data: { subscription } } = supabase.auth.onAuthStateChange(
    (event, session) => {
      if (event === 'SIGNED_IN') {
        console.log('User signed in:', session?.user)
      }
      if (event === 'SIGNED_OUT') {
        console.log('User signed out')
      }
    }
  )

  return () => subscription.unsubscribe()
}, [])
```

## Storage API

### Загрузка файла

```typescript
const file = event.target.files[0]

const { data, error } = await supabase.storage
  .from('avatars')
  .upload(`public/${userId}/${file.name}`, file, {
    cacheControl: '3600',
    upsert: false
  })
```

### Получение публичного URL

```typescript
const { data } = supabase.storage
  .from('avatars')
  .getPublicUrl(`public/${userId}/avatar.jpg`)

console.log(data.publicUrl)
```

### Скачивание файла

```typescript
const { data, error } = await supabase.storage
  .from('avatars')
  .download(`public/${userId}/avatar.jpg`)
```

### Удаление файла

```typescript
const { error } = await supabase.storage
  .from('avatars')
  .remove([`public/${userId}/avatar.jpg`])
```

### Список файлов

```typescript
const { data, error } = await supabase.storage
  .from('avatars')
  .list(`public/${userId}`)
```

## Realtime API

### Подписка на изменения таблицы

```typescript
'use client'

useEffect(() => {
  const channel = supabase
    .channel('posts-changes')
    .on(
      'postgres_changes',
      {
        event: '*',  // INSERT, UPDATE, DELETE, или '*'
        schema: 'public',
        table: 'posts'
      },
      (payload) => {
        console.log('Change received:', payload)
      }
    )
    .subscribe()

  return () => {
    supabase.removeChannel(channel)
  }
}, [])
```

### Фильтрация realtime событий

```typescript
const channel = supabase
  .channel('user-posts')
  .on(
    'postgres_changes',
    {
      event: 'INSERT',
      schema: 'public',
      table: 'posts',
      filter: `user_id=eq.${userId}`
    },
    (payload) => {
      console.log('New post:', payload.new)
    }
  )
  .subscribe()
```

## RPC (Remote Procedure Call)

### Вызов PostgreSQL функции

```sql
-- Создание функции в Supabase
CREATE OR REPLACE FUNCTION get_post_count(user_uuid UUID)
RETURNS INTEGER AS $$
  SELECT COUNT(*) FROM posts WHERE user_id = user_uuid;
$$ LANGUAGE SQL;
```

```typescript
// Вызов в коде
const { data, error } = await supabase
  .rpc('get_post_count', { user_uuid: userId })

console.log('Post count:', data)
```

## Обработка ошибок

### Типичные ошибки

```typescript
const { data, error } = await supabase.from('posts').select('*')

if (error) {
  console.error('Database error:', error.message)
  // PGRST116 - не найдено
  // 42501 - нет прав (проверьте RLS)
}
```

### Best practices

Всегда проверяйте `error` перед использованием `data`.

## Типизация с TypeScript

### Использование сгенерированных типов

```typescript
import type { Database } from '@/lib/types/database'

// Типизация клиента
const supabase = createClient<Database>()

// Типы таблиц
type Profile = Database['public']['Tables']['profiles']['Row']
type ProfileInsert = Database['public']['Tables']['profiles']['Insert']
type ProfileUpdate = Database['public']['Tables']['profiles']['Update']

// Использование
const profile: Profile = {
  id: 'uuid',
  user_id: 'uuid',
  full_name: 'John Doe',
  avatar_url: null,
  created_at: '2024-01-01T00:00:00Z'
}
```

## Middleware

Middleware проверяет авторизацию на каждом запросе и перенаправляет неавторизованных пользователей на `/login`. См. `src/middleware.ts`.

## Производительность

### Оптимизация запросов

```typescript
// ❌ Плохо: N+1 запросов
const posts = await supabase.from('posts').select('*')
for (const post of posts.data) {
  const profile = await supabase
    .from('profiles')
    .select('*')
    .eq('user_id', post.user_id)
    .single()
}

// ✅ Хорошо: JOIN запрос
const { data } = await supabase
  .from('posts')
  .select(`
    *,
    profiles (
      full_name,
      avatar_url
    )
  `)
```

### Кэширование

Supabase запросы не кэшируются по умолчанию. Для fetch используйте `cache: 'force-cache'` или `next: { revalidate: 60 }`.

## Лимиты и квоты

### Supabase Free Tier

- 500 MB database
- 1 GB file storage
- 2 GB bandwidth
- 50,000 monthly active users

### Оптимизация

- Используйте `.range(0, 9)` для пагинации
- Выбирайте только нужные поля, не `select('*')`
