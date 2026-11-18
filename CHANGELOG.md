# Fusion360 机电协同 -- 变更记录

# 1.2.7
### 新功能 / Features

- **导出设置对话框**（中英文）：导入目标、向上轴、板子偏移/旋转、捕获位置方式；新菜单项 **导出设置** / Export settings dialog (Chinese/English): import target, up axis, board offset/rotation, Capture Position behaviour; new menu item **Fusion 360 export settings**
- **导入到当前 Fusion 设计**，保留时间线 / Import into the active Fusion design, keeping its timeline
- **Y 轴向上 / Z 轴向上** 与板子偏移/旋转；修改后点击保存会重新放置已导入的板子；板子在设计中带标记，重启后可重新找到 / Y-up/Z-up handling and board offset/rotation; Save re-places an imported board; the board is tagged so it is found again after a restart
- **元件移动在板子坐标系中计算**：板子可在外壳中移动或旋转 / Part moves are computed in the board's frame, so the board can be moved or rotated inside an enclosure
- **捕获位置**：同步停止时、移动停顿后或从不；可只保留一个捕获。板子放置与元件移动分别保存在 **EasyEDA placement** 和 **EasyEDA PCB** 捕获中 / Capture Position when sync stops, after a pause, or never; optionally a single capture. Placement and part moves use separate captures

### 修复 / Fixes

- **防止重复导出**：一次点击只会导出一次 / A single click can no longer trigger two exports
- **板子不作为设计中的第一个组件**：Fusion 把第一个组件视为固定，捕获位置时会丢弃它的移动 / The board is never the design's first component; Fusion treats that one as anchored and Capture Position drops its move

# 1.1.3
### 修复 / Fixes

- **STEP 导入超时（issue #2）**：改用官方 `importManager.importToNewDocument()`，替代 `Translator.Import` 文本命令；超时延长到 10 分钟，导入失败时保留 STEP 文件 / Fix the `_import_step` timeout by using the documented `importManager` API instead of the `Translator.Import` text command; 10-minute timeout; the STEP file is kept when an import fails
- **线程安全日志**：后台线程不再直接调用 `app.log()`，日志同时写入 `~/easyeda_fusion.log` / Thread-safe logging: background threads no longer call `app.log()`; every line also goes to `~/easyeda_fusion.log`
- **停止附加模块时关闭 WebSocket 服务器**，释放端口，停止/运行可正确重新加载代码 / Stopping the add-in shuts down the WebSocket server and frees port 8767, so Stop/Run reloads the code
- **仅监听本机**：WebSocket 服务器绑定 `127.0.0.1` 而不是 `0.0.0.0` / The WebSocket server listens on `127.0.0.1` only instead of `0.0.0.0`
- **上传更快**：分片拼接和 WebSocket 解码不再随文件大小急剧变慢 / Faster chunk reassembly and frame unmasking for large files
- **防止重复导入**：导入进行中时拒绝新的上传 / Uploads arriving while an import runs are rejected
- **元件移动保留翻转**：底层元件不再被翻回顶层 / Position updates keep each part's orientation; bottom-side parts stay flipped

# 1.1.1
### 优化

- 使用新的logo

# 1.1.0
### 优化

- 多语言优化

# 1.1.0
### 新功能

- **自动新建文档**：每次导出 3D 模型时自动创建新 Fusion 设计文档，不再需要手动新建
- **选中高亮**：交叉定位从闪烁效果改为选中状态 + 相机导航，高亮持续到用户点击其他位置
 

# 1.0.0

- 初始版本
