## 🔧 一、环境管理

|操作|命令|
|---|---|
|查看所有环境|`conda env list` 或 `conda info --envs`|
|创建新环境|`conda create -n env_name python=3.10`|
|激活环境|`conda activate env_name`|
|退出当前环境|`conda deactivate`|
|删除环境|`conda remove -n env_name --all`|
|重命名环境|没有直接命令，可导出-再创建|

---

## 📦 二、包管理

| 操作          | 命令                                 |
| ----------- | ---------------------------------- |
| 安装包         | `conda install package_name`       |
| 指定版本安装      | `conda install package_name=1.2.3` |
| 升级包         | `conda update package_name`        |
| 升级 conda 本体 | `conda update conda`               |
| 删除包         | `conda remove package_name`        |
| 查看已安装包      | `conda list`                       |

---

## 📁 三、环境导出与复制

|操作|命令|
|---|---|
|导出环境（推荐 `--no-builds`）|`conda env export --no-builds > env.yml`|
|从 `.yml` 创建环境|`conda env create -f env.yml`|
|更新环境（从 `.yml`）|`conda env update -f env.yml --prune`|

---

## 🌐 四、源与通道管理（可加速）

|操作|命令|
|---|---|
|添加通道|`conda config --add channels conda-forge`|
|设置为默认通道|`conda config --set channel_priority strict`|
|查看配置|`conda config --show`|
|清除索引缓存|`conda clean --all`|

推荐设置国内镜像（如清华、阿里）以加速下载。

---

## 🧪 五、Python/Conda 版本管理

|操作|命令|
|---|---|
|查看当前 Python 版本|`python --version`|
|更换 Python 版本（环境中）|`conda install python=3.x`|
|创建不同 Python 版本环境|`conda create -n py39 python=3.9`|

---

## 🧩 六、虚拟环境内使用 pip

你可以混合使用 `conda` 和 `pip`，但建议：

- 优先用 `conda` 安装（更安全依赖管理）
    
- 必须用 pip 时，确保环境已激活再运行：
    

```bash
conda activate myenv
pip install package-name
```

---

## 🪄 七、补充技巧

- 初始化 shell（首次安装后）：
    
    ```bash
    conda init zsh  # 或 bash
    ```
    
- 查看某包依赖：
    
    ```bash
    conda info package-name
    ```
    
- 克隆已有环境：
    
    ```bash
    conda create --name newenv --clone oldenv
    ```
    

---
