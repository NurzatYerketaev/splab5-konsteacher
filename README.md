# Lab5 - race condition and critical section

This lab is the homework of [Chapter 26](http://www.cs.wisc.edu/~remzi/OSTEP/threads-intro.pdf)

Format your answers neatly and submit.

### Q1

Used the codes [x86.py](https://github.com/sduclassroom/splab5-NurzatYerketaev/blob/master/x86.py) and [loop.s](https://github.com/sduclassroom/splab5-NurzatYerketaev/blob/master/loop.s)

```
./x86.py -p loop.s -t 1 -i 100 -R dx -c
...
   dx          Thread 0         
    0   
   -1   1000 sub  $1,%dx
   -1   1001 test $0,%dx
   -1   1002 jgte .top
   -1   1003 halt
```

---

__1.__ Can you figure out what the value of %dx will be during the run?

_`%dx` will be `-1`._



### Q2

```
./x86.py -p loop.s -t 2 -i 100 -a dx=3,dx=3 -R dx
...
    dx          Thread 0                Thread 1         
    3   
    2   1000 sub  $1,%dx
    2   1001 test $0,%dx
    2   1002 jgte .top
    1   1000 sub  $1,%dx
    1   1001 test $0,%dx
    1   1002 jgte .top
    0   1000 sub  $1,%dx
    0   1001 test $0,%dx
    0   1002 jgte .top
   -1   1000 sub  $1,%dx
   -1   1001 test $0,%dx
   -1   1002 jgte .top
   -1   1003 halt
    3   ----- Halt;Switch -----  ----- Halt;Switch -----  
    2                            1000 sub  $1,%dx
    2                            1001 test $0,%dx
    2                            1002 jgte .top
    1                            1000 sub  $1,%dx
    1                            1001 test $0,%dx
    1                            1002 jgte .top
    0                            1000 sub  $1,%dx
    0                            1001 test $0,%dx
    0                            1002 jgte .top
   -1                            1000 sub  $1,%dx
   -1                            1001 test $0,%dx
   -1                            1002 jgte .top
   -1                            1003 halt
```
---

__1.__ What values will %dx see?
_`%dx` see firstly 3, which we give as the value, after that it's decrements by 1 for every iteration, until value be equal to `-1`._

__2.__ Does the presence of multiple threads affect anything about your calculations? Is there a race condition in this code?

_Multiple threads don't affect our calculations because there is no race condtion. The threads are executed without interleavings because the threads complete before an interrupt occurs._




### Q3

```
./x86.py -p loop.s -t 2 -i 3 -r -a dx=3,dx=3 -R dx
...
   dx          Thread 0                Thread 1         
    3   
    2   1000 sub  $1,%dx
    2   1001 test $0,%dx
    2   1002 jgte .top
    3   ------ Interrupt ------  ------ Interrupt ------  
    2                            1000 sub  $1,%dx
    2                            1001 test $0,%dx
    2                            1002 jgte .top
    2   ------ Interrupt ------  ------ Interrupt ------  
    1   1000 sub  $1,%dx
    1   1001 test $0,%dx
    2   ------ Interrupt ------  ------ Interrupt ------  
    1                            1000 sub  $1,%dx
    1   ------ Interrupt ------  ------ Interrupt ------  
    1   1002 jgte .top
    0   1000 sub  $1,%dx
    1   ------ Interrupt ------  ------ Interrupt ------  
    1                            1001 test $0,%dx
    1                            1002 jgte .top
    0   ------ Interrupt ------  ------ Interrupt ------  
    0   1001 test $0,%dx
    0   1002 jgte .top
   -1   1000 sub  $1,%dx
    1   ------ Interrupt ------  ------ Interrupt ------  
    0                            1000 sub  $1,%dx
   -1   ------ Interrupt ------  ------ Interrupt ------  
   -1   1001 test $0,%dx
   -1   1002 jgte .top
    0   ------ Interrupt ------  ------ Interrupt ------  
    0                            1001 test $0,%dx
    0                            1002 jgte .top
   -1   ------ Interrupt ------  ------ Interrupt ------  
   -1   1003 halt
    0   ----- Halt;Switch -----  ----- Halt;Switch -----  
   -1                            1000 sub  $1,%dx
   -1                            1001 test $0,%dx
   -1   ------ Interrupt ------  ------ Interrupt ------  
   -1                            1002 jgte .top
   -1                            1003 halt
```

This makes the interrupt interval quite small and random; use different seeds with `-s` to see different interleavings.

---

__1.__ Does the frequency of interruption change anything about this program?

_Frequency affect the interleavings, i.e. the frequency of interleavings, but not the result, because the two threads don't have critical sections._


### Q4

Used the codes [x86.py](https://github.com/sduclassroom/splab5-NurzatYerketaev/blob/master/x86.py) and [looping-race-nolock.s](https://github.com/sduclassroom/splab5-NurzatYerketaev/blob/master/looping-race-nolock.s)

```
./x86.py -p looping-race-nolock.s -t 1 -M 2000 -c
...
 2000          Thread 0         
    0   
    0   1000 mov 2000, %ax
    0   1001 add $1, %ax
    1   1002 mov %ax, 2000
    1   1003 sub  $1, %bx
    1   1004 test $0, %bx
    1   1005 jgt .top
    1   1006 halt

```

---

__1.__ What value is found in x (i.e., at memory address 2000) throughout the run? Use `-c` to check your answer.

_The values of x show the memory/register contents AFTER the instruction on the right has executed. For example, after the "add" instruction, we can see that x has been incremented to the value 1; after the second "mov"
instruction, also we can see that the memory contents at 2000 are now also incremented._


### Q5

```
./x86.py -p looping-race-nolock.s -t 2 -a bx=3 -M 2000
...
2000          Thread 0                Thread 1         
    0   
    0   1000 mov 2000, %ax
    0   1001 add $1, %ax
    1   1002 mov %ax, 2000
    1   1003 sub  $1, %bx
    1   1004 test $0, %bx
    1   1005 jgt .top
    1   1000 mov 2000, %ax
    1   1001 add $1, %ax
    2   1002 mov %ax, 2000
    2   1003 sub  $1, %bx
    2   1004 test $0, %bx
    2   1005 jgt .top
    2   1000 mov 2000, %ax
    2   1001 add $1, %ax
    3   1002 mov %ax, 2000
    3   1003 sub  $1, %bx
    3   1004 test $0, %bx
    3   1005 jgt .top
    3   1006 halt
    3   ----- Halt;Switch -----  ----- Halt;Switch -----  
    3                            1000 mov 2000, %ax
    3                            1001 add $1, %ax
    4                            1002 mov %ax, 2000
    4                            1003 sub  $1, %bx
    4                            1004 test $0, %bx
    4                            1005 jgt .top
    4                            1000 mov 2000, %ax
    4                            1001 add $1, %ax
    5                            1002 mov %ax, 2000
    5                            1003 sub  $1, %bx
    5                            1004 test $0, %bx
    5                            1005 jgt .top
    5                            1000 mov 2000, %ax
    5                            1001 add $1, %ax
    6                            1002 mov %ax, 2000
    6                            1003 sub  $1, %bx
    6                            1004 test $0, %bx
    6                            1005 jgt .top
    6                            1006 halt


```

---

__1.__ Do you understand why the code in each thread loops three times? 

_`-t` the number of threads, and the interrupt interval, which is how often a scheduler will be woken
and run to switch to a different task. If there is only one thread in code, this interval does not matter. If there is `-t 2`, after first thread(3 of them) is done, then will be appear the __"Halt;Switch"__ line, which  is inserted whenever a thread halts and another thread must be run._

__2.__ What will the final value of x be?

_Final value will be `6`._


### Q6

```
./x86.py -p looping-race-nolock.s -t 2 -M 2000 -i 4 -r -s 0
...
 2000          Thread 0                Thread 1         
    0   
    0   1000 mov 2000, %ax
    0   1001 add $1, %ax
    1   1002 mov %ax, 2000
    1   1003 sub  $1, %bx
    1   ------ Interrupt ------  ------ Interrupt ------  
    1                            1000 mov 2000, %ax
    1                            1001 add $1, %ax
    2                            1002 mov %ax, 2000
    2                            1003 sub  $1, %bx
    2   ------ Interrupt ------  ------ Interrupt ------  
    2   1004 test $0, %bx
    2   ------ Interrupt ------  ------ Interrupt ------  
    2                            1004 test $0, %bx
    2   ------ Interrupt ------  ------ Interrupt ------  
    2   1005 jgt .top
    2   1006 halt
    2   ----- Halt;Switch -----  ----- Halt;Switch -----  
    2                            1005 jgt .top
    2                            1006 halt
```

Then change the random seed, setting `-s 1`, then `-s 2`, etc. 

---

__1.__ Can you tell, just by looking at the thread interleaving, what the final value of x will be?

_The threads are executed with interleavings because the threads complete after an interrupt occurs._

__2.__ Does the exact location of the interrupt matter? Where can it safely occur? 

_Location of interrupt is important because it's determines thread interleavings._

__3.__ Where does an interrupt cause trouble? In other words, where is the critical section exactly?

_The code has a critical section which loads the value of a variable  (at address 2000), then adds 1 to the value, then stores it back._




### Q7

```
./x86.py -p looping-race-nolock.s -a bx=1 -t 2 -M 2000 -i 1
...
 2000          Thread 0                Thread 1         
    0   
    0   1000 mov 2000, %ax
    0   ------ Interrupt ------  ------ Interrupt ------  
    0                            1000 mov 2000, %ax
    0   ------ Interrupt ------  ------ Interrupt ------  
    0   1001 add $1, %ax
    0   ------ Interrupt ------  ------ Interrupt ------  
    0                            1001 add $1, %ax
    0   ------ Interrupt ------  ------ Interrupt ------  
    1   1002 mov %ax, 2000
    1   ------ Interrupt ------  ------ Interrupt ------  
    1                            1002 mov %ax, 2000
    1   ------ Interrupt ------  ------ Interrupt ------  
    1   1003 sub  $1, %bx
    1   ------ Interrupt ------  ------ Interrupt ------  
    1                            1003 sub  $1, %bx
    1   ------ Interrupt ------  ------ Interrupt ------  
    1   1004 test $0, %bx
    1   ------ Interrupt ------  ------ Interrupt ------  
    1                            1004 test $0, %bx
    1   ------ Interrupt ------  ------ Interrupt ------  
    1   1005 jgt .top
    1   ------ Interrupt ------  ------ Interrupt ------  
    1                            1005 jgt .top
    1   ------ Interrupt ------  ------ Interrupt ------  
    1   1006 halt
    1   ----- Halt;Switch -----  ----- Halt;Switch -----  
    1   ------ Interrupt ------  ------ Interrupt ------  
    1                            1006 halt
```

---

__1.__ See if you can guess what the final value of the shared variable x will be. What about when you change -i 2, -i 3, etc.? 

_Yes we can guess. Final value of x will be `1`, for `-i 1` and `-i 2`, after we run with `-i 3` and others final value will be `2`. Because there is we select `-t` as 2 , after that it's not effect to our code, final value can be only maximum as `2`._

__2.__ For which interrupt intervals does the program give the вЂњcorrectвЂќ final answer?

_Correct answer is the value of `bx`, is `1`, when `-i` is lower then or equal to `-t`'s value_


### Q8

```
./x86.py -p looping-race-nolock.s -a bx=100 -t 2 -M 2000 -i 1
...
2000          Thread 0                Thread 1         
    0   
    0   1000 mov 2000, %ax
    0   ------ Interrupt ------  ------ Interrupt ------  
    0                            1000 mov 2000, %ax
    0   ------ Interrupt ------  ------ Interrupt ------  
    0   1001 add $1, %ax
    0   ------ Interrupt ------  ------ Interrupt ------  
    0                            1001 add $1, %ax
    0   ------ Interrupt ------  ------ Interrupt ------  
...
  100   1002 mov %ax, 2000
  100   ------ Interrupt ------  ------ Interrupt ------  
  100                            1002 mov %ax, 2000
  100   ------ Interrupt ------  ------ Interrupt ------  
  100   1003 sub  $1, %bx
  100   ------ Interrupt ------  ------ Interrupt ------  
  100                            1003 sub  $1, %bx
  100   ------ Interrupt ------  ------ Interrupt ------  
  100   1004 test $0, %bx
  100   ------ Interrupt ------  ------ Interrupt ------  
  100                            1004 test $0, %bx
  100   ------ Interrupt ------  ------ Interrupt ------  
  100   1005 jgt .top
  100   ------ Interrupt ------  ------ Interrupt ------  
  100                            1005 jgt .top
  100   ------ Interrupt ------  ------ Interrupt ------  
  100   1006 halt
  100   ----- Halt;Switch -----  ----- Halt;Switch -----  
  100   ------ Interrupt ------  ------ Interrupt ------  
  100                            1006 halt

```

---
Now run the same code for more loops (e.g., set -a bx=100).

__1.__ What interrupt intervals, set with the -i flag, lead to a вЂњcorrectвЂќ outcome? Which intervals lead to surprising results?

_Correct outcome is the value of `bx`, is `100`, when `-i` is lower then or equal to `-t`'s value_



### Q9

Used the codes [x86.py](https://github.com/sduclassroom/splab5-NurzatYerketaev/blob/master/x86.py) and [wait-for-me.s](https://github.com/sduclassroom/splab5-NurzatYerketaev/blob/master/wait-for-me.s)

```
./x86.py -p wait-for-me.s -a ax=1,ax=0 -R ax -M 2000
...
 2000      ax          Thread 0                Thread 1         
    0       1   
    0       1   1000 test $1, %ax
    0       1   1001 je .signaller
    1       1   1006 mov  $1, 2000
    1       1   1007 halt
    1       0   ----- Halt;Switch -----  ----- Halt;Switch -----  
    1       0                            1000 test $1, %ax
    1       0                            1001 je .signaller
    1       0                            1002 mov  2000, %cx
    1       0                            1003 test $1, %cx
    1       0                            1004 jne .waiter
    1       0                            1005 halt
```

---

__1.__ How should the code behave? How is the value at location 2000 being used by the threads? 

_The values show the memory/register contents AFTER the instruction on the right has executed. For example, after the "je .signaller" instruction, we can see that the value has been incremented to the value 1._

__2.__ What will its final value be?

_Final value will be '1'._

### Q10

```
./x86.py -p wait-for-me.s -a ax=0,ax=1 -R ax -M 2000
...
 2000      ax          Thread 0                Thread 1         
    0       0   
    0       0   1000 test $1, %ax
    0       0   1001 je .signaller
    0       0   1002 mov  2000, %cx
    0       0   1003 test $1, %cx
    0       0   1004 jne .waiter
    0       0   1002 mov  2000, %cx
    0       0   1003 test $1, %cx
    0       0   1004 jne .waiter
    0       0   1002 mov  2000, %cx
    0       0   1003 test $1, %cx
    0       0   1004 jne .waiter
    0       0   1002 mov  2000, %cx
    0       0   1003 test $1, %cx
    0       0   1004 jne .waiter
    0       0   1002 mov  2000, %cx
    0       0   1003 test $1, %cx
    0       0   1004 jne .waiter
    0       0   1002 mov  2000, %cx
    0       0   1003 test $1, %cx
    0       0   1004 jne .waiter
    0       0   1002 mov  2000, %cx
    0       0   1003 test $1, %cx
    0       0   1004 jne .waiter
    0       0   1002 mov  2000, %cx
    0       0   1003 test $1, %cx
    0       0   1004 jne .waiter
    0       0   1002 mov  2000, %cx
    0       0   1003 test $1, %cx
    0       0   1004 jne .waiter
    0       0   1002 mov  2000, %cx
    0       0   1003 test $1, %cx
    0       0   1004 jne .waiter
    0       0   1002 mov  2000, %cx
    0       0   1003 test $1, %cx
    0       0   1004 jne .waiter
    0       0   1002 mov  2000, %cx
    0       0   1003 test $1, %cx
    0       0   1004 jne .waiter
    0       0   1002 mov  2000, %cx
    0       0   1003 test $1, %cx
    0       0   1004 jne .waiter
    0       0   1002 mov  2000, %cx
    0       0   1003 test $1, %cx
    0       0   1004 jne .waiter
    0       0   1002 mov  2000, %cx
    0       0   1003 test $1, %cx
    0       0   1004 jne .waiter
    0       0   1002 mov  2000, %cx
    0       0   1003 test $1, %cx
    0       0   1004 jne .waiter
    0       1   ------ Interrupt ------  ------ Interrupt ------  
    0       1                            1000 test $1, %ax
    0       1                            1001 je .signaller
    1       1                            1006 mov  $1, 2000
    1       1                            1007 halt
    1       0   ----- Halt;Switch -----  ----- Halt;Switch -----  
    1       0   1002 mov  2000, %cx
    1       0   1003 test $1, %cx
    1       0   1004 jne .waiter
    1       0   1005 halt
```

---

__1.__ How do the threads behave?

_After `.signaller`'s code it's jumping to the `.waiter` and this code makes cpu to wait several times._

__2.__ What is thread 0 doing?

_Thread `0` is doing contains before the second mov of `.signaller`'s code._

__3.__ How would changing the interrupt interval (e.g., -i 1000, or perhaps to use random intervals) change the trace outcome?

_When we change row as `./x86.py -p wait-for-me.s -a ax=0,ax=1 -i 1000 -R ax -M 2000 -c` there is no change in output._

__4.__ Is the program efficiently using the CPU?

_The `-c` argument is what enables changing the вЂњCPU AffinityвЂќ thus should be untouched. Because of that we use `-c`, we can say that we are using CPU too._

---
