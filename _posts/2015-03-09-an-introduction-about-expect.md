---
title: Expect脚本学习
category: linux
tags:
- expect
---

## 介绍

1. shell脚本可以使用here文档实现某些交互，但无法完成需要用户手动去输入的交互（如passwd、scp）。expect由此而生，可以轻松的完成此类交互。

2. expect是一种可以提供“分支和嵌套结构”来引导程序流程的解释型脚本语言，主要运用于自动交互，能够按照脚本内容里面设定的方式与交互式程序进行交互。

3. expect可以根据程序的不同提示或反馈内容来进行不同的应答，并且可随时在需要的时候把控制权交给用户，然后再还给脚本。

4. expect脚本是基于TCL（Tool Command Language）演变的，其语法与TCL语法类似。

<!--more-->

## 例子

### example1

```
check_passwd()
{
        local client_ip=$1

        # use timeout to handle the wrong passwd situation, because in the old solution:
        #       expect {
        #               *Permission* {exit 1}
        #               ]* { send exit\r }
        #       } ;
        # In case when the password is wrong but *Permission* wasn't printed out in time, 'exit 1' won't be executed and this expect cmd will exit normally,
        # making this password check process fail to function
        #
        # in the below new solution:
        # if passwd is wrong, output will be "Permission denied, please try again.", and "]*" can't be matched, expect will stay waiting there
        # and timeout will be triggered after 30 sec, and expect will exit with code 1
        expect -c "set timeout 30
                           spawn ssh ${USER_NAME}@${client_ip}
                           expect {
                                           *yes/no { send \"yes\r\"; exp_continue }
                                           *assword* { send \"${USER_PASSWD}\r\" }
                                           ]*    { send \" \r\" };
                           } ;
                           expect {
                                   timeout { exit 1 }
                                   ]* { send exit\r }
                           }
                                expect eof ;
                           "  >/dev/null
}
```

### example2

```
check_disk_space()
{
        local client_ip=$1
        local status

        # $base_dir_fs and $tmp_dir_fs refers to the Filesystem of ${BASE_DIR} and ${TMP_SRC_DIR}, such as /dev/mapper/centos-root
        expect -c "set timeout -1
                        spawn ssh ${USER_NAME}@${client_ip}
                        expect {
                                *yes/no { send \"yes\r\"; exp_continue }
                                *assword* { send \"${USER_PASSWD}\r\" }
                                ]*       { send \" \r\" };
                        } ;

                        expect ]* { send \" mkdir -p ${BASE_DIR}; echo RSTATUS:\$?\r \" };
                        expect -re \"RSTATUS:(\[0-9\]+)\" ;
                        if { \$expect_out(1,string) != 0 } {
                                exit 99
                        }

                        expect ]* { send \" stdbuf -o0 df -P ${BASE_DIR} | stdbuf -o0 tail -1 | awk '{print \\\$1, \\\$4; fflush()}'\r \" }
                        expect -re \"fflush.*\r\n(.*) (.*)\r\n\" ;
                        set base_dir_fs \$expect_out(1,string)
                        set base_dir_space_KB \$expect_out(2,string)

                        expect ]* { send \" mkdir -p ${TMP_SRC_DIR}\r \" };
                        expect ]* { send \" stdbuf -o0 df -P ${TMP_SRC_DIR} | stdbuf -o0 tail -1 | awk '{print \\\$1, \\\$4; fflush()}'\r \" }
                        expect -re \"fflush.*\r\n(.*) (.*)\r\n\" ;
                        set tmp_dir_fs \$expect_out(1,string)
                        set tmp_dir_space_KB \$expect_out(2,string)

                        if { \"\$base_dir_fs\" == \"\$tmp_dir_fs\" } {
                                if { \$base_dir_space_KB < 10 * 1024 * 1024 } {
                                        puts \" ERROR: ... \"
                                        puts \" due to install dir and temp dir use the same partition, at least 10GB space be needed \"
                                        puts \" currently only \$base_dir_fs KB is free \"
                                        exit 1
                                }
                        } else {
                                if { \$base_dir_space_KB < 6 * 1024 * 1024 || \$tmp_dir_space_KB < 4 * 1024 * 1024 } {
                                        puts \" ERROR: ... \"
                                        puts \" install dir needs 6GB space, temp dir needs 4GB space \"
                                        puts \" currently install dir: \$base_dir_space_KB KB, temp dir: \$tmp_dir_space_KB KB \"
                                        exit 1
                                }
                        }

                        expect ]* { send \" exit\r \" };
                        expect eof ;
                        " >/dev/null
```


### example3

```
copy_source()
{
        local client_ip=$1

        expect -c "set timeout -1
                           spawn ssh ${USER_NAME}@${client_ip}
                           expect {
                                          *yes/no { send \"yes\r\"; exp_continue }
                                          *assword* { send \"${USER_PASSWD}\r\" }
                                          ]*    { send \" \r\" };
                           } ;
                           expect ]* { send \" rm -fr ${SKYDISCOVERY_DIR}\r \" };
                           expect ]* { send exit\r };
                           expect eof ;
                           " >/dev/null

        # to fix wildcard in expect script doesn't work
        # http://stackoverflow.com/questions/11092667/wildcard-in-expect-script-doesnt-work
        expect -c "set timeout -1
                           spawn bash -c \"scp -P 22 -r ${CLUSTER_SRC}/* ${USER_NAME}@${client_ip}:${TMP_SRC_DIR}\";
                           expect {
                                          *yes/no { send \"yes\r\"; exp_continue }
                                          *assword* { send \"${USER_PASSWD}\r\"; exp_continue }
                                          expect eof
                           } ;
                           " >/dev/null

        return 0
}

```

### example4

```
install()
{
        local client_ip=$1

        expect -c "set timeout -1
                           spawn ssh ${USER_NAME}@${client_ip}
                           expect {
                                          *yes/no { send \"yes\r\"; exp_continue }
                                          *assword* { send \"${USER_PASSWD}\r\" }
                                          ]*    { send \" \r\" };
                           } ;

                           expect ]* { send \" echo ${SKYDISCOVERY_DIR} > ~/.SkyDiscovery-Installation-Location\r \" }

                           if { \"${TYPE}\" != \"\" } {
                                  expect ]* { send \" ${TMP_SRC_DIR}/SkyDiscovery-spark -b -t ${TYPE} -p ${SKYDISCOVERY_DIR}; echo RSTATUS:\$?\r \" };

                                          expect -re \"RSTATUS:(\[0-9\]+)\";
                                          if { \$expect_out(1,string) != 0 } {
                                                  exit 99
                                          }
                           } else {
                                      expect ]* { send \" ${TMP_SRC_DIR}/SkyDiscovery-spark -b -p ${SKYDISCOVERY_DIR}; echo RSTATUS:\$?\r \" };

                                          expect -re \"RSTATUS:(\[0-9\]+)\";
                                          if { \$expect_out(1,string) != 0 } {
                                                  exit 99
                                          }
                           }

                           expect ]* { send exit\r };
                           expect eof;
                           "

        status="$?"
        if [[ "$status" -eq 99 ]]; then
                log_error "install SkyDiscovery-spark failed."
                return "$status"
        fi

        return 0
}
```



完整文章请转到下面link:
[Expect脚本学习](https://github.com/kulong0105/kulong0105.github.io/blob/master/documents/Expect%E8%84%9A%E6%9C%AC%E5%AD%A6%E4%B9%A0.pdf)

[下载链接](https://github.com/kulong0105/kulong0105.github.io/raw/master/documents/Expect%E8%84%9A%E6%9C%AC%E5%AD%A6%E4%B9%A0.pdf)
