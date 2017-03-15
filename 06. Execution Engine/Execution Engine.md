# Execution Engine
## Execution Engine 이란?
Class Loader를 통해 Load된 Class의 Byte Code를 실행하는 Runtime Module 이다.  
자바 코드 실행 과정에 관한 앞선 내용들을 상기 해 보면, 아래와 같다.  
// 이미지 추가
1. 소스 코드(.java)를 javac로 컴파일하여 Bytecode(.class) 생성
2. Class Loader가 Class Bytecode를 JVM으로 Load 및 Linking
3. Execution Engine이 Bytecode를 instruction code로 전환하며, 이 때 interpreter 또는 JIT Compiler가 사용 됨

// 이미지 추가

예제 소스
```java
	public static void main(String[] args) {
		int num = 0;
		String str = "String";
		for( ; ; ) {
			num = num + 1;
			num *= 2;
		}
	}
```

```java
  public static void main(java.lang.String[]);
    Code:
       0: iconst_0
       1: istore_1
       2: ldc           #2                  // String String
       4: astore_2
       5: iload_1
       6: iconst_1
       7: iadd
       8: istore_1
       9: iload_1
      10: iconst_2
      11: imul
      12: istore_1
      13: goto          5

```

## Execution Engine의 방식
Java는 Bytecode가 JVM에서 바로 수행되지 않는 구조로 설계가 되었기 때문에, JVM에서 바로 수행하도록 하는 과정이 필요하며 이를 수행 하는 것이 Execution Engine이다.  
즉, Execution Engine이 Bytecode(.class)를 내부적으로 Instruction으로 변경하여 수행한다.  
Execution Engine이 Bytecode를 해석하는 방법은 2가지가 있다.  

1. interpreter 방식  
Bytecode를 한 줄씩 해석하는 방식이다.  
Bytecode를 한 줄씩 읽으므로 해석하는 시간이 짧다는 것이 장점이다.  
하지만 Interpreter의 결과물을 수행하는 실행 시간은 많이 걸린다는 단점이 있다.(초기 JVM의 약점)  
// 설명 추가

2. JIT(Just-In-Time) Compiler 방식  
적절한 시점에 Compile을 수행하는 방식이다.  
Load된 Class에 대해 Interpreter 방식으로 동작을 하다가 반복 수행을 감지하면 적절하게 JIT Compiler가 동작하여 실행속도를 향상 시킨다.  
Bytecode로부터 Native Code를 생성한 뒤에 실행하는 것이 핵심이다.  
실행속도가 느린 Interpreter 방식의 단점을 이렇게 극복한 것 이다.  
대신에 Interpreter보다 Native Code로 변환하는 시간은 더 길어진다.  
JIT Compiler의 전체 실행시간 = Bytecode를 Native Code로 변환 하는 시간 + Native Code를 실행하는 시간  
Native Code는 기본적으로 Memory Cache가 이루어지기 때문에 반복 호출 시 성능이 극대화 된다.  
그러나 반복 수행이 되지 않으면, 오히러 Interpreter보다 성능이 떨어질 수 있다.  

그래서 JVM은 기본적으로 Interpreter를 사용하다가 일정 기준을 넘어서게 되면 Jit Compliler를 사용한다.  
이 방법을 'Lazy Fashion'이라고 한다.  


## Execution Engine의 성능 이슈
Server Application의 경우 전체 수행시간에서 Execution Engine이 사용하는 시간은 절반이 넘고, Client Application에서는 30%이상의 시간을 소요한다.  
Application 수행 시간의 결정적인 요소라고 부를 수 있을 정도인데, Execution Engine의 성능 이슈가 발생 하는 부분을 봐야한다.  
1. Loop문에서 Bytecode를 반복 수행  
// 소스 추가
// 이미지 추가
2. 다차원 배열을 지원하지 않음
// 소스 추가
// 이미지 추가

하지만 Loop 또는 다차원 배열의 사용을 꺼리 필요는 없다. 그 이유는 JVM을 구현한 벤더들이 이러한 약점을 보완하기 위하여 무수히 많은 최적화 기법을 동원하고 있고, Execution Engine 자체 성능이 지속적으로 개선되고 있기 때문이다.

## Hotspot Compiler
// 이미지 추가

// 이미지 추가