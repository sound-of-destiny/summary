apue

- int signal(SIGINT, sig_int) : 指明信号处理函数 sig_int
- sysconf(), pathconf(), fpathconf() : 打印系统的各种限制对应的值
- 硬链接 : 硬链接直接指向文件的 i 节点
- 符号链接 : 引入符号链接的原因是为了避开硬链接的一些限制
  - 硬链接通常要求链接和文件位于同一文件系统中
  - 只有超级用户才能创建指向目录的硬链接