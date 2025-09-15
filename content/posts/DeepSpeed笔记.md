##### 1.环境搭建

安装conda：https://www.anaconda.com/products/distribution

安装成功后执行以下命令安装webUI运行环境：

```shell
conda create --name py_SD1 python=3.10
```

安装成功后会提示以下信息：

```shell
#
# To activate this environment, use
#
#     $ conda activate py_SD1
#
# To deactivate an active environment, use
#
#     $ conda deactivate
```

执行上面给的命令`conda activate py_SD1`切换环境。

然后开始安装WebUI：

```shell
# linux可以直接wget sh脚本：bash <(wget -qO- https://raw.githubusercontent.com/AUTOMATIC1111/stable-diffusion-webui/master/webui.sh)

# mac使用git下载源码
git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui
cd stable-diffusion-webui/
./webui.sh

# 安装过程中提示unable to access 'https://github.com/Stability-AI/stablediffusion.git/': HTTP/2 stream 1 was not closed cleanly before end of the underlying stream
# 解决：修改git默认http协议版本：git config --global http.version HTTP/1.1

# git clone超时问题：
# git config --global --unset http.proxy

# 安装完后会自动启动，但是一般都会启动失败，提示To create a public link, set `share=True` in `launch()`：
```

启动WebUI：

```shell
# 如果没有--no-gradio-queue参数的话不能开代理，否则访问后面的url会报错
bash webui.sh --share --listen --no-gradio-queue
```

看到如下截图表示启动成功：

![Xnip2023-06-23_18-52-03](https://raw.githubusercontent.com/wangxiaohong123/p-bed/main/uPic/Xnip2023-06-23_18-52-03.png)

可以在浏览器中输入http://localhost:7860/访问diffusion webUI

**mac m系列芯片使用自带的mps来适配pytorch，其他GPU使用CUDA适配，需要单独安装**

另起窗口新建并切换conda环境：

```shell
# 新建chat01环境
conda create -n chat01 python=3.10 anaconda
conda activate chat01
```

下载DeepSpeedExamples源码并切换目录

```shell
git clone https://github.com/microsoft/DeepSpeedExamples.git
cd DeepSpeedExamples/applications/DeepSpeed-Chat/

# 安装依赖
# 这里需要开启clash X的复制终端代理命令，要不然下载很慢，使用镜像都不行
pip install -r requirements.txt
```

安装mac m1对应的pytorch：

```shell
pip3 install --pre torch torchvision torchaudio --extra-index-url https://download.pytorch.org/whl/nightly/cpu
```

##### 2.运行训练demo

训练的代码就是DeepSpeedExamples/applications/DeepSpeed-Chat目录：

```shell
- train.py  # 入口程序
- training  # 训练脚本
  - step1_supervised_finetuning   # 第一步训练
    - evaluation_scripts      # 第一步训练完成后评价用
    - training_scripts        # 模型训练脚本
    - README.md               # 说明文档
    - main.py                 # 主程序，训练过程的实现细节
    - prompt_eval.py          # 评价主程序
  - step2_reward_model_finetuning # 第二步训练
    - 省略
  - step3_rlhf_finetuning         # 第三步训练
    - 省略
  - utils 模型训练，评价的相关函数库
- inference # 测试，评价代码
```

###### 2.1 模型的训练流程

-   a) 监督微调（SFT：supervised finetuning）（对应 DS-Chat 中的 Step1）；
-   b) 奖励模型微调（RM：Reward model finetuning）（对应 DS-Chat 中的 Step2）；
-   c) 基于人类反馈的强化学习（RLHF：Reinforcement learning with human feedback）（对应 DS-Chat 中的 Step3）。

相对于ChatGPT的训练流程这里少了一个预训练过程，这步是最耗时的。DeepSpeedExamples使用的是Facebook预训练好的模型，详情：https://arxiv.org/abs/2205.01068。

DS这3步都支持单GPU、多GPU和多node执行。

###### 2.2 监督微调







ALTER TABLE `stars_user`.`login_device` 
DROP INDEX `first_register_id`,
ADD UNIQUE INDEX `first_register_id`(`first_register_id` ASC, `id`) USING BTREE;
