# 安装nvm npm yarn
- [nvm安装](https://github.com/nvm-sh/nvm#installing-and-updating)
```shell
# 查看npm版本
nvm ls-remote
nvm install 14.14.0
npm version
# 全局安装yarn
npm install -g yarn
yarn --version
# 查看registry字段是否为国内源
yarn config list
yarn config set registry https://registry.npm.taobao.org
```

# python
```shell
python -V
# 安装虚拟环境支持多版本python
pip install virtualenv
# 查看已安装的第三方包
pip list
# 环境变量中的 Python3/Scripts 在前则使用 Python3/Scripts 作为虚拟环境的解释器
# 创建虚拟环境
virtualenv [虚拟环境名字]
# 进入虚拟环境后包的安装卸载都是在这个虚拟环境中，不会影响到外部
# win进入虚拟环境：进入环境变量中的 Scripts 文件夹中，执行 activate
# *nix进入虚拟环境：source /path/to/virtualenv/bin/activate
# 退出虚拟环境： deactivate
```

# 其他
- [protoc安装](https://github.com/protocolbuffers/protobuf/releases)
- [graphviz](https://www.graphviz.org/download/)