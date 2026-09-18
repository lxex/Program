# Powershell 安装 wsl2 和Ubuntu2204 并建立cuda开发环境
## 1. Ubuntu安装和设置
Powershell> wsl -l -v
如果version显示1则：
Powershell> wsl --set-version Ubuntu-22.04 2
安装：
Powershell> wsl --install -d Ubuntu-22.04
如果失败，尝试：
Powershell> wsl --install Ubuntu-22.04
运行方法：
windows搜索Ubuntu直接启动，或者：
Powershell> wsl -d Ubuntu-22.04

Ubuntu root命令下创建账户：
root# adduser shawn
输入密码，其他默认。
切换用户：
root# su - shawn
未来要切回root使用：
shawn@：sudo su
或者：
shawn@：su -

更新 Ubuntu：
sudo apt update && sudo apt upgrade -y
sudo apt install -y build-essential git curl wget unzip

## 2. CUDA安装
安装 NVIDIA CUDA
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-keyring_1.0-1_all.deb
sudo dpkg -i cuda-keyring_1.0-1_all.deb
sudo apt update
sudo apt install -y cuda-toolkit-12-1

先确认 CUDA 是否正确安装，运行：
ls /usr/local/


配置 CUDA 环境变量：
```
echo 'export PATH=/usr/local/cuda-12.1/bin:$PATH' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=/usr/local/cuda-12.1/lib64:$LD_LIBRARY_PATH' >> ~/.bashrc
source ~/.bashrc
```

检查 CUDA 是否安装成功：
nvcc --version

## 3. 安装 PyTorch（CUDA 12.1 版本）
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121

测试：
import torch
print(torch.cuda.is_available())  # True
print(torch.cuda.get_device_name(0))  # NVIDIA RTX 4060

## 4. 安装 训练所需要的库
pip install transformers accelerate datasets peft bitsandbytes loralib
解释：
- transformers → 加载和微调 Hugging Face 模型
- accelerate → GPU 训练加速
- datasets → 处理 JSONL 格式的数据
- peft → 使用 QLoRA 进行低显存微调
- bitsandbytes → 4-bit 量化计算
- loralib → LoRA 低秩适配训练

## 5. 配置代理
Windows代理使用tun模式


## 6. 安装Transformers
pip install --upgrade transformers
检查版本，如果 transformers 版本太低（例如低于 4.30），继续上面一条更新Transformers
python3 -c "import transformers; print(transformers.__version__)"

## 7. 登录huggingface并下载访问token
在 https://huggingface.co/settings/tokens
$ huggingface-cli login
输入：
YOUR_HUGGINGFACE_TOKEN

## 8. 运行下载模型代码，去下载模型基座

```
from transformers import AutoModelForCausalLM, AutoTokenizer

# 预训练模型路径
model_name = "Qwen/Qwen2.5-VL-7B-Instruct"

# 加载模型和分词器
model = AutoModelForCausalLM.from_pretrained(
    model_name,
    token="YOUR_HUGGINGFACE_TOKEN",  # 传入 Hugging Face 访问令牌
    torch_dtype="float16",  # 使用混合精度，以节省显存
    device_map="auto"  # 自动分配模型到设备
)

tokenizer = AutoTokenizer.from_pretrained(model_name)

print("训练前模型加载成功！")

```
如果出现配置错误，尝试使用 Qwen2ForCausalLM 等专用模型类：

```
from transformers import Qwen2ForCausalLM, AutoTokenizer, AutoConfig

# 载入模型配置
config = AutoConfig.from_pretrained("Qwen/Qwen2.5-VL-7B-Instruct")

# 修改 rope_type 为 "standard" 或其他兼容的类型
config.rope_type = "standard"  # 或根据实际情况修改

# 使用修改后的配置加载模型
model = Qwen2ForCausalLM.from_pretrained(
    "Qwen/Qwen2.5-VL-7B-Instruct",
    config=config,
    token=YOUR_HUGGINGFACE_TOKEN",  # 传入 Hugging Face 访问令牌
    torch_dtype="float16",  # 使用混合精度
    device_map="auto"
)

tokenizer = AutoTokenizer.from_pretrained("Qwen/Qwen2.5-VL-7B-Instruct")

print("模型加载成功！")

```
结果显示加载成功：
```
shawn@DESKTOP-R51GKR8:~/dlqwen$ python3 dlqwen.py
Sliding Window Attention is enabled but not implemented for `sdpa`; unexpected results may be encountered.
Loading checkpoint shards: 100%|█████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 5/5 [00:05<00:00,  1.16s/it]
generation_config.json: 100%|█████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 244/244 [00:00<00:00, 2.56MB/s]
Some parameters are on the meta device because they were offloaded to the cpu.
tokenizer_config.json: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 7.23k/7.23k [00:00<00:00, 55.2MB/s]
vocab.json: 100%|█████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2.78M/2.78M [00:00<00:00, 3.12MB/s]
merges.txt: 100%|█████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1.67M/1.67M [00:00<00:00, 5.92MB/s]
tokenizer.json: 100%|█████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 7.03M/7.03M [00:00<00:00, 10.5MB/s]
模型加载成功！
```

