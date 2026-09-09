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
## ⏳ ДОПОЛНИТЬ (по мере прохождения моков)
- [ ] Coroutines + Flow — продолжение (вт)
- [ ] Compose (ср)
- [ ] Android Core (чт)
