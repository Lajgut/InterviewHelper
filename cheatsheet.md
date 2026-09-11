# ⚡ ШПАРГАЛКА — повторение перед интервью (и 5 минут до созвона)

> Живой файл — пополняется по мере прохождения моков. Только сжатое, для беглого повторения.
> Обновлено: 2026-09-07

---

## 1 · Smart cast — 4 случая ОТКАЗА (спросят с вероятностью 80%)

Не работает для:
1. **`var`-свойство класса** (другой поток изменит между проверкой и использованием)
2. Свойство с **custom getter'ом** (нет backing field)
3. **`open`-свойство** (переопределят наследники)
4. Локальная **`var` в модифицирующей лямбде**

**Фикс всегда один — локальная val-копия:**
```kotlin
val n = name ?: return   // дальше n уже String
```
Альтернативы: `x?.length`, `x?.let { }`. `!!` — почти никогда.

## 2 · Scope functions — таблица

| | внутри | возвращает | кейс |
|---|--------|-----------|------|
| `let` | `it` | **результат** | `x?.let { }` ← главный кейс |
| `run` | `this` | **результат** | `val a = rect.run { w * h }` |
| `with` | `this` | результат, **НЕ extension** | `with(x) { ...пачка... }` |
| `apply` | `this` | **ОБЪЕКТ** | `Intent().apply { putExtra() }` |
| `also` | `it` | **ОБЪЕКТ** | `.also { log(it) }` |

**Мнемоника: «A» = объект вернётся** (Apply, Also). `also` = «также» = side-effect.
`with` = `run`, но аргументом, не точкой.

## 3 · `==` / `===` / equals

```kotlin
a == b  →  a?.equals(b) ?: (b === null)   // поэтому null-safe, не падает
```
- `===` — ссылки. `null == null` → `true`
- **Integer cache: -128..127** (127==127 true, 128==128 false в Java)
- Строки: литерал из string pool; `new String("s")` — новый объект

## 4 · inline-семейство

- **inline** = тело+лямбды копируются в место вызова → нет аллокаций лямбд, код «как есть» в вызывающем
- **non-local return** — `return` из лямбды выходит из вмещающей fun (возможен ТОЛЬКО из-за inline)
- **reified** — фраза: *«после встраивания стирается не тип, а граница функции — тип известен в точке вызова»*. Кейсы: `filterIsInstance<T>()`, `Intent(this, T::class.java)`
- **noinline** — оставить лямбду объектом (сохранить/передать)
- **crossinline** — лямбда уезжает в другой контекст (`Handler.post { }`, поток) → non-local return запрещён
- Public inline не видит private члены класса; из Java inline = обычный вызов

## 5 · Variance — out / in

- **`out` = output / producer** — только ОТДАЁТ T: `List<out E>`
- **`in` = input / consumer** — только ПРИНИМАЕТ T: `Comparable<in T>`
- PECS: Producer Extends, Consumer Super
- `List<out Cat>` → можно как `List<Animal>` ✅ (читаем)
- **`List<*>`** = «один неизвестный тип»: читать **только как `Any?`** (НЕ Cat/Animal!), писать ❌ — даже `MutableList<*>` не позволит add. В рантайме информации о типе НЕТ (type erasure) — узнать можно только `is`/кастом по элементу
- **`List<Any?>`** = «любые вперемешку»: писать ✅
- Java-массивы ковариантны → `Object[] o = new String[1]; o[0]=42` → **ArrayStoreException**; Kotlin `Array<T>` инвариантен (дыры нет)

## 6 · data class / sealed / companion

- **data class = 5 методов: equals + hashCode + toString + copy + componentN**
- Контракт: *равные → равные hashCode (обратное НЕ обязательно)*. Только equals без hashCode → дубли в HashSet, потери в HashMap
- `copy(name = "x")` — именованные аргументы; MVI: `state.copy(isLoading = true)`
- **sealed: наследники в том же модуле** → `when` exhaustive без `else`. Enum = константы без данных; sealed = классы с состоянием
- **companion object = РЕАЛЬНЫЙ объект-синглтон** в классе: реализует интерфейсы, расширяется extensions, передаётся как значение. Java видит `Foo.Companion` → `@JvmStatic` для настоящего static

