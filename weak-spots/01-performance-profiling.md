# 🧩 Слабое место #1 · Оптимизация производительности и профилирование

> **Спрашивают:** ⭐⭐⭐ Очень часто (Senior/Lead — обязательно)
> **Статус:** 🟡 В работе → цель 🟢 Готово
> Обновлено: 2026-08-06

---

## 📌 Что реально спрашивают на интервью

Это **топовая тема** для senior/lead. Чаще всего в трёх форматах:

1. **Открытый вопрос:** «Как бы ты нашёл и устранил проблему производительности в приложении?» / «Приложение тормозит / лагает — твои действия?»
2. **Конкретика по областям:** память (утечки), запуск (startup), UI-плавность (jank), Compose recomposition, батарея.
3. **Инструменты:** «Какими инструментами профилируешь?», «Что такое Baseline Profiles?»

> 💡 **Главный совет:** отвечай по **методологии**, а не россыпью фактов. Интервьюер хочет видеть системность: *измерить → найти узкое место → исправить → измерить снова*.

---

## 🧭 Карта темы (5 направлений + инструменты)

```
Производительность Android
├── 1. Запуск приложения (App Startup)
├── 2. UI-плавность (Jank / Frame rate / 16ms)
├── 3. Память (Memory leaks, GC pressure, OOM)
├── 4. Compose-специфика (Recomposition)
├── 5. Батарея / сеть / фоновые задачи
└── 🛠️ Инструменты: Profiler, Macrobenchmark, Perfetto, LeakCanary, Baseline Profiles
```

---

## 1️⃣ Запуск приложения (App Startup)

### Типы запуска
- **Cold start** — процесс приложения не существует (с нуля). Главный показатель.
- **Warm start** — процесс существует, но активити надо создать.
- **Hot start** — всё есть, просто выводим на передний план.

### Что оптимизировать
- `Application.onCreate()` — убрать тяжёлые инициализации. Использовать **App Startup library** (Jetpack) или **Lazy initialization**.
- Отложить не-критичные SDK (аналитика, push) — фоновые воркеры / WorkManager.
- **Baseline Profiles** 🔥 — главный современный инструмент: предкомпиляция критических путей кода при установке → **ускорение старта до 30%** (Duolingo кейс).
- Избегать I/O и сети на главном потоке при старте.

### Метрики
- **Time to Initial Display (TTID)** — когда отрисован первый кадр.
- **Time to Full Display (TTFD)** — когда контент готов к взаимодействию (`reportFullyDrawn()`).

---

## 2️⃣ UI-плавность (Jank / Frame Rate)

### Принцип 60 FPS
- **16 ms на кадр** (при 60 Гц). Не уложился → **jank** (дроп кадра = лаг).
- На 120 Гц — **8 ms**. Ещё жёстче.

### Главные причины jank
- Тяжёлые вычисления на главном потоке (сортировка, парсинг, JSON).
- Сложная/глубокая иерархия View → долгие `measure`/`layout`.
- Overdraw (рисование поверх уже нарисованного).
- Чрезмерная валидация `requestLayout()` / `invalidate()`.
- Большие bitmap в памяти без масштабирования.

### Что отвечать на «приложение лагает»
1. Включить **Profile GPU Rendering** / **GPU overdraw** (в Developer options).
2. В Profiler (CPU) найти кадры >16ms, посмотреть trace.
3. **Perfetto** / **systrace** — детальный trace по потокам.
4. Убрать работу с главного потока, упростить layout, кэшировать bitmap.

---

## 3️⃣ Память (Memory) — ⭐ самая частая подтема

### Утечки памяти (Memory Leaks)
**Что такое:** объект удерживается в памяти дольше, чем нужно, потому что на него есть сильная ссылка.

**Частые причины:**
- **Static-ссылка на `Context` / `View` / `Activity`** — классика.
- **Незарегистрированные слушатели / BroadcastReceiver / Observer** — не отписался.
- **Inner (non-static) класс + внешний контекст** — анонимный класс `Runnable`, удерживающий Activity.
- **Singleton удерживает Activity** (через поле или listener).
- **Coroutines**: scope пережил Activity (GlobalScope / неправильно отменённый Job).
- **View в Compose через AndroidView** при неправильной очистке.

### Как находить
- **LeakCanary** 🔥 — стандарт де-факто, автоматически ловит утечки в debug.
- **Memory Profiler** в Android Studio → Heap dump → искатьretained объекты по пути GC Roots.
- **Allocations tracking** — кто и сколько аллоцирует.

### OOM (Out Of Memory)
- Часто от bitmap — грузить через **Glide/Coil** с даунсемплингом (`inSampleSize`).
- Пула connection/bitmap — освобождать в `onDestroy`/`onCleared`.
- **Memory pressure / GC pressure** — частые/alлокации → частый GC → jank. Решение: переиспользование объектов, primitive массивы, избегать autoboxing.

---

## 4️⃣ Compose-специфика (Recomposition) 🔥

Современная топ-тема. Обязательно спросят у senior.

### Суть проблемы
Compose перерисовывает (recomposes) функции при изменении состояния. Лишние recomposition = jank и батарея.

### Ключевые концепции
- **Smart recomposition** — Compose пытается пересчитать только изменённые функции.
- **Stable vs Unstable параметры:** если параметр `unstable` (например, `List` вместо `ImmutableList`), Compose не может гарантировать неизменность → пересчитывает.
- **`@Stable` / `@Immutable`** аннотации — помогают Compose пропускать recomposition.
- **Compose Compiler Metrics** — генерирует отчёт стабильности, смотреть какие параметры unstable.

