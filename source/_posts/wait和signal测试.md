---
title: wait和signal测试
date: 2019-12-24 11:33:00
mathjax: true
tags:
- UNIX编程
---

```
$ uname -ra
Linux Rapture 4.15.0-72-generic #81~16.04.1-Ubuntu SMP Tue Nov 26 16:34:21 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
```

`signal.cpp`
```cpp
#include <cstdio>
#include <cstring>
#include <cstdlib>

#include <time.h>
#include <unistd.h>
#include <signal.h>

using namespace std;

void handle(int num) {
    printf("signal num: %d\n", num);
    sleep(5);
    printf("sleep over\n");
}

int main(int argc, char *argv[]) {
    for (int i = 1; i < argc;) {
        if (strcmp(argv[i], "--handle") == 0) {
            signal(atoi(argv[i + 1]), handle);
            i += 2;
            continue;
        }
        i++;
    }

    printf("pid: %d\n", getpid());

    while (true) {
        printf("%d\n", time(NULL));
        sleep(1);
    }
    return 0;
}
```
```
$ ./signal &
[1] 6223
pid: 6223
1577155844
1577155845
$ kill -KILL 6223
[1]  + 6223 killed     ./signal

$ ./signal --handle 9 &
[1] 6261
pid: 6261
1577155864
1577155865
$ kill -KILL 6261
[1]  + 6261 killed     ./signal --handle 9
```
```
$ ./signal &
[1] 6471
pid: 6471
1577156088
1577156089
$ kill -SIGSTOP 6471
[1]  + 6471 suspended (signal)  ./signal
......
$ kill -SIGCONT 6471
1577156113
1577156114
1577156115
1577156116
1577156117

$ ./signal --handle 18 19 &
[1] 6566
pid: 6566
1577156249
1577156250
$ kill -SIGSTOP 6566
[1]  + 6566 suspended (signal)  ./signal --handle 18 19
......
$ kill -SIGCONT 6566
signal num: 18
sleep over
1577156270
1577156271
```

```
$ ./signal --handle 15 &
[1] 7148
pid: 7148
1577156583
1577156584
1577156585
$ kill -SIGTERM 7148
$ kill -SIGTERM 7148
$ kill -SIGTERM 7148
$ kill -SIGTERM 7148
$ kill -SIGTERM 7148
signal num: 15
sleep over
signal num: 15
sleep over
1577156597
1577156598
1577156599
1577156600
1577156601
......
$ kill -KILL 7148
[1]  + 7148 killed     ./signal --handle 15
```

`wait.cpp`
```cpp
#include <cstdio>
#include <cstring>
#include <cstdlib>

#include <signal.h>
#include <time.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>

using namespace std;

void handle(int num) {
    printf("signal num: %d\n", num);
}

int main(int argc, char *argv[]) {
    int options = 0;
    bool divide_zero = false;
    bool invalid_memory = false;

    for (int i = 1; i < argc;) {
        if (strcmp(argv[i], "--WUNTRACED") == 0) {
            options |= WUNTRACED;
            i++;
            continue;
        } else if (strcmp(argv[i], "--WNOHANG") == 0) {
            options |= WNOHANG;
            i++;
            continue;
        } else if (strcmp(argv[i], "--WCONTINUED") == 0) {
            options |= WCONTINUED;
            i++;
            continue;
        } else if (strcmp(argv[i], "--handle") == 0) {
            signal(atoi(argv[i + 1]), handle);
            i += 2;
            continue;
        } else if (strcmp(argv[i], "--divide-zero") == 0) {
            divide_zero = true;
            i++;
            continue;
        } else if (strcmp(argv[i], "--invalid-memory") == 0) {
            invalid_memory = true;
            i++;
            continue;
        }
        i++;
    }

    pid_t cpid, w;
    int status;

    cpid = fork();
    if (cpid == -1) {
        fprintf(stderr, "fork failed\n");
        exit(EXIT_FAILURE);
    }

    if (cpid == 0) {
        printf("Child PID is %ld\n", (long) getpid());
        if (divide_zero) {
            int x = 1337;
            int y = x / 0;
            printf("%d\n", y);
        }
        if (invalid_memory) {
            *(int *) 0 = 1337;
        }
        while (true) {
            printf("time: %d\n", time(NULL));
            sleep(1);
        }
    } else {
        printf("Father PID is %ld\n", (long) getpid());
        do {
            w = waitpid(cpid, &status, options);
            printf("w: %d, WIFEXITED: %d, WIFSIGNALED: %d, WIFSTOPPED: %d, WIFCONTINUED: %d\n",
                   w, WIFEXITED(status), WIFSIGNALED(status), WIFSTOPPED(status), WIFCONTINUED(status));
            if (w == 0) {
                continue;
            }
            if (w == -1) {
                fprintf(stderr, "waitpid error\n");
                exit(EXIT_FAILURE);
            }
            if (WIFEXITED(status)) {
                printf("exited, status=%d\n", WEXITSTATUS(status));
            } else if (WIFSIGNALED(status)) {
                printf("killed by signal %d\n", WTERMSIG(status));
            } else if (WIFSTOPPED(status)) {
                printf("stopped by signal %d\n", WSTOPSIG(status));
            } else if (WIFCONTINUED(status)) {
                printf("continued\n");
            }
        } while ((!WIFEXITED(status) && !WIFSIGNALED(status)) || w == 0);
        exit(EXIT_SUCCESS);
    }
}
```

