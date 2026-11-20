<div dir="rtl">

# Python Backend Q&A

مجموعه‌ای فارسی از **سؤال‌ها، مسائل و راه‌حل‌های کاربردی در حوزه توسعه Backend با Python**.

این پروژه با هدف جمع‌آوری و مستندسازی پرسش‌هایی ایجاد شده است که برنامه‌نویسان Backend در مسیر یادگیری، توسعه پروژه‌های واقعی و کار با تکنولوژی‌های مختلف با آن‌ها مواجه می‌شوند.

تمرکز فعلی پروژه روی Python، Django و Django REST Framework است و در ادامه موضوعات مرتبط با FastAPI و سایر ابزارها و تکنولوژی‌های Backend نیز به پروژه اضافه خواهند شد.

---

## 🎯 هدف پروژه

هدف اصلی این Repository ایجاد یک منبع فارسی و قابل توسعه برای **یادگیری، حل مسئله و به‌اشتراک‌گذاری تجربه‌های Backend** است.

این پروژه صرفاً یک مجموعه آموزش تئوری نیست.

تمرکز اصلی روی سؤال‌ها و مسئله‌هایی است که یک Backend Developer ممکن است در مسیر یادگیری یا هنگام کار روی پروژه‌های واقعی با آن‌ها مواجه شود.

موضوعاتی مانند:

* طراحی API
* طراحی Database
* Authentication
* Authorization
* Security
* Testing
* Performance
* Deployment
* معماری و ساختار پروژه

در این مجموعه مورد بررسی قرار می‌گیرند.

این پروژه می‌تواند برای موارد زیر مفید باشد:

* برنامه‌نویسان تازه‌کار
* Python Developers
* Django Developers
* Backend Developers
* افرادی که در حال یادگیری Django و DRF هستند
* افرادی که قصد یادگیری FastAPI را دارند
* افرادی که برای مصاحبه‌های فنی آماده می‌شوند
* توسعه‌دهندگانی که با یک مسئله واقعی مواجه شده‌اند

---

## 🗂️ ساختار پروژه

تمام سؤال‌ها و پاسخ‌ها در قالب فایل‌های Markdown با پسوند `.md` نگهداری می‌شوند.

ساختار فعلی پروژه:

<div dir="ltr">

```text
python-backend-qna-fa/
│
├── README.md
│
└── questions/
    │
    ├── python/
    ├── django/
    ├── drf/
    ├── fastapi/
    ├── authentication/
    ├── authorization/
    ├── database/
    ├── postgresql/
    ├── redis/
    ├── celery/
    ├── security/
    ├── testing/
    ├── docker/
    ├── deployment/
    └── architecture/
```

</div>


💡 راهنمای آمادگی برای مصاحبه

برای کمک به افرادی که در مسیر آماده شدن برای مصاحبه‌های Backend هستند، بخشی از تجربیات و نکات مرتبط با مصاحبه‌های فنی نیز در این Repository مستند خواهد شد.

<p align="center"> <img src="questions/images/interview-helps.jpg" alt="راهنمای آمادگی برای مصاحبه Backend" width="800"> </p>

---

## 🐍 Python

سؤال‌ها و مسائل مربوط به زبان Python و مباحث مورد نیاز برای توسعه Backend.

---

## 🌱 Django

موضوعات مربوط به Django، از جمله:

* Models
* ORM
* QuerySet
* Migrations
* Views
* Middleware
* Signals
* Custom User Model
* Management Commands
* Django Security

---

## 🔌 Django REST Framework

موضوعات مرتبط با توسعه API با استفاده از Django REST Framework، از جمله:

* Serializers
* Views
* ViewSets
* Routers
* Permissions
* Authentication
* Validation
* Pagination
* Filtering
* Throttling
* API Design

---

## ⚡ FastAPI

موضوعات مربوط به FastAPI، از جمله:

* ASGI
* Event Loop
* Routing
* Pydantic
* Request & Response
* Dependency Injection
* Middleware
* Authentication
* Background Tasks
* SQLAlchemy
* Lifespan
* Production Deployment

---

## 🔐 Authentication

