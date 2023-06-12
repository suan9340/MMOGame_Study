
GameRoom에 JobSerializer상속받음  
-> lock이 필요없어진다. (lock을 다 지우기) (FindPlayer 에는 todo로 표시해두기)

각각의 함수들이 사용을 하던 부분으로 가서 다 push하는 방식으로 바꾸기   


---  

Push 기능을 만들었으므로 코드 수정을 하자 !

```csharp
using System;
using System.Collections.Generic;
using System.Text;

namespace Server.Game {
    public class JobSerializer {
        Queue<IJob> _jobQueue = new Queue<IJob>();
        object _lock = new object();
        bool _flush = false;


        public void Push(Action action) { Push(new Job(action)); }
        public void Push<T1>(Action<T1> action, T1 t1) { Push(new Job<T1>(action, t1)); } // 편의성 증가
        public void Push<T1, T2>(Action<T1, T2> action, T1 t1, T2 t2) { Push(new Job<T1, T2>(action, t1, t2)); } // 위와 동일
        public void Push<T1, T2, T3>(Action<T1, T2, T3> action, T1 t1, T2 t2, T3 t3) { Push(new Job<T1, T2, T3>(action, t1, t2, t3)); } // 위와 동일

        public void Push(IJob job) { // 추가
            lock (_lock) {
                _jobQueue.Enqueue(job); // queue에 넣어주기
            }
        }

        public void Flush() { // 처리 
            while (true) {
                IJob job = Pop();
                if (job == null)
                    return;

                job.Execute();
            }
        }

        IJob Pop() { // 반환
            lock (_lock) {
                if (_jobQueue.Count == 0) { // queue에 남은 작업이 없으면 
                    _flush = false;
                    return null;
                }
                return _jobQueue.Dequeue(); // queue에서 빼주기
            }
        }
    }
}
```
