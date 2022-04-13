# 开发文档

## 预期目标
在U740开发板上运行完整的TEE环境，在大核上启动TEE Enclave运行在小核上。

## 2022.4.13
### 修改目标
考虑到host上的环境较为依赖linux，因此调整目前的短期目标为：
- 在host上执行linux环境，移植rustsbi与rust runtime跑通一个完整的TEE系统
- 如果后期时间充裕，再将host环境移植到zCore上；
### 修改短期分工
- 孙宇涛，赖瀚宇：修改rustsbi，使得其可以兼容secruity monitor；
- 安一帆：移植enclave runtime与enclave sdk；

## 2022.4.9
### 短期分工
- 孙宇涛：移植keystone-driver到rust上，并且植入zCore；
- 安一帆：研究U740启动多核的细节；
- 赖瀚宇：修改rustsbi，使得其可以兼容secruity monitor；
