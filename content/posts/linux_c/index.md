+++
date = '2024-09-26T22:27:43+08:00'
title = 'Linux系统程序设计实验记录'
tags = ["linux","c"]
+++

学校里开设的稍微有点用的一节课，所有代码均在ubuntu20.04下测试成功。
# 文件操作
1、利用creat()、open()和close()函数进行系统调用，实现创建、打开和关闭一个文件。
(1)以可读写方式新建并关闭一个文件“stu.txt”，如果成功则返回新建文件的文
件名，不成功则输出错误提示信息。
(2)	以可读写方式打开并关闭一个文件“file.txt”，如果成功则返回文件的文件名，
不成功则输出错误提示信息。
```c
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdio.h>

int main()
{
	const char *stufilename = "stu.txt";
	int stufd = creat(stufilename, S_IRWXG);
	if (stufd == -1)
	{
		printf("%s creat failed\n", stufilename);
	}
	close(stufd);

	const char *filename = "file.txt";
	int filefd = open(filename, O_RDWR | O_CREAT | O_TRUNC, 0777);
	if (filefd == -1)
	{
		printf("%s open failed!\n", filename);
	}

	return 0;
}

```
2、编程实现一个简单的学生信息管理程序：
(1)从键盘输入3个学生信息（学号、姓名、年龄），存放于结构体数组中，将结
构体写入文件“stu.txt”中。
(2)从键盘输入某个学生的学号，从“stu.txt”文件中读出指定学号的学生的全部
信息并显示。
(3)	实现人机交互操作：可以循环查询并显示查询结果，直到用户不再操作，结
束程序运行。
```c
#include <fcntl.h>
#include <unistd.h>
#include <stdio.h>
#include <stdbool.h>

typedef struct
{
    int id;
    char name[50];
    int age;
} Student;

const char *StuFileName = "./stu.txt";

int inputStudents(int fd, int count);
int searchStudent(int fd, int id, int count);

int main()
{
    int stufd = open(StuFileName, O_RDWR | O_CREAT | O_TRUNC, 0777);
    if (stufd == -1)
    {
        printf("%s creat failed\n", StuFileName);
        return 0;
    }

    int inre = inputStudents(stufd, 3);
    if (inre == -1)
    {
        printf("写入失败\n");
    }

    while (true)
    {
        int id;
        printf("请输入要查找的学生id(输入-1退出):");
        scanf("%d", &id);
        if (id == -1)
            break;

        int sere = searchStudent(stufd, id, 3);
        if (inre == -1)
        {
            printf("查找失败\n");
        }
    }

    close(stufd);

    return 0;
}

int inputStudents(int fd, int count)
{
    Student student;
    for (int i = 0; i < count; i++)
    {
        printf("输入第 %d 个学生的信息：\n", i + 1);
        printf("学号：");
        scanf("%d", &student.id);
        printf("姓名：");
        scanf("%s", student.name);
        printf("年龄：");
        scanf("%d", &student.age);

        off_t fdof = lseek(fd, i * sizeof(student), SEEK_SET);
        if (fdof == -1)
            return -1;

        if (write(fd, &student, sizeof(student)) == -1)
        {
            printf("write err\n");
            return -1;
        }
    }

    off_t fdof = lseek(fd, 0, SEEK_SET);
    if (fdof == -1)
        return -1;

    return 0;
}

int searchStudent(int fd, int id, int count)
{
    Student student;
    for (int i = 0; i < count; i++)
    {
        off_t fdof = lseek(fd, i * sizeof(student), SEEK_SET);
        if (fdof == -1)
            return -1;
        if (read(fd, &student, sizeof(student)) == -1)
        {
            printf("read err\n");
            return -1;
        }

        if (student.id == id)
        {
            printf("查找到：\n");
            printf("姓名：%s\n", student.name);
            printf("年龄：%d\n", student.age);
        }
    }

    if (student.id != id) {
        printf("未查找到！\n");
    }

    off_t fdof = lseek(fd, 0, SEEK_SET);
    if (fdof == -1)
        return -1;

    return 0;
}
```
# 进程创建
1、使用fork()函数循环创建n个子进程（n≥2），并显示每个子进程的进程号以
及其父进程的进程号。
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/wait.h>

void create_processes(int n)
{
    for (int i = 0; i < n; i++)
    {
        pid_t pid = fork();
        if (pid < 0)
        {
            printf("进程创建失败");
            exit(1);
        }
        else if (pid == 0)
        {
            printf("创建成功，子进程 PID: %d, 父进程 PID: %d\n", getpid(), getppid());
            exit(0);
        }
    }

    for (int i = 0; i < n; i++)
    {
        wait(NULL);
    }
}