## 9. 在如上代码后面添加，训练前测试代码

```
# 训练前测试文本
print("测试前提问：")
inputs = tokenizer("请介绍一下人工智能？\n", return_tensors="pt").to(model.device)
outputs = model.generate(
    **inputs,
    max_length=200,  # 设置最大生成长度
    num_return_sequences=1  # 设置生成的文本数量
)
print(tokenizer.decode(outputs[0], skip_special_tokens=True))
inputs = tokenizer("什么是深度学习？\n", return_tensors="pt").to(model.device)
outputs = model.generate(
    **inputs,
    max_length=200,  # 设置最大生成长度
    num_return_sequences=1  # 设置生成的文本数量
)
print(tokenizer.decode(outputs[0], skip_special_tokens=True))
```
Qwen/Qwen2.5-VL-7B-Instruct回复(约10分钟)：
```
测试前提问：
请介绍一下人工智能？
人工智能（Artificial Intelligence，简称AI）是指通过计算机程序模拟人类智能行为的技术。它涉及机器学习、自然语言处理、计算机视觉等多个领域，旨在使计算机能够执行通常需要人类智能才能完成的任务。

人工智能的发展可以追溯到20世纪50年代，当时科学家们开始探索如何让计算机模拟人类的思维过程。随着技术的进步和数据量的增长，人工智能在近年来取得了显著进展，并在许多领域得到了广泛应用，如医疗诊断、自动驾驶汽车、语音识别等。

人工智能的核心在于算法和模型的设计与优化。通过训练大量的数据集，这些模型可以学习到规律并预测未来的行为。此外，深度学习作为一种强大的机器学习方法，在图像识别、语音识别等领域表现尤为出色。

尽管人工智能带来了许多便利，但也引发了关于隐私保护、就业影响以及伦理道德等方面的讨论。因此，在发展人工智能的同时，也需要关注其潜在风险，并制定相应的政策法规来规范其应用。
什么是深度学习？
深度学习是一种机器学习方法，它模仿人类大脑的神经网络结构来处理和分析大量数据。这种技术通过构建多层神经网络来模拟人类大脑的复杂功能，从而实现对数据的高级理解和预测。

深度学习的主要特点包括：

1. 多层神经网络：深度学习模型通常包含多个隐藏层，这些隐藏层可以捕捉到数据的不同层次特征。
2. 自动特征提取：深度学习模型能够自动从原始数据中提取有用的特征，而无需人工干预。
3. 高级泛化能力：深度学习模型具有强大的泛化能力，能够在未见过的数据上取得较好的性能。
4. 大规模训练：深度学习模型需要大量的计算资源进行训练，因此通常需要高性能计算机或云计算平台的支持。

深度学习已经在许多领域取得了显著的成功，如图像识别、语音识别、自然语言处理等。随着计算能力的不断提升以及算法的不断优化，深度学习的应用范围
```

采用Qwen/Qwen2.5-VL-3B-Instruct回复(约10分钟)：
```
模测试前提问：
请介绍一下人工智能？
人工智能（Artificial Intelligence，简称AI）是指由计算机系统所表现出的智能行为。它是一种模拟人类智能的技术，旨在使计算机能够执行需要人类智能才能完成的任务。

人工智能可以分为弱人工智能和强人工智能两种类型。弱人工智能是指只能在特定任务上表现出来的智能，例如语音识别、图像识别等；而强人工智能则是指具有与人类智能相当甚至超越人类智能的能力，能够在各种领域中进行自主决策和学习。

人工智能的应用非常广泛，包括自然语言处理、机器翻译、自动驾驶、医疗诊断、金融分析、智能家居等领域。其中，深度学习是人工智能的一个重要分支，通过模拟人脑神经网络的工作原理，实现对大量数据的学习和推理。

尽管人工智能已经取得了许多成就，但仍然存在一些挑战和问题，例如数据隐私、算法偏见、伦理道德等问题。因此，人工智能的发展需要在技术、法律和社会等多个方面进行综合考虑和规范。
什么是深度学习？
深度学习是一种机器学习技术，它通过构建多层神经网络来模拟人脑的神经元结构和工作方式。这些神经网络可以自动从大量数据中学习特征，并且能够处理复杂的非线性关系。

深度学习在计算机视觉、自然语言处理、语音识别等领域取得了显著的成果，例如图像分类、物体检测、文本分类等任务。此外，深度学习还可以用于推荐系统、自动驾驶、医疗诊断等领域。

深度学习的核心是神经网络，其中包含多个层次的神经元，每个神经元都与前一层的神经元相连。输入数据被传递到第一层的神经元，然后逐层传递到最后一层的神经元，最终输出结果。每一层的神经元都会对输入进行加权求和，并将结果传递给下 一层的神经元。这个过程可以通过反向传播算法来优化权重，使得模型能够更好地拟合训练数据。

深度学习的优点在于它可以自动提取
```