## 7 · Sequence vs коллекции

- List (eager): **«горизонтально»** — операция за операцией, каждая по ВСЕЙ коллекции + новая коллекция
- Sequence (lazy): **«вертикально»** — один элемент сквозь весь пайплайн, потом следующий
- **Без терминальной (`toList`/`first`) не вычислится НИ ОДИН элемент**
- Выигрыш: ранний выход (`first`), **1000+ элементов**, длинные цепочки. Проигрыш: маленькие списки
- `generateSequence(seed) { next }` — бесконечные последовательности

## 8 · Исключения / Result / runCatching ⚠️ КРИТИЧНО

```kotlin
runCatching = try { ... } catch (e: Throwable)  // ловит ВСЁ
```
- **Ловит CancellationException → ЛОМАЕТ отмену корутин!** kotlinx: НЕ использовать в suspend
- Правило suspend-кода:
```kotlin
try { api.get() }
catch (e: CancellationException) { throw e }   // ВСЕГДА rethrow
catch (e: IOException) { fallback }            // только конкретные типы
```
- `Result<T>`: fold / getOrElse / getOrNull / onSuccess / onFailure. Компилятор ЗАПРЕЩАЕТ `Result` как return type (почему в компаниях запрет)
- `@Throws` — только для Java-интеропа (пишет throws в байткод)

## 9 · Делегаты

- `by lazy` режимы: **SYNCHRONIZED** (default, double-checked) / **PUBLICATION** (гонка, победитель кешируется) / **NONE** (однопоточно)
- `lateinit`: только `var` + non-null + НЕ примитивы; проверка **`::prop.isInitialized`** (не try-catch!); DI — легитимно
- `Delegates.observable` — слушать изменения; `vetoable` — запретить значение
- Свой делегат: `getValue`/`setValue` (ReadWriteProperty)

## 10 · Разное из Kotlin-секции

- **Platform types** `String!` — из Java, null-безопасности нет → NPE возможен
- Коллекции: `map/filter` на List — выполняются сразу (eager), каждая аллоцирует
- `componentN` → деструктуризация: `val (a, b) = pair`

## 11 · Иерархия типов (Any / Unit / Nothing / Throwable)

- **Any** = родитель всех не-null типов = наш «Object» (на JVM → java.lang.Object). Методы: equals, hashCode, toString
- **Unit** = «вернулось, но без значения» (тип с 1 инстансом); `() -> Unit` совместим с любой лямбдой
- **Nothing** = НИКОГДА не вернётся (throw, TODO(), exitProcess); подтип ВСЕХ типов → `if (ok) 1 else throw E` — тип Int; `emptyList<Nothing>` → `List<T>` для любого T
- **Throwable → Exception (ловим) / Error (НЕ ловим: OOM, StackOverflow)**. Checked-исключений в Kotlin НЕТ
- **lazy default (SYNCHRONIZED): лямбда выполнится ОДИН РАЗ** — остальные ждут и берут кеш (double-checked). PUBLICATION — параллельные вычисления, кешируется первый результат
- Проверка lateinit: **`::tracker.isInitialized`** (ссылка на свойство, НЕ метод объекта)
- observable — ПОСЛЕ изменения, нельзя отменить; vetoable — ДО, может запретить (false)
- Свой делегат: `ReadWriteProperty` с `getValue`/`setValue`

## 23 · Dagger 2 — фичи

