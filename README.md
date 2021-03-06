#音乐识别系统
----------

一个音乐指纹识别系统，使用的语言为JAVA，同时需要用到MySQL数据库(虽然不是必须的，但这个系统采用他保存指纹和音乐信息)。他包含了指纹生成，数据库存储，和简易的服务器和客户端。

他通过生成和记录音乐指纹，能够识别来自麦克风、文件等各个来源的音乐，并且有很高的抗噪性，同时他对文件属性和音乐质量不敏感。你可以使用服务器给手机或者其他程序提供音乐识别服务。

你可以根据需求调节里面的参数，当前参数是为了在较短时间识别来自极大噪声和失真的音源，1500个左右的文件将产生接近24000000个指纹数据。如果你只用于识别文件并且没有严重的噪声与失真，你可以修改参数，1个文件只需要少量指纹就可以识别，对于噪声较低的音源10s 200个指纹已经满足大多需求。

##简易使用方法
依赖的库：com.springsource.org.json，JTransforms，mysql-connector-java

1. 需要安装MySQL，并执行Fingerprint. sql， 同时你可能需要修改max_allowed_packet参数，因为添加歌曲需要发送较大的包，我采用的参数是32M。
2. 修改MysqlDB中的数据库信息为你的数据库信息，如：

	    private final String url = "jdbc:mysql://127.0.0.1:3306/musiclibary?user=yecheng";
	    private final String user = "yecheng";
	    private final String password = "yecheng";


3. 添加文件的方法:
	1. 将文件转码为WAV，采样率为8000。
	2. 调用Insert，参数为文件名或者文件夹。
	
	*Ps:你可以重写添加的方法或者制作脚本或者直接使用其他软件实现转码功能，目前他能够从%title%}}%album%}}%artist%的文件名中获得信息。*

- 搜索音乐
	- 你可以调用Search+文件名搜索。
	- 在数据库较大的情况推荐采用运行Server，使用Client+文件名搜索。

##主要参数介绍

Fingerprint:

	NPeaks:一个周期中每个子带的峰值点的个数
	fftSize:FFT的窗口大小
	overlap:FFT的窗口重叠大小
	C:一个周期包含多少个窗口
	peakRange:取峰值点时与多大范围的邻居比较
	range_time:取点对的时候的时间范围，单位为秒
	range_freq:取点对的时候的频率范围，单位为频率
	Band:分成的子带，值对应FFT产生的数组索引
	minFreq:最小频率
	maxFreq:最大频率
	minPower:最小能量

修改的建议：

- 提高识别率：
	- 减小minPower, 增加Band、NPeaks、range_time

- 降低数据量：
	- 增大minPower,减小Band、NPeaks、rang_time

其中建议先修改Band和minPower。

----------

Server:

	port:服务器的端口

Client:

	ip:服务器的ip
	port:服务器的端口


##性能与效果
**数据量：**音乐库为1500首歌，指纹数量为24000000个左右，服务器稳定后占用内存约340M。

**速度：**处理器i7-3632QM,添加1500首歌用时约1919秒，一首歌约用时1.3秒。使用服务器查找10s的歌曲用时约0.2秒（不考虑客户端读取文件的时间）。

**准确度：**对噪声较低的音频有很高的识别率，对噪声较高的也有接近商用的准确率，但是相对来说如果对于未出现在曲库的歌曲，也有一定的误报率。

**抗噪性：**能够抵抗较强的失真和噪声，可以参考我给的测试音频。

##工作原理
参考文档：

- [Shazam](http://www.ee.columbia.edu/~dpwe/papers/Wang03-shazam.pdf "shazam")
- [Mel scale](https://en.wikipedia.org/wiki/Mel_scale "Mel_scale")

本算法实现类似Shazam，首先我计算出音频的频谱图，将频谱根据频率分成若干子带，对每个子带查找若干个峰值点，本算法子带划分基于Mel频率。

将获得的峰值点根据频率、时间范围组成点对。

本算法的取点对频率范围为在子带内，其目的在于减少点对的数目并且提高分布式能力。取点对的时间范围为1s-4s。你可以根据需要修改这些参数。

##联系方式
EMAIL:hsyecheng@hotmail.com
