1. InnoDB申请的AIO队列的长度只有256，由常量OS_AIO_N_PENDING_IOS_PER_THREAD（os0file.h）定义。mysql8中为32

```
/** Win NT does not allow more than 64 */
static const ulint OS_AIO_N_PENDING_IOS_PER_THREAD = 32;
```

可以改成innodb_aio_pending_ios_per_thread=1024

本段来自：http://www.penglixun.com/tech/database/case_about_innodb_faster_than_oracle.html

