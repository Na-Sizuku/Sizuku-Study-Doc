# ArchLinux 办公软件

到这里恭喜你已经完成了基础操作系统安装和桌面环境安装，你可能需要一些办公软件去打开 Word 文档、PPT 幻灯片、Excel 表格又或者是 PDF 文件。  
通常情况下你可以通过安装如`LibreOffice`和`WPS`直接拥有全套基础 Office。  
出于兼容性考虑一般情况下推荐使用官方提供的软件`LibreOffice`，当然如果你觉得使用`LibreOffice`不适应你也可以通过 AUR 安装`WPS`。

## 关于 LibreOffice

LibreOffice 是一款开源跨平台的 Office 办公套件，除去支持 Word 文档、PPT 幻灯片、Excel 表格也支持包括 PDF、数据库等其他东西。  
在 Linux 平台上 LibreOffice 因具有很好的多种办公文档支持而广受欢迎和使用，但由于 LibreOffice 是从 Xorg 时代就诞生的产物在 Wayland 上面的使用和体验或多或少有些许问题，官方也在大力推进和解决适配问题。

LibreOffice 具有两个分支，安装的时候仅需要选择其一即可。  
对于保守的用户，不喜欢大量更新或者喜欢相对稳定的版本则可以直接安装`libreoffice-still`  
对于喜欢新功能或者需要高级功能的用户则可以安装`libreoffice-fresh`  
对于汉化，只需要安装对应版本的汉化包即可`libreoffice-<替换为你安装的分支>-zh-cn`

由于 LibreOffice 是 AIO 级别的开源办公软件，你只需要安装一个 LibreOffice 实际上拥有了编辑 Word、PPT、Excel、PDF 等许多文件功能。
