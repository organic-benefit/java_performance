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

#### Operand Stack
#### Frame Data

## Native Method Stacks