- **Compile-time** кодогенерация → ошибки графа = ошибки компиляции, без рефлексии
- **`@Component`**: мост к графу (`DaggerXxx` генерится). `dependencies=` — видит только экспортированное родителем
- **`@Subcomponent`**: наследует ВЕСЬ граф родителя, живёт в его scope (Hilt: ActivityRetained внутри Singleton). vs dependencies: «ребёнок с полным доступом» против «сосед по списку»
- **`@Scope`**: кеш инстанса в рамках компонента (Singleton = один на компонент; @ActivityScope = на activity). Скопы компонента и провайдера должны совпадать
- **`@Binds`** (абстрактный, интерфейс→импл, эффективнее) vs **`@Provides`** (с телом, для сторонних: Retrofit/OkHttp)
- **`@IntoSet` / `@IntoMap` + `@StringKey`/`@ClassKey`**: мульти-биндинги (наборы валидаторов, Map фабрик VM)
- **`@Assisted` + `@AssistedFactory`**: рантайм-параметры вне графа → `factory.create(id)`. Классика: Worker, VM с аргументом
- **`@Qualifier`** (типобезопасно) / `@Named("x")`: два биндинга одного типа
- **`Lazy<T>`** (первый get + кеш) vs **`Provider<T>`** (каждый get → провайдер заново)

## 22 · Архитектура (Яндекс: «паттерны и принципы»)

- **Ось MVP/MVVM/MVI**: MVP — Presenter держит **ссылку на View**; MVVM — VM **не знает View**, тот наблюдает (Observer); MVI — **единый State-объект** + Intent'ы (View→Intent→Reducer→State→View)
- **UDF**: события в одну сторону, состояние — в другую, мутирует один reducer
- **VM ПЕРЕЖИВАЕТ поворот** — 3 отдельных LiveData = проблема НЕ сброса, а **рассинхрона частичных состояний** → единый `data class UiState` + `StateFlow` + `copy()`
- **Clean Architecture Dependency Rule**: зависимости направлены ВНУТРЬ (внешние знают про внутренние, никогда наоборот). Физически: модуль `:domain` не может импортировать `:data`
- **DTO не торчит из репозитория**: DataSource маппит `UserDto → User`, репо отдаёт Entity. Смена сети не ломает feature
- **By layer** (общие `:core:data` и т.д.) — малые проекты; **by feature** (вертикальный срез фичи) — большие: параллельная сборка, владение end-to-end
- **api/impl**: api — что торчит наружу, impl — внутренности фичи (+ naming-изоляция: разные Product не конфликтуют)
- **DI**: выскоуровневые НЕ зависят от низкоуровневых — оба от абстракций (DIP). **Hilt vs Koin: Hilt — compile-time (забыл биндинг = не собралось), Koin — runtime (краш у юзера)**
- `@HiltViewModel` + `@Inject constructor` на VM; `@Binds` — на абстрактном методе `@Module` (связка интерфейс↔реализация)
- **SOLID**: S — один класс одна причина изменения; O — расширяемость без правки; L — наследник взаимозаменяем; I — много узких интерфейсов лучше одного толстого; D — зависимости на абстракции. God Object → нарушает S
- Кейс для завтра: ВК = MVI-фреймворк (унификация суперапп+ТВ); Маруся = shared SDK-модуль для ТВ/часов/колонки/авто — переиспользование как необходимость

## 21 · Android Core essentials

- **Main = 1 поток**: рисует UI, ивенты. **Looper** крутит MessageQueue (loop()); **Handler** — класть сообщения. Looper есть только у main и HandlerThread
- **ANR: input 5с / broadcast 10с / service 20с**
- **postDelayed НИКТО не ждёт** — сообщение лежит в очереди до времени → забытый пост = утечка Activity + крэш. Фикс: `lifecycleScope.launch { delay(); ... }` (авто-отмена) или `removeCallbacksAndMessages(null)` в onDestroy
- **ViewModel**: переживает конфиг-изменения (ViewModelStore), НЕ переживает process death → **SavedStateHandle**
- Поворот → VM жива; process death → VM новая + данные из Bundle/SavedStateHandle
- Утечки (твоя тема!): static Context, слушатели не отписаны, inner non-static, корутины вне scope, Handler-посты → **LeakCanary**, Memory Profiler heap dump
- Компоненты: Service (started/bound/foreground), BroadcastReceiver, WorkManager (отложенный гарантированный фон) vs AlarmManager
- Context: Application (живёт вечно, безопасно держать) vs Activity (утечка в singleton!)
- Холодный старт: зигота → Application.onCreate → Activity — тяжелое в onCreate = тормозит старт (твой кейс 23s→13s!)

## 20 · Compose: remember + side-effects

