---
title: Sıfırdan Bir Django Uygulaması Geliştirme
categories: [Web Development, Django]
image:
  path: /assets/img/sifirdan-django/django-logo-negative.png
---

Bu dökümanı hazırlamamın amacı, başta kendim olmak üzere Django framework'üne aşina olan ancak zamanla kullanımını ve temel fonksiyonlarını unutan kişilere yardımcı olmaktır. Bu nedenle döküman boyunca konuların detaylarına girmeyecek ve daha ziyade püf noktalara değineceğim, zira Django yeni olanlar için internette birçok çeşitli kaynak mevcut bulunmaktadır.

Bu dökümanda Django Framework ile dinamik bir kişisel blog uygulaması yapacağım. Blog uygulamasında kullanacağım hazır bootstrap şablonuna [buradan](https://startbootstrap.com/theme/clean-blog) ulaşabilirsiniz.

## Kurulum adımları

Doğrudan konuya girerek, gereken bileşenlerin kurulumlarını yaparak başlıyorum.

```bash
python3 -m venv env
source env/bin/activate
pip3 install Django
```

> Bu döküman boyunca gösterilen ve çalıştırılan tüm komutlar `linux` işletim sistemine göre olacaktır. Ek olarak, benim bu dökümanı hazırladığım esnada mevcut Django sürümünün `5.0.1` olduğunu belirtirim.
{: .prompt-tip }

Django projemizi oluşturmak için gereken bileşenler hazır hale geldiğine göre, projemize bir isim vererek oluşturmaya başlayabiliriz. Ben burada mysite olarak isimlendireceğim.

```bash
django-admin startproject mysite
```

Django projeniz şayet başarıyla oluşturulduysa, aşağıdaki dizin yapısına sahip olmalısınız.

```bash
(env) alchustan@debian:~/mysite$ tree
.
├── manage.py
└── mysite
    ├── asgi.py
    ├── __init__.py
    ├── settings.py
    ├── urls.py
    └── wsgi.py

2 directories, 6 files
```


## Bir Django uygulaması oluşturmak

Django uygulamalarının mantığına kısaca değinecek olursam; Bir teknoloji şirketiniz ve bu şirketinizin websitesi olduğunu farz edelim.

Sitenize ait bir haberler sayfanız, teknoloji severlerin aralarında iletişim kurabileceği bir forumunuz, ve danışmanlık sağladığınız diğer şirketlere hizmet verdiğiniz başka bir web servisiniz mevcut. Bu durumda, her bir web servisinizi, Django üzerinde ayrı birer uygulama olarak yönetmeniz mümkündür. Böylece servislerinizi birbirlerinden bağımsız olarak yönetmeniz ve kullanmanız sizin için çok daha kolay olacaktır.

Bu kadar açıklama yeterli olduğuna göre, aşağıda yer alan komut ile Django uygulamanızı oluşturabilirsiniz. Ben burada uygulamamın adını blog yapacağım.

```bash
python3 manage.py startapp blog
```

### Uygulamanın kurulumu

Her oluşturduğumuz uygulamayı, Django tanıtmamız gerekmekte. Bu nedenle Django ayarlarımızın tutulduğu `settings.py` içerisine, bu uygulamımızın dizindeki yerini tanımlamamız gerekiyor.

```python
   .  .  .
INSTALLED_APPS = [
    'blog.apps.BlogConfig', # <-- Bu oluşturduğumuz uygulamamız
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
   .  .  .
```
{: file='mysite/settings.py'}

Aynı şekilde bu blog uygulaması içerisinde url yönetim işlemlerini yapabilmemiz adına, ana uygulamamız içerisinde yer alan `urls.py` ile blog urls dosyasına yönlendirme yapıyorum. 

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('blog.urls')),
]
```
{: file='mysite/urls.py'}

Oluşturduğumuz bu blog uygulaması içerisinde `urls.py` dosyası bulunmadığı için bunu da biz oluşturuyoruz. Dosyanın içeriği ise aşağıda gördüğünüz gibi olacak.

```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.home, name='home-page'),
    path('about/', views.about, name='blog-about'),
    path('post/', views.post),
]
```
{: file='blog/urls.py'}

Peki views ne içeriyor bir de ona bakalım.

```python
from django.shortcuts import render

