#### 安装WSL2方法

1.开启“适用于Linux的Windows子系统”与 “虚拟机平台”，以管理员打开cmd

```bash
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart

```

2.下载内核包 [wsl_update_x64.msi]https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi

3.执行

```
wsl --update 
```

> 如果不行切换到web那个命令更新

```
# 将 WSL 默认版本设置为 WSL 2
wsl --set-default-version 2
```

4. 下载发行版本

```
Invoke-WebRequest -Uri https://aka.ms/wslubuntu2004 -OutFile Ubuntu.appx -UseBasicParsing
```

> 开启梯子下载即可