int main()
{
    int n = 0;
    printf("需要创建进程的数量:");
    scanf("%d", &n);
    create_processes(n);
    return 0;
}
```
2、使用fork()函数创建一个子进程，在子进程中使用execl()函数执行ls命令
（ls命令的参数自定），并在屏幕上显示“任务完成！”提示。
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h> 
#include <sys/wait.h>
#include<sys/types.h>
#include<stdbool.h>

void create_processes(int n)
{
    for (int i = 0; i < n; i++)
    {
        pid_t pid = fork();
        if (pid < 0)
        {
            printf("进程创建失败");
            exit(1);
        }
        else if (pid == 0)
        {
            printf("子进程%d，开始执行ls -l\n", getpid());
            execl("/bin/ls", "ls", "-l", NULL);
            exit(0);
        }
        int status = 0;

        waitpid(pid, &status, 0);
        bool a = WIFEXITED(status);
        if (a) {
            printf("子进程%d，执行ls -l成功\n", pid);
        }
    }

    for (int i = 0; i < n; i++)
    {
        wait(NULL);
    }
}

int main()
{
    int n = 0;
    printf("需要创建进程的数量:");
    scanf("%d", &n);
    create_processes(n);
    return 0;
}
```
3、使用system()函数顺序执行以上两个可执行程序。
```c
#include <stdio.h>
#include <stdlib.h>

int main() {
    system("./create_process");
    system("./execute_ls");
    return 0;
}
```
# 进程间通信
1、在主程序中用pipe()创建一个管道，用fork()创建一个子进程，其父进程接收
从键盘输入一个字符串，并写入管道中；子进程从管道中读取该字符串，显示信息如下：
  ①统计字符串中字符的个数，并显示；
  ②显示字符串内容。
```c
#include<stdio.h>
#include<string.h>
#include<unistd.h>
#include<sys/types.h>

#define ERR -1

int main(void)
{
    int res = ERR;
    int fd[2], nbytes;

    int *write_fd = &fd[1];
    int *read_fd = &fd[0];

    res = pipe(fd);

    if (res == ERR)
    {
        printf("创建管道失败！");
        return ERR;
    }

    pid_t pid = fork();

    if (pid == ERR)
    {
        printf("创建子进程失败！");
        return -1;
    }

    if (pid == 0)
    {
        close(*read_fd);
        char string[100];
        printf("向子进程写入:");
        scanf("%s", string);
        write(*write_fd, string, strlen(string));
    }
    else
    {
        close(*write_fd);
        char readbufffer[100];
        ssize_t num = read(*read_fd, readbufffer, sizeof(readbufffer));
        printf("子进程(pid=%d)，接收到%ld字节数据: %s\n", pid, num, readbufffer);
    }

    return 0;
}
```
2、编写两个程序，程序1接收到SIGUSR1信号时显示一行信息：“sigusr1 signal 
received!”，程序2每隔2秒钟发送一个SIGUSR1信号给程序1。
注意：该题由两个程序组成，需要在两个窗口分别运行。运行顺序：首先运行程序1，因为在程序1中需要编写接收SIGUSR1信号函数，并得到父进程ID；然后运行程序2，程序2的作用是接收到程序1中父进程的ID后，确认发送信息对象，然后每隔2秒钟发送一个SIGUSR1信号给程序1中的父进程。
程序运行用Ctrl+C结束。
```c
#include <stdio.h>
#include <unistd.h>
#include <signal.h>
#include <stdbool.h>

void handler(int signum)
{
    printf("接收到SIGUSR1\n");
    return;
}

int main(void)
{
    printf("我的PID是%d\n", getpid());
    signal(SIGUSR1, handler);

    while (true)
        ;

    return 0;
}
```
```c
#include <stdio.h>
#include <stdlib.h>
#include <signal.h>
#include <unistd.h>
#include <sys/types.h>

#define ERR -1

int main() {
    pid_t pid;

    printf("输入程序1的PID：");
    scanf("%d", &pid);

    while (1) {
        if (kill(pid, SIGUSR1) == ERR) {
            perror("发送信号失败\n");
            return -1;
        }
        printf("发送成功\n");
        sleep(2);
    }

    return 0;
}
```
3、设计一个程序：要求主程序运行时，即使用户按下Ctrl+C键也不能影响正在运行的程序，即让信号处于阻塞状态；当主程序运行完毕后才进入自定义信号处理函数执行。要求：
(1)主程序：每2秒钟输出提示“当前程序处于信号阻塞状态！”，循环显示5次
(2)自定义信号处理函数：查看当前路径下的所有文件信息。
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <signal.h>
#include <dirent.h>
#include <stdbool.h>
#include <sys/types.h>

#define ERR -1

void handler(int signum)
{
    printf("接收到SIGINT\n");
    execl("/bin/ls", "ls", "-l", NULL);
    return;
}

int main(void)
{
    sigset_t set;
    sigemptyset(&set);

    sigaddset(&set, SIGINT);

    if (sigprocmask(SIG_BLOCK, &set, NULL) == ERR)
    {
        printf("屏蔽SIGINT信号失败");
        return -1;
    }

    signal(SIGINT, handler);

    for (int i = 0; i < 5; i++)
    {
        printf("当前程序处于阻塞状态！\n");
        sleep(2);
    }

    if (sigprocmask(SIG_UNBLOCK, &set, NULL) == ERR)
    {
        printf("解除信号屏蔽失败！\n");
        return -1;
    }
    printf("解除信号屏蔽成功！\n");

    while (true)
    {
        pause();
    }

    return 0;
}
```