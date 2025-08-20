## LazyVim 快捷键对照表（中文说明）

### 1. 光标移动优化

- `j` / `<Down>` → 如果没有输入数字计数，就移动到屏幕下一行（`gj`），否则正常下移一行。
    
- `k` / `<Up>` → 如果没有输入数字计数，就移动到屏幕上一行（`gk`），否则正常上移一行。
    

### 2. 窗口跳转

- `<C-h>` → 跳到左边窗口
    
- `<C-j>` → 跳到下方窗口
    
- `<C-k>` → 跳到上方窗口
    
- `<C-l>` → 跳到右边窗口
    

### 3. 窗口调整大小

- `<C-Up>` → 增加窗口高度
    
- `<C-Down>` → 减少窗口高度
    
- `<C-Left>` → 减少窗口宽度
    
- `<C-Right>` → 增加窗口宽度
    

### 4. 移动行

- 普通模式 `Alt-j` / `Alt-k` → 向下 / 向上移动当前行
    
- 插入模式 `Alt-j` / `Alt-k` → 向下 / 向上移动当前行并保持在插入模式
    
- 可视模式 `Alt-j` / `Alt-k` → 移动选中行
    

### 5. 缓冲区（Buffer）管理

- `<S-h>` / `[b` → 上一个缓冲区
    
- `<S-l>` / `]b` → 下一个缓冲区
    
