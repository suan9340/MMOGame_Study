## 유니티_1번째 강의  
#### 유니티를 활용하여 실제로 우리가구동해 볼 수 있는 클라이언트를 새로 만들어서 구현해야한다!
-> DummyClient에서 사용하던 코드들을 재활용 하여 사용하자.    

---   


#### 일단 쓸 코드들을 다 가져와보자!  

> **ServerCore 쪽에서**   
Connector, JobQueue, Listener, PriorityQueue, RecvBuffer, SendBuffer, Session
        

> **DummyClient 쪽에서**   
ServerSession, Packet폴더   

에 있는 파일들을 가져와서 유니티에 넣자!   
   

#### 삭제해야 할 파일들    
- JobQueue (유니티에 있는 코루틴으로 대체 가능)
- Listener (서버에서 대기할때 사용 클라에서는 필요 없다)
- Priority Queue (우선순위큐, 딱히 필요 없다)

---   

~~이제 유니티에서 나는 에러를 하나씩 고쳐야한~~


**GenPackets**


```
public ArraySegment<byte> Write()
	{
		ArraySegment<byte> segment = SendBufferHelper.Open(4096);
		ushort count = 0;

		count += sizeof(ushort);
		Array.Copy(BitConverter.GetBytes((ushort)PacketID.S_BroadcastLeaveGame), 0, segment.Array, segment.Offset + count, sizeof(ushort));
		count += sizeof(ushort);
		Array.Copy(BitConverter.GetBytes(this.playerId), 0, segment.Array, segment.Offset + count, sizeof(int));
		count += sizeof(int);

		Array.Copy(BitConverter.GetBytes(count), 0, segment.Array, segment.Offset, sizeof(ushort));

		return SendBufferHelper.Close(count);
	}
```    
      
메세지를 직렬화 하여 네트워크를 통해 전송할 수 있도록 BitConverter를 사용해서 바이트 배열 세그먼트로 반환을 하고 있다.   
Array.Copy로 넣기.

