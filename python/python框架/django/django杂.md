##  一、model常用类型

字段|类型|说明
---|---|---
AutoFleld| int(11)|自增主键， Django Model默认提供，可以被重写。它的完整定义是id = models.Autofield(primary_key=True)。
BooleanField |tinyint(1)|布尔类型字段，一般用于记录状态标记
Decimalfield| decimal|.开发对数据精度要求较高的业务时考虑使用，比如做支付相关、金融相关。定义时，需要指定精确到多少位，比如cash= models.Decimalfield(max_digits=8， decima1_places=2， defau1t=0， verbose_name="消费金额")就是定义长度为8位、精度为2位的数字。比方说，你想保存6666这样的数字，那么你的max_digits就需要为5， Decimalplaces需要为2。同时需要注意的是，在 Python中也要使用 Decimal类型来转换数据( from decimal import Decimal)。
IntegerField|int(11)|它同 AutoField一样，唯一的差别就是不自增。
PositiveIntegerField||同 IntergerField，只包含正整数
smallIntegerField|smallint|小整数时一般会用到。
charfield |varchar。|基础的 varchar类型。
URLField||继承自 Charfield，但是实现了对URL的特殊处理。
UUIDField |char(32)|除了在 PostgreSQL中使用的是uuid类型外，在其他数据库中均是固定长度char(32)，用来存放生成的唯一id 
EmailField||同 URLField一样，它继承自 CharField，多了对E-mail的特殊处理。
FieField||同 URLField一样，它继承自 CharField，多了对文件的特殊处理。当你定义一个字段为FileField时，在 admin部分展示时会自动生成一个可上传文件的按钮。
TextFileld||Longtext。一般用来存放大量文本内容，比如新闻正文、博客正文。
ImagField||继承自FileField，用来处理图片相关的数据，在展示上会有不同。
DateField||
DateTimeField||
TimeField||
ForeignKey||
OneToOneField||
ManyToManyField||

## 参数
参数名|解释
---|---
null|可以同blank对比考虑，其中null用于设定在数据库层面是否允许为空。
blank|针对业务层面，该值是否允许为空。
choices|前面介绍过，配置字段的 choices后，在 admin页面上就可以看到对应的可选项展示。
db_column|默认情况下，我们定义的Field就是对应数据库中的字段名称，通过这个参数可以指定 Model中的某个字段对应数据库中的哪个字段。
db_index|索引配置。对于业务上需要经常作为查询条件的字段，应该配置此项。
default|默认值配置
editale|是否可编辑，默认是True。如果不想将这个字段展示到页面上，可以配置为False。
error_message|用来自定义字段值校验失败时的异常提示,字典格式。key的可选项为null、 blank、 invalid、 invalid_choice、 unique和 unique_for_date。
help_text|字段提示语，配置这一项后，在页面对应字段的下方会展示此配置。
primary_key|主键，一个 Model只允许设置一个字段为 primary_key_nique。唯一约束，当需要配置唯一值时，设置 unique=rue，设置此项后，不需要设置db_index。
unique_for_date|针对date(日期)的联合约束，比如我们需要一天只能有一篇名为《学习 Django实战》的文章，那么可以在定义title字段时配置参数：unique_for_date="created time"