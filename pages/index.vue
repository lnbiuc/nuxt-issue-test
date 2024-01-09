<script setup lang="ts">
import { MdCatalog, MdPreview } from 'md-editor-v3'
import 'md-editor-v3/lib/preview.css'

const id = 'preview-only'

const md = ref<string>('> 本文介绍一种基于windows远程桌面，使用任意设备远程连接到windows电脑的方式\n>\n> 至少需要1台服务器（本文以centos7.9操作系统的云服务器为例）\n>\n> 需要1台windows电脑（本文以内核版本为19045.2965的windows10家庭版为例）\n>\n> 任意远程桌面软件（本文以mac操作系统下的Minecraft Remote Destop为例）\n\n## 服务端配置\n\n### 安装\n\ncentos7.9操作系统下载以下文件\n\n[frp Github Release](https://github.com/fatedier/frp/releases)\n\n\n![image](https://user-pic-1308549476.cos.ap-nanjing.myqcloud.com/pic/13141684757099377.png/default)\n\n- 将文件上传到服务器用户目录，以root为例\n\n使用`tar -zxvf frp_0.48.0_linux_amd64.tar.gz`解压\n\n![image](https://user-pic-1308549476.cos.ap-nanjing.myqcloud.com/pic/6881684757119856.png/default)\n\n- 进入`frp_0.48.0_linux_amd64`目录修改配置文件\n\n其中以ftpc开头的文件均为客户端需要，服务端可以直接删除\n![image](https://user-pic-1308549476.cos.ap-nanjing.myqcloud.com/pic/58351684757133847.png/default)\n\n### 修改配置\n\n修改`frps.ini`\n\n```ini\n[common]\nbind_port = 7000\ndashboard_port = 7500\ntoken = Dd112211\ndashboard_user = violet\ndashboard_pwd = Dd112211\n```\n\n**bind_port**：客户端与服务器通信的端口，默认为7000\n\n**dashboard_port：frp**管理面板端口\n\n**token**：客户端与服务器连接的验证token\n\n**dashboard_user**：管理面板用户名\n\n**dashboard_pwd**：管理面板密码\n\n### 启动\n\n执行以下命令，此种方法需要服务器连接窗口一直开启，不推荐使用\n\n```shell\n./frps -c ./frps.ini\n```\n\n### 自启动\n\n使用system工具实现开机自启和后台运行\n\n> 以下内容来自[frp官方文档](https://gofrp.org/docs/setup/systemd/)\n\n- 安装system工具\n\n```shell\nyum install systemd\n```\n\n- 修改启动配置文件\n\n```\nvi /etc/systemd/system/frps.service\n```\n\n- 文件写入内容\n\n```ser\n[Unit]\n# 服务名称，可自定义\nDescription = frp server\nAfter = network.target syslog.target\nWants = network.target\n\n[Service]\nType = simple\n# 启动frps的命令，需修改为您的frps的安装路径\nExecStart = /root/frp_0.48.0_linux_amd64/frps -c /root/frp_0.48.0_linux_amd64/frps.ini\n\n[Install]\nWantedBy = multi-user.target\n```\n\n- 启动命令\n\n```shell\n# 启动frp\nsystemctl start frps\n# 停止frp\nsystemctl stop frps\n# 重启frp\nsystemctl restart frps\n# 查看frp状态\nsystemctl status frps\n```\n\n- 设置开机自启\n\n```shell\nsystemctl enable frps\n```\n\n- 服务端开启端口7000，7500，7001（后面会用到）\n\n```shell\nsudo firewall-cmd --permanent --zone=public --add-port=7000/tcp\nsudo firewall-cmd --permanent --zone=public --add-port=7500/tcp\nsudo firewall-cmd --permanent --zone=public --add-port=7001/tcp\n\n# 重启防火墙\nsudo firewall-cmd --reload\n```\n\n至此，服务端配置完毕\n\n## 客户端\n\n### 安装\n\n[frp Github Release](https://github.com/fatedier/frp/releases)\n\n![image](https://user-pic-1308549476.cos.ap-nanjing.myqcloud.com/pic/59441684757152013.png/default)\n\n下载后解压到`D:\\Program Files\\frp_0.48.0_windows_amd64`\n\n以frps开头的文件均为服务端所需要，可以直接删除\n\n### 配置\n\n修改frpc.ini\n\n```ini\n[common]\nserver_addr = <此处替换为你的服务器ip>\nserver_port = 7000\ntoken = Dd112211\n\n[rdp]\ntype = tcp\nlocal_ip = 127.0.0.1           \nlocal_port = 3389\nremote_port = 7001  \n```\n\nserver_addr：服务区地址\n\nserver_port：客户端同服务区交互端口\n\ntoken：连接到服务端所需token\n\ntype：连接类型：udp或tcp\n\nlocal_ip：本地访问ip\n\nlocal_port：本地访问端口，windows远程桌面默认访问端口为3389\n\nremote_port：远程访问端口\n\n> 此处配置文件的意思时，使用server_addr:remote_port连接到local_ip:local_port，相当与访问<此处替换为你的服务器ip>:7001 == 访问127.0.0.1:3389\n\n### 启动\n\n![image](https://user-pic-1308549476.cos.ap-nanjing.myqcloud.com/pic/33931684757171733.png/default)\n\n进入powershell，使用以下命令启动\n\n```shell\n./frpc.exe -c ./frpc.ini\n```\n\n![image](https://user-pic-1308549476.cos.ap-nanjing.myqcloud.com/pic/11441684757181534.png/default)\n\n之后使用其他设备通过<你的服务器ip>:7001，就可以远程访问到windows设备了\n\n如果此时无法访问，请按照以下步骤检查\n\n### 自启动\n\n程序目录新建文件`start.bat`\n\n```bat\n@echo off\n:home\n\n./frpc.exe -c ./frpc.ini\n\ngoto home\n```\n\n打开任务计划程序\n\n![image](https://user-pic-1308549476.cos.ap-nanjing.myqcloud.com/pic/36741686924391076.png/default)\n\n![image](https://user-pic-1308549476.cos.ap-nanjing.myqcloud.com/pic/68801686924484760.png)\n\n![image](https://user-pic-1308549476.cos.ap-nanjing.myqcloud.com/pic/59551686924545528.png)\n\n程序或脚本填start.bat文件位置，例如:`D:\\Program Files\\frp_0.48.0_windows_amd64\\start.bat`\n\n起始于填`start.bat`所在文件夹路径，例如：`D:\\Program Files\\frp_0.48.0_windows_amd64`\n\n![image](https://user-pic-1308549476.cos.ap-nanjing.myqcloud.com/pic/18471686924639504.png)\n\n## 错误检查\n\n### 服务器端口是否开启\n\n ```shell\n sudo firewall-cmd --list-ports\n ```\n\n### 客户端电脑是否允许3389端口\n\n进入防火墙高级设置，新建入站规则\n\n![image](https://user-pic-1308549476.cos.ap-nanjing.myqcloud.com/pic/19891684757189055.png/default)\n添加允许3389端口\n\n![image](https://user-pic-1308549476.cos.ap-nanjing.myqcloud.com/pic/40961684757195401.png/default)\n### 客户端是否开启远程桌面\n\n![image](https://user-pic-1308549476.cos.ap-nanjing.myqcloud.com/pic/45481684757202809.png/default)\n如果以上检查无误，仍然无法连接，请到frp github查看相关issue\n\n> 注：windows家庭版无法使用被远程桌面连接，专业版可以被远程桌面连接如果你是windows家庭版，需要进行以下配置\n\n## windows家庭版配置\n\nwindows家庭版无法通过windows10设置开启远程桌面，需要使用补丁（此方法在windows版本19045.2965可用，高于此版本可能导致失效），如果该方法失效，请到github issue寻找最新的解决方案。\n\n> 推荐升级到windows专业版\n\n[Github Issue](https://github.com/stascorp/rdpwrap/issues/999)\n\n[该方法原作者](https://www.cnblogs.com/hwajie/p/12447224.html)（此方法已在2020年失效）\n\n2023年以后的请看以下方法，穿越回去的可以忽略\n\n### 下载软件\n\n[RDPWrap-v1.6.2.zip下载地址(作者StasM Corp)](https://github.com/stascorp/rdpwrap/releases/download/v1.6.2/RDPWrap-v1.6.2.zip)\n\n[autoupdate-v07.09.2019.zip下载地址(作者asmtron)](https://github.com/stascorp/rdpwrap/files/3588284/autoupdate-v07.09.2019.zip)\n\n### 安装1\n\n将RDPWrap解压到`C:\\Program Files\\RDP Wrapper`\n\n使用管理员身份运行`install.bat`\n\n等待安装结束，点击`RDPConf.exe`\n\n![image](https://user-pic-1308549476.cos.ap-nanjing.myqcloud.com/pic/46711684757213020.png/default)\n左上角`Wrapper state`，`Service state`，`Listener state`应该为全绿状态，此时远程桌面可以正常访问，如果有红色，请看下面方法\n\n### 安装2\n\n解压autoupdate到`C:\\Program Files\\RDP Wrapper`目录下，使用管理员身份运行`update.bat`\n\n以管理员身份打开powershell，运行以下命令\n\n```shell\nnet stop termservice\n\nnet start termservice\n```\n\n再次打开`RDPConf.exe`查看左上角状态是否为绿，如果不是继续下一步\n\n打开`rdpwrap.ini`，如果没有自行创建，删除原文件内容，粘贴以下内容（从下方链接获取）\n\nhttps://raw.githubusercontent.com/sebaxakerhtc/rdpwrap.ini/master/rdpwrap.ini\n\n之后再执行：以管理员身份打开powershell，运行以下命令\n\n```shell\nnet stop termservice\n\nnet start termservice\n```\n\n再次打开`RDPConf.exe`查看左上角状态是否为绿，如果此时状态为绿的，再次启动客户端frp即可`./frpc.exe -c ./frpc.ini`，如果还是不行，到[Github](https://github.com/stascorp/rdpwrap/issues/999)寻找最新方法\n\n\n\n至此，你应该可以使用任意软件通过服务器远程到windows电脑了。",')

const color = useColorMode()

const theme = ref<'light' | 'dark'>(color.value === 'dark' ? 'dark' : 'light')

watchEffect(() => {
  theme.value = color.value === 'light' ? 'light' : 'dark'
})

let scrollElement: string | HTMLElement | undefined

onMounted(() => {
  scrollElement = document.documentElement
  theme.value = color.value === 'light' ? 'light' : 'dark'
})
</script>

<template>
  <div class="flex flex-row justify-center text-left">
    <MdPreview id="mdPreview" :theme="theme" :editor-id="id" :model-value="md" class="max-w-[1000px]" />
    <div class="catalog relative flex flex-col">
      <MdCatalog :editor-id="id" :theme="theme" :scroll-element="scrollElement" class="max-h-[100vh]" />
      theme: {{ theme }}  color: {{ color.value }}
    </div>
  </div>
</template>

<style>
.catalog {
  position: sticky;
    top: 10px;
    max-height: 100vh;
    overflow: auto;
}
</style>
