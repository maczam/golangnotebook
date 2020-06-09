# 锁

golang中sync包实现了两种锁Mutex （互斥锁）和RWMutex（读写锁），其中RWMutex是基于Mutex实现的，只读锁的实现使用类似引用计数器的功能．

``` go 
type Mutex
    func (m *Mutex) Lock()
    func (m *Mutex) Unlock()
```
``` go
type RWMutex
    func (rw *RWMutex) Lock()
    func (rw *RWMutex) RLock()
    func (rw *RWMutex) RLocker() Locker
    func (rw *RWMutex) RUnlock()
    func (rw *RWMutex) Unlock()
```



## Mutex

   Mutex为互斥锁，Lock()加锁，Unlock()解锁，使用Lock()加锁后，便不能再次对其进行加锁，直到利用Unlock()解锁对其解锁后，才能再次加锁，该锁部分读写，也叫全局锁。

```go
package main
 
import (
	"fmt"
	"sync"
)
 
func main() {
	var l sync.Mutex
	l.Lock()
	defer l.Unlock()
	fmt.Println("1")
}

out put : 1
```

``` go
import (
	"fmt"
	"sync"
)
 
func main() {
	var l sync.Mutex
	l.Unlock()
	fmt.Println("1")
	l.Lock()
}

out put : panic: sync: unlock of unlocked mutex
```



## RWMutex



RWMutex是一个读写锁，该锁可以加多个读锁或者一个写锁，其经常用于读次数远远多于写次数的场景．



``` go

type lockA struct {
	rwMutex sync.RWMutex
}

func main() {

	lockA := lockA{}
	go func() {
		lockA.rwMutex.RLock()
		defer lockA.rwMutex.RUnlock()
		fmt.Println("start111>", time.Now())
		time.Sleep(time.Second * 1)
		fmt.Println("end111>", time.Now())
	}()

	go func() {
		lockA.rwMutex.RLock()
		defer lockA.rwMutex.RUnlock()
		fmt.Println("start2222>", time.Now())
		time.Sleep(time.Second * 1)
		fmt.Println("end222>", time.Now())
	}()

	go func() {
		lockA.rwMutex.Lock()
		defer lockA.rwMutex.Unlock()
		fmt.Println("startLock>", time.Now())
		time.Sleep(time.Second * 1)
		fmt.Println("endLock>", time.Now())
	}()

	fmt.Println("startmain>", time.Now())

	lockA.rwMutex.Lock()
	defer lockA.rwMutex.Unlock()
	fmt.Println("startLock222>", time.Now())
	time.Sleep(time.Second * 1)
	fmt.Println("endLock2222>", time.Now())

	fmt.Println("endmain>", time.Now())

	time.Sleep(time.Second*10)

}
```

### 输出

```
startmain> 2020-06-05 14:30:31.830546 +0800 CST m=+0.000481665
start111> 2020-06-05 14:30:31.830556 +0800 CST m=+0.000491589
start2222> 2020-06-05 14:30:31.830576 +0800 CST m=+0.000511205
end111> 2020-06-05 14:30:32.832625 +0800 CST m=+1.002546010
end222> 2020-06-05 14:30:32.832625 +0800 CST m=+1.002546001
startLock> 2020-06-05 14:30:32.832718 +0800 CST m=+1.002639472
endLock> 2020-06-05 14:30:33.837619 +0800 CST m=+2.007526476
startLock222> 2020-06-05 14:30:33.83788 +0800 CST m=+2.007786891
endLock2222> 2020-06-05 14:30:34.840212 +0800 CST m=+3.010104619
endmain> 2020-06-05 14:30:34.840253 +0800 CST m=+3.010145824

```



### 死锁



###  RWMutex 源码

```go
func (rw *RWMutex) Lock() {
	if race.Enabled {
		_ = rw.w.state
		race.Disable()
	}
	// First, resolve competition with other writers.
	rw.w.Lock()
	// Announce to readers there is a pending writer.
	r := atomic.AddInt32(&rw.readerCount, -rwmutexMaxReaders) + rwmutexMaxReaders
	// Wait for active readers.
	if r != 0 && atomic.AddInt32(&rw.readerWait, r) != 0 {
		runtime_SemacquireMutex(&rw.writerSem, false)
	}
	if race.Enabled {
		race.Enable()
		race.Acquire(unsafe.Pointer(&rw.readerSem))
		race.Acquire(unsafe.Pointer(&rw.writerSem))
	}
}

func (rw *RWMutex) RUnlock() {
	if race.Enabled {
		_ = rw.w.state
		race.ReleaseMerge(unsafe.Pointer(&rw.writerSem))
		race.Disable()
	}
	if r := atomic.AddInt32(&rw.readerCount, -1); r < 0 {    //// readerCount进行非零判断
		if r+1 == 0 || r+1 == -rwmutexMaxReaders {
			race.Enable()
			throw("sync: RUnlock of unlocked RWMutex")
		}
		// A writer is pending.
		if atomic.AddInt32(&rw.readerWait, -1) == 0 {
			// The last reader unblocks the writer.
			runtime_Semrelease(&rw.writerSem, false)
		}
	}
	if race.Enabled {
		race.Enable()
	}
}

func (rw *RWMutex) RLock() {
	if race.Enabled {
		_ = rw.w.state
		race.Disable()
	}
	if atomic.AddInt32(&rw.readerCount, 1) < 0 {   //// 如果 readerCount 小于0，可以以为是等待写，
		// A writer is pending, wait for it.
		runtime_SemacquireMutex(&rw.readerSem, false)
	}
	if race.Enabled {
		race.Enable()
		race.Acquire(unsafe.Pointer(&rw.readerSem))
	}
}
```



### 死锁实例

```go
	
rwMutex := sync.RWMutex{}
rwMutex.Lock()
rwMutex.RLock()

out put : fatal error: all goroutines are asleep - deadlock!

```





### 总结

1. 读写互斥
2. 可重入读，不可重入写
3. 防止死锁

