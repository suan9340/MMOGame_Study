
현재 우리의 로직이  
만약 클라쪽에서 요청이 와서 패킷 핸들에서 게임 룸에 있는 함수를 실행하게 되면서 JobSerialize에서 실행을 하게 되는데   
진짜 극단적으로 게임 룸에 한명도 없어서 아무런 패킷도 오지 않는다면 update에 있는 몬스터들은 실행이 안된다.   

-> 근데 몬스터들은 어찌됬던 한번은 꼭 실행이 되야한다.
현재 구현 방식은 무한루프 속에서 몬스터들을 업데이트로 관리하는데, 이 방식은 좋지 않은 방식. 

**유니티에서 처럼 코루틴 같은 곳에서 관리를 해야 좀 더 좋다.**

---   

**바꿔야 할 것**  

예약하는 시스템이 필요하고, 업데이트를 어떻게 해야 효율적으로 바꿀 수 있을까? 

---  

fps, rpg 등 장르에 따라 Update주기가 다르다. 업데이트를 어떤 주기로 할지도 정해야 함

Push랑 Flush를 따로하며 장점은 관리가 좋지만 단점은 운이 안 좋으면 50ms뒤에 실행된


```csharp
using System;
using System.Collections.Generic;
using System.Text;
using ServerCore;

namespace Server.Game {
    struct JobTimerElem : IComparable<JobTimerElem> {
        public int execTick; // 실행 시간
        public IJob job; // Action -> IJob으로 수정하며 코드 수정

        public int CompareTo(JobTimerElem other) {
            return other.execTick - execTick;
        }
    }

    public class JobTimer {
        PriorityQueue<JobTimerElem> _pq = new PriorityQueue<JobTimerElem>();
        object _lock = new object();

        // 전역으로 사용하던 Instance를 삭제
        // 관리를 JobSerializer에서 작업

        public void Push(IJob job, int tickAfter = 0) {
            JobTimerElem jobElement;
            jobElement.execTick = System.Environment.TickCount + tickAfter;
            jobElement.job = job;

            lock (_lock) {
                _pq.Push(jobElement);
            }
        }

        public void Flush() { // 무한 루프를 돌며 틱 확인하여 하는것도 낫베드
            while (true) {
                int now = System.Environment.TickCount;

                JobTimerElem jobElement;

                lock (_lock) {
                    if (_pq.Count == 0)
                        break;

                    jobElement = _pq.Peek();
                    if (jobElement.execTick > now)
                        break;

                    _pq.Pop();
                }

                jobElement.job.Execute(); // 실행
            }
        }
    }
}
```     


**jobSerializer**
```csharp
        JobTimer _timer = new JobTimer();

        public void PushAfter(int tickAfter, Action action) { PushAfter(tickAfter, new Job(action)); }
        public void PushAfter<T1>(int tickAfter, Action<T1> action, T1 t1) { PushAfter(tickAfter, new Job<T1>(action, t1)); }
        public void PushAfter<T1, T2>(int tickAfter, Action<T1, T2> action, T1 t1, T2 t2) { PushAfter(tickAfter, new Job<T1, T2>(action, t1, t2)); }
        public void PushAfter<T1, T2, T3>(int tickAfter, Action<T1, T2, T3> action, T1 t1, T2 t2, T3 t3) { PushAfter(tickAfter, new Job<T1, T2, T3>(action, t1, t2, t3)); }

        public void PushAfter(int tickAfter, IJob job) { 
            _timer.Push(job, tickAfter);
        }

        public void Flush() { // 처리 
            while (true) {
                IJob job = Pop();
                if (job == null)
                    return;

                job.Execute();
            }
        }

```

**Program**
```csharp
using System;
using System.Collections.Generic;
using System.Net;
using System.Net.Sockets;
using System.Reflection;
using System.Text;
using System.Threading;
using System.Threading.Tasks;
using Google.Protobuf;
using Google.Protobuf.Protocol;
using Google.Protobuf.WellKnownTypes;
using Server.Data;
using Server.Game;
using ServerCore;

namespace Server
{
	class Program
	{
		static Listener _listener = new Listener();
		static List<System.Timers.Timer> _timers = new List<System.Timers.Timer>(); // 주기적으로 도는 모든 타이머 리스트

		static void TickRoom(GameRoom room, int tick = 100)
		{
			var timer = new System.Timers.Timer(); // 타이머 설정 
			timer.Interval = tick; // 몇 tick 마다 실행할지
			timer.Elapsed += ((s, e) => { room.Update(); }); // 무슨 이벤트를 실행할지
			timer.AutoReset = true; // 매번 다시 리셋
			timer.Enabled = true; // 실행

			_timers.Add(timer);
		}

		static void Main(string[] args)
		{
			ConfigManager.LoadConfig();
			DataManager.LoadData();

			GameRoom room = RoomManager.Instance.Add(1);
			TickRoom(room, 50); // 룸 생성하면 50ms 마다 실행

			// DNS (Domain Name System)
			string host = Dns.GetHostName();
			IPHostEntry ipHost = Dns.GetHostEntry(host);
			IPAddress ipAddr = ipHost.AddressList[0];
			IPEndPoint endPoint = new IPEndPoint(ipAddr, 7777);

			_listener.Init(endPoint, () => { return SessionManager.Instance.Generate(); });
			Console.WriteLine("Listening...");

			//FlushRoom();
			//JobTimer.Instance.Push(FlushRoom);

			// TODO
			while (true)
			{
				//JobTimer.Instance.Flush();
				Thread.Sleep(100); // 꺼지지 않게 유지만
			}
		}
	}
}
```