### Приёмы оптимизации
- Использовать `remember` / `derivedStateOf` — чтобы производные состояния не пересчитывались.
- `key()` — стабильные ключи в списках.
- Передавать примитивы / @Stable-классы вместо «голых» List/Map.
- `ImmutableList` (Kotlinx collections) вместо `List`.
- Избегать лямбд-allocations в горячих местах — `remember { }` для callback'ов.
- **Layout Inspector** + **Recomposition Counts** — визуально увидеть, что пересчитывается лишний раз.

> 🔑 **Частый вопрос:** «Как минимизировать recomposition в Compose?» → перечислить: стабильные типы, remember/derivedStateOf, key, ImmutableList, Profiler.

---

## 5️⃣ Батарея / сеть / фон

- **WorkManager** — для отложенных/гарантированных фоновых задач.
- **Doze mode / App Standby** — система ограничивает фоновую активность.
- Избегать wake locks, частых polling'ов.
- Батчить сетевые запросы, использовать кэш, **FCM** для пушей вместо polling.
- **Battery Historian** — анализ расхода батареи.

---

## 🛠️ Инструменты (выучи названия — спрашивают!)

| Инструмент | Для чего | Уровень |
|-----------|----------|---------|
| **Android Studio CPU Profiler** | Flame chart, метод по методу — где время | Базовый |
| **Android Studio Memory Profiler** | Heap dump, allocations, утечки | Базовый |
| **LeakCanary** | Авто-поиск утечек (debug) | Базовый, обязательно |
| **Layout Inspector** | Иерархия View + **Recomposition counts** (Compose) | Базовый |
| **Macrobenchmark** (Jetpack) | Замеры startup, scroll в CI | Современный, важно |
| **Baseline Profiles** 🔥 | Предкомпиляция кода → +30% startup | Современный, важно |
| **Perfetto** | Системный trace (пришёл на смену systrace) | Продвинутый |
| **Android Performance Analyzer (APA)** | Новый объединённый профайлер CPU/GPU/mem/power | Новинка 2024–2025 |
| **GPU Overdraw / Profile GPU Rendering** | Отрисовка, overdraw, jank в Dev options | Базовый |

---

## 🗣️ Как отвечать (универсальный алгоритм)

Если спрашивают «как найти/починить проблему производительности» — отвечай по шагам:

> **1. Воспроизвести и измерить.** Сначала метрики: startup time, frame rate, heap size. Без цифр — не оптимизируем («premature optimization»).
> **2. Профилировать.** Профайлером (CPU/Memory) или Macrobenchmark нахожу узкое место. Не гадаю «наверное тут».
> **3. Найти root cause.** Flame chart → конкретный метод / аллокация / рекомпозиция.
> **4. Исправить целевым образом.** Убрать с главного потока, кэшировать, стабилизировать типы, Baseline Profile и т.д.
> **5. Замерить снова.** Подтвердить улучшение цифрами. Добавить benchmark в CI, чтобы не откатилось.

Этот алгоритм — то, что интервьюер хочет услышать. Покажет зрелость подхода.

---

## ✅ Чек-лист готовности (что уметь)

- [ ] Объяснить, что такое memory leak и назвать 4+ типичных причины
- [ ] Рассказать про LeakCanary + Memory Profiler
- [ ] Объяснить jank, 16ms, как найти дроп кадра
- [ ] Знать: App Startup (cold/warm), TTID/TTFD
- [ ] Что такое Baseline Profiles и зачем Macrobenchmark
- [ ] Compose recomposition: stable/unstable, remember/derivedStateOf, ImmutableList, Compose Compiler Metrics
- [ ] Назвать 3+ инструмента профилирования и их назначение
- [ ] Рассказать методологию «измерить → найти → починить → измерить»

---

## 📚 Ресурсы для углубления

### Практика + теория
- 🔥 [Android Performance Optimization Interview Q&A (Medium)](https://medium.com/@sauravsushant58/android-performance-and-optimization-interview-questions-answers-ec076baa3f56)
- [Android Interview Questions (Amit Shekhar, ex-Instagram, GitHub)](https://github.com/amitshekhariitbhu/android-interview-questions) — есть раздел по Compose perf
- [Android App Performance Optimization (dataford.io)](https://dataford.io/questions/android-app-performance-optimization)

### Baseline Profiles / Macrobenchmark (современное)
- 🔥 [Duolingo: Cut startup time 30% with Baseline Profiles](https://blog.duolingo.com/slashed-android-startup-time-baseline-profiles/)
- [Macrobenchmark overview (офиц.)](https://developer.android.com/topic/performance/benchmarking/macrobenchmark-overview)
- [Measure Baseline Profile (офиц.)](https://developer.android.com/topic/performance/baselineprofiles/measure-baselineprofile)
- [Baseline Profiles Codelab (Google)](https://codelabs.developers.google.com/android-baseline-profiles-improve)
- [Startup Profiles: quick fix for slow startups (ProAndroidDev)](https://proandroiddev.com/startup-profiles-the-quick-fix-for-painfully-slow-app-startups-2f1c9c0a8fd9)

### Видео
- [The Next Evolution in Profiling for Android — Android Performance Analyzer (YouTube)](https://www.youtube.com/watch?v=peplbYt0Ohg)
- [Boost performance with Baseline Profiles (YouTube)](https://www.youtube.com/watch?v=hqYnZ5qCw8Y)

### Подготовка к интервью (senior)
- [How to Prepare for Senior Android Interviews (Jointaro)](https://www.jointaro.com/question/LrS2IELrVoZeWdygua4u/how-to-prepare-for-androidmobile-senior-software-interviews/)

---
_Связанные файлы: [Блок 2 — собеседования](../russia/02-interview-prep.md) · [Трекер слабых мест](./README.md)_
