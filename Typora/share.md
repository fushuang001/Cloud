[TOC]



> simply topology:
>
> F5 with 10.215.136.x — L213 — Spine — remote leaf — EPx
>
> from the log analysis, currently suspect EPx use ip 10.215.136.x generate packets, send to ACI network and this lead to L213 EP 10.215.136.x `IP move` event.



## 1. Check L213 epm log to identify remote EPx

### 1). find EPx mac address

```diff
show endpoint ip 10.215.136.x detail  // replace 10.215.136.x with the impacted IP address
show system internal epm endpoint ip 10.215.136.x
- if 10.215.136.x now status is B(B for bounce), then you will find the Interface Tunnelx
- when you find the interface tunnelx, you can bypass following epm, or oldolg check.
vsh_lc -c 'show system internal epmc endpoint mac E.E.E' // replace E.E.E with remote mac

cd /var/log/dme/log/
grep -Hni -C50 10.215.136.x epm* // replace 10.215.136.x with the impacted IP address
```

example output

![image-20190802131634654](/Users/fushuang/Desktop/MD_Note/Workday Focus/MD Pictures/image-20190802131634654.png)

==if the epm/epmc log already got flushed, then please check with oldlog==

```
cd /var/log/dme/oldlog/
zegrep -Hni -C50 10.215.136.x epm*
```



### 2). find L213 tunnel index

```
show interface snmp-ifindex | grep 0x1801006b      // replace 0x1801006b with above tep_ip_tun_ifindex, get the output of tunnel x
show interface tunnel x     // replace x with above output id, find the dest ip
```

example output

![image-20190802133400097](/Users/fushuang/Desktop/MD_Note/Workday Focus/MD Pictures/image-20190802133400097.png)



## 2. Check APIC to find remote Leaf x

```
bash
moquery -c dhcp.Client -f 'dhcp.Client.ip=="10.219.132.100”’    // replace 10.219.132.100 with above output of "Tunnel destination" ip address
```

example output

![image-20190802133434934](/Users/fushuang/Desktop/MD_Note/Workday Focus/MD Pictures/image-20190802133434934.png)

## 3. Collect remote leaf x EP/VLAN log

```
show endpoint ip 10.215.136.x detail // replace 10.215.136.x with the impacted IP address
show system internal epm endpoint ip 10.215.136.x
vsh_lc -c "show system internal epmc endpoint ip 10.215.136.x"
show vlan extend
```



## 4. resume service



## 5. Collect L213 and remote leaf x epm log

### 1). L213

```
scp <username>@<leaf213-ip>:/var/log/dme/log/epm-trace.txt .
scp <username>@<leaf213-ip>:/var/log/dme/log/epmc-trace.txt .
tar -czvf /tmp/epm_oldlog.tgz /var/log/dme/oldlog/epm*
scp <username>@<leaf213-ip>:/tmp/epm_oldlog.tgz .
```

### 2). remote leaf x

==if VPC pair, then collect on both VPC parts==

```
scp <username>@<leafxxx-ip>:/var/log/dme/log/epm-trace.txt .
scp <username>@<leafxxx-ip>:/var/log/dme/log/epmc-trace.txt .
tar -czvf /tmp/epm_oldlog.tgz /var/log/dme/oldlog/epm*
scp <username>@<leafxxx-ip>:/tmp/epm_oldlog.tgz .
```





APIC Temporary Root Password Generator

```
1. the token available for all the devices in same ACI Fabric, for APIC/Spine/Leaf
2. life time 60 minutes
3. ALWAYS told customer use with caution

```



login APIC GUI, navigate to `Operations - Visibility & Troubleshooting`, we will use the GUI Tool to confirm traffic forwarding path inside ACI fabric.

**Supported following:**

```
1. Endpoint to Endpoint
2. Endpoint to External IP
3. External IP to Endpoint
```



Provide the `Targets Source & Destion`, could be `MAC, IPv4, IPv6, or VM name`.

```
Embedded Logic Analyzer Module (ELAM) , Nexus 5/6/7/9 all support it, ACI Leaf/Spine based on N9K, support ELAM as well; 
ELAM could capture ONE packet from ASIC, provide the details report of packets, forwarding decision, drop reason.
```













# save your time when udpate case note

  https://scripts.cisco.com/ui/use/add_case_note?dev=false



## coding 原因：

按照最新计算工作量的方式，每天早晨及时 update case note. <!--会对 performance 有积极影响-->
正常来说，每天到办公室以后，打开 CSOne，然后打开具体的 case，等 case note 加载完成，贴一条 note ，表示已经开始当天的工作。
这个过程需要 2-3 分钟。
想要简化这个过程，用更少的时间更灵活的去完成 update case note 这件事。

## 相似工具：

Quicker CSOne Backlog

