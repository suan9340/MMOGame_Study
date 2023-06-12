
락을 걸면서 게임 룸에서 브로드캐스팅을 하고, 어떤 클라에서 패킷을 보냈을 때도 락을 걸고 게임 룸에 접근을 하는 방식.    
과연 이게 뭐가 문제가 될까?   

**식당으로 비유해보자**  
-> 식당 전체 : 서버       |        직원 : 하나의 쓰레드  
만약 이 상황에서 고객이 주문을 했다고 가정. 제육덮밥 하나를 주문.  

---   
**현재 우리가 구현해 놓은 것은**  
-> 하나의 직원이 주문을 받자마자 주방으로 달려가서 요리를 하고 요리를 주고 결제를 받는 방식 

이 방식의 문제점은 주방에서 요리할 수 있는 사람의 최대치가 있으면 밀리기 시작한다. (들어간 사람이 요리를 끝날때 까지 멍하게 기다리는 멍청한 방법)

---   

**그렇다면 올바른 방법은?**
-> 주방을 담당하는 사람, 주문을 받는사람을 나누자.  

주문서라는 개념을 만들어서 주문서를 주방장에게 넘겨주고, 주문을 받는 사람은 자기가 할 일을 하러 가도록. 주문서 라는 개념을 만들자!

---   

아까와는 다르게 계속 대기하는 것이 아닌, 주문서를 받고 자신이 할 일을 할 수 있다.

```csharp
### 이것이 바로 커맨드 패턴!!

using System;
using System.Collections.Generic;
using System.Text;

namespace Server.Game {
    public interface IJob {
        void Execute(); // 실행 함수
    }

    public class Job : IJob {
        Action _action; // 함수를 델리게이트 방식으로 저장

        public Job(Action action) { // 생성자
            _action = action;
        }

        public void Execute() { // 실행
            _action.Invoke();
        }
    }

    public class Job<T1> : IJob { // 제네릭으로 
        Action<T1> _action;
        T1 _t1;

        public Job(Action<T1> action, T1 t1) {
            _action = action;
            _t1 = t1;
        }

        public void Execute() {
            _action.Invoke(_t1);
        }
    }

    public class Job<T1, T2> : IJob { // 위와 동일
        Action<T1, T2> _action;
        T1 _t1;
        T2 _t2;

        public Job(Action<T1, T2> action, T1 t1, T2 t2) {
            _action = action;
            _t1 = t1;
            _t2 = t2;
        }

        public void Execute() {
            _action.Invoke(_t1, _t2);
        }
    }

    public class Job<T1, T2, T3> : IJob { // 위와 동일
        Action<T1, T2, T3> _action;
        T1 _t1;
        T2 _t2;
        T3 _t3;

        public Job(Action<T1, T2, T3> action, T1 t1, T2 t2, T3 t3) {
            _action = action;
            _t1 = t1;
            _t2 = t2;
            _t3 = t3;
        }

        public void Execute() {
            _action.Invoke(_t1, _t2, _t3);
        }
    }
}
```
