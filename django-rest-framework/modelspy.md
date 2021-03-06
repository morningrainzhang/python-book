### Novel and Section

首先根据需求创建模型。

首先，我们可以让用户阅读小说以及小说章节，这里先创建Novel\(小说\),Section\(章节\)，另它们成为一对多的关系。

```py
class Novel(models.Model):
    """
    小说模型
    """
    title = models.CharField(max_length=100, verbose_name='小说标题', help_text="小说标题")
    state = models.CharField(max_length=20, null=True, verbose_name='小说状态', help_text="小说状态")
    author = models.CharField(max_length=50, verbose_name='小说作者', help_text="小说作者")
    type = models.CharField(max_length=20, default='未分类', verbose_name='小说类型', help_text="小说类型")
    count_word = models.IntegerField(null=True, verbose_name='小说字数', help_text="小说字数")
    abstract = models.CharField(max_length=500, verbose_name='小说简介', help_text="小说简介")
    novel_image = models.URLField(verbose_name='小说封面', help_text="小说封面")
    add_time = models.DateTimeField(auto_now_add=True, verbose_name='添加时间', help_text="添加时间")
    novel_url = models.URLField(null=True, verbose_name='小说网址', help_text="小说网址")

    class Meta:
        verbose_name = '小说'
        verbose_name_plural = verbose_name

    def __str__(self):
        return self.title


class Section(models.Model):
    """
    小说章节模型
    """
    title = models.CharField(max_length=50, verbose_name='章节标题')
    content = models.TextField(verbose_name='章节内容', null=True)
    add_time = models.DateTimeField(auto_now_add=True, verbose_name='更新时间')
    num = models.IntegerField(null=True, verbose_name='章节顺序')
    section_url = models.URLField(null=True, verbose_name='章节网址')

    novel = models.ForeignKey(Novel, on_delete=models.CASCADE, verbose_name='小说', related_name='novel')

    class Meta:
        verbose_name = '章节'
        verbose_name_plural = verbose_name
        ordering = ['id']

    def __str__(self):
        return self.title
```

小说具有小说名称，作者，封面图片等属性。

章节具有章节标题，以及章节顺序等属性，以及包含Novel外键。



### User

既然我们要实现用户收藏功能，需要进行身份验证，所以我们创建User表。其实在django中已经有自带的auth.user，例如我们createsuperuser创建的id=1的管理员。我们只需要添加我们需要的属性，和自带的User表融为一体\(融合成一张表UserProfile

 \)。

```py
class UserProfile(AbstractUser):
    """
    用户表，新增字段如下
    """
    GENDER_CHOICES = (
        ("male", u"男"),
        ("female", u"女")
    )
    # 用户注册时我们要新建user_profile 但是我们只有邮箱号
    name = models.CharField(max_length=30, null=True, blank=True, verbose_name="姓名")
    # 保存出生日期，年龄通过出生日期推算
    birthday = models.DateField(null=True, blank=True, verbose_name="出生年月")
    gender = models.CharField(max_length=6, choices=GENDER_CHOICES, default="female", verbose_name="性别")
    mobile = models.CharField(null=True, blank=True, max_length=11, verbose_name="电话")
    # mobile = models.CharField(max_length=11, verbose_name="电话")
    email = models.EmailField(max_length=100, null=True, blank=True, verbose_name="邮箱")
    image = models.ImageField(upload_to='image/user/', verbose_name='用户头像')

    class Meta:
        verbose_name = "用户"
        verbose_name_plural = verbose_name

    def __str__(self):
        return self.username
```

另外注册新用户时我们需要选用一种验证方式，例如手机号码验证或者邮箱认证。处于经济方面的考虑，选择使用免费的邮箱发送验证码，来提供验证信息。所以创建一个EmailVerifyRecord表。

```py
class EmailVerifyRecord(models.Model):
    SEND_CHOICES = (
        ("register", u"注册"),
        ("forget", u"找回密码")
    )
    code = models.CharField(max_length=50, verbose_name='验证码')
    email = models.EmailField(verbose_name='邮箱')
    send_type = models.CharField(choices=SEND_CHOICES, max_length=20, verbose_name='发送类型')
    add_time = models.DateTimeField(default=datetime.now, verbose_name="添加时间")

    class Meta:
        verbose_name = "邮箱验证"
        verbose_name_plural = verbose_name

    def __str__(self):
        return self.code
```

在注册一级找回密码功能时我们都会使用验证码。 

### UserFav

用户收藏功能

```py
 class UserFav(models.Model):
    """
    用户收藏操作
    """
    user = models.ForeignKey(User, on_delete=models.CASCADE, verbose_name="用户")
    novel = models.ForeignKey(Novel, on_delete=models.CASCADE, verbose_name="小说", help_text="小说id")
    add_time = models.DateTimeField(auto_now_add=True, verbose_name=u"添加时间")

    class Meta:
        verbose_name = '用户收藏'
        verbose_name_plural = verbose_name

        # 多个字段作为一个联合唯一索引
        unique_together = ("user", "novel")

    def __str__(self):
        return self.user.username
```

在这里我们有user与novel两个重要属性，但是我们不希望在这张表中出现相同user以及相同novel的一条数据，所以我们创建一个联合唯一索引。最重要的是，在查询数据时，我们将搜索novel时屏蔽，只能使用user查询。  




