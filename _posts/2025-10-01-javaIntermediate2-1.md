---
layout: post
title: "실전 자바 중급 2편 (1) 제네릭과 List 시리즈"
---
# 1. 제네릭
제네릭: 클래스나 메소드에서 사용할 데이터 타입을 지정할 수 있는 기능
제네릭을 사용해 코드 재사용, 타입 안전성을 가질 수 있다.

- `<>`: 다이아몬드
- 제네릭 타입: 객체가 생성하는 시점에 타입 인자를 전달
- 제네릭 메서드: 메서드를 호출하는 시점에서 타입 인자를 전달
제네릭 메서드가 제네릭 타입보다 높은 우선순위를 가진다.

1. 실제 사용하는 생성 시점에 타입을 결정한다.
```java
public class GenericBox<T> {

    private T value; // T는 실제 인스턴스 생성 시 지정되는 타입이 된다.

    public void set(T value) {
        this.value = value;
    }

    public T get() {
        return value;
    }
}

```
- T를 타입 매개변수라고 한다. 위 T를 Integer, String으로 변경할 수 있다.

```java
GenericBox<Integer> integerBox = new GenericBox<Integer>() // 타입 직접 입력
GenericBox<Integer> integerBox2 = new GenericBox<>() // 타입 추론
```
- 제네릭 클래스는 생성하는 시점에 <> 사이에 원하는 타입을 지정한다.
- 아래처럼 자바 컴파일러가 타입을 추론할 수 있다.

> - **자바 컴파일러**가 입력한 타입 정보를 기반으로 이런 코드가 있다고 가정하고
컴파일 과정에 타입 정보를 반영한다. 이 과정에서 타입이 맞지 않으면 **컴파일 오류**가 발생한다.

2. 메서드는 매개변수에 인자를 전달해서 사용할 값을 결정한다.
제네릭 클래스는 타입 매개변수에 타입 인자를 전달해서 사용할 타입을 결정한다.

3. 용어 정리
- 제네릭: 일반적인, 범용적인이라는 뜻으로,
특정 타입에 속한 것이 아니라 일반적으로 사용할 수 있다는 뜻이다. 
- 제네릭 타입: 클래스나 인터페이스를 정의할 때 타입 매개변수를 사용하는 것이다.
제네릭 클래스, 제네릭 인터페이스를 합친 것을 말한다.
- 타입 매개변수: 제네릭 타입이나 메서드에서 사용되는 변수로, 실제 타입으로 대체 된다.
`GenericBox<T>` ▶️ T
- 타입 인자: 제네릭 타입을 사용할 때 제공되는 실제 타입이다.
`GenericBox<Integer>`  ▶️ Integer

4. 명명 관례
대문자로 사용하며, 용도에 맞는 단어 첫글자를 사용하는 관례를 따른다.
>- E - Element
>- K - Key
>- N - Number
>- T - Type
>- V - Value
>- S,U,V etc. - 2nd, 3rd, 4th types

5. 와일드카드
와일드카드를 사용해 제네릭 타입을 더 편리하게 사용할 수 있다.

와일드카드는 * , ? 와 같이 하나 이상의 문자들을 상징하는 특수 문자를 뜻한다.
이미 만들어진 제네릭 타입을 활용할 때 사용한다.

```java
static <T> void pringV1(Box<?> box)
```

```java
//제네릭 메서드
//Box<Dog> dogBox를 전달한다. 타입 추론에 의해 타입 T가 Dog가 된다.
static <T> void printGenericV1(Box<T> box) {
	System.out.println("T = " + box.get());
}

//일반적인 메서드
//Box<Dog> dogBox를 전달한다. 와일드카드 ?는 모든 타입을 받을 수 있다.
static void printWildcardV1(Box<?> box) {
	System.out.println("? = " + box.get());
}
```

6. 타입 이레이저
제네릭은 자바 컴파일 단계(.java)에서만 사용되고, 컴파일 이후(.class)에는 제네릭 정보가 삭제된다.


