# lerobot开发流程

![img](https://alidocs.dingtalk.com/core/api/resources/img/5eecdaf48460cde5b47413fbdf55368e4acc223264ed006575b8339e1c4c24831b75b38faadcd24bec177c308ebd53049e7cf433a792252d0352ccf87ab29f9d007c06bb09b3301683a90728f3fcce81aae348cf83b2c8594fb4c8ed7016461c?tmpCode=56bbdfe2-e24d-404c-ab24-1095febe17d0)

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

![img](https://alidocs.dingtalk.com/core/api/resources/img/5eecdaf48460cde5b47413fbdf55368e4acc223264ed006575b8339e1c4c24831b75b38faadcd24bec177c308ebd53048298e2379da977c38d29cb20b1bc33854d23f94037893344bce90ea344da028f2a516dd71d6c514c4fb4c8ed7016461c?tmpCode=56bbdfe2-e24d-404c-ab24-1095febe17d0)

具体连接：

- 连接驱动板与机械臂：
  - 简化版：任意一个驱动板连接到机械臂最下面的电机

具体解释：机械臂上的各个电机已经设置好了对应ID序号，将驱动板连接到机械臂的1号电机，其余电机通过连接线依次串联驱动

![img](https://alidocs.dingtalk.com/core/api/resources/img/5eecdaf48460cde5b47413fbdf55368e4acc223264ed006575b8339e1c4c24831b75b38faadcd24bec177c308ebd5304237bc6c62184b47381db900b2615c7b11f3e068a1f36839f246a4a3bf86350eba7fa8dea3cd378884fb4c8ed7016461c?tmpCode=56bbdfe2-e24d-404c-ab24-1095febe17d0)

- 用USB线连接驱动板与电脑
  - 简化版：先插到电脑上的USB线连接主动臂，后插到电脑上的USB线连接从动臂（注意：最好将驱动板连接到**USB扩展坞**上，电脑本身的USB接口留给后面的摄像头）

具体解释：

在之前的设置中，已经将主动臂的端口号设置为/dev/ttyACM0；从动臂为/dev/ttyACM1。

设置端口的代码储存在lerobot/common/robot_devices/robots/configs.py文件中

![img](https://alidocs.dingtalk.com/core/api/resources/img/5eecdaf48460cde5b47413fbdf55368e4acc223264ed006575b8339e1c4c24831b75b38faadcd24bec177c308ebd5304ef5e48a51e9166e258b91737a51b6f5e176e7d7cfa3c74bb36e6aa60bcb7e6cf2b8ab8bc43b7127a4fb4c8ed7016461c?tmpCode=56bbdfe2-e24d-404c-ab24-1095febe17d0)

在linux系统中，系统会给先插入的USB线分配端口号/dev/ttyACM0，后插入的的USB线分配端口号/dev/ttyACM1（**与端口的位置没关系！**）



判断端口号的方法（更换电脑等设备时可能需要）：

运行命令：python lerobot/scripts/find_motors_bus_port.py（将想要查看的端口号插入USB后运行）

结果如下图：

![img](https://alidocs.dingtalk.com/core/api/resources/img/5eecdaf48460cde5b47413fbdf55368e4acc223264ed006575b8339e1c4c24831b75b38faadcd24bec177c308ebd5304a03dca4a4272f85df96f2fbd9f493237a541c15151d0504cbb214ad4e3b562148c4b8b92bc6031414fb4c8ed7016461c?tmpCode=56bbdfe2-e24d-404c-ab24-1095febe17d0)

此时拔出想要查看的端口USB，然后按下Enter键，此时会显示刚才拔出USB的端口号，如下图：

![img](https://alidocs.dingtalk.com/core/api/resources/img/5eecdaf48460cde5b47413fbdf55368e4acc223264ed006575b8339e1c4c24831b75b38faadcd24bec177c308ebd530465f2c08563dbb70151c22b38e731b340c3633c5e7ec7b32744ff1931c7a64760ff7be7cbb82f153f4fb4c8ed7016461c?tmpCode=56bbdfe2-e24d-404c-ab24-1095febe17d0)

以及USB接线：

![img](https://alidocs.dingtalk.com/core/api/resources/img/5eecdaf48460cde5b47413fbdf55368e4acc223264ed006575b8339e1c4c24831b75b38faadcd24bec177c308ebd5304ca106fae6f818b1b7c334a86c295307183c017da6d058666eecaab68cc33fb251660d0815640afcc4fb4c8ed7016461c?tmpCode=56bbdfe2-e24d-404c-ab24-1095febe17d0)



- 给驱动板连接电源：
  - 简化版：将5v电源连接主动臂的驱动板；12v电源连接从动臂（连接后如果电机正常亮红灯表示连接好了），如果出现：
    - 电机红灯闪烁：先检查电源，是不是把12V电源连接到主动臂上了；是不是连续使用时间过长，让电机休息一下
    - 电机不显示红灯：检查一下是不是没接线，或者线连接顺序有问题

具体解释：

用F1-F6来代表Follower机械臂的1到6的关节舵机，L1-L6来代表Leader机械臂从1到6的关节舵机,对应的舵机型号关节及减速比信息如下：

![img](https://alidocs.dingtalk.com/core/api/resources/img/5eecdaf48460cde5b47413fbdf55368e4acc223264ed006575b8339e1c4c24831b75b38faadcd24bec177c308ebd53045c9a13e4456d0ec8c619fce9808346a82e472d66b3b9c46b36990e1444ad478321ca4d610cf51c0b4fb4c8ed7016461c?tmpCode=56bbdfe2-e24d-404c-ab24-1095febe17d0)



- 连接摄像头：将摄像头连接到电脑本身的USB接口上

具体解释：

摄像头的配置文件也写在lerobot/lerobot/common/robot_devices/robots/configs.py中，这里写好了camera_index分别是4和6，由于使用了两个摄像头占用了电脑的USB，这个应该不需要调整

![img](https://alidocs.dingtalk.com/core/api/resources/img/5eecdaf48460cde5b47413fbdf55368e4acc223264ed006575b8339e1c4c24831b75b38faadcd24bec177c308ebd53048289d08eae89dd0b1605d54bfdf5f9d8cb988378a95644caed1aa7fabbf957b41c7774a6e7652bb44fb4c8ed7016461c?tmpCode=56bbdfe2-e24d-404c-ab24-1095febe17d0)

如果camera_index有误，无法连接摄像头，运行：

python lerobot/common/robot_devices/cameras/opencv.py \

​    --images-dir outputs/images_from_opencv_cameras

这个命令会使用检测到的摄像头拍一些照片，可以在 outputs/images_from_opencv_cameras 目录中找到每个摄像头拍摄的图片，看一下文件名中的数字，这个就是对应摄像头的camera_index，找到后修改上面的configs.py文件即可，拍摄的图片如下图：

![img](https://alidocs.dingtalk.com/core/api/resources/img/5eecdaf48460cde5b47413fbdf55368e4acc223264ed006575b8339e1c4c24831b75b38faadcd24bec177c308ebd5304b290e22dad5d1706f8ea589a3ad5bbe7716bd8dc2cd413fea8e9981b4fab534a1cac4d0a3dc1d0184fb4c8ed7016461c?tmpCode=56bbdfe2-e24d-404c-ab24-1095febe17d0)



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

![img](https://alidocs.dingtalk.com/core/api/resources/img/5eecdaf48460cde5b47413fbdf55368e4acc223264ed006575b8339e1c4c24831b75b38faadcd24bec177c308ebd53041ca6dc648aab42aea8c33bd5fdee95cc0cd4e1e22fbdda1ddc3777201ee55691a995dd5be5f7c9214fb4c8ed7016461c?tmpCode=56bbdfe2-e24d-404c-ab24-1095febe17d0)

- 输出结果如下图：![img](https://alidocs.dingtalk.com/core/api/resources/img/5eecdaf48460cde5b47413fbdf55368e4acc223264ed006575b8339e1c4c24831b75b38faadcd24bec177c308ebd53040c02039502a6ffa15505a4557342f6db2dcf6df5a58bf5badf65fa81f52d014e862ab6b4fed946cd4fb4c8ed7016461c?tmpCode=56bbdfe2-e24d-404c-ab24-1095febe17d0)

# **带摄像头遥操作**

- 运行下面的命令，能够在遥操作时在计算机上显示摄像头

```
python lerobot/scripts/control_robot.py \
  --robot.type=so101 \
  --control.type=teleoperate \
  --control.display_data=true
```

- 运行后会弹出窗口显示两个摄像头实时的拍摄内容![img](https://alidocs.dingtalk.com/core/api/resources/img/5eecdaf48460cde5b47413fbdf55368e4acc223264ed006575b8339e1c4c24831b75b38faadcd24bec177c308ebd530496e51c94ead9861795d9c09c1cef3193897acdefb0be7f2ded26c679a472c491d236bab539ae35b44fb4c8ed7016461c?tmpCode=56bbdfe2-e24d-404c-ab24-1095febe17d0)

如果第一次弹出了窗口，后面运行的时候没有弹出，可能是窗口一直在后台，点击最左上角的按钮显示所有窗口就可以看到。



# **数据集制作采集**

## **连接Hugging Face制作**

- 创建 Hugging Face 令牌，进入https://huggingface.co/settings/tokens，创建一个新的访问令牌（创建的时候把能选择的访问权限都选上）

![img](https://alidocs.dingtalk.com/core/api/resources/img/5eecdaf48460cde5b47413fbdf55368e4acc223264ed006575b8339e1c4c24831b75b38faadcd24bec177c308ebd5304b90510f2fd5b1dcd65871d6a45cd5ab372c9a0978ad5e9e9c8616b51ffc3a12306787cec16a2b7804fb4c8ed7016461c?tmpCode=56bbdfe2-e24d-404c-ab24-1095febe17d0)

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

![img](https://alidocs.dingtalk.com/core/api/resources/img/5eecdaf48460cde5b47413fbdf55368e4acc223264ed006575b8339e1c4c24831b75b38faadcd24bec177c308ebd53045e9da0ea090503a535c15a9ecd96974c5d207fdde8f6632741ad6d997e7420d97799beb61226a1f44fb4c8ed7016461c?tmpCode=56bbdfe2-e24d-404c-ab24-1095febe17d0)

训练的显存占用：

![img](https://alidocs.dingtalk.com/core/api/resources/img/5eecdaf48460cde5b47413fbdf55368e4acc223264ed006575b8339e1c4c24831b75b38faadcd24bec177c308ebd5304f9e1cc422cab7efbd8c7c3100fe4dffb66fbf815d5c111fd4d0f3cd6dc5511e76a27da5dc7b63a634fb4c8ed7016461c?tmpCode=56bbdfe2-e24d-404c-ab24-1095febe17d0)

- 可以在`outputs/train/act_so100_test/checkpoints` 中找到训练结果的权重文件（每20000次训练会自动保存一次模型权重），模型文件大小约为200MB

![img](https://alidocs.dingtalk.com/core/api/resources/img/5eecdaf48460cde5b47413fbdf55368e4acc223264ed006575b8339e1c4c24831b75b38faadcd24bec177c308ebd5304bd204e2fa55e7762bc79335e57218bb9ee10fb9ee3016f114f7b11cb4f306238a5c78c22badcfd474fb4c8ed7016461c?tmpCode=56bbdfe2-e24d-404c-ab24-1095febe17d0)![img](https://alidocs.dingtalk.com/core/api/resources/img/5eecdaf48460cde5b47413fbdf55368e4acc223264ed006575b8339e1c4c24831b75b38faadcd24bec177c308ebd53040aa0a05f3acafcdb4ebd88e38031d3f6a0026757507a3c24136c37173304752c3efd04052c15ce734fb4c8ed7016461c?tmpCode=56bbdfe2-e24d-404c-ab24-1095febe17d0)



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

![img](https://alidocs.dingtalk.com/core/api/resources/img/5eecdaf48460cde5b47413fbdf55368e4acc223264ed006575b8339e1c4c24831b75b38faadcd24bec177c308ebd5304f1d1aaa7a10bd0e397f255cedf046998c74cf794b407507725f89a9f75282c3440de34750495bede4fb4c8ed7016461c?tmpCode=56bbdfe2-e24d-404c-ab24-1095febe17d0)

## **feetech debug software**

feetech debug software软件可以查看电机连接通信等情况，用于前期的调试，具体使用流程可以参考https://zhuanlan.zhihu.com/p/345309655

关键点：

- 需要使用Windows系统下载
- 打开后会自动搜索到端口号，选择波特率为1000000（最大的那个），选择**打开**（容易忽略），点击搜索可以在下方看到连接的所有电机以及ID（根据这里可以判断电机的ID以及连接是否有问题）

![img](https://alidocs.dingtalk.com/core/api/resources/img/5eecdaf48460cde5b47413fbdf55368e4acc223264ed006575b8339e1c4c24831b75b38faadcd24bec177c308ebd5304d86ca86c41d076297c1437d2607c07138e6da44452d580018b46888b2b2f89b660ba644a258ba70e4fb4c8ed7016461c?tmpCode=56bbdfe2-e24d-404c-ab24-1095febe17d0)

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
