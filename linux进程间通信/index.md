# Linux进程间通信


<!--more-->

# 管道
## 匿名管道(Pipe)
**特点**
1. 半双工
2. 只能用于具有亲缘关系进程(父子，兄弟)
3. 可以视作特殊文件，读写可以使用write，read函数

```c
#include"stdio.h"
#include"unistd.h"

// https://learnku.com/articles/44477
// **特点**
// 1. 半双工
// 2. 只能用于具有亲缘关系进程(父子，兄弟)
// 3. 可以视作特殊文件，读写可以使用write，read函数

int main() {
    int fd[2];
    pid_t pid;
    char buf[32];

    //创建管道
    if (pipe(fd) < 0) {
        printf("Create Pipe Error \n");
    }

    //创建子进程
    if ((pid = fork()) < 0) {
        printf("Fork Error \n");
    }
    else if (pid > 0) {
        close(fd[0]); // 关闭父进程读端
        write(fd[1], "hello world", 8); //父进程写端写入
    }
    else {
        close(fd[1]); // 关闭子进程写端
        read(fd[0], buf, 8); // 子进程读取父进程消息
        printf("Child Recv Msg: %s", buf);
    }
}
```

## 命名管道(FIFO)
**特点**
1. 和匿名管道不同，FIFO可以在无关进程间通信
2. FIFO有路径名与之关联，以一种特殊文件形式存在于文件系统中

fifo_read.c

```c
#include"stdio.h"
#include"stdlib.h"
#include"unistd.h"
#include"errno.h"
#include"fcntl.h"
#include"sys/stat.h"

int main() {
    int fd;
    int len;
    char buf[1024];
    if (mkfifo("/home", 0666) < 0 && errno != EEXIST) {
        perror("Create FIFO Failed");
    }

    if ((fd = open("/home", O_RDONLY)) < 0) {
        perror("Open FIFO Failed");
        exit(1);
    }

    while ((len = read(fd, buf, 1024)) > 0) {
        printf("Read Message: %s", buf);
    }

    close(fd);

    return 0;
}
```

fifo_write.c

```c
#include"stdio.h"
#include"unistd.h"
#include"stdlib.h"
#include"fcntl.h" // O_WRONLY
#include"sys/stat.h"
#include"time.h"
// ## 命名管道(FIFO)
// ** 特点**
// 1. 和匿名管道不同，FIFO可以在无关进程间通信
// 2. FIFO有路径名与之关联，以一种特殊文件形式存在于文件系统中



int main() {
    int fd;
    int n, i;
    char buf[1024];
    time_t tp;

    printf("Parent Process PID:", getpid());

    if ((fd = open("/home", O_WRONLY)) < 0) { // 写模式打开FIFO
        perror("Open FIFO Failed");
        exit(1);
    }

    for (i = 0; i < 10; ++i) {
        time(&tp);// 当前系统时间
        n = sprintf(buf, "Process %d's time is %s", getpid(), ctime(&tp));
        printf("Send message: %s", buf);

        // 写入数据到FIFO中
        if (write(fd, buf, n + 1) < 0) {
            perror("Write FIFO Failed");
            close(fd);
            exit(1);
        }

        sleep(1);
    }

    close(fd);
    return 0;
}

```

# 消息队列
**特点**
1. 消息队列是面向记录的，其中的消息具有特定的格式以及特定的优先级。
2. 消息队列独立于发送与接收进程。进程终止时，消息队列及其内容并不会被删除。
3. 消息队列可以实现消息的随机查询，消息不一定要以先进先出的次序读取，也可以按消息的类型读取。

msg_client.c
```c
#include"stdio.h"
#include"stdlib.h"
#include"unistd.h"
#include"sys/msg.h"

#define MSG_FILE "/etc/passwd"

// 结构体第一个字段一定为长整型
struct msg_from
{
    long mtype;
    char mtext[256];
};

int main() {
    int msqid;
    key_t key;
    struct msg_from msg;

    // key值不变，要么确保ftok()的文件不被删除，要么不用ftok()，指定一个固定的key值。
    // 获取key值
    if ((key = ftok(MSG_FILE, 'z')) < 0) {
        perror("ftok error");
        exit(1);
    }

    printf("Message Queue - Client key is: %d.\n", key);

    // 打开消息队列
    if ((msqid = msgget(key, IPC_CREAT | 0777)) == -1) {
        perror("msgget error");
        exit(1);
    }

    // 打印消息队列ID和进程ID
    printf("My msqid is: %d.\n", msqid);
    printf("My pid is: %d.\n", getpid());

    // 添加消息，类型为888
    msg.mtype = 888;
    sprintf(msg.mtext, "hello, I'm client %d", getpid());
    //msgid是由msgget函数返回的消息队列标识符。

    // __msgp是一个指向准备发送消息的指针，消息的数据结构却有一定的要求，
    // 指针msg_ptr所指向的消息结构一定要是以一个**长整型**成员变量开始的结构体，接收函数将用这个成员来确定消息的类型
    msgsnd(msqid, &msg, sizeof(msg.mtext), 0);

    // msgrcv函数type参数有以下几种可能：
    // type == 0，返回队列中的第一个消息；
    // type > 0，返回队列中消息类型为 type 的第一个消息；
    // type < 0，返回队列中消息类型值小于或等于 type 绝对值的消息，如果有多个，则取类型值最小的消息。
    // 读取类型为999的消息
    msgrcv(msqid, &msg, 256, 999, 0);
    printf("Client: receive msg.mtext is: %s.\n", msg.mtext);
    printf("Client: receive msg.mtype is: %d.\n", msg.mtype);
}
```

