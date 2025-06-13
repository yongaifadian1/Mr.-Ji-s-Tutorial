
# AutoDL服务器Pyannote-Audio项目Linux新手保姆级教程

## 前言

欢迎来到 AutoDL 和 `pyannote-audio` 的世界！本教程专为从未接触过 Linux 系统的新手设计，将手把手带你从零开始，在 AutoDL 云服务器上成功部署和使用强大的声音工具包 `pyannote-audio`。

`pyannote-audio` 是一个基于 PyTorch 的开源工具包，用于**说话人区分 (Speaker Diarization)**。简单来说，它可以分析一段包含多人对话的音频，并告诉你“谁在什么时间说话”。这在会议记录整理、语音内容分析、智能客服等领域有广泛应用。

**本教程将带你完成以下目标：**

1.  **连接服务器**：在自己的 Windows 电脑上，远程连接到 AutoDL 的 Linux 服务器。
2.  **Linux入门**：学习几个最基本、最常用的 Linux 命令，让你不再害怕“小黑窗”。
3.  **环境配置**：使用 Conda 创建一个纯净、独立的 Python 环境，避免项目间的依赖冲突。
4.  **项目部署**：下载并成功安装 `pyannote-audio` 及其所有依赖。
5.  **运行示例**：亲手运行一个官方示例，体验 `pyannote-audio` 的神奇功能。

## 1. AutoDL平台介绍和准备工作

### 1.1 AutoDL 是什么？

AutoDL 是一个面向人工智能开发者的云服务器平台。它最大的优点是：

*   **价格实惠**：按小时计费，不用的时候可以关机，非常适合学习和实验。
*   **GPU资源丰富**：提供多种型号的 NVIDIA GPU，是运行深度学习项目（如 `pyannote-audio`）的绝佳选择。
*   **预装环境**：服务器已经预装了常用的深度学习框架（如 PyTorch、TensorFlow）、CUDA、Conda 等，省去了大量繁琐的环境配置工作。

### 1.2 准备工作

在开始之前，请确保你已经完成了以下准备：

