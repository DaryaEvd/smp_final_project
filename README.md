# smp_final_project
Repo for smp final project course at NSU

Проект по СМП: Persitent data structures (Неизменяемые структуры данных)

# Выполняли
Евдокимова Дарья 25221
Пальчунова Олеся 25221

# Требования 1
Реализуйте библиотеку со следующими структурами данных в persistent-вариантах:
- Массив (константное время доступа, переменная длинна)
- Двусвязный список
- Ассоциативный массив (на основе Hash-таблицы, либо бинарного дерева)

Требуется собственная реализация перечисленных структур. Найти соответствующие алгоритмы также является частью задания. Язык реализации не фиксируется, но рекомендуется Java/C#/C++. Для языков типа Clojure или Haskell задание бессмысленно, т.к. такие структуры и так встроены в язык. В динамических языках (Python, Ruby, JS и т.п.) реализация будет слишком медленная и неэффективная. В базовом варианте решения все
структуры данных могут быть сделаны на основе fat-node.

# Требования 2
Должен быть единый API для всех структур, желательно использовать естественный API для выбранной платформы

# Дополнительные требования
- Обеспечить произвольную вложенность данных (по аналогии с динамическими
языками), не отказываясь при этом полностью от типизации
посредством generic/template.
- Реализовать универсальный undo-redo механизм для перечисленных структур с
поддержкой каскадности (для вложенных структур)
- Реализовать более эффективное по скорости доступа представление структур данных, чем fat-node.


# Описание выполнения работы

## Что изучали
- [Статья 1](https://neerc.ifmo.ru/wiki/index.php?title=%D0%9F%D0%B5%D1%80%D1%81%D0%B8%D1%81%D1%82%D0%B5%D0%BD%D1%82%D0%BD%D1%8B%D0%B5_%D1%81%D1%82%D1%80%D1%83%D0%BA%D1%82%D1%83%D1%80%D1%8B_%D0%B4%D0%B0%D0%BD%D0%BD%D1%8B%D1%85)
- [Статья 2](https://arpitbhayani.me/blogs/persistent-data-structures-introduction/)
- [Статья 3](https://www.geeksforgeeks.org/dsa/persistent-data-structures/)
- [Статья 4](https://softwaremill.com/persistent-data-structures-in-functional-programming/)
- [Видос с csc](https://www.youtube.com/watch?v=cSiiVofy2Tw)


## Алгоритм
Основные статьи:
- [J.R. Driscoll, Neil Sarnak, D.D. Sleator, R.E Tarjan Making Data Structures Persistent](https://www.cs.cmu.edu/~sleator/papers/another-persistence.pdf) - в пдфке прикреплено в директорию `articles`
- [Advanced Algorithms Persistent Data Structures](https://ocw.mit.edu/courses/6-854j-advanced-algorithms-fall-2005/resources/lec05_1999/) - в пдфке прикреплено в директорию `articles`


Алгоритм основан на узле, содержащем ссылку на левый узел, правый узел, значение в узле, а так же информацию о модификации левого узла, правого узла, или значения, а так же версию начиная с которой это изменение было применено.
![plot](img/pic01.png)

Для реализации структур данных на основе такого подхода, была реализована вспомогательная структура [ModificationBoxNode<T, V extends Comparable<V>>](persistent-data-structure-lib/src/main/java/ru/nsu/ccfit/persistent/data/structure/node/ModificationBoxNode.java)

## Единый API
В качестве единого API создан интерфейс
```java
/**
 * Структура данных поддерживающая операции возврата к предыдущему состоянию.
 */
public interface PersistentStructure {

    /**
     * Выполняет возврат к предыдущей версии.
     */
    void undo();


    /**
     * Отменяет возврат к предыдущей версии.
     */
    void redo();

}
```

```java
/**
 * Обновляемый узел.
 *
 * @param <T> Тип значения в узле.
 * @param <V> Тип значения версии.
 */
public class ModificationBoxNode<T, V extends Comparable<V>> {

    /**
     * Возвращает значение левого узла в запрашиваемой версии.
     *
     * @param version Версия.
     * @return Значение левого узла в запрашиваемой версии.
     */
    public ModificationBoxNode<T, V> getLeft(V version) { ... }

    /**
     * Возвращает значение правого узла в запрашиваемой версии.
     *
     * @param version Версия.
     * @return Значение правого узла в запрашиваемой версии.
     */
    public ModificationBoxNode<T, V> getRight(V version) { ... }

    /**
     * Возвращает значение в запрашиваемой версии.
     *
     * @param version Версия.
     * @return Значение в запрашиваемой версии.
     */
    public T getValue(V version) { ... }

    /**
     * Возвращает текущую модификацию узла.
     *
     * @return Текущую модификацию узла.
     */
    public ModificationBox<T, V> getModificationBox() { ... }

    /**
     * Возвращает обновленный узел.
     *
     * @param modification Обновление.
     * @return Обновленный узел.
     */
    public ModificationBoxNode<T, V> modify(ModificationBox<T, V> modification) { ... }

    /**
     * Удаляет из узла информацию о всех модификациях совершенных в версиях выше указанной.
     *
     * @param version Версия.
     */
    public void cleanFromVersion(V version) { ... }

}
```

На основе такой структуры данных реализованы структуры:
- Двусвязный список
- Ассоциативный массив

### Массив
[PersistentArray\<E>](persistent-data-structure-lib/src/main/java/ru/nsu/ccfit/persistent/data/structure/array/PersistentArray.java) реализует естественный для Java интерфейс List<E>
и основывается на структуре [ArrayHead\<E>](persistent-data-structure-lib/src/main/java/ru/nsu/ccfit/persistent/data/structure/array/utils/ArrayHead.java) - используем path copying

### Двусвязный список
[PersistentDoubleLinkedList\<V>](persistent-data-structure-lib/src/main/java/ru/nsu/ccfit/persistent/data/structure/list/PersistentDoubleLinkedList.java) реализует естественный для Java интерфейс List<V>
и основывается на структуре [ModificationBoxNode<V, Long>](persistent-data-structure-lib/src/main/java/ru/nsu/ccfit/persistent/data/structure/node/ModificationBoxNode.java)

### Ассоциативный массив
[PersistentMap<K, V>](persistent-data-structure-lib/src/main/java/ru/nsu/ccfit/persistent/data/structure/map/PersistentMap.java) реализует естественный для Java интерфейс Map<K, V>
и основывается на структуре [ModificationBoxNode<Map.Entry<K, V>, Long>](persistent-data-structure-lib/src/main/java/ru/nsu/ccfit/persistent/data/structure/node/ModificationBoxNode.java)
