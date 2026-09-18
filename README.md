# Code style

## Принцип работы фронтенда

Фронтенд в проекте является реактивным.

В 90% случаев фронтенд:

- реагирует на действия пользователя
- описывает реакцию на эти действия
- не инициирует процессы самостоятельно

Базовая модель приложения — **действие → реакция**.

---

### Действие → реакция

Под действием пользователя понимается:

- клик по кнопке
- ввод данных
- выбор элемента
- изменение состояния интерфейса

Фронтенд описывает:

- что происходит после действия
- как меняется состояние
- как обновляется представление

Если логика **не является реакцией на действие пользователя** —
она должна быть явно обоснована.

---

### `Не`пользовательская логика (исключения)

Логика, не связанная напрямую с действиями пользователя, допускается
только в следующих случаях:

- интеграция с внешними (imperative) системами
- подписки / listeners
- жизненный цикл инфраструктуры

Примеры:

- WebSocket
- Leaflet / PixiJS / Three.js
- document / window listeners
- observers / timers
- аппаратные или системные события

---

### useEffect и принцип явности

`useEffect` допускается **только** для:

- управления жизненным циклом внешних систем
- подписок и отписок
- интеграции с imperative API

`useEffect` **запрещён** для:

- бизнес-логики
- вычислений
- синхронизации state ↔ state
- инициализации данных без явной причины

Если `useEffect` можно заменить:

- вычислением
- derived state
- Zustand / TanStack Query
  — `useEffect` использовать нельзя.

---

### useMemo / useCallback / memo

- использование по умолчанию запрещено
- допускается только при:
- доказанной проблеме производительности
- требованиях сторонних библиотек
- работе с imperative API

Каждое использование должно быть осознанным.
При необходимости допускается комментарий с объяснением причины.

---

### Требование к расположению скрытой логики

Любая фоновая или скрытая логика должна быть:

- максимально близко к точке входа
- логически связана с действием, которое её запускает
- легко обнаружима новым разработчиком

#### Пример

Пользователь нажал кнопку **«Подключить»**:

- кнопка вызывает хук `useConnect`
- внутри хука:
- инициализируется соединение
- запускаются listeners
- обрабатываются сообщения от backend

```jsx
// ui/ConnectButton.tsx
import { Button } from '@/shared/ui';
import { useConnect } from '@/features/connection/connect';

export const ConnectButton = () => {
  const { connect } = useConnect();

  return <Button onClick={connect}>Подключить</Button>;
};
```

```js
// model/useConnect.ts
import { useEffect } from 'react';

export const useConnect = () => {
  const connect = () => {
    // инициализация соединения
  };

  useEffect(() => {
    // запуск listener'а
    // обработка сообщений от backend
    // cleanup при отключении
  }, []);

  return { connect };
};
```

## Автоформатирование и порядок импортов

В проекте используется ESLint с плагином сортировки импортов.

В IDE обязательно должны быть включены:

- `eslint --fix` при сохранении файла
- Prettier при сохранении файла

Порядок импортов должен определяться ESLint, а не поддерживаться вручную.

Импорт локального файла стилей компонента должен находиться **ниже импортов библиотек, UI-компонентов и остальных зависимостей**.

```ts
import { Button } from 'antd';
import { CloseIcon } from '@/shared/assets/icons';

import styles from './Component.module.scss';
```

Такой порядок важен не только для единообразия: он влияет на CSS cascade. Если локальные стили импортированы выше стилей используемого UI-компонента, стили UI-библиотеки могут оказаться ниже в итоговом CSS и перебить переданный `className`.

Запрещено компенсировать неправильный порядок импортов или неправильную специфичность через `!important`.

Если возникает необходимость использовать `!important`, сначала необходимо проверить:

- порядок импортов
- специфичность селекторов
- структуру SCSS
- наличие подходящего API для стилизации у UI-компонента

`!important` не должен использоваться как средство исправления проблем cascade.

---

## Правила экспорта и импорта

В проекте запрещено использование неявных экспортов и импортов.

Придерживаемся принципа:

> Явное — лучше неявного

---

### Запрещено

- `export * from '...'`
- `import * as X from '...'`

Причины:

- ломают явность public API
- скрывают реальные зависимости
- делают границы модулей неочевидными
- усложняют поддержку и рефакторинг
- противоречат архитектуре FSD

---

### Разрешено

- только **явные именованные экспорты**
- только **явные именованные импорты**
- экспортируется только то, что является частью public API

```ts
// index.ts
export { UserCard } from './ui/UserCard';
export { useUserStore } from './model/store';
```

```ts
// в компоненте
import { UserCard } from '@/entities/user';
```

## Стили — базовые правила

#### Разрешено

> - SCSS
> - SCSS Modules
> - design tokens (scss variables)
> - mixins / functions / placeholders

