# PowerShell Invoke-WebRequest 代理指南

[![推广](https://github.com/bright-cn/Awesome-Web-Scraping/blob/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner%20CN.png)](https://www.bright.cn/proxy-types) 

阅读完这篇 **Invoke-WebRequest PowerShell 代理指南**后，你将了解：

1. [什么是 PowerShell Invoke-WebRequest？](#什么是-powershell-invoke-webrequest)
2. [安装 Invoke-WebRequest](#安装-invoke-webrequest)  
   2.1. [Windows](#windows)  
   2.2. [macOS 和 Linux](#macos-和-linux)
3. [在 PowerShell 中开始使用代理的前提条件](#在-powershell-中开始使用代理的前提条件)
4. [如何在 Invoke-WebRequest 中指定 HTTP 代理](#如何在-invoke-webrequest-中指定-http-代理)  
   4.1. [使用命令行选项](#使用命令行选项)  
   4.2. [使用环境变量](#使用环境变量)
5. [如何在 PowerShell 中使用 HTTPS 和 SOCKS 代理](#如何在-powershell-中使用-https-和-socks-代理)
6. [你需要了解的技巧和窍门](#你需要了解的技巧和窍门)  
   6.1. [忽略 PowerShell 代理配置](#忽略-powershell-代理配置)  
   6.2. [避免 SSL 证书错误](#避免-ssl-证书错误)
7. [你应该使用哪种 PowerShell 代理？](#你应该使用哪种-powershell-代理)

让我们深入了解吧！

---

## 什么是 PowerShell Invoke-WebRequest？

**Invoke-WebRequest** 是一个 PowerShell cmdlet，用于向 Web 服务器和 Web 服务发送 **HTTP、HTTPS 和 FTP** 请求。默认情况下，它会自动解析服务器生成的响应，并返回表单、链接、图像或其他重要 HTML 元素的集合。

它通常用于访问 REST API、从网页下载文件或与 Web 服务交互。下面是 **Invoke-WebRequest** 请求的基本语法：

```powershell
Invoke-WebRequest [-Uri] <Uri> [-Method <WebRequestMethod>] [-Headers <IDictionary>] [-Body <Object>]
```

**需要牢记的关键参数**：

- Uri：请求所发送到的 Web 资源 URI。
- Method：请求使用的 HTTP 方法（例如 GET、POST、PUT、DELETE）。
- Invoke-WebRequest 默认发送 GET 请求。
- Headers：请求中要包含的其他 HTTP 标头。
- Body：要发送到服务器的请求正文。
 
如你所见，唯一必需的参数是 <Uri>。因此，向指定 URI 发送 GET 请求的最简单语法是：

```powershell
Invoke-WebRequest <Uri>
```

## 安装 Invoke-WebRequest

要使用 Invoke-WebRequest，你需要安装 PowerShell。接下来，了解如何安装 PowerShell 并使用 Invoke-WebRequest cmdlet！

### Windows

首先要知道，Windows PowerShell 和 PowerShell 是两个不同的工具。Windows PowerShell 是 Windows 自带的 PowerShell 版本（最新版本为 5.1），其中提供了 Invoke-WebRequest cmdlet。如果你使用的是较新的 Windows 版本，就可以直接开始！对于较旧的版本，请参阅官方 PowerShell 安装指南。

Invoke-WebRequest 的某些功能从 PowerShell 7.x 开始才可用。有关安装方法的详细信息，请参阅从 Windows PowerShell 5.1 迁移到 PowerShell 7 的官方指南。请注意，PowerShell 7.x 会安装到新目录中，并可与 Windows PowerShell 5.1 并行运行。

你可以使用以下命令检查 Windows 计算机上当前安装的 PowerShell 版本：

```powershell
$PSVersionTable
```

在 PowerShell 7.x 中，可能会输出如下内容：

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

PowerShell 7.x 可以安装在 macOS 和 Linux 上。不过，如果只是为了使用 Invoke-WebRequest cmdlet，在这些操作系统上安装整个 PowerShell 生态系统并不划算。你可以改用 curl，它预装在 macOS 和大多数 Linux 发行版中，并提供相同的功能。请参阅我们的 [curl 代理指南](https://www.bright.cn/blog/proxy-101/curl-with-proxies)了解更多信息。

## 在 PowerShell 中开始使用代理的前提条件

代理充当客户端与目标服务器之间的中介：它会拦截你的请求，将其转发到服务器，接收响应，然后将响应返回给你。这样，目标服务器看到的请求来自所选代理服务器的 IP 和位置，而不是你的 IP 和位置。

要开始将 PowerShell 代理与 Invoke-WebRequest 搭配使用，你需要先了解代理服务器 URL 的格式。

下面是 PowerShell Invoke-WebRequest 代理的 URL：

```powershell
<PROTOCOL>://[<USERNAME>:<PASSWORD>]@<HOST>[:<PORT>]
```

**它由以下部分组成：**

- `PROTOCOL`：用于连接代理服务器的协议。
- `HOST`：代理服务器的 IP 地址或主机名 URL。
- `PORT`：代理服务器监听的端口号。
- `USERNAME`：用于代理身份验证的可选用户名。
- `PASSWORD`：用于代理身份验证的可选密码。

> 💡 **重要提示：**\
> **Invoke-WebRequest 要求必须包含 <PROTOCOL>:// 部分。如果省略该部分，请求将失败并显示：**\
> **Invoke-WebRequest：相对 URI 不支持此操作。**

在 PowerShell 5.1 中，Invoke-WebRequest 仅支持 HTTP，而在 PowerShell 7.x 中，它还支持 HTTPS 和 SOCKS。

现在来获取一个有效的 HTTP 代理！

你可以在网上找到免费代理，例如：

```
Protocol: HTTP; IP Address: 190.6.23.219; Port: 999
```

将其组合为：

```
http://190.6.23.219:999
```

>⚠️ **警告：**\
> **免费代理不可靠、容易出错、速度慢、可能会窃取数据，而且生命周期很短。请勿使用！**

解决方案是什么？使用市场领先提供商 Bright Data 的高级代理。订阅并免费试用我们可靠的代理。

[Bright Data 的代理服务](https://www.bright.cn/proxy-types)受身份验证保护，只有受信任的用户才能访问。 

假设：

协议：`HTTP` 主机：`45.103.203.109` 端口：`9571` 用户名：`admin-4521` 密码：`rUuH3tJqf`

那么 Invoke-WebRequest 代理 URL 为：

```powershell
http://admin-4521:@rUuH3tJqf45.103.203.109:9571
```

## 如何在 Invoke-WebRequest 中指定 HTTP 代理

开始前，请运行：

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

重点关注 **Content** 字段，其中包含**你的 IP**。

为什么？因为 `https://httpbin.org/ip` 会返回请求的来源 IP。因此，如果未设置代理，它返回的就是你计算机的 IP。

如果你只想获取 Content 字段：

```powershell
$response = Invoke-WebRequest "https://httpbin.org/ip"
$response.Content
```

可能会输出：

```json
{
  "origin": "194.34.233.12"
}
```

如果通过代理路由该请求，你将看到代理服务器的 IP。这是确认 PowerShell Invoke-WebRequest 是否确实在使用代理的绝佳测试方法。

可以通过以下几种方式在 Invoke-WebRequest 中设置 PowerShell 代理：

## 使用命令行选项

Invoke-WebRequest 提供 `-Proxy` 标志：

```powershell
Invoke-WebRequest -Proxy "<PROTOCOL>://[<USERNAME>:<PASSWORD>]@<HOST>[:<PORT>]" <Uri>
```

因此：

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

结果为：

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
这里的 `origin` 与代理服务器 IP 一致，表明请求已通过代理发送。非常好！

> <strong>注意：</strong>免费代理的生命周期很短。如果该代理失效，请换一个。

## 使用环境变量

从 PowerShell 7.0 开始，Invoke-WebRequest 支持通过环境变量进行代理配置。

设置两个环境变量：

- `HTTP_PROXY:` HTTP 请求的代理 URL。
- `HTTPS_PROXY:` HTTPS 请求的代理 URL。

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

在 macOS/Linux 上：

```bash
export HTTP_PROXY="<PROTOCOL>://[<USERNAME>:<PASSWORD>]@<HOST>[:<PORT>]"
export HTTPS_PROXY="<PROTOCOL>://[<USERNAME>:<PASSWORD>]@<HOST>[:<PORT>]"
```

例如：

```bash
export http_proxy="http://190.6.23.219:999"
export https_proxy="http://190.6.23.219:999"
```

现在，所有 Invoke-WebRequest 调用都会自动通过这些代理。运行：

```powershell
Invoke-WebRequest "https://httpbin.org/ip"
```

你会再次在 `origin` 中看到代理 IP。非常好。

要关闭这些代理，请取消设置相应的环境变量：

```powershell
$env:HTTP_PROXY  = ""
$env:HTTPS_PROXY = ""
```

macOS/Linux：

```bash
unset HTTP_PROXY
unset HTTPS_PROXY
```

之后，Invoke-WebRequest `"https://httpbin.org/ip"` 会再次显示你的 IP。

## 如何在 PowerShell 中使用 HTTPS 和 SOCKS 代理

要使用 HTTPS 或 [SOCKS 代理](https://www.bright.cn/solutions/socks5-proxies)，你必须使用 PowerShell 7.x 或更高版本，否则会看到：

```
Invoke-WebRequest : The ServicePointManager does not support proxies with the https scheme.
```

对于 SOCKS，则会看到：

```
Invoke-WebRequest : The ServicePointManager does not support proxies with the socks scheme.
```

在 PowerShell 7.x 中，其结构仍然是：

```powershell
Invoke-WebRequest -Proxy "<PROTOCOL>://[<USERNAME>:<PASSWORD>]@<HOST>[:<PORT>]" <Uri>
```

但 `<PROTOCOL>` 可以是 `https`、`socks4`、`socks4a`、`socks5` 或 `socks5a`（而不再仅限于 `http`）。

如果尝试使用这些选项之外的协议，你会看到：

```
Invoke-WebRequest: Only the 'http', 'https', 'socks4', 'socks4a' and 'socks5' schemes are allowed for proxies.
```

SOCKS 代理示例：

```powershell
Invoke-WebRequest -Proxy "socks5://94.14.109.54:3567" "http://httpbin.org/ip"
```

可能会输出：

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

## 你需要了解的技巧和窍门

了解如何像专业人士一样使用 PowerShell Invoke-WebRequest 代理的实用技巧。

### 忽略 PowerShell 代理配置

使用 `-NoProxy` 可跳过通过环境变量配置的代理：

```powershell
Invoke-WebRequest -NoProxy <Uri>
```

此命令会在不使用代理的情况下访问 `<Uri>`。

你可以先在环境变量中设置代理，然后运行以下命令进行验证：

```powershell
Invoke-WebRequest -NoProxy "https://httpbin.org/ip"
```

你会看到自己的 IP，而不是代理 IP。

### 避免 SSL 证书错误

使用 HTTP 代理时，可能会因 SSL 证书错误而失败。请使用 -SkipCertificateCheck：

```powershell
Invoke-WebRequest -SkipCertificateCheck -Proxy "<PROTOCOL>://[<USERNAME>:<PASSWORD>]@<HOST>[:<PORT>]" <Uri>
```

`-SkipCertificateCheck` 允许与不安全的服务器建立连接。这样做并不安全，因此只能用于受信任的主机。

示例：

```powershell
Invoke-WebRequest -SkipCertificateCheck -Proxy "http://190.6.23.219:999" "https://httpbin.org/ip"
```

## 你应该使用哪种 PowerShell 代理？

这取决于你使用 Invoke-WebRequest 的目标。请考虑以下主要代理类型：

- [**数据中心代理：**](https://www.bright.cn/proxy-types/datacenter-proxies)速度快、价格低，但一旦被识别就很容易遭到封锁。
- [**住宅代理：**](https://www.bright.cn/proxy-types/residential-proxies)使用来自真实设备的动态 IP 地址，非常适合访问受地理限制的内容或规避反爬虫机制。
- [**ISP 代理：**](https://www.bright.cn/proxy-types/isp-proxies)由 ISP 提供安全、快速的静态 IP，是 SEO 或市场研究的理想选择。

请参阅我们的[代理 IP 类型指南](https://www.bright.cn/blog/proxy-101/ultimate-guide-to-proxy-types)了解更多信息。
