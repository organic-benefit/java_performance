# Garbage collection에 대해 알아본다.

- http://www.slideshare.net/novathinker/3-garbage-collection
- GC Algorithm

<br>
<br>
<br>

## Garbage collection

<br>
<br>

### Storage(memory) management

프로그래머가 메모리 관리하는 경우

`(물리적인 메모리 -> 프로그램 메모리) -> 메모리 할당 -> 메모리 해제`

프로그램이 자동으로 관리하는 경우(Garbage collector)

`메모리 할당 -> 쓸모 없어진 혹은 사용하지 않는 Object 의 메모리 해제`

<br>
<br>

### JVM

```php
Heap storage for objects is reclaimed 
by an automatic storage management system(typically a garbage collector);
objects are never explicitly deallocated.

JVM 은 `가비지 컬렉터` 같은 자동화 시스템으로 Object가 저장되는 heap 메모리의 여유공간을 확보한다.
Object의 메모리 해제는 `절대` 직접 할 수 없다.
```

<br>
<br>

### Garbage object : 현재 사용하고 있지 않는 Object

- Root Set과 어떤 식으로든 Reference 관계가 있다면 Reachable object => 현재 사용하고 있는 Object로 간주

- Root Set?
 
 - Stack의 참조 정보 : Local Variable Section, Operand Stack에 Object의 Reference 정보가 있는 경우 => 현재의 Thread에서 사용하고 있는 정보들이기 때문.
 - Method Area > Constant Pool 에 있는 Reference 정보 : Constant Pool을 통해 간접적으로 Link된 Reachable object
 - 메모리에 남아 있는 Native Method로 넘겨진 Object reference : JNI 형태로 현재 참조관계가 있기 때문

 ![Root Set](https://github.com/doublemetal/doublemetal.github.io/blob/master/images/study/root-set.PNG)

<br>
<br>

### Memory Leak

- Reachable But not Live 

![ReachableObject](https://github.com/doublemetal/doublemetal.github.io/blob/master/images/study/garbage-object-reachable.jpg)

```php
class Main {
    public static void main (String[] args) {
        Leak leak = new Leak();
        for(int i=0; i<99999999; i++) {
            leak.addList(i);
            leak.removeStr(i);
        }
}
class Leak {
    List<String> list = new ArrayList<>();
    public void addList(int number) {
        list.add("Count number : " + number);
    }

    public void removeStr(int i) {
        Object obj = list.get(i);
        obj = null; // ???
    }
}
```

<br>
<br>

### When do garbage collect? and how?

- 메모리가 부족할 때
- 메모리를 해제하여 Heap 메모리의 가용공간을 늘려 사용한다. 

 => Fragmentation(단편화)이 발생하기도 한다.
 => Fragmentation 문제를 해결하는 Algorithm 적용(Compaction?)

<br>
<br>

## GC Algorithm

- Garbage Collection이 알려진 것은 Java의 영향이 크지만, 시작은 Java가 아니다.
 
 - Perl, Python, PHP, .NET, Unix File System, Adobe Flash, Photoshop

- 하지만, Java가 나온 후 JVM 벤더에서 자신들의 JVM에 맞도록 Algorithm도 발전

- 이제부터 설명할 Algorithm들은 JVM에서 핵심적인 것들로 이 Algorithm을 바탕으로 발전된 것들이 많기 때문에 도움이 될 수 있을 것이다.

- 어떤 Algorithm 이 있는가 : 
 
 - Reference Counting Algorithm
 - Mark-and-Sweep Algorithm
 - Mark-and-Compacting Algorithm
 - Copying Algorithm
 - Generational Algorithm
 - Train Algorithm

<br>
<br>

### 1

- Find Garbage & Remove Garbage
- 초기의 Algorithm은 Garbage를 찾는데에 집중 => 어떻게 제거해서 Heap을 재활용하는가에 초점
- 1번은 Reference Count를 관리하여 Reference Count가 0이 될 때마다 GC를 수행하는 것

 - `Reference Counting Algorithm`

 ![example](https://github.com/doublemetal/doublemetal.github.io/blob/master/images/study/rca-1.PNG)

- Garbage Object 를 확인하는 작업이 너무나도 간단하다.
- Reference Count가 0이 될 때마다 GC가 일어나기 때문에 Pause Time이 분산되어 실시간 작업에도 거의 영향을 주지 않는다.
- 하지만, count를 관리하는 비용이 크고, Object가 Garbage가 될 때, GC가 연쇄적으로 일어날 수 있다.
- Linked List에서 Memory Leak이 발생할 가능성이 높다.

 - 순환 구조인 Linked List에서 HEAD의 참조변수가 미아가 되는 경우, 실제 Linked List는 Garbage여야 하지만, 참조가 끊기지 않기 때문에, Reference Count는 0이 되지 않고, GC도 일어나지 않음.

<br>
<br>

### 2

- Reference Counting Algorithm의 단점 극복
- Root Set에서 Reference를 추적하는 방식 > 효과적인 Garbage Detection  => Detection에 자주 사용됨
- Mark Phase : Object header(flag), Bitmap Table로 marking
- Sweep Phase : Mark Phase 후 바로 시작, marking되지 않은 Object를 모두 지움 => marking 초기화

![makAndSweep](https://github.com/doublemetal/doublemetal.github.io/blob/master/images/study/makAndSweep.jpg)

- Reference 관계의 변경 시에 부가적인 작업(Counting)을 하지 않기 때문에 속도 향상
- GC 중 Heap 사용이 제한됨 => Mark 작업의 정확성 및 Memory Corruption(잘못된 주소 참조 등)을 방지하기 위함
 - 프로그램이 잠깐 멈추는 현상(Suspend, HotspotJVM의 Stop the world) 발생
- Fragmentation : Sweep Phase 후 가용한 공간이 듬성듬성 생기는 것 => OOM 발생 가능
- `Mark and Sweep Algorithm(Tracing Algorithm)`

<br>
<br>

### 3

- Fragmentation 극복 => Compaction(Sweep)
- Compaction : Live Object를 메모리에 차곡차곡 다시 적재하는 작업

![compaction](https://github.com/doublemetal/doublemetal.github.io/blob/master/images/study/compaction.jpg)

<br>
<br>
<br>
<br>
<br>
<br>

:Compaction 방식
- Arbirary : 무작위 방식
- Lenear : 메모리 참조 순서
- Sliding : 메모리 할당 순서

- Sliding이 제일 좋다.
- Compaction 시에 Handle 자료구조를 사용할 수도 있다.

![markPhase](https://github.com/doublemetal/doublemetal.github.io/blob/master/images/study/markPhase.jpg)

![compactionPhase](https://github.com/doublemetal/doublemetal.github.io/blob/master/images/study/compactionPhase.jpg)

- 장점 : Fragmentation이 최소화 되어 메모리 공간을 효율적으로 사용
 - Compaction 이후 Reference를 업데이트할 때의 Overhead 가능성 => 모든 Object Access
- Mark Phase, Compaction Phase 모두 Suspend 발생
- 'Mark and Compaction Algorithm'

<br>
<br>

### 4

- Fragmentation 이슈 해결을 위해 나온 Algorithm
- Heap => Active, Inactive
 - Active에만 Object 할당 가능
 - Active가 꽉 차면, GC 수행 => GC 후에 Active가 Free memory가 되면, 영역을 뒤바꿈(Scavenge)
- GC => Suspend => Copy(Live Object -> Inactive) => Copy가 되면, 자연스럽게 Compaction까지 된다!

![Copying](https://github.com/doublemetal/doublemetal.github.io/blob/master/images/study/copying.jpg)

- Fragmentation의 방지에는 효과적이지만, Heap의 절반만 사용가능
- Suspend와 Copy Overhead는 피할 수 없다.
- `Copying Algorithm(Stop the Copying Algorithm)`

<br>
<br>

### 5

- 대부분의 프로그램에서 생성되는 대다수의 Object는 생성된 지 얼마 되지 않아 Garbage가 된다.
- 간혹 오래 사는 녀석도 분명히 있다.
- Copying Algorithm에서 발전 => Active, Inactive => Age별로 Heap을 나눔
- Youngest Generation Sub Heap에 Object 할당 => 나이를 먹음(Age에 맞는 Sub Heap으로 이동) ... => Oldest Generation Sub Heap
 - Promotion : Age가 임계값을 넘어 다음 Generation으로 Copy되는 것

![Generation](https://github.com/doublemetal/doublemetal.github.io/blob/master/images/study/generational.jpg)

- Mark Phase에서 Dead, Live, Matured 상태 구별
 - Promotion될 정보의 Age가 되지 않은 Live Object는 Age를 하나 추가함
- `Generational Algorithm`

<br>
<br>

### 6

- `Train Algorithm(Incremental Algorithm)`
- Suspend를 줄이려는 노력에서 탄생
- Heap을 쪼개서 => Memory Block => Block 단위로 GC 수행(Mark Phase, Copy Phase)
- Block에 대해서만 Suspend가 발생 => 분산으로 전체 Pause Time을 줄이는 아이디어

- Train(Car1, Car2, Car3, ... Car(n)) => Car는 고정크기의 Memory Block
- Add Train or Add Car 
- Remember Set : Car 외부에서의 참조를 기억

![Train](https://github.com/doublemetal/doublemetal.github.io/blob/master/images/study/train.jpg)

- Remember Set의 효능
 - Copy나 Compaction으로 Object가 다른 Train이나 Car 이동하게 되었을 때 Remember Set이 있으면, 쉽게 Reference를 업데이트 할 수 있음
 - 이게 없으면.. 최악의 경우 Root Set과 Object의 Reference를 모두 찾아 다녀야 할 수도 있다.
- Write Barrier : Object를 참조하게 되는 경우, 참조할 대상이 같은 Car에 있는 녀석이 아니라면, 해당 Car의 Remember Set에 기록
 - 이벤트 프로그램 또는 트리거
 - Car1 A -> Car2 C => Car2{A}

- Train Algorithm은 Suspend 시간을 쪼개서, 한 번의 긴 Suspend는 피할 수 있겠으나, 시간을 총합하면 다른 Algorithm에 비해 더 긴 시간일 수도 있다.
- 예제에서 A와 B를 Train 1-3이 아니라 Train 3로 옮긴 이유? : Copying Algorithm? Fragmentation?
- http://knoxxs.github.io/programming/java/2015/05/24/garbage-collection/

<br>
<br>

### Adaptive Algorithm

- 특정 Algorithm이 아님
- Heap의 현재 상황에 맞게 적절한 Algorithm을 적용하는 것 또는 Heap Sizing을 자동화하는 방법
- Hotspot JVM : Ergonomics, Adaptive Option
- IBM JVM : Tiling

<br>
<br>
<br>

## End

지금까지 살펴본 여러 Algorithm은 이전의 Algorithm을 보완하면서 진행되어 온 것을 알 수 있다.

하지만 어떤 Algorithm이라 하더라도 모두 Trade Off 관계가 있기 때문에 Application이나 사용자 패턴에 따를

적절한 Algorithm의 선택이 중요하다. 

이제 이러한 기본적인 Algorithm을 바탕으로 Hotspot JVM과 IBM JVM에서는 어떤 식으로 Garbage Collector들이 구현이 되어 있는지 살펴보자.
