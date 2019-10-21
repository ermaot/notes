连接池需要实现的内容：
1. 连接对象（包括几个元素：驱动，URL，用户名，密码，数据库名，端口，socket，连接大小的参数，连接flag等）
2. 连接对象的队列（连接对象类，允许的连接数量，当前连接的数量，连接的状态）
3. 连接的入列出列处理
4. 连接队列按需增大和减小
5. 连接的关闭


样例1（CPP版本）：
https://github.com/primer9999/MysqlPool

样例2（java版本）：
https://github.com/Edenwds/javaweb/tree/master/ConnPool