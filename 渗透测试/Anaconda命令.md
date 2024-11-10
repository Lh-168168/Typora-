conda create -n 环境名 python=3.6  创建新环境

activate 环境名 激活环境名

列出所有环境：conda env list，可以查看已创建的所有环境
激活环境：conda activate myenv，将当前环境切换到"myenv"环境
删除环境 conda remove -n py36 --all
安装包：conda install package_name，将包名替换为您要安装的实际包名
安装特定版本的包：conda install package_name=1.0，可以指定要安装的包的特定版本号
更新包：conda update package_name，更新已安装的包到最新版本
卸载包：conda remove package_name，卸载指定的包
查看已安装的包：conda list，列出当前环境中已安装的所有包

nvidia-smi 看显卡运行