موضوعات مرتبط با احراز هویت و مدیریت Session و Token، از جمله:

* Authentication
* JWT
* Access Token
* Refresh Token
* Token Rotation
* Token Blacklisting
* Cookie Security
* Session Management
* CSRF

---

## 🛡️ Authorization

موضوعات مربوط به مدیریت دسترسی کاربران، از جمله:

* Permissions
* RBAC
* Role Management
* Access Control
* Object-level Permissions

---

## 🗄️ Database

موضوعات مرتبط با Database، از جمله:

* ACID
* Normalization
* Primary Keys
* Indexing
* Transactions
* Isolation Levels
* Concurrency Control
* Locks & Deadlocks
* Query Optimization
* Connection Pooling
* Partitioning & Sharding
* Migrations

---

## 🐘 PostgreSQL

موضوعات مرتبط با PostgreSQL، از جمله:

* Database Design
* Relationships
* Indexing
* Constraints
* Transactions
* Query Optimization
* Data Integrity

---

## ⚡ Redis و Celery

موضوعات مرتبط با Redis و Celery، از جمله:

* Caching
* Background Tasks
* Message Processing
* Task Queues
* Performance

---

## 🔒 Security

مسائل امنیتی مرتبط با Backend، API و سرویس‌های تحت وب.

هدف این بخش بررسی مشکلات امنیتی و روش‌های صحیح پیاده‌سازی قابلیت‌های Backend است.

---

## 🧪 Testing

موضوعات مرتبط با تست نرم‌افزار، از جمله:

* Unit Testing
* Integration Testing
* API Testing
* Pytest
* Django Testing
* Authentication Testing
* Performance Testing

---

## 🐳 Docker و Deployment

موضوعات مرتبط با اجرای پروژه‌های Backend و آماده‌سازی آن‌ها برای محیط Production، از جمله:

* Docker
* Docker Compose
* Nginx
* Gunicorn
* Production Configuration
* Deployment

---

## 🏗️ Architecture

موضوعات مرتبط با طراحی و معماری Backend، از جمله:

* Project Structure
* Separation of Concerns
* Service Layer
* Business Logic
* Scalability
* Maintainability
* API Architecture

---

## 👥 Community

بخش مهمی از این پروژه حاصل **مشارکت و همراهی اعضای جامعه برنامه‌نویسی** است.

بسیاری از سؤال‌ها و موضوعات این Repository توسط دوستان و اعضای **Community گروه Django مهندس علی بیگدلی** مطرح و بررسی می‌شوند.

پاسخ‌ها نیز با مشارکت اعضای مختلف جامعه شکل می‌گیرند و ممکن است در طول زمان با تجربه‌ها و دیدگاه‌های جدید تکمیل شوند.

هدف از جمع‌آوری این مطالب، کمک به یکدیگر، انتقال تجربه و ایجاد یک منبع مفید برای جامعه فارسی‌زبان برنامه‌نویسی است.

در این مسیر ممکن است یک سؤال توسط یک نفر مطرح شود، فرد دیگری تجربه یا راه‌حل خود را به اشتراک بگذارد و شخص دیگری پاسخ را کامل‌تر یا اصلاح کند.

**همین مشارکت و تبادل تجربه، بخش اصلی شکل‌گیری این مجموعه است.**

---

## 🤝 همکاری و مشارکت

این پروژه با هدف **کمک و همکاری بین برنامه‌نویسان Python و Backend** ایجاد شده است.

اگر سؤال، تجربه، راه‌حل یا نکته‌ای دارید که می‌تواند برای سایر توسعه‌دهندگان مفید باشد، می‌توانید در توسعه این مجموعه مشارکت کنید.

مشارکت می‌تواند شامل موارد زیر باشد:

* مطرح کردن سؤال‌های جدید
* ارائه پاسخ برای سؤال‌ها
* تکمیل یا اصلاح پاسخ‌های موجود
* اضافه کردن تجربه‌های واقعی
* پیشنهاد موضوعات جدید
* گزارش اشتباهات
* بهبود مستندات
* ارائه راه‌حل‌های بهتر

