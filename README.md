# myflaskapp

一个 Flask 示例应用，配套 Jenkins Pipeline 实现完整 CI/CD 流程。

## 项目结构

```
myflaskapp/
├── app.py                 # Flask 应用，根路由返回 "Hello, CI/CD with Jenkins!"
├── requirements.txt       # 依赖清单（锁定 Flask 2.0.1 / Werkzeug 2.0.3）
├── .flake8                # flake8 配置（忽略 E302/E305/W292）
├── tests/
│   └── test_app.py        # pytest 测试用例
├── Jenkinsfile            # Windows 版流水线（指导书原版，使用 bat 步骤）
├── Jenkinsfile.linux      # Linux 版流水线（等价实现，使用 sh 步骤）
└── .gitignore
```

## 关于两个 Jenkinsfile

两份流水线的**阶段划分、顺序、语义完全一致**，区别只在执行平台：

| 文件 | 目标平台 | 关键差异 |
|---|---|---|
| `Jenkinsfile` | Windows | 使用 `bat` 步骤；`PYTHON_PATH` 指向 Windows 下的 `python.exe` |
| `Jenkinsfile.linux` | Linux / Docker | 使用 `sh` 步骤；`PYTHON_PATH` 指向 `/usr/bin/python3` |

**为什么需要两份**：`bat` 是 Jenkins **Windows 节点专有**的步骤，在 Linux 上不存在。因此同一份 Jenkinsfile 无法同时跑在两个平台上。

在 Jenkins 项目配置里，通过 **Pipeline → Script Path** 指定用哪一份：
- Windows 上的 Jenkins：留默认 `Jenkinsfile`
- Linux/Docker 里的 Jenkins：填 `Jenkinsfile.linux`

### Windows 版的路径说明

`Jenkinsfile` 中的 `PYTHON_PATH` 写的是 **8.3 短路径**：

```
C:\Users\7071~1\AppData\Local\Programs\Python\PYTHON~1\python.exe
```

它等价于完整路径 `C:\Users\海祇雪莉\AppData\Local\Programs\Python\Python310\python.exe`。使用短路径的原因是：本机用户名含中文，而 Jenkins 的 `bat` 步骤会把脚本写成 `.bat` 临时文件并按系统码页（GBK）解释，中文路径会变成乱码并报 `The system cannot find the path specified`。短路径与编码无关，GBK/UTF-8 下均可正常调用。

## 流水线阶段

| 阶段 | 作用 |
|---|---|
| Verify Python Path | 验证 Python 解释器路径可用 |
| Checkout | 拉取 Git 仓库代码 |
| Fix pip | 修复 / 升级 pip 环境 |
| Install Dependencies | 安装 `requirements.txt` 依赖 |
| Lint | flake8 代码风格检查 |
| Test | pytest 单元测试 + 覆盖率报告（HTML） |
| Build | PyInstaller 打包为单文件可执行程序 |
| Deploy | 启动应用并通过 HTTP 请求验证服务可用 |

## 本地运行

```bash
pip install -r requirements.txt
python app.py                 # 访问 http://127.0.0.1:5000
pytest --cov=app tests/       # 运行测试
flake8 app.py tests/          # 代码检查
```