def home(request):
    return render(request, 'home.html')

def about(request):
    return render(request, 'about.html')

def post(request):
    return render(request, 'post.html')
```
{: file='blog/views.py'}

## Django Template yapısı

Pekala, artık oluşturduğumuz blog uygulamamızı Django'ya başarılı bir şekilde tanımladık ve ardından urls ile views kısmını isteğimiz doğrultusunda düzenledik.

Ben burada örnek olarak 3 sayfa oluşturmaya karar verdim. Bunlardan bir tanesi anasayfamız olan home, sitemizin hakkımızda sayfası olan about bölümü ve oluşturacağımız blog yazıları için post. Uygulamamızın son yapısına bakacak olursak kabaca aşağıdaki tablodaki görünecek:

| urls.py  | views.py   | Tarayıcıda Görünecek Olan Adres |
|:---------|:-----------|:--------------------------------|
| ''       | home.html  |   http://127.0.0.1:8000/        |
| 'about/' | about.html |   http://127.0.0.1:8000/about/  |
| 'post/'  | post.html  |   http://127.0.0.1:8000/post/   |

> Şayet ben eğer `mysite/urls.py` dosyası içerisinde 6. satır için şu şekilde bir düzenleme yapsaydım `path('blog/', include('blog.urls')),` bu sefer tüm blog uygulamama ilişkin adreslerin başına aşağıdaki tabloda da görüldüğü üzere **/blog** eklenirdi.
{: .prompt-info }

| urls.py  | views.py   |  Tarayıcıda Görünecek Olan Adres  |
|:---------|:-----------|:----------------------------------|
| ''       | home.html  | http://127.0.0.1:8000**/blog**/       |
| 'about/' | about.html | http://127.0.0.1:8000**/blog**/about/ |
| 'post/'  | post.html  | http://127.0.0.1:8000**/blog**/post/  |

### Templates klasörü

Pekala buraya kadar mutabıksak sitemizde görüntülecek olan html dosyalarımızın nerede tutulacağına geçebiliriz. Django varsayılan olarak uygulamamız içerisinde bir `templates` klasörünün varlığını kontrol eder ve bu klasörün içine bakar. Bu nedenle views.py içerisinde bahsi geçen `home.html`, `about.html` ve `post.html` dosyalarını, uygulama dizinimin altında oluşturduğum `templates` klasöründe barındıracağım.

Bu adımları uyguladığınızda güncel Django dizin yapınız aşağıdakine benzer bir durumda olacaktır.

``` bash
(env) alchustan@debian:~/mysite$ tree
.
├── manage.py
├── db.sqlite3
├── blog
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   ├── models.py
│   ├── templates
│   │   └── home.html
│   │   └── about.html
│   │   └── post.html
│   ├── tests.py
│   ├── urls.py
│   └── views.py
└── mysite
    ├── __init__.py
    ├── asgi.py
    ├── settings.py
    ├── urls.py
    └── wsgi.py

