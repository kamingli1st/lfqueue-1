# lfqueue [![Build Status](https://travis-ci.org/darkautism/lfqueue.svg?branch=master)](https://travis-ci.org/darkautism/lfqueue)

Minimize lock-free queue, it's easy to use and easy to read. It's only 50 line code so it is easy to understand, best code for education ever!

Support multiple comsumer and multiple producer at sametime.

## How to use

Just copy past everywhere and use it. If you copy these code into your project, you must use -O0 flag to compile.

## Example

It is an minimize sample code for how to using lfq.

``` c
#include <stdio.h>
#include <stdlib.h>
#include "lfq.h"

int main() {
	long ret;
	struct lfq_ctx ctx;
	lfq_init(&ctx);
	lfq_enqueue(&ctx,(void *)1);
	lfq_enqueue(&ctx,(void *)3);
	lfq_enqueue(&ctx,(void *)5);
	lfq_enqueue(&ctx,(void *)8);
	lfq_enqueue(&ctx,(void *)4);
	lfq_enqueue(&ctx,(void *)6);

	while ( (ret = (long)lfq_dequeue(&ctx)) != 0 )
		printf("lfq_dequeue %ld\n", ret);

	lfq_clean(&ctx);
	return 0;
}
```

## Issues

### ENOMEM

This lfqueue do not have size limit, so count your struct yourself.

### Can i iterate lfqueue inner struct?

No, iterate inner struct is not threadsafe.

If you do not get segmentation fault because you are iterate lfqueue in single thread.

We should always iterate lfqueue by `lfq_enqueue` and `lfq_dequeue` method.

### CPU time slice waste? Other thread blocking and waiting for swap?

**Enqueue**

No, CAS operator not lock. Loser not block and wait winner.

If a thread race win a CAS, other thread will get false and try to retrive next pointer. Because winner already swap tail, so other losers can do next race.

**Example:**
```
4 thread race push
1 win Push A, 2 lose, 1 do not have CPU time slice
1 win go out queue, 2 losers race A as tail, 1 race initnode as tail
1 win go out queue, 1 win push B, 1 lose, 1 race A failed because tail not initnode now
```

So lock-free queue have better performance then lock queue.

**Dequeue**

This lfq dequeue like mini spin lock, so dequeue is thread safe, but it's not suggestion dequeue with multiple thread.

There has many papers to resolve this problem:

- Lock-FreeReference Counting
- ABA-Prevention Tags
- Hazard Pointers

### CPU cache miss?

This issue cannot been resolved. CAS operator always cause this problem.

We can resolved it if cpu will not cache miss after CAS.

### ABA problem?

No, ABA problem will segement fault. But we won't.

## Double width compare and swap (DWACS)

Branch [experimental-DCAS](https://github.com/darkautism/lfqueue/tree/experimental-DCAS), still

## License

WTFPL
