# 高清笔记页面目录修复

状态：Windows / Android 的 1.1.15 正式附件已替换；iOS 1.1.15 Beta 2 已发布到 fork 与上游，应用主版本不变。

## 原因

云笔记资源中包含整本笔记的 `page_router.bin`。它记录原始 UUID 页面目录，而资源适配层已经把页面目录规范化成 `pageKey/`。旧逻辑将整本页面路由作为单页文件放进 VFS；`ezy-board-viewer` 优先读取这份路由，随后在不存在的 UUID 目录查找 `snapshot.bin`，误报文件缺失。

## 修复

- 收集页面资源时排除整本页面路由，不改动云端笔记或原始下载文件。
- VFS 的文件列表和读取接口也排除这份旧路由，兼容直接传入旧资源清单的调用。
- 查看器从规范化后的真实页面目录发现页面，保留文字、笔迹、图片、MDB 背景资源及页序。

## 验证

测试使用合成笔记，包含原始 UUID 页面路由、两张矢量页及一张截图页，没有读取或修改个人笔记。

- 修复前，新加入的路由回归检查失败，可以复现目录冲突。
- Microsoft Edge：23 项笔记检查通过。
- Windows 原生 WebView2：183 项完整检查通过，包括实际挂载高清组件、显示第一页、翻到另一 UUID 对应页及 PDF/SVG 导出。
- Android 原生 WebView 模拟器：23 项笔记检查通过，包括实际挂载组件和翻页。
- iOS：Xcode 编译两份 arm64 设备 IPA；实际 iPhone / iPad 模拟器 WKWebView 回归通过（5 + 2 项 XCTest），涵盖高清笔记显示、翻页、PDF / SVG 导出及上传。未进行真机测试。
- [iOS 原生回归记录](https://github.com/nickfox395/ZhongYuToolBox_Web/actions/runs/37569342316)。

Windows 安装程序 / 便携包、Android 正式签名 APK、iOS 未签名 IPA 均已上传并核对公开下载字节和 SHA-256；没有打入测试页面或签名资料。正式页不包含 IPA，iOS 继续独立内测。结构化结果的 `publishedPackages` 保存本次公开附件校验记录，`localPackages` 保留此前本地验证包的历史记录。

结构化结果见 [note-preview-20261007.json](verification/note-preview-20261007.json)。这项修复解决错误目录引发的误报；笔记真实缺少页面资源时仍应明确报错，不生成空白页面掩盖问题。
