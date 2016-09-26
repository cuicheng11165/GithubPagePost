---
layout: post
title:  "使用Fiddler Core 来测试你的Web API"
date:   2015-12-15
author: 独上高楼
category: 编程
tags: [CSharp]
---

<p>大家在调试Web相关的API时，经常会用Fiddler来查看相关的请求，以及返回结果。当然你也可以尝试修改或者重复你的请求信息。本文主要介绍如何使用代码来实现fiddler的功能。</p>
<h2>Fiddler Core API</h2>
<p>Fiddler Core几乎实现了你能用fiddler做的所有功能。直接在NuGet上搜索FiddlerCore即可下载FiddlerCore的.Net API。</p>
<p><img src="http://images2015.cnblogs.com/blog/147759/201512/147759-20151213123051262-2134426614.png" alt="" /></p>
<h2>开启Fiddler Application</h2>
<p>使用下面的代码来开启FiddlerApplication</p>
<p><span style="color: #2b91af; font-family: Consolas; font-size: 9pt;"><span style="background-color: white;">FiddlerApplication<span style="color: black;">.Startup(9898, <span style="color: #2b91af;">FiddlerCoreStartupFlags<span style="color: black;">.Default);</span></span></span></span> </span></p>
<p>执行后，fiddler会运行一个http代理服务器，你可以使用FiddlerCoreStartupFlags.RegisterAsSystemProxy 来把这个代理服务器指定为系统代理，这样就可以监听到本机所有的http请求。</p>
<p>当程序结束的时候，记得使用下面的语句来关闭代理。</p>
<p><span style="color: #2b91af; font-family: Consolas; font-size: 9pt;"><span style="background-color: white;">FiddlerApplication<span style="color: black;">.Shutdown();</span></span> </span></p>
<h2>捕获HttpRequest/HttpResponse</h2>
<p>开启了Fiddler Application之后，Fiddler在捕获Request/Response的时候会触发下面这两个事件，你只需要定义事件来实现如何处理捕获到的请求即可。</p>
<p><span style="color: black; font-family: Consolas; font-size: 9pt; background-color: white;"> <span style="color: green;">//</span></span></p>
<p><span style="color: black; font-family: Consolas; font-size: 9pt; background-color: white;"> <span style="color: green;">// Summary:</span></span></p>
<p><span style="color: black; font-family: Consolas; font-size: 9pt; background-color: white;"> <span style="color: green;">// This event fires when a client request is received by Fiddler</span></span></p>
<p><span style="color: black; font-family: Consolas; font-size: 9pt; background-color: white;"> <span style="color: blue;">public<span style="color: black;"> <span style="color: blue;">static<span style="color: black;"> <span style="color: blue;">event<span style="color: black;"> <span style="color: #2b91af;">SessionStateHandler<span style="color: black;"> BeforeRequest; </span></span></span></span></span></span></span></span></span></p>
<p><span style="color: black; font-family: Consolas; font-size: 9pt; background-color: white;"> <span style="color: green;">//</span></span></p>
<p><span style="color: black; font-family: Consolas; font-size: 9pt; background-color: white;"> <span style="color: green;">// Summary:</span></span></p>
<p><span style="color: black; font-family: Consolas; font-size: 9pt; background-color: white;"> <span style="color: green;">// This event fires when a server response is received by Fiddler</span></span></p>
<p><span style="color: black; font-family: Consolas; font-size: 9pt;"><span style="background-color: white;"> <span style="color: blue;">public<span style="color: black;"> <span style="color: blue;">static<span style="color: black;"> <span style="color: blue;">event<span style="color: black;"> <span style="color: #2b91af;">SessionStateHandler<span style="color: black;"> BeforeResponse;</span></span></span></span></span></span></span></span></span> </span></p>
<h2>安装证书</h2>
<p>那么如何捕获https协议的页面呢？众所周知，https通过通信证书来实现了服务器端和客户端的加密，避免通信过程被监听。Fiddler通过中间人的方式来实现https协议的捕获，所谓中间人就是Fiddler注入到应用程序和服务器的中间，fiddler相对于服务器扮演客户端的角色，相对于客户端扮演服务器的角色，既然fiddler需要扮演服务器的角色，就需要一个证书，并且你的客户端需要信任Fiddler的证书。我们以中国银行的网站为例：</p>
<p>不开启Fiddler登陆网银时，证书信息为：</p>
<p><img src="http://images2015.cnblogs.com/blog/147759/201512/147759-20151213123051919-1844797516.png" alt="" /></p>
<p>&nbsp;</p>
<p>开启Fiddler登陆网银后证书信息为：</p>
<p><img src="http://images2015.cnblogs.com/blog/147759/201512/147759-20151213123052637-506067430.png" alt="" /></p>
<p>由于我的机器已经信任过Fiddler的证书，我们可以发现，在开启了Fiddler后，和中行网银的通信证书变为了：DO_NOT_TRUST_FiddlerRoot。如果使用FiddlerCore，我们同样需要信任这个证书，相关的代码如下：</p>
<div class="cnblogs_Highlighter">
<pre class="brush:csharp;gutter:true;">public static bool InstallCertificate()
{
    if (!CertMaker.rootCertExists())
    {
        if (!CertMaker.createRootCert())
            return false;

        if (!CertMaker.trustRootCert())
            return false;
    }

    return true;
}</pre>
</div>
<h2>开始捕获</h2>
<p>使用这种方式，可以在不改变你现有代码的情况下，测试你的API返回结果是否正确。下面的例子是一个用FiddlerCoreAPI来测试SharePointOnline认证是否通过的例子。</p>
<div class="cnblogs_Highlighter">
<pre class="brush:csharp;gutter:true;">using Fiddler;
using Microsoft.SharePoint.Client;
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Net;
using System.Security;
using System.Text;
using System.Threading.Tasks;

namespace FiddlerCoreTest
{
    class Program
    {
        static void Main(string[] args)
        {
            ServicePointManager.ServerCertificateValidationCallback = (a, b, c, d) =&gt; true;

            FiddlerApplication.BeforeRequest += FiddlerApplication_BeforeRequest;
            FiddlerApplication.BeforeResponse += FiddlerApplication_BeforeResponse;
            FiddlerApplication.Startup(9898, FiddlerCoreStartupFlags.Default | FiddlerCoreStartupFlags.RegisterAsSystemProxy);
            try
            {
                ClientContext context = new ClientContext("https://domain.sharepoint.com");

                SecureString se = new SecureString();
                foreach (var cc in "password")
                {
                    se.AppendChar(cc);
                }

                var cre = new SharePointOnlineCredentials("user@domain.onmicrosoft.com", se);
                var cookie = cre.GetAuthenticationCookie(new Uri("https://domain.sharepoint.com"));
            }
            catch (Exception e)
            {

            }

            FiddlerApplication.Shutdown();
            Console.ReadLine();
        }

        static void FiddlerApplication_BeforeResponse(Session oSession)
        {
            //想如何改写Response信息在这里随意发挥了
            Console.WriteLine("BeforeResponse: {0}", oSession.responseCode);
        }

        static void FiddlerApplication_BeforeRequest(Session oSession)
        {
            //想如何改写Request信息在这里随意发挥了
            Console.WriteLine("BeforeRequest: {0}, {1}", oSession.fullUrl, oSession.responseCode);
        }
    }
}
</pre>
</div>
<p>　　</p>
