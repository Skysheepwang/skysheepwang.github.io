---
title: 使用Jmeter工具对微信公众号后端进行压力测试
date: 2021-12-08 19:48:53
tags: 
  - Stress Testing
  - JMeter
categories:
  - [Software Engineering, Testing]
---

<h2 id="一-jmeter环境配置"><a class="markdownIt-Anchor" href="#一-jmeter环境配置"></a> 一、JMeter环境配置</h2>
<ol>
<li>
<p>根据官方文档进行安装</p>
<p>网址：<a href="http://jmeter.apache.org/" target="_blank" rel="noopener">http://jmeter.apache.org/</a></p>
<p>解压缩后在/bin中运行jar包即可启动GUI</p>
</li>
<li>
<p>添加测试计划</p>
<p>（1）添加线程组</p>
<p>测试计划–右键–添加–线程–线程组</p>
<p>（2）添加取样器</p>
<p>根据需求选择相应的取样器，本次共使用3个Html取样器，分别对应<strong>用户绑定</strong>（即身份认证）、<strong>抢票</strong>、<strong>查票</strong>操作。</p>
<p>（3）添加循环控制器</p>
<p>上述3个操作应当为一个用户的基本操作流程，故应当1个线程先后进行，因此</p>
<p>线程组–右键–添加–逻辑控制器–循环控制器，再把3个取样器放进循环控制器中即可。</p>
<p>（4）并发量设置</p>
<p>线程组–线程属性</p>
<p>设置线程数（用户数量），Ramp-Up时间（测试完成的最大时间）</p>
</li>
<li>
<p>添加结果分析器</p>
<p>（1）察看结果树</p>
<p>用于查看每一次执行的结果。</p>
<p>取样器结果会显示线程的详细信息，数据包信息等；</p>
<p>请求会显示Request的body和headers内容；</p>
<p>响应数据显示Respond的body和headers内容。</p>
<p>（2）聚合报告</p>
<p>显示所有线程的统计结果。</p>
</li>
</ol>
<h2 id="二-post内容"><a class="markdownIt-Anchor" href="#二-post内容"></a> 二、POST内容</h2>
<ol>
<li>
<p>直接对服务器的api POST</p>
<p>在这个地方我卡了很久，服务器一直返回400 bad request Error。最终发现是Content-Type的问题，本次传输的应是JSON格式，故Content-Type应为application/json</p>
<p>在JMeter中手动改POST的Content-Type的方法为：取样器–右键–添加–配置元件–HTTP信息头管理器，再添加名称为Content-Type，值为所需值即可。</p>
</li>
<li>
<p>对微信公众号接口POST</p>
<p>这里主要参考微信官方公众平台技术文档，微信向后台传输xml格式数据，根据所需要测试的内容，本次为特定按钮，则参考“自定义菜单事件”中的内容，设置相应的参数即可。需要注意的是fromuser是openid，需要在服务器中预设；同时，不同的按钮对应的事件类型要匹配。</p>
<p>注：不要随便在xml中插入空格！</p>
</li>
</ol>
<h2 id="三-一些细节的处理"><a class="markdownIt-Anchor" href="#三-一些细节的处理"></a> 三、一些细节的处理</h2>
<ol>
<li>
<p>返回乱码的处理</p>
<p>问题出在JMeter默认的编码方式为ISO-8859-1 ，不支持中文，因此需要：</p>
<p>取样器–右键–添加–后置处理器–Bean Shell Post Processor</p>
<p>然后在其中Script中输入prev.setDataEncoding(“UTF-8”); 改变编码方式即可。</p>
</li>
<li>
<p>动态改变POST的body</p>
<p>本次项目需要每个模拟用户使用不同的openid，故需要在POST的body中加入脚本，动态改变openid的值：</p>
<p><code>${__V(&quot;openid${__threadNum}&quot;)}</code></p>
<p>使用如上的代码即可将线程号附在模拟openid后，从而实现每个模拟用户的信息独立。</p>
</li>
</ol>
<h2 id="四-几个重要测试指标"><a class="markdownIt-Anchor" href="#四-几个重要测试指标"></a> 四、几个重要测试指标</h2>
<p>使用取样器–右键–添加–监听器–聚合报告</p>
<p>在聚合报告中，尤其需要注意的有：</p>
<ul>
<li><strong>Average</strong>：单个Request的平均响应时间，计算方法是总运行时间除以发送到服务器的总请求数，对应图形报表中的平均值。(单位是毫秒)</li>
<li><strong>Error%</strong>：本次测试中出错率，请求的数量/请求的总数。</li>
<li><strong>Throughput</strong>：吞吐量，默认情况下表示每秒完成的请求数。（单位：请求/秒）</li>
<li><strong>Received</strong> <strong>KB/Sec</strong>：每秒从服务器接收到的数据量</li>
</ul>
<h2 id="五-参考资料"><a class="markdownIt-Anchor" href="#五-参考资料"></a> 五、参考资料</h2>
<p>[1]<a href="https://www.cnblogs.com/tangmaokai/p/5830344.html" target="_blank" rel="noopener">https://www.cnblogs.com/tangmaokai/p/5830344.html</a></p>
<p>[2]<a href="https://blog.csdn.net/go_pig/article/details/79570799" target="_blank" rel="noopener">https://blog.csdn.net/go_pig/article/details/79570799</a></p>
<p>[3]<a href="http://www.51testing.com/html/28/116228-238479.html" target="_blank" rel="noopener">http://www.51testing.com/html/28/116228-238479.html</a></p>
<p>[4]<a href="https://mp.weixin.qq.com/wiki?t=resource/res_main&amp;id=mp1421140454" target="_blank" rel="noopener">https://mp.weixin.qq.com/wiki?t=resource/res_main&amp;id=mp1421140454</a></p>

