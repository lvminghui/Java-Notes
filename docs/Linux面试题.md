# Linux面试题

如今程序员生产的代码99%都部署在linux环境下，代码发现缺陷，程序员的第一反应是到 Linux 上拉日志下来看。程序员不是运维，不需要掌握非常多复杂命令。 

# 推荐终端工具

- SecureCRT
- xshell

## 常用命令（重要）

ls/ll、cd、mkdir、rm-rf、cp、mv、ps -ef | grep xxx、kill、free-m、tar -xvf file.tar 

**查看进程：**（例：如何查看所有xx进程）

　ps -ef | grep xxx

​	ps -aux | grep xxx（-aux显示所有状态）

**查看日志：**

tail -f  *.log ： 适用于实时查看日志,开发环境还行，生产就算了，日志会很多。

**tail -f error.log**  ：生产中一般用这个实时看异常日志

**编辑 vi/vim ： **

**vi x.log**  编辑你的日志文件

i  写入

:wq 保存退出  

:q! 或者 ctrl+c 退出不保存 

Shift+g 跳至当前文本最后一行，看最新的日志，都在最下面

## grep 查找（重要）

**grep 是必备日志分析命令**

**grep -r '关键字如商品ID' \*.log （使用频率最高）** 

**grep '关键字如商品ID' \*.log | grep 免费商品（在管道符前条件结果中，在加条件筛选下) **

**grep '关键字如商品ID' \*.log >> anan.txt 【相关日志输入到一个txt中，下载到本地慢慢看，我最喜欢】**

grep "被查找的字符串" 文件名
`grep -n 2019-10-24 00:01:11' *.log`
可以查找 *.log文件中，查到时间内的所有信息

## 查找特定文件 find 

**find ~ -name "需要查找的文件名"**

比如：`find ~ -name "本机ip.txt"` 就可以得到文件名所在的目录

### 管道操作符    |

可将指令连接起来,前一个指令的输出作为后一个指令的输入

### 杀僵尸进程

 部分程序员，肯定喜欢下面命令

ps -ef | grep java 【先查java进程ID】

kill -9 java进程ID 【生产环境谨慎使用】

## 对文件内容做统计 awk 


## 批量替换 sed

sed 配合正则表达式批量替换文本内容