4 directories, 16 files
```

> Siz Django uygulamanızı çalıştırdığınızda, ayrıca `__pycache__` tarzında dizinler görebilirsiniz, ancak bunlar gözardı edilebilir. Mühim olmadıkları için yukarıdaki dizin yapısında bunları eklemedim.
{: .prompt-warning }

Pekala, şimdi Django uygulamamız sorunsuz bir şekilde çalışıyormu teyit edelim.

![Home](/assets/img/sifirdan-django/home-page.png)
_home.html_

![About](/assets/img/sifirdan-django/about1.png)
_about.html_

ve görüldüğü üzere Django, sayfaları başarıyla önümüze getiriyor.


### Böl ve Yönet

Django, bizlere kullanım kolaylığı sağlamak açısından dinamik sayfalar üretebilmemize ve kod tekrarını önlemimize yardımcı oluyor. Böylece tüm sayfalarda yer alacak kodları, tekrar yazmaktan kurtulmuş oluyoruz.

Aşağıda yer alan görselde, yapacağımız sitenin bir görünümü yer almakta ve siteyi üç farklı renk ile parçalara ayırdım. Bu kısımları sırasıyla izah edecek olursam:

- Kırmızı renkli kısım: Sitenin header adını verdiğimiz bölüm ve bu kısım **statik**, yani sitenin tüm sayfalarında aynı kalacak.
- Mavi renkli kısım: Sitenin ana içeriği burada yer alacak ve bu kısım **dinamik**, yani sayfanın içeriğine göre değişken olacak.
- Sarı renkli kısım: Sitenin footer adını verdiğimiz bölüm ve bu kısım **statik**, yani sitenin tüm sayfalarında aynı kalacak.

![Site Görünümü](/assets/img/sifirdan-django/web-preview.jpeg){: width="600" height="600" .w-65 .center}
_Bu şablon açık kaynak kodlu ve kullanımı ücretsizdir, [buradan](https://startbootstrap.com/theme/clean-blog) ulaşabilirsiniz._

Burada ilk iş olarak <u>statik html</u> kodlarımın duracağı `templates` klasörü içerisinde `components` adında bir klasör daha oluşturuyorum. Ardından bu klasör içerisinde sırasıyla ilgili dosyaları oluşturuyorum:

1. `default.html` -> Sitemde, sayfa oluştururken baz alacağım temel şablondur ve tüm sayfalar bu şablon baz alınarak türetilecektir.
2. `header.html` -> Sitemin her sayfasında yer alacak olan header menüsüne ait kodlarım.
3. `footer.html` -> Sitemin her sayfasında yer alacak olan footer kısmına ait kodlarım.

Pekala, şimdi default dosyamızın neler içerdiğine bir göz atalım.

{% raw %}
```html
{% load static %}
<!DOCTYPE html>
<html lang="en">
    <head>
        <link rel="icon" type="image/x-icon" href="{% static 'assets/favicon.ico' %}" />
        <link href="{% static 'css/styles.css' %}" rel="stylesheet" />
    </head>
    <body>
        <!-- Header-->
        {% include 'components/header.html' %}

        <!-- Main Content-->
        {% block content %}
        {% endblock %}
        
        <!-- Footer-->
        {% include 'components/footer.html' %}
        <script src="{% static 'js/scripts.js' %}"></script>
    </body>
</html>

