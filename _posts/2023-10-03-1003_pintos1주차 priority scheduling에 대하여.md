### priority scheduling에 대하여



##### thread의 생애주기



```
enum thread_status
{
	THREAD_RUNNING, /* Running thread. 실행중(딱 하나)*/
	THREAD_READY,	/* Not running but ready to run. (돌아가기만 하면 되는상태)*/
	THREAD_BLOCKED, /* Waiting for an event to trigger. 의도적으로 이벤트에 의해 멈춤. 꺠워주지않으면 멈춤 레디큐에 못들어가게 함. */
	THREAD_DYING	/* About to be destroyed. 죽였다고 마킹하는것. 실제로 죽이는건 모아서 따로*/
};
```

코드로 알 수 있었다. 총 4가지의 단계가 있고,

실행중인 쓰레드 = running

실행을 위해 대기중인 쓰레드들 = ready

의도적으로 멈춰있는 쓰레드들 = blocked

죽은 쓰레드 = dying



##### 현재 핀토스의 상태

```
static struct thread *
next_thread_to_run (void) 
{
  if (list_empty (&ready_list))
    return idle_thread;
  else
    return list_entry (list_pop_front (&ready_list), struct thread, elem);
}
```



ready_list에서 가장 앞에 있는것을 pop해서 다음 실행 thread를 결정하고 있다. 이 코드로 인하여 round-robin 상태를 채택하고 있다는것을 알수 있다. 이 방식은 쓰레드의 우선순위를 고려하지 않고 ready_list에 들어온 순서대로 실행하는 FIFO형식이다.

우리는 우선순위 순으로 thread가 실행되도록 변경해야 한다!



때문에 우선순위 순서대로 ready_list에 넣어줘야 하기 때문에 아래 코드로 변경해줘야 한다.

ready_list가 변경될때는 thread_unblock될때와 thread_yield 할때이다.

```
list_insert_ordered(&ready_list, &t->elem, cmp_priority, NULL);
```



```
bool cmp_priority(struct list_elem *a_, struct list_elem *b_, void *aux UNUSED)
{
	const struct thread *a = list_entry(a_, struct thread, elem);
	const struct thread *b = list_entry(b_, struct thread, elem);
	return (a->priority > b->priority);
}
```



##### 그리고 현재 쓰레드의 priority 가 변경되는경우 바뀐 priority와 ready_list의 가장 높은 priority보다 낮다면 CPU를 양보해야 한다.

현재 쓰레드의 priority 가 변경되는 경우: thread_create(), thread_set_priority()

```
void test_max_priority(void)
{
	struct thread *curr = thread_current();
	// ready list의 가장
	list_sort(&ready_list, cmp_priority, NULL);
	if (!list_empty(&ready_list) && curr->priority < list_entry(list_begin(&ready_list), struct thread, elem)->priority)
	{
		thread_yield();
	}
}

```



donation을 제외한 테스트 통과!

![다운로드](https://p.ipic.vip/r2ki1r.jpg)