# 开发文档

## 预期目标
1. 将Keystone核心代码用rust重写，实现在zCore上创建enclave
2. 增强enclave内runtime的能力，在enclave内运行更完整的内核以支持更高效的系统调用

## 2022.7.13
### Driver
- 重构创建enclave时U态和S态的调用规范，完成S态对于enclave写页表的操作，兼容zCore的GenericPageTable接口；

### sdk
- 重构创建的运行逻辑，删除之前对于页表和elf的处理，将这部分转移到S态，增加传入S态的参数（runtime和eapp的指针和大小）；

## 2022.6.23
### Driver
- 重构runtime和eapp的加载逻辑，将整个功能转移到host s态中执行，包括写页表，elf加载，并以此重构了epm的内存分配逻辑；

## 2022.6.3
### Overall
- 在使用原有sm的基础上完成enclave创建和销毁的测试，可以验证host和sdk的主体部分正确实现

### sm
- 修复sm层汇编段代码以及enclave平台相关代码

### Driver
- 修改内存管理的分配逻辑，将物理页帧的分配与vmo的创建解耦，以实现在连续内存块上的动态分配；

## 2022.5.27
### sm
- 已实现 sm 所用 opensbi 代码，化简 sm 层，删除多核部分代码
- 正在处理代码编译与依赖问题，尝试编译运行
### sdk
- 在上周问题的基础上排查到是因为keystone分配了一片连续的内存，但zCore的内存分配是遵循zircon标准的，其中一个检查禁止了连续内存上的mmap，禁用此检查后导致系统产生死锁，因此在不修改系统内核代码的前提下应该不能绕过这一问题，考虑的两种解决方案有
  - 放弃使用连续的内存，但这可能会导致sm在尝试隔离内存时遇到更多问题，以及潜在的tee os中的页表问题
  - 在初始化enclave时计算好所需的内存大小，直接分配适当大小的连续内存，由enclave调度，而不是反复使用mmap
- 处理了一些编码上的错误

## 2022.5.20
### Overall
将zCore整合到keystone的体系当中，包括：runtime的编译，c版本的sm的编译与运行，解决了诸多bug：
- device tree在进入zCore时无法识别，原因是sm修改了dtb的位置，然而这个位置也存在bug，则取消这个修改，将qemu传递的参数原封不动传给zCore；
- bootrom(bootloader)与sm的地址重合，无法简单地解决，于是将bootrom的功能扔进sm中；
- 重新整理了完整的编译流程，方便从0开始测试整个模块；

在整体可以编译的基础上，调试整个流程：
- 测试创建enclave的流程，调试bug，修改了若干之前疏忽留下的没有完全符合约定的调用；
- 发现了关于内存分配的新问题，还在进一步检查中，具体来讲keystone的设计是在enclave中自己维护了简单的页表，但这些内存的分配实现并不清晰，在我们的测试中出现了非法内存访问的情况，需要与host OS结合考虑如何处理好；

### sm
- 经过与学长和助教的讨论后，原本预计使用fw_dynamic模式将opensbi编译成动态库，再透过rust ffi调用opensbi，但是经过几天的研究，发现opensbi在fw_dynamic模式下仍无法编译成动态库
- 讨论后决定还是迁移到rustsbi上，正在研究rust-qemu实现，并修改原来sm调用opensbi代码

## 2022.5.13
### 
- 完整 sm 层的所有代码
- 添加大部分 sm 层所用到的 opensbi 接口
- 由于 sm 层所使用到的部分 opensbi 接口与 opensbi 内部实现深度耦合，无法转到rustsbi。 与学长讨论以后决定尝试使用 fw_dynamic 模式交叉编译 sm-rust 与 opensbi。

### sdk
- 在尝试了musl和gnu编译器，前者对于rust的支持不够好，后者无法支持std的静态链接；
- 决定使其完整支持no-std环境：解耦了之前的std支持，手写了命令行arg的读取；
- 修复了若干bug；

### zCore
- 进一步加强了理解，修复了out of memory的bug；

### Overall
- 研究发现runtime的编译耦合了c版本的sdk，sm也有与opensbi耦合的部分，因此接下来需要研究c和rust混合编译的问题；
- 根据现有的开发进度，决定就现有的sm来进行debug，而sm-rust进行同步开发；

## 2022.5.6
### sm
修复 sm 层 bug，已完成 sm 层功能；与洛佳学长讨论，由于 keystone 的 sm 层大量使用 opensbi 内部实现，与 opensbi 紧密耦合，无法直接换成 rustsbi，决定根据 sm 所用到功能自行实现在 opensbi 模块，目前已实现部分功能，正在进行中。
### Driver
- 增加mmap函数模块，使得上层调用时可以映射共享内存；
- 修改enclave管理的设计模式，仿照rcore的函数式编程完成了内部可变的全局管理器；
- 完整通过编译检查，修复了大量bug；
- 修改了部分接口约定；
### sdk
- 拆分了单独的git仓库，调整了仓库结构；
### Overall
- 将包含sdk的测试代码，runtime的仓库加入zCore的运行框架内，想要使得系统可以运行，还需要将sm整合进系统中；
- 
## 2022.5.4
### sdk
完成全部 sdk 移植工作，开始进行测试

## 2022.5.3
### sdk
由于没有现成的 riscv 的 elf 文件支持，单独写了 elf crate

## 2022.4.29
### sm
完成 attest, enclave, ipi, mprv 模块，添加 assert 模块，thread 模块修改，大致完成所有接口，剩余 pmp 模块待修改；开始看 rustsbi 实现，rustsbi-qemu，尝试加上缺少函数与功能（待完成）；


## 2022.4.22
### sm
根据加密算法构造模块crypto.rs，完成cpu、page、thread模块，修改之前的create_enclave及其相关函数错误，完成destroy_enclave移植、初步完成run_enclave、exit_enclave等函数；
### common
ed25519通过测试，完成可供sdk使用的模块，准备日后重构项目依赖；
### sdk
完成enclave app接口的移植，部分完成host app接口的移植，重新设计host app接口的实现（出于语言特性和整个项目的考虑）

## 2022.4.21
### sm
完成ed25519加密算法，用于生成私钥，进行签名；

## 2022.4.15
### Driver
主体代码移植完成，其主要功能是对于enclave的内存管理以及sbi call的传递，修改了一些调用规范以兼容zCore，需要之后开发sdk时再做讨论；

### sdk
移植edge call相关代码，梳理host app和enclave app所使用的接口

### sm
梳理sm层整体框架；
完成主要接口create_enclave函数的移植

## 2022.4.9
### 短期分工
- 孙宇涛：移植keystone-driver到rust上，并且植入zCore；
- 安一帆：研究U740启动多核的细节；
- 赖瀚宇：修改rustsbi，使得其可以兼容secruity monitor；
