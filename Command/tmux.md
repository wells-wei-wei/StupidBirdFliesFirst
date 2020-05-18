<!--
 * @Author: your name
 * @Date: 2020-05-18 15:34:37
 * @LastEditTime: 2020-05-18 15:39:26
 * @LastEditors: Please set LastEditors
 * @Description: In User Settings Edit
 * @FilePath: \undefinedc:\Users\conan\Desktop\tmux.md
--> 
# tmux常用命令
## 查看所有会话
```
tmux list-session 
tmux ls
```

## 新建会话
```
tmux # 新建一个无名称的会话
tmux new -s demo # 新建一个名称为demo的会话
```

## 进入之前的会话
```
tmux a # 默认进入第一个会话
tmux a -t demo # 进入到名称为demo的会话
```

## 退出会话
```
exit # 退出并删除当前会话
tmux detach # 断开当前会话，会话在后台运行
```

## 删除会话
```
tmux kill-session -t demo # 关闭demo会话
tmux kill-server # 关闭服务器，所有的会话都将关闭
```