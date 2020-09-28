
# python ubuntu+Django+uwsgi+nginx 設定

## 主要内容
> ### python安裝
> ### 虛擬環境設置
> ### Django安裝+專案測試
> ### uwsgi安裝+測試
> ### nginx安裝+測試

# python 安裝

使用miniconda來設置python環境
直接到miniconda的官網下載 .sh安裝檔

`wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh`

利用bash指令來安裝

`bash Miniconda3-latest-Linux-x86_64.sh`






# 設置虛擬環境

常用的指令

建立環境
`conda create -n myenv python=3.6`

查看現有環境
`conda env list`

刪除環境
`conda remove --name myenv --all`

進入環境
`conda activate myenv`

退出環境
`conda deactivate`



# Django 安裝+專案測試

1.`pip install django`

2.`django-admin.py startproject demosite`

3.`python manage.py startapp myapp`

4.修改`myapp/view.py`

```python
from django.http import HttpResponse
def index(request):
    return HttpResponse("Hello, world.")

```
5.新增`myapp/urls.py`

```python
from django.urls import path

from . import views

urlpatterns = [
    path('', views.index, name='index'),
]

```

6.`修改demosite/urls.py`

```python
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('myapp/', include('myapp.urls')),
    path('admin/', admin.site.urls),
]

```


7.`python manage.py runserver`

即可`http://localhost:8000/myapp` 看到結果!



# 安裝 uwsgi + 測試

若是直接使用`pip install uwsgi`可能會看到錯誤訊息，估計是gcc compile的版本不對所造成，`sudo apt install gcc`更新版本也不能解決。

在這裡建議使用conda安裝，已經有有志之士整理好套件了。

用`conda install -c conda-forge uwsgi`可以正常安裝

>1.測試uwsgi是否正常

建立`test.py`

```python

def application(env, start_response):
    start_response('200 OK', [('Content-Type','text/html')])
    return [b"Hello World"] # python3


```


輸入`uwsgi --http :8001 --wsgi-file test.py`就可以在`http://localhost:8001/` 看到Hello World


>2.用uwsgi 測試django server

在demosite根目錄`uwsgi --http :8001 --module demosite.wsgi`

就可以在`http://localhost:8001/myapp` 看到結果



# 安裝 nginx + 測試

1.`sudo apt update`

2.`sudo apt install nginx`

3.`sudo systemctl start nginx`

4.在http://localhost:80 可以看到結果。nginx設定檔在 `etc/nginx/site-enabled/deafult`

# [實戰]用nginx+uwgsi 來架設django

1.在demosite跟目錄，建立`uwsgi_params`檔案

```
uwsgi_param  QUERY_STRING       $query_string;
uwsgi_param  REQUEST_METHOD     $request_method;
uwsgi_param  CONTENT_TYPE       $content_type;
uwsgi_param  CONTENT_LENGTH     $content_length;
 
uwsgi_param  REQUEST_URI        $request_uri;
uwsgi_param  PATH_INFO          $document_uri;
uwsgi_param  DOCUMENT_ROOT      $document_root;
uwsgi_param  SERVER_PROTOCOL    $server_protocol;
uwsgi_param  REQUEST_SCHEME     $scheme;
uwsgi_param  HTTPS              $https if_not_empty;
 
uwsgi_param  REMOTE_ADDR        $remote_addr;
uwsgi_param  REMOTE_PORT        $remote_port;
uwsgi_param  SERVER_PORT        $server_port;
uwsgi_param  SERVER_NAME        $server_name;

```
2.在`demosite`跟目錄建立`mySite_nginx.conf`

利用nginx開啟一個port :8000的server，並且連接到 :8001

```
upstream django {
    server 127.0.0.1:8001;
}
 
# configuration of the server
server {
    listen      8000;
    server_name 127.0.0.1;
    charset     utf-8;
 
    location /static {
        alias /home/lin/django/demosite/static;
    }
 
    location / {
        uwsgi_pass  django;
        include     /home/lin/django/demosite/uwsgi_params;
    }
}

```




3.建立`symbolic link`到nginx設定檔目錄下

>`sudo ln -s ~/django/demosite/mySite_nginx.conf /etc/nginx/sites-enabled/`

4.兩個方式開啟uwsgi

> 1.`uwsgi --socket :8001 --module demosite.wsgi`


> 2.在`demosite`建立`uwsgi.ini`，輸入`uwsgi --ini uwsgi.ini`

```
[uwsgi]
# http = 127.0.0.1:8000
socket = :8001  ;這邊要注意http
chdir=/home/lin/django/demosite/
; module=mysite.wsgi7
wsgi-file=/home/lin/django/demosite/demosite/wsgi.py
master = True
processes = 2
threads = 4
vacuum = true
pidfile = uwsgi.pid
chmod-socket = 664
home=/home/lin/miniconda3/envs/django/

```

5.`http://localhost:8000` 完成!


