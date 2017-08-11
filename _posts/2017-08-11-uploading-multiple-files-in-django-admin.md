---
layout: post
title: Upload multiple files in Django admin
---

In my current project I wanted to let user upload multiple photos at once in Django admin. I got intimidated by huge libraries that do this and dozens of other things. So I ended up using small tweak which works surprisingly well.

All we need are two files. `change_form.html` template override and `admin.py`. Template needs to be located in following structure.

```
├── admin.py
├── templates
│   ├── admin
│   │   └── catalog(app name)
│   │       └── car(model name)
│   │           └── change_form.html
```


In file `change_form.html` we need to override `inline_field_sets` block and include single `<input` there.
{% highlight django %}
{% raw %}
{% extends "admin/change_form.html" %}
{% block inline_field_sets %}
{% for inline_admin_formset in inline_admin_formsets %}
    {% include inline_admin_formset.opts.template %}
{% endfor %}
<fieldset class="module">
  <h2>Upload Photos</h2>
  <input name="photos_multiple" type="file" multiple />
</fieldset>
{% endblock %}
{% endraw %}
{% endhighlight %}

Then we need to handle data we receive when the model forms gets submitted. We will do that by overriding `save_model` inside our model admin. All we need to do is reach out for `photos_multiple` field and save all files that might be there. My `Car` model references `photos` which has only one `ImageField` called `image`.

{% highlight python %}
class CarAdmin(admin.ModelAdmin):

    def save_model(self, request, obj, form, change):
        obj.save()

        for afile in request.FILES.getlist('photos_multiple'):
            obj.photos.create(image=afile)
{% endhighlight %}

If everything worked out fine you should see *Upload Photos* section under inline model. There's a complete `admin.py` below if you need a context.

{% highlight python %}
from django.contrib import admin
from .models import Car, Photo

class PhotoInline(admin.StackedInline):
    model = Photo
    extra = 1

class CarAdmin(admin.ModelAdmin):
    inlines = [PhotoInline]

    def save_model(self, request, obj, form, change):
        obj.save()

        for afile in request.FILES.getlist('photos_multiple'):
            obj.photos.create(image=afile)

admin.site.register(Car, CarAdmin)
{% endhighlight %}

In later post I will probably cover how to configure Travis CI to run `LiveServerTestCase`.
