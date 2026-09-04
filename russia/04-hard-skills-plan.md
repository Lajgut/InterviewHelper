# Блок 4 · 💪 Хард-скиллы: план подготовки (Revolut / Яндекс / Ozon)

> Целевые компании: **Revolut, Яндекс, Ozon Tech**. Роли: Android Team Lead / Senior.
> Обновлено: 2026-08-31

---

## ❓ Одинаковая ли база для лида и сеньора?

**ДА, одинаковая.** Технические секции у лида и сеньора в этих компаниях не различаются: те же алгоритмы, тот же Android core, тот же system design. Разница:

| Аспект | Senior (IC) | Team Lead |
|--------|-------------|-----------|
| Алгоритмы | те же | те же |
| Android core | те же | те же |
| System Design | те же | те же |
| **Менеджерская секция** | ❌ нет | ✅ есть (команда, конфликты, процессы) |
| **Ожидаемая глубина** | глубокие детали | глубина + широта, зрелость решений |
| Порог прохода | чуть выше по технике | чуть мягче по технике, но с лид-компетенциями |

→ **Готовимся по одной базе.** Менеджерскую секцию добавляем отдельным блоком (для лида). Revolut интересный случай: там лид-роли часто ближе к strong IC.

---

## 🎯 Специфика твоих трёх компаний (проверено, 2025–2026)

### Revolut — 5–6 этапов, всё на английском ⚠️
1. Recruiter screen (мотивация + базовые концепции)
2. **Online assessment / Take-home** (HackerRank DS&A или задача 4–8 часов)
3. **Live coding: мини-приложение** — UI + обработка данных (RecyclerView, Jetpack, Kotlin, coroutines/Flow, clean architecture)
4. Ревью take-home: защита решений, trade-offs (threading, DI, тесты)
5. **Mobile System Design**: приложение end-to-end (offline sync, кэширование, модульность, сеть)
6. Team fit / Hiring manager

**Особенности:** темп быстрый (раунды бэк-ту-бэк), смотрят на подход, а не заученность. **Английский обязателен** — Revolut не русскоязычный трек, это про Дубай/intl. До уровня языка — готовимся к Яндексу/Ozon первыми.

### Яндекс — новый формат секций (важно!)
Порядок сменился: **сначала платформенная секция (AC), потом алгоритмы (AA)**, для senior+ — System Design.
1. Тех. скрининг
2. **Платформенная секция Android** (Kotlin, Compose, lifecycle, многопоточность)
3. **Алгоритмическая секция** (задача на СД/алгоритмы с кодом, можно на бумаге)
4. System Design (senior+)
5. Финал (для лида — менеджерская)