```
{: file='blog/templates/components/default.html'}


Bu dosya içeriğindeki kodları sırasıyla izah edecek olursam:

- `{% load static %}` -> Bir takım statik dosyalara ihtiyacımız olacağını belirtiyoruz.
- `{% static 'css/styles.css' %}` -> Statik dosyalarımızın barındırıldığı klasör dizinine giderek `css/styles.css` dosyasını çağırır.
- `{% include 'components/header.html' %}` -> Include fonksiyonu, bir başka html dosyasını çağırmamızı sağlar. Burada `header.html` içerisinde yer alan tüm kodların kopyalanaraktan, 8. satıra yapıştırıldığını düşünebilirsiniz.
- `{% block content %} {% endblock %}` -> Bu default sayfadan türeteceğim ve oluşturacağım tüm dinamik içeriklerimin buraya yerleştirildiğini işaret ediyorum.
{% endraw %}

> Tüm bu kodları parçalar haline getirmemizin bize yararı, sonrasında yapmak isteyebileceğimiz bir değişiklikte tek bir yere bakacağız ve kodu yönetme, değiştirme ve düzenleme işlemleri bizim için çok daha az uğraştırıcı olacak.
{: .prompt-tip }

Bu değişikliklerin sonucunda html sayfalarımızın durumuna bir bakalım.

{% raw %}
```html
{% extends 'components/default.html' %}
{% block content %}
    <div class="container px-4 px-lg-5">
        .  .  .
    </div>
{% endblock %}
```
{: file='blog/templates/home.html'}
{% endraw %}

### Statik dosyalarımız

Günümüzde her sitede olduğu üzere kullandığımız bir takım `.css` , `.js` ve görsel dosyalarımız mevcut olacaktır. Bunları kimi zaman artık CDN altyapılarında barındırsakta, birçok websitesi bu dosyaları hala kendi bünyesinde saklamaya devam ediyor.

Django bu tür *statik* dosyalarımızı, oluşturduğumuz uygulama dizinimizin altında `static` klasörü olarak kabul eder ve bu klasöre bakar. Bu nedenle bizde sitemizde kullanacağımız dosyaları bu klasör altına yerleştireceğiz.

> Statik dosyalarınızın bulunduğu klasörün adını veya konumunu değiştirmek istersek `settings.py` içerisinde yer alan `STATIC_URL = 'static/'` değiştirmemiz yeterli olacaktır.

Bu değişikliklerden sonra ise güncel `blog` uygulamamızın dizin yapısı aşağıdaki gibi görünecektir.
 

```bash
(env) alchustan@debian:~/mysite/blog$ tree
.
├── __init__.py
├── admin.py
├── apps.py
├── models.py
├── tests.py
├── urls.py
└── views.py
├── static
│   ├── assets
│   │   └── favicon.ico
│   ├── css
│   │   └── styles.css
│   └── js
│       └── scripts.js
├── templates
│   ├── components
│   │   ├── default.html
│   │   ├── footer.html
│   │   └── header.html
│   ├── home.html
│   ├── about.html
│   └── post.html

7 directories, 16 files
```

## Django Models yapısı

Models yapısı ile Django veritabanımızın genel hatlarını oluşturacağız. Burada oluşturacağımız her bir sınıf, yani **class** veritabanında bir tabloyu temsil eder, ve bu sınıf içerisinde oluşturduğumuz değişkenler ise kolonları temsil etmektedir.

Bu durumda aşğıda yer alan models dosyasına bakacak olursak; **Post** isimli bir tablo oluşturuyor ve sonrasında bu tablo içerisinde **title**, **content**, **date_posted** ve **author** kolonlarını oluşturduğumuzu söyleyebiliriz.

```python
from django.db import models
from django.utils import timezone
from django.contrib.auth.models import User

class Post(models.Model):
    title = models.CharField(max_length=100)
    content = models.TextField()
    date_posted = models.DateTimeField(default=timezone.now)
    author = models.ForeignKey(User, on_delete=models.CASCADE)
```
{: file='blog/models.py'}

### Bir kullanıcı oluşturmak

Burada dikkat etmemiz gereken bir diğer nokta da user kısmı. Henüz projemizde bir user bulunmamakta, bu nedenle yaptığımız değişiklikleri veritabanına uygulamadan önce, bir super-user, yani *admin* rolüne sahip bir kullanıcı oluşturmalıyız.

Bunun için sırasıyla aşağıdaki iki komutu çalıştırarak kullanıcı oluşturabiliriz. 


```shell
(env) alchustan@debian:~/mysite$ python3 manage.py migrate
(env) alchustan@debian:~/mysite$ python3 manage.py createsuperuser
```

Bir user oluşturduğumuza göre veritabanı değişikliklerimizi uygulayabiliriz. Aşağıda yer alan iki komutu sırasıyla çalıştırdığımız taktirde tablo ve kolonlarımız oluşmuş olacaktır.

```shell
(env) alchustan@debian:~/mysite$ python3 manage.py makemigrations
(env) alchustan@debian:~/mysite$ python3 manage.py migrate
```

### Django'da CRUD işlemleri


Bu noktadan sonra, Django üzerine okuduğum ve izlediğim içeriklerin neredeyse hepsi python shell arayüzüne geçiş yaparak veri yönetimi işlemlerini gerçekleştiriyor.

Ancak ben, bu dökümanda farklılık yapacağım ve oluşturacağımız bir python dosyası üzerinden bu işlemleri gerçekleştireceğiz.

Django projemin ana dizininde `myscript` isimli yeni bir python dosyası oluşturuyorum.

```python
import django
import os

