# 提高ECS实例的安全性

您可以把一个ECS实例等同于一台虚拟机，本地维护的虚拟机一般会做虚拟机实例级别的安全防护，防止攻击和入侵等。同样的，ECS实例也需要安全性防护。除了置身于阿里云自身的安全平台外，您需要根据实际的需求进一步强化安全方案。

使用本教程进行操作前，请确保您已经注册了阿里云账号。如还未注册，请先完成[账号注册](https://account.alibabacloud.com/register/intl_register.htm)。

如果ECS实例没有设置安全防护，可能会带来许多不良的影响。例如：

-   受到DDoS攻击而导致业务中断。
-   受到Web入侵而导致网页被篡改、挂马。
-   被注入而导致信息和数据泄漏等，影响ECS的使用，使其无法正常提供服务。

您可以通过以下方式提高ECS实例的安全性：

-   [设置安全组](#section_bp7_wr3_8nr)
-   [开启DDoS基础防护服务](#section_xyh_fkq_gfb)
-   [接入云安全中心](#section_ezh_fkq_gfb)
-   [接入Web应用防火墙](#section_9b9_grv_99n)

## 设置安全组

安全组是一种虚拟防火墙，具备状态检测包过滤功能。设置安全组的作用如下：

-   设置单台或多台云服务器的网络访问控制。安全组规则可以允许或者禁止与安全组相关联的云服务器ECS实例的公网和内网的入出方向的访问。
-   如果没有正确设置安全组或者安全组规则过于开放，则降低了访问的限制级别，存在安全隐患。

完成以下操作，为ECS实例所在安全组添加安全组规则：

1.  登录[ECS管理控制台](https://ecs.console.aliyun.com)。

2.  在左侧导航栏，单击**网络与安全** \> **安全组**。

3.  在顶部菜单栏左上角处，选择地域。

4.  找到要配置授权规则的安全组，在**操作**列中，单击**配置规则**。

5.  单击**添加安全组规则**。

6.  在弹出的对话框中，设置参数。

    例如：只允许特定IP远程登录到ECS实例，只需要在公网入方向配置规则即可。以Linux服务器为例，设置只让特定IP访问22端口。

    1.  添加一条公网入方向安全组规则。规则内容如下：

        -   **授权策略**选择**允许**。
        -   **协议类型**选择**自定义 TCP**。
        -   **端口范围**设置为22/22。
        -   **授权类型**选择**IPv4地址段访问**。
        -   **授权对象**设置为允许远程连接的IP地址段，格式为x.x.x.x/xx，即IP地址/子网掩码。本示例中的地址段为10.x.x.x/32。优先级为1。
    2.  再添加一条公网入方向安全组规则。规则内容如下：

        -   **授权策略**选择**拒绝**。
        -   **协议类型**选择**自定义 TCP**。
        -   **端口范围**设置为22/22。
        -   **授权类型**选择**IPv4地址段访问**。
        -   **授权对象**设置为0.0.0.0/0。优先级为2。
7.  单击**确定** 。

    设置安全组规则后，可以实现以下效果：

    -   来自IP地址10.x.x.x访问22端口优先执行优先级为1的允许规则。
    -   来自其他IP访问22端口优先执行优先级为2的拒绝规则。

## 开启DDoS基础防护服务

DDoS（Distributed Denial of Service，即分布式拒绝服务）攻击指借助于客户/服务器技术，联合多个计算机作为攻击平台，对一个或多个目标发动攻击，成倍地提高拒绝服务攻击的威力，影响业务和应用对用户提供服务。

阿里云云盾可以防护SYN Flood、UDP Flood、ACK Flood、ICMP Flood、DNS Flood、CC攻击等3到7层DDoS的攻击。DDoS基础防护免费提供高达5GB的默认DDoS防护能力。ECS实例默认开启DDoS基础防护服务。使用DDoS基础防护服务，无需采购昂贵清洗设备，受到DDoS攻击时不会影响访问速度，带宽充足不会被其他用户连带影响，保证业务可用和稳定。ECS实例创建后，您可以设置清洗阈值，具体步骤请参见[设置清洗阈值](/intl.zh-CN/DDoS原生防护用户指南/清洗设置/设置清洗阈值.md)。

在此基础上，阿里云推出了安全信誉防护联盟计划，将基于安全信誉分进一步提升DDoS防护能力，您可获得高达100GB以上的免费DDoS防护资源。您可以在云盾DDoS基础防护控制台中查看您账号当前的安全信誉分以及安全信誉详情和评分依据。详情请参见[安全信誉防护联盟](/intl.zh-CN/相关协议/原生防护相关协议/安全信誉防护联盟.md)。

## 接入云安全中心

云安全中心是一个实时识别、分析、预警安全威胁的统一安全管理系统，通过防勒索、防病毒、防篡改、合规检查等安全能力，实现威胁检测、响应、溯源的自动化安全运营闭环，保护云上资产和本地主机并满足监管合规要求。

Agent插件是云安全中心提供的本地安全插件，您必须在要防护的服务器上安装该插件才能使用云安全中心的服务。如何安装Agent插件，请参见[安装Agent](/intl.zh-CN/接入云安全中心/安装Agent.md)。

**说明：** 在购买ECS实例时，选择**安全加固**即可自动安装Agent并开通云安全中心基础版，无需您手动安装Agent。

云安全中心自动为您开通基础版功能。基础版仅提供主机异常登录检测、漏洞检测、云产品安全配置项检测，如需更多高级威胁检测、漏洞修复、病毒查杀等功能，请登录[云安全中心控制台](https://yundunnext.console.aliyun.com/?p=sas)。

## 接入Web应用防火墙

云盾Web应用防火墙（Web Application Firewall，简称WAF）基于云安全大数据能力实现，通过防御SQL注入、XSS跨站脚本、常见Web服务器插件漏洞、木马上传、非授权核心资源访问等OWASP常见攻击，过滤海量恶意CC攻击，避免您的网站资产数据泄露，保障网站的安全与可用性。

接入Web应用防火墙的好处如下：

-   无需安装任何软、硬件，无需更改网站配置、代码，它可以轻松应对各类Web应用攻击，确保网站的Web安全与可用性。除了具有强大的Web防御能力，还可以为指定网站做专属防护。适用于在金融、电商、o2o、互联网+、游戏、政府、保险等各类网站的Web应用安全防护上。
-   如果缺少WAF，只有前面介绍的防护措施，会存在短板，例如在面对数据泄密、恶意CC、木马上传篡改网页等攻击的时候，不能全面地防护，可能会导致Web入侵。

接入Web应用防火墙的具体步骤，请参见[部署WAF防护]()。

阿里云为ECS实例的安全性提供了这么多的安全产品保驾护航，您可以根据实际需要选择相应的产品，加强对系统和数据的防护，减少ECS实例受到侵害，使其稳定、持久地运行。