# 微调模型
## 安装 LoRA 相关依赖
LoRA（低秩适配）能够优化显存使用，减少训练时的显存消耗。
pip install loralib


```
from transformers import Qwen2ForCausalLM, AutoTokenizer, AutoConfig, Trainer, TrainingArguments
from datasets import load_dataset
from peft import LoraConfig, get_peft_model

# 预训练模型路径
model_name = "Qwen/Qwen2.5-VL-3B-Instruct"

# 载入模型配置
config = AutoConfig.from_pretrained(model_name)

# 修改 rope_type 为 "standard" 或其他兼容的类型
config.rope_type = "standard"  # 或根据实际情况修改

# 使用修改后的配置加载模型
model = Qwen2ForCausalLM.from_pretrained(
    model_name,
    config=config,
    token="YOUR_HUGGINGFACE_TOKEN",  # 传入 Hugging Face 访问令牌
    torch_dtype="float16",  # 使用混合精度
    device_map="auto"
)

tokenizer = AutoTokenizer.from_pretrained(model_name)

print("模型加载成功！")

print("测试前提问：")
inputs = tokenizer("请介绍一下人工智能？\n", return_tensors="pt").to(model.device)
outputs = model.generate(
    **inputs,
    max_length=200,  # 设置最大生成长度
    num_return_sequences=1  # 设置生成的文本数量
)
print(tokenizer.decode(outputs[0], skip_special_tokens=True))
inputs = tokenizer("什么是深度学习？\n", return_tensors="pt").to(model.device)
outputs = model.generate(
    **inputs,
    max_length=200,  # 设置最大生成长度
    num_return_sequences=1  # 设置生成的文本数量
)
print(tokenizer.decode(outputs[0], skip_special_tokens=True))

# 加载数据集
dataset = load_dataset("json", data_files={"train": "./train.jsonl"})

# LoRA 配置（低秩适配）
lora_config = LoraConfig(
    r=8,  # 低秩适配维度（可以根据显存调整）
    lora_alpha=32,
    lora_dropout=0.1,
    bias="none",
    task_type="CAUSAL_LM",
    target_modules=["q_proj", "v_proj", "k_proj", "o_proj"]  # 针对自注意力模块的层
)

# 使用 Qwen2.5-VL-3B-Instruct 进行微调
model = get_peft_model(model, lora_config)

# 设置训练参数
training_args = TrainingArguments(
    output_dir="./output",  # 保存训练结果
    per_device_train_batch_size=1,  # 批量大小1，适合显存较小的显卡
    gradient_accumulation_steps=16,
    evaluation_strategy="no",# no:不进行评估，steps进行评估
    save_steps=500,
    eval_steps=500,
    logging_steps=100,
    learning_rate=2e-5,
    weight_decay=0.01,
    fp16=True,  # 使用混合精度，节省显存
    push_to_hub=False,
    remove_unused_columns=False,  # 禁用移除未使用的列
    label_names=["labels"],  # 显式指定标签列
)

# 数据预处理
def preprocess_function(examples):
    # 使用 tokenizer 对 "instruction" 和 "input" 进行编码（即合并为输入）
    inputs = tokenizer(examples["instruction"], examples["input"], truncation=True, padding="max_length", max_length=512)
    
    # 使用 tokenizer 对 "output" 进行编码作为标签
    labels = tokenizer(examples["output"], truncation=True, padding="max_length", max_length=512)
    
    # 将 labels 设为处理后的 "output" tokenized
    inputs["labels"] = labels["input_ids"]
    return inputs

# 应用数据预处理
train_dataset = dataset["train"].map(preprocess_function, batched=True)

# 创建 Trainer 进行训练
trainer = Trainer(
    model=model,
    args=training_args,
    train_dataset=train_dataset
)

# 开始训练
trainer.train()

# 保存微调后的模型和 tokenizer
trainer.save_model("my_trained_qwen")# 保存整个模型及其配置
model.save_pretrained("./my_trained_qwen")  # 保存模型和配置文件
tokenizer.save_pretrained("my_trained_qwen")

print("训练完成并保存模型！")
```


## 加载训练后的模型，并测试
```
# 加载微调后的模型
model = AutoModelForCausalLM.from_pretrained("my_trained_qwen")
tokenizer = AutoTokenizer.from_pretrained("my_trained_qwen")

print("训练后模型加载成功！")

# 测试训练后的模型
inputs = tokenizer("请介绍一下人工智能？", return_tensors="pt").to(model.device)
outputs = model.generate(**inputs)
print(tokenizer.decode(outputs[0], skip_special_tokens=True))

```


❌ 训练可能没有足够效果。建议尝试以下方法：
  1. 增加训练轮数
  2. 提高学习率
  3. 使用更明确的训练数据
  4. 尝试全参数微调而非LoRA