هدف این است که دانش و تجربه‌های پراکنده در زمینه Python Backend Development به یک مجموعه منظم، قابل استفاده و قابل توسعه تبدیل شود.

---

## 🙏 تشکر

در اینجا لازم است از **مهندس علی بیگدلی** و تمام دوستان و اعضای Community که در مطرح کردن سؤال‌ها، ارائه پاسخ‌ها، انتقال تجربه و کمک به سایر برنامه‌نویسان همراه ما هستند، صمیمانه تشکر کنیم.

این مجموعه با مشارکت و همراهی اعضای جامعه شکل گرفته است و ارزش اصلی آن نیز در همین همکاری و اشتراک تجربه‌هاست.

از تمام دوستانی که با مطرح کردن یک سؤال، پاسخ دادن به سؤال دیگران یا حتی به اشتراک گذاشتن یک تجربه کوچک به یادگیری دیگران کمک می‌کنند، سپاسگزاریم.

امیدواریم این Repository بتواند بخشی از دانش و تجربه جامعه برنامه‌نویسی را مستند کند و در ادامه با کمک همه اعضای Community کامل‌تر و مفیدتر شود.

---

## 👨‍💻 Contributors

این پروژه با مشارکت جامعه Python و Backend توسعه پیدا می‌کند.

از تمام افرادی که با اضافه کردن سؤال، پاسخ، تجربه، اصلاح محتوا یا بهبود مستندات در توسعه این Repository مشارکت می‌کنند، سپاسگزاریم.

<!-- CONTRIBUTORS:START -->

### 🙏 دوستان و مشارکت‌کنندگان

از تمام افرادی که با سؤال، پاسخ، تجربه، اصلاح محتوا یا مشارکت فنی به توسعه این پروژه کمک می‌کنند، صمیمانه سپاسگزاریم.

<p align="center">
<a href="https://github.com/moeinrezai"><img src="https://avatars.githubusercontent.com/u/164065593?v=4" width="80" alt="moeinrezai"><br><sub><b>moeinrezai</b></sub></a>
<a href="https://github.com/hamres"><img src="https://avatars.githubusercontent.com/u/314216067?v=4" width="80" alt="hamres"><br><sub><b>hamres</b></sub></a>
</p>

<!-- CONTRIBUTORS:END -->

اگر تجربه یا محتوای مفیدی دارید، می‌توانید با **Fork کردن Repository** و ارسال **Pull Request** در توسعه این مجموعه مشارکت کنید.


## 🌱 هدف بلندمدت

هدف بلندمدت این پروژه ایجاد یک **Knowledge Base فارسی برای Python Backend Development** است.

با گسترش پروژه، موضوعات و تکنولوژی‌های بیشتری اضافه خواهند شد و مجموعه‌ای از مسائل واقعی Backend در یک محل جمع‌آوری و مستند خواهند شد.

تمرکز این پروژه صرفاً روی یک Framework نیست.

هدف اصلی ایجاد بستری برای:

**یادگیری → حل مسئله → انتقال تجربه → همکاری**

در جامعه Python و Backend فارسی‌زبان است.

---

## 📌 وضعیت پروژه

این پروژه در حال توسعه است و مطالب جدید به مرور به آن اضافه خواهند شد.

تمرکز اولیه روی موارد زیر است:

<div dir="ltr">

```text
Python
    ↓
Django
    ↓
Django REST Framework
```

</div>

و در ادامه موضوعاتی مانند:

<div dir="ltr">

```text
FastAPI
Database
Security
Testing
Infrastructure
Architecture
```

</div>

نیز به پروژه اضافه خواهند شد.

---

## ❤️ با هم یاد بگیریم

اگر این پروژه برای شما مفید است، می‌توانید با مشارکت در آن به رشد جامعه برنامه‌نویسی کمک کنید.

گاهی یک سؤال ساده که امروز برای یک نفر مطرح می‌شود، می‌تواند فردا مشکل چندین برنامه‌نویس دیگر را حل کند.

**هدف این پروژه یادگیری در کنار یکدیگر، انتقال تجربه و کمک به رشد جامعه Python و Backend فارسی‌زبان است.**

</div>
