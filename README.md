# XV6实验报告

## 2024年小学期：操作系统课程设计

2251556 张昊宸

软件工程 同济大学

## 1.实验环境配置

### 1.1 工具

1.下载`VMware 16 pro`虚拟机和`ubuntu-20.04.6`，为下载虚拟机，首先进入VM官网，找到对应之虚拟机后选择立即下载，得到VM安装包，点击安装包进行安装即可。安装完成后，下载需要的 Ubuntu 20.04 镜像。可以从国内一些高校的镜像站中进行下载。例如，清华大学镜像站提供 Ubuntu 20.04 的镜像，下载完成后即可得到安装包。

<img src="E:\课程设计\操作系统\实验报告\屏幕截图 2024-07-22 110026.png" alt="屏幕截图 2024-07-22 110026" style="zoom:50%;" />

2.在虚拟机中完成`ubuntu-20.04.6`之安装，打开下载好的VMware 16 pro，选择创建新的虚拟机，由此进入新建虚拟机向导界面并勾选自定义，在安装过程中注意：1.选择虚拟机工具版本Workstation 16.x，2.安装程序光盘映像文件时，从windows系统中找到安装好的Ubuntu20.04系统。3.自定义Ubuntu系统的账户信息（务必牢记），同时建议更改虚拟机安装位置。

<img src="E:\课程设计\操作系统\实验报告\屏幕截图 2024-07-22 111033.png" alt="屏幕截图 2024-07-22 111033" style="zoom:50%;" />

<img src="E:\课程设计\操作系统\实验报告\屏幕截图 2024-07-22 111053.png" alt="屏幕截图 2024-07-22 111053" style="zoom:50%;" />

### 1.2 软件源更新和环境准备

启动Ubuntu,安装本项目所需的所有软件，运行：

```
$ sudo apt-get update && sudo apt-get upgrade
$ sudo apt-get install git build-essential gdb-multiarch qemu-system-misc gcc-riscv64-linux-gnu binutils-riscv64-linux-gnu
```

测试安装

```
$ make qemu
$ riscv64-linux-gnu-gcc --version
```

结果应为：

![屏幕截图 2024-07-22 111749](E:\课程设计\操作系统\实验报告\屏幕截图 2024-07-22 111749.png)

### 1.3 编译内核

设置完成后，下载xv6内核源码：

```
$ git clone git://github.com/mit-pdos/xv6-riscv.git
```

更新镜像源：

```
$ sudo nano /etc/apt/sources.list
$ sudo apt-get update
```

## 2.Xv6 和 Unix 实用程序

首先，启动xv6，获取其源代码并查看util分支

```
$ git clone git://g.csail.mit.edu/xv6-labs-2021
$ cd xv6-labs-2021
$ git checkout util
```

<img src="E:\课程设计\操作系统\实验1\0101.png" alt="0101"  />

### 2.1 sleep

#### 实验目的

为 xv6 实现一个 UNIX 程序 `sleep`。该程序应该按照用户指定的 ticks 数暂停，其中 tick 是 xv6 内核定义的时间概念，即定时器芯片两次中断之间的时间。解决方案应在 `user/sleep.c` 文件中实现。

#### 实验步骤

1. **阅读和参考**：

   开始编码之前，阅读 xv6 book 的第 1 章。

   查看 `user/` 目录中的其他程序（例如 `user/echo.c`、`user/grep.c` 和 `user/rm.c`），了解如何获取传递给程序的命令行参数。

   在命令行中运行以下命令打开文件并查看其内容：

   ```
   $ nano user/echo.c
   ```

   - 使用任何文本编辑器打开文件，例如 Vim、Nano、Gedit 等。

   - 在 Nano 编辑器中打开文件后，要退出并返回终端命令行界面：

     1.按下 `Ctrl + X` 退出编辑器。

     2.如果对文件进行了修改，Nano 会提示您保存更改：

     - 按下 `Y` 键确认保存，或按下 `N` 键放弃更改。

     3.如果选择保存更改，Nano 会提示您输入文件名：

     - 按下 `Enter` 键确认保存到当前文件。

