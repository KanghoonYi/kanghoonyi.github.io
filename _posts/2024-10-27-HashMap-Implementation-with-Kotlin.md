---
title: HashMap Implementation with Kotlin
author: KanghoonYi(pour)
name: KanghoonYi(pour)
date: 2024-10-27 03:22:00 +0900
categories: [Programming, kotlin]
tags: [Computer Science, programming, Data Structure, kotlin]
pin: true
math: true
---

## Overview

Hashmap은 Software의 탐색 속도를 개선하는데 활용하게 됩니다.  
이때, Hashmap이 어떻게 구현(implementation)되어 있는지 확인해보려 합니다.

## Source Code

```kotlin
import java.io.Serializable
import kotlin.collections.*

object EmptyIterator : ListIterator<Nothing> {
    override fun hasNext(): Boolean = false
    override fun hasPrevious(): Boolean = false
    override fun nextIndex(): Int = 0
    override fun previousIndex(): Int = -1
    override fun next(): Nothing = throw NoSuchElementException()
    override fun previous(): Nothing = throw NoSuchElementException()
}

object EmptySet : Set<Nothing>, Serializable {
    private const val serialVersionUID: Long = 3406603774387020532

    override fun equals(other: Any?): Boolean = other is Set<*> && other.isEmpty()
    override fun hashCode(): Int = 0
    override fun toString(): String = "[]"

    override val size: Int get() = 0
    override fun isEmpty(): Boolean = true
    override fun contains(element: Nothing): Boolean = false
    override fun containsAll(elements: Collection<Nothing>): Boolean = elements.isEmpty()

    override fun iterator(): Iterator<Nothing> = EmptyIterator

    private fun readResolve(): Any = EmptySet
}

class HashMapImpl<K, V> constructor() {
    private val defaultCapacity = 16
    private var loadFactor = 0.75f
    private var size = 0
    private var sizeThreshold = (defaultCapacity * loadFactor).toInt()
    val entries: Set<Map.Entry<K, V>> get() = EmptySet

    private class Bucket<K, V> constructor(val key: K?, val value: V?) {
        var next: Bucket<K, V>? = null;
    }

    private var _buckets: Array<Bucket<K, V>?>? = null;
    private var buckets: Array<Bucket<K, V>?>
        get() {
            if (_buckets == null) {
                _buckets = kotlin.arrayOfNulls<Bucket<K, V>>(defaultCapacity);
            }
            return _buckets ?: throw AssertionError("Set to null by another thread");
        }
        set(value) {
            _buckets = value
        }

    private var _values: Collection<V>? = null
    val values: Collection<V>
        get() {
            if (_values == null) {
                _values = object : AbstractCollection<V>() {
                    override operator fun contains(element: @UnsafeVariance V): Boolean = containsValue(element)

                    override operator fun iterator(): Iterator<V> {
                        val entryIterator = entries.iterator()
                        return object : Iterator<V> {
                            override fun hasNext(): Boolean = entryIterator.hasNext()
                            override fun next(): V = entryIterator.next().value
                        }
                    }

                    override val size: Int get() = this@HashMapImpl.size
                }
            }
            return _values!!
        }

    private var _keys: AbstractSet<K>? = null;
    val keys: AbstractSet<K>
        get() {
            if (_keys == null) {
                _keys = object : AbstractSet<K>() {
                    override operator fun contains(element: K): Boolean = containsKey(element)

                    override operator fun iterator(): Iterator<K> {
                        val entryIterator = entries.iterator()
                        return object : Iterator<K> {
                            override fun hasNext(): Boolean = entryIterator.hasNext()
                            override fun next(): K = entryIterator.next().key
                        }
                    }

                    override val size: Int get() = this@HashMapImpl.size
                }
            }
            return _keys!!
        }

    fun containsKey(key: K): Boolean {
        return implFindEntry(key) != null
    }

    fun put(key: K, value: V): V {
        val index = getIndex(key);
        val bucket = Bucket(key, value);

        if (index in buckets.indices) {
            // depth 1단계에 bucket이 이미 존재한다면, linked list로 연결.
            buckets[index]?.next = bucket;
        } else {
            buckets[index] = bucket;
        }
        size += 1

        if (size >= sizeThreshold) {
            resize()
        }

        return value;
    }

    fun get(key: K): V? {
        val index = getIndex(key);
        if (index !in buckets.indices) {
            return null;
        }

        var bucketHead = buckets[index] ?: return null;

        while (bucketHead.next != null && bucketHead.key != key) {
            bucketHead = bucketHead.next!!;
        }
        return bucketHead.value;
    }

    private fun getIndex(key: K): Int {
        return Math.floorMod(key.hashCode(), buckets.size)
    }

    private fun resize() {
        val tempBuckets = buckets;
        buckets = Array(buckets.size * 2) { null };

        tempBuckets.forEachIndexed { idx, bucket ->
            buckets[idx] = bucket
        }
    }

    private fun implFindEntry(key: K): Map.Entry<K, V>? = entries.firstOrNull { it.key == key }
    private fun containsValue(value: V): Boolean = entries.any { it.value == value }

    fun size() = size
}

fun main() {
		// test를 위한 code
    val hashMap = HashMapImpl<String, String>();

    hashMap.put("firstKey", "firstValue");
    hashMap.put("secondKey", "secondValue")
    assert(hashMap.keys == hashSetOf(arrayOf("firstKey", "secondKey")));
    assert(hashMap.get("secondKey") == "secondValue");
}
```