#### Запрещено

> - global SCSS (кроме reset и tokens)
> - :global внутри SCSS Modules
> - styled-components
> - @emotion
> - inline styles (style={{}})
> - динамические классы через JS, если это можно решить в SCSS

### Структура стилей

```bash
shared/styles/
  ├── reset.scss
  ├── tokens/
  │   ├── colors.scss
  │   ├── spacing.scss
  │   ├── typography.scss
  │   ├── shadows.scss
  │   ├── z-index.scss
  │   └── transitions.scss
  ├── mixins/
  └── functions/
```

### Правила для компонентов

```bash
Component/
  ├── Component.tsx
  ├── Component.module.scss
  └── index.ts
```

### Использование в компонентах

```js
import styles from './Component.module.scss';

<div className={styles.root} />;
```

### Именование классов

> - один корневой класс: root
> - вложенность через SCSS
> - без BEM

```scss
.root {
  display: flex;

  .title {
  }
  .icon {
  }

  &.disabled {
  }
}
```

### Media queries

- media queries используются только через tokens / mixins
- прямые значения breakpoint'ов запрещены

### Динамика — через классы, `не через JS!`

```js
// <div style={{ opacity: disabled ? 0.5 : 1 }} /> ПЛОХАЯ ПРАКТИКА!!!


// На примере кнопки
import classNames from 'classnames';
import styles from './Button.module.scss';

type Props = {
  disabled?: boolean;
  variant?: 'primary' | 'secondary';
};

export const Button = ({ disabled, variant = 'primary' }: Props) => {
  const className = classNames(
    styles.root,
    styles[variant],
    {
      [styles.disabled]: disabled,
    }
  );

  return <button className={className} />;
};
```

### Tokens как API дизайна

> - никаких цветов/отступов напрямую
> - всё только из tokens

```scss
// color: #fff;  ПЛОХАЯ ПРАКТИКА!!!
// margin: 10px; ПЛОХАЯ ПРАКТИКА!!!

color: $text-primary;
margin: $sp-10;
```

</details>

## Нейминг — общие правила

### Файлы и папки

> - `camelCase` для файлов
> - `PascalCase` для компонентов
> - `без сокращений`
> - `без аббревиатур`, если они не общеприняты (`ws`, `api`, `ui` — ок)

```bash
entities/user/
  ├── model/
  │   ├── store.ts
  │   ├── selectors.ts
  │   └── types.ts
  ├── lib/
  │   ├── mapUserDto.ts
  │   ├── getUserInitials.ts
  │   └── formatUserName.ts
  ├── ui/
  │   ├── UserCard.tsx
  │   └── UserCard.module.scss
  └── index.ts
```

### Компоненты — имя = `что это`, а не где используется

```bash
# AdminUserTable ПЛОХАЯ ПРАКТИКА!!!
# MapLeftPanel   ПЛОХАЯ ПРАКТИКА!!!

UserTable
ControlPanel
```

### UI компоненты

- UI-компонент всегда живёт в отдельной папке
- один публичный компонент = одна папка
- index.ts — обязательный

```bash
shared/ui/Button/
  ├── Button.tsx
  ├── Button.module.scss
  └── index.ts
```

##### `shared/ui/Button/index.ts`

```js
export { Button } from './Button';
```

##### Использование

```js
import { Button } from '@/shared/ui';
```

### Хуки — одно действие / ответственность

```bash
useLogin
useHeartbeatSocket
useMapMarkers
```

### Zustand stores — имя отражает домен, не страницу

> Структура имени: `useXxxStore`

```bash
useUserStore
useMapStore
useRulerStore
```

### Переменные

> - boolean → is / has / can
> - функции → глагол

```bash
isLoading
hasError
canEdit

fetchUsers
resetState
```

</details>

</details>

## Public API

## Главное правило — импорт ТОЛЬКО через public API (`index.ts`)

```js
// import { Button } from '@/shared/ui/Button/Button';  ПЛОХАЯ ПРАКТИКА!!!
import { Button } from '@/shared/ui';
```

### Ограничения public API

- запрещено экспортировать всё подряд
- public API должен быть осознанным
- лишние экспорты считаются архитектурной ошибкой
  > если есть сомнение, экспортировать или нет — **не экспортируем**.

> `entities/user/index.ts`

```js
// ПЛОХАЯ ПРАКТИКА!!!
export * from './model'; // можем экспортировать только 1 конкретный стор
export * from './ui'; // можем экспортировать только 1 конкретный ui компонент
export * from './lib'; // утилиты можно экспортировать только на слое shared

// Хорошая практика
export { useUserStore } from './model';
export { UserCard } from './ui';
```

### Примеры public API

#### Кнопка `shared/ui`