msg_server.c

```c
#include"stdio.h"
#include"stdlib.h"
#include"unistd.h"
#include"sys/msg.h"

#define MSG_FILE "/etc/passwd"

// 结构体第一个字段一定为长整型
struct msg_from
{
    long mtype;
    char mtext[256];
};

int main() {
    int msqid;
    key_t key;
    struct msg_from msg;

    // 获取key值
    if ((key = ftok(MSG_FILE, 'z')) < 0) {
        perror("ftok error");
        exit(1);
    }

    // 打印key值
    printf("Message Queue - Server key is: %d.\n", key);

    // 创建消息队列
    if ((msqid = msgget(key, IPC_CREAT | 0700)) == -1) {
        perror("msgget error");
        exit(1);
    }

    printf("My msqid is: %d.\n", msqid);
    printf("My pid is: %d.\n", getpid());

    for (;;) {
        // 返回类型为888的第一个消息
        msgrcv(msqid, &msg, 256, 888, 0);

        printf("Server: receive msg.mtext is: %s.\n", msg.mtext);
        printf("Server: receive msg.mtype is: %d.\n", msg.mtype);

        msg.mtype = 999;
        sprintf(msg.mtext, "hello, I'm server %d", getpid());
        msgsnd(msqid, &msg, sizeof(msg.mtext), 0);
    }

    return 0;
}

```

# 信号量

**特点**
1. 是一个计数器
2. 用于实现进程间互斥和同步,不是存储进程间通信数据

**示例**
```c
#include"stdio.h"
#include"stdlib.h"
#include"unistd.h"
#include"sys/sem.h"

union semun
{
    int val;
    struct semid_ds* buf;
    unsigned short* array;
};

int init_sem(int sem_id, int value) {
    union semun tmp;
    tmp.val = value;
    if (semctl(sem_id, 0, SETVAL, tmp) == -1) {
        perror("Init Semaphore Error");
        return -1;
    }

    return 0;
}

// p操作
// 若信号量值为1，获取资源并将信号值置为-1
// 若信号量值为0，进程挂起等待
int sem_p(int sem_id) {
    struct sembuf sbuf;
    sbuf.sem_num = 0; // 序号
    sbuf.sem_op = -1; // 操作
    sbuf.sem_flg = SEM_UNDO;

    if (semop(sem_id, &sbuf, 1) == -1) {
        perror("P operation Error");
        return -1;
    }

    return 0;
}

// v操作
// 释放资源并将信号量值+1
// 如果有进程正在挂起等待，则唤醒
int sem_v(int sem_id) {
    struct sembuf sbuf;
    sbuf.sem_num = 0; //序号
    sbuf.sem_op = 1; //v操作
    sbuf.sem_flg = SEM_UNDO;

    if (semop(sem_id, &sbuf, 1) == -1) {
        perror("V operation Error");
        return -1;
    }

    return 0;
}

// 删除信号量集
int del_sem(int sem_id) {
    union semun tmp;
    if (semctl(sem_id, 0, IPC_RMID, tmp) == -1) {
        perror("Delete Semaphore Error");
        return -1;
    }

    return 0;
}

int main() {
    int sem_id;
    key_t key;
    pid_t pid;

    // 获取key值
    if ((key = ftok("/home", 'z')) < 0) {
        perror("ftok error");
        exit(1);
    }

    // 创建信号量集，其中只有一个信号量集
    if ((sem_id == semget(key, 1, IPC_CREAT | 0666)) == -1) {
        perror("semget error");
        exit(1);
    }

    //初始化：初值设为0资源被占用
    init_sem(sem_id, 0);

    if ((pid = fork()) == -1) {
        perror("Fork Error");
    }
    else if (pid == 0) { // 子进程创建成功
        sleep(2);
        printf("Process child: pid=%d\n", getpid());
        sem_v(sem_id);
    }
    else {
        sem_p(sem_id);
        printf("Process father: pid=%d\n", getpid());
        sem_v(sem_id);
        del_sem(sem_id);
    }

    return 0;
}
```