2. **参考实现**：

   - 通过 `kernel/sysproc.c` 中的 `sys_sleep` 获取实现 `sleep` 系统调用的 xv6 内核代码。
   - 通过 `user/user.h` 获取可从用户程序调用 `sleep` 的 C 语言定义。
   - 通过 `user/usys.S` 获取从用户代码跳转到内核以实现 `sleep` 的汇编代码。

3. **编写程序**：

   - 在程序中使用 `sleep` 系统调用。

   - 命令行参数以字符串形式传递，可以使用 `atoi`（参见 `user/ulib.c`）将其转换为整数。

   - 确保 `main` 调用 `exit()` 以退出程序。

   - 如果用户忘记传递参数，`sleep` 应打印错误信息。

   - 使用`nano`编辑器修改`sleep`程序：

     - 使用 `argc` 整数参数，其值是程序运行时命令行参数的数量加 1（1 表示可执行文件的名称本身也作为一个参数传递给程序）。通过检查 `argc` 的值，程序判断是否提供了足够的命令行参数。如果 `argc` 的值小于 2，说明没有传递足够的参数，程序输出错误消息并调用 `exit(1)` 终止程序的执行。

     - 命令行参数在 `argv` 数组中以字符串形式存储，并可以通过索引访问。`argv[0]` 存储的是可执行文件的名称，而 `argv[1]`、`argv[2]` 等存储的是传递给程序的其他命令行参数。

       ```c
       #include "kernel/types.h"
       #include "kernel/stat.h"
       #include "user/user.h"
       
       int main(int argc, char *argv[])
       {
         if (argc < 2)
         {
           fprintf(2, "need a param");
           exit(1);
         }
         int i = atoi(argv[1]);
         sleep(i);
         exit(0);
       }
       ```

4. **修改 Makefile**：

   - 将编写好的 `sleep` 程序添加到 Makefile 的 `UPROGS` 中。

   - 输入命令行 

     ```
     $ nano Makefile
     ```

      打开 Makefile 文件，在 Makefile 中找到名为 `UPROGS`的行，这是一个定义用户程序的变量。在 `UPROGS`行中，添加 `sleep` 程序的目标名称：

     ```
     $U/_sleep\
     ```

5. **编译和运行程序**：

   - 在终端中，运行 `make qemu` 命令编译 `xv6` 并启动虚拟机，随后通过测试程序来检测 `sleep` 程序的正确性。

     ![0102](E:\课程设计\操作系统\实验1\0102.png)

     

#### 实验问题与解决方法

在 `kernel/sysproc.c` 中，`sys_sleep` 函数是实现 `sleep` 系统调用的关键。该函数通过获取用户传递的 tick 数（通过 `argint` 函数获取），然后在一个循环中进行暂停。在每次循环中，它检查是否达到了指定的 tick 数，如果没有，则调用 `sleep` 函数将进程置为睡眠状态，并释放 `tickslock`。

因此，我需要在 `user/sleep.c` 中编写一个与 `sys_sleep` 对应的用户程序函数 `sleep`。该函数将负责向内核发起 `sleep` 系统调用，并将用户传递的 tick 数作为参数传递给内核。在传递参数时，起初出现了不匹配的现象，后来通过仔细阅读发现系统中有一个可以将命令行参数转换为整数的 `atoi` 函数。

为了使用 `sleep` 函数，用户程序需要包含一系列相关的头文件。在确定所需头文件的过程中，我通过阅读 `user/user.h` 等头文件并结合控制台报错信息，最终确定了需要在 `main` 函数中使用的头文件，从而成功调用 `sleep` 函数。

#### 心得体会

