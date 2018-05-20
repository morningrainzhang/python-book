用户验证规则



```py
class CustomBackend(ModelBackend):
     """
    自定义 CustomBackend 类, 用来实现邮箱登录, 继承自 django 的 ModelBackend,
    ModelBackend 类有一个 authenticate 方法, 会被 django 自动调用
    """

    def authenticate(self, request, username=None, password=None, **kwargs):
        # 这里覆写 authenticate 方法, 在此完成自己的后台逻辑, 实现可以通过用户名或邮箱登录网站
        try:
            # 首先根据用户名和密码来查询一下这个用户是否存在于数据库
            # 因为用户名在数据库中是不能重名的, 所以可以用 .get() 方法来查询
            # django 中, 如果直接写 .get(username=username, email=username),
            # 这个查询结果是"并集", 如果想使用"或", 即"username=username 或 email=username",
            # 则可以利用 django 的 Q 对象, 它允许使用|（OR）和&（AND）操作构建复杂的数据库查询
            user = User.objects.get(Q(username=username) | Q(email=username))
            # 这里查询密码的方式是不能和上面查询用户名一样的,
            # 因为 django 在将密码存储到数据库中是加密的,
            # 所以不能简单的使用 objects.get(password=password) 来查询,
            # 前台 request 传进来的明文 password 是不可能和数据库中加密的 password 匹配的, 所以无法查询,
            # 不过 因为 UserProfile 继承自 django 的 AbstractUser,
            # 而 AbstractUser 有一个 check_password 方法, 可以将传进去的明文 password 进行加密处理,
            # 然后和 user 对象的 password 字段做对比, 验证密码是否和这个用户的密码
            if user.check_password(password):
                # 如果判断成功, 即用户名和密码相匹配, 返回 user 对象
                return user
        except Exception as err:
            # 遇到异常, 即用户名或密码不匹配, 返回 None
            return None

```



密码保存

signals.py

https://segmentfault.com/a/1190000008455657

```
# encoding: utf-8

from django.conf import settings
from django.db.models.signals import post_save
from django.dispatch import receiver
from rest_framework.authtoken.models import Token

from django.contrib.auth import get_user_model
User = get_user_model()


# 参数一接收哪种信号，参数二是接收哪个model的信号
@receiver(post_save, sender=User)
def create_auth_token(sender, instance=None, created=False, **kwargs):
    # 是否新建，因为update的时候也会进行post_save
    if created:
        password = instance.password
        instance.set_password(password)
        instance.save()
```

```
default_app_config = 'novel.apps.NovelConfig'
```

app.py

```
    def ready(self):
        import novel.signals
```