- **remember** — переживает рекомпозицию (пока функция в композиции); **rememberSaveable** — + Bundle → переживает поворот И смерть процесса
- `remember { list.size * 2 }` без ключа — **застрянет** при изменении list. Фиксы: без remember (если дёшево) / `remember(ключ)` / `derivedStateOf`
- **`derivedStateOf`** — производный State с ленивым пересчётом: рекомпозиция только при пересечении порога (`list.size > 10`), не на каждый элемент
- **`LaunchedEffect(key)`** — 1 раз на вход в композицию (НЕ на рекомпозицию!); перезапуск при смене ключа; **поворот → повтор** (ловушка дублирования аналитики → флаг в VM)
- **`SideEffect`** — после КАЖДОЙ рекомпозиции (синк не-Compose мира: обновить поле библиотеки)
- **`DisposableEffect`** — вход + `onDispose` при уходе (подписка/отписка)

## 19 · Compose: recomposition + stability

- **Recomposition** = повторный вызов composable при изменении **State, который функция ЧИТАЛА**
- **Smart recomposition**: перезапускается МИНИМУМ функций; со stable-неизменными параметрами — **SKIP**
- **Unstable — это ТИП, не val/var!** `val tags: List<String>` → класс unstable: List — интерфейс без гарантии immutability. Фикс: `ImmutableList` (kotlinx) или `@Immutable`/`@Stable`
- Unstable-параметр → родитель рекомпознулся → ребёнок НЕ скипнется (даже при равных данных)
- `key = { it.id }` в items → identity для reorder/анимаций/переиспользования (аналог diffUtil)
- Лямбда-параметр меняется каждую рекомпозицию → фикс: `remember(onItemClick) { ... }`; свежий компилятор мемоизирует лямбды без unstable-захватов
- Инструменты: Compose Compiler Metrics (отчёт стабильности), Layout Inspector (recomposition counts)

## 18 · Flow-буферизация + Channels (экспресс)