在实验中，我们需要弄清楚要编写的程序的功能，并阅读该程序相关的依赖文件，理清参数传递和头文件依赖关系，避免参数传递出错或缺少头文件等问题。

在编译并运行 `sleep` 程序之前，除了需要正确配置 xv6 环境，还需要确保系统支持并正确实现 `sleep` 系统调用，否则程序将无法被系统调用并运行测试。

### 2.2 pingpong

#### 实验目的

通过编写一个程序，了解 UNIX 系统的 ping-pong 方法：使用 UNIX 系统调用通过一对管道在两个进程之间“ping-pong”一个字节，每个管道对应一个方向。父进程应该向子进程发送一个字节；子进程应该打印": received ping"并显示其进程 ID，将管道上的字节写给父进程，然后退出；父进程应该从子进程读取字节，打印": received pong"后退出。解决方案应该在文件 `user/pingpong.c` 中实现。

目标是理解父进程和子进程的关系及其执行顺序，学习使用管道进行进程间通信，实现父进程和子进程之间的数据交换，并掌握进程同步的概念，确保父进程和子进程在适当的时机进行通信。

#### 实验步骤

1.查看库函数：在 `user/user.h` 中查看 xv6 上用户程序的一组库函数。在 `user/ulib.c`、`user/printf.c` 和 `user/umalloc.c` 中查看其他源代码（系统调用除外）。

2.使用 `pipe` 创建管道。

3.使用 `fork` 创建子进程。

4.使用 `read` 从管道中读取数据，使用 `write` 向管道中写入数据。

5.使用 `getpid` 查找调用进程的进程 ID。

6.将程序添加到 Makefile 的 `UPROGS` 中。

7.运行并测试程序，如果程序在两个进程之间交换一个字节并产生上述输出，说明程序正确。

![0104](E:\课程设计\操作系统\实验1\0104.png)

```c
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"

int main(int argc, char *argv[])
{
  int parent_fd[2]; 	// File Descriptors for Storage Pipes
  int child_fd[2];
  pipe(parent_fd);	// Calling pipe creates a parent pipe
  pipe(child_fd);	// Calling pipe creates a child pipe

  // Creating child processes with fork
  if (fork() == 0)
  {
    char buf[10];			// For storing read data
    read(parent_fd[0], buf, sizeof buf); // Read data from parent_fd[0], read data stored in buf
    int id = getpid();			// Get the ID of the current process and print it
    printf("%d: received %s\n", id, buf);
    write(child_fd[1], "pong", 4);	// Write string "pong" to child_fd[1] pipe
    // Close the read side and write side of child_fd.
    close(child_fd[0]);
    close(child_fd[1]);
  }
  else
  {
    char buf[10];
    int id = getpid();
    write(parent_fd[1], "ping", 4);	
    // Write the string "ping" to the parent_fd[1] pipe and close the write side of that pipe
    close(parent_fd[1]);
    int status;
    wait(&status); 
    read(child_fd[0], buf, sizeof buf);
    printf("%d: received %s\n", id, buf);
    close(child_fd[0]);
  }

  exit(0);
}
```

#### 实验问题与解决方法

实验中有一段要求：父进程向子进程发送一个字节，随后子进程打印": received ping"，将管道字节写入父进程后退出，父进程再从子进程读取字节，打印": received pong"后退出。起初，我忽视了这一段要求中的逻辑，导致程序编写不够完善。这实际上对父子进程的先后逻辑关系做出了要求。因此，我在父进程中使用了 `wait()` 函数，等待子进程结束，并获取子进程的退出状态。通过这种同步机制，确保父进程在子进程完成之后继续执行。

如果在父进程中不使用 `wait()` 函数，尽管本次的测试结果与不使用 `wait()` 函数的结果相一致，但父进程可能会在子进程执行完成之前继续执行自己的代码。这可能会导致父进程在子进程还没有完成时就退出，从而使子进程成为孤儿进程（没有父进程的进程）。此外，没有正确等待子进程完成的父进程可能无法获取子进程的退出状态，也无法做进一步的处理。

