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

### Изоляция стилей компонента

SCSS Modules не должны содержать широкие незаанкоренные селекторы по HTML-тегам.

```scss
// ПЛОХАЯ ПРАКТИКА!!!
section h3 {
  margin: 0;
}

header h2 {
  font-size: 20px;
}
```

`*.module.scss` локализует имена CSS-классов, но не делает HTML-теги локальными. Такой селектор может задеть чужую разметку.

Все стили компонента должны быть заанкорены на локальный класс, начиная с `.root`, и по возможности обращаться к элементам через локальные классы.

```scss
.root {
  .section {
    .title {
      margin: 0;
    }
  }
}
```

```tsx
<section className={styles.section}>
  <h3 className={styles.title}>Заголовок</h3>
</section>
```

Нежелательно:

```scss
.root {
  .sourceText span {
    font-size: 14px;
  }
}
```

Предпочтительно:

```scss
.root {
  .sourceText {
    .label {
      font-size: 14px;
    }
  }
}
```

При ревью сгенерированных моделью стилей отдельно проверяются:

- широкие селекторы по HTML-тегам
- выход стилей за границы компонента
- избыточная специфичность
- cascade
- появление `!important`

---

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

### Использование design tokens без подмены дизайна

Design token используется, если в дизайн-системе существует **точное соответствующее значение**.

Нельзя заменять специфичное значение на приблизительно похожий token только ради формального соблюдения правила.

Особенно это относится к специфичным для карты:

- цветам
- теням
- opacity
- размерам и значениям, связанным с рендерингом MapLibre / deck.gl

Правило:

- есть точный token → использовать token
- точного token нет → допускается локальное специфичное значение
- нельзя менять внешний вид или runtime-поведение только ради формального рефакторинга

---

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

### Zustand store должен называться Store

Если сущность является Zustand store, это должно быть явно отражено в имени.

```ts
// ПЛОХАЯ ПРАКТИКА!!!
useMapLibreCommentCreateState;

// Хорошая практика
useMapLibreCommentCreateStore;
```

Имя не должно маскировать store под обычный state или helper.

---

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

### Boolean naming — состояние, setter — действие

Для boolean используются семантические префиксы:

- `is`
- `are`
- `has`
- `can`

```ts
// ПЛОХАЯ ПРАКТИКА!!!
showComments;
setShowComments;

// Хорошая практика
areCommentsVisible;
setCommentsVisible;
```

Setter должен называться как действие, а не повторять имя boolean-переменной механически.

### Нейминг ассетов

Названия ассетов должны отражать смысл и не содержать неочевидных сокращений.

```text
 ПЛОХАЯ ПРАКТИКА!!!
commentCursorSat.svg
commentCursorSchem.svg

 Хорошая практика
commentCursorHybrid.svg
commentCursorScheme.svg
```

Используются только общепринятые сокращения. Если сокращение требует расшифровки — пишется полное слово.

---

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

### Строгий public API

Public API должен содержать только то, что действительно используется снаружи slice / модуля.

Не следует экспортировать:

- внутренние типы только «на всякий случай»
- editor / tooltip / helper-компоненты, используемые только внутри реализации
- детали внутреннего состояния
- вспомогательные типы WebSocket / MapLibre, если они не являются внешним контрактом

Пример: внутренний `EventsSocketEventMeta` не должен попадать в `index.ts`, если он не используется потребителями модуля.

Если компонент нужен только внутри публичного компонента, он остаётся приватным даже при наличии собственной папки:

```text
MapLibreCommentsLayer/
CommentEditor/
CommentTooltip/
```

Наличие отдельной папки и `index.ts` внутри локальной структуры не означает, что компонент необходимо поднимать в public API slice.

---

> **Важно:** правило явных именованных экспортов имеет приоритет над любыми старыми примерами ниже. Если в примере встречается `export *`, это не является разрешением на wildcard-export. В актуальном коде используются только явные именованные экспорты.

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

### FSD boundaries

Зависимости между слоями должны быть направлены вниз.

Базовая матрица:

```text
app      → pages / widgets / features / entities / shared
pages    → widgets / features / entities / shared
widgets  → features / entities / shared
features → entities / shared
entities → shared
shared   → не знает о доменных слоях
```

Импорт из вышележащего слоя запрещён.

Отдельно запрещается зависимость `feature → feature`, если ответственность можно корректно разделить через `entities`, `shared`, композицию на уровне `widgets/pages` или публичные входные параметры.

Необходимо избегать:

- cross-import между независимыми features
- прямых импортов внутренних файлов соседнего slice
- циклических зависимостей
- переноса доменной логики в `shared` только ради обхода FSD

---

### Naming внутри SCSS

> - root — всегда
> - состояния → .disabled, .active, .error
> - варианты → .primary, .secondary

**_Без BEM, без \_\_, без --._**

</details>

## Рефакторинг без изменения поведения

Если задача заключается в code-style / FSD / structural refactoring, она должна быть **behaviour-preserving**.

Без отдельной причины и отдельной задачи нельзя одновременно менять:

- WebSocket flow
- TTL и время жизни событий
- batching
- stale-event protection
- API-контракты
- Zustand state model
- MapLibre / deck.gl rendering
- геометрию overlay
- CSS-значения, влияющие на внешний вид
- пользовательские сценарии

Рефакторинг стиля не должен становиться скрытой функциональной переработкой.

Если функциональное изменение необходимо, оно должно быть явно выделено и обосновано.

---

## Imperative lifecycle: MapLibre / deck.gl / listeners

Для imperative-интеграций должен существовать понятный owner жизненного цикла.

Всегда должно быть очевидно:

- кто создаёт объект / overlay / listener
- кто обновляет его
- кто вызывает `setProps`
- кто запускает `requestAnimationFrame`
- кто останавливает `requestAnimationFrame`
- кто выполняет cleanup
- кто удаляет source / layer / image / listener

Не допускаются несколько конкурирующих owner'ов одного lifecycle.

Для MapLibre / deck.gl необходимо отдельно проверять:

- source / layer / image не создаются повторно без необходимости
- overlay не пересоздаётся при каждом render
- cleanup симметричен initialization
- React state не обновляется каждый кадр без реальной необходимости
- `requestAnimationFrame` не живёт после unmount / отключения режима
- imperative API не используется как скрытый второй store

---

## Техническая гигиена

Перед завершением задачи необходимо проверить отсутствие:

- silent `catch`
- необоснованных `any`
- необоснованных `as` / type assertions
- dead code
- magic values без объяснимого смысла
- лишних hooks
- лишних public exports
- необоснованных SSR guards
- дублирующей логики
- комментариев, которые пересказывают очевидный код вместо причины решения

`catch` не должен молча поглощать ошибку без обоснованной причины.

Magic value допускается только если его смысл очевиден из контекста либо значение локально и действительно не заслуживает отдельной константы. Повторяющиеся или доменно значимые значения должны быть именованы.

Type assertion допускается только там, где тип нельзя корректно вывести или сузить средствами TypeScript/API. Assertion не используется для подавления ошибки типов.

---

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
