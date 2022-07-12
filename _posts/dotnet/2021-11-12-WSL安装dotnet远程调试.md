---
   layout: default
   title: 安装Jekyll
   tags: Jekyll
---

# {{ page.title }}
{{ page.date | date: "%Y-%m-%d" }}

# 简介

在内网开发时，Visual Studio无法在WSL中自动安装远程调试环境，下面介绍手动安装方法。

## 安装方法

1. 下载安装脚本
   https://aka.ms/getvsdbgsh
2. 修改下载的`GetVsDbg.sh`参考下列代码，注释所有下载相关的代码。
   ```sh
   download()
   {
    if [ "$__UseZip" = false ]; then
        vsdbgFileExtension=".tar.gz"
    else
        echo "Warning: Version '${__VsDbgMetaVersion}' is only avaliable in zip."
        vsdbgFileExtension=".zip"
    fi
    vsdbgCompressedFile="vsdbg-${__RuntimeID}${vsdbgFileExtension}"
    target="$(echo "${__VsDbgVersion}" | tr '.' '-')"
    url="https://vsdebugger.azureedge.net/vsdbg-${target}/${vsdbgCompressedFile}"

    # check_internet_connection "$url"

    echo "Downloading ${url}"
    # if hash wget 2>/dev/null; then
    #     wget -q "$url" -O "$vsdbgCompressedFile"
    # elif hash curl 2>/dev/null; then
    #     curl -s "$url" -o "$vsdbgCompressedFile"
    # fi

    # if [ $? -ne  0 ]; then
    #     echo
    #     echo "ERROR: Could not download ${url}"
    #     exit 1;
    # fi

    __VsdbgCompressedFile=$vsdbgCompressedFile
   }
   ```
3. 下载安装包
   https://vsdebugger.azureedge.net/vsdbg-17-0-10712-2/vsdbg-linux-x64.tar.gz
4. 将下载的安装包复制到WLS发行版的`/root/vsdbg`目录下
5. 运行
   ```sh
   sh GetVsDbg.sh -v latest -l /usr/local/.vsdbg
   ```