#### 心得体会

在本次实验中，我成功编写了 `pingpong.c` 程序，通过父进程和子进程之间的管道实现了数据的交换。在实验过程中，我遇到了一些挑战，也收获了以下心得：

1. **进程间通信的重要性**： 本次实验让我认识到进程间通信在多进程编程中的重要性。使用管道作为通信机制，可以在父进程和子进程之间传递数据，实现数据的共享和交换。
2. **进程同步的关键**： 实现正确的进程同步是实验中的关键。通过适当的管道读写操作和进程等待机制（如使用 `wait()` 函数），我成功实现了父进程和子进程的同步，确保了数据的正确交换和打印顺序。
3. **熟悉管道的使用**： 通过编写 `pingpong` 程序，我更加熟悉了管道的使用。我学会了创建管道、通过文件描述符进行读写操作，以及如何关闭管道的读写端。
4. **父子进程关系的理解**： 通过观察和分析父进程和子进程的输出顺序，我对父子进程之间的关系有了更深入的理解。我了解到子进程是由父进程派生出来的，它们共享某些资源，并在不同的代码路径中执行。
5. **调试和错误处理的重要性**： 在实验过程中，我遇到了一些编程错误和逻辑问题。通过仔细分析错误信息、调试和追踪程序执行流程，我能够及时发现问题并进行修复。这个过程提醒我在编程中注重错误处理和调试能力的重要性。

### 2.3 primes

#### 实验目的

通过编写一个基本筛选器的并发版本，学习使用管道。该想法来自于 Unix 管道的发明者 Doug McIlroy。解决方案位于 `user/primes.c` 中。

学习使用 `pipe` 和 `fork` 来设置管道。第一个进程将数字 2 到 35 输入管道。对于每个素数创建一个进程，该进程通过一个管道从左边的邻居读取数据，并通过另一个管道向右边的邻居写入数据。由于 xv6 的文件描述符和进程数量有限，第一个进程可以在 35 处停止。

目标是理解并掌握进程间通信的概念和机制，并学习使用管道在父子进程之间进行数据传递。

#### 实验步骤

1.创建父进程，父进程将数字 2 到 35 输入管道，此时不必创建其后所有进程。每一步迭代更新一对相对的父子进程（仅在需要时才创建管道中的进程）。

2.创建素数进程，对于 2-35 中的每个素数创建一个进程，进程之间需要进行数据传递：该进程通过一个管道从左边的父进程读取数据，并通过另一个管道向右边子进程写入数据。

3.处理数据，对于每一个生成的进程而言，当前进程最顶部的数即为素数。对每个进程中剩下的数进行检查，如果是素数则保留并写入下一进程，如果不是素数则跳过。

4.管理文件描述符，完成数据传递或更新时，需要及时关闭一个进程不需要的文件描述符，防止程序在父进程到达 35 之前耗尽 xv6 的资源。

5.等待子进程结束，在数据传递的过程中，父进程需要等待子进程的结束，并回收共享的资源和数据等。一旦第一个进程到达 35，它应该等待直到整个管道终止，包括所有子进程、孙进程等。因此，主 `primes` 进程应该在所有输出都打印完毕，并且所有其他 `primes` 进程都退出后才退出。

6.编写 `primes.c`，按照上述逻辑完成 `primes.c` 的编写。

7.在 Makefile 中将程序添加到 `UPROGS` 中。

8.编译运行并测试 `primes.c`，确保程序按预期工作。

![0105](E:\课程设计\操作系统\实验1\0105.png)

