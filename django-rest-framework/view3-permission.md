### 权限问题

解决了Novel类数据的查询，筛选，检索，过滤，以及重要的用户认证。那么接下来需要考虑的就是权限问题。

问题:

1. 未登录用户只能看小说的前十章。
2. 登录用户可以收藏自己喜欢的小说。
3. 登录用户只能查看自己收藏的小说，并对它进行增加，删除。

接下来我们一一解决这些问题。

#### Novel收藏

##### View

```py
 class UserFavViewset(viewsets.GenericViewSet, mixins.ListModelMixin, mixins.CreateModelMixin, mixins.RetrieveModelMixin,
                     mixins.DestroyModelMixin):
    """
    list:
        获取用户收藏列表
    retrieve:
        判断小说是否已经收藏
    create:
        收藏小说
    """
    permission_classes = (IsAuthenticated, IsOwnerOrReadOnly)
    serializer_class = UserFavSerializer
    lookup_field = 'novel_id'

    authentication_classes = (JSONWebTokenAuthentication, SessionAuthentication)

    def get_queryset(self):
        return UserFav.objects.filter(user=self.request.user)

    # 设置动态的Serializer
    def get_serializer_class(self):
        if self.action == "list":
            return UserFavDetailSerializer
        elif self.action == "create":
            return UserFavSerializer

        return UserFavSerializer
```

在这个view中，我们实现了对用户收藏表数据的增删改查。

复写get\_serializer\_class方法设置动态Serializer。

增加新收藏时，通过UserFavSerializer方法，判断用户重复收藏，并实现查询的唯一联合。

##### permissions.py

```py
# encoding: utf-8

from rest_framework import permissions

"""
在utils中新建permissions，这是我们自定义的permissions，然后粘贴上面的IsOwnerOrReadOnly
这个自定义的permission类继承了我们的BasePermission。它有一个方法叫做has_object_permission，是否有对象权限。
会检测我们从数据库中拿出来的obj的owner是否等于request.user
这个obj是我们数据库中的表，所以这里的owner应该改为我们数据库中的外键user
安全的方法也就是不会对数据库进行变动的方法，总是可以让大家都有权限访问到。
"""


class IsOwnerOrReadOnly(permissions.BasePermission):
    """
  只允许作者修改但允许所有人读的权限设置
    """

    def has_object_permission(self, request, view, obj):
        # 所有用户都允许读取,所以安全的http方法会直接放行
        # SAFE_METHODS = ('GET', 'HEAD', 'OPTIONS')
        if request.method in permissions.SAFE_METHODS:
            return True

        # 写入权限需要作者本人
        return obj.user == request.user
```

这个权限控制方法作用是判断用户是否为收藏的拥有者，杜绝非拥有者的操作。

注册url

```py
router = DefaultRouter()
# 配置用户收藏的url
router.register(r'userfavs', UserFavViewset, base_name="userfavs")
```

通过url [http://127.0.0.1:8000/userfavs/    可以对UserFav进行增删查](http://127.0.0.1:8000/userfavs/可以对UserFav进行增删查)

请求headers:Authorization:Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjo0LCJ1c2VybmFtZSI6InJvb3QiLCJleHAiOjE1MjY1ODQ0NjcsImVtYWlsIjoicm9vdCJ9.TPzrgYJPGUJk5TIs2H1gnPyIVM\_62oDKhN\_dUqe279Y

* 增：POST  [http://127.0.0.1:8000/userfavs/](http://127.0.0.1:8000/userfavs/) body:{"novel":"1"}

* 删：DELETE  [http://127.0.0.1:8000/userfavs/id/](http://127.0.0.1:8000/userfavs/id/)

* 查：GET  [http://127.0.0.1:8000/userfavs/](http://127.0.0.1:8000/userfavs/) or [http://127.0.0.1:8000/userfavs/id/](http://127.0.0.1:8000/userfavs/id/)



