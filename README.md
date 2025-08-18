<div dir="rtl">

# استقرار پروژه با Docker و Docker Compose

این پروژه برای اجرا از **دو سرویس** استفاده می‌کند:  
۱. وب‌سرور جنگو (Django)  
۲. پایگاه‌داده PostgreSQL  

همه‌چیز داخل یک فایل `docker-compose.yml` تعریف شده و تنظیمات جنگو هم طوری به‌روزرسانی شده که از متغیرهای محیطی (`env`) برای اتصال به پایگاه‌داده استفاده کند.

---

## مراحل استقرار

### ۱. ساخت image و بالا آوردن container
```bash
docker-compose up -d --build
```
- سرویس پایگاه‌داده روی پورت ۵۴۳۲ بالا می‌آید.  
- وب‌سرور جنگو روی پورت ۸۰۰۰ در دسترس است:  
  [http://localhost:8000](http://localhost:8000)

### ۲. اجرای مایگریشن‌ها به‌صورت خودکار
در فایل `docker-compose.yml` تنظیم کردیم که هر بار کانتینر جنگو بالا می‌آید، ابتدا:
```bash
python manage.py migrate
```
اجرا شود.  
این دستور اگر مایگریشنی وجود نداشته باشد، کاری نمی‌کند و امن است.

### ۳. ساخت کاربر ادمین
برای ساختن superuser در محیط توسعه، کافیست:
```bash
docker-compose exec web python manage.py createsuperuser
```
این دستور داخل همان کانتینر جنگو اجرا می‌شود و نیازی به تغییر Dockerfile ندارد.
یا کافیست داخل shel کانتینر شویم و با اجرای دستور:

```bash
python manage.py createsuperuser
```
در پوشه پروژه اینکار را انجام دهیم.

---

## وظایف فایل‌های مختلف

### **Dockerfile**
- تصویر پایه پایتون را مشخص می‌کند.  
- وابستگی‌ها را از روی `requirements.txt` نصب می‌کند.  
- سورس کد پروژه را به کانتینر منتقل می‌کند.  
- پورت ۸۰۰۰ را باز می‌کند و وب‌سرور را اجرا می‌کند.

### **docker-compose.yml**
- دو سرویس تعریف می‌کند:  
  - **db**: پایگاه‌داده PostgreSQL به‌همراه نام دیتابیس، یوزر و پسورد مشخص و ذخیره‌سازی داده‌ها روی volume.  
  - **web**: سرویس جنگو که به Postgres وصل می‌شود، مایگریشن را اجرا می‌کند و وب‌سرور را روی پورت ۸۰۰۰ بالا می‌آورد.  
- متغیرهای محیطی (env) را برای اتصال به پایگاه‌داده تنظیم می‌کند.  
- ترتیب اجرای سرویس‌ها را با `depends_on` مدیریت می‌کند.

### **settings.py**
- به‌روزرسانی شده تا اطلاعات دیتابیس را از **متغیرهای محیطی** بخواند و دیگر به‌صورت ثابت نباشد.


## ارسال درخواست‌ها به وب سرور
برای این بخش یک فایل `requests.ipynb` نوشتیم که صرفا درخواست ها را با استفاده از کتابخانه `requests` ارسال می‌کند و خروجی را ذخیره و چاپ می‌کند.
===== نتایج =====
</div>
<div>
```
create_user
Request:
{
  "method": "POST",
  "url": "http://localhost:8000/users/create/",
  "payload": {
    "username": "user1",
    "password": "1234"
  }
}
Response:
{
  "status": 200,
  "body": {
    "id": 2,
    "username": "user1"
  }
}
--------------------------------------------------
login
Request:
{
  "method": "POST",
  "url": "http://localhost:8000/users/login/",
  "payload": {
    "username": "user1",
    "password": "1234"
  }
}
Response:
{
  "status": 200,
  "body": {
    "message": "Login successful"
  }
}
--------------------------------------------------
create_note_1
Request:
{
  "method": "POST",
  "url": "http://localhost:8000/notes/create/",
  "payload": {
    "title": "title1",
    "body": "body1"
  }
}
Response:
{
  "status": 200,
  "body": {
    "id": 1,
    "title": "title1",
    "body": "body1",
    "create_time": "2025-08-18T11:34:55.316Z"
  }
}
--------------------------------------------------
create_note_2
Request:
{
  "method": "POST",
  "url": "http://localhost:8000/notes/create/",
  "payload": {
    "title": "title2",
    "body": "body2"
  }
}
Response:
{
  "status": 200,
  "body": {
    "id": 2,
    "title": "title2",
    "body": "body2",
    "create_time": "2025-08-18T11:34:55.375Z"
  }
}
--------------------------------------------------
list_notes
Request:
{
  "method": "GET",
  "url": "http://localhost:8000/notes/",
  "payload": null
}
Response:
{
  "status": 200,
  "body": [
    {
      "id": 1,
      "title": "title1",
      "body": "body1",
      "create_time": "2025-08-18T11:34:55.316Z"
    },
    {
      "id": 2,
      "title": "title2",
      "body": "body2",
      "create_time": "2025-08-18T11:34:55.375Z"
    }
  ]
}
--------------------------------------------------
```
</div>


# Notes project

## Requirements
- Python3
- Postgres

## How to run

### Setup virtual environment

#### Create venv
```
python -m venv ./venv
```

#### Install requirements
```
python -m pip install -r requirements.txt
```

#### Activate venv
```
source ./venv/bin/activate
```

### Setup database
1. Create an instance of postgres database
2. Make migrations
    ```
    python manage.py makemigrations
    ```
3. Migrate
    ```
    python manage.py migrate
    ```

### Create an admin
```
python manage.py createsuperuser
```

## Important end-points
```
users/login/ --> login a user
users/me/ --> get information of logged-in user
users/create/ --> create a user
users/<id>/delete/ --> delete a user
notes/ --> list all notes of current user
notes/<id>/ --> get details of a note
notes/create/ --> create a note
notes/<id>/delete/ --> delete a note
```