```c
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"

int main(int argc, char *argv[])
{
  printf("primes has been called \n");

  int first, v;
  // 用于在父进程和子进程之间交换管道文件描述符。
  int *fd1;
  int *fd2;
  // 创建第一个管道和之后的管道
  int first_fd[2];
  int second_fd[2];

  pipe(first_fd);

  // 创建父进程，将数字2-35输入管道
  if (fork() > 0)
  {
    for (int i = 2; i <= 35; i++)
      write(first_fd[1], &i, sizeof(i));
    // 输入后关闭写
    close(first_fd[1]);
    // 等待子进程结束
    int status;
    wait(&status);
  }
  // 对于子进程，循环处理，为素数创建新进程
  else
  {
    // 子进程从 first_fd 管道中读取数据，并创建一个新的管道 fd2
    fd1 = first_fd;
    fd2 = second_fd;

    while (1)
    {
      // 创建一个管道，为下一次读取做准备。
      pipe(fd2);
      // 子进程关闭了前一个管道的写端，从管道 fd1 中读取第一个值(素数)并打印
      close(fd1[1]);
      if (read(fd1[0], &first, sizeof(first)))
        printf("prime %d\n", first);
      else
        break;
      // 子进程遍历读取 fd1 管道的剩余值，对每个值进行判断
      if (fork() > 0)
      {
        int i = 0;
        while (read(fd1[0], &v, sizeof(v)))
        {
          // 如果可以整除 first（当前素数），则跳过该值，继续下一个
          if (v % first == 0)
            continue;
          i++;
          // 如果不可以整除，则将该值写入 fd2 管道中，以供下一个子进程使用。
          write(fd2[1], &v, sizeof(v));
        }
        // 关闭fd1的读端和fd2的写端，并等待子进程结束
        close(fd1[0]);
        close(fd2[1]);
        int status;
    	wait(&status);
        break;
      }
      // 子进程之间数据的传递
      else
      {
        close(fd1[0]);//close fd1 read
        int *tmp = fd1;
        fd1 = fd2;
        fd2 = tmp;
      }
    }

  }
  //父进程在子进程结束后退出
  exit(0);
}
```

#### 实验问题与解决方法

起初，我对父子进程的逻辑处理和数据传递感到疑惑。后来，我深入理解了 `fork()` 函数系统调用。`fork()` 用于创建一个新的进程（子进程）作为当前进程（父进程）的副本。子进程会继承父进程的代码、数据、堆栈和文件描述符等资源的副本。子进程和父进程在 `fork()` 调用点之后的代码是独立执行的，并且拥有各自独立的地址空间。因此，父进程和子进程可以在 `fork()` 后继续执行不同的逻辑，实现并行或分支的程序控制流程。

为了实现数据传递，可以在 `fork()` 判定为子进程的分支上进行数据“交换”，将子进程变为下一级的父进程，从而实现了数据传递。这让我解决了程序的逻辑问题，并更好地理解了父子进程的关系及其相对关系。

#### 心得体会

在这个实验中，我成功地编写了一个素数查找程序，利用管道实现了基本并发筛选器功能，并使用了管道进行父子进程之间的通信。通过编写这个程序，我深入理解了 `fork()` 函数的作用和使用方法，以及管道的基本原理和用法。

- `fork()` 函数的返回值

  对于父进程，`fork()` 函数返回子进程的进程 ID（PID），即大于 0 的整数值。

  对于子进程，`fork()` 函数返回 0。

  如果创建子进程失败，`fork()` 函数返回一个负值，表示创建进程的失败。

在编写过程中，我遇到了一些问题，比如如何处理管道的读写端关闭以及父子进程关系的处理。但通过调试和查阅资料，我解决了这些问题。通过这次实验，我对操作系统中进程的创建和通信有了更深入的了解，并且对并发编程有了更好的理解。

### 2.4 find

#### 实验目的

学习并编写一个简单版本的 UNIX 查找程序，实现在目录树中查找带有特定名称的所有文件。解决方案位于文件 `user/find.c` 中。目标是理解文件系统中目录和文件的基本概念和组织结构，熟悉在 xv6 操作系统中使用系统调用和文件系统接口进行文件查找操作，并应用递归算法实现在目录树中查找特定文件。