# 2. List 자료구조
List는 동적으로 데이터를 추가할 수 있는 자료구조이다.

>실무팁
대부분의 경우 ArrayList가 성능상 유리하다.
데이터를 앞쪽에서 추가, 삭제할 일이 많다면 LinkedList를 고려하자.

## ArrayList
List 자료 구조를 사용하는데, 내부 데이터는 배열에 보관한다.

![](https://velog.velcdn.com/images/mymy/post/a74cf250-dfe3-4a5c-ae99-561920a700bf/image.png)

```java
package com.study.collection;

import java.util.Arrays;

// (V4) 제네릭
public class MyArrayListV1<E> {
    private static final int DEFAULT_CAPACITY = 5; // 수용량

    private Object[] elementData;
    private int size = 0;
    
    /* (V4) 제네릭 도입시, 생성자는 제네릭의 타입 매개변수를 사용할 수 없는 한계.
    따라서 배열을 생성할 때 Object 배열을 사용 -> 제네릭이 타입을 고정해줌
    -> 고정된 타입으로 Object 배열을 보관하고, 고정된 타입으로 안전하게 다운캐스팅이 가능!
     */
    public MyArrayListV1() {
        elementData = new Object[DEFAULT_CAPACITY];
    }

    public MyArrayListV1(int initialCapacity) {
        elementData = new Object[initialCapacity];
    }

    public int size() {
        return size;
    }

    public void add(String e) {
        // (V2) 값 추가
        if(size == elementData.length) {
            grow();
        }
        elementData[size] = e;
        size++;
    }

    // (V3-1) 값 추가시
    public void add(int index, Object e){
        if (size == elementData.length) {
            grow();
        }
        shiftRightFrom(index);
        elementData[index] = e;
        size++;
    }
    // (V3-1) 값 추가시 값 오른쪽으로 밀기
    private void shiftRightFrom(int index) {
        for (int i = size; i > index; i--) {
            elementData[i] = elementData[i - 1];
        }
    }

    // (V2) 값 추가시 배열 늘리기
    private void grow() {
        int oldCapacity = elementData.length;
        // 길이가 2배인 배열에 복사
        int newCapacity = oldCapacity * 2;
        elementData = Arrays.copyOf(elementData, newCapacity);
    }

    public String get(int index) {
        return (String) elementData[index];
    }

    // (V3-2) 값 삭제하기
    public String remove(int index) {
        Object oldValue = get(index);
        shiftLeftFrom(index);

        size--;
        elementData[size] = null;
        return (String) oldValue;
    }

    private void shiftLeftFrom(int index) {
        for (int i = index; i < size - 1; i++) {
            elementData[i] = elementData[i + 1];
        }
    }

    public String set(int index, E element) {
        String oldValue = get(index);
        elementData[index] = element;
        return oldValue;
    }

    public int indexOf(E o) { // 검색 기능
        for (int i = 0; i < size; i++) {
            if (o.equals(elementData[i])) {
                return i;
            }
        }
        return -1;
    }

    @Override
    public String toString() {
        return Arrays.toString(Arrays.copyOf(elementData, size)) + " size=" +
                size + ", capacity=" + elementData.length;
    }
}

```

> ⚠️`ArrayList` 단점
- 배열의 크기를 미리 확보해야 하기에 공간이 낭비된다.
- 배열의 앞이나 중간에 데이터를 추가하면 기존 데이터들이 오른쪽으로 한 칸씩 이동해야 한다.
- 삭제시, 빈 공간을 채우기 위해 왼쪽으로 이동해야 한다. (삭제시 성능 BAD)


---


## LinkedList
노드를 사용해 연결된 List를 만들어보자.

![](https://velog.velcdn.com/images/mymy/post/c48dd066-22f0-449c-b2b4-2e73a52d30d2/image.png)

- 노드를 사용해 필요한 메모리를 확보해 사용한다.
- 노드는 `데이터`와 다음 노드에 대한 `참조`를 갖고 있다.
  - 데이터 추가시, 동적으로 노드를 만들어 연결하면 된다.
  - 데이터 삭제시, 참조가 `NULL`이 되면  GC의 대상이 되어 제거된다.
  
```java
public class MyLinkedListV3<E> {
    private Node<E> first;
    private int size = 0;

    public void add(int index, Object e) {
        Node newNode = new Node(e); // 새 노드 생성
        if (index == 0) {
            // 1) 맨 앞에 삽입
            newNode.next = first; // 새 노드가 기존 첫 번째 노드를 가리킴
            first = newNode;      // 새 노드를 첫 번째로 지정
        } else {
            // 2) 중간/끝에 삽입
            Node prev = getNode(index - 1); // 삽입 위치 바로 앞 노드(prev) 찾기
            newNode.next = prev.next;       // (2단계) 새 노드가 prev.next(기존 다음 노드)를 가리킴
            prev.next = newNode;            // (3단계) prev가 새 노드를 가리키게 연결
        }
        size++;
    }

    private Node<E> getLastNode() {
        Node<E> x = first;
        while (x.next != null) {
            x = x.next;
        }
        return x;
    }

    public void add(int index, E e) {
        Node<E> newNode = new Node<>(e);
        if (index == 0) {
            newNode.next = first;
            first = newNode;
        } else {
            Node<E> prev = getNode(index - 1);
            newNode.next = prev.next;
            prev.next = newNode;
        }
        size++;
    }

    public E set(int index, E element) {
        Node<E> x = getNode(index);
        E oldValue = x.item;
        x.item = element;
        return oldValue;
    }

    public E remove(int index) {
        Node<E> removeNode = getNode(index);
        E removedItem = removeNode.item;
        if (index == 0) {
            first = removeNode.next;
        } else {
            Node<E> prev = getNode(index - 1);
            prev.next = removeNode.next;
        }
        removeNode.item = null;
        removeNode.next = null;
        size--;
        return removedItem;
    }

    public E get(int index) {
        Node<E> node = getNode(index);
        return node.item;
    }

    private Node<E> getNode(int index) {
        Node<E> x = first;
        for (int i = 0; i < index; i++) {
            x = x.next;
        }
        return x;
    }

    public int indexOf(E o) {
        int index = 0;
        for (Node<E> x = first; x != null; x = x.next) {
            if (o.equals(x.item)) return index;
            index++;
        }
        return -1;
    }

    public int size() {
        return size;
    }

    @Override
    public String toString() {
        return "MyLinkedListV3{" +
                "first=" + first +
                ", size=" + size +
                '}';
    }

    private static class Node<E> {
        E item;
        Node<E> next;

        public Node(E item) {
            this.item = item;
        }

        @Override
        public String toString() {
            StringBuilder sb = new StringBuilder();
            Node<E> temp = this;
            sb.append("[");
            while (temp != null) {
                sb.append(temp.item);
                if (temp.next != null) {
                    sb.append("->");
                }
                temp = temp.next;
            }
            sb.append("]");
            return sb.toString();
        }
    }
}

```

`LinkedList`는 단일 연결 리스트다.
노드를 앞뒤로 모두 연결하는 이중 연결 리스트를 사용하면 성능을 더 개선할 수 있다.

---

## List
자바 컬렉션 프레임워크가 제공하는 가장 대표적인 자료구조인데,
위 ArrayList나 LinkedList 또한 List에 해당된다. 사용자 입장에서는 둘 다 같은 기능을 제공한다는 것이다. 

![](https://velog.velcdn.com/images/mymy/post/1ae9d1af-5cff-436a-bc6e-e4c46b447451/image.png)

- List 인터페이스: `java.util` 패키지의 컬렉션 프레임워크의 핵심 인터페이스
 - List, Set, Queue 같은 하위 인터페이스와 함께 사용해 List, Set, Queue를 사용할 수 있다.