os.environ.setdefault("DJANGO_SETTINGS_MODULE", "mysite.settings")
django.setup()

from blog.models import Post
from django.contrib.auth.models import User
```
{: file='mysite/myscript.py'}

> Burada önce django ve os kütüphaneleri dahil edildikten sonra django ayarlarımızı kaydetmiş olmamız önemli. Eğer Post ve User başta eklemeye çalışırsanız, django ayarları kayıt olmadığı için hata alacaksınız.
{: .prompt-warning }

#### Create (Oluşturma)

```python
# Yeni bir post oluşturup kaydetme
post1 = Post(title='Proje Başlığı', content='. . .' ,author=User.objects.get(id=1))
post1.save()
```

#### Read (Okuma)

```python
# Tüm postları çekme
all_posts = Post.objects.all()
print(all_posts)

# Belirli bir postu id kullanarak çekme
specific_post = Post.objects.get(id=1)
print(specific_post)
```

#### Update (Güncelleme)

```python
# Belirli bir postu güncelleme
post_to_update = Post.objects.get(id=1)
post_to_update.title = 'Güncellenmiş Başlık'
post_to_update.content = 'Güncellenmiş İçerik'
post_to_update.save()
```

#### Delete (Silme)

```python
# Belirli bir postu silme
post_to_delete = Post.objects.get(id=1)
post_to_delete.delete()
```

### Shell ile dosya arasındaki fark

Peki bu işlemleri python shell ile, oluşturduğumuz bir dosya üzerinde gerçekleştirmemiz arasındaki fark nedir?

Bu işlemleri bir dosya üzerinden gerçekleştirmeyi istememin sebebi; Birçok veriyi döngüler ile veritabanına aktarabilmek. Eğer kapsamlı bir web uygulaması yapmak istiyorsanız, düzenli olarak işlediğiniz verileri sitenizde görünür kılmak isteyebilirsiniz. Bu durumda verileri shell üzerinden teker teker işlemeniz imkansız olacaktır.

## Verilerimizi görüntüleme

Pekala, artık verilerimizi veritabanımıza işlediğimize göre, bunları sitemizde görünür kılmaya geçebiliriz. Sırasıyla, önce verilerimi views ile html dosyama aktarıyorum.

```python
from django.shortcuts import render
from .models import Post

def home(request):
    context = {'posts': Post.objects.all()}
    return render(request, 'home.html', context)
```
{: file='blog/views.py'}

Ardından bir for döngüsü ile verilerimi sırasıyla alıyorum.

{% raw %}
```html
{% extends 'components/default.html' %}
{% block content %}
<div class="container px-4 px-lg-5">
  <div class="row gx-4 gx-lg-5 justify-content-center">
    <div class="col-md-10 col-lg-8 col-xl-7">

    {% for post in posts %}
      <!-- Post preview-->
      <div class="post-preview">
        <a href="post.html">
          <h2 class="post-title">{{ post.title }}</h2>
          <h3 class="post-subtitle">{{ post.content }}</h3>
        </a>
        <p class="post-meta"> Posted by <a href="#!">{{ post.author }}</a>
        on {{ post.date_posted }}
        </p>
      </div>
      <!-- Divider-->
      <hr class="my-4" />
    {% endfor %}

    </div>
  </div>