1.  **注册并充值 AutoDL 账号**：访问 [AutoDL 官网](https://www.autodl.com/) 并完成注册和实名认证，同时保证账户有少量余额。
2.  **租用一台服务器**：
    *   登录 AutoDL 控制台，选择“租用新主机”。
    *   **地区**：选择一个延迟较低的地区。
    *   **主机型号**：对于本教程，选择任意一款 **单卡** 的 GPU 即可（例如 RTX 3080 或 RTX 3090），价格相对便宜。
    *   **镜像**：这是最关键的一步！请选择 **`PyTorch`** 相关的镜像，例如 `PyTorch 1.12.1` 或更高版本。`pyannote-audio` 需要 `Python 3.9+` 和 `PyTorch`，选择合适的镜像能省去很多麻烦。
    *   确认租用，等待服务器实例创建成功。

3.  **获取 Hugging Face 访问令牌**：
    *   `pyannote-audio` 需要从 Hugging Face 模型中心下载预训练模型，这需要一个访问令牌 (Access Token)。
    *   访问 [Hugging Face 官网](https://huggingface.co/) 并注册一个账号。
    *   登录后，访问 [Access Tokens 页面](https://huggingface.co/settings/tokens)。
    *   点击 “New token”，给它起个名字（例如 `pyannote-token`），权限设置为 “read”，然后创建。
    *   **立即复制并保存好这个令牌！** 它只会显示一次。我们后续会用到，用 `YOUR_HUGGINGFACE_TOKEN` 指代它。

4.  **接受模型用户协议**：
    *   `pyannote-audio` 依赖两个关键模型。你需要访问它们的 Hugging Face 页面并接受用户协议。
    *   访问 [pyannote/speaker-diarization-3.1](https://huggingface.co/pyannote/speaker-diarization-3.1)
    *   访问 [pyannote/segmentation-3.0](https://huggingface.co/pyannote/segmentation-3.0)
    *   在页面上找到并点击 "Agree and access repository" 或类似的按钮。请确保用你申请令牌的 Hugging Face 账户登录。

## 2. Windows SSH连接工具安装和使用

当你租用到服务器后，AutoDL 会提供一个 SSH 登录指令。SSH 是一种安全的远程登录协议。我们需要一个 SSH 客户端工具来从 Windows 连接到这台远在云端的 Linux 服务器。

### 2.1 找到连接信息

在 AutoDL 控制台，找到你租用的服务器，可以看到“开机”、“关机”等按钮。在旁边有一个“SSH”按钮或直接显示了 SSH 登录指令，格式如下：

```bash
ssh -p [端口号] root@[地区节点]
```

例如：`ssh -p 12345 root@region-1.autodl.com`

同时，你也会看到这台服务器的 **密码**。请复制并保存好 **SSH指令** 和 **密码**。

### 2.2 推荐工具：MobaXterm

对于新手，我们强烈推荐 **MobaXterm**。它是一个功能强大的终端工具，优点包括：

*   内置 SSH 客户端。
*   **图形化的文件浏览器**：可以直接拖拽上传和下载文件，对新手极其友好。
*   支持多标签页。

**安装步骤：**

1.  访问 [MobaXterm 官网](https://mobaxterm.mobatek.net/download.html)。
2.  下载免费的 "Home Edition" 安装包 (Installer edition)。
3.  解压并安装。

**使用 MobaXterm 连接服务器：**

1.  打开 MobaXterm。
2.  点击左上角的 "Session" 按钮。
3.  在弹出的窗口中，选择 "SSH"。
4.  在 "Remote host" 中，填写 `@` 后面的部分，例如 `region-1.autodl.com`。
5.  勾选 "Specify username"，填写 `root`。
6.  在 "Port" 中，填写 `-p` 后面的端口号，例如 `12345`。
7.  点击 "OK"。
8.  终端会提示你输入密码。输入你从 AutoDL 复制的密码，然后按回车。
    *   **注意**：输入密码时，屏幕上不会有任何显示（没有星号也没有光标移动），这是 Linux 的正常安全机制。自信地输入，然后回车即可。
9.  连接成功后，你会看到一个命令行界面，通常以 `root@autodl-container:~#` 开头。左侧会出现一个图形化的文件浏览器，显示了服务器上的文件目录。

**【截图说明】**
一张 MobaXterm 的 Session 设置截图。"Remote host" 填写了服务器地址，"Specify username" 填写了 "root"，"Port" 填写了端口号。

## 3. Linux系统基础知识和常用命令

现在你已经进入了 Linux 的世界！这个“小黑窗”就是你的主战场。别怕，我们从最基础的几个命令开始。在 MobaXterm 的终端里输入命令，然后按回车执行。

### 3.1 命令行是什么？

它是一个文本界面的操作系统“遥控器”。你输入指令，系统执行并返回结果。

*   **`root`**：表示你当前是“超级用户”，拥有最高权限。
*   **`~`**：表示你当前所在的目录是“家目录” (`/root`)。
*   **`#`**：是命令提示符。

### 3.2 常用命令

*   **`ls` (list) - 列出文件和目录**
    *   `ls`：列出当前目录下的文件和文件夹。
    *   `ls -l`：以长格式（列表）显示，包含更多细节，如权限、大小、修改日期。
    *   `ls -a`：显示所有文件，包括以 `.` 开头的隐藏文件。

*   **`pwd` (print working directory) - 显示当前路径**
    *   `pwd`：告诉你当前你“身在何处”。

*   **`cd` (change directory) - 切换目录**
    *   `cd [目录名]`：进入指定的目录。例如 `cd /` 进入根目录，`cd autodl-tmp` 进入名为 `autodl-tmp` 的目录。
    *   `cd ..`：返回上一级目录。
    *   `cd ~` 或 `cd`：直接返回家目录。

*   **`mkdir` (make directory) - 创建目录**
    *   `mkdir [新目录名]`：创建一个新的空文件夹。例如 `mkdir my_project`。

*   **`cp` (copy) - 复制文件或目录**
    *   `cp [源文件] [目标文件/目录]`：复制文件。
    *   例如 `cp main.py /autodl-tmp/` 把 `main.py` 复制到 `/autodl-tmp` 目录下。
    *   `cp -r [源目录] [目标目录]`：复制整个目录（需要加 `-r`）。

*   **`mv` (move) - 移动或重命名文件/目录**
    *   `mv [源文件] [目标目录]`：移动文件。
    *   `mv [旧名字] [新名字]`：重命名。
    *   例如 `mv old_name.txt new_name.txt`。

*   **`rm` (remove) - 删除文件或目录**
    *   `rm [文件名]`：删除文件。
    *   `rm -r [目录名]`：删除目录及其下所有内容。
    *   **【警告】** `rm` 命令是**毁灭性**的，没有回收站！删除前请再三确认！特别是 `rm -rf /`，这个命令会删掉你的整个系统，千万不要尝试！

## 4. Conda环境管理详解

AutoDL 的镜像已经为我们装好了 Conda。Conda 是一个强大的环境管理器，可以为你的每个项目创建一个隔离的“房间”，每个房间里的 Python 版本和库都是独立的，互不干扰。

### 4.1 创建新的 Conda 环境

`pyannote-audio` 要求 `Python 3.9` 或更高版本。我们来创建一个专门用于运行它的环境。

```bash
conda create -n pyannote python=3.9 -y
```

*   `conda create`：创建环境的命令。
*   `-n pyannote`：给环境起个名字，这里叫 `pyannote`。
*   `python=3.9`：指定这个环境的 Python 版本为 3.9。
*   `-y`：对所有提示自动回答“yes”。

等待命令执行完成，Conda 就会为你创建一个名为 `pyannote` 的新环境。

### 4.2 激活和退出环境

创建好后，你需要“进入”这个环境才能使用它。

*   **激活环境**：
    ```bash
    conda activate pyannote
    ```
    激活后，你会发现命令行提示符前面多了 `(pyannote)` 的字样，表示你已成功进入该环境。

*   **退出环境**：
    ```bash
    conda deactivate
    ```
    执行后，提示符前的 `(pyannote)` 会消失，你就回到了基础环境 (base)。

**【重要】** 之后所有关于 `pyannote-audio` 的安装和运行，都必须在**激活 `pyannote` 环境后**进行！

## 5. pyannote-audio项目下载和安装

现在，我们正式开始部署 `pyannote-audio`。

**确保你已经执行了 `conda activate pyannote`！**

### 5.1 安装 PyTorch

`pyannote-audio` 依赖于 PyTorch。AutoDL 的服务器虽然预装了 PyTorch，但通常是装在 `base` 环境中。我们需要在自己的 `pyannote` 环境里安装一次。

访问 [PyTorch 官网](https://pytorch.org/get-started/locally/)，根据你的服务器 CUDA 版本选择合适的安装命令。你可以在 AutoDL 的控制台或通过在终端输入 `nvidia-smi` 命令查看 CUDA 版本。

通常，安装命令如下（以 CUDA 11.7 为例）：

```bash
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu117
```

*   `pip` 是 Python 的包管理器，用于安装库。
*   这个命令会从 PyTorch 官方源下载和安装与 CUDA 11.7 兼容的版本。请务必使用官网提供的命令，不要直接 `pip install torch`，否则可能装上的是 CPU 版本。

### 5.2 安装 pyannote.audio

主角登场！安装 `pyannote-audio` 非常简单：

```bash
pip install pyannote.audio
```

`pip` 会自动下载并安装 `pyannote.audio` 以及它所需要的所有其他 Python 库。

## 6. 项目使用示例和常见问题解决

### 6.1 准备工作

1.  **创建一个工作目录**
    ```bash
    mkdir ~/pyannote-project
    cd ~/pyannote-project
    ```
    我们创建了一个名为 `pyannote-project` 的文件夹并进入，后续操作都在这里进行。

2.  **上传音频文件**
    *   你需要一个 `.wav` 格式的音频文件来进行测试。可以自己录制一段，或者从网上下载一个。
    *   **使用 MobaXterm 上传**：在 MobaXterm 左侧的文件浏览器中，导航到 `/root/pyannote-project` 目录。然后，直接从你的 Windows 电脑桌面把 `audio.wav` 文件拖拽到这个目录里。
    *   上传后，可以在终端输入 `ls`，确认能看到 `audio.wav` 文件。

### 6.2 编写并运行Python脚本

1.  **创建 Python 脚本文件**
    ```bash
    touch run_diarization.py
    ```
    `touch` 命令可以创建一个空文件。

2.  **编辑脚本**
    *   在 MobaXterm 中，双击左侧文件浏览器中的 `run_diarization.py`，它会用内置的文本编辑器打开这个文件。
    *   将下面的代码复制粘贴进去：

    ```python
    import torch
    from pyannote.audio import Pipeline

    # 1. 加载预训练管道
    # !!! 重要：将 "YOUR_HUGGINGFACE_TOKEN" 替换成你自己的 Hugging Face 访问令牌 !!!
    try:
        pipeline = Pipeline.from_pretrained(
            "pyannote/speaker-diarization-3.1",
            use_auth_token="YOUR_HUGGINGFACE_TOKEN")
    except Exception as e:
        print(f"加载模型失败，请检查：")
        print(f"1. 你的Hugging Face令牌是否正确。")
        print(f"2. 你是否已在Hugging Face网站上接受了模型的用户协议。")
        print(f"   - 模型1: https://huggingface.co/pyannote/speaker-diarization-3.1")
        print(f"   - 模型2: https://huggingface.co/pyannote/segmentation-3.0")
        print(f"原始错误: {e}")
        exit()

    # 2. 将管道发送到 GPU
    if torch.cuda.is_available():
        print("检测到 GPU，将模型发送到 CUDA")
        pipeline.to(torch.device("cuda"))
    else:
        print("未检测到 GPU，将使用 CPU（速度较慢）")

    # 3. 定义音频文件路径
    audio_file = "audio.wav"

    # 4. 应用管道到音频文件
    print(f"正在处理音频文件: {audio_file} ...")
    try:
        diarization = pipeline(audio_file)
    except Exception as e:
        print(f"处理音频文件时出错。请确保 'audio.wav' 文件存在于当前目录，并且是有效的WAV格式。")
        print(f"原始错误: {e}")
        exit()


    # 5. 打印结果
    print("处理完成，结果如下：")
    for turn, _, speaker in diarization.itertracks(yield_label=True):
        print(f"start={turn.start:.1f}s stop={turn.end:.1f}s speaker_{speaker}")
    ```
    **【再次提醒】** 一定要把代码中的 `YOUR_HUGGINGFACE_TOKEN` 替换成你自己在 **步骤 1.2** 中获取的真实令牌！

3.  **保存并关闭编辑器**

4.  **运行脚本**
    在终端中（确保你仍在 `(pyannote)` 环境和 `~/pyannote-project` 目录下），运行脚本：
    ```bash
    python run_diarization.py
    ```

### 6.3 查看结果

如果一切顺利，程序会先下载所需的模型（第一次运行时需要一些时间），然后开始处理音频。处理完成后，你会在终端看到类似下面的输出：

```
检测到 GPU，将模型发送到 CUDA
正在处理音频文件: audio.wav ...
处理完成，结果如下：
start=0.2s stop=1.5s speaker_0
start=1.8s stop=3.9s speaker_1
start=4.2s stop=5.7s speaker_0
...
```

这个结果告诉你，从 0.2 秒到 1.5 秒是 `speaker_0` 在说话，从 1.8 秒到 3.9 秒是 `speaker_1` 在说话，依此类推。恭喜你，成功了！

### 6.4 常见问题（FAQ）

*   **Q: 提示 `ModuleNotFoundError: No module named 'pyannote'`**
    *   **A:** 你忘记激活 Conda 环境了！请执行 `conda activate pyannote`，然后再运行 python 脚本。

*   **Q: 提示 `401 Client Error: Unauthorized` 或加载模型失败。**
    *   **A:** 这通常是 Hugging Face 令牌的问题。请检查：
        1.  代码里的令牌是不是真的替换成你自己的了？有没有复制错？
        2.  你是否用申请令牌的账号登录 Hugging Face，并接受了 **两个** 模型（`diarization-3.1` 和 `segmentation-3.0`）的用户协议？这是必须的步骤。

*   **Q: 提示 `FileNotFoundError: [Errno 2] No such file or directory: 'audio.wav'`**
    *   **A:** Python 脚本找不到 `audio.wav`。请用 `ls` 命令检查当前目录下是否存在这个文件。如果不存在，请重新上传。如果存在但名字不同（例如 `my_audio.wav`），请修改代码中 `audio_file = "audio.wav"` 这一行。

*   **Q: 运行速度很慢。**
    *   **A:** 检查脚本的输出，看是否有 "检测到 GPU" 的提示。如果没有，说明 PyTorch 没有正确安装成 GPU 版本。请返回 **步骤 5.1**，严格按照 PyTorch 官网的说明，根据你的 CUDA 版本重新安装。

## 结语

恭喜你完成了本教程！你不仅成功运行了 `pyannote-audio`，更重要的是，你已经迈出了进入 Linux 世界的第一步，学会了如何管理服务器、配置环境和解决基本问题。这是进行任何深度学习或服务器开发工作的核心技能。

**不要忘记**：当你不再使用服务器时，在 AutoDL 控制台将其**关机**，以停止计费！

希望这篇教程对你有所帮助。继续探索，享受编码的乐趣吧！
