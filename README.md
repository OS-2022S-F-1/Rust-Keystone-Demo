# 开发文档

## 预期目标
1. 将Keystone核心代码用rust重写，实现在zCore上创建enclave
2. 增强enclave内runtime的能力，在enclave内运行更完整的内核以支持更高效的系统调用

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
