# python爬取QQ空间说说并生成词云
> 原理是利用python来模拟登陆QQ空间，对一个QQ的空间说说内容进行爬取，把爬取的内容保存在txt文件中，然后根据txt文件生成词云。

以下是生成的词云图
![](https://ws3.sinaimg.cn/large/006tNbRwgy1fpdv0sysv5j318g0xcb29.jpg)

我的环境：`Mac`,`Anaconda`,`Python2.7`,以及各种用到的`Python`库

### 先来说下Anaconda
Anaconda 是一个可用于科学计算的 Python 发行版，支持 Linux、Mac、Windows系统，内置了常用的科学计算包。它解决了官方 Python 的两大痛点。

* 第一：提供了包管理功能，Windows 平台安装第三方包经常失败的场景得以解决，
* 第二：提供环境管理的功能，功能类似 Virtualenv，解决了多版本Python并存、切换的问题。

conda 是 Anaconda 下用于包管理和环境管理的工具，功能上类似 pip 和 vitualenv 的组合。安装成功后 conda 会默认加入到环境变量中，因此可直接在命令行窗口运行命令 conda

conda 的环境管理与 virtualenv 是基本上是类似的操作。

```
# 查看帮助
conda -h 
# 基于python3.6版本创建一个名字为python36的环境
conda create --name python36 python=3.6 
# 激活此环境
source activate python36 # linux/mac
# 再来检查python版本，显示是 3.6
python -V  
# 退出当前环境
source deactivate python36 
# 删除该环境
conda remove -n python36 --all
# 查看所以安装的环境
conda info -e
```

conda 的包管理功能可 pip 是一样的，当然你选择 pip 来安装包也是没问题的。

```
# 安装 matplotlib 
conda install matplotlib
# 查看已安装的包
conda list 
# 包更新
conda update matplotlib
# 删除包
conda remove matplotlib
```
在 conda 中 anything is a package。conda 本身可以看作是一个包，python 环境可以看作是一个包，anaconda 也可以看作是一个包，因此除了普通的第三方包支持更新之外，这3个包也支持。比如：
Anaconda 的镜像地址默认在国外，用 conda 安装包的时候会很慢，目前可用的国内镜像源地址有清华大学的。修改 ~/.condarc (Linux/Mac) 

```
channels:
 - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
 - defaults
show_channel_urls: true
```

如果使用conda安装包的时候还是很慢，那么可以考虑使用pip来安装，同样把 pip 的镜像源地址也改成国内的，豆瓣源速度比较快。修改 ~/.pip/pip.conf (Linux/Mac)

```
[global]
trusted-host =  pypi.douban.com
index-url = http://pypi.douban.com/simple
```

环境搭建好之后就可以开始愉快地玩数据分析了。

### 爬取动态内容


1. 因为动态页面的内容是动态加载出来的，所以我们需要不断下滑，加载页面
2. 切换到当前内容的frame中,也有可能不是frame,这里需要查看具体情况
3. 获取页面源数据，然后放入xpath中，然后读取

```python
  # 下拉滚动条，使浏览器加载出动态加载的内容，
  # 我这里是从1开始到6结束 分5 次加载完每页数据
        for i in range(1,6):
            height = 20000*i#每次滑动20000像素
            strWord = "window.scrollBy(0,"+str(height)+")"
            driver.execute_script(strWord)
            time.sleep(4)

        # 很多时候网页由多个<frame>或<iframe>组成，webdriver默认定位的是最外层的frame，
        # 所以这里需要选中一下说说所在的frame，否则找不到下面需要的网页元素
        driver.switch_to.frame("app_canvas_frame")
        selector = etree.HTML(driver.page_source)
        divs = selector.xpath('//*[@id="msgList"]/li/div[3]')
```

### 生成词云
生成词云需要用到的库：
1. `wordcloud`, 生成词云
2. `matplotlib`, 生成词云图片
3. `jieba`,显示中文。

```python
#coding:utf-8

from wordcloud import WordCloud
import matplotlib.pyplot as plt
import jieba

#生成词云
def create_word_cloud(filename):
    text= open("{}.txt".format(filename)).read()
    # 结巴分词
    wordlist = jieba.cut(text, cut_all=True)
    wl = " ".join(wordlist)

    # 设置词云
    wc = WordCloud(
        # 设置背景颜色
       background_color="white",
         # 设置最大显示的词云数
       max_words=2000,
         # 这种字体都在电脑字体中，一般路径
       font_path='/System/Library/Fonts/PingFang.ttc',
       height= 1200,
       width= 1600,
        # 设置字体最大值
       max_font_size=100,
     # 设置有多少种随机生成状态，即有多少种配色方案
       random_state=30,
    )

    myword = wc.generate(wl)  # 生成词云
    # 展示词云图
    plt.imshow(myword)
    plt.axis("off")
    plt.show()
    wc.to_file('py_book.png')  # 把词云保存下

if __name__ == '__main__':
    create_word_cloud('qq_word')
```


所有完整代码已放github

github地址https://github.com/Jimmy9876/QZone_spider


参考：
https://foofish.net/anaconda-install.html