## 해설

Hashmap안에, 실제로 값(key, value 모두)이 저장되는건 Array의 element로 저장됩니다.  
처음 HashMap을 생성할때, 특정 값(`defaultCapacity`)에 해당하는 길이(size)만큼의 Array를 생성해서 사용합니다.

### Bucket

Hashmap내부에선 key와 value를 `Bucket` 이라는 instance를 생성하여 HashMap의 array에 추가합니다.  
만약 값을 추가할 때(put 할 때), ‘hash collision’이 발생하면 Bucket을 ‘Linked List’형태로 추가하게 됩니다.

![HashMap 내부 데이터 구조.
From [https://oshyshkov.com/2021/07/07/java-hashmap-implementation/](https://oshyshkov.com/2021/07/07/java-hashmap-implementation/)](/assets/img/for-post/HashMap%20Implementation%20with%20Kotlin/image.png)
__HashMap 내부 데이터 구조. From [https://oshyshkov.com/2021/07/07/java-hashmap-implementation/](https://oshyshkov.com/2021/07/07/java-hashmap-implementation/)__

### Rehashing

HashMap에 값이 많이 추가될 수록, LinkedList의 길이가 커지며, HashMap 성능인 $O(n)$의 이점을 누리기가 어려워집니다. 그래서 `Rehashing` 이라는 작업이 이루어 집니다.  
`Rehashing`은 특정 threshold(임계점)을 기준으로, 이보다 더 많은 element가 쌓이면 이루어집니다.  
내부 Array를 더 큰 사이즈(보통 2배 크기)로 다시 생성하여, **기존 Array의 값을 모두 copy합니다.**

### loadFactor

HashMap에서 `loadFactor`는 해시 맵의 **성능과 효율성**을 관리하는 중요한 요소입니다. `loadFactor`는 해시 맵이 얼마나 채워지면 **리해싱(rehashing)**을 수행할지를 결정하는 기준입니다.  
`loadFactor`는 해시 맵이 **얼마나 채워졌는지**를 rate(비율) 값으로 나타내며, 기본적으로는 **0.75**로 설정됩니다. 즉, **현재 크기** 대비 **허용 가능한 최대 항목 수**를 정의합니다.

$$
\text{loadFactor}=(\text{저장된 항목 수}/\text{해시 맵의 버킷 array 크기})
$$

## Q&A

### 왜 `loadFactor = 0.75`가 기본값일까?

**성능과 메모리의 균형을 갖춘 값입니다.**
: 0.75는 일반적으로 충돌을 최소화하면서도 메모리 사용을 적절히 유지하는 값으로 알려져 있다고 합니다. 너무 낮은 loadFactor를 설정하면 리해싱이 자주 발생해서 성능이 떨어질 수 있고, 너무 높은 loadFactor를 설정하면 충돌이 자주 발생하여 탐색 속도가 느려질 수 있습니다.

### 왜 ArrayList가 아니라 Array를 사용하는가?

어차피 Rehashing과정이 있다면, 이를 Dynamic Array인 ‘ArrayList’를 사용하면 편하지 않을까요?  
- `ArrayList`는 동적(dynamic)으로 크기를 조정하면서 빈 공간을 유지할 수 있습니다. 이는 메모리 낭비를 초래할 수 있습니다. 반면에, `Array`는 고정된 크기의 메모리를 사용하므로, `HashMap`에서는 메모리 할당을 보다 **효율적으로 관리**할 수 있습니다.
- `HashMap`에서도 `ArrayList`와 같이 메모리 재할당이 이루어지긴 합니다.  
  하지만, ArrayList에서는 기존의 순서대로 Array를 복사하는 반면, **HashMap에서는 Bucket이 들어가야 하는 index 위치도 바꿔줍니다(key값을 기준으로 Hashing을 다시 진행하기 때문에).**  
  이는, **기존의 Linked List 형태로 있던 데이터를 고르게 분포시키는 효과**가 있습니다.

## Conclusion

HashMap의 간단한 동작 과정을 정리하며 마무리 합니다.

1. **해시 계산**: 입력된 키에 대해 해시값을 계산합니다.
2. **배열 인덱스 계산**: 해시값을 배열의 인덱스로 변환하여, 어느 위치에 저장할지 결정합니다.
3. **버킷 저장**: 배열의 해당 인덱스에 데이터를 저장하고, 충돌이 발생하면 연결 리스트 또는 트리로 관리합니다.
4. **리해싱**: 배열이 꽉 차면 **리해싱**을 통해 배열 크기를 두 배로 늘리고 데이터를 재배치합니다.

## References

Kotlin의 HashMap Spec확인했던 곳
: [HashMap - Kotlin Programming Language](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-hash-map/)

Kotlin Source Code
: [https://github.com/JetBrains/kotlin/blob/09273fdd828836a1c45dcbb877b2aa48ef9da80a/libraries/stdlib/src/kotlin/collections/AbstractMap.kt](https://github.com/JetBrains/kotlin/blob/09273fdd828836a1c45dcbb877b2aa48ef9da80a/libraries/stdlib/src/kotlin/collections/AbstractMap.kt)

또 다른 Kotlin구현 example
: [https://github.com/darian-catalin-cucer/HashMap](https://github.com/darian-catalin-cucer/HashMap)
