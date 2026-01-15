# -ModSecurity-Web-WAF-
在 Ubuntu 18.04 上部署 Apache + ModSecurity，并利用 OWASP CRS 防御 SQL 注入攻击 。

这是一个已经为你排版好的 Markdown 内容。你可以将其复制并保存为 `README.md`。

---

## 如何在 GitHub 中上传并显示截图

要在 GitHub 报告中优雅地展示你的实验截图，建议采用以下步骤：

1. 
**整理本地文件夹**：在你的 Git 仓库根目录下创建一个名为 `images` 的文件夹 。


2. 
**存放截图**：将你的拓扑图、日志图、Wireshark 结果图放入该文件夹 。


3. **在 Markdown 中引用**：使用以下语法引用图片：
`![图片描述](images/你的文件名.png)`
4. 
**推送至 GitHub**：使用 `git add .`、`git commit` 和 `git push` 将图片和文档一起上传。GitHub 会自动解析路径并显示图片 。



---

## README.md 完整内容

```markdown
# 实验报告：基于 ModSecurity (WAF) 的 Web 攻击防护实战

## 1. 实验目标 🎯
* [cite_start]在 Ubuntu 18.04 上部署基础 Web 环境（Apache + PHP） [cite: 3, 16]。
* [cite_start]安装并配置 ModSecurity (WAF) 引擎 [cite: 4, 20]。
* [cite_start]使用 OWASP 核心规则集 (CRS) 拦截 SQL 注入等常见 Web 攻击 [cite: 5, 29]。
* [cite_start]通过 Wireshark 抓包分析攻击请求与拦截响应 [cite: 6, 31]。

---

## 2. 网络拓扑 🌐
[cite_start]实验环境基于 EVE-NG 搭建，包含攻击者、防火墙网关及目标服务器 [cite: 10]。

![网络拓扑图](images/topology.png) 
*(注：请将你的拓扑截图命名为 topology.png 放入 images 文件夹)*

| 节点名称 | 角色 | IP 地址 |
| :--- | :--- | :--- |
| **Kali** | 攻击机 | [cite_start]192.168.113.131 [cite: 8] |
| **Ubuntu** | Web 服务器 | [cite_start]192.168.113.132 [cite: 9] |
| **ASAv** | 安全网关 | 202.100.1.254 (Outside) / 192.168.1.254 (Inside) [根据图片内容] |

---

## 3. 实验过程 🛠️

### A. Web 环境与 WAF 安装
[cite_start]首先在 Ubuntu 上安装 Apache 和 PHP 环境，并加载安全模块 [cite: 15, 16]：
```bash
# 安装 Apache, PHP 及 ModSecurity 模块
[cite_start]apt install apache2 php7.2 libapache2-mod-php7.2 libapache2-mod-security2 -y [cite: 16, 20]
# 启用安全模块
[cite_start]a2enmod security2 [cite: 21]
[cite_start]systemctl restart apache2 [cite: 17]

```

### B. WAF 拦截配置

将 WAF 从“仅检测”模式切换为“拦截”模式以实现主动防御 ：

```bash
# 初始化配置文件
[cite_start]cp /etc/modsecurity/modsecurity.conf-recommended /etc/modsecurity/modsecurity.conf [cite: 24]
# 修改引擎状态为 On
[cite_start]sed -i 's/SecRuleEngine DetectionOnly/SecRuleEngine On/' /etc/modsecurity/modsecurity.conf [cite: 25]

```

---

## 4. 攻击测试与结果分析 ⚖️

### 场景模拟

在 Kali 上使用 `curl` 模拟 SQL 注入攻击 ：

```bash
# 攻击载荷：id=1' OR '1'='1
[cite_start]curl -I "[http://192.168.113.132/login.php?id=1%27%20OR%20%271%27=%271](http://192.168.113.132/login.php?id=1%27%20OR%20%271%27=%271)" [cite: 29]

```

### 防护效果验证

1. 
**日志确认**：查看 `/var/log/apache2/modsec_audit.log`，确认 WAF 已命中规则并执行 `Intercepted` 操作 。


2. 
**流量分析**：Wireshark 抓包显示服务器直接返回了 **403 Forbidden**，证明恶意请求被成功阻断 。



---

## 5. 经验总结 (Troubleshooting) 📝

在实验中，通过不断排查解决了以下关键问题 ：

| 遇到的问题 | 根本原因分析 | 解决方案 |
| --- | --- | --- |
| **PHP 源代码泄露** | Nginx 未配置 FastCGI 处理 PHP 文件 。

 | 切换到对 ModSecurity 支持更原生的 Apache 环境 。

 |
| **HTTP 400 错误** | 攻击载荷中包含单引号 `'` 等字符，未进行编码 。

 | 使用 URL 编码（%27）转义攻击载荷 。

 |
| **Wireshark 无流量** | 监听了错误的网口（e0） 。

 | 重新在实际承载流量的接口开启捕获 。

 |

---

## 6. 结论

通过本实验，成功验证了 ModSecurity 配合 OWASP CRS 在应对应用层 SQL 注入攻击时的有效性 。同时，实验强调了在复杂网络环境下（如经过 ASAv 防火墙）进行流量分析和路径定位的重要性 。

