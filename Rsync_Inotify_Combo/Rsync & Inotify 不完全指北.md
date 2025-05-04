# Rsync & Inotify 不完全指北

## Rsync

首先, 将双方配置密钥[否则同步需要输入密码]

两台主机node1@192.168.1.1 , node2@192.168.1.2 用户root 密码123456

## node1:

```ini
ssh-keygen -f ~/.ssh/id_rsa -t ed25519 # -f指定密钥保存位置 -t指定密钥类型 ed25519为推荐安全类型
# passphrase推荐设置,这是访问密钥文件的密码 [可以不设置]

ssh-copy-id 192.168.1.2
# 会问你是否继续连接,写入yes
# 填入node2的用户密码
# 配置完成
ssh 192.168.1.2 # 测试是否成功
```

## node2:

```ini
ssh-keygen -f ~/.ssh/id_rsa -t ed25519 # -f指定密钥保存位置 -t指定密钥类型 ed25519为推荐安全类型
# passphrase推荐设置,这是访问密钥文件的密码 [可以不设置]

ssh-copy-id 192.168.1.2
# 会问你是否继续连接,写入yes
# 填入node1的用户密码
# 配置完成
ssh 192.168.1.1 # 测试是否成功
```



同步双方双方均需要安装rsync:

```ini
dnf -y install rsync
```

检查安装情况:

```ini
rsync --version
```

------

## Rsync 基本语法

```ini
rsync [选项] 源路径... 目标路径
```

- `-v`：详细模式输出，显示传输过程中的文件信息。
- `-a`：归档模式，表示递归传输文件，并保持所有文件属性。
- `-z`：对备份的文件在传输时进行压缩处理。
- `-P`：显示进度条。
- `--delete`：删除目标目录中源目录中没有的文件（可选，用于实现双向同步）
- -e ：用于指定用于传输文件的远程 shell 程序  默认为 ssh，也可以选择其他如rsh或者自定义的ssh端口

## 常见搭配

```ini
本地同步：
rsync -av /src/directory/ /dest/directory/

远程同步：
rsync -avz /src/directory/ root@192.168.1.2:/dest/directory/

拉取远程数据到本地：
rsync -avz user@192.168.1.2:/src/directory/ /dest/directory/

镜像服务器(会保持一致,删除不存在的文件):
rsync -av --delete /usr/local/nginx-1.8.0/html /usr/local/nginx-1.8.0/backup/
```

------

## 同步实例

将node1本地的/test/sync/的内容同步到/backup/

```ini
rsync -av /test/sync/ /backup/
```

将node1的/test/sync/的内容同步到node2的/backup/中

```ini
rsync -avz /test/sync/ root@192.168.1.2:/backup/
```

将node2的/test/sync/的内容同步到本地node1的/backup/

```ini
rsync -avz root@192.168.1.2:/test/sync/ /backup/
```

需要镜像功能需要加上"--delete"参数

------

## Inotify

