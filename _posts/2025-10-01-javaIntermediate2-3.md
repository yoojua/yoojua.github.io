---
layout: post
title: "실전 자바 중급 2편 (3) 자료구조 순회와 정렬"
---

중급 2편에서 배운 다양한 자료 구조는 구조마다 데이터를 접근하는 방법이 모두 다르다.
한 번에 사용할 수 있게 다뤄보자.


# 순회
순회: 자료 구조에 있는 데이터를 차례대로 접근해 처리하는 것
자료 구조의 구현과 관계 없이 모든 자료 구조를 동일한 ㅂ아법으로 순회할 수 있는
일관성 있는 방법을 알아보자.

## Iterable, Iterator
```java
public interface Iterable<T> {
	Interator<T> iterator();
}
```

```java
public interface Iterator<E> {
	boolean hasNext();
    E next();
}
```
- hasNext(): 다음 요소가 있는지 확인한다.
- next(): 다음 요소를 반환한다.
- Iterator(반복자) 디자인 패턴이라고 불린다.

> 자료 구조에 들어있는 데이터를 처음부터 끝까지 순회하는 방법은
자료 구조에 다음 요소가 있는지 물어보고,
있으면 다음 요소를 꺼내는 과정을 반복하면 된다. 만약 다음 요소가 없다면 종료하면 된다.

![](https://velog.velcdn.com/images/mymy/post/b48a5209-aad9-4782-8145-abe17875ea71/image.png)

- 자바 컬렉션 프레임워크는 배열 리스트, 연결 리스트, 해시 셋, 연결 해시 셋, 트리 셋 등 다양한 자료 구조를 제공한다.
- 최상위 인터페이스에 Iterable이 있다는 것은 모든 컬렉션은 Iterable과
Iterator를 사용해서 순회할 수 있다는 뜻이다.
- Map의 경우 Key 뿐만 아니라 Value까지 있어 바로 순회를 할 수 없다.
  - 대신 Key나 Value를 정해 순회할 수 있다.
  - `keySet()`, `values()`를 호출하면 Set, Collection을 반환하기에 Key나 Value를 정해서 순회할 수 있다.

```java
import java.util.*;

public class JavaIterableMain {
    public static void main(String[] args) {
        List<Integer> list = new ArrayList<>();
        list.add(1);
        list.add(2);
        list.add(3);

        Set<Integer> set = new HashSet<>();
        set.add(1);
        set.add(2);
        set.add(3);

        printAll(list.iterator());
        printAll(set.iterator());

        foreach(list);
        foreach(set);
    }

    private static void printAll(Iterator<Integer> iterator) {
        System.out.println("iterator = " + iterator.getClass());
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
    }

    private static void foreach(Iterable<Integer> iterable) {
        System.out.println("iterable = " + iterable.getClass());
        for (Integer i : iterable) {
            System.out.println(i);
        }
    }
}

```

---

# 정렬
- `Arrays.sort()`

  ```java
  Arrays.sort(array, new AscComparator()); // 오름차순
  Arrays.sort(array, new DescComparator()); // 내림차순
  ```

비교자(Comparator)를 넘겨주면 알고리즘이 어떤 값이 더 큰지 두 값을 비교할 때, 비교자를 사용한다.

- Comparable 인터페이스 구현
  내가 만든 객체를 정렬할 때 필요하다.

  ```java
  public interface Comparable<T> {
      public int compareTo(T o);
  }
  ```

  - 자기 자신과 인수로 넘어온 객체를 비교해서 반환하면 된다.
    - 현재 객체가 인수로 주어진 객체보다 더 작으면 음수, 예(-1)
    - 두 객체의 크기가 같으면 0
    - 현재 객체가 인수로 주어진 객체보다 더 크면 양수, 예(1)
  - 기본 정렬 외 다른 정렬 방법을 사용해야 할 경우 비교자(Comparator)를 별도로 구현해
  정렬 메서드에 전달하면 된다.


- `Collections.sort(list)`
순서가 있는 List도 정렬을 할 수 있다.
  - 하지만 객체 스스로 정렬 메서드를 가지고 있는 `list.sort()` 사용을 더 권장한다.
  
- `list.sort(null)`
별도의 비교자가 없으므로 Comparable 로 비교해서 정렬한다.
자연적인 순서로 비교한다.

- `list.sort(new IdComparator())`
전달한 비교자로 비교한다.


> 정리
객체의 정렬이 필요한 경우 Comparable을 통해 기본 자연 순서를 제공하자.
자연 순서 외에 다른 정렬 기준이 추가로 필요하면 Comparator를 제공하자.



# 총정리
> 실무 선택 가이드
List 의 경우 대부분 ArrayList 를 사용한다.
Set 의 경우 대부분 HashSet 을 사용한다.
Map 의 경우 대부분 HashMap 을 사용한다.
Queue 의 경우 대부분 ArrayDeque 를 사용한다

> 선택 가이드
- 순서가 중요하고 중복이 허용되는 경우: List 인터페이스를 사용하자. ArrayList가 일반적인 선택이지만, 추가/삭제 작업이 앞쪽에서 빈번한 경우에는 LinkedList가 성능상 더 좋은 선택이다.
- 중복을 허용하지 않고 순서가 중요하지 않은 경우: HashSet을 사용하자. 순서를 유지해야 하면 LinkedHashSet을, 정렬된 순서가 필요하면 TreeSet을 사용하자
- 요소를 키-값 쌍으로 저장하려는 경우: Map 인터페이스를 사용하자. 순서가 중요하지 않다면 HashMap을, 순서를 유지해야 한다면 LinkedHashMap을, 정렬된 순서가 필요하면 TreeMap을 사용하자
- 요소를 처리하기 전에 보관해야 하는 경우: Queue, Deque 인터페이스를 사용하자. 스택, 큐 구조 모두 ArrayDeque를 사용하는 것이 가장 빠르다. 만약 우선순위에 따라 요소를 처리해야 한다면 PriorityQueue를 고려하자.
