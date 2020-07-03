# Django with react docker docker-compose

## 安装
1. 配置 dockerfile 和 docker-compose.yml 文件
2. Create a requirements.txt in your project directory.
3. `docker-compose build` 构建镜像
  > * 因为镜像的工作目录是 /app  所以文件都是相对/app 的路径，
  > * 生成 django 项目目录
    > > * `docker-compose run --rm --no-deps web django-admin startproject famous  ./famous/`   # /app/famous/famous  
    注意 __docker-compose.yml volumes: - .:/app__, . 是主机famous-platform目录，所以 会生成 famous-platform/famous/famous  
    > >  * 如果 __-.:/app__   `docker-compose run --rm --no-deps web django-admin startproject famous` # /app/famous  
    此时 /famous-platform/famous 项目内容都会在此目录里, 没有分开
    > > * 如果 - **./famous:/app**   `docker-compose run --rm --no-deps web django-admin startproject famous ./`  #/app/famous
      此时 成功 创建。如果不加 ./ 报 /app/famous 已存在

  > * 创建项目后 
     挂载券改为 下

      ``` 
      volumes:
          - ./famous:/app
          - ./famous/famous/:/app/famous:Z 
      ```

  > * 创建 app

    ```
    docker-compose run --rm --no-deps web python3 manage.py startapp polls 
    ```

4. famous/settings.py 设置 db

## 运行 
  ```
  docker-compose up
  ```

## dockerfile 解释 
  多段构建 您可以选择性地将工件从一个阶段复制到另一个阶段，从而在最终image中只留下您想要的内容。
  famous docker 分两段构建 第一段构建应用程序所需的所有内容，第二段构建 copy 第一段 内容 并运行

# famous 项目结构

  ``` 
  famous/
      manage.py 
      famous/ 
          __init__.py
          settings.py 
          urls.py 
          asgi.py
          wsgi.py 
  ```