#### 实验步骤

1. **查看 `user/ls.c`**：
   - 了解如何读取目录。`user/ls.c` 中包含一个 `fmtname` 函数，用于格式化文件名称。它通过查找路径中最后一个 '/' 后的第一个字符来获取文件的名称部分。如果名称的长度大于等于 `DIRSIZ`，则直接返回名称。否则，将名称拷贝到一个静态字符数组 `buf` 中，并用空格填充剩余的空间，保证输出的名称长度为 `DIRSIZ`。
2. **编写 `main` 函数**：
   - 完成参数检查和功能函数的调用。检查命令行参数的数量，如果参数数量小于 3，则输出提示信息并退出程序。否则，将第一个参数作为路径，第二个参数作为要查找的文件名称，调用 `find` 函数进行查找，并最后退出程序。
3. **编写 `match` 函数**：
   - 用于检查给定的路径和名称是否匹配。通过查找路径中最后一个 '/' 后的第一个字符来获取文件的名称部分，然后与给定的名称进行比较。如果匹配，则返回 1，否则返回 0。
4. **编写 `find` 函数**：
   - 用于递归地在给定的目录中查找带有特定名称的文件。首先打开目录，并使用 fstat 函数获取目录的信息。根据目录的类型进行不同的处理：
     - 如果是文件（`T_FILE`），则使用 `match` 函数检查文件名称是否与给定的名称匹配，如果匹配则打印文件的路径。
     - 如果是目录（`T_DIR`），则遍历目录中的每个文件。首先，检查拼接后的路径长度是否超过了缓冲区 `buf` 的大小。如果超过了，则输出路径过长的提示信息。然后，将路径拷贝到 `buf` 中，并在路径后添加 '/'。接下来，循环读取目录中的每个文件信息，注意不要递归到 '.' 和'..'，跳过当前目录 '.' 和上级目录 '..'。将文件名拷贝到 `buf` 中，并使用递归调用 `find` 函数在子目录中进行进一步查找。（比较时要注意： == 不能像 Python 那样比较字符串，使用 `strcmp()` 代替）
   - 关闭目录文件描述符。

**函数功能**

- **`match`**：判断文件名是否与指定名称匹配
- **`fmtname`**：格式化输出文件名称
- **`find`**：通过打开目录、读取目录内容以及递归调用自身实现对目录树的遍历
- **`main`**：对命令行参数进行检查，并调用 `find` 函数进行查找

在 Makefile 中将程序添加到 `UPROGS` 中。

编译运行并测试 `find.c`。运行中需要注意，对文件系统的修改会在运行 qemu 时持续存在。要获得一个干净的文件系统，运行 `make clean`，然后再运行 `make qemu`。

![0106](E:\课程设计\操作系统\实验1\0106.png)

```c
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"
#include "kernel/fs.h"

int match(char *path, char *name)
{
    char *p;

    // 查找最后一个'/'后的第一个字符
    for (p = path + strlen(path); p >= path && *p != '/'; p--)
        ;
    p++;
    
    if (strcmp(p, name) == 0)
        return 1;
    else
        return 0;

}

void find(char *path, char *name)
{
    char buf[512], *p;
    int fd;
    struct dirent de;
    struct stat st;

    if ((fd = open(path, 0)) < 0)
    {
        fprintf(2, "ls: cannot open %s\n", path);
        return;
    }
    
    if (fstat(fd, &st) < 0)
    {
        fprintf(2, "ls: cannot stat %s\n", path);
        close(fd);
        return;
    }
    
    switch (st.type)
    {
    case T_FILE:
        if (match(path, name))
        {
            printf("%s\n", path);
        }
        break;
    
    case T_DIR:
        if (strlen(path) + 1 + DIRSIZ + 1 > sizeof buf)
        {
            printf("ls: path too long\n");
            break;
        }
        strcpy(buf, path);
        p = buf + strlen(buf);
        // 给path后面添加 /
        *p++ = '/';
        while (read(fd, &de, sizeof(de)) == sizeof(de))
        {
            // 如果读取成功, 一直都会在while loop中
            if (de.inum == 0)
                continue;
            if (strcmp(de.name, ".") == 0 || strcmp(de.name, "..") == 0)
                continue;
            // 把 de.name 拷贝到p中
            memmove(p, de.name, DIRSIZ);
            p[DIRSIZ] = 0;
            find(buf, name);
        }
        break;
    }
    close(fd);

}

int main(int argc, char *argv[])
{
    if (argc < 3)
    {
        printf("argc is %d and it is less then 2\n", argc);
        exit(1);
    }
    find(argv[1], argv[2]);
    exit(0);
}
```

