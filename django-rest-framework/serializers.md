### Novel and Section

```py
class NovelSerializer(serializers.ModelSerializer):
    class Meta:
        model = Novel
        fields = '__all__'
        # read_only_fields = []#不让前端修改


class SectionSerializer(serializers.ModelSerializer):
    # novel = NovelSerializer(many=True, read_only=True)
    novel = NovelSerializer(read_only=True)  # 外键显示信息

    class Meta:
        model = Section
        fields = '__all__'
```

### UserFav

```py
class UserFavDetailSerializer(serializers.ModelSerializer):
    # 通过novel_id拿到商品信息。就需要嵌套的Serializer
    novel = NovelSerializer()

    class Meta:
        model = UserFav
        fields = ("novel", "id")


class UserFavSerializer(serializers.ModelSerializer):
    # 表示当前用户的默认类
    user = serializers.HiddenField(
        default=serializers.CurrentUserDefault()
    )

    class Meta:
        model = UserFav

        # 使用validate方式实现唯一联合
        validators = [
            UniqueTogetherValidator(
                queryset=UserFav.objects.all(),
                fields=('user', 'novel'),
                message="已经收藏"
            )
        ]

        fields = ("user", "novel", "id")
```



