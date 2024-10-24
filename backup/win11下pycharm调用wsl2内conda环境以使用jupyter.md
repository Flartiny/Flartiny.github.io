```
conda install jupyter
jupyter notebook --generate-config
vim ~/.jupyter/jupyter_notebook_config.py
```
需调整内容如下，可查看相应部分解释，若默认已符合以下状态则不用修改，否则取消注释并完成更改
> c.ServerApp.open_browser = False
> c.ServerApp.allow_origin = '*'
> c.ServerApp.ip = '0.0.0.0'
> c.ServerApp.allow_root = True

第四条某种程度上是不推荐的做法，但是暂时没有找到好的解决方法

---
## 2024/10/24更新
重装了wsl2，这次网络模式使用了Mirrored，可在wsl2内运行```jupyter lab --no-browser```获取相关信息，配置在pycharm-setting-jupyter servers-configured server中便可启动