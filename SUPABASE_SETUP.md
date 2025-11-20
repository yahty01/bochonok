# 🔧 Настройка Supabase для Bochonok

Этот документ поможет вам настроить Supabase для работы с вашим приложением для ведения личного бюджета.

## 📋 Пошаговая инструкция

### 1. Создание проекта Supabase

1. Перейдите на [supabase.com](https://supabase.com)
2. Нажмите "Start your project"
3. Войдите в систему или создайте аккаунт
4. Нажмите "New project"
5. Выберите организацию
6. Заполните данные проекта:
   - **Name**: Bochonok (или любое другое название)
   - **Database Password**: создайте надежный пароль
   - **Region**: выберите ближайший к вам регион
7. Нажмите "Create new project"

### 2. Получение ключей API

После создания проекта:

1. Перейдите в **Settings** → **API**
2. Найдите секцию **Project API keys**
3. Скопируйте следующие значения:
   - **Project URL** (URL вашего проекта)
   - **Project API keys** → **anon public** (публичный ключ)
   - **Project API keys** → **service_role** (приватный ключ, опционально)

### 3. Настройка переменных окружения

1. В корне проекта создайте файл `.env.local`:
   ```bash
   cp .env.example .env.local
   ```

2. Откройте `.env.local` и заполните переменные:
   ```env
   NEXT_PUBLIC_SUPABASE_URL=https://your-project-ref.supabase.co
   NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key-here
   SUPABASE_SERVICE_ROLE_KEY=your-service-role-key-here
   ```

### 4. Настройка аутентификации

1. В панели Supabase перейдите в **Authentication** → **Settings**
2. В разделе **Site URL** добавьте:
   ```x
   http://localhost:3000
   ```
3. В разделе **Redirect URLs** добавьте:
   ```
   http://localhost:3000/auth/callback
   ```

### 5. Создание первой таблицы (опционально)

Создайте тестовую таблицу для проверки подключения:

1. Перейдите в **Table Editor**
2. Нажмите "Create a new table"
3. Название: `profiles`
4. Добавьте колонки:
   - `id` (uuid, primary key, default: gen_random_uuid())
   - `user_id` (uuid, foreign key to auth.users)
   - `full_name` (text)
   - `avatar_url` (text)
   - `created_at` (timestamptz, default: now())

### 6. Настройка Row Level Security (RLS)

Для безопасности включите RLS:

```sql
-- Включить RLS для таблицы profiles
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;

-- Политика: пользователи могут видеть только свои профили
CREATE POLICY "Users can view own profile" ON profiles
  FOR SELECT USING (auth.uid() = user_id);

-- Политика: пользователи могут обновлять только свои профили
CREATE POLICY "Users can update own profile" ON profiles
  FOR UPDATE USING (auth.uid() = user_id);
```

### 7. Обновление TypeScript типов

После создания таблиц обновите файл `src/lib/types/database.ts`:

```typescript
export type Database = {
  public: {
    Tables: {
      profiles: {
        Row: {
          id: string
          user_id: string
          full_name: string | null
          avatar_url: string | null
          created_at: string
        }
        Insert: {
          id?: string
          user_id: string
          full_name?: string | null
          avatar_url?: string | null
          created_at?: string
        }
        Update: {
          id?: string
          user_id?: string
          full_name?: string | null
          avatar_url?: string | null
          created_at?: string
        }
      }
    }
    Views: {
      [_ in never]: never
    }
    Functions: {
      [_ in never]: never
    }
    Enums: {
      [_ in never]: never
    }
    CompositeTypes: {
      [_ in never]: never
    }
  }
}
```

## 🧪 Тестирование подключения

1. Запустите проект:
   ```bash
   npm run dev
   ```

2. Откройте http://localhost:3000

3. Попробуйте зарегистрировать нового пользователя

4. Проверьте в Supabase **Authentication** → **Users**, что пользователь был создан

## 🔍 Автоматическая генерация типов

Для автоматической генерации TypeScript типов установите Supabase CLI:

```bash
npm install -g supabase
supabase login
supabase init
supabase link --project-ref your-project-ref
supabase gen types typescript --local > src/lib/types/database.ts
```

## 🚀 Дополнительные возможности

### Настройка провайдеров OAuth

В **Authentication** → **Providers** вы можете настроить:
- Google
- GitHub  
- Discord
- Facebook
- И многие другие

### Настройка Storage

Для загрузки файлов:
1. Перейдите в **Storage**
2. Создайте новый bucket
3. Настройте политики доступа

### Настройка Edge Functions

Для серверной логики:
1. Перейдите в **Edge Functions**
2. Создайте новую функцию
3. Разверните код

## ❓ Часто задаваемые вопросы

**Q: Почему я получаю ошибку "Invalid API key"?**
A: Проверьте правильность ключей в `.env.local` и перезапустите сервер разработки.

**Q: Пользователи не могут зарегистрироваться**
A: Проверьте настройки Site URL и Redirect URLs в Authentication Settings.

**Q: Как сбросить пароль базы данных?**
A: В Settings → Database найдите опцию "Reset database password".

## 📞 Поддержка

- [Документация Supabase](https://supabase.com/docs)
- [Сообщество Discord](https://discord.supabase.com)
- [GitHub Issues](https://github.com/supabase/supabase/issues)
