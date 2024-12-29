# Deploy-GLM-on-Android
借助开源项目[executorch](https://github.com/pytorch/executorch)，在Android上部署GLM-Edge-1.5B-Chat模型.


## 效果展示
https://github.com/user-attachments/assets/cce57b27-029d-49a1-9f68-66e36c306320

## 部署思路
executorch能够在Android上部署LLAMA模型，参考下列链接一步一步执行即可。两个链接对运行环境的要求不同，它们都可以成功在Android上部署LLAMA模型
* https://github.com/pytorch/executorch/blob/v0.4.0/examples/demo-apps/android/LlamaDemo/docs/delegates/xnnpack_README.md
* https://github.com/pytorch/executorch/blob/v0.4.0/examples/demo-apps/android/LlamaDemo/docs/delegates/qualcomm_README.md

由于GLM-Edge-1.5B-Chat的模型结构与LLAMA的模型结构相似，我们可以复用executorch部署LLAMA模型的代码，部署GLM-Edge-1.5B-Chat模型。
只需要解决下列挑战：
1. 将GLM-Edge-1.5B-Chat模型权重转换为LLAMA模型权重

2. 将GLM-Edge-1.5B-Chat的tokenizer.json转换成tokenizer.model

    GLM-Edge-1.5B-Chat的使用huggingface transformers的tokenizer，其词表存储在`tokenizer.json`中。而executorch使用基于`tiktoken`的tokenizer，其词表存储在`tokenizer.model`中

3. 修改executorch的代码，增加GLM-Edge-1.5B-Chat的ChatTemplate

    * cpp代码<https://github.com/pytorch/executorch/blob/6a085fff7f78cb51443d97a827503acc6ae28e3c/examples/models/llama2/tokenizer/llama_tiktoken.cpp#L21> 中硬编码了special tokens
    * java代码<https://github.com/pytorch/executorch/blob/6a085fff7f78cb51443d97a827503acc6ae28e3c/examples/demo-apps/android/LlamaDemo/app/src/main/java/com/example/executorchllamademo/PromptFormat.java#L18-L63> 中硬编码了chat template


## 部署方法1（二选一）：全流程的编译、转换、部署
有空再更新


## 部署方法2（二选一）：下载转换好的模型和预编译好的aar
1. 下载转换好的模型
    * xnnpack: <https://huggingface.co/wandz/glm-edge-1.5B-xnnpack>
    * qnn: to-be-continued
    

2. 将模型传到手机上
    ```
    adb shell mkdir -p /data/local/tmp/llama
    adb push ./glm-edge-1.5B-xnnpack/glm_edge_1.5B_xnnpack.pte /data/local/tmp/llama
    adb push ./glm-edge-1.5B-xnnpack/glm_edge_tokenizer.model /data/local/tmp/llama
    ```

    注意:
    * 手机要开启开发者模式
    * 需要安装adb，参考<https://developer.android.com/tools/adb?hl=zh-cn>


3. 将`./prebuilt_libs/xnnpack/executorch-llama.aar`复制到`./LlamaDemo/app/libs`目录下

4. 编译java项目

    用Android Studio打开`./LlamaDemo`目录，运行app(^R) 。 Android Studio将完成编译，并在手机上安装app（可以操作手机，同意安装）

    参考<https://github.com/pytorch/executorch/blob/v0.4.0/examples/demo-apps/android/LlamaDemo/docs/delegates/xnnpack_README.md#run-the-android-demo-app>


## 项目文件说明
- LlamaDemo: 基于<https://github.com/pytorch/executorch/tree/v0.4.0/examples/demo-apps/android/LlamaDemo>做了修改, 增加了GLM模型的ChatTemplate
- prebuilt_libs: 我预先编译好的`executorch-llama.aar`


