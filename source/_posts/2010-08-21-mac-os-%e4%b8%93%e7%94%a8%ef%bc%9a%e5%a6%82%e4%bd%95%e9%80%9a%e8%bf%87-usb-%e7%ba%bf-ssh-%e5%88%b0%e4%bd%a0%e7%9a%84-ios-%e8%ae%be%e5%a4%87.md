--- 
categories: 
  - iPhone
  - Mac
comments: true
layout: post
published: true
status: publish
tags: 
  - iOS
  - Mac
  - SSH
  - USB
title: "Mac OS 专用：如何通过 USB 线 SSH 到你的 iOS 设备"
type: post
---
<p>本文在于教会大家如何使用，不在于解释原理, 方法源自于此：<a href="http://iphonedevwiki.net/index.php/SSH_Over_USB">SSH Over USB</a>。这里做了一个开机启动项方便大家使用</p>
<p>1. 下载<a href="http://drp.ly/1zV69a">这里的</a>打包文件</p>
<p>2. 解压，把 iPhoneSSH 文件夹放到任意合适位置，记住文件夹的路径</p>
<p><img style="display:block;margin-left:auto;margin-right:auto;" title="Screen shot 2010-08-21 at 10.05.37 PM.png" src="http://codeleaks.files.wordpress.com/2010/08/screen-shot-2010-08-21-at-10-05-37-pm1.png" border="0" alt="Screen shot 2010-08-21 at 10.05.37 PM.png" width="596" height="499" /></p>
<p>3. 打开 com.overboming.iphonessh.plist 文件，修改该文件中路径到刚才的地方，我这里放到的是 ~/code/ios/ 目录下，所以将 ProgramArguments 下的命令设置为</p>
<pre>	/Users/malic/code/ios/iphonessh/python-client/tcprelay.py
</pre>
<p><img style="display:block;margin-left:auto;margin-right:auto;" title="Screen shot 2010-08-21 at 10.05.56 PM.png" src="http://codeleaks.files.wordpress.com/2010/08/screen-shot-2010-08-21-at-10-05-56-pm1.png" border="0" alt="Screen shot 2010-08-21 at 10.05.56 PM.png" width="579" height="458" /></p>
<p>4. 然后将 com.overboming.iphonessh.plist 放置到 ~/Library/LaunchAgents/ 目录下</p>
<p>5. 重启系统（或者可以手动执行 plist 中输入的命令）后插入任何装过 OpenSSH 的 iOS 设备，就可以在命令行中通过</p>
<pre>	ssh -l root -p 2222 localhost
</pre>
<p>登入你的设备了，enjoy！</p>
<p> </p>
<p><img style="display:block;margin-left:auto;margin-right:auto;" title="Screen shot 2010-08-21 at 10.06.07 PM.png" src="http://codeleaks.files.wordpress.com/2010/08/screen-shot-2010-08-21-at-10-06-07-pm1.png" border="0" alt="Screen shot 2010-08-21 at 10.06.07 PM.png" width="609" height="420" /></p>
<p>此外，还可以通过 Transmit 4 自带的将sftp mount为文件系统的功能将 iPhone 的整个文件系统 mount 为一个 Volume，非常方便，而且速度和比通过 Wireless 的方式快很多，也不用担心锁屏断线了 :)</p>
<p><img style="display:block;margin-left:auto;margin-right:auto;" title="Screen shot 2010-08-21 at 9.59.16 PM.png" src="http://codeleaks.files.wordpress.com/2010/08/screen-shot-2010-08-21-at-9-59-16-pm1.png" border="0" alt="Screen shot 2010-08-21 at 9.59.16 PM.png" width="402" height="205" /> <img style="display:block;margin-left:auto;margin-right:auto;" title="Screen shot 2010-08-21 at 10.06.15 PM.png" src="http://codeleaks.files.wordpress.com/2010/08/screen-shot-2010-08-21-at-10-06-15-pm1.png" border="0" alt="Screen shot 2010-08-21 at 10.06.15 PM.png" width="596" height="499" /></p>