# 共享内存
**特点**
1. 共享内存是最快的一种 IPC，因为进程是直接对内存进行存取。
2. 因为多个进程可以同时操作，所以需要进行同步。
3. 信号量 + 共享内存通常结合在一起使用，信号量用来同步对共享内存的访问。

**示例**
```
#include"stdio.h"
#include"stdlib.h"
#include"unistd.h"
#include"sys/shm.h"
#include"string.h"

int main() {

    key_t key;
    int shmid;
    if ((key = ftok("/etc", 'k')) < 0) {
        perror("ftok error");
        exit(-1);
    }

    pid_t pid;

    if ((pid = fork()) < 0) {
        perror("fork error");
        exit(-1);
    }
    else if (pid == 0) {
        // 创建共享内存
        shmid = shmget(key, 1024 * 1024 * 10, IPC_CREAT | 0666);

        // 连接共享内存到当前进程
        // 0 表示可读可写
        char *s = (char*)shmat(shmid, NULL, 0);

        printf("shm write data\n");
        memset(s, 'B', 1024);

        // 卸载共享内存
        shmdt(s);

    }
    else {
        // 创建共享内存
        shmid = shmget(key, 1024 * 1024 * 10, IPC_CREAT | 0666);

        // 连接共享内存到当前进程
        // SHM_RDONLY表示只读
        char *str = (char*)shmat(shmid, NULL, SHM_RDONLY);

        printf("read data from shm is:%s\n", str);

        // 卸载共享内存
        shmdt(str);
    }

    return 0;
}
```


# 信号
**特点**
1. 可靠信号与不可靠信号，前32种信号为不可靠信号，后32种为可靠信号。
2. 不可靠信号： 也称为非实时信号，不支持排队，信号可能会丢失, 比如发送多次相同的信号, 进程只能收到一次. 信号值取值区间为1~31；
3. 可靠信号： 也称为实时信号，支持排队, 信号不会丢失, 发多少次, 就可以收到多少次. 信号值取值区间为32~64

**示例**
```c
#include <sys/types.h>
#include <signal.h>
#include<stdio.h>
#include <unistd.h>


int main(int argc, char** argv)
{
    if (3 != argc)
    {
        printf("[Arguments ERROR!]\n");
        printf("\tUsage:\n");
        printf("\t\t%s <Target_PID> <Signal_Number>\n", argv[0]);
        return -1;
    }
    int pid = atoi(argv[1]);
    int sig = atoi(argv[2]);
    if (pid > 0 && sig > 0)
    {
        kill(pid, sig);
    }
    else
    {
        printf("Target_PID or Signal_Number MUST bigger than 0!\n");
    }

    return 0;
}

```

# 套接字
**特点**
1. socket可以在同一个主机也可以在不同主机间进行通信

**示例**
```c
#include"stdio.h"
#include"stdlib.h"
#include"unistd.h"
#include"sys/socket.h"

int main() {
    int fd[2];
    int pair = socketpair(AF_UNIX, SOCK_STREAM, 0, fd);
    if (pair < 0) {
        perror("socketpair()");
        exit(1);
    }

    pid_t pid = fork();

    if (pid < 0) {
        perror("fork failure");
        exit(1);
    }
    else if (pid > 0) {
        close(fd[1]);
        printf("child process pid: %d\n", getpid());
        char* send_str = "this is child process msg\0";
        char* buf[1024];
        while (1)
        {
            write(fd[0], &send_str, sizeof(send_str));
            read(fd[0], &buf, sizeof(buf));
            printf("recv msg from parent process is: %s\n", buf);
        }

    }
    else {
        sleep(2);
        close(fd[0]);
        printf("parent process pid: %d\n", getpid());
        char* send_str = "this is parent process msg\0";
        char* buf[1024];
        read(fd[1], &buf, sizeof(buf));
        printf("recv msg from child process is: %s\n", buf);
        write(fd[1], &send_str, sizeof(send_str));
        sleep(5);
    }


}
```

**参考链接**

[进程间的五种通信方式介绍-详解 | Laravel China 社区 (learnku.com)](https://learnku.com/articles/44477)

---

> 作者:   
> URL: https://release.blog-12x.pages.dev/linux%E8%BF%9B%E7%A8%8B%E9%97%B4%E9%80%9A%E4%BF%A1/  

