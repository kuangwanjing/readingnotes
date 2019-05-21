## Condition Variable

### Background

The thread wishes to check whether a condition is true before continuing its execution.  Ordering problem like Example: 

```C
void *child (void *arg) {
  printf("child\n");
  // here the child finishes its execution, what to do is to indicate it is done. <- this is a condition
  return NULL;
} 
int main(int argc, char *argv[]) {
  printf("begin\n");
  pthread_t c;
  Pthread_create(&c, NULL, child, NULL); // create child
  // XXX how to wait for child? <- this is the check of the condition
 	printf("parent: end\n");
  return 0;
}
```

### Solution

Condition Variable (CV) : an explicit queue that threads can put themselves on when some state of execution is not as desired and wake one of the threads when it meets some desired state.  (channels)

Operations:

1. Wait - put the thread into the queue
2. Signal — wake up one of the threads in the queue

```C
int done = 0;
pthread_mutex_t m = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t c = PTHREAD_COND_INITIALIZER;

void thr_exit() {
	Pthread_mutex_lock(&m);
	done = 1;
	Pthread_cond_signal(&c);
	Pthread_mutex_unlock(&m);
}

void *child(void *arg) {
	printf("child\n");
	thr_exit();
	return NULL;
}

void thr_join() {
 Pthread_mutex_lock(&m); // why a lock is needed here? because we need to somehow manipulate the shared variable concurrently. 
 while (done == 0) // it is a good idea to always use "while" instead of "if"
     Pthread_cond_wait(&c, &m);
 Pthread_mutex_unlock(&m);
}

int main(int argc, char *argv[]) {
 printf("parent: begin\n");
 pthread_t p;
 Pthread_create(&p, NULL, child, NULL);
 thr_join();
 printf("parent: end\n");
 return 0;
}
```

**Note:  CV records the value the threads are interested in knowing. The sleeping, waking, and locking all are built around it**

### Application

The Producer/Consumer(Bounded Buffer) Problem: 

1. Links Pool: Producers put HTTP request into the pool and the consumer take the request out of the pool and process it. 
2. Unix Pipeline: "grep foo file.txt | wc -l ", threads grep and wc are running concurrently while grep works as the producer and wc works as the consumer. 

Go through the implementations of Producer/Consumer model with CV:

```C
// version 1: race condition exists
int buffer;
int count = 0;

void put(int value) {
  assert(count == 0);
  count = 1;
  buffer = value;
}

int get() {
  assert(count == 1);
  count = 0;
  return buffer;
}

void producer(void *arg) {
  int i;
  int loop = int(arg);
  for (i = 0; i < loop; i++) {
    put(i);
  }
}

void consumer(void *arg) {
  while (1) {
    int tmp = get();
    printf("%d\n", tmp);
  }
}
```

```C
// version 2: broken
int loops; // initialized somewhere
cond_t cond;
mutex_t mutex;

void producer(void *arg) {
  int i;
  for (i = 0; i < loops; i++) {
    Pthread_mutex_lock(&mutex);
    if (count == 1) // 
      Pthread_cond_wait(&cond, &mutex); // if a cv wakes up the thread, thread continues execution after this point, 
    put(i); // which puts data directly without checking the condition
    Pthread_cond_signal(&cond);
    Pthread_mutex_unlock(&mutex);
  }
}

void consumer(void *arg) {
  while (1) {
    Pthread_mutex_lock(&mutex);
    if (count == 0) // the same reason why race condition happens with this implementation.
      Pthread_cond_wait(&cond, &mutex);
    int tmp = get();
    Pthread_cond_signal(&cond);
    Pthread_mutex_unlock(&mutex);
    printf("%d\n", tmp);
  }
}
```

**Note:  scenario of 2 consumers and 1 producer may break the correctness, similarly 2 producers and 1 consumers does the same thing.**

```c
// version 3: still broken
int loops; // initialized somewhere
cond_t cond;
mutex_t mutex;

void producer(void *arg) {
  int i;
  for (i = 0; i < loops; i++) {
    Pthread_mutex_lock(&mutex);
    while (count == 1) // prevent the condition from changing before waking up
      Pthread_cond_wait(&cond, &mutex);  
    put(i); 
    Pthread_cond_signal(&cond); // here the producer may wake up a producer
    Pthread_mutex_unlock(&mutex);
  }
}

void consumer(void *arg) {
  while (1) {
    Pthread_mutex_lock(&mutex);
    while (count == 0) 
      Pthread_cond_wait(&cond, &mutex);
    int tmp = get();
    Pthread_cond_signal(&cond); // also the consumer may wake up a consumer
    Pthread_mutex_unlock(&mutex);
    printf("%d\n", tmp);
  }
}
```

**Note:  the threads are waiting for the wrong condition. A producer may wake up another producer and a consumer may wake up another consumer.**

```C
// version 4: two conditions -- one is the buffer is not empty and another one is the buffer is not full.
int loops; // initialized somewhere
cond_t empty, fill;
mutex_t mutex;

void producer(void *arg) {
  int i;
  for (i = 0; i < loops; i++) {
    Pthread_mutex_lock(&mutex);
    while (count == 1) // prevent the condition from changing before waking up
      Pthread_cond_wait(&empty, &mutex); // waiting for the empty condition so that the producer can fill up an item.
    put(i); 
    Pthread_cond_signal(&fill); // here the producer may wake up a producer
    Pthread_mutex_unlock(&mutex);
  }
}

void consumer(void *arg) {
  while (1) {
    Pthread_mutex_lock(&mutex);
    while (count == 0) 
      Pthread_cond_wait(&fill, &mutex); // waiting for the filled condition so that a consumer can get an item.
    int tmp = get();
    Pthread_cond_signal(&empty); // also the consumer may wake up a consumer
    Pthread_mutex_unlock(&mutex);
    printf("%d\n", tmp);
  }
}
```

**To Wrap Up the principles of using CV: protecting the CV from data race and determine what condition a thread is interested in.**