> структура

```bash
shared/ui/
  ├── Button/
  │   ├── Button.tsx
  │   ├── Button.module.scss
  │   └── index.ts
  ├── Input/
  ├── Modal/
  └── index.ts
```

> `shared/ui/index.ts`

```js
export { Button } from './Button';
export { Input } from './Input';
```

---

#### Сущность карточки пользователя `entities/user`

> структура

```bash
entities/user/
  ├── model/
  │   ├── store.ts
  │   ├── selectors.ts
  │   └── types.ts
  ├── lib/
  │   ├── mapUserDto.ts
  │   ├── getUserInitials.ts
  │   └── formatUserName.ts
  ├── ui/
  │   ├── UserCard.tsx
  │   └── UserCard.module.scss
  └── index.ts
```

> `entities/user/index.ts`

```js
export * from './model';
export * from './ui';
```

### Что НЕ экспортируем

> - внутренние утилиты
> - временные хелперы
> - private хуки
> - scss файлы

**_Всё, что не экспортировано — считается приватным!_**

### Aliases (важно для FSD)

```bash
@/app
@/pages
@/widgets
@/features
@/entities
@/shared
```

#### Запрет:

> - ../../../../
> - прямые пути внутрь слоёв

### Naming внутри SCSS

> - root — всегда
> - состояния → .disabled, .active, .error
> - варианты → .primary, .secondary

**_Без BEM, без \_\_, без --._**

</details>

## Gitflow

### Conventional Commits

#### Формат коммита

```bash
<type>(<scope>): <subject> # заголовок

<body>                     # тело
```

#### Заголовок

> - не более 50 символов
> - в настоящем времени
> - без точки в конце
> - описывает что сделано, а не как

```bash
feat(auth): add login form
fix(map): prevent cluster overlap
refactor(ws): extract heartbeat handler
```

#### Scope (в скобках)

> - для домена
> - для фичи
> - для слоя
> - для зоны слоя (model / lib / ui / store)

```bash
feat(auth)
fix(ws)
refactor(shared-ui)
```

#### Тело коммита (опционально)

> - если заголовок невозможно вписать в 50 символов
> - если есть:
>   - неочевидная логика
>   - побочные эффекты
>   - причины решения (почему, не как)

##### Формат body

> - отделяется пустой строкой
> - без ограничения по длине

```bash
fix(ws): handle reconnect on token refresh

Reconnects socket after auth token update.
Prevents missing heartbeat events after relogin.
```

#### Типы коммитов (семантика)

##### `feat`

> - добавляется новая функциональность
> - расширение существующей функциональности

##### `fix`

> - исправления багов
> - исправления некорректного поведения

##### `refactor`

> - изменение внутренней структуры
> - улучшение архитектуры
> - упрощение кода

##### `chore` — не влияет на runtime

> - обновление зависимостей
> - конфиги
> - CI
> - скрипты

##### `docs`

> - README
> - архитектурные документы
> - комментарии

##### `test`

> - добавление или правка тестов
> - без изменения логики

### Merge Request

#### Перед отправкой MR разработчик ОБЯЗАН:

> - подтянуть актуальный develop
> - конфликты решает автор MR
> - выполнить yarn build и сразу исправить ошибки, если таковые возникнут
> - убедиться, что:
>   - нет нарушений FSD
>   - нет нарушений Code Style
>   - импорты только через public API
>   - удостовериться в отсутствии конфликтов перед отправкой ссылки MR на проверку

#### Title

```bash
<type>: <task-id> <краткое описание>
```

> - соответствует Conventional Commits
> - ≤ 50 символов
> - без лишних слов
> - type: feat | fix | refactor | chore | docs | test

##### Примеры

```bash
feat: 9790 task
fix: 10231 ws heartbeats
refactor: 11002 map store
```

#### Description — должно отвечать на вопросы:

> - какую задачу решаем
> - сколько времени ушло
> - демо (скриншоты или запись экрана)
> - что именно сделано
> - как проверить (опционально, если что-то специфическое)

#### MR Template (копировать и использовать)

```md
# Задача

[№ <TASK_ID_BITRIX>](TASK_URL_BITRIX): <Краткое описание задачи из битрикса>

# Затрачено времени

<например: 3ч>

# Демо

<!-- Скриншоты / видео / gif -->
<!-- Если не требуется — удалить раздел -->

# Проведены следующие работы

- [x] <Кратко и по делу>
- [x] <Что именно изменено>
- [x] <Без воды и повторов задачи>

# Как проверить (опционально)

1. <Шаг 1>
2. <Шаг 2>
3. <Ожидаемый результат>
```

#### Чеклисп перед отправкой MR на проверку

- [ ] MR не содержит несвязанных изменений
- [ ] MR решает одну задачу

</details>