**Материалы от самого Яндекса:**
- [Как устроено собеседование в Яндекс (мобайл)](https://enigmai.ru/interview/yandex/mobile-engineering/) — разбор с примерами кода
- [Банк вопросов: 137 реальных вопросов Android в Яндекс](https://h.careers/interview-questions/android-developer/company/yandeks)
- [Новый формат секций — обсуждение на Хабре](https://habr.com/ru/articles/882030/comments/)
- YouTube: [Собеседование Яндекс. Платформа Android](https://www.youtube.com/watch?v=dVzym6I-K1U) · [Собеседование Яндекс. Алгоритмы](https://www.youtube.com/watch?v=tfvm2k5c9JI)

### Ozon Tech
1. Скрининг
2. 2–3 технические секции (алгоритмы + Android core)
3. **System Design**
4. Финал / менеджерская (для лида)

**Материалы:**
- [Опыт собеседования Ozon Senior (LinkedIn, разбор этапов)](https://www.linkedin.com/posts/oleg-tschumin_my-interview-experience-with-ozon-senior-activity-7343550683196911618-IJAg)
- [Ozon Senior SDE — реальные вопросы (swe180)](https://www.swe180.com/interview-experience/ozon/senior-sde)
- [Ozon Android — Glassdoor](https://www.glassdoor.com/Interview/OZON-ru-Junior-Android-Developer-Interview-Questions-EI_IE626074.0,7_KO8,32.htm)

**Пересечение трёх компаний:** везде алгоритмы + Android core + System Design. Готовишь одну базу — закрываешь все три.

---

## 📚 Программа подготовки (6 недель)

### Неделя 1–2 · 🔤 Kotlin + Coroutines/Flow (фундамент)

**Kotlin (спрашивают все трое):**
- [ ] Null-safety, smart-casts, `let/run/apply/also/with` — когда что
- [ ] Функции высшего порядка, lambda, inline/noinline/crossinline
- [ ] reified, дженерики, variance (`in/out`), star-projection
- [ ] Extension functions/properties, operator overloading
- [ ] Data/sealed classes, objects, companion, интерфейсы с дефолтами
- [ ] Delegates (by lazy, observable, custom), property delegation
- [ ] Scope functions, результат выражений, Elvis
- [ ] Коллекции: последовательные vs ленивые (Sequence), операции
- [ ] `==` vs `===`, equals/hashCode/toString контракты
- [ ] Исключения: checked-подход Kotlin, Result, runCatching

**Coroutines + Flow (топ-тем вопросов):**
- [ ] Structured concurrency, CoroutineScope, Job, SupervisorJob
- [ ] suspend — что реально происходит (CPS, state machine)
- [ ] Dispatchers (Default/IO/Main/Main.immediate), withContext
- [ ] Cancellation: cooperate, ensureActive, NonCancellable, cleanup
- [ ] Exceptions: CoroutineExceptionHandler, try/catch в launch vs async
- [ ] async/await, awaitAll, отмена родителя→детей
- [ ] Channels, Select, Mutex/Semaphore
- [ ] **Flow:** cold/hot, операторы (map/filter/combine/zip/flatMapConcat…)
- [ ] **StateFlow vs SharedFlow vs LiveData** — различия, use-cases (спрашивают ВСЕ)
- [ ] Flow: buffer, conflate, collectLatest, debounce/distinctUntilChanged
- [ ] callbackFlow/channelFlow, тестирование (Turbine, runTest)

**Практика:** 1–2 задачи в день на Kotlin (прогрев под алгоритмы параллельно).

### Неделя 3 · 🎨 Jetpack Compose + UI

- [ ] Declarative vs imperative — суть и следствия
- [ ] Recomposition: когда, почему, smart skips, stability
- [ ] **Stable/Unstable типы, @Stable/@Immutable, ImmutableList** (твоя weak-spot тема!)
- [ ] remember, rememberSaveable, derivedStateOf
- [ ] State hoisting, single source of truth
- [ ] side-effect API: LaunchedEffect, DisposableEffect, SideEffect
- [ ] Modifier: порядок важен, custom modifiers
- [ ] Layout: Column/Row/Box, constraints, custom layout
- [ ] Lists: LazyColumn, keys, contentPadding
- [ ] Compose + Navigation, ViewModel integration
- [ ] Производительность Compose (в связке с твоей темой #1)
- [ ] Интероп: AndroidView, Views в Compose

### Неделя 4 · 🤖 Android Core (глубина)

- [ ] Lifecycle: Activity/Fragment, saved state, process death
- [ ] ViewModel: scope, onCleared, kotlinx-serialization SavedStateHandle
- [ ] Запуск приложения: что происходит от тапа до первого кадра (твоя тема! 23s→13s кейс)
- [ ] **MainThread, Looper, Handler, MessageQueue** — механика
- [ ] Context: виды, утечки через context (тема утечек)
- [ ] Process/Task/Backstack, launchMode, intent flags
- [ ] Services (started/bound/foreground), WorkManager vs AlarmManager
- [ ] Транзакции фрагментов, FragmentFactory, общие ошибки
- [ ] PendingIntent, Broadcasts, permissions (runtime, special)
- [ ] Хранение: SharedPreferences vs DataStore, Room (сущности, миграции, потоки)
- [ ] Сеть: OkHttp перехватчики, кэш, Retrofit, сериализация
- [ ] Изображения: Glide/Coil — загрузка, кэш, даунсемплинг
- [ ] Память: GC, leaks, LeakCanary, Memory Profiler (тема #1!)
- [ ] Doze/App Standby, battery, фоновые ограничения

### Неделя 5 · 🏗 Архитектура + System Design

**Архитектура:**
- [ ] MVVM vs MVI vs MVP — trade-offs, когда что
- [ ] Clean Architecture: слои, dependency rule, когда избыточна
- [ ] **Многомодульность**: by feature vs by layer, api/impl, границы
- [ ] DI: Hilt/Dagger — components, scopes, почему не Koin для больших
- [ ] Offline-first, single source of truth, синхронизация
- [ ] Feature flags, релизные поезда (твой опыт ВК!)
- [ ] Тестирование: unit/ui/integration, test doubles, архитектура для тестируемости

**System Design Mobile (главный фильтр senior+):**
Фреймворк ответа: **requirements → estimation → high-level → детализация клиента → trade-offs**

Типовые задачи (прорешать вслух по 3–5):
- [ ] Лента новостей / соцсеть
- [ ] Мессенджер (offline, доставка, статусы)
- [ ] E-commerce каталог+корзина (Ozon любит)
- [ ] Финтех-приложение: авторизация, безопасность, транзакции (Revolut)
- [ ] Стриминг видео (твой профиль ВК Видео!)
- [ ] SDK аналитики (твой опыт!)
- [ ] Голосовой ассистент (твой опыт Маруси!)

Ключевые темы внутри: модульность, offline sync, кэш-стратегии, пагинация, retry/idempotency, безопасность (token storage, certificate pinning), мониторинг.

### Неделя 6 · 🧮 Алгоритмы + финальная полировка

**Яндекс-специфика:** уровень задач — LeetCode Medium, иногда Hard; можно готовиться «на бумаге».

- [ ] Массивы/строки: two pointers, sliding window, prefix sums
- [ ] Hash maps/sets: частоты, группировки
- [ ] Стеки/очереди, monotonic stack
- [ ] Linked lists: reverse, merge, cycle
- [ ] Бинарный поиск и вариации
- [ ] Деревья: BFS/DFS, BST, LCA
- [ ] Графы: BFS/DFS, топологическая сортировка, union-find
- [ ] Базовое DP: knapsack, LIS, edit distance
- [ ] Жадные алгоритмы, intervals
- [ ] Сортировки: знать сложность, quickselect

**Норматив:** Medium за 20–25 минут с проговариванием вслух.

**Ресурсы:** LeetCode (top 100 easy→medium), [NeetCode 150](https://neetcode.io/practice), Яндекс.Контесты (для калибровки под их стиль).

---

## 🗓️ Связка с твоими weak-spots

| Тема | Где в плане | Файл |
|------|-------------|------|
| Performance & Profiling | Неделя 4 (память) + System Design | [weak-spots/01](../weak-spots/01-performance-profiling.md) |
| Метрики/RUM | System Design (мониторинг) + рассказ о ВК | [weak-spots/02](../weak-spots/02-performance-analytics-rum.md) |

**Твои козырные кейсы (вставлять везде, где уместно):**
- Холодный старт 23с→13с (Revolut/Ozon: performance, profiling, приоритизация)
- Device tiering + jank-метрики (monitoring, RUM)
- 10 AOSP-вендоров (широтa, системная экспертиза — редкость!)
- Backend-driven UI в ВТБ (system design, архитектура)
- TVT-прирост (продуктовое мышление)

---

## ✅ Чек-лист готовности «выходить на интервью»

- [ ] Решаю LC Medium за 20–25 мин вслух (10+ задач подряд стабильно)
- [ ] Объясняю StateFlow vs SharedFlow без подготовки
- [ ] Раскладываю Compose recomposition + stability на пальцах
- [ ] Прорешал 5+ mobile system design задач вслух с таймером
- [ ] Рассказываю кейс 23s→13s за 3 минуты (проблема→анализ→решение→результат)
- [ ] Прошёл 2+ mock-интервью ([it-interview.io](https://it-interview.io/mock-interview) или с коллегой)

---
_Связанные файлы: [Блок 2 — общий план собеседований](./02-interview-prep.md) · [Weak spots](../weak-spots/README.md) · [Истории для интервью](../interview-stories.md)_
