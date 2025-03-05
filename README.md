# PowerShell Invoke-WebRequest 代理指南

[![促销](https://github.com/bright-cn/LinkedIn-Scraper/blob/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://www.bright.cn/proxy-types)

在阅读完这份 **PowerShell Invoke-WebRequest 代理指南** 后，你将了解：

1. [什么是 PowerShell Invoke-WebRequest？](#什么是-powershell-invoke-webrequest)
2. [如何安装 Invoke-WebRequest](#如何安装-invoke-webrequest)  
   2.1. [Windows](#windows)  
   2.2. [macOS 和 Linux](#macos-和-linux)
3. [在 PowerShell 中使用代理的前提条件](#在-powershell-中使用代理的前提条件)
4. [如何在 Invoke-WebRequest 中设置 HTTP 代理](#如何在-invoke-webrequest-中设置-http-代理)  
   4.1. [使用命令行选项](#使用命令行选项)  
   4.2. [使用环境变量](#使用环境变量)
5. [如何在 PowerShell 中使用 HTTPS 和 SOCKS 代理](#如何在-powershell-中使用-https-和-socks-代理)
6. [你需要知道的技巧](#你需要知道的技巧)  
   6.1. [忽略 PowerShell 代理配置](#忽略-powershell-代理配置)  
   6.2. [避免 SSL 证书错误](#避免-ssl-证书错误)
7. [应当使用哪种 PowerShell 代理？](#应当使用哪种-powershell-代理)

让我们开始吧！

---

## 什么是 PowerShell Invoke-WebRequest？

**Invoke-WebRequest** 是一个 PowerShell cmdlet，用于向 Web 服务器和 Web 服务发送 **HTTP、HTTPS 和 FTP** 请求。默认情况下，它会自动解析服务器产生的响应，并返回表单、链接、图片或其他重要的 HTML 元素集合。

通常，它用于访问 REST API、从网络下载文件或与 Web 服务进行交互。下面是一个 **Invoke-WebRequest** 请求的基本语法：

```powershell
Invoke-WebRequest [-Uri] <Uri> [-Method <WebRequestMethod>] [-Headers <IDictionary>] [-Body <Object>]
```

**需要记住的关键参数**：

- Uri：发送请求的目标 Web 资源的 URI。
- Method：请求使用的 HTTP 方法（例如 GET、POST、PUT、DELETE）。
- Invoke-WebRequest 默认发送 GET 请求。
- Headers：在请求中包含的额外 HTTP 头。
- Body：发送到服务器的请求正文。

如你所见，唯一必需的参数是 <Uri>。因此，向给定 URI 执行 GET 请求的最简写法是：

```powershell
Invoke-WebRequest <Uri>
```

## 如何安装 Invoke-WebRequest

要使用 Invoke-WebRequest，你需要安装 PowerShell。下面让我们了解如何安装 PowerShell 并使用 Invoke-WebRequest cmdlet 吧！

### Windows

请先了解，Windows PowerShell 和 PowerShell 是不同的东西。Windows PowerShell 是随 Windows 一起提供的 PowerShell 版本（最新版本为 5.1），并自带 Invoke-WebRequest cmdlet。如果你使用的是较新的 Windows 版本，那么你已经可以直接使用了！对于更老的版本，请参阅官方的 PowerShell 安装指南。

从 PowerShell 7.x 开始，Invoke-WebRequest 才支持更多的新特性。如何安装请参阅官方从 Windows PowerShell 5.1 迁移到 PowerShell 7 的指南。需要注意的是，PowerShell 7.x 会被安装到新的目录，并且可以与 Windows PowerShell 5.1 并存。

你可以通过以下命令来查看当前 Windows 机器上 PowerShell 的版本：

```powershell
$PSVersionTable
```

在 PowerShell 7.x 上，可能会输出如下内容：

```powershell
PSVersion                   7.4.1
PSEdition                   Core
GitCommitId                 7.4.1
OS                          Microsoft Windows 10.0.22631
Platform                    Win32NT
PSCompatibleVersions        {1.0, 2.0, 3.0, 4.0…}
PSRemotingProtocolVersion   2.3
SerializationVersion        1.1.0.1
WSManStackVersion           3.0
```

### macOS 和 Linux

PowerShell 7.x 可以安装在 macOS 和 Linux 上。不过，如果你只想使用 Invoke-WebRequest cmdlet，在 macOS 或 Linux 上安装整个 PowerShell 生态系统可能意义不大。你也可以使用预装在大多数 macOS 和 Linux 发行版上的 curl，它具备相同的功能。想要了解更多可参考我们的 [curl 代理指南](https://www.bright.cn/blog/proxy-101/curl-with-proxies)。

## 在 PowerShell 中使用代理的前提条件

代理在客户端和目标服务器之间充当中间人：它会拦截你的请求，将其转发给服务器，并接收服务器的响应后再传回给你。这样，目标服务器看到的请求就会来自所选代理服务器的 IP 和位置，而不是你的真实地址。

要在 PowerShell 中使用 Invoke-WebRequest 代理，你需要先了解代理服务器 URL 的基本结构。

这是一个 PowerShell Invoke-WebRequest 代理的 URL 格式：

```powershell
<PROTOCOL>://[<USERNAME>:<PASSWORD>]@<HOST>[:<PORT>]
```

**它由以下部分组成**：

- `PROTOCOL`：连接到代理服务器所使用的协议。
- `HOST`：代理服务器所在主机的 IP 地址或域名。
- `PORT`：代理服务器监听的端口号。
- `USERNAME`：可选，用于代理身份验证的用户名。
- `PASSWORD`：可选，用于代理身份验证的密码。

> 💡 **重要提示：**  
> **在 Invoke-WebRequest 中，<PROTOCOL>:// 这一部分是必需的。如果你省略它，请求会失败并提示：**  
> **Invoke-WebRequest : This operation is not supported for a relative URI.**

在 PowerShell 5.1 中，Invoke-WebRequest 仅支持 HTTP，而在 PowerShell 7.x 中还支持 HTTPS 和 SOCKS。

接下来，让我们获取一个有效的 HTTP 代理！

你可以在网上免费找到一些，如：

```
Protocol: HTTP; IP Address: 190.6.23.219; Port: 999
```

将其组合为：

```
http://190.6.23.219:999
```

>⚠️ **警告：**  
> **免费代理通常不可靠、容易出错、速度慢、可能窃取数据，而且寿命短。不要使用它们！**

解决方案？使用 Bright Data 的付费优质代理——市场上最好的提供商。订阅并免费试用我们可靠的代理服务。

[Bright Data 的代理服务](https://www.bright.cn/proxy-types) 受身份验证保护，只有受信任的用户才能访问。

假设：

Protocol: `HTTP`  
Host: `45.103.203.109`  
Port: `9571`  
Username: `admin-4521`  
Password: `rUuH3tJqf`

那么对应的 Invoke-WebRequest 代理 URL 为：

```powershell
http://admin-4521:@rUuH3tJqf45.103.203.109:9571
```

## 如何在 Invoke-WebRequest 中设置 HTTP 代理

在开始之前，先运行：

```powershell
Invoke-WebRequest "https://httpbin.org/ip"
```

可能会输出：

```yaml
StatusCode         : 200
StatusDescription  : OK
Content           : {
                      "origin": "194.34.233.12"
                    }
RawContent        : HTTP/1.1 200 OK
                    Connection: keep-alive
                    Access-Control-Allow-Origin: *
                    Access-Control-Allow-Credentials: true
                    Content-Length: 32
                    Content-Type: application/json
                    Date: Thu, 01 Feb 2024 10:46:14 GMT...
Forms            : {}
Headers          : {[Connection, keep-alive], [Access-Control-Allow-Origin, *], 
                   [Access-Control-Allow-Credentials, true], [Content-Length, 32]...}
Images           : {}
InputFields      : {}
Links            : {}
ParsedHtml       : mshtml.HTMLDocumentClass
RawContentLength : 32
```

关注 **Content** 字段，其中包含你的 **IP**。

为什么？因为 `https://httpbin.org/ip` 会返回请求的源 IP。如果没有设置代理，它就会显示你自己的机器 IP。

如果你只想看 Content 字段，可以这样：

```powershell
$response = Invoke-WebRequest "https://httpbin.org/ip"
$response.Content
```

输出类似：

```json
{
  "origin": "194.34.233.12"
}
```

如果你通过代理路由该请求，你会看到返回的 IP 是代理服务器的 IP。这是确认 PowerShell Invoke-WebRequest 是否确实使用了你配置的代理的好方法。

在 PowerShell 中设置代理有以下几种方式：

### 使用命令行选项

Invoke-WebRequest 提供了 `-Proxy` 参数：

```powershell
Invoke-WebRequest -Proxy "<PROTOCOL>://[<USERNAME>:<PASSWORD>]@<HOST>[:<PORT>]" <Uri>
```

示例：

```powershell
Invoke-WebRequest -Proxy "http://190.6.23.219:999" "https://httpbin.org/ip"

Invoke-WebRequest -Uri "http://httpbin.org/ip" `
  -Proxy "http://brd.superproxy.io:33335" `
  -ProxyCredential (
    New-Object System.Management.Automation.PSCredential(
      "brd-customer-CUSTOMER_ID-zone-ZONE’S_NAME",
      ("ZONE’S_PASSWORD" | ConvertTo-SecureString -AsPlainText -Force)
    )
  )
```

结果可能是：

```yaml
StatusCode         : 200
StatusDescription  : OK
Content           : {
                      "origin": "190.6.23.219"
                    }
RawContent        : HTTP/1.1 200 OK
                    Connection: keep-alive
                    Access-Control-Allow-Origin: *
                    Access-Control-Allow-Credentials: true
                    Content-Length: 31
                    Content-Type: application/json
                    Date: Thu, 01 Feb 2024 12:36:56 GMT...
Forms            : {}
Headers          : {[Connection, keep-alive], [Access-Control-Allow-Origin, *], 
                   [Access-Control-Allow-Credentials, true], [Content-Length, 31]...}
Images           : {}
InputFields      : {}
Links            : {}
ParsedHtml       : mshtml.HTMLDocumentClass
RawContentLength : 31
```

这里的 `origin` 就变成了代理服务器的 IP，说明请求已通过代理转发。完美！

> **注意：** 免费代理往往寿命短，如果那个失效了，你需要换另一个。

### 使用环境变量

从 PowerShell 7.0 开始，Invoke-WebRequest 支持通过环境变量自动配置代理。

你需要设置以下两个环境变量：

- `HTTP_PROXY:` 供 HTTP 请求使用的代理 URL。
- `HTTPS_PROXY:` 供 HTTPS 请求使用的代理 URL。

在 Windows 上：

```powershell
$env:HTTP_PROXY  = "<PROTOCOL>://[<USERNAME>:<PASSWORD>]@<HOST>[:<PORT>]"
$env:HTTPS_PROXY = "<PROTOCOL>://[<USERNAME>:<PASSWORD>]@<HOST>[:<PORT>]"
```

例如：

```powershell
$env:HTTP_PROXY  = "http://190.6.23.219:999"
$env:HTTPS_PROXY = "http://190.6.23.219:999"
```

在 macOS / Linux 上：

```bash
export HTTP_PROXY="<PROTOCOL>://[<USERNAME>:<PASSWORD>]@<HOST>[:<PORT>]"
export HTTPS_PROXY="<PROTOCOL>://[<USERNAME>:<PASSWORD>]@<HOST>[:<PORT>]"
```

例如：

```bash
export http_proxy="http://190.6.23.219:999"
export https_proxy="http://190.6.23.219:999"
```

现在，所有的 Invoke-WebRequest 调用都会自动通过这些代理发送。执行：

```powershell
Invoke-WebRequest "https://httpbin.org/ip"
```

你将在 `origin` 字段中再次看到代理的 IP，说明一切正常。

要关闭这些代理，只需清空或取消设置它们：

```powershell
$env:HTTP_PROXY  = ""
$env:HTTPS_PROXY = ""
```

macOS / Linux 上：

```bash
unset HTTP_PROXY
unset HTTPS_PROXY
```

然后再次执行 `Invoke-WebRequest "https://httpbin.org/ip"`，你会看到你的真实 IP。

## 如何在 PowerShell 中使用 HTTPS 和 SOCKS 代理

要使用 HTTPS 或 [SOCKS 代理](https://www.bright.cn/solutions/socks5-proxies)，则必须使用 PowerShell 7.x 及更新版本，否则会出现：

```
Invoke-WebRequest : The ServicePointManager does not support proxies with the https scheme.
```

或 SOCKS：

```
Invoke-WebRequest : The ServicePointManager does not support proxies with the socks scheme.
```

在 PowerShell 7.x 中，使用方式仍然是：

```powershell
Invoke-WebRequest -Proxy "<PROTOCOL>://[<USERNAME>:<PASSWORD>]@<HOST>[:<PORT>]" <Uri>
```

但 `<PROTOCOL>` 可以为 `https`、`socks4`、`socks4a`、`socks5`、`socks5a`（不再局限于 `http`）。

如果使用了这几种协议以外的值，你会收到提示：

```
Invoke-WebRequest: Only the 'http', 'https', 'socks4', 'socks4a' and 'socks5' schemes are allowed for proxies.
```

下面是一个 SOCKS 代理的示例：

```powershell
Invoke-WebRequest -Proxy "socks5://94.14.109.54:3567" "http://httpbin.org/ip"
```

输出可能为：

```yaml
StatusCode         : 200
StatusDescription  : OK
Content           : {
                      "origin": "94.14.109.54"
                    }
RawContent        : HTTP/1.1 200 OK
                    Connection: keep-alive
                    Access-Control-Allow-Origin: *
                    Access-Control-Allow-Credentials: true
                    Content-Length: 31
                    Content-Type: application/json
                    Date: Thu, 01 Feb 2024 12:47:56 GMT...
Forms            : {}
Headers          : {[Connection, keep-alive], [Access-Control-Allow-Origin, *],
                   [Access-Control-Allow-Credentials, true], [Content-Length, 31]...}
Images           : {}
InputFields      : {}
Links            : {}
ParsedHtml       : mshtml.HTMLDocumentClass
RawContentLength : 31
```

## 你需要知道的技巧

以下是在 PowerShell Invoke-WebRequest 中使用代理时一些有用的小贴士。

### 忽略 PowerShell 代理配置

可以在命令里使用 `-NoProxy` 跳过环境变量中配置的代理：

```powershell
Invoke-WebRequest -NoProxy <Uri>
```

这样就会直连 `<Uri>`，不使用代理。

你可以先设置环境变量代理，然后执行：

```powershell
Invoke-WebRequest -NoProxy "https://httpbin.org/ip"
```

你会看到返回的是自己的 IP，而不是代理的 IP。

### 避免 SSL 证书错误

使用 HTTP 代理时，有时会因为 SSL 证书问题导致请求失败。可以使用 `-SkipCertificateCheck` 忽略 SSL 证书检查：

```powershell
Invoke-WebRequest -SkipCertificateCheck -Proxy "<PROTOCOL>://[<USERNAME>:<PASSWORD>]@<HOST>[:<PORT>]" <Uri>
```

`-SkipCertificateCheck` 会忽略不安全的服务器连接。仅在你非常信任该主机时使用此选项。

示例：

```powershell
Invoke-WebRequest -SkipCertificateCheck -Proxy "http://190.6.23.219:999" "https://httpbin.org/ip"
```

## 应当使用哪种 PowerShell 代理？

这取决于你在使用 Invoke-WebRequest 时的目标。常见的几类代理如下：

- [**数据中心代理**](https://www.bright.cn/proxy-types/datacenter-proxies)：速度快，价格相对便宜，但如果被识别可能容易被屏蔽。  
- [**住宅代理**](https://www.bright.cn/proxy-types/residential-proxies)：提供真实设备的 IP 地址，会定期轮换，非常适合绕过地理位置限制或反爬虫策略。  
- [**ISP 代理**](https://www.bright.cn/proxy-types/isp-proxies)：由运营商提供的静态 IP，速度快且高度可靠，适合用于 SEO、市场调研等需求。  
- [**移动代理**](https://www.bright.cn/proxy-types/mobile-proxies)：来自真实移动设备，适合针对移动端应用或网站的场景。

更多详情可参阅我们的 [代理 IP 类型终极指南](https://www.bright.cn/blog/proxy-101/ultimate-guide-to-proxy-types)。
