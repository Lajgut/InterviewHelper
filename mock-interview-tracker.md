# 🎤 Мок-интервью — трекер глубины ответов

> Формат: я задаю вопросы порциями (как интервьюер Яндекс/Ozon), ты отвечаешь.
> Я отмечаю глубину ответов. После всех тем — итоговый разбор + список на переповтор.
> Обновлено: 2026-08-31

**Шкала глубины:**
- 🔴 Нет ответа / совсем поверхностно — обязательно переповторять
- 🟡 Средне: ответил, но с пробелами / путаницей — выборочно переповторить
- 🟢 Глубоко: уверенно, с деталями и нюансами — ок

---

## Неделя 1 · Kotlin

| # | Тема | Глубина | Комментарий |
|---|------|--------|-------------|
| 1 | Null-safety, smart-casts | 🟡 | База верно + бонус: platform types (String!). НО: `!!` подан как основной способ (а он unsafe, главный — `?.`/`?:`/`let`); smart cast не раскрыт ВООБЩЕ (половина вопроса): не сказано про auto-приведение после if и 4 случая когда не работает (var-свойство, custom getter, open, var в лямбде). **Переповторить: smart cast cases + safe call/Elvis как основные инструменты** |
| 2 | let/run/apply/also/with | 🔴 | Критичная ошибка: сказал «let/run возвращают объект» — а они возвращают РЕЗУЛЬТАТ ЛЯМБДЫ (объект — только apply/also). Верно: it vs this ось, apply-пример, честно признал пробел. Не назвал: `?.let {}` (самый частый кейс), with — не extension. **Тема на зубрёжку — выучить таблицу + мнемонику «A = объект вернётся»** |
| 3 | inline/noinline/crossinline, reified | 🟡 | Верно: встраивание + нет аллокаций, non-local returns как мотивация, crossinline определён ТОЧНО, reified-связка с inline. Пробелы: не объяснил механику reified (type erasure → после встраивания тип известен в точке вызова), нет примеров (filterIsInstance, startActivity<T>), noinline не затронут, формулировка trade-off перевёрнута (при частых вызовах — code bloat). Дан подробный разбор с примерами (non-local vs return@label, postDelayed-задача). **Повторить: почему reified требует inline — одной фразой; noinline-кейс «сохранить лямбду»** |
| 4 | Дженерики, variance (in/out) | 🔴 | Термины кова/контра знает, связь с Java wildcards есть. НО ядро — соответствие in/out — НЕ знает («не помню, помоги запомнить»). Star-projection объяснил НЕВЕРНО — через «тип известен в рантайме и cast» (это type erasure, не star-projection); отличие List<*> от List<Any?> не знает. Примеры stdlib (List<out E>, Comparable<in T>) не назвал. Массивы Java (ArrayStoreException) — не знал. Дан разбор с мнемониками (out=output/producer, in=input/consumer, PECS). **Выучить: out/in мнемонику + List<*> vs List<Any?> + примеры stdlib** |
| 5 | Delegates (lazy, observable, custom) | 🟡 | Верно: lazy = создание при первом обращении + кеш (Lazy<T>); ключевое различие lazy/lateinit объяснил точно; вспомнил by remember/by viewModels. Ошибки: try-catch вместо `::prop.isInitialized` (антипаттерн); «lateinit — моветон» (нет: DI/тесты); эксепшен примерно (UninitializedPropertyAccessException). Не знал: LazyThreadSafetyMode (SYNCHRONIZED/PUBLICATION/NONE), ограничения lateinit (var, non-null, не примитивы), observable/vetoable, свой делегат = getValue/setValue операторы. **Повторить: режимы lazy, isInitialized, observable/vetoable** |
| 6 | Data/sealed classes, objects | 🟡 | Верно: генерируемые методы (3 из 5), механика бакетов hashCode, copy по сути, sealed vs enum зрело (даже компиляция), object. Пробелы: componentN не назван; контракт equals/hashCode не сформулирован словами («равные → равные hashCode, обратно нет»); ограничение sealed (наследники в том же модуле) не названо; copy с именованными аргументами + MVI/UDF кейс не назван; companion-механика не знал (реальный объект: интерфейсы, extensions, передача; @JvmStatic для Java). Дан разбор companion. **Повторить: полный список data-class методов, контракт equals/hashCode, sealed-ограничение, companion-фокус** |
| 7 | Коллекции vs Sequence | 🟡 | Верно: суть выигрыша (нет промежуточных аллокаций, «3 оператора = 3 коллекции»), практическое правило (1 оператор — смысла нет). ЯДРО не раскрыл: механика eager vs lazy («горизонтально» операция-за-операцией vs «вертикально» элемент-за-элементом), ранний выход (first() не трогает хвост), «без toList() не выполнится НИ ОДИН элемент». Терминология: перепутал «терминальный/промежуточный». sequence{}/generateSequence не знает. Дан разбор с картинкой-примером (6 вычислений+2 списка vs 2 вычисления+0). **Повторить: механику element-by-element + порог 1000+/ранний выход + builder** |
| 8 | ==/===, equals/hashCode | 🟡 | Верно: == → equals, === → ссылки; Integer cache -128..127 назвал ТОЧНО (сильный момент); суть строк Kotlin vs Java понял. Провал п.2: «почему == не падает при null» — не ответил механикой (компилируется в `a?.equals(b) ?: (b === null)`, null-ветка уходит в ===, equals не вызывается); интернирование строк не упомянул. **Повторить: формулу компиляции == (это же ответ на null-вопрос)** |
| 9 | Исключения, Result, runCatching | 🔴 | Верно: «в Kotlin все unchecked». Ошибки: п.3 ИНВЕРТИРОВАН — сказал «runCatching ловит всё КРОМЕ CancellationException», на деле ловит ВСЁ Throwable ВКЛЮЧАЯ CancellationException (главная опасность, ломает отмену корутин — kotlinx официально не рекомендует в suspend). @Throws не знает (только для Java-интеропа, в байткод throws). Операции Result не назвал. Код-ревью задачу не решил. Бонус-инсайт: «Result под запретом у нас в компании» — объяснены причины (нельзя как return type, глотание ошибок, CancellationException). **Повторить: runCatching = catch(Throwable) → CancellationException rethrow; ловить конкретные типы** |

