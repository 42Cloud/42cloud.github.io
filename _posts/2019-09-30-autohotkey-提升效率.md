---
title: autohotkey 提升效率
date: 2019-09-30 15:39:27
tags:
---

# 安装

通过 chocolatey 包管理工具安装，方便。

# 需求

自动连接校园网（有线或者无线），并启动 ss

# 实现

相关命令：rasdial, netsh wlan

## 连接校园有线网

```C
#IfWinActive, 桌面整理
!c::

LANname := "以太网"
WIFIname := "ZJUWLAN"
ScanLoopInMinutes := 2
DelayedLoop := 20
DisplayALerts := True


CheckAdapterStatus(str, delimter, adapterName)
{
    adpaterState = 2    ; adpater does not exist
    inputStrSplit := StrSplit(str, delimter)
    Loop % inputStrSplit.MaxIndex()
    {
        this_str := inputStrSplit[A_Index]
        if (RegExMatch(this_str, adapterName))
        {
            adpaterState = 1    ; adpater found
            ;MsgBox, %this_str%
            if RegExMatch(this_str, "已连接") 
                adpaterState = 0    ; connected
        }
    }
    return adpaterState
}

netshCommand(shellID, adapterName, displayALerts)
{
    shellID.exec("netsh interface set interface """ . adapterName . """ Disable")
    if displayALerts
        MsgBox , 0x40310, AHK, Wifi Adapter Disabled, 5 
    sleep 10000
    shellID.exec("netsh interface set interface """ . adapterName . """ Enable")
    if displayALerts
        MsgBox , 0x40310, AHK, Wifi Adapter Enabled, 5 
}

shell := comobjcreate("wscript.shell")
cmd = powershell.exe -command "netsh interface show interface"
netshStatus := shell.exec(cmd).stdout.readall()
;MsgBox, %netshStatus%
sLAN := CheckAdapterStatus(netshStatus, "`r`n", LANname)
sWIFI := CheckAdapterStatus(netshStatus, "`r`n", WIFIname)

;MsgBox, %sLAN%{+}%sWIFI%
;Return

if( sLAN = 0) ; connect zju_vpn
{
    RunWait, rasdial.exe vpn username password, C:\Windows\System32, Hide|UseErrorLevel
    if (ErrorLevel) {
        TrayTip, 啟動 VPN 連線, 連線至 VPN 發生錯誤 (%A_LastError%), 3, 3
    } else {
        TrayTip, 啟動 VPN 連線, 成功連線至 VPN  (%A_LastError%), 3, 1
    }
}

if( sLAN = 1) ; connect zju_wifi
{
    Runwait, powershell -Command "netsh wlan connect ZJUWLAN"
    ;Runwait, powershell -NoExit -Command "netsh wlan connect ZJUWLAN"
    ;Runwait, %comspec% /k netsh wlan connect ZJUWLAN
    ;结合 chrome 记住密码以及 vimium 插件进行以下操作
    Runwait, http://net.zju.edu.cn
    Sleep 2000
    #ifWinActive, 浙江大学认证系统 - Google Chrome
    Send, {f down}{f up}
    Sleep 200
    Send, {j down}{j up}
    Sleep 100 
}

; run ss-h on desktop
path = %A_Desktop%\ss-h.lnk
Run, %path%
Return

if ( sLAN = 1 and sWIFI = 1)          ; both adapter are found, but not connected
{
    if DisplayALerts
        MsgBox , 0x40310, AHK, I am resetting WiFi adpater, 4

    ;netshCommand(shell, WIFIname, DisplayALerts)
} 
else if ( sLAN = 0 and sWIFI = 0)          ; both adapter connected -> Disabled WiFi adapter
{
    shell.exec("netsh interface set interface """ . adapterName . """ Disable")
}
else
{
     ScanLoopInMinutes := 1
}

if sLAN = 2
    if DisplayALerts
        MsgBox , 0x40310, AHK, LAN adpater name does not match, 5
if sWIFI = 2
    if DisplayALerts
        MsgBox , 0x40310, AHK, WIFI adpater name does not match, 5

Return
```

善用 window spy 来确定窗口
## 无线连接


## 中文编码问题

参考：https://zhuanlan.zhihu.com/p/19712731

使用 utf8-BOM 编码

# 参考

- [windows用命令行连接到无线WIFI网络](https://blog.csdn.net/qiantianyuan001/article/details/79873718)
- [DOS批处理笔记本连接WIFI和有线网络](https://blog.csdn.net/guyue35/article/details/49099469)
- [如何透過命令列工具快速連接 Windows 內建的 VPN 連線](https://blog.miniasp.com/post/2019/02/12/Dial-up-VPN-connection-using-CLI-tool-rasdial)
- [Automating browser actions with Vimium and AutoHotKey](http://codebetter.com/kylebaley/2010/06/23/automating-browser-actions-with-vimium-and-autohotkey/)
- [Win下最爱效率神器:AutoHotKey](https://www.jeffjade.com/2016/03/11/2016-03-11-autohotkey/)
- [Connect to WiFi if Ethernet disconnected](https://www.autohotkey.com/boards/viewtopic.php?t=63858&p=273725)