```
$ ./wait &
[1] 7523
Father PID is 7523
Child PID is 7525
time: 1577156913
time: 1577156914
time: 1577156915
time: 1577156916
$ kill 7525
w: 7525, WIFEXITED: 0, WIFSIGNALED: 1, WIFSTOPPED: 0, WIFCONTINUED: 0
killed by signal 15

$ ./wait --handle 15
Father PID is 7566
Child PID is 7567
time: 1577156938
time: 1577156939
time: 1577156940
time: 1577156941
$ kill 7567
signal num: 15
time: 1577156941
time: 1577156942
$ kill -KILL 7567
w: 7567, WIFEXITED: 0, WIFSIGNALED: 1, WIFSTOPPED: 0, WIFCONTINUED: 0
killed by signal 9
```

```
$ ./wait --divide-zero
Child PID is 7744
Father PID is 7743
w: 7744, WIFEXITED: 0, WIFSIGNALED: 1, WIFSTOPPED: 0, WIFCONTINUED: 0
killed by signal 8

$ ./wait --divide-zero --handle 8
Father PID is 7716
Child PID is 7717
signal num: 8
signal num: 8
signal num: 8
......
```

```
$ ./wait --invalid-memory
Father PID is 8085
Child PID is 8086
w: 8086, WIFEXITED: 0, WIFSIGNALED: 1, WIFSTOPPED: 0, WIFCONTINUED: 0
killed by signal 11

$ ./wait --invalid-memory --handle 11
Father PID is 8432
Child PID is 8433
signal num: 11
signal num: 11
......
```

```
$ ./wait --WNOHANG &
Father PID is 9033
Child PID is 9034
w: 0, WIFEXITED: 1, WIFSIGNALED: 0, WIFSTOPPED: 0, WIFCONTINUED: 0
w: 0, WIFEXITED: 1, WIFSIGNALED: 0, WIFSTOPPED: 0, WIFCONTINUED: 0
time: 1577157886
time: 1577157887
......
$ kill 9034
w: 9034, WIFEXITED: 0, WIFSIGNALED: 1, WIFSTOPPED: 0, WIFCONTINUED: 0
killed by signal 15
```

```
$ ./wait --WUNTRACED --WCONTINUED --handle 15 &
[1] 9312
Child PID is 9314
time: 1577158011
Father PID is 9312
time: 1577158013
time: 1577158014
$ kill 9314
signal num: 15
time: 1577158015
time: 1577158016
......
time: 1577158025
time: 1577158026
time: 1577158027
time: 1577158028
$ kill -SIGSTOP 9314
w: 9314, WIFEXITED: 0, WIFSIGNALED: 0, WIFSTOPPED: 1, WIFCONTINUED: 0
stopped by signal 19
$ kill -SIGCONT 9314
w: 9314, WIFEXITED: 0, WIFSIGNALED: 0, WIFSTOPPED: 0, WIFCONTINUED: 1
continued
time: 1577158038
...
time: 1577158041
$ kill -SIGKILL 9314
w: 9314, WIFEXITED: 0, WIFSIGNALED: 1, WIFSTOPPED: 0, WIFCONTINUED: 0
killed by signal 9
```