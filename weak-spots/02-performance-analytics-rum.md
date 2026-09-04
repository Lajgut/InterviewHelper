# 🧩 Слабое место #2 · Замеры производительности + аналитика (RUM)

> **Спрашивают:** ⭐⭐⭐ Часто на Senior/Lead, отличает «теоретика» от «практика»
> **Статус:** 🟢 Готово
> Обновлено: 2026-08-06

---

## 📌 Что реально спрашивают

1. «Куда поставить ивент старта приложения? Где понять, что старт окончен?»
2. «Как замерять эффективность скролла?»
3. «Как сделать FPS-мониторинг в проде?»
4. «Как отправлять эти метрики в аналитику? Куда, как часто, в каком виде?»
5. «Чем отличается замер в деве от замера в проде (RUM)?»

> 💡 **Ключевой инсайт для интервью:** продакшн-мониторинг (RUM — Real User Monitoring) ≠ профайлер в Android Studio. Профайлер — для дева, **FrameMetrics/JankStats** — для прода. Это разделение показывает зрелость.

---

## 🧭 Карта темы

```
Замеры производительности
├── 1. Startup (TTID / TTFD) — куда ставить ивенты
├── 2. FPS / Jank — FrameMetrics API (API 24+)
├── 3. JankStats (androidx.metrics) — продакшн-стандарт
├── 4. Scroll performance — замеряем плавность списков
├── 5. Отправка в аналитику — батчинг, семплинг, sink
└── 🆚 Профайлер (dev) vs RUM (prod)
```

---

## 1️⃣ Startup: куда ставить ивенты (TTID / TTFD)

### Две ключевые метрики
| Метрика | Что значит | Кто меряет |
|---------|-----------|------------|
| **TTID** (Time To Initial Display) | Первый кадр отрисован | Система (автоматически) |
| **TTFD** (Time To Full Display) | Контент загружен, UI интерактивен | **Ты** — через `reportFullyDrawn()` |

> Система меряет TTID сама. TTFD — **твоя ответственность**: надо явно сказать системе «я готов».

### Точка старта (отсчёт)
- Система сама берёт за точку старта момент `zygote fork` / `ActivityManager` start.
- В логкате ищи `Displayed` (TTID) и `reportFullyDrawn` (TTFD): `ActivityTaskManager: Displayed com.example/.MainActivity: +1s234ms`.

### Куда ставить «старт окончен» (TTFD)
❌ **НЕ после `setContentView()`** — UI пустой, контента ещё нет.
✅ **После того как:**
- Загружены реальные данные (из сети/БД) И отрисован первый осмысленный контент.
- Главная фция экрана готова к взаимодействию.

```kotlin
// View-подход
class MainActivity : Activity() {
    override fun onCreate(...) {
        // ...
        viewModel.state.collect { state ->
            render(state)
            if (state.isFirstContentLoaded) {
                reportFullyDrawn() // ← TTFD фиксируется системой
            }
        }
    }
}
```

```kotlin
// Compose-подход (предпочтительно)
// ReportDrawn / ReportDrawnWhen / ReportDrawnAfter
setContent {
    AppContent(
        reportFullyDrawn = { /* триггерится когда первый контент готов */ }
    )
}

// через androidx.activity.compose:
val reportDrawn = remember { mutableStateOf(false) }
ReportDrawnWhen { reportDrawn.value } // TTFD когда условие true
```

### Бенчмарки
- **Cold start < 500ms** (идеал), < 5с — «slow cold start» (Android Vitals флагает).
- **TTID** обычно < 2с.
- Замерять через **Macrobenchmark** (`StartupTimingMetric`) в CI.

### В аналитику отправляем
```kotlin
val coldStartMs = System.currentTimeMillis() - appStartTimeMs
analytics.log("app_start", mapOf(
    "type" to "cold",          // cold / warm / hot
    "ttid_ms" to ttidMs,       // из logcat Displayed или frame callback
    "ttfd_ms" to ttfdMs,       // твой замер
    "device_tier" to deviceTier // low/mid/high — важно для сегментации
))
```

> 🔑 **Точка старта в коде**: фиксируй `System.currentTimeMillis()` в `Application.onCreate()` (самый ранний хук) или через `ActivityManager.ProcessLifecycle`.

---

## 2️⃣ FPS / Jank мониторинг — FrameMetrics API (API 24+)

### Что даёт
`Window.OnFrameMetricsAvailableListener` — колбэк на **каждый отрисованный кадр** с таймингами фаз.