## 🔁 Round 2 — Kotlin (пере-опрос после разбора ошибок)

> Начат 2026-09-07. Вопросы переформулированы, добавлены подвохи. Цель: закрыть 🔴 и подтянуть 🟡 до стабильных.

| # | Тема | Round 1 | Round 2 | Комментарий |
|---|------|---------|---------|-------------|
| 1 | Null-safety / smart cast | 🟡 | — | |
| 2 | Scope functions | 🔴 | — | |
| 3 | inline/noinline/crossinline, reified | 🟡 | — | |
| 4 | Variance | 🔴 | — | |
| 5 | Делегаты | 🟡 | — | |
| 6 | data/sealed, companion | 🟡 | — | |
| 7 | Sequence | 🟡 | — | |
| 8 | ==/===, equals | 🟡 | — | |
| 9 | Исключения, Result | 🔴 | — | |

## Неделя 1–2 · Coroutines + Flow

| # | Тема | Глубина | Комментарий |
|---|------|--------|-------------|
| 1 | Structured concurrency, Scope/Job/SupervisorJob | — | |
| 2 | suspend под капотом (CPS) | — | |
| 3 | Dispatchers, withContext | — | |
| 4 | Cancellation (cooperative, NonCancellable) | — | |
| 5 | Exceptions (Handler, launch vs async) | — | |
| 6 | Channels, Select, Mutex | — | |
| 7 | Flow cold/hot, операторы | — | |
| 8 | StateFlow vs SharedFlow vs LiveData | — | |
| 9 | buffer/conflate/collectLatest | — | |
| 10 | callbackFlow, тестирование | — | |

## Неделя 3 · Compose
_ожидает_

## Неделя 4 · Android Core
_ожидает_

## Неделя 5 · Архитектура + System Design
_ожидает_

## Неделя 6 · Алгоритмы
_ожидает_

---

## 📊 Промежуточные итоги

### Секция Kotlin (завершена 2026-09-07): 6×🟡 · 3×🔴 · 0×🟢
**Вывод:** база есть по всем темам, но глубины 🟢 нет нигде. Секцию бы прошёл слабо-средне: интервьюер Яндекса докопается до механики.

**Сильный момент:** В8 — Integer cache -128..127 назван точно (редкость).
**Слабейшие темы:** scope functions (таблица не знает), variance (ядро in/out), Result/runCatching (инверсия + практика).

**Переповтор перед реальным интервью (по приоритету):**
1. 🔴 **Scope functions таблица** + мнемоника «A = объект вернётся» (apply/also)
2. 🔴 **Variance**: out=producer/output, in=consumer/input, PECS; List<*> vs List<Any?>; ArrayStoreException Java
3. 🔴 **runCatching/CancellationException** — критично, мост в секцию Coroutines!
4. 🟡 Smart cast: 4 случая когда не работает + `?.`/`?:` как главные инструменты (не `!!`)
5. 🟡 Sequence: element-by-element механика, ранний выход, порог 1000+
6. 🟡 Компаньон-механика (объект, интерфейсы, @JvmStatic); lazy-режимы; data class полный список (componentN); контракт equals/hashCode; sealed-ограничение (модуль)