#### 实验问题与解决方法

在实验过程中，我参考了目录相关操作的程序的写法，但在编写 `find` 函数时遇到了一些问题：

1. **输出格式问题**：
   - 在编写 `find` 函数时，我参考了 `ls.c` 中的 `ls` 函数。`ls.c` 主要用于列出指定路径下的文件和目录信息，其输出格式与本实验需求不同。这个问题起初我并未注意到。
2. **输入参数问题**：
   - `ls.c` 的输入仅需要一个文件地址参数，而文件查找需要路径 `path` 和查找文件名 `name` 两个参数。由于测试中使用了 `find . b` 和 `find a b`，都提供了两个参数，因此函数的输入也需要做出相应的修改。
3. **递归遍历问题**：
   - `ls.c` 程序只能提供基本的文件和目录信息，并不包含递归遍历子目录的功能。如果不对 `find` 函数进行递归遍历，它只能查找指定目录下的直接子文件和子目录，而无法继续向下递归地查找子目录中的文件，这也会导致在测试时无法得到全面的查找结果。

#### 心得体会

**文件系统理解**：

- 通过编写 `find.c` 程序，我深入理解了文件系统中目录和文件的关系，以及如何通过系统调用和文件系统接口来访问和操作文件。

**递归算法应用**：

- 在编写 `find.c` 过程中，我学会了使用递归算法实现对目录树的深度遍历，以便在整个目录结构中查找符合条件的文件。

**文件操作技巧**：

- 通过阅读相似功能的源代码，我进一步掌握了在 xv6 操作系统中进行文件操作的技巧和方法，如打开文件、读取目录等。

**问题解决与提升**：

- 实验过程中遇到了一些问题，例如理解文件系统的目录结构和文件的属性，以及如何正确处理文件路径等。通过查阅文档和调试代码，我逐渐解决了这些问题，并提高了对文件系统的理解和应用能力。

### 2.5 xargs

#### 实验目的

编写一个UNIX xargs程序的简单版本，该程序从标准输入中读取行，并为每一行运行一个命令，将行作为参数提供给命令。解决方案应当在 `user/xargs.c` 文件中实现。

- 熟悉命令行参数的获取和处理：实验需要读取命令行参数并进行适当的处理，包括选项解析和参数拆分。
- 学习外部命令的执行：实验中需要调用 `exec` 函数来执行外部命令，理解执行外部程序的基本原理。

#### 实验步骤

**1.理解 xargs 的工作原理**：

- 读取用户输入：读取单行输入，每次读取一个字符，直到遇到换行符（`\n`）。
- 将输入的字符串按单词拆分出参数：利用 `kernel/param.h` 声明的 `MAXARG`，用于检查声明的数组中参数个数是否超出限制。

**2.使用 fork 和 exec 调用命令**：

- 在父进程中使用 `wait` 等待子进程完成命令。
- 在子进程中调用 `exec` 执行命令时，将每个参数逐个提供给 `exec` 函数，以符合实验要求，即不一次性向命令提供多个参数，而是每次提供一个参数。

