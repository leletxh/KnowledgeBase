ollama官网死活访问不上去

只能用MS或者镜像

```
sudo apt update && sudo apt install -y pciutils lshw zstd
pip install modelscope -U
modelscope download --model=modelscope/ollama-linux --local_dir ./ollama-linux --revision v0.14.1
cd ollama-linux
chmod +x ollama-modelscope-install.sh
./ollama-modelscope-install.sh
```
