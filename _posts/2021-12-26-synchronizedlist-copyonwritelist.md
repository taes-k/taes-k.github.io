---
layout: post
comments: true
title: 동기화 리스트 사용하기 (synchronizedList vs copyOnWriteList)
tags: [synchronizedList, copyOnWriteList]
---

### list 동기화의 필요성

아마 이 글을 읽으시는 독자분들 께서는 멀티쓰레드 환경에서 `list.add(...)` 혹은 `list.get(...)` 을 수행할때 오류가 발생함을 경험해보셨을것이라 생각합니다.

일반적으로는 다른 쓰레드에서 list 내부 array의 size가 변경되면서 `out-of-index`로 인해 발생하는 exception 혹은, 경쟁이 일어나면서 기대하던 데이터를 가지고오지 못하는 현상일것입니다. 

이는 쓰레드간에 경쟁을 하면서 list의 데이터가 변경되는 이유인데 해결하기 위해서는 경쟁 구간을 thread-safe하게 만들거나 가장 간단하게는, thread-safe 하게 구현된 list를 사용하는것이 해결방안 일 것입니다.

- syncrhonizedList
- copyOnWriteList

---

### synchronizedList 

먼저 `synchronizedList`을 살펴보겠습니다.

```java
// java.util.Collections

...

static class SynchronizedList<E> extends SynchronizedCollection<E> implements List<E> {
    private static final long serialVersionUID = -7754090372962971524L;

    final List<E> list;

    ...

    public E get(int index) {
        synchronized (mutex) {return list.get(index);}
    }
    public E set(int index, E element) {
        synchronized (mutex) {return list.set(index, element);}
    }
    public void add(int index, E element) {
        synchronized (mutex) {list.add(index, element);}
    }
    public E remove(int index) {
        synchronized (mutex) {return list.remove(index);}
    }

    public int indexOf(Object o) {
        synchronized (mutex) {return list.indexOf(o);}
    }
    public int lastIndexOf(Object o) {
        synchronized (mutex) {return list.lastIndexOf(o);}
    }

    ...
}

```

`java.util.Collections` 에 `List`를 상속받아 구현되어 있으며, 모든 함수에 `Synchronized` 키워드를 통해 쓰레드 세이프하게 사용 할 수 있도록 해줌을 보실 수 있습니다.


---

### CopyOnWriteArrayList

다음으로 `CopyOnWriteArrayList`을 살펴보겠습니다.

```java
// java.util.concurrent.CopyOnWriteArrayList

public class CopyOnWriteArrayList<E> implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
    private static final long serialVersionUID = 8673264195747942595L;

    final transient Object lock = new Object();

    private transient volatile Object[] array;

    ...

    public E get(int index) {
        return elementAt(getArray(), index);
    }

    public E set(int index, E element) {
        synchronized (lock) {
            Object[] es = getArray();
            E oldValue = elementAt(es, index);

            if (oldValue != element) {
                es = es.clone();
                es[index] = element;
            }
            setArray(es);
            return oldValue;
        }
    }

    public boolean add(E e) {
        synchronized (lock) {
            Object[] es = getArray();
            int len = es.length;
            es = Arrays.copyOf(es, len + 1);
            es[len] = e;
            setArray(es);
            return true;
        }
    }

    ...
}
```

위 코드에서 볼 수 있듯이 `set`, `add` 과정에서 lock과 함께 array를 copy 후 write하는 방식으로 처리를 하기때문에, `get`을 수행할때에는 thread-safe하게 기존의 데이터를 조회 할 수 있습니다.

---

### synchronizedList VS CopyOnWriteArrayList

`Silver bulllet`은 없듯이, 모든곳에서 최적인 concurrent list는 없을것입니다. 그렇다면 어떤 list를 사용하는것이 이득일까요?  

결론부터 정리해보자면 아래와 같습니다.

- read 작업량 < write 작업량 : `SynchronizedList`
- read 작업량 > write 작업량 : `CopyOnWriteList`

`SynchronizedList`는 `get` 작업에서도 락이 걸리기때문에 멀티 쓰레드 환경에서 조회가 많은 작업일경우에는 오버헤드가 더 클 것 입니다. `CopyOnWriteList`는 위에서도 살펴보았다시피 락과 함께 `set`, `add` 과정에서 데이터를 복제후 설정하는 방식으로 사용하기때문에 추가적인 오버헤드가 발생합니다. 따라서 멀티쓰레드 환경에서 어떤 작업이 주요로 수행되는가에 따라 여러분의 코드에 사용될 리스트를 선택하셔야 할 것 입니다.