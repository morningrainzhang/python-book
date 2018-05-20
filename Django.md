#Django框架学习记录
Django==1.8.2
Python==3.6.2

`django-admin startproject django_blog`  
`python manage.py runserver`  
`python manage.py startapp blog`

	INSTALLED_APPS = [
	    'blog',  # 将新建的app注册到项目中
	]
	
	TEMPLATES = [
	    {
	        'DIRS': [
	            os.path.join(BASE_DIR, 'templates'),  # 模板文件的路径设置
	        ],
	    },
	]
	
	STATICFILES_DIRS = (
	    os.path.join(BASE_DIR, 'static'),  # 静态文件路径设置
	)


	{% load staticfiles %}
	{% static 'css/main.css'%}


配置日志文件
	import pymysql
	
	pymysql.install_as_MySQLdb()
	
	python3 manage.py migrate
	python3 manage.py createsuperuser

python3  manage.py makemigrations && python3 manage.py migrate 


	my.cnf 改为utf8mb4
	修改mysql配置文件my.cnf（windows为my.ini）
	
	my.cnf一般在etc/mysql/my.cnf位置。找到后请在以下三部分里添加如下内容：
	
	[client]
	
	default-character-set = utf8mb4
	
	[mysql]
	
	default-character-set = utf8mb4
	
	[mysqld]
	
	character-set-client-handshake = FALSE
	
	character-set-server = utf8mb4
	
	collation-server = utf8mb4_unicode_ci
	
	init_connect='SET NAMES utf8mb4'
	
	
	show variables like '%character%';
	show variables like'%collation%'
	
ln -s
rm 
source env/bin/activate
## ORM

## 虚拟环境

virtualenv TestEnv 创建
virtualenv --system-site-packages TestEnv 添加site-packages
source ./bin/activate 进入
deactivate quit

## 导出python库
pip freeze > requirements.txt
pip install -r requirements.txt