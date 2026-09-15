# 🧩 Слабое место #3 · Хитрые задачи: equals / hashCode (по фидбеку Яндекса)

> Причина: фидбек этапа 1 Яндекса (сентябрь 2026) — «запутался в контрактах, не решил задачу с hashMap без hashCode».
> Формат: задача → подумай сам → ответ+разбор ниже. Прогонять руками, вслух.
> Обновлено: 2026-09-14

---

## 🔑 Контракт (повторение перед задачами)

1. `a == b` (equals true) → **обязаны** `hashCode(a) == hashCode(b)`
2. Равные hashCode → equals **НЕ обязателен** (это коллизия — легально)
3. equals: **рефлексивность** (a=a), **симметричность** (a=b ⇔ b=a), **транзитивность** (a=b, b=c → a=c), **консистентность** (не меняется при неизменных полях), `x.equals(null)` = false
4. Нарушил любой пункт → HashSet/HashMap ведут себя непредсказуемо

---

## Задача 1 · Сколько в HashSet?

```kotlin
class P(val x: Int)                      // ОБЫЧНЫЙ класс, ничего не переопределено
val set = hashSetOf(P(1)); set.add(P(1))
println(set.size)                        // ?
```
<details><summary>Ответ</summary>

**2.** Без equals/hashCode работает identity: два объекта `P(1)` — разные инстансы → разные hashCode → разные бакеты → оба лежат. (С `data class` было бы 1.)
</details>

## Задача 2 · HashMap без hashCode (та самая с интервью!)

```kotlin
class K(val id: Int) {
    override fun equals(other: Any?) = (other as? K)?.id == id
    // hashCode НЕ переопределён
}
val map = hashMapOf(K(1) to "a")
println(map[K(1)])                       // ?
```
<details><summary>Ответ</summary>

**null.** Сначала HashMap считает hashCode ключа поиска → identity-хэш ≠ хэшу ключа при вставке → идёт **не в тот бакет** → equals даже не вызовется. Значение «потеряно», хотя «равный» ключ лежит. Фикс: переопределить hashCode согласованно (= id).
</details>

## Задача 3 · Мутабельный ключ (самая хитрая!)

```kotlin
data class Key(var id: Int)
val map = hashMapOf(Key(1) to "v")
val k = map.keys.first()
k.id = 2                                 // изменили ПОСЛЕ вставки
println(map[Key(2)])                     // ?
```
<details>с<summary>Ответ</summary>

**null!** Ключ положен в бакет по hashCode(id=1). После `k.id = 2` его hashCode изменился, но объект остался **в старом бакете**. Поиск по `Key(2)`: новый hashCode → новый бакет → там пусто. Объект «застрял» не в своём бакете. **Правило: никогда не мутировать поля ключа/элемента set после вставки** (или не включать mutable-поля в equals/hashCode).
</details>

## Задача 4 · hashCode есть, equals нет

```kotlin
class C(val x: Int) {
    override fun hashCode() = x
}
val set = hashSetOf(C(1)); set.add(C(1))
println(set.size)                        // ?
println(set.contains(C(1)))              // ?
```
<details><summary>Ответ</summary>

**size = 2, contains = false.** Коллизия отправила оба в ОДИН бакет, но equals (identity) сказал «разные» → оба сохранились. `contains(C(1))` находит бакет по hashCode, сравнивает equals → false. Равные hashCode без equals = просто коллизия, легальная, но set хранит «дубликаты по смыслу».
</details>

## Задача 5 · HashSet и удаление

```kotlin
data class D(val id: Int)
val set = hashSetOf(D(1))
set.add(D(1))
set.remove(D(1))
println(set.size)                        // ?
```
<details><summary>Ответ</summary>

**0.** data class: equals+hashCode по id согласованы → D(1) в set = D(1) поиск → найден и удалён. remove работает по тому же контракту, что и add/contains.
</details>

## Задача 6 · Нарушение симметричности (Effective Java, классика)

