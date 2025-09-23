# User Guide

## Install

1. nstall uv
```shell
curl -LsSf https://astral.sh/uv/install.sh | sh
```

2. install packages
```shell
uv lock
uv pip install -e ".[all]" 
```

3. install model

访问 https://zenodo.org/ ，搜索`aizynthfinder`,找到模型并下载，解压到当前工程的`aiz_models`目录下


## Usage

```shell
cp config_sample.yml config_local.yml
```
并将config_local中的路径修改成当前目录的绝对路径

### 命令提示符下运行
在本地执行
```shell
aizynthcli --config config_local.yml --smiles smiles_sample.txt
```
执行完成后生成`output.json.gz`，用gunzip进行解压

```shell
gunzip output.json.gz
```

### Jupyter下运行
config_local.yml中的路径修改成绝对路径后，然后执行如下命令

```shell
    aizynthapp --config config_local.yml
```
