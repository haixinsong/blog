# Office 2019 部署

[^_^]: # (url:deploy-office)
[^_^]: # (tag:office,#tech)
[^_^]: # (excerpt:借助官方工具部署纯净的 Office 软件包)
[^_^]: # (pre-post：build-your-own-kms-server-image)

> 前文提到了如何创建自己的 kms-serverDocker 镜像，此文记录部署 Office2019 的步骤  
> 本文基于虚拟机部署 Office2019，使用 kms 激活，仅做测试用途。  
> 笔者已入手Office365家庭版，目前坑位尚有，欢迎合租。有意者邮箱联系或者本文留言。

## 生成配置文件 Configuration.xml

前往[https://config.office.com/deploymentsettings](https://config.office.com/deploymentsettings)，配置自己喜好的选项，用以生成配置文件供下一步部署工具使用。

### 产品和版本

此处选择64位的2019 Volume License版，如下图，请根据自身情况适当调整。

![config.office.com_arch](/home/nediiii/CodeProjects/blog/img/post/deploy-office/config.office.com_arch.png)

### 语言

主要语言选择中文，附加语言选择匹配操作系统和英文，随后点击更新。更新后可在右侧看到配置情况。

![config.office.com_lang](/home/nediiii/CodeProjects/blog/img/post/deploy-office/config.office.com_lang.png)

### 安装

语言选择完毕之后进入下一步的安装选项，主要是设定 Office 产品的部署方式，包括 CDN 和本地，以及安装时的用户交互及细节设定。根据需要进行选择。个人用户选择 CDN 即可。

![config.office.com_install-opt](/home/nediiii/CodeProjects/blog/img/post/deploy-office/config.office.com_install-opt.png)

### 更新和升级

对安装部署方式设定完毕之后进入下一步的更新和升级选项。和上一步对应可选有 CDN 和本地源。这里默认勾选了删除传统 Office MSI 版本的选项。以及具体的安装卸载设定。根据需要进行勾选即可。

![config.office.com_update](/home/nediiii/CodeProjects/blog/img/post/deploy-office/config.office.com_update.png)

### 授权和激活

为后续的KMS激活，此处选择KMS，自动接受EULA。

共享计算机激活选项，一般情况下不勾选，详情可参阅[Office 365 专业增强版的共享计算机激活概述](https://docs.microsoft.com/zh-cn/deployoffice/overview-of-shared-computer-activation-for-office-365-proplus)

![config.office.com_key](/home/nediiii/CodeProjects/blog/img/post/deploy-office/config.office.com_key.png)

### 常规

顾名思义，不赘述。可跳过设置。

![config.office.com_regular](/home/nediiii/CodeProjects/blog/img/post/deploy-office/config.office.com_regular.png)

### 应用程序首选项

此处设置应用程序的一些默认首选项，一般不需要调整，默认即可。

![config.office.com_prefer](/home/nediiii/CodeProjects/blog/img/post/deploy-office/config.office.com_prefer.png)

### 配置概览

在右侧有配置概览，查阅无误后便可导出配置。

![config.office.com_conf](/home/nediiii/CodeProjects/blog/img/post/deploy-office/config.office.com_conf.png)

### 导出配置

点击页面顶端的“导出”按钮，勾选`我接受许可协议中的条款`，点击导出。将下载的文件保存好，供下一步使用。

![config.office.com_export](/home/nediiii/CodeProjects/blog/img/post/deploy-office/config.office.com_export.png)

## 下载部署工具

从 Microsoft 下载中心免费下载 [Office 部署工具](https://www.microsoft.com/download/details.aspx?id=49117)

![www.microsoft.com_en-us_download_details](/home/nediiii/CodeProjects/blog/img/post/deploy-office/www.microsoft.com_en-us_download_details.png)

下载 Office 部署工具后，运行它（`officedeploymenttool.exe`），接受协议

![odt-lic](/home/nediiii/CodeProjects/blog/img/post/deploy-office/odt-lic.png)

解压文件到某个目录

![odt-path](/home/nediiii/CodeProjects/blog/img/post/deploy-office/odt-path.png)



完成后，解压到的文件夹中应该有多个文件：`setup.exe`和一些示例`configuration.xml`文件。用我们之前配置好后下载的 xml 文件替换解压后的文件夹中的`configuration.xml`文件。

![odt-file](/home/nediiii/CodeProjects/blog/img/post/deploy-office/odt-file.png)

## 部署

1. 官方建议在安装 Office 2019 的批量许可版本之前卸载 Office 的任何旧版本。更多请参阅[升级到 Office 365 专业增强版时删除现有 MSI 版本的 Office](https://docs.microsoft.com/zh-cn/deployoffice/upgrade-from-msi-version)。

2. 打开提升的命令提示符(以管理员身份运行)，将命令提示符的工作路径转到保存 ODT 和 configuration.xml 文件的文件夹，然后键入以下命令（如果已使用其他名称保存 configuration.xml 文件，请在命令中使用该名称），便会开始运行Office部署工具：

   ```
   setup /configure configuration.xml
   ```

   ![odt-run](/home/nediiii/CodeProjects/blog/img/post/deploy-office/odt-run.png)

3. 等待安装完成。

   ![odt-done](/home/nediiii/CodeProjects/blog/img/post/deploy-office/odt-done.png)

4. 安装完成，开始菜单可找到Office等应用。

   ![office-icon](/home/nediiii/CodeProjects/blog/img/post/deploy-office/office-icon.png)

## 激活

命令参阅：<https://docs.microsoft.com/en-us/deployoffice/vlactivation/tools-to-manage-volume-activation-of-office>  
密钥参阅：<https://docs.microsoft.com/en-us/DeployOffice/vlactivation/gvlks>  
如果是按照上述步骤部署的Office2019,直接从第四步开始即可（貌似部署时默认设置好了密钥，无须人工干预），只需设置kms服务器地址，然后激活。  

1. 使用docker运行kms-server，监听1688端口

    ```bash
    docker run -d -p 1688:1688 nediiii/kms-server
    ```

    ![docker-run-kms-server](/home/nediiii/CodeProjects/blog/img/post/deploy-office/docker-run-kms-server.png)

2. 查看运行kms-server主机的ip,后续需要使用

3. 进入到`C:\Program Files\Microsoft Office\Office16`目录下，用管理员权限运行命令行，用以下命令激活Office

    - 检查状态
      `cscript ospp.vbs /dstatus`

      ![cmd-dstatus](/home/nediiii/CodeProjects/blog/img/post/deploy-office/cmd-dstatus.png)

    - 卸载密钥:  
      `cscript ospp.vbs /unpkey:XXXXX`

    - 输入密钥:  
      `cscript ospp.vbs /inpkey:XXXXX-XXXXX-XXXXX-XXXXX-XXXXX`

    - 设置kms服务器地址
      `cscript ospp.vbs /sethst:ip`  

      ![cmd-sethst](/home/nediiii/CodeProjects/blog/img/post/deploy-office/cmd-sethst.png)

    - 激活
      `cscript ospp.vbs /act`  

      ![cmd-act](/home/nediiii/CodeProjects/blog/img/post/deploy-office/cmd-act.png)

4. 打开Office软件，查看是否已激活。如下图，软件皆已激活。

    ![word-act](/home/nediiii/CodeProjects/blog/img/post/deploy-office/word-act.png)

    ![visio-act](/home/nediiii/CodeProjects/blog/img/post/deploy-office/visio-act.png)

    ![project-act](/home/nediiii/CodeProjects/blog/img/post/deploy-office/project-act.png)

## 结语

在kms服务器一直有效（没有关停，没有改变地址）的状态下，Office会自动定期更新激活状态。到此为止，Office2019的部署已经完成。对于长期频繁使用Office的用户，依然是建议入手Office365，像本文描述的非官方提供的kms激活，实质上是盗版的Office，除用于试验用途外，反对以任何形式将其用于牟利和白嫖。
