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

访问 https://zenodo.org/ ，搜索`aizynthfinder`,找到模型并下载，解压到当前工程的aiz_models目录下


## Usage
在本地执行
```shell
aizynthcli --config config_local.yml --smiles smiles_sample.txt
```
会

