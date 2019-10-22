## HTTP数据类型与编码
- HTTP通过MIME（多用途互联网邮件扩展，multipurpose Internet mail extensions）来告知数据格式
- MIME把数据分为8大类，每个大类下又有多个子类，形式是“type/subtype”。MIME类型举例：
1. text：文本格式的可读数据。最常见的是text/html超文本文档。还有纯文本text/plain、样式表text/css等
2. image：图像文件。有image/gif ， /image/jpeg，  image/png等
3. audio/video：音频和视频数据，例如audio/mpeg、video/mp4等
4. application：数据格式不固定，可能是文本也可能是二进制，由上层应用程序解释。常见的有application/json,application/javascript,application/pdf等，以及不透明的二进制数据application/octet-stream
- 有时需要压缩数据，即“encoding type”，有三种：
1. gzip：GNU zip压缩格式，互联网最流行的压缩格式
2. deflate：zlib（deflate）压缩格式
3. br：专门为HTTP优化的新压缩算法（Brotli）

## 数据类型使用的头字段
