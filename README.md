***AuditdMemCOMM 修改了[auditd的源代码](https://github.com/linux-audit/audit-userspace)，增加了采用共享内存的进程间通信，将系统日志进行重定向到处理进程，而不再必须写入log中再进行处理***

### Build notes:

1. 运行 `which auditd` 观察当前电脑上是否安装了auditd，如果没有则`sudo yum install audit audit-libs`进行安装, 再采用`which auditd` 观察所在路径
2. `git pull code` 拉取AuditdMemCOMM
3. `yum install autoconf automake libtool tcp_wrappers-devel openldap-devel`(build依赖库，无须重复安装)
4. `./configure --sbindir=xxx`   xxx为auditd路径， 本例中为`--sbindir=/sbin`
5. `make & make install`
6. `sudo auditctl -l` 观察是否已定义存在规则，没有的话先添加,如与`SPADE`一同运行，则无需配置
7. `cat /etc/sysconfig/selinux` 观察Selinux工作模式是否为**enforcing**, (centos7下默认配置为enforcing)，如是的话执行如下命令：
    ```
    $ checkmodule -M -m -o auditdwatch.mod auditdwatch.te
        checkmodule:  loading policy configuration from auditdwatch.te
        checkmodule:  policy configuration loaded
        checkmodule:  writing binary representation (version 17) to auditdwatch.mod
    $ semodule_package -o auditdwatch.pp -m auditdwatch.mod
    $ semodule -i auditdwatch.pp
    $ semodule -l | grep auditdwatch //查看模块是否安装成功
    ```
8. `service auditd status` 观察aduitd状态，如果为 running 状态则调用 `service auditd stop`
9. `service auditd start` 启动auditd,`service auditd stop` 关闭auditd，观察能否成功启动，卡死的话证明存在问题
10. 每次重启auditd时，需要手动关闭共享内存，执行命令：
    ```
    $  ipcs -m //查看key=0x000010f7的对应shmid,例如为131072 
    $  ipcrm shm shmid  //例如 ipcrm shm 131072 
    ```