</div>
{% endblock %}
```
{: file='templates/home.html'}
{% endraw %}

## Django Admin paneli

Models yapısını anlattığımız esnada bir user oluşturmuştuk, peki bu super-user nelere kadir?

Django Framework, yapısı itibariyle bir admin arayüzü ile birlikte gelmekte. Bu arayüz, CRUD işlemlerimizin tamamını görsel olarak yapabilmemizi mümkün kılıyor.

Bu arayüze geçmeden önce, veritabanımızda bulunan **Blog** uygulamamızın **Post** tablosunu, bu admin paneline tanıtmamız gerek.

```python
from django.contrib import admin
from .models import Post

admin.site.register(Post)
```
{: file='blog/admin.py'}

Bu işlemi tamamladıktan sonra panelimize geçiş yapabiliriz.

> Bu panel, sitemizin `/admin` kısmında yer alıyor. Bu bilgiyi nereden edindiğimiz sorulacak olursa `mysite/urls.py` içerisinde bir yönlendirme mevcut.
{: .prompt-info }

http://127.0.0.1:8000/admin/ Adresine gidersek, bizi bir giriş ekranı karşılayacak. Oluşturduğumuz super-user bilgileri ile buraya giriş yapabiliyoruz.

Giriş yaptığımız taktirde bizi karşılayacak olan arayüz aşağıda yer alan görseldeki gibi olacaktır.

![Admin Panel](/assets/img/sifirdan-django/panel1.png)

Eğer Post kısmına bakacak olursak, mevcut postlarımızı görüntüleyebiliyoruz. Ayrıca sağ üst kısımda yer alan **Add Post** seçeneği ile yeni bir post ekleyebildiğimizi görüyoruz.

![Posts](/assets/img/sifirdan-django/image.png)

Ancak burada mevcut Postlarımıza baktığımızda, isimleri bizim için yeterince anlaşılır değiller. Bunları kendi Post başlıklarımız ile değiştirirsek, bizim için çok daha anlaşılır olacaklardır.

Bunun için models içerisinde oluşturduğum Post sınıfıma `__str__` fonksiyonunu eklemem yeterli olacaktır. `return self.title` ile başlığı döndürmesini istediğimi belirtiyorum.

```python
from django.db import models
from django.utils import timezone
from django.contrib.auth.models import User

class Post(models.Model):
    title = models.CharField(max_length=100)
    content = models.TextField()
    date_posted = models.DateTimeField(default=timezone.now)
    author = models.ForeignKey(User, on_delete=models.CASCADE)

    def __str__(self):
        return self.title
```
{: file='blog/models.py'}

Tekrar aynı arayüze dönecek olursak, artık her bir Post'u, bizim isimlendirdiğimiz başlık ile görebiliriz.

![Updated Posts](/assets/img/sifirdan-django/panel.png)


## Sonuç

Buraya kadar okuduysanız, artık temel düzeyde bir Django projesinin başlatılmasından, uygulamaların oluşturulmasına, oluşturduğumuz uygulamaların yönetimlerine, bu uygulamaların veritabanı ile etkileşimlerine ve admin paneline hakimsiniz demektir.

Bu döküman, başlangıçta kendim için çıkardığım notlardan oluşurken, bunları açık olarak paylaşma isteğim sonucunda blog yazım olarak evrildi. Dökümanda eksik veya yanlış olduğunu gördüğünüz noktalar varsa, düzeltmem için beni bilgilendirmenizden müteşekkir olurum. Ayrıca ihtiyaç duyduğum taktirde, zaman içerisinde dökümana eklemelerde bulunabilirim.


## Yararlandığım Kaynaklar 

1. Django'nun kendi dökümantasyonu [↩](https://docs.djangoproject.com/en/)
2. Django Tutorials - Corey Schafer
[↩](https://youtube.com/playlist?list=PL-osiE80TeTtoQCKZ03TU5fNfx2UY6U4p&si=uaJSkBOftKBeZUZP)
