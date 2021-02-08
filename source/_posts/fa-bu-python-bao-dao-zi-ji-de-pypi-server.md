---
title: '发布Python包到自己的pypi server'
date: 2019-02-13 02:25:18
tags: [python,pypi]
published: true
hideInList: false
feature: 
---
第一步：写一个自己的Python包，`setup.py`:
<!-- more -->


```python
inst_data_dirs = ("sample_files", )

data_files = []
for inst_dir in inst_data_dirs:
    dir_full_path = os.path.join(os.path.dirname(__file__), inst_dir)
    for dirpath, dirnames, filenames in os.walk(dir_full_path):
        list_files = [os.path.join(dirpath, f) for f in filenames]
        data_files.append([dirpath, list_files])

install_proj_path = "/usr/local/share/mypackage"
for df in data_files:
    df[0] = os.path.join(install_proj_path, df[0])


setuptools.setup(
    name = "mypackage",
    version = "1.0.0",
    description = "<description>",
    author = "<Author>",
    author_email = "<Email>",
    url = "<URL>",
    data_files = data_files,
    packages = ["mypackage"],
    scripts = ["bin/mypackage-cmd"],
    zip_safe = False
)
```

第二步：Install到本地做测试：`python setup.py install`

第三步：打包并上传到自己的pypi server:
1. `sudo python setup.py sdist`
2. 上传到服务器 `sftp myserver`, and `put dist/mypackage.tar.gz`
3. 把这个tar.gz文件已到pypi的根目录下，例如：`/home/project/pypi`

#### 关于setup.py

**Case 1: with PostInstall:**

```
def rename_script():
    if platform.system().lower() == "windows":
        return

    paths = os.environ["PATH"].split(":")
    for path in paths:
        src_path = os.path.join(path, "ospider.py")
        if os.path.exists(src_path):
            dest_path = os.path.join(path, "ospider")
            os.rename(src_path, dest_path)
            break


class PostDevelop(develop):
    def run(self):
        develop.run(self)
        rename_script()


class PostInstall(install):
    def run(self):
        install.run(self)
        rename_script()
 
 
setup(
   ...
   cmdclass = {
        "develop": PostDevelop,
        "install": PostInstall
    },
    ...
)
```


**Case 2: include all data files inside one folder:**

In file `MANIFEST.in`: `recursive-include package_folder *` and in `setup.py` 's `setup()` function, add parameter `include_package_data = True`

#### 问题

这里有个问题待解决，当用自己PostInstall时，放到pypi server上，然后用pip install去安装这个包，发现PostInstall的脚本根本没执行，在上面的例子里，也就是`rename_script()`这个函数没有执行。[Stackoverflow上有人问这个问题](https://stackoverflow.com/questions/44549600/custom-post-install-script-not-running-with-pip)，但遗憾的是并没有一个好的解决方法，这个问题里他最后的解决方法是直接从repo里安装，不从pypi server上。所以我还挺奇怪"把包放pypi server上，然后pip不跑post install脚本"这种应该是很常见的问题，居然没有搜不到其他人问。

#### Reference:
[如何建立自己的pypi server](https://linode.com/docs/applications/project-management/how-to-create-a-private-python-package-repository/)