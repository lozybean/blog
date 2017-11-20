Title: ModelChoiceField实现增加以及修改功能
Date: 2017-11-20
Category: pages
Tags: python, django

在一个表单中, 若某个字段通过外键查询另外的数据库, 一般借助ModelChoiceField完成. 本文介绍如何在使用ModelChoiceField时, 自定义控件, 并且以弹出窗口的形式, 实现增加、修改的功能.

## 来自django.admin的例子

!(django-admin)[http://wx2.sinaimg.cn/large/95202659gy1flolgm72gdj20ni0oaq3f.jpg]

如上图, 在ForeignKey的字段, 可点击右侧的按钮, 并弹出一个窗口供修改、增加一个新的记录.

实现步骤为:

1. 弹出窗口的表单以及路由规则;
2. 自定义修改、增加的按钮widget;
3. 在django-admin带的RelatedObjectLookups.js基础上实现前端功能;

## 使用django-addanother

django-addanother提供了处理弹出窗口的Mixin类, 只需要在自己的view类中, 继承这个Mixin即可:

```python
from django.views import generic
from django_addanother.views import CreatePopupMixin, UpdatePopupMixin
from your_app.form import YourModelForm

class AddPopupView(CreatePopupMixin, generic.CreateView):
    template_name = 'form_popup.html'
    form_class = YourModelForm
    fields = None

    def get(self, request, *args, **kwargs):
        # do something
        return super().get(request, *args, **kwargs)

    def post(self, request, *args, **kwargs):
        # do something
        return super().post(request, *args, **kwargs)


class EditPopupView(UpdatePopupMixin, generic.UpdateView):
    template_name = 'form_popup.html'
    form_class = YourModelForm
    fields = None

    def get(self, request, *args, **kwargs):
        # do something
        return super().get(request, *args, **kwargs)

    def post(self, request, *args, **kwargs):
        # do something
        return super().post(request, *args, **kwargs)
```

## 自定义widget

### 自定义widget模板
django-addanother默认使用了django-admin的widget, 通过修改默认的template来修改widget, 如使用bootstrap写的template:

```html
{% load i18n static %}
<div class="related-widget-wrapper input-group">
    {{ widget }}

    {% if edit_related_url %}
        <span class="input-group-addon">
        <a data-href-template="{{ edit_related_url }}?{{ url_params }}"
           class="related-widget-wrapper-link change-related"
           title="{% blocktrans %}Edit {{ name }}{% endblocktrans %}"
           id="change_id_{{ name }}">
            <span class="glyphicon glyphicon-edit"></span>
        </a>
        </span>
    {% endif %}
    {% if add_related_url %}
        <span class="input-group-addon">
        <a href="{{ add_related_url }}?{{ url_params }}"
           class="related-widget-wrapper-link add-related"
           title="{% blocktrans %}Add {{ name }}{% endblocktrans %}"
           id="add_id_{{ name }}">
            <span class="glyphicon glyphicon-plus"></span>
        </a>
        </span>
    {% endif %}
</div>
```

### 定义并使用widget

```python
from django_addanother.widgets import BaseRelatedWidgetWrapper


class MyBaseRelatedWidgetWrapper(BaseRelatedWidgetWrapper):
    template = 'forms/widgets/related_widget_wrapper.html'

    class Media:
        css = {
            'all': ('django_addanother/addanother.css',)
        }
        js = (
            'django_addanother/django_jquery.js',
            static('js/RelatedObjectLookups.js'),
            # 'admin/js/admin/RelatedObjectLookups.js',
        )
        if django.VERSION < (1, 9):
            # This is part of "RelatedObjectLookups.js" in Django 1.9
            js += ('admin/js/related-widget-wrapper.js',)


class AddAnotherWidgetWrapper(MyBaseRelatedWidgetWrapper):
    """Widget wrapper that adds an add-another button next to the original widget."""

    def __init__(self, widget, add_related_url, add_icon=None):
        super(AddAnotherWidgetWrapper, self).__init__(
            widget, add_related_url, None, add_icon, None
        )


class EditSelectedWidgetWrapper(MyBaseRelatedWidgetWrapper):
    """Widget wrapper that adds an edit-related button next to the original widget."""

    def __init__(self, widget, edit_related_url, edit_icon=None):
        super(EditSelectedWidgetWrapper, self).__init__(
            widget, None, edit_related_url, None, edit_icon
        )


class AddAnotherEditSelectedWidgetWrapper(MyBaseRelatedWidgetWrapper):
    """Widget wrapper that adds both add-another and edit-related button
    next to the original widget.
    """
```

上述代码中, media将系统的js文件替代成了自己的, 便于实现一些特定功能.

在自己的form中, 使用刚定义的widget就可以了:

```python
from django import forms
from django.urls import reverse_lazy

from your_app.models import YourModel, YourForeignModel
from your_app.widgets import AddAnotherEditSelectedWidgetWrapper

class YourModelForm(form.ModelForm):
    foo_fk = form.ModelChoiceField(
        query_set = YourForeignModel.objects,
        widget = AddAnotherEditSelectedWidgetWrapper(
            widget=forms.Select,
            add_related_url=reverse_lazy('foo_add_popup'),
            edit_related_url=reverse_lazy('foo_edit_popup', args=['__fk__']),
        )
    )

    class Meta:
        model = YourModel
        fields = ['__all__']
```

### 模板适配

最后别忘了在弹出窗口中隐藏导航栏等不必要的部件, 如:

```html
<!--base.html-->
<! ... >

{% if not view.is_popup %}
    {% include 'navbar.html' %}
{% endif %}

<! ... >
```

## 前端功能

django-admin已经写好了一个`RelatedObjectLookups.js`脚本, 里面实现了基本的前端功能, 只需要在这个基础上进行必要的修改即可.

如果使用的是django-admin中带的`RelatedObjectLookups.js`来实现前端功能, 则需要注意:

1. 整个widget必须以`related-widget-wrapper`为类;
2. 按钮连接的类必须包含`related-widget-wrapper-link`以及`(add|change|delete)-related`其中的一个;
3. 为了修改按钮实现动态URL, 该ID以**change_id_{{ name }}**的形式标记(便于下一步js中的处理)

默认的`RelatedObjectLookups.js`中的`updateRelatedObjectLinks`函数, 只支持`a`元素作为`select`元素的同级元素, 在上述widget中, `a`元素写在了`span`中, 所以默认的函数不能实现该功能, 需要定义一个新的函数来用过ID处理:

```js
function updateRelatedObjectLinksById(triggeringLink) {
    var $this = $(triggeringLink);
    var triggerId = triggeringLink.id;
    var changeId = 'change_'.concat(triggerId);
    var siblings = $("#"+changeId);
    if (!siblings.length) {
        return;
    }
    var value = $this.val();
    if (value) {
        siblings.each(function() {
            var elm = $(this);
            elm.attr('href', elm.attr('data-href-template').replace('__fk__', value));
        });
    } else {
        siblings.removeAttr('href');
    }
}
```

并且在主函数中修改相应的event监听函数.

## 实现效果

![1](http://wx4.sinaimg.cn/large/95202659gy1flooycxzlmj20pe0p50t8.jpg)

编辑:

![2](http://wx3.sinaimg.cn/large/95202659gy1flooyf3gt1j20u30gpmxk.jpg)

新增:

![3](http://wx1.sinaimg.cn/large/95202659gy1flooyh5k25j20v20i774x.jpg)