```kotlin
window.addOnFrameMetricsAvailableListener(
    { _, frameMetrics, _ ->
        val metrics = frameMetrics ?: return@addOnFrameMetricsAvailableListener
        val totalDurationNs = metrics.getMetric(FrameMetrics.TOTAL_DURATION)
        val layoutMeasureNs = metrics.getMetric(FrameMetrics.LAYOUT_MEASURE_DURATION)
        val drawNs = metrics.getMetric(FrameMetrics.DRAW_DURATION)

        val frameMs = totalDurationNs / 1_000_000
        val isJank = frameMs > 16 // 60fps порог; 8ms для 120fps

        if (isJank) {
            jankCounter++
            analytics.log("frame_jank", mapOf(
                "frame_ms" to frameMs,
                "phase" to when {
                    layoutMeasureNs > 8_000_000 -> "layout"
                    drawNs > 8_000_000 -> "draw"
                    else -> "cpu"
                },
                "screen" to currentScreenName // важно
            ))
        }
    },
    null // handler; null = основной поток
)
```

### Метрики FrameMetrics (знаешь — плюс)
- `TOTAL_DURATION` — полное время кадра.
- `LAYOUT_MEASURE_DURATION` — measure/layout.
- `DRAW_DURATION` — отрисовка.
- `CPU_DURATION`, `GPU_DURATION` — CPU vs GPU.
- `ANIMATION_DURATION`.

> ⚠️ **Порог jank:** 16ms для 60 Гц, **8ms для 120 Гц**. Берётся из `Display.getRefreshRate()`.

### Минусы «голого» FrameMetrics
- Не даёт эвристику «реально ли это jank» (может быть системная анимация).
- Надо самому аккумулировать и агрегировать (не слать каждый кадр!).
- Только с API 24 (Android 7.0).

---

## 3️⃣ JankStats (androidx.metrics) — продакшн-стандарт 🔥

**Главный ответ на «как сделать FPS-мониторинг в проде».**

Библиотека Jetpack поверх FrameMetrics, **добавляет эвристику jank** и удобную агрегацию. Используется даже **Datadog RUM SDK под капотом**.

```kotlin
// build.gradle
implementation("androidx.metrics:metrics-performance:1.0.0-beta02")

// В Activity / Fragment / Compose
val jankFrameListener = JankStats.OnFrameListener { frameData ->
    // frameData.isJank — эвристика библиотеки (учитывает refresh rate)
    // frameData.frameDurationUiNanos
    // frameData.states — список текущих UI-состояний (например имя экрана)
    if (frameData.isJank) {
        rumReporter.report(frameData)
    }
}

val jankStats = JankStats.createAndTrack(window, jankFrameListener)
jankStats.trackPerformance = true
```

### Сильные стороны JankStats
- **Эвристика jank** — сама определяет, реально ли кадр «просажен» относительно refresh rate.
- **PerformanceStateHistory** — можно привязать к кадру контекст («какой экран», «какая фича»).
- Работает на API 21+ (до 24 — только UI-часть, без CPU/GPU).
- Стандарт для RUM-библиотек.

---

## 4️⃣ Scroll performance — как замерять плавность списков

### Что реально важно
Пользователь оценивает плавность скролла, а не абстрактный FPS. Метрики:
- **% janky frames** во время скролла (доля кадров >16ms).
- **Frozen frames** (кадры >700ms — это уже видимый «зависон»).
- **Scroll hesitation** — микростаны при скролле.

### Подход
1. **FrameMetrics/JankStats** + пометка состояния `state = "scrolling_<screen>"`:
   ```kotlin
   jankStats.putState(PerformanceInfoState("scrolling_feed"))
   // ... скролл ...
   jankStats.removeState(...)
   ```
2. Для конкретных списков можно мерять через **RecyclerView.OnScrollListener** + колбеки кадров.
3. **Macrobenchmark** в CI — `FrameTimingMetric` / `ScrollBaselineProfile` для регрессионного контроля.

### В аналитику
```kotlin
// Агрегируем за сессию скролла, не каждый кадр
analytics.log("scroll_perf", mapOf(
    "screen" to "feed",
    "total_frames" to totalFrames,
    "jank_frames" to jankFrames,
    "jank_pct" to jankFrames * 100.0 / totalFrames,
    "frozen_frames" to frozenCount, // >700ms
    "p95_frame_ms" to p95
))
```

---

## 5️⃣ Отправка в аналитику — БАТЧИНГ И СЕМПЛИНГ (часто упускают, а спрашивают)

❌ **Нельзя слать каждый janky frame** — убьёшь батарею и сеть.
✅ **Правильно:**

| Практика | Почему |
|----------|--------|
| **Семплинг** (e.g. 1–10% юзеров) | Метрик и так много; репрезентативности хватает |
| **Агрегация за сессию/экран** | % janky frames вместо N событий |
| **Батчинг** (пачками по таймеру/размеру) | Меньше сетевых запросов |
| **Отправка в фоне / на WiFi** | Не садим батарею |
| **p50/p95/p99 перцентили** | Среднее бесполезно, нужны хвосты |
| **Сегментация** (device tier, OS, версия app) | «Тормозит на low-end» — иначе не поймёшь |

