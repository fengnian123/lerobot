# **整体流程**

![img](https://github.com/fengnian123/lerobot/blob/main/image/bbb01233b9a6460fb3f4857c41d99eed.png)

前置步骤已经完成，下文主要从**数据采集**开始，引用内容为设备更换或出现问题时可以参考的内容



# **环境搭建**

- 激活环境：

```
conda activate lerobot
```

- 进入lero文件夹

```
cd ~/lerobot
```

从零开始环境搭建：

步骤1：安装Miniconda管理Python环境（注意Miniconda的版本，这里使用Linux-x86，根据实际需要替换）

```
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86.sh
sh ./Miniconda3-latest-Linux-aarch64.sh
source ~/.bashrc
```

步骤2：创建并激活conda环境，克隆Lerobot仓库，安装飞特舵机的驱动

```
conda create -y -n lerobot python=3.10 && conda activate lerobot
git clone https://github.com/huggingface/lerobot.git ~/lerobot
cd ~/lerobot && pip install -e ".[feetech]"
```

步骤3：如果是Linux用户，还需要安装一些额外的依赖：

```
conda install -y -c conda-forge ffmpeg
pip uninstall -y opencv-python
conda install -y -c conda-forge "opencv>=4.10.0"
```

原文链接：https://blog.csdn.net/u010634066/article/details/145903175



# **线路连接**

整体连接状态如下：

- 整体为白色的是主动臂，整体为黑色的是从动臂
- 两个电源分别为 12v 与 5v （电源上有写）

![img](https://github.com/fengnian123/lerobot/blob/main/image/IMG20250616150717.jpeg)

具体连接：

- 连接驱动板与机械臂：
  - 简化版：任意一个驱动板连接到机械臂最下面的电机

具体解释：机械臂上的各个电机已经设置好了对应ID序号，将驱动板连接到机械臂的1号电机，其余电机通过连接线依次串联驱动

![img](https://github.com/fengnian123/lerobot/blob/main/image/9f85edebf45b447caec3c570ec1acf51.png)

- 用USB线连接驱动板与电脑
  - 简化版：先插到电脑上的USB线连接主动臂，后插到电脑上的USB线连接从动臂（注意：最好将驱动板连接到**USB扩展坞**上，电脑本身的USB接口留给后面的摄像头）

具体解释：

在之前的设置中，已经将主动臂的端口号设置为/dev/ttyACM0；从动臂为/dev/ttyACM1。

设置端口的代码储存在lerobot/common/robot_devices/robots/configs.py文件中

![img](https://github.com/fengnian123/lerobot/blob/main/image/%E6%88%AA%E5%B1%8F2025-06-16%2015.34.44.png)

在linux系统中，系统会给先插入的USB线分配端口号/dev/ttyACM0，后插入的的USB线分配端口号/dev/ttyACM1（**与端口的位置没关系！**）



判断端口号的方法（更换电脑等设备时可能需要）：

运行命令：python lerobot/scripts/find_motors_bus_port.py（将想要查看的端口号插入USB后运行）

结果如下图：

![img](https://github.com/fengnian123/lerobot/blob/main/image/mmexport1750217907174.png)

此时拔出想要查看的端口USB，然后按下Enter键，此时会显示刚才拔出USB的端口号，如下图：

![img](https://github.com/fengnian123/lerobot/blob/main/image/mmexport1750217908064.png)

以及USB接线：

![img](https://github.com/fengnian123/lerobot/blob/main/image/%E6%88%AA%E5%B1%8F2025-06-17%2015.50.38.png)



- 给驱动板连接电源：
  - 简化版：将5v电源连接主动臂的驱动板；12v电源连接从动臂（连接后如果电机正常亮红灯表示连接好了），如果出现：
    - 电机红灯闪烁：先检查电源，是不是把12V电源连接到主动臂上了；是不是连续使用时间过长，让电机休息一下
    - 电机不显示红灯：检查一下是不是没接线，或者线连接顺序有问题

具体解释：

用F1-F6来代表Follower机械臂的1到6的关节舵机，L1-L6来代表Leader机械臂从1到6的关节舵机,对应的舵机型号关节及减速比信息如下：

![img](https://github.com/fengnian123/lerobot/blob/main/image/%E6%88%AA%E5%B1%8F2025-06-16%2015.46.08.png)



- 连接摄像头：将摄像头连接到电脑本身的USB接口上

具体解释：

摄像头的配置文件也写在lerobot/lerobot/common/robot_devices/robots/configs.py中，这里写好了camera_index分别是4和6，由于使用了两个摄像头占用了电脑的USB，这个应该不需要调整

![img](https://github.com/fengnian123/lerobot/blob/main/image/%E6%88%AA%E5%B1%8F2025-06-17%2016.02.40.png)

如果camera_index有误，无法连接摄像头，运行：

python lerobot/common/robot_devices/cameras/opencv.py \

​    --images-dir outputs/images_from_opencv_cameras

这个命令会使用检测到的摄像头拍一些照片，可以在 outputs/images_from_opencv_cameras 目录中找到每个摄像头拍摄的图片，看一下文件名中的数字，这个就是对应摄像头的camera_index，找到后修改上面的configs.py文件即可，拍摄的图片如下图：

![img](https://github.com/fengnian123/lerobot/blob/main/image/mmexport1750217910041.png)



# **遥操作**

- 运行以下命令来赋予 USB 端口访问权限：

```
sudo chmod 666 /dev/ttyACM*
```

- 运行下面的命令，移动主动臂，从动臂会跟随移动

```
python lerobot/scripts/control_robot.py \
  --robot.type=so101 \
  --robot.cameras='{}' \
  --control.type=teleoperate
```

注意：夹取部分张开的范围有限制，如果张开过大的话会报错（）

![img](https://github.com/fengnian123/lerobot/blob/main/image/%E6%88%AA%E5%B1%8F2025-06-17%2015.35.53.png)

- 输出结果如下图：![img](https://github.com/fengnian123/lerobot/blob/main/image/mmexport1750217908881.png)

# **带摄像头遥操作**

- 运行下面的命令，能够在遥操作时在计算机上显示摄像头

```
python lerobot/scripts/control_robot.py \
  --robot.type=so101 \
  --control.type=teleoperate \
  --control.display_data=true
```

- 运行后会弹出窗口显示两个摄像头实时的拍摄内容![img](https://github.com/fengnian123/lerobot/blob/main/image/mmexport1750217911129.png)

如果第一次弹出了窗口，后面运行的时候没有弹出，可能是窗口一直在后台，点击最左上角的按钮显示所有窗口就可以看到。



# **数据集制作采集**

## **连接Hugging Face制作**

- 创建 Hugging Face 令牌，进入https://huggingface.co/settings/tokens，创建一个新的访问令牌（创建的时候把能选择的访问权限都选上）

![img](https://github.com/fengnian123/lerobot/blob/main/image/%E6%88%AA%E5%B1%8F2025-06-17%2016.16.30.png)

- 命令行登录 Hugging Face（把${HUGGINGFACE_TOKEN}换成刚才创建的访问令牌）：

```
huggingface-cli login --token ${HUGGINGFACE_TOKEN} --add-to-git-credential
```

- 将 Hugging Face 仓库名称存储在一个变量中，运行以下命令：

```
HF_USER=$(huggingface-cli whoami | head -n 1)
echo $HF_USER
```

- 记录数据集同时上传到 Hub：

具体参数：

- warmup-time-s: 指初始化时间。
- episode-time-s: 表示每次收集数据的时间。
- reset-time-s: 是每次数据收集之间的准备时间。
- **num-episodes: 表示预期收集多少组数据。（主要修改这个参数来更改录制的数据集数量）**
- push-to-hub: 决定是否将数据上传到 HuggingFace Hub。

```
python lerobot/scripts/control_robot.py \
  --robot.type=so101 \
  --control.type=record \
  --control.fps=30 \
  --control.single_task="Grasp a lego block and put it in the bin." \
  --control.repo_id=${HF_USER}/so101_test \
  --control.tags='["so101","tutorial"]' \
  --control.warmup_time_s=5 \
  --control.episode_time_s=30 \
  --control.reset_time_s=30 \
  --control.num_episodes=2 \
  --control.display_data=true \
  --control.push_to_hub=true
```

运行命令后开始录制数据集，具体过程：

- 在一个回合的记录过程中任何时候按下右箭头 -> 可提前停止并进入重置状态。可提前停止并进入下一个回合记录。（一般就是**录好了一段动作按右箭头 -> 进入下一段**）
- 在录制或重置到早期阶段时，随时按左箭头 <- 可以重新录制。（录制有失误的时候）
- 在录制过程中随时按 ESCAPE ESC 可提前结束会话，直接进入视频编码和数据集上传。
- 录制好的视频会保存在`/home/modelscope/.cache/huggingface/lerobot/`，可以直接点击视频查看录制的效果
- 可以使用如下命令来复刻录制好的视频当中的操作（`--control.episode=0`表示复刻第一个录制的动作）：

```
python lerobot/scripts/control_robot.py \
  --robot.type=so101 \
  --control.type=replay \
  --control.fps=30 \
  --control.repo_id=${HF_USER}/so101_test \
  --control.episode=0
```

建议：录制操作时在同一个场景下有要有一些**差别**（比如物体位置、选择角度等等），提高模型的适用性

## **本地制作**

- 记录数据集：

具体参数：

- --control.repo_id改为本地的文件夹目录（注意需要二级目录，如data/test，否则会报错）
- push-to-hub改为false

```
python lerobot/scripts/control_robot.py \
  --robot.type=so101 \
  --control.type=record \
  --control.fps=30 \
  --control.single_task="Grasp a lego block and put it in the bin." \
  --control.repo_id=data/test \
  --control.tags='["so101","tutorial"]' \
  --control.warmup_time_s=5 \
  --control.episode_time_s=30 \
  --control.reset_time_s=30 \
  --control.num_episodes=2 \
  --control.display_data=true \
  --control.push_to_hub=false
```

- 同样可以使用如下命令来复刻录制好的视频当中的操作（`--control.episode=0`表示复刻第一个录制的动作，注意添加`--control.local_files_only=true`使用本地文件夹）：

```
python lerobot/scripts/control_robot.py \
  --robot.type=so101 \
  --control.type=replay \
  --control.fps=30 \
  --control.repo_id=data/test \
  --control.episode=0
  --control.local_files_only=true
```



# **训练**

## **notebook训练**

- 首先将文件夹上传到魔搭notebook（推荐先上传到github，再在notebook上拉取）
- 在notebook中安装环境：

```
pip install -e ".[feetech]"
```

- 这里与本地不同的是，由于没有使用conda，需要改一下numpy版本：

```
# 卸载当前 NumPy 2.x
!pip uninstall numpy
# 安装 NumPy 1.x（推荐 1.26.x，与 Python 3.11 兼容）
!pip install numpy==1.26.4
```

- 输入训练脚本：

具体参数：

- --dataset.repo_id：提供了数据集本地路径或上传到Huggingface的路径（根据收集时的设置即可）
- policy.type：训练模型，目前官方使用ACT，也可以选择diffusion、pi0、pi0fast、tdmpc、vqbet等策略（但官方仅测试了ACT的效果）
- policy.device：有显卡的话使用`cuda`，没有的话用`cpu`

```
python lerobot/scripts/train.py \
  --dataset.repo_id=data/first-test-3 \
  --policy.type=act \
  --output_dir=outputs/train/act_so101_test \
  --job_name=act_so101_test \
  --policy.device=cuda \
  --wandb.enable=false
```

- 训练过程中可以看到：

训练的默认参数为：

- cfg.steps =100000 ：训练步数默认为100K，在notebook大约需要几个小时
- dataset.num_frames=2183 (2K)：模型会使用2183 帧的数据作为输入来进行预测。
- dataset.num_episodes=3 ：数据的视频数量，这里的示例是三个，实际过程中推荐至少记录 50 个场景，每个位置 10 个场景
- num_total_params=51597238 (52M)：模型的参数量，使用的是52M的模型

![img](https://github.com/fengnian123/lerobot/blob/main/image/mmexport1750223353327.png)

训练的显存占用：

![img](https://github.com/fengnian123/lerobot/blob/main/image/%E6%88%AA%E5%B1%8F2025-06-18%2013.54.13.png)

- 可以在`outputs/train/act_so100_test/checkpoints` 中找到训练结果的权重文件（每20000次训练会自动保存一次模型权重），模型文件大小约为200MB

![img](https://github.com/fengnian123/lerobot/blob/main/image/%E6%88%AA%E5%B1%8F2025-06-18%2014.04.24.png)![img](https://github.com/fengnian123/lerobot/blob/main/image/%E6%88%AA%E5%B1%8F2025-06-18%2015.40.11.png)



## **本地训练（跑不动）**

- 训练脚本相似，policy.device改为cpu：

```
python lerobot/scripts/train.py \
  --dataset.repo_id=${HF_USER}/so101_test \
  --policy.type=act \
  --output_dir=outputs/train/act_so101_test \
  --job_name=act_so101_test \
  --policy.device=cpu \
  --wandb.enable=false
```

- 可以在`outputs/train/act_so100_test/checkpoints` 中找到训练结果的权重文件。



# **评估与测试**

- 使用下面的命令进行测试：

与记录训练集的命令类似，只有几处变化：

- --control.policy.path ：模型权重的路径
- 数据集的名称以 eval 开头，表示正在运行推理（例如 data/eval_act_so100_test）

```
python lerobot/scripts/control_robot.py \
  --robot.type=so101 \
  --control.type=record \
  --control.fps=30 \
  --control.single_task="Grasp a lego block and put it in the bin." \
  --control.repo_id=data/eval_act_so101_test \
  --control.tags='["tutorial"]' \
  --control.warmup_time_s=5 \
  --control.episode_time_s=30 \
  --control.reset_time_s=30 \
  --control.num_episodes=10 \
  --control.push_to_hub=true \
  --control.policy.path=outputs/train/act_so101_test/checkpoints/last/pretrained_model
```

- 测试效果：经测试，成功还是比较高的

<video controls src="[https://github.com/fengnian123/lerobot/blob/main/outputs/VID20250619135943.mp4](https://github.com/fengnian123/lerobot/blob/main/image/VID20250619135943.mp4)" width="500" height="300"></video>
```HTML
<video src="https://github.com/fengnian123/lerobot/blob/main/outputs/VID20250619135943.mp4" controls="controls" width="500" height="300"></video>
```



# **报错合集**

- 电机不在可以选择的范围，可能的原因：电机ID设置不对 / 校准时位置没有正确读取（已解决）

```
ValueError: No integer found between bounds [low_factor=np.float32(-0.0007324219), upp_factor=np.float32(-0.0007324219)]
```

- **最常见**问题，整体连接有问题，可能的原因：电源没开 / 电机版本较低需要升级（已解决） / 线没接好

```
ConnectionError: Read failed due to communication error on port COM10 for group_key Present_Position_shoulder_pan_shoulder_lift_elbow_flex_wrist_flex_wrist_roll_gripper: [TxRxResult] There is no status packet!
```

- 端口权限问题，输入`sudo chmod 777`加对应端口

```
Permission denied……
```



# **其他**

## **电脑使用**

- 双系统使用：如果电脑重启，会默认进入Linux系统，如果需要使用Windows系统，在下图的界面中选择Windows Boot Manager选项

![img](https://alidocs.dingtalk.com/core/api/resources/img/5eecdaf48460cde5b47413fbdf55368e4acc223264ed006575b8339e1c4c24831b75b38faadcd24bec177c308ebd5304f1d1aaa7a10bd0e397f255cedf046998c74cf794b407507725f89a9f75282c3440de34750495bede4fb4c8ed7016461c?tmpCode=849758f7-f581-4789-ac00-a457ee27ac08)

- Windows系统密码：contestant0017
- Linux系统密码：modelscope；Linux sudo密码：modelscope

## **feetech debug software**

feetech debug software软件可以查看电机连接通信等情况，用于前期的调试，具体使用流程可以参考https://zhuanlan.zhihu.com/p/345309655

关键点：

- 需要使用Windows系统下载
- 打开后会自动搜索到端口号，选择波特率为1000000（最大的那个），选择**打开**（容易忽略），点击搜索可以在下方看到连接的所有电机以及ID（根据这里可以判断电机的ID以及连接是否有问题）

![img](https://alidocs.dingtalk.com/core/api/resources/img/5eecdaf48460cde5b47413fbdf55368e4acc223264ed006575b8339e1c4c24831b75b38faadcd24bec177c308ebd5304d86ca86c41d076297c1437d2607c07138e6da44452d580018b46888b2b2f89b660ba644a258ba70e4fb4c8ed7016461c?tmpCode=849758f7-f581-4789-ac00-a457ee27ac08)

- 点击“升级”可以把电机从3.9升级到3.10



## **Windows使用Ubuntu子系统的问题**

如果使用 Window下的Ubuntu子系统，会出现USB无法识别的情况，使用下面的流程（详见https://learn.microsoft.com/zh-cn/windows/wsl/connect-usb）：

- 安装 USBIPD

```
sudo apt-get install usbip
```

- 输入以下命令列出连接到 Windows 的所有 USB 设备（主要记录USB的端口号）。

```
usbipd list
```

- 使用下面的命令来共享设备（<busid>更换为刚才记录的USB端口号）

```
usbipd bind --busid <busid>
```

- 运行以下命令挂载USB端口（<busid>更换为刚才记录的USB端口号）

```
usbipd attach --wsl --busid <busid>
```



# **参考内容**

- 文档：

https://blog.csdn.net/u010634066/article/details/145903175

https://huggingface.co/docs/lerobot/so101

https://wiki.seeedstudio.com/lerobot_so100m/

- 视频：

https://www.bilibili.com/video/BV1TQmYYPE4b/?vd_source=03b39476c1d4fd1a0e24dbcc62560bdf