**3.将程序添加到 Makefile 中的 UPROGS**：

- 更新 Makefile，确保 `xargs` 程序能被正确编译和链接。

**4.编译运行并测试 xargs.c**：

- 运行中需要注意，对文件系统的修改会在运行 `qemu` 时持续存在；要获得一个干净的文件系统，运行 `make clean`，然后再运行 `make qemu`。

![0107](E:\课程设计\操作系统\实验1\0107.png)

```c
#include "kernel/types.h"
#include "kernel/stat.h"
#include "kernel/param.h"
#include "user/user.h"

int main(int argc, char *argv[])
{
    int argv_len = 0;// 参数个数
    char buf[128] = {'\0'}; // 存储用户输入字符串
    char *new_argv[MAXARG];

    // 检查参数个数是否超过了限制
    if (argc > MAXARG)
    {
        printf("Too many arguments\n");
        exit(1);
    }
    
    // 把argv的内容拷贝到new_argv
    for (int i = 1; i < argc; i++)
    {
        new_argv[i - 1] = argv[i];
    }
    
    // 循环读取用户的输入
    while (gets(buf, sizeof(buf)))
    {
        int buf_len = strlen(buf);  // 获取读取到的字符串的长度
        // printf("buf length is %d\n", buf_len);
        if (buf_len < 1)
            break;
        argv_len = argc - 1;
        buf[buf_len - 1] = '\0'; // 将读取到的字符串中的换行符替换为字符串结束符
    
        // 把buf中读取到用户输入的内容按照word拆分到每个new_argv中
        for (char *p = buf; *p; p++)
        {
            // printf("in the loop, p is %s\n", p);
            // 跳过连续的空格字符，p指向第一个非空格字符
            while (*p && (*p == ' '))
            {
                *p++ = '\0';
            }
            // 新参数
            if (*p)
            {
                // 检查参数个数是否超过了限制
                if (argv_len >= MAXARG - 1)
                {
                    printf("Too many arguments\n");
                    exit(1);
                }
                new_argv[argv_len++] = p;
            }
            // 跳过当前参数剩余字符，p指向下一个空格字符或字符串结束符
            while (*p && (*p != ' '))
            {
                p++;
            }
        }
    
        // 终止string
        new_argv[argv_len] = "\0";
    
        // 父进程，需要等待子进程结束
        if (fork() > 0)
        {
            int status;
            wait(&status);
        }
        else
        {
            // printf("exec start: \n");
            // 修改exec函数的调用方式，传递可执行文件路径作为第一个参数
            exec(new_argv[0], new_argv);
        }
    }
    
    exit(0);

}
```

#### 实验问题与解决方法

在实验中，输出结果不一致时，我遇到了一些问题。由于输出结果包含三行“hello”，且测试程序的透明性使得定位错误变得困难。因此，我添加了许多用于检查运行状况的命令行提示文字输出，例如：

- `printf("buf length is %d\n", buf_len);` 用于检查当前字符串长度。
- `printf("in the loop, p is %s\n", p);` 用于检查当前参数内容。
- `printf("exec start: \n");` 用于确保 `exec` 传参正确开始执行。

这些提示帮助我更清晰地了解代码的运行逻辑。

#### 心得体会

**参数处理**：实验要求将输入按照空格拆分为多个参数，并将它们作为命令行参数传递给外部命令。我学会了如何处理命令行中的输入字符串，跳过空格，并将参数存储在适当的数据结构中。

**外部命令执行**：通过调用 `exec` 函数执行外部命令，我深入了解了进程创建和替换的过程。我了解了如何在子进程中执行外部程序，并将程序路径和参数传递给 `exec` 函数。

**错误处理**：在编写实验代码时，我学会了处理错误情况并进行适当的错误报告。例如，当参数个数超过限制或无法执行外部命令时，我通过输出错误信息和退出程序来处理这些情况。
