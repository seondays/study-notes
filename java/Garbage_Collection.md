# 자바의 가비지 컬렉션 (GC) [🔗](https://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/index.html)
## 정의
자바 가비지 컬렉션이란, 동적으로 할당된 객체들이 존재하는 `heap`메모리 영역에서 더 이상 사용하지 않는 객체를 주기적으로 정리하여 메모리 영역을 확보하는 작업이다.  
가비지 컬렉션 덕분에 프로그래머는 수동으로 메모리 공간에 신경쓰지 않아도 된다.
## 가비지 컬렉션 과정
### step 1 : marking
첫 번째로 마킹 동작이 진행되는데 이는 가비지 컬렉터가 사용 중인 객체와 사용하지 않는 객체를 식별하는 과정이다.  
&rarr; 여기서 사용되지 않는 객체라는 것은 참조되지 않는 객체를 의미한다.  
이 마킹 단계에서 모든 객체를 스캔하여 사용 여부를 확인하는데, 전체 객체의 스캔이 이루어지므로 시간이 오래 소요될 수 있다.
### step 2 : Normal Deletion
다음 단계는 일반 삭제로, 사용하지 않는다고 판단한 객체를 삭제한다. 이 때 메모리를 할당하는 역할을 하는 부분은, 삭제로 만들어진 여유 공간에 대한 참조를 보유한다.
### Step 2a: Deletion with Compacting
사용하지 않는 객체들을 삭제하고 나면, 메모리는 할당되어 있는 부분과 중간중간 비어있는 부분이 생기게 되는데, 성능 개선을 위해서 남아있는 객체들을 앞으로 쭉 당겨오는
압축 작업이 가능하다. 이런 작업을 통해 메모리 단편화를 방지하고 효율을 올릴 수 있다.

## JVM Generations
프로그램을 사용하면서 많은 양의 객체가 만들어지는데, 이를 모두 스캔하고 객체를 지우는 작업을 반복하게 되면 성능에 문제가 생길 수 있지만, 일반적으로 대부분의 객체는 수명이 길지 않다.
그렇기 때문에 이 특성을 이용하여 JVM의 성능을 향샹시키는 구조를 구성할 수 있다.

힙 메모리는 내부적으로 Young Generation, Old Generation 으로 나뉘는 구조를 가지고 있다. (Permanent Generation은 java8부터 mataspace로 배치 이동)
### Young Generation
- 새롭게 생성된 객체가 할당되는 영역이다. 객체들은 대부분 수명이 짧기 때문에, 이 영역은 빠르게 채워지고 또 빠르게 정리된다.
- 이 영역이 가득 차면 minor gc가 발생된다. 여기에서 살아남은 객체들이 Old Generation으로 이동하게 된다.
- Young Generation 내부는 또 다시 Eden과 s0, s1 생존자 구간으로 나누어진다. 이 내부 구간에서의 상세 프로세스는 다음과 같다.
#### 새롭게 만들어지는 모든 객체는 eden 공간에 할당된다.
![image](https://github.com/user-attachments/assets/8a9682e5-0d08-400a-b8de-df13b36578ba)

#### eden 공간이 가득 차면, minor GC가 트리거된다.
![image](https://github.com/user-attachments/assets/02b1e2ae-e46e-47ae-82be-092cfc1ceb24)

#### 참조된 객체는 s0 생존자 구간으로 이동되고, 참조되지 않은 객체는 정리된다.
![image](https://github.com/user-attachments/assets/715a82cf-a3b1-4e89-b132-ed48afcca667)

#### 다음으로 또 다시 eden이 가득 차게 될 경우 GC가 발생해 참조된 오브젝트는 생존자 공간으로 이동한다. 다만 이번에는 s0이 아닌 s1로 이동하는데, eden의 객체 뿐 아니라 기존 s0에 있던 객체들을 대상으로도 gc가 진행되어, s0의 객체들도 s1로 이동된다. 이 때 age가 1씩 증가한다. 그 후 eden과 s0 공간이 정리된다.
![image](https://github.com/user-attachments/assets/92b74d74-f314-40e1-a67b-309f65921358)

#### 또 다음 minor GC에서도 동일한 일이 일어나는데, 다만 달라지는 것은 이번에는 비어 있는 s0 공간으로 이동한다는 것이다. 즉 생존자 공간은 번갈아 가면서 비어 있는 곳을 사용하게 된다.
![image](https://github.com/user-attachments/assets/754c7732-550f-421f-8aa3-e872628c6bca)

#### 이렇게 minor GC를 반복하다가 객체가 특정 age에 도달하면 해당 객체는 Young Generation에서 Old Generation으로 승격하게 된다.
![image](https://github.com/user-attachments/assets/efbeb0af-ae9a-41bc-8bb2-0554dd8a56b6)

#### minor GC가 계속해서 진행되면 자연스럽게 승격하는 객체도 많아지게 된다. 이를 통해 Old Generation 영역이 꽉 차게 되면 major GC가 트리거된다. Major GC는 Old Generation을 검사하여 참조되지 않는 객체를 제거한다.
![image](https://github.com/user-attachments/assets/5ac3d51d-b780-4ebe-93c7-24f7b3a2cca8)

## Minor GC & Major GC
- Minor GC는 Eden 영역이 가득 찰 경우 수행되는 GC, Major GC는 Old Generation 영역이 가득 찼을 때 수행되는 GC
- 두 GC 모두 어플리케이션 스레드들이 멈추는 Stop-the-World 상황이 발생한다.
- 상대적으로 용량이 더 큰 Old Generation을 정리하는 Major GC가 발생했을 때 성능적으로 문제가 될 가능성이 높다.