## 解释
- 最外层的 famous/ 根目录只是你项目的容器， 根目录名称对Django没有影响，你可以将它重命名为任何你喜欢的名称。
- manage.py: 一个让你用各种方式管理 Django 项目的命令行工具。你可以阅读 [django-admin and manage.py](https://docs.djangoproject.com/zh-hans/3.0/ref/django-admin/)。
- 里面一层的 famous/ 目录包含你的项目，它是一个纯 Python 包。它的名字就是当你引用它内部任何东西时需要用到的 Python 包名。 (比如 famous.urls).
- famous/__init__.py：一个空文件，告诉 Python 这个目录应该被认为是一个 Python 包
- famous/settings.py：Django 项目的配置文件。如果你想知道这个文件是如何工作的，请查看 [Django 配置](https://docs.djangoproject.com/zh-hans/3.0/topics/settings/) 了解细节。
- famous/urls.py：Django 项目的 URL 声明，就像你网站的“目录”。阅读 [URL调度器](https://docs.djangoproject.com/zh-hans/3.0/topics/http/urls/)文档
- famous/asgi.py：作为你的项目的运行在 ASGI 兼容的Web服务器上的入口。阅读 如何使用 [WSGI 进行部署](https://docs.djangoproject.com/zh-hans/3.0/howto/deployment/wsgi/) 了解更多细节。
- famous/wsgi.py：作为你的项目的运行在 WSGI 兼容的Web服务器上的入口。阅读 如何使用 [WSGI 进行部署](https://docs.djangoproject.com/zh-hans/3.0/howto/deployment/wsgi/) 了解更多细节。

## wsgi 解释
  wsgi 是web 服务器 和 web 应用的python 标准,  就是 web服务器 与 Django 交互用,  

## uWSGI 是 应用容器服务器, 托管Django 

# 数据库

```
# setting.py
   
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'postgres',
        'USER': 'postgres',
        'PASSWORD': 'postgres',
        'HOST': 'db',
        'PORT': 5432,
    }
}
```
These settings are determined by the postgres Docker image specified in docker-compose.yml.

## 应用迁移 
``` 
docker-compose run --rm web python3 manage.py migrate
```
这个 migrate 命令检查 INSTALLED_APPS 设置, 至于具体会创建什么，这取决于 famous/settings.py 设置文件和每个应用的数据库迁移文件

## 创建模型 并激活模型
  * polls/models.py 添加
  * 把  polls.apps.PollsConigy 加到 famous/settings.py
  * 生成迁移文件

  ```
  docker-compose run --rm web python3 manage.py makemigrations polls
  ```
  polls/migrations/0001_initial.py 并可以手动调整
  * 查看对应SQL
  ```
  python manage.py sqlmigrate polls 0001
  ```

  * 再次运行 migrate 命令  在数据库里创建新定义的模型的数据表
  ``` 
  docker-compose run --rm web python3 manage.py migrate
  ```

  ### 总结
  - 编辑 models.py 文件，改变模型。
  - 运行 `python manage.py makemigrations` 为模型的改变生成迁移文件。
  - 运行 `python manage.py migrate` 来应用数据库迁移。

## 进入 django shell
我们使用这个命令而不是简单的使用 "Python" 是因为 manage.py 会设置 DJANGO_SETTINGS_MODULE 环境变量，这个变量会让 Django 根据 famous/settings.py 文件来设置 Python 包的导入路径

 ```docker-compose run --rm web python3 manage.py shell```

## admin 页面 添加 model 修改
polls/admin.py 
```
from django.contrib import admin

from .models import Question

admin.site.register(Question)  # 添加 Question 模型

```

# 静态文件管理
##  配置静态文件
1. 确保INSTALLED_APPS包含了django.contrib.staticfiles。
2. 在 setting 里配置 STATIC_URL 
3. 在模板中，用static模板标签基于配置*STATICFILES_STORAG*E位给定的相对路径构建URL
  ```
  {% load static %}
  <img src="{% static "polls/example.jpg" %}" alt="My image">
  ````
4. 将您的静态文件保存至程序中称为static的目录中。例如polls/static/polls/example.jpg。
  
## 开发时提供静态文件服务
例如，若MEDIA_URL定义为*/media/*，你可以通过将以下代码片段加入*urls.py*实现目的：
  ``` 
  from django.conf.urls.static import static

  urlpatterns = [
      # ... the rest of your URLconf goes here ...
  ] + static(settings.STATIC_URL, document_root=settings.STATIC_ROOT)
  ```

## 部署
1. 将*STATIC_ROOT*配置成你喜欢的目录，在这个目录提供服务，例如：
```
STATIC_ROOT = "/var/www/example.com/static/"
```
2. 运行collectstatic管理命令：`docker-compose run --rm web python manage.py collectstatic`
这将会把静态目录下的所有文件复制到STATIC_ROOT目录。

3. 选定一个Web服务器为这些文件提供服务。

  > 1. 在同一服务器提供站点和静态文件服务
    配置的Web服务器，在使其STATIC_URL下为STATIC_ROOT目录下的文件提供静态文件服务
    [例如，这里是如何以Apache的配合mod_wsgi的开始](https://docs.djangoproject.com/zh-hans/3.0/howto/deployment/wsgi/modwsgi/#serving-files)。

  > 2. 专用服务器提供静态文件服务
      使用一个独立的Web服务器 如Nginx
      本地将STATIC_ROOT推送到静态文件服务器提供服务的目录。rsync的是一个常见选项，因为这种配置只会传输文件修改部分的数据流。

  > 3. 从云服务或CDN提供静态文件服务

  > > 使用这些服务时，基本的工作流程与上面类似，此外还要使用静态文件传输给存储服务商或CDN，而不是用rsync将静态文件传输给服务器。

  > > 您可以通过多种方式来执行此操作，但是如果提供程序具有API，则可以使用自定义文件存储后端 将CDN与Django项目集成。如果您已经编写或正在使用第三方的自定义存储后端，则可以collectstatic通过设置STATICFILES_STORAGE存储引擎来告诉您使用它。

  > >  例如，若你已在myproject.storage.S3Storage中写了一个S3存储预设，可以这么用：
  `STATICFILES_STORAGE = 'myproject.storage.S3Storage'`

  > >  完成此操作后，所有您需要做的就是运行，collectstatic并且您的静态文件将通过存储包推送到S3。如果以后需要切换到其他存储提供商，则只需更改STATICFILES_STORAGE设置即可。









