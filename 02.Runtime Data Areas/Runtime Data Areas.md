# Runtime Data Areas

## Runtime Data Areas의 구조
- JVM이 프로그램을 수행하기 위해 OS로부터 할당받는 메모리 영역
- WAS를 사용 할 때 가장많이 성능문제가 발생하는 영역
- JVM 벤더마다 구현이 다름
- PC Registers, Java Virtual Machine Stacks, Native Method Stacks, Method Area, Heap 영역으로 구분
- PC Registers, Stacks는 Thread 별로 생성
- 나머지는 모든 Thread에서 공유

## PC Registers

### CPU Register
- 프로그램의 실행은 CPU에서 명령어(Instruction)을 수행하는 과정으로 이루어짐
- Instruction을 수행하는 동안 필요한 정보를 레지스터라는 CPU 내의 기억장치를 사용
- "1 + 2"라는 연산을 하기 위해서는 1,2 그리고 결과값 데이타(Operand)와 add라는 Instruction을 CPU 기억장치에 잠시 저장해야 함

### Runtime Data Areas의 PC Registers
- Stack-Base로 작동
- JVM은 CPU에 직접 Instruction을 수행하지 않고 Stack에서 Operand를 뽑아내어 별도의 메모리에 저장 - PC Register
- Thread마다 하나씩 존재, Thread 시작 시 생성
- 현재 수행중인 JVM Instruction 주소를 가지고 있음
- Native Method 실행 시에는 undefined 상태

## Java Virtual Machine Stacks
- Thread의 수행 정보를 기록하는 Frame을 저장하는 메모리 영역
- Thread 별로 하나씩 존재, Thread 시작 시 생성
- Thread safe 영역(ex : 로컬변수)
- 현재 수행하고 있는 Method의 정보를 저장하는 것을 Current Frame이라고 함

### Stack Frame
- Thread가 수행하고 있는 Application을 Method 단위로 기록 하는 곳
- Class의 메타정보를 이용하여 저절한 크기로 생성됨
- Compile Time에 이미 결정됨

#### LocalVariable Section
- Method의 Parameter Variable과 Local Variable을 저장하는 영역
- 0 부터 시작하는 인덱스를 가진 Array
- Method Parameter는 선언된 순서대로 인덱스 생성
- Local Variable은 Compiler가 알아서 인덱스 생성(사용되지 않는 변수는 인덱스를 할당하지 않을 수도 있음)
- 인덱스 생성은 저장공간을 마련한다는 것과 같은 의미

##### 메모리 할당
- Primitive Type(byte,char,int 등)인 경우는 고정된 크기 할당
- Object나 Array같은 객체는 가변 크기 --> reference 타입으로 저장
- Heap 메모리에 존재하는 객체의 위치를 말해주는 reference를 저장
- Integer VS int
- Reference를 통해 객체를 찾아 다니는 작업은 CPU 연산을 필요로 함
- Method Area의 java.lang.Integer의 클래스 정보를 읽어 Instance를 생성
- 메모리 & CPU 사용

```java
public int testMethod(int a,char b, long c, flat d, Object e, double f, String g,
bye h, short i, boolean j){
    return 0;
}
```
![stack](./variable_stack.png)

- char, byte, short, boolean 선언이 LocalVariable Section에서는 int형으로 할당 됨
- Primiive Type이라 할지라도 JVM 지원 여부와 Java Virtual Machine Stacks의 저장형태에 따라 달라짐
- byte, short, char는 LocalVariable Section과 Operand Stack에서는 int 형으로 저장되고 Heap 등의 다른 곳에서는 원래 형으로 저장
- boolean의 경우 모든 곳에서 int형을 유지함
- 0번 인덱스에 저장된 Reference를 통해 Heap에 있는 Class의 Instance 데이타를 찾아감
- Class Method(static method)는 0번 reference 정보가 존재하지 않음(Method Area에서 설명)

#### Operand Stack
- JVM 작업 공간
- 프로그램을 수행하며 연산을 위해 사용하는 데이터 및 그 결과를 Operand Stack에 저장하여 사용

