# AutoCut: Cut video with subtitles
AutoCut automatically generates subtitles for your videos. Then you select the sentences you want to keep, and AutoCut will crop and save the corresponding segments in your video. You don't need to use video editing software, just edit the text file to complete the cut.

# Example of use
Suppose your recorded video is placed in the 2022-11-04/ folder. then run

autocut -d 2022-11-04
Tip: If you use OBS to record the screen, you can change the space to / in Settings->Advanced->Recording->File Name Format, which is %CCYY-%MM-%DD/%hh-%mm-%ss. Then the video files will be placed in the date-named folder.

AutoCut will continue to extract and cut subtitles for the videos in this folder. For example, you just finished a video recording and saved it in 11-28-18.mp4. AutoCut will generate 11-28-18.md. After you select the sentence you want to keep in it, AutoCut will cut out 11-28-18_cut.mp4 and generate 11-28-18_cut.md to preview the result.

You can use any Markdown editor. For example, I often use VS Code and Typora. The picture below is the editing of 11-28-18.md by Typora.



After all is completed, after selecting the video to be spliced ​​in autocut.md, AutoCut will output autocut_merged.mp4 and the corresponding subtitle file.

Install
First install the Python package

pip install git+https://github.com/mli/autocut.git
The above will install pytorch. If you need the GPU to run and the version installed by default does not match, you can install Pytorch first. If you have problems installing Whipser, please refer to the official documentation.

Also need to install ffmpeg

# on Ubuntu or Debian
sudo apt update && sudo apt install ffmpeg

# on Arch Linux
sudo pacman -S ffmpeg

# on MacOS using Homebrew (https://brew.sh/)
brew install ffmpeg

# on Windows using Scoop (https://scoop.sh/)
scoop install ffmpeg
More usage options
Transcribe a video to generate .srt and .md results.
autocut -t 22-52-00.mp4
If you are not satisfied with the transcription quality, you can use a larger model, e.g.

autocut -t 22-52-00.mp4 --whisper-model large
The default is small. Better models are medium and large, but GPU is recommended for better speed. Faster tiny and base can also be used, but the transcription quality will be reduced.

If you have a lot of long pauses in your video, you can use --vad to pre-identify these pauses using an extra VAD model, making the timestamp recognition more accurate.

cut a video
autocut -c 22-52-00.mp4 22-52-00.srt 22-52-00.md
The default video bitrate is --bitrate 10m, you can adjust it as needed.

If you are not used to Markdown files, you can also delete unnecessary sentences in the srt file directly without passing in the md file name when cutting. is autocut -c 22-52-00.mp4 22-52-00.srt

If there is only srt file, it is inconvenient to edit, you can use the following command to generate the md file, and then edit the md file, but at this time, it will be generated completely against the srt, and no prompt text such as no speech will appear.

autocut -m test.srt test.mp4
autocut -m test.mp4 test.srt # Support out-of-order incoming video and subtitles
autocut -m test.srt # You can also just pass in the subtitle file
some tips
The fluent video will have a higher quality transcription due to the distribution of the Whisper training data. For a video, you can roughly select the sentence first, and then cut it again on the cut video.

The resulting subtitles for the final video usually require some minor editing as well. You can edit md files directly (more compact than srt files, with embedded video). Then use autocut -s 22-52-00.md 22-52-00.srt to generate the updated subtitle 22-52-00_edited.srt. Note that whether the sentence is selected or not will be ignored here, but all converted to srt.

The resulting subtitles for the final video usually require some minor editing as well. But there are too many blank lines in srt. You can use autocut -s 22-52-00.srt to generate a more compact version of 22-52-00_compact.srt for editing (this format is not legal, but editors such as VS Code will still do syntax highlighting) . After editing, autocut -s 22-52-00_compact.srt converts back to normal format.

Editing markdown with Typora and VS Code is very convenient. They all have corresponding shortcut keys to mark one or more lines. But there seems to be something wrong with the VS Code video preview.

The video is exported via ffmpeg. On the Apple M1 chip it doesn't use the GPU, resulting in not as fast export as professional video software.

common problem
Is the output garbled?

AutoCut default output encoding is utf-8. Make sure your editor also uses utf-8 decoding. You can specify other encoding formats with --encoding. However, it should be noted that the encoding format of the generated subtitle file and the editing of the subtitle file needs to be consistent. For example using gbk.