- `<leader>bb` / ``` → 切换到上一个缓冲区
    
- `<leader>bd` → 删除当前缓冲区
    
- `<leader>bo` → 删除其他缓冲区
    
- `<leader>bD` → 删除缓冲区并关闭窗口
    

### 6. 清除搜索高亮

- `<esc>`（在普通、插入、选择模式） → 清除搜索高亮，并停止 snippet
    

### 7. 重新绘制与更新

- `<leader>ur` → 清除搜索高亮、更新 diff、重绘界面
    

### 8. 搜索优化

- `n` / `N` → 搜索结果跳转时保持光标在可见区域
    
- 可视/操作模式下 `n` / `N` → 同上
    

### 9. 插入模式撤销点

- `,` `.` `;` → 插入模式时增加撤销断点
    

### 10. 文件保存

- `<C-s>`（所有模式） → 保存文件
    

### 11. 缩进优化

- 选择模式 `<` / `>` → 缩进后保持选区
    

### 12. 注释

- `gco` → 在当前行下方新建一行并加注释
    
- `gcO` → 在当前行上方新建一行并加注释
    

### 13. Lazy 管理器

- `<leader>l` → 打开 LazyVim 插件管理界面
    

### 14. 新建文件

- `<leader>fn` → 新建一个空文件
    

### 15. 位置列表 / 快速修复列表

- `<leader>xl` → 打开/关闭位置列表（loclist）
    
- `<leader>xq` → 打开/关闭快速修复列表（quickfix）
    
- `[q` → 上一个 quickfix
    
- `]q` → 下一个 quickfix
    

### 16. 格式化代码

- `<leader>cf` → 强制格式化代码
    

### 17. 诊断信息（LSP）

- `<leader>cd` → 查看当前行诊断信息（浮动窗口）
    
- `]d` / `[d` → 下一个 / 上一个诊断
    
- `]e` / `[e` → 下一个 / 上一个错误
    
- `]w` / `[w` → 下一个 / 上一个警告
    

### 18. 选项切换（Snacks）

- `<leader>uf` / `<leader>uF` → 开关自动格式化
    
- `<leader>us` → 开关拼写检查
    
- `<leader>uw` → 开关自动换行
    
- `<leader>uL` → 开关相对行号
    
- `<leader>ud` → 开关诊断
    
- `<leader>ul` → 开关行号
    
- `<leader>uc` → 开关隐藏文本（conceal）
    
- `<leader>uA` → 开关标签栏
    
- `<leader>uT` → 开关 Tree-sitter
    
- `<leader>ub` → 切换背景亮/暗
    
- `<leader>uD` → 开关 dim 模式
    
- `<leader>ua` → 开关动画
    
- `<leader>ug` → 开关缩进线
    
- `<leader>uS` → 开关滚动动画
    
- `<leader>dpp` → 开关性能分析器
    
- `<leader>dph` → 开关性能分析器高亮
    
- `<leader>uh` → 开关 inlay hints（如果支持）
    

### 19. Git（需要 lazygit）

- `<leader>gg` → 在项目根目录打开 lazygit
    
- `<leader>gG` → 在当前目录打开 lazygit
    
- `<leader>gf` → 查看当前文件 Git 历史
    
- `<leader>gl` → 查看项目 Git 日志（root）
    
- `<leader>gL` → 查看当前目录 Git 日志
    
- `<leader>gb` → 查看当前行 Git blame
    
- `<leader>gB` → 打开当前行/选中内容对应的远程仓库链接
    
- `<leader>gY` → 复制当前行/选中内容对应的远程仓库链接
    

### 20. 退出

- `<leader>qq` → 退出所有
    

### 21. 调试 / 检查

- `<leader>ui` → 检查光标下元素信息
    
- `<leader>uI` → 检查 Tree-sitter 语法树
    

### 22. LazyVim 更新日志

- `<leader>L` → 打开 LazyVim 更新日志
    

### 23. 终端

- `<leader>fT` → 在当前目录打开浮动终端
    
- `<leader>ft` / `<C-/>` / `<C-_>` → 在项目根目录打开浮动终端
    
- 终端模式 `<C-/>` / `<C-_>` → 关闭浮动终端
    

### 24. 窗口管理

- `<leader>-` → 横向分屏
    
- `<leader>|` → 纵向分屏
    
- `<leader>wd` → 关闭当前窗口
    
- `<leader>wm` / `<leader>uZ` → 最大化窗口
    
- `<leader>uz` → Zen 模式
    

### 25. 标签页

- `<leader><tab>l` → 最后一个标签页
    
- `<leader><tab>o` → 关闭其他标签页
    
- `<leader><tab>f` → 第一个标签页
    
- `<leader><tab><tab>` → 新建标签页
    
- `<leader><tab>]` → 下一个标签页
    
- `<leader><tab>d` → 关闭标签页
    
- `<leader><tab>[` → 上一个标签页
    
---

# 🧭 LazyVim LSP 快捷键总表

|快捷键|功能|说明|
|---|---|---|
|**gd**|跳转到定义 (_go to definition_)|跳到函数/变量的定义处|
|**gD**|跳转到声明 (_go to declaration_)|通常用于类型或变量声明|
|**gi**|跳转到实现 (_go to implementation_)|适用于接口/抽象方法的实现|
|**gr**|查找引用 (_find references_)|查看符号被使用的地方|
|**K**|显示文档|浮动窗口展示函数/变量的文档注释|
|**ca**|代码操作 (_code action_)|比如快速修复、导入、生成代码等|
|**rn**|重命名符号 (_rename_)|变量/函数/类统一重命名|
|**ds**|查看文档符号 (_document symbols_)|查看当前文件的函数/类/变量列表|
|**ws**|查看工作区符号 (_workspace symbols_)|跨项目搜索符号|
|**f**|格式化代码 (_format_)|调用 LSP 格式化代码|
|**[d**|跳到上一个诊断|代码报错/警告上一个位置|
|**]d**|跳到下一个诊断|代码报错/警告下一个位置|
|**e**|查看诊断信息|打开浮动窗口显示报错内容|
|**q**|把诊断信息加入 quickfix|可以在 quickfix 列表中浏览错误|

---

## 📌 使用小技巧

1. **快速跳转回来**：用 `Ctrl-o` 返回跳转前位置（和 `gd` 配合）。
    
2. **多引用浏览**：`gr` 后会调用 Telescope/FZF，你可以在弹出的列表中选择具体位置。
    
3. **接口开发**：`gi` 很有用，可以快速找到接口的具体实现。
    

---