## script 演示：

  1. **case number** 可以从outlook 邮件获取。一般只要电脑不关机，outlook 就很容易打开，CSOne 需要重新登录。
  2. **whats in the note**, 具体的 case note;   <!--支持 GCI-->
  3. **type of note**, 支持 Problem Description, Current Status, KT Action Plan, Other, Phone Log, Email In
  4. **Internal or External**

  ==Run Task==，一般 10 秒内完成。
  演示一下贴 GCI 的过程；<!--User A 用 script 贴 case note，显示的 CEC ID 就会是 User A-->
  GCI update比较快，邮件 Email In 慢一些，60 秒。
  不过建议是，保存书签，然后每天早晨更轻松地 update case note。











EP Learning Issue

大纲，包括什么，直接搜，或者跳转

做之前，找一下 EPM 的 NPI 看看

1. what is EP, what is COOP
2. EP Type
3. EP learning source，PPT
4. EPM & EPMC, TZ 和 PPT
5. checklist, 笔记
   1. EP Location, PPT moquery, APIC show, GUI EP Tracker
   2. EPM & EPMC log collect
   3. CLIs
6. 参考 link

https://techzone.cisco.com/t5/Application-Centric/EPM-EPMC-Learning-amp-Endpoint-Management-in-ACI/ta-p/882281

https://techzone.cisco.com/t5/Application-Centric/ACI-EP-Move-Flap-Detection/ta-p/889910

https://techzone.cisco.com/t5/Application-Centric/Understanding-Endpoint-Learning-via-EPM-TRACE-and-EPMC-TRACE/ta-p/844271

https://techzone.cisco.com/t5/Application-Centric/Troubleshooting-EndPoint-Learning-using-epm-trace-txt/ta-p/786846

