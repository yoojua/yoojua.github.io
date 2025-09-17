---
layout: post
title: "실전 자바 중급 2편 (2) Set 시리즈(List, Set, Queue)"
---

# List vs Set
![](https://velog.velcdn.com/images/mymy/post/46ea3b3e-1c69-4a1c-bc4a-6b8e9c1f73cb/image.png)

- List
  - 순서유지: 리스트에 추가된 요소는 특정한 순서를 유지한다. 
  이 순서는 요소가 추가된 순서를 반영할 수 있다.
  - 중복 허용: 동일한 값이나 객체의 중복을 허용한다.
  - 인덱스 접근: 리스트의 각 요소는 인덱스를 통해 접근할 수 있다.
  - ✅용도: 순서가 중요하거나 중복된 요소를 허용해야 하는 경우 사용한다.
  
- Set
  - 유일성: 중복된 요소가 존재하지 않는다.
  - 순서 미보장: 요소를 출력할 때 입력 순서와 다를 수 있다.
  - 빠른 검색: 빠른 검색을 할 수 있게 최적화 되어있다.
  - ✅용도: 중복을 허용하지 않고, 요소의 유무만 중요한 경우에 사용한다.
  
> 예
> list: 장바구니 목록, 순서가 중요한 일련의 이벤트 목록.
> Set: 회원 ID 집합, 고유한 항목의 집합


# Set 자료구조
## Hash
- 해시인덱스: 배열의 인덱스로 사용할 수 있도록 원래의 값을 계산한 인덱스
![](https://velog.velcdn.com/images/mymy/post/279715c2-cd6f-4ae3-a3b0-3bbc43bbad9e/image.png)

```java
public class HashStart {
    static final int CAPACITY = 10;

    public static void main(String[] args) {
        //{1, 2, 5, 8, 14, 99 ,9}
        LinkedList<Integer>[] buckets = new LinkedList[CAPACITY];
        for (int i = 0; i < CAPACITY; i++) {
            buckets[i] = new LinkedList<>();
        }
        add(buckets, 1);
        add(buckets, 2);
        add(buckets, 5);
        add(buckets, 8);
        add(buckets, 14);
        add(buckets, 99);
        add(buckets, 9); //중복
        System.out.println("buckets = " + Arrays.toString(buckets));

        //검색
        int searchValue = 9;
        boolean contains = contains(buckets, searchValue);
        System.out.println("buckets.contains(" + searchValue + ") = " +
                contains);
    }

    private static void add(LinkedList<Integer>[] buckets, int value) {
        int hashIndex = hashIndex(value);
        LinkedList<Integer> bucket = buckets[hashIndex]; // O(1)
        if (!bucket.contains(value)) { // O(n)
            bucket.add(value);
        }
    }

    private static boolean contains(LinkedList<Integer>[] buckets, int
            searchValue) {
        int hashIndex = hashIndex(searchValue);
        LinkedList<Integer> bucket = buckets[hashIndex]; // O(1)
        return bucket.contains(searchValue); // O(n)
    }

    static int hashIndex(int value) {
        return value % CAPACITY;
    }
}

```

> ⚠️하지만 값들이 같은 해시 인덱스에 겹치면 성능이 매우 떨어진다.
그래도 확률적으로 어느정도 넓게 값이 퍼지기에 대부분 O(1)의 성능을 제공한다.
통계적으로 입력한 데이터의 수가 배열의 크기를 75% 넘지 않으면 자주 충돌하지 않는다.


---

## HashSet
> ⚠️위 Hash는 데이터를 추가할 때 중복 데이터가 있는지 전체 데이터를 항상 확인해야 한다.
데이터를 찾을 때 모든 데이터를 찾고 비교해야 한다. O(n)

![](https://velog.velcdn.com/images/mymy/post/5afc6cda-a591-4989-bd7c-4ce371d33170/image.png)

- 해시코드: `hashCode()`메서드를 통해 만들어진 코드
- Object.hashCode(): 모든 객체를 자신만의 해시 코드를 표현할 수 있는 기능을 제공한다.
정수형, 문자형, 개발자가 직접 정의한 사용자 정의타입까지 제공할 수 있다.
- equals(): 해시 인덱스가 같아도 실제 저장된 데이터는 다를 수 있기에, 데이터가 하나만 있어도 `equals()`로 찾는 데이터가 맞는지 검증해야 한다.
  - 동일성 비교
  - IDE의 도움으로 구현 가능

```java
public interface MySet<E> {
    boolean add(E element);
    boolean remove(E value);
    boolean contains(E value);
}
```

```java
import java.util.Arrays;
import java.util.LinkedList;

public class MyHashSetV3<E> implements MySet<E> {
    static final int DEFAULT_INITIAL_CAPACITY = 16;
    private LinkedList<E>[] buckets;
    private int size = 0;
    private int capacity = DEFAULT_INITIAL_CAPACITY;

    public MyHashSetV3() {
        initBuckets();
    }

    public MyHashSetV3(int capacity) {
        this.capacity = capacity;
        initBuckets();
    }

    private void initBuckets() {
        buckets = new LinkedList[capacity];
        for (int i = 0; i < capacity; i++) {
            buckets[i] = new LinkedList<>();
        }
    }

    @Override
    public boolean add(E value) {
        int hashIndex = hashIndex(value);
        LinkedList<E> bucket = buckets[hashIndex];
        if (bucket.contains(value)) {
            return false;
        }
        bucket.add(value);
        size++;
        return true;
    }

    @Override
    public boolean contains(E searchValue) {
        int hashIndex = hashIndex(searchValue);
        LinkedList<E> bucket = buckets[hashIndex];
        return bucket.contains(searchValue);
    }

    @Override
    public boolean remove(E value) {
        int hashIndex = hashIndex(value);
        LinkedList<E> bucket = buckets[hashIndex];
        boolean result = bucket.remove(value);
        if (result) {
            size--;
            return true;
        } else {
            return false;
        }
    }

    private int hashIndex(Object value) {
        // hashCode의 결과로 음수가 나올 수 있다. abs()를 사용해서 마이너스를 제거한다.
        return Math.abs(value.hashCode()) % capacity;
    }

    public int getSize() {
        return size;
    }

    @Override
    public String toString() {
        return "MyHashSetV3{" +
                "buckets=" + Arrays.toString(buckets) +
                ", size=" + size +
                ", capacity=" + capacity +
                '}';
    }
}

```

---

## Set
![](https://velog.velcdn.com/images/mymy/post/c1baa63a-ef8c-4f33-bf47-0f16095704f5/image.png)
- Collection 인터페이스
java.util 패키지의 컬렉션 프레임워크의 핵심 인터페이스 중 하나다.
Collection 인터페이스는 List, Set, Queue 같은 하위 인터페이스와 함께 사용되어
리스트, 세트, 큐 형태로 관리할 수 있다.


---

## TreeSet
### 트리구조
![](https://velog.velcdn.com/images/mymy/post/ded6e6ea-1fcb-4860-8c38-99d5e352b2f5/image.png)

트리는 부모 노드와 자식 노드로 구성된다.
가장 높은 조상이 루트이고, 자식이 2개까지 올 수 있는 트리를 이진 트리라고 한다.
위 그림처럼 노드의 왼쪽 자손은 더 작은 값을 가지고, 오른쪽 자손은 더 큰 값을 가지는 것을 `이진 탐색 트리`라고 한다.
TreeSet은 이진 탐색 트리를 개선한 `레드-블랙 트리`를 사용한다.

![](https://velog.velcdn.com/images/mymy/post/faedde9a-d7c9-40f1-b674-18dfea117156/image.png)

```java
class Node {
	Object item;
    Node left;
    Node right;
   	}
```


# Map, Stack, Queue
![](https://velog.velcdn.com/images/mymy/post/b0233af8-6735-4e74-95ad-d9caff9e17c6/image.png)
- Map은 인터페이스이기에 인스턴스를 직접 생성할 수 없고
HashMap, TreeMap, LinkedHashMap 등 다양한 Map 구현체를 사용해 쓸 수 있다.
  - 이 중 HashMap을 가장 많이 사용한다.
- Map은 키-값 쌍으로 저장하는 자료 구조이다.
키는 중복을 허용하지 않고, 순서를 보장하지 않는다.
  - Set과 거의 같은 구조이다.

- HashMap 예시
```java
import java.util.Collection;
import java.util.HashMap;
import java.util.Map;
import java.util.Set;

public class MapMain1 {
    public static void main(String[] args) {
        Map<String, Integer> studentMap = new HashMap<>();

        // 학생 성적 데이터 추가
        studentMap.put("studentA", 90);
        studentMap.put("studentB", 80);
        studentMap.put("studentC", 80);
        studentMap.put("studentD", 100);

        System.out.println(studentMap);

        // 특정 학생의 값 조회
        Integer result = studentMap.get("studentD");
        System.out.println("result = " + result);

        System.out.println("keySet 활용");
        Set<String> keySet = studentMap.keySet();
        for (String key : keySet) {
            Integer value = studentMap.get(key);
            System.out.println("key=" + key + ", value=" + value);
        }

        System.out.println("entrySet 활용");
        Set<Map.Entry<String, Integer>> entries = studentMap.entrySet();
        for (Map.Entry<String, Integer> entry : entries) {
            String key = entry.getKey();
            Integer value = entry.getValue();
            System.out.println("key=" + key + ", value=" + value);
        }

        System.out.println("values 활용");
        Collection<Integer> values = studentMap.values();
        for (Integer value : values) {
            System.out.println("value = " + value);
        }
    }
}

```

- Stack 예시
```java
import java.util.ArrayDeque;
import java.util.Deque;

public class SimpleHistoryTest {
    public static void main(String[] args) {
        Deque<String> stack = new ArrayDeque<>();

        stack.push("youtube.com");
        stack.push("google.com");
        stack.push("facebook.com");

        System.out.println(stack.pop());
        System.out.println(stack.pop());
        System.out.println(stack.pop());
    }
}

```


- Queue 예시
```java
import java.util.LinkedList;
import java.util.Queue;

public class PrinterQueueTest {
    public static void main(String[] args) {
        Queue<String> printQueue = new LinkedList<>();

        printQueue.offer("doc1");
        printQueue.offer("doc2");
        printQueue.offer("doc3");

        System.out.println("출력: " + printQueue.poll());
        System.out.println("출력: " + printQueue.poll());
        System.out.println("출력: " + printQueue.poll());
    }
}

```
