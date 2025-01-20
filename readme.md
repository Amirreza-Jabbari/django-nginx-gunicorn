<div dir="rtl">

# استقرار پروژه Django در VPS:

- پیش‌نیازها:
    - ایجاد فایل requirements.txt که شامل تمام بسته‌های استفاده شده در پروژه باشد:
        - `pip freeze > requirements.txt`
    - تنظیم فایل‌های استاتیک:
        - نصب whitenoise:
            - `pip install whitenoise`
            - `pip freeze > requirements.txt`
        - افزودن خط زیر به ابتدای لیست middleware:
            - `"whitenoise.middleware.WhiteNoiseMiddleware",`
        - افزودن تنظیمات زیر:
            ```python
            STATIC_URL = '/static/'
            STATIC_ROOT = BASE_DIR / "staticfiles"
            STATICFILES_DIRS = [
                os.path.join(BASE_DIR, 'static'),
            ]

            STATICFILES_STORAGE = "whitenoise.storage.CompressedManifestStaticFilesStorage"
            ```
        - جمع‌آوری فایل‌های استاتیک:
            - `python manage.py collectstatic`

    - انتقال به مخزن GitHub

1. اتصال به سرور با SSH:
    - اجرای دستورات زیر:
        - `ssh username@server_ip_address`
        - وارد کردن رمز عبور
        &nbsp;

2. به‌روزرسانی و ارتقاء سرور:
    - `sudo apt-get update`
    - `sudo apt-get upgrade`
    &nbsp;

3. نصب Python و pip:
    - `sudo apt install python3`
    - `sudo apt install python3-pip`
    &nbsp;

4. ایجاد و فعال‌سازی محیط مجازی:
    - `virtualenv /opt/myproject`
    - `source /opt/myproject/bin/activate`
    &nbsp;

5. کلون کردن مخزن:
    - `cd /opt/myproject`
    - `mkdir myproject`
        - این ممکن است به نظر تکراری باشد، اما باعث می‌شود نام محیط مجازی و پروژه یکی باشد.
    - `cd myproject`
    - بررسی نصب بودن git:
        - `git status`
        - `sudo apt-get install git`
    - کلون کردن مخزن:
        - `git clone repo-url`
    - نصب نیازمندی‌ها:
        - `cd repo-name`
        - `pip install -r requirements.txt`
        - `pip install gunicorn`
    &nbsp;

6. پیکربندی Nginx:
    - نصب nginx:
        - `sudo apt install nginx`
    - ایجاد و ویرایش فایل پیکربندی:
        - `sudo nano /etc/nginx/sites-available/myproject`
    - وارد کردن محتویات زیر در فایل:
         ```nginx
        server {
            listen 80;
            server_name yourip;

            access_log /var/log/nginx/website-name.log;

            location /static/ {
                alias /opt/myproject/myproject/path-to-static-files/;
            }

            location / {
                proxy_pass http://127.0.0.1:8000;
                proxy_set_header X-Forwarded-Host $server_name;
                proxy_set_header X-Real-IP $remote_addr;
                add_header P3P 'CP="ALLDSP COR PSAa PSDa OURNOR ONL UNI COM NAV"';
            }
        }
         ```
    - ایجاد لینک نمادین در دایرکتوری sites-enabled:
        - `cd /etc/nginx/sites-enabled`
        - `sudo ln -s ../sites-available/myproject`

    - ویرایش فایل nginx.conf:
        - `/etc/nginx/nginx.conf`
        - لغو کامنت این خط:
            - `# server_names_hash_bucket_size 64;`

    - راه‌اندازی مجدد nginx:
        - `sudo service nginx restart`
    
    &nbsp;

7. تنظیم فایروال:

    - قبل از آزمایش nginx، فایروال باید تنظیم شود تا به این سرویس دسترسی داشته باشد. nginx به طور خودکار به عنوان یک سرویس در ufw ثبت می‌شود و تنظیم آن ساده است.

    - `sudo apt-get install ufw`
    - `sudo ufw allow 8000`

    - راه‌اندازی مجدد nginx:
        - `sudo service nginx restart`
    &nbsp;

8. آزمایش پیکربندی:

    - اجرای gunicorn:
        - `cd /opt/myproject/myproject/repo-name`
        - `gunicorn --bind 0.0.0.0:8000 project_name.wsgi`
    - بازدید از پروژه روی آدرس IP سرور و پورت 8000:
        - `yourServerIp:8000`

    &nbsp;

9. اتصال دامنه:
    - ورود به ثبت‌کننده دامنه و تنظیمات DNS دامنه مشخص شده

    - افزودن رکورد A با نام @ و اشاره به آدرس IP سرور

    - ذخیره تغییرات

    - ویرایش فایل پیکربندی nginx:
        - `sudo nano /etc/nginx/sites-available/myproject`

    - اعمال تغییرات زیر:

        ```nginx
        server {
            listen 80;
            server_name yourdomain.com www.yourdomain.com;

            access_log /var/log/nginx/website-name.log;

            location /static/ {
                alias /opt/myproject/myproject/path-to-static-files/;
            }

            location / {
                proxy_pass yourServerIp:8000;
                proxy_set_header X-Forwarded-Host $server_name;
                proxy_set_header X-Real-IP $remote_addr;
                add_header P3P 'CP="ALL DSP COR PSAa PSDa OUR NOR ONL UNI COM NAV"';
            }
        }
        ```
    - راه‌اندازی مجدد nginx:
        - `sudo service nginx restart`
        
    - منتظر ماندن برای انتشار تغییرات DNS و بررسی وب‌سایت روی دامنه متصل شده
    &nbsp;

10. نصب SSL بر روی VPS با Let's Encrypt (نیاز به دامنه دارد):

    - `sudo apt install certbot`
    - `sudo apt install certbot python3-certbot-nginx`
    - اجرای دستور زیر و دنبال کردن مراحل:
        - `sudo certbot --nginx -d your_domain.com`
    - بررسی پیکربندی nginx:
        - `sudo nginx -t`
    - بارگذاری مجدد nginx:
        - `sudo systemctl reload nginx`
    - اجرای gunicorn:
        - `gunicorn --bind 0.0.0.0:8000 project_name.wsgi`

- اجرای gunicorn در پس‌زمینه:
    - `nohup gunicorn --bind 0.0.0.0:8000 project_name.wsgi &`
    &nbsp;

- متوقف کردن gunicorn در حال اجرا در پس‌زمینه:
    - `pkill gunicorn`
    &nbsp;

- اعمال تغییرات

# منابع

## مقالات:
</div>

 - https://www.digitalocean.com/community/tutorials/how-to-deploy-a-local-django-app-to-a-vps

- https://www.digitalocean.com/community/tutorials/how-to-set-up-django-with-postgres-nginx-and-gunicorn-on-ubuntu-22-04

- https://stackoverflow.com/questions/37339383/nginx-gunicorn-django-not-working