### Что отправлять (минимальный набор RUM-метрик)
- `app_start`: type (cold/warm/hot), ttid_ms, ttfd_ms, device_tier
- `frame_jank` (агрегировано): screen, jank_pct, p95_frame_ms, frozen_count
- `screen_load`: screen_name, load_ms (TTFD экрана, не всего приложения)
- `custom_flow`: flow_name, duration_ms (критические юзер-пути: логин, оплата)

### Куда отправлять
- **Firebase Performance Monitoring** — простая интеграция, бесплатная база.
- **Datadog / New Relic / Sentry** — коммерческий RUM (под капотом часто JankStats).
- **Своя аналитика** — если есть команда платформы.

---

## 🆚 Профайлер (dev) vs RUM (prod) — ключевое различие

| | Dev (Android Studio Profiler) | Prod (RUM) |
|---|-------------------------------|------------|
| **Где** | Локально, на твоём девайсе | На реальных устройствах юзеров |
| **Зачем** | Найти конкретную причину | Отловить регрессию на масштабе |
| **Данные** | Глубокие (flame chart, allocation) | Агрегированные (p95, %jank) |
| **Инструмент** | CPU/Memory Profiler, Perfetto | FrameMetrics, JankStats, Firebase Perf |
| **Нагрузка** | Тяжёлая, вешает девайс | Лёгкая, незаметна для юзера |
| **Когда** | При отладке | Всегда, на % юзеров |

> 💡 На интервью скажи: *«В деве — Profiler/Perfetto для root cause; в проде — JankStats + Firebase/Datadog для мониторинга регрессий на реальных пользователях. Это два разных процесса, и нужны оба.»* — это ответ уровня lead.

---

## 🗣️ Универсальный алгоритм ответа на «как замерять X»

> 1. **Что измеряем и зачем** (метрика, цель — например «p95 TTFD < 1с»).
> 2. **Где точка отсчёта и точка завершения** (для старта — `Application.onCreate` / `Displayed`; конец — `reportFullyDrawn`).
> 3. **Какой инструмент** (FrameMetrics для fps, JankStats для прод-эвристики, Macrobenchmark для CI).
> 4. **Как агрегируем и отправляем** (семплинг, батчинг, перцентили).
> 5. **Как предотвращаем регрессию** (бенчмарк-гейты в CI).

---

## ✅ Чек-лист готовности

- [ ] Объяснить TTID vs TTFD и где вызывать `reportFullyDrawn()`
- [ ] Знать `Window.OnFrameMetricsAvailableListener`, назвать 2+ метрики
- [ ] Рассказать про **JankStats (androidx.metrics)** как продакшн-стандарт
- [ ] Объяснить порог jank (16ms / 8ms, зависит от refresh rate)
- [ ] Как мерять scroll perf (% jank, frozen frames, агрегация по экрану)
- [ ] Семплинг, батчинг, перцентили — почему не слать каждый кадр
- [ ] Различать dev-профайлер vs prod-RUM
- [ ] Назвать Firebase Performance / Datadog / Sentry как sink

---

## 📚 Ресурсы

### Официальное (главное)
- 🔥 [App startup time — TTID/TTFD (офиц.)](https://developer.android.com/topic/performance/vitals/launch-time)
- [App startup analysis & optimization (офиц.)](https://developer.android.com/topic/performance/appstartup/analysis-optimization)
- 🔥 [JankStats Library (офиц.)](https://developer.android.com/topic/performance/jankstats)
- [androidx.metrics releases](https://developer.android.com/jetpack/androidx/releases/metrics)
- [FrameMetrics (API reference)](https://developer.android.com/reference/android/view/FrameMetrics)
- [UI jank detection (Android Studio)](https://developer.android.com/studio/profile/jank-detection)

### Практика и кейсы
- [FrameMetrics — realtime smoothness tracking (froger_mcs)](https://medium.com/@froger_mcs/framemetrics-realtime-app-smoothness-tracking-3d8550413c1c) + [GitHub ActivityFrameMetrics](https://github.com/frogermcs/ActivityFrameMetrics)
- [JankStats Goes Alpha (Android Developers)](https://medium.com/androiddevelopers/jankstats-goes-alpha-8aff942255d5)
- [App Startup: Lessons from Facebook (Android Blog)](https://android-developers.googleblog.com/2021/11/improving-app-startup-facebook-app.html)
- [How Back Market cut startup time 50%](https://engineering.backmarket.com/how-we-enhanced-our-android-apps-startup-time-by-over-50-0fb220d27b14)
- [Perfetto FrameTimeline (jank source)](https://perfetto.dev/docs/data-sources/frametimeline)

### Видео
- [Chasing 60 FPS: Frame Time Profiling in Compose (YouTube)](https://www.youtube.com/watch?v=E1Kj_Pmqgl4)

---
_Связанные файлы: [Тема #1 — Performance & Profiling](./01-performance-profiling.md) · [Трекер слабых мест](./README.md)_