autocut -t test.mp4 --encoding=gbk
autocut -c test.mp4 test.srt test.md --encoding=gbk
If another encoding format (such as gbk, etc.) is used to generate an md file and open it with Typora, the file may be automatically transcoded into other encoding formats by Typora. At this time, encoding may occur when editing through the encoding format specified at the time of generation. Does not support waiting for an error. Therefore, you can use the editing function after editing with Typora and then modify it to the encoding format you need through VS Code, etc., and then use the editing function.

How to use GPU to transcribe?

When you have an Nvidia GPU and the corresponding version of Pytorch is installed, transcription is done on the GPU. You can use the command to check whether the GPU is currently supported.

python -c "import torch; print(torch.cuda.is_available())"
Otherwise you can manually install the corresponding GPU version of Pytorch before installing autocut.

Using the GPU is an error of insufficient video memory.

Whisper's large models require a certain amount of GPU memory. If you don't have enough video memory, you can use a smaller model, such as small. If you still want to use a large model, you can force the use of cpu with --device. E.g

autocut -t 11-28-18.mp4 --whisper-model large --device cpu
Can it be installed directly with pip install autocut?

Because autocut's dependency on whisper does not release the pypi package, it can only be released by pip install git+https://github.com/mli/autocut.git at present. Students who need it can check whether the whisper model can be downloaded directly from huggingface hub, so as to get rid of the dependence of the whisper package.

How to get involved
code structure
autocut
│ .gitignore
│ LICENSE
│ README.md # Generally, the content of README.md needs to be updated correspondingly if users need to know about new additions or modifications.
│ setup.py
│
└─autocut # The core code is located in the autocut folder, and the implementation of new functions is generally modified or added here
   │ cut.py
   │ daemon.py
   │ main.py
   │ transcribe.py
   │ utils.py
   └─ __init__.py

# Install dependencies
Before starting to install the required dependencies of this project, it is recommended to understand the use of the virtual environment of Anaconda or venv. It is recommended to use the virtual environment to build the development environment of the project. The specific installation method is to follow the above installation steps in the virtual environment you build.

Why is it recommended to use a virtual environment for development?

On the one hand, it is to ensure that various development environments do not pollute each other.

More importantly, this project is actually a Python Package, so the code of autocut will actually become your environment dependency after you install it. So after you update the code, you need to reinstall the new code into the environment before the new code can be called.

# Develop
The code style currently follows PEP-8 and can be done using the relevant auto-formatting software.
utils.py is mainly some tools and methods that are shared globally.
transcribe.py is the part that calls the model to generate srt and md.
cut.py provides the function of cutting and merging videos according to md or srt after marking.
What daemon.py provides is to monitor the folder to generate subtitles and cut videos.
main.py declares command line parameters and calls corresponding functions according to the input parameters.
During the development process, please try to ensure that the modifications are made in the correct place, and the code can be reused reasonably, and the tool functions should be placed in utils.py as much as possible. The code format currently follows PEP-8, and variable naming should be as semantic as possible.

After the development is completed, the most important point is to test. Please ensure that all the parts directly related to your changes and the parts that your changes will affect have been tested before submitting, and ensure that the functions are normal. There is currently no CI with test cases, which will be improved in the future.

# Submit
The commit information can describe the changes you have made clearly in English, starting with lowercase letters.
It is best to ensure that the changes involved in a commit are relatively small, and can be described briefly and clearly, which is also convenient for later search when there are changes.
When PR, the title briefly describes what modifications are made, and the contents can specifically write down the modified content.

-------------------

# AutoCut: 通过字幕来剪切视频

AutoCut对你的视频自动生成字幕。然后你选择需要保留的句子，AutoCut将对你视频中对应的片段裁切并保存。你无需使用视频编辑软件，只需要编辑文本文件即可完成剪切。

## 使用例子

假如你录制的视频放在 `2022-11-04/` 这个文件夹里。那么运行

```bash
autocut -d 2022-11-04
```

> 提示：如果你使用OBS录屏，可以在 `设置->高级->录像->文件名格式` 中将空格改成`/`，既 `%CCYY-%MM-%DD/%hh-%mm-%ss`。那么视频文件将放在日期命名的文件夹里。