[http://frankenberry.cisco.com/ppt_files/converted/1397362483%20(Web)/](http://frankenberry.cisco.com/ppt_files/converted/1397362483 (Web)/)

[http://frankenberry.cisco.com/ppt_files/converted/1397362473%20(Web)/](http://frankenberry.cisco.com/ppt_files/converted/1397362473 (Web)/)

[http://frankenberry.cisco.com/ppt_files/converted/1397362475%20(Web)/](http://frankenberry.cisco.com/ppt_files/converted/1397362475 (Web)/)

[http://frankenberry.cisco.com/ppt_files/converted/1397430409%20(Web)/](http://frankenberry.cisco.com/ppt_files/converted/1397430409 (Web)/)

[https://cxtraining.cisco.com/ppt_files/converted/1397369231%20(Web)](https://cxtraining.cisco.com/ppt_files/converted/1397369231 (Web))



```diff
- Ask the customer to do KT first
What is Expect behavior
Which object in wrong status
When did it happen
Compare to the right objects

- Find the EP Location, Leaf/Path/VRF/BD
+ APIC
GUI EP Tracker
moquery -c epmMacEp
moquery -c epmIpEp
show endpoint <mac/ip>

+ Leaf
show endpoint <mac/ip/interface/vrf> detail
show system internal epm endpoint ip x.x.x.x
show system internal epm endpoint mac E.E.E
vsh_lc
show system internal epmc endpoint mac E.E.E

+ Spine
show coop internal info repo ep | grep -B 50 -A 15 <ip-mac-address> 
show coop internal info ip-db | grep -A 15 x.x.x.x'

- CLIs to check EP status, update/delete/move event
show version
show module
show endpoint details
show vrf all
show system internal epm vrf all
show vlan
show vlan extend
show interface ex/y switchport
show interface ex/y
show port-channel summary
show vpc
show interface snmp-ifindex
show interface tunnel 1-30
show isis dteps vrf overlay-1
show system internal epm global
show system internal epm endpoint all
show system internal epm endpoint ip x.x.x.x
show system internal epm endpoint mac E.E.E
show system internal epm counters all | grep 'Number of flush request rcvd'  <!--if TCN,flush-->
show mcp internal info vlan x <!--Last TC flush at-->
show spanning-tree vlan x interface ex/y details
show mcp internal event-history trace detail | more
show system internal epm vlan all detail
less /var/log/dme/log/eltm_trace.log.txt  <!--查看实时的 eltm trace 信息-->

+ vsh
show vlan internal info <!--BD, FD vlan-->
clear system internal epm endpoint key vrf <vrf-name> ip <ip-address> 
<!--exec in vsh mode, refers to CSCvf48852-->

+ vsh_lc
show system internal epmc endpoint ip x.x.x.x
show system internal epmc endpoint all
show system internal eltmc info vlan brief
show system internal eltmc info vlan access_encap_vlan x  <!--x 是对 server 的 encap vlan-->
show system internal eltmc info vlan <vlan-id> detail 

- ELAM/tcpdump, if necessary

```



## Typora 常用技巧和快捷键

1. 列表 - [ ]; 目录 <kbd>toc</kbd>
2. ==highlight==, <kbd>Command</kbd> + <kbd>Shift</kbd> + <kbd>H</kbd>
3. **加粗** <kbd>Control</kbd> + <kbd>B</kbd>
4. <!--注释-->, <kbd>Control</kbd> + <kbd>-</kbd>
5. [超链接，先复制 link](http://support.typora.io/HTML/), 然后 <kbd>Command</kbd> + <kbd>K</kbd>
6. 表格， |item1|item2|
7. <span style='color:red'>红色字体</span>, <span style='color:green'> 绿色字体</span>
8. <span style='background:black;color:white'>背景色黑--字体白, 可以自定义的 高亮字体</span>
9. <span style='font-size:2.5rem'>大号字体</span>
10. <span style='font-size:1.5rem;background:green;color:white'>组合大号字体、背景色</span>
11. HTML Block，可以临时隐藏某一部分

<details>
  <summary> 在这里提问，比如需要思考的，或者回答起来比较多的内容不需要一直展示的</summary>
  在这里写答案
</details>

12. ~~划掉错误的笔记~~ <!--strike-->,  <kbd>Control</kbd> + <kbd>Shift</kbd> + <kbd>`</kbd>

13. [^ Note]: 脚注或者提醒，不过用的不多

14. 引用，应该能替代 13. <kbd>></kbd> 

> 适合比较短的笔记

14. 分割线 <kbd>---</kbd>
15. 代码块，diff，<kbd>```</kbd> + <kbd>diff</kbd>

```diff
- red
+ green
```







1. Pod2 计算节点，如 leaf 201，没有 66.66 的路由

   ​	`可能的原因是什么`

---

2. `Pod2的border leaf，如 BL307 ，关于 66.66 的路由，都可以从哪里学到？`

​	从 C3850-2 的 L3out, EIGRP邻居学到；

​	从 Pod1的 BL305-306，跨BGP EVPN/RR 传递过来；

​	BL307 把 66.66 路由从 EIGRP 重定向到 BGP EVPN，自己产生的；

---

3. `Pod2 计算节点 leaf 201关于 66.66 的路由，应该从哪里学到？`

​	客户认为compute leaf 关于外部路由肯定是从 border leaf，也就是 Pod2 BL307

​	实际是从 Pod1 BL305-306学到。

---

4. `ulti-Pod EVPN 如何传递路由(EVPN type5)、EP 的 MAC/IP(EVPN type2) 信息

   `Spine – IPN 有没有 BGP 邻居`

   `两个 pod 的 Spine 是什么关系`

   



## ![image-20200306141857264](/Users/fushuang/Desktop/MD_Note/Workday Focus/MD Pictures/image-20200306141857264.png)



```diff
- cd /Users/fushuang/Documents/GitHub/fushuang001.github.io
- jekyll serve
    Server address: http://127.0.0.1:4000/
  Server running... press ctrl-c to stop.

+ 文章中展示的图片，先 upload 到 /img 文件夹，然后找到图片点 Download, 贴对应的 link。直接右键复制的 link 好像是加载失败了。
图片应该还可以尝试 iPic 图床，先压缩、再上传
网站图标 favicon，尺寸 32 * 32
遗留问题：catalog 目录显示不正常。

目前关闭了评论、留言功能

首页的信息，由 index.html 决定，code 很容易理解 http://jekyllcn.com/docs/pages/
blog 信息放在 post 文件夹，注意事项可以参考 http://jekyllcn.com/docs/posts/


- 简单的静态页面、可以分级目录的
https://fushuang001.github.io/ 可以定制化的主页、post 不同的 blog
https://fushuang001.github.io/forest.io/ 是目前我自己的简历，网页主题可选；可以用来发布个人课程信息
https://fushuang001.github.io/forest.io/0203.md  是测试页面，网页主题不可选，是文本格式


https://18642870203.github.io
最开始修改好 _config.yml 之后，首页并没有展示出来。后来发现模板首页是 index.html 来定义的，需要修改 index.html 文件。

如果网站修改好了没有生效，去看一下 Settings - Github pages, 是不是有报错.
186的一直有报错，修改、测试之后无效，直接从 https://github.com/kaeyleo/jekyll-theme-H2O 这里，_config.yml 编辑，全部复制过来，覆盖 186 的，之后再修改。

在186 里面加入了一个 About，
	layout:default 无法加载图片，页面也很难看。试着 layout:post, bingo!
  <img src="11ha.jpg" alt="小女孩" /> 可以在 .html 中插入图片，比如微信二维码
subtitle 也是好用的;
插入表格feature get;
https://18642870203.github.io/test.html 用来测试子页面，test.html 不妨到_post 文件夹；这个信息并不会主动发布出来，也就是每个人只知道自己的 test 应该怎么替换； 表格 get; test.md文件不可以
```









- [ ] 

## 尚艺熹/普拉提/健康会馆

- [ ] 录屏
- [ ] GitHub pages 做网站的 server，登录
- [ ] 网站主页已经写好，不需要改什么东西
- [ ] 如果需要发布新的文章/推送，就在 _post 里面新建一个 .md 文件；演示一下
- [ ] 新建学员信息的方式 .html 文件，直接复制 test 的信息去修改
- [ ] 重点讲一下表格修改的方式
- [ ] 不要轻易删除，我这里有备份
- [ ] 录屏发给文哥





