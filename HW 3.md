# HW 3

## 配置环境及基线运行

```shell
pku0008@pku0008-101230-5697b6dbd7-8598f:~/homework_3$ ./nbody
Simulator is calculating positions correctly.
4096 Bodies: average 0.078 Billion Interactions / second
pku0008@pku0008-101230-5697b6dbd7-8598f:~/homework_3$ ./nbody_GPU_basic
Simulator is calculating positions correctly.
4096 Bodies: average 32.018 Billion Interactions / second
pku0008@pku0008-101230-5697b6dbd7-8598f:~/homework_3$ ./nbody_GPU_shared
Simulator is calculating positions correctly.
4096 Bodies: average 199.254 Billion Interactions / second
```

## 性能分析与理解

- 哪几行代码的修改，使得`nbody_parallel.cu`比`01-nbody.cu`执行快？为什么？
  
  Answer: line39`bodyForce`和line68`integrate_position`两个CUDA核函数的实现使得可以调用kernel并行更新每个天体的速度和位置。在原本的cpu版本的代码当中，这两个都是通过for循环遍历每个天体实现的。

- `nbody_shared.cu`比`nbody_parallel.cu`执行快多少倍，为什么？
  
  Answer: 6.25倍。每一个天体对应一个线程，在计算所受重力时，这个线程都需要与其他线程的位置数据进行交互。如果每个线程从全局内存单独加载这些数据，就会产生大量冗余访问。所以先用shared memory存储这些数据。这样会节省很多访问时间。

- 哪几行代码的修改，使得`nbody_shared.cu`比`nbody_parallel.cu`执行快？为什么？
  
  Answer: line38-50，先用shared memory存储每个天体的数据。这样每个线程在访问其他天体的数据时速度会变快。

- `nbody_shared.cu`比`01-nbody.cu`执行快多少倍，为什么？
  
  Answer: 2500倍。CUDA核函数并行加速，同时访问shared memory也会加速。

## 参数分析与优化

- 分析`nbody_parallel.cu`中`BLOCK_SIZE`的取值对性能的影响。
- 分析`nbody_shared.cu`中`BLOCK_SIZE`、`BLOCK_STRIDE`的取值对性能的影响。

| 程序       | block size | block stride | GPU功率 | 运行时间   | 显存占用  |
| -------- | ---------- | ------------ | ----- | ------ | ----- |
| parallel | 32         |              | 173W  | 4.596s | 551MB |
| parallel | 128        |              | 271w  | 4.601s | 551MB |
| parallel | 512        |              | 301w  | 4.693s | 551MB |
| shared   | 32         | 4            | 280W  | 4.198s | 435MB |
| shared   | 128        | 4            | 271W  | 4.180s | 435MB |
| shared   | 512        | 4            | 268W  | 4.138s | 435MB |
| shared   | 128        | 1            | 275W  | 4.265  | 435MB |
| shared   | 128        | 2            | 265W  | 4.190  | 435MB |
| shared   | 128        | 8            | 265W  | 4.174  | 435MB |

这里对1<<18个天体进行计算。结论：shared速度普遍快于parallel；同一个程序无论如何调整参数占用显存相同；GPU功率对shared memory的参数影响不大，对普通并行程序parallel有很大影响。猜想因为寄存器资源少于共享内存资源，因而更容易受block size影响。
