---
title: "Git_connect"
date: 2024-10-05T12:33:26+08:00
draft: false
---
# VSCode GitHub 代理问题排查笔记

## 问题描述

在使用 VSCode 通过远程服务器进行 GitHub 身份验证时，遇到了代理连接问题。尽管系统环境变量中已经设置了代理，并且代理软件已经运行在了我所设置的9903端口上，但是VSCode的Github Verification插件仍然无法成功连接到 GitHub。

- 运行`netstat -tulpn | grep 9903`显示：
  ```
   > netstat -tulpn | grep 9903
    tcp        0      0 127.0.0.1:9903          0.0.0.0:*               LISTEN      490509/clash-meta   
    udp        0      0 127.0.0.1:9903          0.0.0.0:*                           490509/clash-meta 
  ```

## 环境配置

- 操作系统：Ubuntu-22.04
- 代理设置：http://127.0.0.1:9903
- 使用的软件：VSCode、Git、Clash-meta（作为代理）

## 排查步骤

1. **验证环境变量设置**
   - 运行 `echo $http_proxy` 和 `echo $https_proxy`
   - 确认输出均为 `http://127.0.0.1:9903`

2. **检查 Git 全局代理设置**
   - 运行 `git config --global --get http.proxy` 和 `git config --global --get https.proxy`
   - 初始检查可能显示这些设置尚未配置

3. **测试代理连接**
   - 使用 curl 命令测试：`curl -v -x http://127.0.0.1:9903 https://github.com`
   - 确认返回了 GitHub 主页的 HTML 内容，说明代理连接正常

4. **设置 Git 全局代理**
   - 运行以下命令设置 Git 全局代理：
     ```
     git config --global http.proxy http://127.0.0.1:9903
     git config --global https.proxy http://127.0.0.1:9903
     ```

5. **重启 VSCode**
   - 完全关闭并重新打开 VSCode

## 解决方案

问题通过以下步骤得到解决：

1. 设置 Git 全局代理：
   ```
   git config --global http.proxy http://127.0.0.1:9903
   git config --global https.proxy http://127.0.0.1:9903
   ```

2. 重启 VSCode

这个解决方案表明，尽管系统环境变量已经设置了代理，VSCode 可能更依赖于 Git 的全局配置来处理 GitHub 连接。通过显式设置 Git 的全局代理并重启 VSCode，成功解决了身份验证问题。

## 原因分析

1. VSCode 可能优先使用 Git 的配置而不是系统环境变量来处理 Git 操作。
2. 重启 VSCode 可能是必要的，以确保新的 Git 配置被正确加载和应用。

## 预防措施

1. 在配置远程开发环境时，确保同时检查并设置：
   - 系统环境变量
   - Git 全局配置
   - VSCode 的代理设置
2. 在更改任何网络相关配置后，养成重启 VSCode 的习惯。
3. 保持 VSCode 和 Git 版本更新，以获得最新的bug修复和功能改进。

## 参考资源

- [Git 文档 - 配置 Git 使用代理](https://git-scm.com/docs/git-config#Documentation/git-config.txt-httpproxy)
- [VSCode 文档 - 网络代理](https://code.visualstudio.com/docs/setup/network)
- [GitHub 文档 - 配置 Git 使用代理](https://docs.github.com/en/authentication/troubleshooting-ssh/using-ssh-over-the-https-port#enabling-ssh-connections-over-https)
