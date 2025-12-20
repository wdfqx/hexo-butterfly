---
title: 使用acme.sh申请Let's Encrypt的TLS证书
categories:
  - 记录
tags:
  - CA（证书颁发机构）
  - TLS
  - SSL
  - HTTPS
abbrlink: 24439e3a
date: 2025-12-13 23:18:16
---
# 0.引言
众所周知，2025年的一个网站必须有TLS证书（别杠，我说的是普遍情况和事实标准，不是技术强制要求），申请证书有免费有付费，在**ACME（Automated Certificate Management Environment/自动证书管理环境）** 协议标准发布后，主要的各大CA都兼容了这些协议，自动签发SSL/TLS证书，支持HTTP-01和DNS-01等 **挑战（英文表述为"challenge" ，实际操作为验证域名的控制权）** 方式，本文使用DNS-01挑战申请证书。
# 1.安装acme.sh
>提醒：本节操作在Linux环境下运行，如果是Windows或其它系统请自行安装

安装很简单，直接打开终端敲这行命令：
```bash
curl https://get.acme.sh | sh -s email=you@example.com
```
*提示：安装时请把邮箱换成你自己的，尽管你直接复制我的命令也能申请就是*
终端输出如下，如果你看到的输出像我这样，就说明安装成功了
```bash
naki@naki-im:~$ curl https://get.acme.sh | sh -s email=me@naki.im
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  1032    0  1032    0     0   1377      0 --:--:-- --:--:-- --:--:--  1379
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  226k  100  226k    0     0   244k      0 --:--:-- --:--:-- --:--:--  244k
[2025年 12月 13日 星期六 23:41:28 CST] Installing from online archive.
[2025年 12月 13日 星期六 23:41:28 CST] Downloading https://github.com/acmesh-official/acme.sh/archive/master.tar.gz
[2025年 12月 13日 星期六 23:41:31 CST] Extracting master.tar.gz
[2025年 12月 13日 星期六 23:41:31 CST] It is recommended to install socat first.
[2025年 12月 13日 星期六 23:41:31 CST] We use socat for the standalone server, which is used for standalone mode.
[2025年 12月 13日 星期六 23:41:31 CST] If you don't want to use standalone mode, you may ignore this warning.
[2025年 12月 13日 星期六 23:41:31 CST] Installing to /home/naki/.acme.sh
[2025年 12月 13日 星期六 23:41:31 CST] Installed to /home/naki/.acme.sh/acme.sh
[2025年 12月 13日 星期六 23:41:31 CST] Installing alias to '/home/naki/.bashrc'
[2025年 12月 13日 星期六 23:41:31 CST] Close and reopen your terminal to start using acme.sh
[2025年 12月 13日 星期六 23:41:31 CST] Installing cron job
[2025年 12月 13日 星期六 23:41:31 CST] bash has been found. Changing the shebang to use bash as preferred.
[2025年 12月 13日 星期六 23:41:34 CST] OK
[2025年 12月 13日 星期六 23:41:34 CST] Install success! # 注意这行，出现就是安装成功了

```
# 2.开始上手使用
## 2.1 坑点
“我问你，安装完成了，那我直接敲`acme.sh`能调用了吧？”——一个不愿透露姓名的提问者
“对不起，不行，看这个”
```bash
naki@naki-im:~$ acme.sh
bash: /usr/local/bin/acme.sh: 没有那个文件或目录
```
这是你安装时不得不品的一环，因为你安装`acme.sh`的时候它默认安装到你的家目录下，而用户的home目录不在Linux系统的`$PATH`变量里面，证据如下：
```bash
naki@naki-im:~$ echo $PATH
/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games
# 可以看到的是，这里面并没有用户的家目录
```
追问：那咋整？难道我每次都要加路径么？
答：不需要的，一个 **符号链接（symbolic link/symlink）** 即可
```bash
naki@naki-im:~$ ln -sf ~/.acme.sh/acme.sh /usr/local/bin/acme.sh
[sudo] naki 的密码：
naki@naki-im:~$
```
完事了，你就可以直接调用无需再补路径了。
*原理：通过Linux系统的符号链接（symbolic link/symlink）机制实现把acme.sh的入口放在$PATH变量内的目录从而全局调用，符号链接在常规的调用中被视为一个普通的文件，系统会自动将文件内容重写到符号链接指向的文件*
## 2.2 需要准备的项目
继续操作之前，请确保你准备好了这些项目，用来填写配置文件
- 一个有效的email地址
- DNS服务商的API Token（本文以Cloudflare为例，acme.sh支持的服务商数量众多，[详见此处配置其它DNS服务商的配置](https://github.com/acmesh-official/acme.sh/wiki/dnsapi)）并确保其有足够的权限，覆盖到每一个要申请证书的域名（具体为读权限和写权限）
- 至少一个准备申请证书的域名，权威NS服务器在你的服务商处
## 2.3 设置配置文件
acme.sh已经能够访问到了，但我们是来申请证书的不是来敲命令行玩的，要申请东西就要填写申请信息这道理是个人都知道，照我这么写，实测正常申请
```text
# 账户邮箱
ACCOUNT_EMAIL='you@example.com'
# 默认CA，设置为Let‘s Encrypt
DEFAULT_CA="letsencrypt"
# 密钥长度
ACCOUNT_KEY_LENGTH=2048 
# 设置证书为EC384提升应用性能，如果需要兼容性可改为rsa-2048
CERT_KEY_LENGTH=ec-384
# 日志级别
LOG_LEVEL=2
# 默认服务器（自动生成的）
DEFAULT_ACME_SERVER='https://acme-v02.api.letsencrypt.org/directory'
# 设置Cloudflare的API Token
SAVED_CF_Token='<此处填写你Cloudflare API Token>'
# 账户ID,这个不用写也可以工作
SAVED_CF_Account_ID=''
# 这个我也不知道干嘛的
USER_PATH='/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games'
```
# 3.申请证书
恭喜你，完成了所有针对acme.sh的配置，接下来正式开始申请TLS证书
配置文件敲好了，直接申请证书就行，命令如此所示，示例为申请单个域名的通配符证书，如果你需要申请多个域名可以接着添加域名
```bash
acme.sh --issue --dns dns_cf -d 987632.xyz -d *.987632.xyz
```
## 3.1 坑点
你需要准备这些，否则可能导致证书申请失败
- 一个稳定的网络环境，我就因为这个被坑过，证据见下文 *用途：确保连接Let's Encrypt的ACME端点时的连接稳定*
- CA的授权文件，即EAB（但Let's Encrypt不需要EAB，如果你后期要换用其它家的CA需要确认其对EAB的要求情况）
看，我就这么被坑的：
```bash
naki@naki-im:~$ acme.sh --issue --dns dns_cf -d 987632.xyz -d *.987632.xyz --keylength ec-384 --force 
[2025年 12月 12日 星期五 23:11:21 CST] Using CA: https://acme-v02.api.letsencrypt.org/directory 
[2025年 12月 12日 星期五 23:11:22 CST] Multi domain='DNS:987632.xyz,DNS:*.987632.xyz' 
[2025年 12月 12日 星期五 23:11:53 CST] Getting webroot for domain='987632.xyz' [2025年 12月 12日 星期五 23:11:53 CST] Getting webroot for domain='*.987632.xyz' [2025年 12月 12日 星期五 23:11:55 CST] Adding TXT value: 0HpYepgVo9X40ovImXHtrgLKSOagT6EBnpxsN3Dl874 for domain: _acme-challenge.987632.xyz 
[2025年 12月 12日 星期五 23:12:00 CST] Please refer to https://curl.haxx.se/libcurl/c/libcurl-errors.html for error code: 35 
[2025年 12月 12日 星期五 23:12:00 CST] error zones?name=_acme-challenge.987632.xyz 
[2025年 12月 12日 星期五 23:12:00 CST] invalid domain 
[2025年 12月 12日 星期五 23:12:00 CST] Error adding TXT record to domain: _acme-challenge.987632.xyz 
[2025年 12月 12日 星期五 23:12:00 CST] Please add '--debug' or '--log' to see more information. 
[2025年 12月 12日 星期五 23:12:00 CST] See: https://github.com/acmesh-official/acme.sh/wiki/How-to-debug-acme.sh
```
## 3.2 申请完成
经过安装、配置、调试之后，申请流程之后，TLS证书应该申请成功了，输出大致如这样：
```bash
naki@naki-im:~/docs/naki.im/hexo-butterfly$ acme.sh --issue --dns dns_cf -d 987632.xyz -d *.987632.xyz
[2025年 12月 14日 星期日 00:34:38 CST] Using CA: https://acme-v02.api.letsencrypt.org/directory
[2025年 12月 14日 星期日 00:34:38 CST] Creating domain key
[2025年 12月 14日 星期日 00:34:38 CST] The domain key is here: /home/naki/.acme.sh/987632.xyz_ecc/987632.xyz.key
[2025年 12月 14日 星期日 00:34:39 CST] Multi domain='DNS:987632.xyz,DNS:*.987632.xyz'
[2025年 12月 14日 星期日 00:34:43 CST] Getting webroot for domain='987632.xyz'
[2025年 12月 14日 星期日 00:34:43 CST] Getting webroot for domain='*.987632.xyz'
[2025年 12月 14日 星期日 00:34:44 CST] 987632.xyz is already verified, skipping dns-01.
[2025年 12月 14日 星期日 00:34:44 CST] *.987632.xyz is already verified, skipping dns-01.
[2025年 12月 14日 星期日 00:34:44 CST] Verification finished, beginning signing.
[2025年 12月 14日 星期日 00:34:44 CST] Let's finalize the order.
[2025年 12月 14日 星期日 00:34:44 CST] Le_OrderFinalize='https://acme-v02.api.letsencrypt.org/acme/finalize/2873092546/458448149496'
[2025年 12月 14日 星期日 00:34:46 CST] Downloading cert.
[2025年 12月 14日 星期日 00:34:46 CST] Le_LinkCert='https://acme-v02.api.letsencrypt.org/acme/cert/06fe7c90aff16aa21790852ad442bd197e81'
[2025年 12月 14日 星期日 00:34:49 CST] Cert success.
-----BEGIN CERTIFICATE-----
MIIDlTCCAxugAwIBAgISBv58kK/xaqIXkIUq1EK9GX6BMAoGCCqGSM49BAMDMDIx
CzAJBgNVBAYTAlVTMRYwFAYDVQQKEw1MZXQncyBFbmNyeXB0MQswCQYDVQQDEwJF
NzAeFw0yNTEyMTMxNTM2MTVaFw0yNjAzMTMxNTM2MTRaMBUxEzARBgNVBAMTCjk4
NzYzMi54eXowWTATBgcqhkjOPQIBBggqhkjOPQMBBwNCAAQ3Ybdl2YvqbzBBk6Oa
LD80FyMUojxVL72eBVrdCXVhhTLfKf0Dfq7lhPT2Go3ql9wV9csECwd1NDbe8xiR
5Dlro4ICLDCCAigwDgYDVR0PAQH/BAQDAgeAMB0GA1UdJQQWMBQGCCsGAQUFBwMB
BggrBgEFBQcDAjAMBgNVHRMBAf8EAjAAMB0GA1UdDgQWBBTLf0PEF6M4kVjq+0Ys
A0j0JiNAWjAfBgNVHSMEGDAWgBSuSJ7chx1EoG/aouVgdAR4wpwAgDAyBggrBgEF
BQcBAQQmMCQwIgYIKwYBBQUHMAKGFmh0dHA6Ly9lNy5pLmxlbmNyLm9yZy8wIwYD
VR0RBBwwGoIMKi45ODc2MzIueHl6ggo5ODc2MzIueHl6MBMGA1UdIAQMMAowCAYG
Z4EMAQIBMCwGA1UdHwQlMCMwIaAfoB2GG2h0dHA6Ly9lNy5jLmxlbmNyLm9yZy8y
LmNybDCCAQsGCisGAQQB1nkCBAIEgfwEgfkA9wB2AA5XlLzzrqk+MxssmQez95Df
m8I9cTIl3SGpJaxhxU4hAAABmxiQkGAAAAQDAEcwRQIhAN5jHBZl9osORcQLxLMP
lqUylaHbShzxsaaCDu48Y1V9AiBcrtL6BM25zXbFC2uPaxXaO5/983pn2Neo5Vj2
kzQ5+gB9AHF+lfPCOIptseOEST0x4VqpYgh2LUIA4AUM0Ge1pmHiAAABmxiQkNwA
CAAABQAEEGd7BAMARjBEAiBhRq0uvQBSxnRbCRAT5LiB5ZhMRUe1H5gEql9ANqql
/AIgSniHnT5xrIMZznuy8rr5RhfskfKYwB20Z95PSkRqaH4wCgYIKoZIzj0EAwMD
aAAwZQIxAK1GzZrGz53CxMM2KiRlCf0uOCre5smA8nX0qdqGMhFvs5m/0mtWEytb
YvLbQVrOzwIwAp+8AUfyhY6WGoBsIhMxw188Xjz8ydKgQClm7xnK5llqoMCQCH0P
9eKFFAkvHFI6
-----END CERTIFICATE-----
[2025年 12月 14日 星期日 00:34:49 CST] Your cert is in: /home/naki/.acme.sh/987632.xyz_ecc/987632.xyz.cer
[2025年 12月 14日 星期日 00:34:49 CST] Your cert key is in: /home/naki/.acme.sh/987632.xyz_ecc/987632.xyz.key
[2025年 12月 14日 星期日 00:34:49 CST] The intermediate CA cert is in: /home/naki/.acme.sh/987632.xyz_ecc/ca.cer
[2025年 12月 14日 星期日 00:34:49 CST] And the full-chain cert is in: /home/naki/.acme.sh/987632.xyz_ecc/fullchain.cer
```
acme.sh会直接把证书的内容输出到终端，如果要读取密钥，则需要从输出中找到密钥文件的路径，使用cat命令或使用文本编辑器打开查看。
# 4.完成
恭喜你，你又学会了个新技能：使用acme.sh申请TLS证书
至于这玩意的用途……建站是需要TLS证书的，但托管平台会自动给你申请，如果有需要设置自定义证书就能把这玩意派上用场。