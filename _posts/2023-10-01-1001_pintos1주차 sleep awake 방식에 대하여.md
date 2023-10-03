### 핀토스1주차: scheduling

현재 쓰레드 실행 구조 = busy-waiting 방식

대기중인 쓰레드가 내가 running이 가능한가?를 계속 확인하는것.

running , ready상태를 반복함.

cpu 를 많이 잡아먹고 있기 때문에 좋은 방법은 아니다.. 그래서 대기중인 쓰레드를 `sleep_list`로 관리하고 꺠울 시간을 부여해서, 잠든 스레드가 깰 시간까지는 ready_list에 추가하지 않고, 깰시간에 도달한 경우에만 `ready_list`에 추가하는 방식으로 변경하는 것이 이번 핀토스 project1의 목표(중에 하나)이다.

![스크린샷 2023-10-02 23.30.15](https://p.ipic.vip/zsl47n.jpg)

##### 🙊 tick이란?

컴퓨터 os내에서의 일정한 시간 단위를 말한다.

os는 틱을 통해서 스레드 전환 등 의 일을 처리할 수 있게 된다.

해야할일

1. sleep_list 선언

   ```
   static struct list sleep_list;
   ```

   그리고 `thread_init`함수에서 sleep_list 초기화해주기

   ```
   list_init(&sleep_list);
   ```

   ​

2. thread 구조체에 필드추가(공식문서에 struct thread에 추가해야할 내용이 써있었다.)

   ```
   int64_t wakeup_ticks; // 일어날 시각 추가
   ```

   ​

3. thread 재우기 로직 변경

   잠든스레드는 ready_list가 아닌 sleep_list에 삽입하는 thread_sleep()함수 호출

   ```
   void timer_sleep(int64_t ticks){
     int64_t wake_tick = timer_ticks() + ticks; // 현재시간 + 잠들어있어야 하는 시간 = 일어나야 하는 시간
     thread_sleep(wake_tick);
   }
   ```

   ​

4. 잠든 스레드를 관리하는 `sleep_list`에 삽입

   ```
   void thread_sleep(int64_t ticks){
     	struct thread *curr = thread_current();
   	enum intr_level old_level;
   	ASSERT(!intr_context());
   	old_level = intr_disable();
   	if (curr != idle_thread)
   	{

   		curr->wakeup_tick = wake_tick;
   		list_insert_ordered(&sleep_list, &curr->elem, cmp_thread_ticks, NULL);

   	}

   	do_schedule(THREAD_BLOCKED);
   	intr_set_level(old_level);
   }
   ```

   ​

5. `cmp_thread_ticks`선언하기

   여기서 좀 헷갈렸는데 tick이 더 작은 순서대로 sleep_list에 넣어야한다.

   왜? 틱이 작은 순서대로 넣어야할까? 틱이 작을 수록 먼저 깨워야 하기 때문이다.

   예를들어서 현재 틱이 100이고, 쓰레드 A의 wake_tick이 200이라면, 100틱 뒤에 꺠워야 한다.

   쓰레드 B wake_tick이 300이라면 200틱 뒤에 꺠워야 한다.

   그렇기 때문에 틱이 작은 순서대로 꺠워줘야 한다.

   ```
   bool cmp_thread_ticks(struct list_elem *a_, struct list_elem *b_, void *aux UNUSED)
   {
   	const struct thread *a = list_entry(a_, struct thread, elem);
   	const struct thread *b = list_entry(b_, struct thread, elem);
   	return (a->wakeup_tick < b->wakeup_tick);
   }
   ```

   ​

6. 현재 시간보다 작거나 같은 wake_ticks을 가진 스레드는 모두 꺠운다.

   꺠운다 = sleep_list 에서 삭제 후, ready_list에 넣는다.

   ```
   void thread_wake(int64_t elapsed) // 입력받는것 = 현재시간;
   {
   	// 스레드의 일어난 시간이 지금 시간보다 작거나 같으면,
   	while (!list_empty(&sleep_list) && list_entry(list_front(&sleep_list), struct thread, elem)->wakeup_tick <= elapsed)
   	{
   		struct list_elem *front_elem = list_pop_front(&sleep_list);
   		thread_unblock(list_entry(front_elem, struct thread, elem));
   	}
   }
   ```

   ​

![스크린샷](https://p.ipic.vip/s23kjq.png)
완료 후, 0이었던 idel thread가 증가했다.
