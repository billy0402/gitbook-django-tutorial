# 部署到 Heroku 平台

## 調整正式環境用專案內容

[官方文件](https://devcenter.heroku.com/articles/django-app-configuration)

### 安裝正式環境需求套件

```shell
$ pipenv install gunicorn whitenoise dj-database-url psycopg2-binary
```

### core/settings.py

```python
MIDDLEWARE = [
    ...
    'whitenoise.middleware.WhiteNoiseMiddleware',
    ...
]

...

# Static files (CSS, JavaScript, Images)
# https://docs.djangoproject.com/en/3.1/howto/static-files/

STATIC_URL = '/static/'

STATICFILES_STORAGE = 'whitenoise.storage.CompressedManifestStaticFilesStorage'

STATIC_ROOT = BASE_DIR / 'staticfiles'

...

# Email

try:
    with open(BASE_DIR / 'core/email.txt', 'r') as file:
        email_data = file.readlines()
        email_account, email_password = email_data

        EMAIL_HOST_USER = email_account.strip()

        EMAIL_HOST_PASSWORD = email_password.strip()
except FileNotFoundError:
    print('core/email.txt is not exists.')
except ValueError:
    print('core/email.txt does not have email account or password.')
except Exception as e:
    print(type(e))
```

### core/production_settings.py

```python
import dj_database_url

from .settings import *

DEBUG = False
ALLOWED_HOSTS = ['coffee-shopp.herokuapp.com']
SECURE_PROXY_SSL_HEADER = ('HTTP_X_FORWARDED_PROTO', 'https')

DATABASES = {
    'default': dj_database_url.config(conn_max_age=600, ssl_require=True),
}
```

### Procfile

```text
web: gunicorn core.wsgi
```

### [runtime.txt](https://devcenter.heroku.com/articles/python-support#specifying-a-python-version)

```text
python-3.8.10
```

### static/.gitkeep

## 安裝 Heroku CLI

- [網址](https://devcenter.heroku.com/articles/heroku-cli)

## 進行部署

```shell
# 登入你的 heroku 帳號
$ heroku login

# 開到你的專案資料夾下 (如果不在專案資料夾下的話)
$ cd ${your_project_path}

# 初始化 git (如果尚未建立過 git 的話)
$ git init

# 將所有檔案異動放入暫存區
$ git add .

# 將暫存區的異動提交到本地儲存庫
$ git commit -am "heroku set up"

# 在專案內設定你的 Heroku 遠端位置
$ heroku git:remote -a coffee-shopp
# 跟 Heroku CLI 幫你做的事等效
$ git remote add heroku https://git.heroku.com/${your_app_name}.git

# 推上 Heroku
$ git push heroku master
```

```shell
Enumerating objects: 178, done.
Counting objects: 100% (178/178), done.
Delta compression using up to 8 threads
Compressing objects: 100% (164/164), done.
Writing objects: 100% (178/178), 27.48 KiB | 2.11 MiB/s, done.
Total 178 (delta 86), reused 0 (delta 0)
remote: Compressing source files... done.
remote: Building source:
remote: 
remote: -----> Building on the Heroku-20 stack
remote: -----> Determining which buildpack to use for this app
remote: -----> Python app detected
remote: -----> Using Python version specified in runtime.txt
remote: cp: cannot stat '/tmp/build_e9295933/requirements.txt': No such file or directory
remote: -----> Installing python-3.8.10
remote: -----> Installing pip 20.2.4, setuptools 47.1.1 and wheel 0.36.2
remote: -----> Installing dependencies with Pipenv 2020.11.15
remote:        Installing dependencies from Pipfile.lock (ac4619)...
remote: -----> Installing SQLite3
remote: -----> $ python manage.py collectstatic --noinput
remote:        core/email.txt is not exists.
remote:        161 static files copied to '/tmp/build_e9295933/staticfiles', 505 post-processed.
remote: 
remote: -----> Discovering process types
remote:        Procfile declares types -> web
remote: 
remote: -----> Compressing...
remote:        Done: 76M
remote: -----> Launching...
remote:        Released v5
remote:        https://coffee-shopp.herokuapp.com/ deployed to Heroku
remote: 
remote: Verifying deploy... done.
To https://git.heroku.com/coffee-shopp.git
 * [new branch]      master -> master
```

### 常用的 Heroku 指令

```shell
# 配置 Heroku 環境變數
$ heroku config:set ENV_VAR='YOUR_ENV_VAR'

# 持續打印日誌
$ heroku logs --tail

# 執行 Heroku 遠端終端機
$ heroku run ${your_command}
```
