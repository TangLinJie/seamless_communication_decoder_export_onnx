# Seamless(S2T) Decoder ONNX模型导出
[Seamless官方仓库](https://github.com/facebookresearch/seamless_communication)未提供onnx导出脚本，本导出教程基于官方代码进行修改，导出的decoder onnx模型精度与官方pt模型一致。

## 1. 特性
* 支持流式SeamlessStreaming(S2T任务)的onnx模型导出
* 支持离线M4t(S2T任务)的onnx模型导出

## 2. 代码修改说明
主要修改内容包括：
* 增加onnx导出代码
* 移除forward自定义输入参数类型，替换为PyTorch导出onnx支持的输入类型
* 变更KV cache作为输入输出
* 为了确保固定序列长度输入的有效性，增加mask作为输入
* 平替一些算子，例如将隐式广播替换为了具体PyTorch函数
* 重设mask的填充值
* 代码不会进行推理，导出完成即退出

## 3. SeamlessStreaming(S2T任务) Decoder ONNX模型导出
SeamlessStreaming(S2T任务)decoder的onnx模型导出需要在x86主机上完成。

- step1：安装和配置依赖环境

为了导出Decoder模块，需要进行如下的环境准备
```bash
# Decoder export
cd seamless_communication_decoder_export_onnx-main
pip install . -i https://pypi.tuna.tsinghua.edu.cn/simple
cd ../fairseq2_decoder_export_onnx-main/
pip install . -i https://pypi.tuna.tsinghua.edu.cn/simple
pip install torchaudio==2.1.1 -i https://pypi.tuna.tsinghua.edu.cn/simple
# 若使用conda需要安装
conda install -c conda-forge libsndfile==1.0.31
```

- step2：下载pt模型
在导出时，代码会自动下载pt模型，值得注意的是对于huggingface上的模型下载很可能出现网络问题，可下载好以后，修改`src/seamless_communication/cards/seamless_streaming_monotonic_decoder.yaml`和`src/seamless_communication/cards/seamless_streaming_unity.yaml`中的模型路径。

- step3：导出onnx模型
执行如下命令导出Decoder的所有子模块(包括decoder frontend、decoder、final proj模块)
```bash
cd ..
mkdir seamless_streaming_monotonic_decoder_text_decoder_frontend seamless_streaming_monotonic_decoder_text_decoder_step_bigger_1 seamless_streaming_monotonic_decoder_text_decoder_step_equal_1 seamless_streaming_monotonic_decoder_final_proj
streaming_evaluate --task s2tt --data-file ./cvssc_ja/test.tsv --audio-root-dir ./cvssc_ja/test --output ./test --tgt-lang eng --dtype fp32 --device cpu
```

## 4. M4t(S2T任务) Decoder ONNX模型导出
M4t(S2T任务)decoder的onnx模型导出需要在x86主机上完成。

- step1：安装和配置依赖环境

为了导出Decoder模块，需要进行如下的环境准备
```bash
# Decoder export
cd seamless_communication_decoder_export_onnx-main
pip install . -i https://pypi.tuna.tsinghua.edu.cn/simple
cd ../fairseq2_decoder_export_onnx-main/
pip install . -i https://pypi.tuna.tsinghua.edu.cn/simple
pip install torchaudio==2.1.1 -i https://pypi.tuna.tsinghua.edu.cn/simple
pip install onnx -i https://pypi.tuna.tsinghua.edu.cn/simple
# 若使用conda需要安装
conda install -c conda-forge libsndfile==1.0.31
```

- step2：下载pt模型
在导出时，代码会自动下载pt模型，值得注意的是对于huggingface上的模型下载很可能出现网络问题，可下载好以后，修改`src/seamless_communication/cards/seamlessM4T_v2_large.yaml`和`src/seamless_communication/cards/vocoder_v2.yaml`中的模型路径。

- step3：导出onnx模型
执行如下命令导出Decoder的所有子模块(包括decoder frontend、decoder、final proj模块)
```bash
cd ..
# 创建结果文件夹
mkdir m4t_decoder_frontend_beam_size_s2t m4t_decoder_beam_size_s2t m4t_decoder_final_proj_beam_size_s2t 
python export_m4t_offline.py
```