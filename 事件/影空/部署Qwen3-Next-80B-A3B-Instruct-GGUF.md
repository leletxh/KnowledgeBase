---
tags:
  - 日记
---
## ollama部分

ollama官网死活访问不上去

只能用MS或者镜像

```bash
sudo apt update && sudo apt install -y pciutils lshw zstd
pip install modelscope -U
modelscope download --model=modelscope/ollama-linux --local_dir ./ollama-linux --revision v0.14.1
cd ollama-linux
chmod +x ollama-modelscope-install.sh
./ollama-modelscope-install.sh
```

## 模型下载部分

用的还是MS，主要是量化了，都96GB显存了还跑不了全量

```bash
modelscope download --model Qwen/Qwen3-Next-80B-A3B-Instruct-GGUF Qwen3-Next-80B-A3B-Instruct-Q8_0.gguf --local_dir /root/autodl-tmp/model/
```
## 启动部分

懒得看doc了，让qwen写了一段

```bash
#!/bin/bash

# ==============================
# Qwen3-80B-Q8_0 一键启动脚本
# 适配 AutoDL 容器 + RTX PRO 6000 96GB
# 避免系统盘写满，模型存于数据盘
# ==============================

set -e  # 遇错退出

# --- 配置区 ---
MODEL_NAME="qwen3-80b-next"
GGUF_PATH="/root/autodl-tmp/model/Qwen3-Next-80B-A3B-Instruct-Q8_0.gguf"
DATA_DIR="/root/autodl-tmp"
OLLAMA_MODELS_DIR="${DATA_DIR}/ollama-models"
MODFILE_DIR="${DATA_DIR}/model"

# --- 环境变量 ---
export OLLAMA_MODELS="${OLLAMA_MODELS_DIR}"
export PATH="$PATH:/usr/local/bin"

# --- 函数：启动 ollama 服务 ---
start_ollama() {
    echo "[*] 检查 Ollama 服务..."
    if pgrep -x "ollama" > /dev/null; then
        echo "[+] Ollama 已在运行"
    else
        echo "[*] 启动 Ollama 服务..."
        mkdir -p "${OLLAMA_MODELS_DIR}"
        nohup ollama serve > /root/ollama.log 2>&1 &
        sleep 5
        if ! pgrep -x "ollama" > /dev/null; then
            echo "[-] Ollama 启动失败，请检查日志: tail -f /root/ollama.log"
            exit 1
        fi
        echo "[+] Ollama 服务已启动"
    fi
}

# --- 函数：创建 Modelfile ---
create_modelfile() {
    echo "[*] 创建 Modelfile..."
    cat > "${MODFILE_DIR}/Modelfile" <<EOF
FROM ${GGUF_PATH}

TEMPLATE "{{ if .System }}<|im_start|>system
{{ .System }}<|im_end|>
{{ end }}{{ if .Prompt }}<|im_start|>user
{{ .Prompt }}<|im_end|>
<|im_start|>assistant
{{ end }}"

PARAMETER num_ctx 4096
PARAMETER stop "<|im_end|>"
EOF
    echo "[+] Modelfile 已生成"
}

# --- 函数：创建模型（如果不存在）---
create_model_if_needed() {
    echo "[*] 检查模型是否已存在..."
    if ollama list | grep -q "^${MODEL_NAME}"; then
        echo "[+] 模型 ${MODEL_NAME} 已存在，跳过创建"
    else
        echo "[*] 创建模型 ${MODEL_NAME} ..."
        create_modelfile
        ollama create "${MODEL_NAME}" -f "${MODFILE_DIR}/Modelfile"
        echo "[+] 模型创建成功！"
    fi
}

# --- 主流程 ---
echo "🚀 启动 Qwen3-80B-Q8_0 (PRO 6000 96GB) ..."

# 检查模型文件是否存在
if [ ! -f "${GGUF_PATH}" ]; then
    echo "[-] 错误：模型文件不存在: ${GGUF_PATH}"
    exit 1
fi

# 创建目录
mkdir -p "${OLLAMA_MODELS_DIR}"

# 启动服务
start_ollama

# 创建模型
create_model_if_needed

# 运行模型
echo "💬 进入对话模式（输入 'exit' 或 Ctrl+D 退出）..."
echo "--------------------------------------------------"
ollama run "${MODEL_NAME}"
```


---
话说，影空和123吵架，我要不要安抚安抚
算了，不管了
希望影空快点发我那个地址
终于发了
影空居然夸我了

想搞个webui
难弄，不弄了