```kotlin
open class Point(val x: Int, val y: Int) {
    override fun equals(other: Any?) =
        other is Point && other.x == x && other.y == y
}
class ColorPoint(x: Int, y: Int, val color: Int) : Point(x, y) {
    override fun equals(other: Any?) =
        other is ColorPoint && super.equals(other) && other.color == color
}
val p = Point(1, 2); val cp = ColorPoint(1, 2, 1)
println(p == cp)                         // ?
println(cp == p)                         // ?
```
<details><summary>Ответ</summary>

**true, потом false** — симметричность НАРУШЕНА! `p == cp`: cp is Point ✅, координаты равны → true. `cp == p`: p is ColorPoint ❌ → false. Один и тот же вопрос — разные ответы. В HashSet с такими объектами возможны «зависит от порядка добавления» баги. Решение из Effective Java: не наследовать с расширением equals — композиция вместо наследования или `other.javaClass == javaClass`.
</details>

## Задача 7 · TreeMap без hashCode

```kotlin
val tm = sortedMapOf(compareBy { (it as Pair<*, *>).first }, 1 to "a")
// сравните: влияет ли hashCode ключей на TreeMap?
```
<details><summary>Ответ</summary>

**Нет.** TreeMap/TreeSet используют ТОЛЬКО Comparator/Comparable — hashCode вообще не участвует. Хитрость наоборот: у TreeMap свои ловушки (несогласованный comparator), но hashCode-контракт к нему отношения не имеет. На интервью: «Sorted-структуры = сравнение, Hash-структуры = hashCode+equals».
</details>

## Задача 8 · data class с var

```kotlin
data class V(var x: Int)
val set = hashSetOf(V(1))
val v = set.first(); v.x = 5
println(set.contains(V(1)))              // ?
println(set.contains(V(5)))              // ?
```
<details><summary>Ответ</summary>

**false и false!** Объект лежит в бакете по hashCode(x=1). `V(5)` ищет по hashCode(x=5) → не тот бакет. `V(1)` ищет по верному бакету, но equals сравнит с объектом, у которого x уже 5 → не равны. Элемент «неищем» вовсе. Иллюстрация задачи 3 через data class с var.
</details>

## Задача 9 · contains по старому инстансу (задача 3-бис)

```kotlin
data class M(val id: Int)
val set = hashSetOf(M(1))
val m = set.first()
set.remove(m)                            // удалить ТЕМ ЖЕ инстансом после смены поля? — поле не меняли
println(set.size)
```
<details><summary>Ответ</summary>

**0** — тривиально, НО сравни с задачей 3: если бы перед remove у m сменили id → remove(false), объект остался бы. Контраст «тот же инстанс + без мутации» vs «мутация между вставкой и поиском» — любимая пара кейсов интервьюеров.
</details>

## Задача 10 · Собери сам (синтез)

Напиши класс `Money(val amount: Int, val currency: String)` с корректными equals/hashCode, чтобы проходило:
```kotlin
hashSetOf(Money(100, "USD")).contains(Money(100, "USD")) == true
```
И объясни, что вернёт `map[Money(100, "USD")]`, если hashCode вернуть константой `= 42`?

<details><summary>Ответ</summary>

Константный hashCode — **легален** (контракт: равные → равные ✅), contains/get будут работать, НО все объекты свалятся в один бакет → HashMap выродится в список → O(n) вместо O(1). Ответ: работать будет, но производительность убита.
</details>

---

## 🎯 Шпаргалка-выводы

1. **equals без hashCode** → get/contains вернут null/false (не тот бакет), «равные» дубли в set
2. **hashCode без equals** → всё в одном бакете, но «разные» → дубли сохраняются
3. **Мутация полей ключа после вставки** → объект застревает в старом бакете → «неищем»
4. **Sorted структуры** игнорируют hashCode (только comparator)
5. **Константный hashCode** — легален, но O(n)-деградация
6. Наследование + расширение equals → нарушение симметричности (композиция!)

---
_Связанные: [Трекер моков](../mock-interview-tracker.md) · [Шпаргалка блок 5-6](../cheatsheet.md)_
