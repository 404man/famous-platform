## 进入 django shell
我们使用这个命令而不是简单的使用 "Python" 是因为 manage.py 会设置 DJANGO_SETTINGS_MODULE 环境变量，这个变量会让 Django 根据 famous/settings.py 文件来设置 Python 包的导入路径

 ```docker-compose run --rm web python3 manage.py shell```