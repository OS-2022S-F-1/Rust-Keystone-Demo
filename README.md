# 开发文档

## 预期目标
在U740开发板上运行完整的TEE环境，在大核上利用zCore启动TEE Enclave运行在小核上

## 2021.4.9
### 短期分工
- 孙宇涛：移植keystone-driver到rust上，并且植入zCore；
- 安一帆：研究U740启动多核的细节；
- 赖瀚宇：修改rustsbi，使得其可以兼容secruity monitor；
