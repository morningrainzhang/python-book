xadmin

```
pip install git+git://github.com/sshwsfc/xadmin.git@django2
```

INSTALLED\_APPS

```
'xadmin',
'crispy_forms',
```

 adminx.py

```
import xadmin
from xadmin import views
from .models import EmailVerifyRecord,Novel


class BaseSetting(object):
    enable_themes = True
    use_bootswatch = True


class GlobalSettings(object):
    site_title = "MORAIN"
    site_footer = "www.morainz.com"
    # menu_style = "accordion"


class VerifyCodeAdmin(object):
    list_display = ['code', 'email', 'send_type','add_time']


xadmin.site.register(EmailVerifyRecord, VerifyCodeAdmin)
xadmin.site.register(views.BaseAdminView, BaseSetting)
xadmin.site.register(views.CommAdminView, GlobalSettings)


class NovelsAdmin(object):
    list_display = ["title", "state", "author", "type", "count_word", "abstract",
                    "novel_image", "create_time", "novel_url", "add_user", ]
    search_fields = ['title', "author"]
    list_editable = ["is_hot", ]
    list_filter = ["title", "state", "author", "type", "count_word", "abstract",
                    "novel_image", "create_time", "novel_url", "add_user",]
    style_fields = {"novels_desc": "ueditor"}

xadmin.site.register(Novel, NovelsAdmin)
```