AutoCut将持续对这个文件夹里视频进行字幕抽取和剪切。例如，你刚完成一个视频录制，保存在 `11-28-18.mp4`。AutoCut将生成 `11-28-18.md`。你在里面选择需要保留的句子后，AutoCut将剪切出 `11-28-18_cut.mp4`，并生成 `11-28-18_cut.md` 来预览结果。

你可以使用任何的Markdown编辑器。例如我常用VS Code和Typora。下图是通过Typora来对 `11-28-18.md` 编辑。

![](imgs/typora.jpg)

全部完成后在 `autocut.md` 里选择需要拼接的视频后，AutoCut将输出 `autocut_merged.mp4` 和对应的字幕文件。

## 安装

首先安装 Python 包

```
pip install git+https://github.com/mli/autocut.git
```

> 上面将安装 [pytorch](https://pytorch.org/)。如果你需要GPU运行，且默认安装的版本不匹配的话，你可以先安装Pytorch。如果安装 Whipser 出现问题，请参考[官方文档](https://github.com/openai/whisper#setup)。

另外需要安装 [ffmpeg](https://ffmpeg.org/)

```
# on Ubuntu or Debian
sudo apt update && sudo apt install ffmpeg

# on Arch Linux
sudo pacman -S ffmpeg

# on MacOS using Homebrew (https://brew.sh/)
brew install ffmpeg

# on Windows using Scoop (https://scoop.sh/)
scoop install ffmpeg
```

## 更多使用选项

### 转录某个视频生成`.srt`和`.md`结果。

```bash
autocut -t 22-52-00.mp4
```

1. 如果对转录质量不满意，可以使用更大的模型，例如

    ```bash
    autocut -t 22-52-00.mp4 --whisper-model large
    ```

    默认是`small`。更好的模型是`medium`和`large`，但推荐使用GPU获得更好的速度。也可以使用更快的`tiny`和`base`，但转录质量会下降。

2. 如果你视频中有较多的长停顿，可以用`--vad`来使用格外的VAD模型预先识别这些停顿，使得对时间戳识别更准确。


### 剪切某个视频

```bash
autocut -c 22-52-00.mp4 22-52-00.srt 22-52-00.md
```

1. 默认视频比特率是 `--bitrate 10m`，你可以根据需要调大调小。
2. 如果不习惯Markdown文件，你也可以直接在`srt`文件里删除不要的句子，在剪切时不传入`md`文件名即可。就是 `autocut -c 22-52-00.mp4 22-52-00.srt`
3. 如果仅有`srt`文件，编辑不方便可以使用如下命令生成`md`文件，然后编辑`md`文件即可，但此时会完全对照`srt`生成，不会出现`no speech`等提示文本。

   ```bash
   autocut -m test.srt test.mp4
   autocut -m test.mp4 test.srt # 支持视频和字幕乱序传入
   autocut -m test.srt # 也可以只传入字幕文件
   ```


### 一些小提示


1. 讲得流利的视频的转录质量会高一些，这因为是Whisper训练数据分布的缘故。对一个视频，你可以先粗选一下句子，然后在剪出来的视频上再剪一次。

2. ~~最终视频生成的字幕通常还需要做一些小编辑。你可以直接编辑`md`文件（比`srt`文件更紧凑，且嵌入了视频）。然后使用 `autocut -s 22-52-00.md 22-52-00.srt` 来生成更新的字幕 `22-52-00_edited.srt`。注意这里会无视句子是不是被选中，而是全部转换成`srt`。~~

3. 最终视频生成的字幕通常还需要做一些小编辑。但`srt`里面空行太多。你可以使用 `autocut -s 22-52-00.srt` 来生成一个紧凑些的版本 `22-52-00_compact.srt` 方便编辑（这个格式不合法，但编辑器，例如VS Code，还是会进行语法高亮）。编辑完成后，`autocut -s 22-52-00_compact.srt` 转回正常格式。

4. 用Typora和VS Code编辑markdown都很方便。他们都有对应的快捷键mark一行或者多行。但VS Code视频预览似乎有点问题。

5. 视频是通过ffmpeg导出。在Apple M1芯片上它用不了GPU，导致导出速度不如专业视频软件。

### 常见问题

1. **输出的是乱码？**

   AutoCut 默认输出编码是 `utf-8`. 确保你的编辑器也使用了`utf-8`解码。你可以通过`--encoding`指定其他编码格式。但是需要注意生成字幕文件和使用字幕文件剪辑时的编码格式需要一致。例如使用 `gbk`.

    ```bash
    autocut -t test.mp4 --encoding=gbk
    autocut -c test.mp4 test.srt test.md --encoding=gbk
    ```

    如果使用了其他编码格式（如gbk等）生成md文件并用Typora打开后，该文件可能会被Typora自动转码为其他编码格式，此时再通过生成时指定的编码格式进行剪辑时可能会出现编码不支持等报错。因此可以在使用Typora编辑后再通过VS Code等修改到你需要的编码格式进行保存后再使用剪辑功能。

2. **如何使用GPU来转录？**

   当你有Nvidia GPU，而且安装了对应版本的Pytorch的时候，转录是在GPU上进行。你可以通过命令来查看当前是不是支持GPU。

   ```bash
   python -c "import torch; print(torch.cuda.is_available())"
   ```

   否则你可以在安装autocut前手动安装对应的GPU版本Pytorch。

3. **使用GPU是报错显存不够。**

   whisper的大模型需要一定的GPU显存。如果你的显存不够，你可以用小一点的模型，例如`small`。如果你仍然想用大模型，可以通过`--device`来强制使用`cpu`。例如

   ```bash
   autocut -t 11-28-18.mp4 --whisper-model large --device cpu
   ```

4. **能不能直接用 `pip install autocut` 安装?**

   因为autocut的依赖whisper没有发布pypi包，所以目前只能用 `pip install git+https://github.com/mli/autocut.git` 这种方式发布。有需求的同学可以查看whisper模型是不是能直接在 huggingface hub 下载，从而摆脱whisper包的依赖。


## 如何参与贡献
### 代码结构
```text
autocut
│  .gitignore
│  LICENSE
│  README.md # 一般新增或修改需要让使用者知道就需要对应更新 README.md 内容
│  setup.py
│
└─autocut # 核心代码位于 autocut 文件夹中，新增功能的实现也一般在这里面进行修改或新增
   │  cut.py
   │  daemon.py
   │  main.py
   │  transcribe.py
   │  utils.py
   └─ __init__.py

```

### 安装依赖
开始安装这个项目的需要的依赖之前，建议先了解一下 Anaconda 或者 venv 的虚拟环境使用，推荐**使用虚拟环境来搭建该项目的开发环境**。
具体安装方式为在你搭建搭建的虚拟环境之中按照[上方安装步骤](./README.md#安装)进行安装。

> 为什么推荐使用虚拟环境开发？
> 
> 一方面是保证各种不同的开发环境之间互相不污染。
> 
> 更重要的是在于这个项目实际上是一个 Python Package，所以在你安装之后 autocut 的代码实际也会变成你的环境依赖。
> **因此在你更新代码之后，你需要让将新代码重新安装到环境中，然后才能调用到新的代码。**

### 开发

1. 代码风格目前遵循 PEP-8，可以使用相关的自动格式化软件完成。
2. `utils.py` 主要是全局共用的一些工具方法。
3. `transcribe.py` 是调用模型生成`srt`和`md`的部分。
4. `cut.py` 提供根据标记后`md`或`srt`进行视频剪切合并的功能。
5. `daemon.py` 提供的是监听文件夹生成字幕和剪切视频的功能。
6. `main.py` 声明命令行参数，根据输入参数调用对应功能。

开发过程中请尽量保证修改在正确的地方，以及合理地复用代码，
同时工具函数请尽可能放在`utils.py`中。
代码格式目前是遵循 PEP-8，变量命名尽量语义化即可。

在开发完成之后，最重要的一点是需要进行**测试**，请保证提交之前对所有**与你修改直接相关的部分**以及**你修改会影响到的部分**都进行了测试，并保证功能的正常。
目前没有测试用例的CI，会在之后进行完善。

### 提交

1. commit 信息用英文描述清楚你做了哪些修改即可，小写字母开头。
2. 最好可以保证一次的 commit 涉及的修改比较小，可以简短地描述清楚，这样也方便之后有修改时的查找。
3. PR 的时候 title 简述有哪些修改， contents 可以具体写下修改内容。