```java
public void operandStack(){
    int a,b,c;
    a = 5;
    b = 6;
    c = a + b;
}
```

```java
public void operandStack(){
    Code:
    0 : iconst_5
    1 : istore_1
    2 : bipush 6
    3 : istore_2
    4 : iload_1
    5 : iload_2
    6 : iadd
    7 : isore_3
    8 : return
}
```

![stack](./operand_stack.png)

- Method 시작시 Stack Frame 생성
- Compile Time에 Operand Stakc 및 Local Variable Section 크기는 정의 됨
- Array로 Local Variable Section 생성(0 ~ 4)
- Operand Stack도 Array 구성
- 인덱스를 사용하지 않고, Push와 Pop을 통해 필요시 공간 할당
1. iconst_5 : 상수 5를 push
2. istore_1 : Local Variable Section 1번 인덱스에 값 저장
3. bipush 6 : 상수 6을 push
4. istore_2 : Local Variable Section 2번 인덱스에 값 저장
5. iload_1  : Local Variable Section 1번 인덱스에서 값 로드
6. iload_2  : Local Variable Section 2번 인덱스에서 값 로드
7. iadd     : int형의 값을 더함. Operand Stack 값들이 모두 Pop되어 연산에 사용. 결과를 Push와
8. isore_3  : Local Variable Section 3번 인덱스에 값 저장 
9. return   : Method의 수행을 마치고 Stack Fraem을 나감

#### Frame Data
- Constant Pool Resolution 정보와 Exception 관련 정보 저장
- Class의 모든 Symbolic Reference는 모두 Method Area의 Constant Pool 이라는 곳에 저장되어 있음
- Constant Pool Resolution은 Method Area를 가르키는 Pointer 정보
- 상수 참조, Method 수행, 특정 변수 접근 시 Constant Pool 참조

##### Resolution
- Symbolic Reference에서 Direct Reference로 변경되는 것을 Resolution이라고 함

```java
public void operandStack(){
    int a,b,c;
    a = 5;
    b = 6;
    try{
        c = a + b;
    }catch(NullPointerException e){
        c = 0;
    }
    
}
```

```java
public void operandStack(){
    Code:
    0 : iconst_5
    1 : istore_1
    2 : bipush 6
    3 : istore_2
    4 : iload_1
    5 : iload_2
    6 : iadd
    7 : isore_3
    8 : goto 12
    9 : astore 4
    10 : iconst_0
    11 : istore_3
    12 : return
    Exception table:
    from    to   target   type
     6       7     9      Class java/lang/NullPointerException
}
```

- from : try 시작 블록 넘버
- to   : try 종료 블록 넘버
- targe : Exception 발생 시 점프해야할 넘버
- type  : 정의한 Exception
- type에 정의한 Class 정보와 비교하여 일치하지 않는 경우에는 상위 Stack Frame에 Exception을 던져 처리 반복

## Native Method Stacks
- Native Method 호출 시 Native Method Stacks 이라는 메모리 공간을 사용
- Java Virtual Machine Stacks에서 Native Method 호출 시 Native Method Stack에 새로운 Frame을 생성하여 수행
- HotSpot JVM이나 IBM JVM은 두 Stack을 구분하지 않고 Native Stack으로 통합되어 있음
- Stack Size 조정은 -Xss와 -Xoss 두가지로 조절 가능
- -Xss는 Native Stack -Xoss는 Java Stack을 조정하는 옵션
- HotSpot JVM에서는 -Xss만 사용

      Java에서 Native Memory를 설정하는 방법은 따로 없고 Process에 가용 메모리에서 Heap, Method Area등
      Java Runtime Memory를 뺀 나머지를 사용 할 수 있음. Native Memory가 부족 할 경우에도 OutOfMemory가 발생함.
      이러한 경우 Java Runtime Memory를 줄이는 방법도 있지만, Native Memory Leak을 의심해 볼 필요가 있음