- **buffer(n)** — эмиттер и коллектор в разных корутинах: эмит не ждёт коллектора
- **conflate** — если потребитель не успевает: промежуточные значения СБРАСЫВАЮТСЯ, остаётся последнее (для UI-стейтов)
- **collectLatest** — при новом значении ОТМЕНЯЕТ выполнение предыдущего коллектора (поиск: старый запрос результатов отменяется)
- **debounce / distinctUntilChanged** — стандарт для поисковых строк
- **callbackFlow { }** — обёртка callback-API в Flow (awaitClose { снять listener'а } — обязательно!)
- **Channel** — очередь между корутинами; RENDEZVOUS (0, ждёт получателя), BUFFERED, CONFLATED. receiveCancellable
- **Mutex** vs synchronized: lock() — suspend (не блокирует поток); withLock { }
- **select { }** — ждать первое из нескольких (Channel.onReceive / onTimeout)

## 17 · StateFlow vs SharedFlow (топ-1 вопрос!)

- **StateFlow** = SharedFlow с `replay=1` + **дедупликация** (equals — повтор не доставляется). Обязателен initial value
- **SharedFlow**: `replay=N` (что получают новые подписчики), `extraBufferCapacity` (буфер эмиттеров), `bufferOverflow` (SUSPEND/DROP_OLDEST/DROP_LATEST)
- **UI-state → StateFlow; одноразовые события (снекбар, навигация) → SharedFlow(replay=0)**
- StateFlow для событий = антипаттерн: replay=1 → повторная доставка при пересоздании экрана (та же sticky-болезнь LiveData)
- Hot = эмитит без подписчиков; новый подписчик StateFlow сразу получает текущее значение
- Исключение в `viewModelScope.launch` без try → **КРЭШ** (SupervisorJob не разносит отмену, но uncaught летит в дефолтный handler) + вечный isLoading. Фикс: try + rethrow Cancellation + catch Exception → error-state

## 16 · Exceptions: launch vs async

- **launch**: исключение uncaught → ВВЕРХ (отмена родителя+братьев) → корень → **handler или КРЭШ**
- **async**: исключение **запечатано в Deferred** → родителя не трогает → бросается **только при `await()`**
- **await не вызван** → исключение потеряно МОЛЧА (в отдельном scope) — ловушка!
- **CoroutineExceptionHandler работает ТОЛЬКО на корневом launch** (или в scope). Поставишь в ребёнка — игнор
- `coroutineScope {}`: ребёнок упал → отмена всех → scope **rethrow'ит первое исключение** наверх
- `supervisorScope {}`: умирает только упавший ребёнок, сиблинги живут

## 15 · Cancellation

- **Кооперативная** = не убить принудительно; корутина уступает САМА: suspend-точки (delay/yield) проверяют отмену автоматически; CPU-код — `while (isActive)` / `ensureActive()`
- `while(true)` + чистый CPU без suspend → **никогда не узнает об отмене → зависает**
- **suspend внутри `finally` → мгновенный CancellationException → чистка недовыполнится**
- Чинится: `finally { withContext(NonCancellable) { rollback() } }` — очистка/логи при уходе пользователя
- Отмена: CancellationException — обычное исключение; глотать нельзя (runCatching!), rethrow всегда

## 14 · Dispatchers

- **Default** = CPU-ядер потоков (min 2) — CPU-bound (сортировка, парсинг)
- **IO** = до **64** потоков — блокирующие сеть/диск/БД
- **Main** = 1 поток (UI); **Main.immediate** = если уже на main → выполнить ИНЛАЙН без поста в очередь (viewModelScope = Main.immediate)
- **Unconfined** — resume на «чужом» потоке; только тесты
- `withContext` = переключение контекста, **НЕ создаёт корутину, НЕ асинхронит** → последовательные withContext = сумма времени; параллельность = `coroutineScope { async/await }`
- withContext возвращает **последнее выражение лямблы** (пустая → Unit)
- Default и IO **делят физический пул** (default-слот / blocking-слот у потока) → switch Default→IO часто без смены потока

## 13 · suspend под капотом (Яндекс любит!)

- **CPS**: `suspend fun getUser(): User` → `fun getUser(c: Continuation<User>): Any?` — скрытый параметр-колбэк; возврат = результат ИЛИ `COROUTINE_SUSPENDED`
- **Continuation** = колбэк `resumeWith(Result<T>)` + context
- **Физических вызовов** = число реальных приостановок + 1 (быстрые suspend без паузы — всё за один вызов)
- **State machine**: тело → `switch(label)` в continuation; **локальные переменные — в полях continuation-объекта (heap!), не на стеке**
- Из обычной функции нельзя — нет continuation-параметра (bridge: launch/runBlocking)
- **Инверсия стека**: приостановка = unwind стека, resume = новый кадр; глубина живёт в linked-list continuation'ов в heap → нет StackOverflow

## 12 · Coroutines: structured concurrency + Job

- Формула: **отмена — вниз по иерархии, ошибка — вверх**
- **Job**: падение ребёнка → отменяет родителя → родитель отменяет ВСЕХ детей; исключение выходит наружу → **без CoroutineExceptionHandler — крэш приложения**
- **SupervisorJob**: падение ребёнка не трогает родителя и братьев. **Умирает только от явного cancel()** — scope жив и переиспользуемый
- Обычный **Job() завершается сам**, когда завершены все дети → scope дальше непригоден
- `viewModelScope = SupervisorJob() + Dispatchers.Main.immediate` — одна упавшая корутина не убивает остальные задачи VM

---
## 🌅 УТРО ПЯТНИЦЫ — порядок повторения (40 мин)

1. **22 (Архитектура)** — свежевыученное, завтра «паттерны и принципы» в письме! + 21 (Android Core) глазами
2. Блоки **2 (scope) → 8 (runCatching) → 16 (exceptions)** — исторически слабейшие
3. **19–20 (Compose unstable + LaunchedEffect)** — свежее
4. **1 (smart cast) + 5 (hashCode)** — повторные провалы дважды
5. **17 (StateFlow vs SharedFlow)** — топ-1 вероятность вопроса

**На интервью:** думать вслух · уточнять условия задачи (null? пустой список?) · после кода самому прогнать тест-кейсы · кейс 23s→13s держать наготове · «не знаю» — честно + рассуждать от принципов
