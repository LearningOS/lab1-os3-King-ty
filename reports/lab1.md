# 简单总结你实现的功能（200字以内，不要贴代码）
1. 修改`TaskControlBlock`结构，添加`syscall_times`和`init_time`成员用于记录系统调用次数和应用时间
2. 修改`TASK_MANAGER`初始化，添加对`syscall_times`和`init_time`的初始化
3. 修改`TaskManager::run_next_task()`方法，判断应用第一次上台时更新中`init_time
4. 在`TaskManager`及`task/mod.rs`中添加`add_syscall_times`,`get_syscall_times`,`get_task_time`,`get_status`等接口
5. 在`syscall`函数中调用`add_syscall_times`更新`syscall_times`
6. 完成`sys_task_info`系统调用实现
7. 设置`entry.asm`中的`boot_stack`，调大一倍

# 完成问答题

1. 正确进入 U 态后，程序的特征还应有：使用 S 态特权指令，访问 S 态寄存器后会报错。 请同学们可以自行测试这些内容 (运行 Rust 三个 bad 测例 (ch2b_bad_*.rs) ， 注意在编译时至少需要指定 LOG=ERROR 才能观察到内核的报错信息) ， 描述程序出错行为，同时注意注明你使用的 sbi 及其版本。

   出错行为：

   报错并杀死应用

   ```
   Into Test store_fault, we will insert an invalid store operation...
   Kernel should kill this application!
   [kernel] PageFault in application, kernel killed it.
   
   Try to execute privileged instruction in U Mode
   Kernel should kill this application!
   [kernel] IllegalInstruction in application, kernel killed it.
   
   Try to access privileged CSR in U Mode
   Kernel should kill this application!
   [kernel] IllegalInstruction in application, kernel killed it.
   ```

   sbi版本：codespace里的版本

2. 深入理解 trap.S 中两个函数 __alltraps 和 __restore 的作用，并回答如下问题:
   1. L40：刚进入 __restore 时，a0 代表了什么值。请指出 __restore 的两种使用情景

      a0代表sp
      两种场景：1. U 态应用trap返回恢复trap上下文 和 2. 开始执行 U 态应用进入用户态。

   2. L46-L51：这几行汇编代码特殊处理了哪些寄存器？这些寄存器的的值对于进入用户态有何意义？请分别解释。

         ```assembly
         ld t0, 32*8(sp)
         ld t1, 33*8(sp)
         ld t2, 2*8(sp)
         csrw sstatus, t0
         csrw sepc, t1
         csrw sscratch, t2
         ```

         处理了t0,t1,t2,sstatus,sepc,sscratch寄存器。前三个是为了空出t0,t1,t2的值用来接收后三者。
         ssstatus：SPP 等字段给出 Trap 发生之前 CPU 处在哪个特权级（S/U）等信息
         sepc：当 Trap 是一个异常的时候，记录 Trap 发生之前执行的最后一条指令的地址
         scause：描述 Trap 的原因

   3. L53-L59：为何跳过了 x2 和 x4？

         ```assembly
         ld x1, 1*8(sp)
         ld x3, 3*8(sp)
         .set n, 5
         .rept 27
         	LOAD_GP %n
         	.set n, n+1
         .endr
         ```

         x2：sp(x2)寄存器，要基于它来找到每个寄存器应该被保存到的正确的位置，需特殊处理
         x4：tp(x4) 寄存器，除非我们手动出于一些特殊用途使用它，否则一般不会被用到

   4. L63：该指令之后，sp 和 sscratch 中的值分别有什么意义？

         ```assembly
         csrrw sp, sscratch, sp
         ```

         该指令交换了sp和sscratch寄存器的值，执行后sp是用户栈，sscratch是内核栈。

   5. __restore：中发生状态切换在哪一条指令？为何该指令执行之后会进入用户态？

          是`csrrw sp, sscratch, sp`指令，因为执行后sp指向用户栈，进入用户态。

   6. L13：该指令之后，sp 和 sscratch 中的值分别有什么意义？
      
         ```assembly
         csrrw sp, sscratch, sp
         ```
         
         该指令交换了sp和sscratch寄存器的值，执行后sp是内核栈，sscratch是用户栈。
         
   7. 从 U 态进入 S 态是哪一条指令发生的？

        `__alltraps`中的：
        
        ```assembly
        csrrw sp, sscratch, sp
        ```

