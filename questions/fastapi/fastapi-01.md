<div dir="rtl">

# ۳۰ سؤال تخصصی `FastAPI` برای مصاحبه `Backend`

این فایل شامل ۳۰ سؤال سطح بالا و کاربردی در مصاحبه‌های شغلی `Backend` با تمرکز بر فریم‌ورک `FastAPI`، معماری ناهمگام (Async)، اعتبارسنجی با `Pydantic V2`، سیستم تزریق وابستگی (`Dependency Injection`)، کار با پایگاه داده و بهترین الگوهای آماده‌سازی برای محیط پروداکشن است.

این پرسش‌ها از دیدگاه ترکیبی **برنامه‌نویس ارشد Backend**، **کارشناس استخدام فنی (Technical Recruiter)** و **مدیر HR** تدوین شده‌اند تا علاوه بر مفاهیم عمیق مهندسی، معیارهای ارزیابی در جلسه مصاحبه و پرچم‌های قرمز (Red Flags) نیز مشخص باشند.

---

## ۱. معماری ناهمگام، ASGI و Event Loop

### ۱. تفاوت `WSGI` و `ASGI` چیست و FastAPI چگونه از آن بهره می‌برد؟

**پاسخ:**

`WSGI` (مانند استانداردی که Django و Flask سنتی استفاده می‌کنند) یک استاندارد همگام (Synchronous) است که به ازای هر درخواست ورودی، یک Thread یا Process را درگیر و مسدود (Block) می‌کند.

`ASGI` استاندارد ناهمگام و جانشین معاصر WSGI است که از ورودی/خروجی غیرمسدودکننده (Non-blocking I/O)، وب‌سوکت‌ها (`WebSockets`) و پروتکل‌های با ماندگاری طولانی مانند `HTTP/2` و `Server-Sent Events` پشتیبانی می‌کند.

فریم‌ورک `FastAPI` روی شالوده `Starlette` پیاده‌سازی شده که یک ASGI Framework سبک‌وزن است و توسط وب‌سرورهایی مانند `Uvicorn` اجرا می‌شود. این معماری به رویدادگردان (Event Loop) پایتون اجازه می‌دهد هزاران اتصال همزمان را بدون نیاز به ایجاد هزاران Thread مجزا مدیریت کند.

> **دیدگاه مصاحبه‌کننده و نکته HR:**
> مصاحبه‌کننده می‌خواهد بداند آیا کاندیدا واقعاً مفهوم رقابت برای منابع، I/O Concurrency و نحوه کار Web Server را می‌فهمد یا صرفاً عبارت «FastAPI سریع است» را تکرار می‌کند.
>
> **Green Flag:** اشاره به تفاوت سوئیچ پروسه‌ها، غیرمسدودکننده بودن سوکت‌ها در سیستم‌عامل و مقایسه هزینه حافظه Thread در WSGI نسبت به Coroutine در ASGI.

---

### ۲. در FastAPI چه زمانی باید مسیر را با `async def` و چه زمانی با `def` معمولی تعریف کنیم؟

**پاسخ:**

اگر مسیر شما عملیات I/O ناهمگام انجام می‌دهد (مانند کوئری به دیتابیس با کتابخانه‌های Async مثل `asyncpg` یا ارسال درخواست شبکه با `httpx`)، حتماً از `async def` و کلیدواژه `await` استفاده کنید.

اگر از کتابخانه‌های همگام و بلاک‌کننده (مانند `requests` یا درایورهای دیتابیس مسدودکننده مثل `psycopg2`) استفاده می‌کنید یا محاسبات سنگین پردازشی (CPU-bound) دارید، باید تابع را به‌صورت `def` معمولی بنویسید.

در صورت نوشتن `def`، خودِ FastAPI آن را به‌صورت خودکار در یک ThreadPool مجزا (`anyio.to_thread`) اجرا می‌کند تا از مسدود شدن Event Loop اصلی برنامه جلوگیری شود. اما اگر یک متد بلاک‌کننده را داخل `async def` قرار دهید، Event Loop برای همه کاربران قفل شده و سرور فلج می‌شود.

<div dir="ltr">

```python
# عملیات غیرمسدودکننده واقعی:
@app.get("/async-data")
async def get_async_data():
    async with httpx.AsyncClient() as client:
        res = await client.get("https://api.example.com/data")
    return res.json()

# اشتباه مرگبار در پروداکشن (مسدود کردن Event Loop):
@app.get("/bad-async")
async def bad_async():
    time.sleep(5)  # این خط کل سرور را برای ۵ ثانیه قفل می‌کند!
    return {"status": "blocked"}
```

</div>

> **دیدگاه مصاحبه‌کننده و نکته HR:**
> این سؤال یکی از مهم‌ترین فیلترهای تفکیک جونیور از سنیور است.
>
> **Red Flag:** داوطلبی که بگوید «من همه چیز را async def می‌نویسم چون کلاً سریع‌تر است». این نشان‌دهنده عدم درک Event Loop است.

---

### ۳. چرخه حیات برنامه (Lifespan Context Manager) چیست و چرا جایگزین رویدادهای قدیمی شد؟

**پاسخ:**

در نسخه‌های قبلی، از دکوراتورهای `@app.on_event("startup")` و `@app.on_event("shutdown")` استفاده می‌شد که اکنون منسوخ (Deprecated) شده‌اند.

روش مدرن و استاندارد، استفاده از قابلیت `lifespan` بر پایه Async Context Manager پایتون است.

<div dir="ltr">

```python
from contextlib import asynccontextmanager
from fastapi import FastAPI

@asynccontextmanager
async def lifespan(app: FastAPI):
    # کدهای مربوط به Startup (مثلاً ایجاد Connection Pool دیتابیس یا کش)
    db_pool = await create_db_pool()
    app.state.db_pool = db_pool
    print("Application initialized successfully")
    
    yield
    
    # کدهای مربوط به Shutdown (مثلاً بستن تمیز ارتباطات)
    await db_pool.close()
    print("Resources cleaned up")

app = FastAPI(lifespan=lifespan)
```

</div>

مزایای این روش:
۱. مدیریت متمرکز و خوانا در یک بلاک کد و تضمین اجرای بخش خروج حتی در صورت بروز خطا با `try...finally`.
۲. به اشتراک‌گذاری داده‌ها و وضعیت‌ها به شکل ایمن و تایپ‌شده در `app.state`.

---

### ۴. فریم‌ورک FastAPI چگونه کارهای متمرکز بر پردازنده (CPU-bound) را مدیریت می‌کند؟

**پاسخ:**

رویکرد `async/await` در پایتون برای عملیات متمرکز بر ورودی/خروجی (I/O-bound) مناسب است و به دلیل وجود قفل مفسر سراسری (GIL)، کارهای سنگین محاسباتی (مانند پردازش تصویر، رمزنگاری سنگین یا آموزش مدل ML) کل فرآیند را معطل می‌کنند.

برای رفع این مشکل در معماری FastAPI سه راهکار وجود دارد:
۱. اجرای توابع محاسباتی در یک `ProcessPoolExecutor` برای خروج از GIL پایتون.
۲. واگذاری وظیفه به صف‌های پردازشی مستقل در پس‌زمینه مانند `Celery` یا `ARQ` با ورکر‌های توزیع‌شده.
۳. استفاده از کامپایل درجا یا کتابخانه‌های سطح پایین بهینه‌شده به زبان C یا Rust.

<div dir="ltr">

```python
import asyncio
from concurrent.futures import ProcessPoolExecutor

executor = ProcessPoolExecutor()

def heavy_computation(data: list) -> int:
    return sum(x * x for x in data)

@app.post("/calculate")
async def calculate(numbers: list[int]):
    loop = asyncio.get_running_loop()
    result = await loop.run_in_executor(executor, heavy_computation, numbers)
    return {"result": result}
```

</div>

---

### ۵. معماری چندلایه زیرساخت FastAPI چیست و هر بخش چه وظیفه‌ای دارد؟

**پاسخ:**

FastAPI برخلاف فریم‌ورک‌های یکپارچه (Monolithic) مثل جنگو، از ترکیب چند کامپوننت پیشرفته و متمرکز ساخته شده است:

۱. **Uvicorn:** وب‌سرور سریع ASGI که پروتکل‌های شبکه را دریافت کرده و به اپلیکیشن تحویل می‌دهد.
۲. **Starlette:** فریم‌ورک هسته ASGI که مسیریابی (Routing)، درخواست و پاسخ (`Request`/`Response`)، WebSockets و مدیریت Middlewareها را به عهده دارد.
۳. **Pydantic:** موتور اعتبارسنجی داده‌ها، سریال‌سازی و مدل‌سازی اسکیماها بر اساس Type Hints پایتون.
۴. **OpenAPI / Swagger UI:** لایه مستندسازی خودکار و تعاملی APIها بر اساس استانداردهای جهانی.

---

## ۲. اعتبارسنجی داده و مدل‌سازی با Pydantic V2

### ۶. تفاوت‌های کلیدی Pydantic V1 و Pydantic V2 در چیست و چرا برای کارایی مهم است؟

**پاسخ:**

هسته اصلی اعتبارسنجی در `Pydantic V2` به زبان `Rust` بازنویسی شده است (`pydantic-core`) که باعث افزایش سرعت بین ۵ تا ۵۰ برابری در اعتبارسنجی و تبدیل داده‌ها شده است.

تغییرات کلیدی در سینتکس:
* استفاده از `@field_validator` به جای دکوراتور منسوخ `@validator`.
* استفاده از `@model_validator(mode='before'/'after')` به جای `@root_validator`.
* متدهای مدل به `model_dump()` و `model_dump_json()` تغییر یافتند (به جای `.dict()` و `.json()`).
* تنظیمات مدل درون کلاسی به جای `class Config`، از `model_config = ConfigDict(...)` بهره می‌برد.

<div dir="ltr">

```python
from pydantic import BaseModel, field_validator, ConfigDict

class UserRegisterSchema(BaseModel):
    model_config = ConfigDict(str_strip_whitespace=True)
    username: str
    age: int

    @field_validator("username")
    @classmethod
    def validate_username(cls, v: str) -> str:
        if len(v) < 3:
            raise ValueError("نام کاربری باید حداقل ۳ کاراکتر باشد")
        return v.lower()
```

</div>

> **دیدگاه مصاحبه‌کننده و نکته HR:**
> آگاهی داوطلب از مهاجرت به Pydantic V2 نشان‌دهنده به‌روز بودن دانش فنی و پیگیری نسخه‌های اخیر اکوسیستم پایتون است.

---

### ۷. تفاوت `BaseModel` در Pydantic با `dataclass` استاندارد پایتون چیست؟

**پاسخ:**

`dataclass` پیش‌فرض پایتون عمدتاً محفظه‌ای برای ذخیره‌سازی داده‌ها (Data Container) است و به‌صورت پیش‌فرض هیچ اعتبارسنجی و تطبیق نوع داده‌ای (Type Coercion) در زمان اجرا (Runtime) انجام نمی‌دهد؛ اگر مقداری با تایپ اشتباه پاس داده شود، بدون ارور پذیرفته می‌شود.

در نقطه مقابل، `BaseModel` در Pydantic به طور اختصاصی برای Parsing، اعتبارسنجی سخت‌گیرانه (Strict Validation) و تبدیل تایپ‌ها طراحی شده است. ورودی‌های نامعتبر را به ارورهای خوانا و استاندارد با کد وضعیت `422 Unprocessable Entity` تبدیل کرده و اسکیماهای استاندارد JSON Schema برای مستندات تولید می‌کند.

---

### ۸. چگونه از `Field` برای اعتبارسنجی دقیق و تولید مستندات غنی OpenAPI استفاده کنیم؟

**پاسخ:**

کلاس `Field` به ما اجازه می‌دهد محدودیت‌های اعتبارسنجی (مانند مقادیر مجاز، طول رشته، عبارات باقاعده Regex) و اطلاعات توصیفی مستندات را تعریف کنیم.

<div dir="ltr">

```python
from pydantic import BaseModel, Field

class ProductCreateSchema(BaseModel):
    title: str = Field(
        ...,
        min_length=3,
        max_length=120,
        description="نام کالا",
        examples=["لپ‌تاپ گیمینگ ایسوس"]
    )
    price: float = Field(
        ...,
        gt=0,
        description="قیمت کالا باید عددی مثبت باشد"
    )
    sku: str = Field(
        pattern=r"^[A-Z]{3}-\d{4}$",
        description="فرمت شناسه: ۳ حرف بزرگ خط تیره ۴ رقم"
    )
```

</div>

این اعتبارسنجی قبل از اینکه درخواست به لایه بیزینس و دیتابیس برسد، خطاهای ورودی را به کاربر بازمی‌گرداند و بار پردازشی سرور را کم می‌کند.

---

### ۹. نقش `response_model` چیست و چگونه از نشت داده‌های حساس (Data Leakage) جلوگیری می‌کند؟

**پاسخ:**

پارامتر `response_model` در دکوراتورهای مسیر مشخص می‌کند که خروجی ارسالی به کلاینت دقیقاً باید طبق چه قالبی فیلتر و اعتبارسنجی شود.

اگر یک شیء از پایگاه داده (مثلاً حاوی فیلدهای `password_hash` یا `internal_notes`) از تابع بازگردانده شود، تعریف `response_model` مناسب تضمین می‌کند که فقط فیلدهای مجاز در پاسخ JSON قرار می‌گیرند.

<div dir="ltr">

```python
class UserResponse(BaseModel):
    id: int
    email: str
    is_active: bool

@app.get("/users/{user_id}", response_model=UserResponse)
async def get_user(user_id: int):
    user_db = await fetch_user_from_db(user_id) 
    # حتی اگر user_db فیلد password_hash داشته باشد، به کاربر ارسال نخواهد شد
    return user_db
```

</div>

همچنین می‌توان با پارامترهایی مانند `response_model_exclude_unset=True` از ارسال فیلدهای اختیاری که کلاینت مقداردهی نکرده صرف‌نظر کرد.

---

### ۱۰. اعتبارسنجی بین چند فیلد (Cross-Field Validation) چگونه در Pydantic V2 پیاده‌سازی می‌شود؟

**پاسخ:**

با استفاده از `@model_validator(mode="after")` یا `mode="before"` می‌توان شرط‌هایی را ارزیابی کرد که به مقدار چند فیلد به‌صورت همزمان وابسته‌اند (مثلاً بررسی تطابق رمز عبور و تکرار آن، یا بررسی اینکه تاریخ پایان بعد از تاریخ شروع باشد).

<div dir="ltr">

```python
from pydantic import BaseModel, model_validator
from datetime import date

class EventBooking(BaseModel):
    start_date: date
    end_date: date
    password: str
    confirm_password: str

    @model_validator(mode="after")
    def verify_rules(self) -> "EventBooking":
        if self.end_date < self.start_date:
            raise ValueError("تاریخ پایان نمی‌تواند قبل از تاریخ شروع باشد")
        if self.password != self.confirm_password:
            raise ValueError("رمز عبور و تکرار آن یکسان نیستند")
        return self
```

</div>

---

## ۳. سیستم تزریق وابستگی (Dependency Injection - Depends)

### ۱۱. مکانیزم `Depends` در FastAPI چگونه کار می‌کند و مزیت معماری آن چیست؟

**پاسخ:**

سیستم Dependency Injection در FastAPI ابزاری برای رعایت اصل وارونگی کنترل (IoC) و تفکیک دغدغه‌ها (Separation of Concerns) است.

زمانی که در پارامتر یک تابع از `Depends(some_callable)` استفاده می‌شود:
۱. فریم‌ورک قبل از اجرای مسیر، تابع وابسته را صدا می‌زند.
۲. پارامترهای مورد نیاز آن تابع را نیز به شکل بازگشتی حل و تزریق می‌کند.
۳. نتیجه بازگشتی را به عنوان آرگومان به تابع اصلی پاس می‌دهد.

کاربردهای اصلی:
* دریافت نشست پایگاه داده (`db: AsyncSession = Depends(get_db)`)
* اعتبارسنجی توکن و احراز هویت کاربر جاری
* بررسی نقش‌ها و دسترسی‌های امنیتی
* اعمال فیلترها و صفحه‌بندی‌های مشترک میان چند مسیر

---

### ۱۲. تفاوت استفاده از `yield` با `return` در توابع وابسته (Dependencies) چیست؟

**پاسخ:**

اگر از `return` استفاده کنید، تابع وابسته اجرا شده و کار آن بلافاصله تمام می‌شود.

اما اگر از `yield` استفاده کنید، فریم‌ورک آن را مانند یک کانتکست منیجر مدیریت می‌کند:
* کدهای قبل از `yield` هنگام ورود درخواست اجرا می‌شوند.
* مقدار روبروی `yield` به مسیر تحویل داده می‌شود.
* پس از ارسال پاسخ به کلاینت (حتی در صورت وقوع خطا در مسیر)، کدهای پس از `yield` اجرا می‌شوند.

این قابلیت حیاتی‌ترین روش برای مدیریت ایمن چرخه‌ی باز و بسته شدن کانکشن‌های دیتابیس است:

<div dir="ltr">

```python
async def get_db_session():
    session = AsyncSessionLocal()
    try:
        yield session
    finally:
        await session.close()
```

</div>

---

### ۱۳. مفهوم وابستگی‌های زنجیره‌ای (Sub-dependencies) و پارامتر `use_cache` چیست؟

**پاسخ:**

یک Dependency خود می‌تواند به چندین Dependency دیگر وابسته باشد و FastAPI یک گراف جهت‌دار بدون دور (DAG) برای حل آن‌ها می‌سازد.

پارامتر پیش‌فرض `use_cache=True` در `Depends` تضمین می‌کند که اگر یک وابستگی مشترک (مانند `get_current_user` یا `get_db`) در چند نقطه از درخت فراخوانی یک درخواست نیاز باشد، فقط **یک بار** اجرا شده و خروجی آن کش می‌شود. اگر به هر دلیلی نیاز باشد در هر مرحله تابع از ابتدا مجدداً اجرا شود، باید `use_cache=False` تنظیم شود.

---

### ۱۴. تفاوت Global Dependencies، Router Dependencies و Path Dependencies چیست؟

**پاسخ:**

* **Path Dependency:** مستقیماً روی یک Endpoint خاص به عنوان آرگومان یا در پارامتر `dependencies=[Depends(...)]` تعریف می‌شود.
* **Router Dependency:** روی یک `APIRouter` اعمال می‌شود و تمام مسیرهای آن زیرشاخه را به صورت خودکار تحت پوشش قرار می‌دهد (مثلاً احراز هویت برای تمام روت‌های `/admin`).
* **Global Dependency:** در زمان تعریف `FastAPI(dependencies=[Depends(...)])` تعریف شده و بدون استثنا روی تک‌تک درخواست‌های کل سرور اجرا می‌شود (مانند بررسی سلامت عمومی سرور، بلاک کردن IPهای نامعتبر).

---

### ۱۵. چگونه در تست‌های نرم‌افزاری، وابستگی‌ها را با `app.dependency_overrides` ماک (Mock) کنیم؟

**پاسخ:**

یکی از درخشان‌ترین ویژگی‌های معماری FastAPI برای تست‌نویسی، دیکشنری `app.dependency_overrides` است که نیاز به استفاده از پچ‌های پیچیده پایتون را رفع می‌کند.

<div dir="ltr">

```python
import pytest
from httpx import AsyncClient
from main import app, get_db

async def override_get_db():
    # بازگرداندن سشن تست درون‌حافظه‌ای SQLite
    yield test_db_session

def test_user_list():
    app.dependency_overrides[get_db] = override_get_db
    
    with TestClient(app) as client:
        response = client.get("/users")
        assert response.status_code == 200
        
    app.dependency_overrides.clear()  # پاک‌سازی برای تست‌های بعدی
```

</div>

> **دیدگاه مصاحبه‌کننده و نکته HR:**
> تسلط بر `dependency_overrides` نشان می‌دهد فرد تجربه عملی نوشتن Integration Testهای تمیز و بدون دردسر را در پروژه‌های واقعی داشته است.

---

## ۴. احراز هویت، امنیت و مدیریت خطاها

### ۱۶. استاندارد `OAuth2PasswordBearer` در FastAPI چگونه جریان احراز هویت را هدایت می‌کند؟

**پاسخ:**

کلاس `OAuth2PasswordBearer(tokenUrl="token")` در واقع یک Dependency است که:
۱. در هدر درخواست ورودی به دنبال هدر `Authorization: Bearer <token>` می‌گردد.
۲. اگر توکن وجود نداشت، خطای `401 Unauthorized` برمی‌گرداند.
۳. مقدار خام رشته توکن را برمی‌گرداند تا در تابع اعتبارسنجی JWT بررسی و دیکود شود.
۴. در مستندات Swagger UI دکمه دایرکت Authorize را فعال می‌کند تا تسترها بتوانند لاگین کنند.

---

### ۱۷. چه تفاوتی میان Middleware و Dependency Injection وجود دارد و چه زمانی از کدام استفاده می‌کنیم؟

**پاسخ:**

| ویژگی | Middleware | Dependency Injection |
|---|---|---|
| **سطح دسترسی** | پایین‌ترین سطح در لایه ASGI قبل از مسیریابی | در سطح فراخوانی روت‌ها و پارامترها |
| **دسترسی به مقادیر مسیر** | فقط به هدرها و خام URL دسترسی دارد | دسترسی کامل به پارامترهای تایپ‌شده، مدل‌های Pydantic و State |
| **کاربرد معمول** | مانیتورینگ زمان کل درخواست، هندل کردن هدرهای امنیتی، CORS | احراز هویت، کنترل دسترسی، واکشی نشست‌های پایگاه داده |
| **سربار کارایی** | روی تمام درخواست‌ها بدون استثنا اعمال می‌شود | فقط روی روت‌هایی که اعلام کرده‌اند اجرا می‌شود |

قانون طلایی: اگر کاری به تحلیل پارامترها، احراز هویت کاربر یا ارتباط دیتابیس نیاز دارد، باید از **Dependency** استفاده شود. اگر کاری مربوط به کل ارتباط HTTP خام (مثل افزودن زمان پاسخ `X-Process-Time`) است، از **Middleware** استفاده کنید.

---

### ۱۸. مدیریت استثناها (Global Exception Handling) چگونه پیاده‌سازی می‌شود؟

**پاسخ:**

با استفاده از دکوراتور `@app.exception_handler` می‌توان هر ارور اختصاصی یا پیش‌فرض را رهگیری کرد و خروجی یکدست با فرمت شرکتی تولید نمود.

<div dir="ltr">

```python
from fastapi import Request
from fastapi.responses import JSONResponse
from fastapi.exceptions import RequestValidationError

class BusinessLogicException(Exception):
    def __init__(self, message: str, code: str):
        self.message = message
        self.code = code

@app.exception_handler(BusinessLogicException)
async def business_exception_handler(request: Request, exc: BusinessLogicException):
    return JSONResponse(
        status_code=400,
        content={"success": False, "error_code": exc.code, "message": exc.message}
    )
```

</div>

---

### ۱۹. چه تنظیماتی برای فعال‌سازی ایمن CORS و TrustedHost در محیط Production ضروری است؟

**پاسخ:**

در محیط لوکال ممکن است تنظیم `allow_origins=["*"]` استفاده شود، اما در پروداکشن این کار یک آسیب‌پذیری امنیتی بزرگ است.

باید مبداهای مجاز به دقت لیست شوند، کوکی‌ها و توکن‌های اعتبارسنجی فقط برای دامنه‌های تعیین‌شده پذیرفته شوند و افزونه `TrustedHostMiddleware` برای جلوگیری از حملات HTTP Host Header Poisoning فعال گردد:

<div dir="ltr">

```python
from fastapi.middleware.cors import CORSMiddleware
from fastapi.middleware.trustedhost import TrustedHostMiddleware

origins = [
    "https://example.com",
    "https://admin.example.com",
]

app.add_middleware(
    CORSMiddleware,
    allow_origins=origins,
    allow_credentials=True,
    allow_methods=["GET", "POST", "PUT", "DELETE"],
    allow_headers=["Authorization", "Content-Type"],
)

app.add_middleware(
    TrustedHostMiddleware, 
    allowed_hosts=["example.com", "*.example.com"]
)
```

</div>

---

### ۲۰. راهکار اعمال محدودیت نرخ درخواست (Rate Limiting) در فریم‌ورک FastAPI چیست؟

**پاسخ:**

بهترین رویکرد برای سیستم‌های توزیع‌شده، استفاده از الگوی Token Bucket به کمک یک کانتینر اشتراکی `Redis` و کتابخانه‌هایی مانند `slowapi` است.

اعمال Rate Limit به‌صورت محلی در حافظه RAM سرور اشتباه است، زیرا با بالا آمدن چند Worker یا کانتینرهای مختلف در Docker / Kubernetes، شمارنده‌ها با هم هماهنگ نخواهند بود و کاربر می‌تواند به تعداد ورکرها سهمیه اضافی بگیرد.

---

## ۵. ارتباط با پایگاه داده و SQLAlchemy 2.0 Async

### ۲۱. نحوه راه‌اندازی سشن‌های ناهمگام دیتابیس در SQLAlchemy 2.0 با `asyncpg` چگونه است؟

**پاسخ:**

در معماری مدرن FastAPI از قابلیت Async Engine در SQLAlchemy 2.0 استفاده می‌شود:

<div dir="ltr">

```python
from sqlalchemy.ext.asyncio import create_async_engine, async_sessionmaker, AsyncSession

DATABASE_URL = "postgresql+asyncpg://user:pass@localhost:5432/dbname"

engine = create_async_engine(
    DATABASE_URL,
    echo=False,
    pool_size=20,
    max_overflow=10,
    pool_pre_ping=True
)

AsyncSessionLocal = async_sessionmaker(
    bind=engine,
    class_=AsyncSession,
    expire_on_commit=False,
    autoflush=False
)

async def get_db() -> AsyncSession:
    async with AsyncSessionLocal() as session:
        yield session
```

</div>

نکته کلیدی: پارامتر `expire_on_commit=False` حیاتی است تا بعد از ذخیره تراکنش، ویژگی‌های اشیاء در حالت غیرهمگام قطع نشده و خطای MissingGreenlet رخ ندهد.

---

### ۲۲. چالش N+1 در SQLAlchemy Async چیست و چگونه حل می‌شود؟

**پاسخ:**

در حالت ناهمگام، بارگذاری تنبل (Lazy Loading) خودکار پشتیبانی نمی‌شود، زیرا ارتباطات I/O نمی‌توانند در زمان دسترسی به خصوصیت (Attribute Access) صدا زده شوند و برنامه با خطای `sqlalchemy.exc.MissingGreenlet` مواجه می‌شود.

برای حل این چالش و جلوگیری از صدور N کوئری اضافی برای دریافت رابطه‌ها، از بارگذاری حریصانه (Eager Loading) صریح استفاده می‌شود:
* **`selectinload`:** برای روابط یک‌به‌چند (One-to-Many) و چندبه‌چند، که یک کوئری با عبارت `WHERE id IN (...)` برای فرزندان صادر می‌کند.
* **`joinedload`:** برای روابط چندبه‌یک (Many-to-One / Foreign Key) که از دستور `LEFT OUTER JOIN` در کوئری اصلی استفاده می‌کند.

<div dir="ltr">

```python
from sqlalchemy.orm import selectinload
from sqlalchemy import select

stmt = select(Author).options(selectinload(Author.books))
result = await session.execute(stmt)
authors = result.scalars().all()
```

</div>

---

### ۲۳. چه فاکتورهایی در تنظیم ابعاد Connection Pool برای جلوگیری از Pool Exhaustion مؤثر است؟

**پاسخ:**

در فریم‌ورک‌های Async، درخواست‌ها بسیار سریع پردازش می‌شوند. اگر استخر اتصالات کوچک باشد، درخواست‌ها در صف مانده و خطای `TimeoutError: QueuePool limit of size X reached` رخ می‌دهد.

محاسبه اندازه استخر به فرمول زیر وابسته است:

$$\text{Total DB Connections} = \text{Workers} \times (\text{pool\_size} + \text{max\_overflow})$$

اگر ۴ ورکر Uvicorn داشته باشید و هر کدام استخر ۲۰ تایی باز کنند، دیتابیس باید ظرفیت حداقل ۸۰ کانکشن همزمان را داشته باشد. همچنین استفاده از `pool_pre_ping=True` تضمین می‌کند کانکشن‌های سوخته و قطع‌شده قبل از استفاده مجدداً احیا شوند.

---

### ۲۴. اجرای Migrationهای بدون وقفه (Zero-downtime) با Alembic در ساختار FastAPI چگونه مدیریت می‌شود؟

**پاسخ:**

فایل تنظیمات `env.py` در Alembic باید برای اجرای ناهمگام کانفیگ شود (`run_migrations_online` با متد `connect()` انجین همگام یا شبیه‌ساز ناهمگام).

در محیط‌های عملیاتی، مایگریشن‌ها نباید هنگام راه‌اندازی کانتینر وب اجرا شوند؛ بلکه در مرحله مستقلی از پایپ‌لاین CI/CD یا توسط یک Kubernetes Job مجزا اجرا می‌شوند تا در صورت وجود چندین Replica، تصادم یا ایجاد Lock روی دیتابیس پیش نیاید.

---

### ۲۵. تفاوت استفاده از SQLAlchemy در مقایسه با درایورهای خام (مثل `asyncpg`) در پروژه‌های پربار چیست؟

**پاسخ:**

* **کتابخانه خام `asyncpg`:** حداکثر کارایی و حداقل سربار CPU را به همراه دارد و بیشترین تعداد کوئری بر ثانیه (QPS) را ثبت می‌کند. عیب آن نوشتن رشته‌های خام SQL، نداشتن مکانیزم Unit of Work و سختی نگهداری کدهای پیچیده در مقیاس‌های بزرگ است.
* **کتابخانه SQLAlchemy 2.0:** سطح انتزاع عالی، جلوگیری خودکار از SQL Injection، مدیریت چرخه حیات تراکنش‌ها، ارتباطات تمیز میان مدل‌ها و ادغام بدون دردسر با ابزارهای تولید مایگریشن مانند Alembic.
* **نتیجه‌گیری کاربردی:** برای ۹۵٪ سناریوها SQLAlchemy 2.0 تعادل ایده‌آلی میان سرعت توسعه و پرفورمنس فراهم می‌کند؛ بخش‌های به شدت گلوگاه را می‌توان با متد `session.execute(text(...))` مستقیماً با SQL خام نوشت.

---

## ۶. عملکرد، وظایف پس‌زمینه و پروداکشن

### ۲۶. تفاوت `BackgroundTasks` داخلی با سیستم‌های توزیع‌شده مانند `Celery` چیست؟

**پاسخ:**

کلاس `BackgroundTasks` داخلی FastAPI تسک را در حافظه همان پروسه وب‌سرور و بعد از ارسال پاسخ اجرا می‌کند.

**محدودیت‌های BackgroundTasks:**
* وظایف در صورت ری‌استارت سرور از دست می‌روند (فاقد صف ماندگار/Persistence).
* برای کارهای سنگین، منابع CPU وب‌سرور را مصرف کرده و پاسخ‌دهی به سایر کاربران را کند می‌کند.
* ابزار مانیتورینگ متمرکز، تلاش مجدد خودکار (Retry) و گزارش وضعیت ندارد.

**چه زمانی از کدام استفاده کنیم؟**
* **`BackgroundTasks`:** کارهای بسیار سبک و بی‌اهمیت درون‌برنامه‌ای (مانند نوشتن یک لاگ ساده در فایل متنی).
* **`Celery` یا `ARQ`:** ارسال پیامک و ایمیل، ساخت فایل PDF، تراکنش‌های مالی، ارتباط با وب‌سرویس‌های ثالث و پردازش تصاویر.

---

### ۲۷. بهترین معماری برای استقرار کانتینری FastAPI با Uvicorn و Gunicorn در Production چیست؟

**پاسخ:**

در محیط کانتینری (Docker/Kubernetes)، بهترین الگو معمولاً یکی از دو حالت زیر است:

۱. **در محیط بدون داکر ارکستریتور:** اجرای `Gunicorn` به عنوان Process Manager همراه با ورکر کلاس `UvicornWorker`:

<div dir="ltr">

```bash
gunicorn main:app -w 4 -k uvicorn.workers.UvicornWorker --bind 0.0.0.0:8000
```

</div>

فرمول تعیین تعداد ورکرها:

$$\text{Workers} = (2 \times \text{CPU Cores}) + 1$$

۲. **در محیط Kubernetes:** توصیه مدرن این است که در هر پاد (Pod) یک ورکر تکی Uvicorn اجرا شود و مقیاس‌پذیری افقی (Horizontal Pod Autoscaling) بر عهده کوبرنتیز قرار گیرد تا ایزولاسیون حافظه و پایش سلامت دقیق‌تر باشد.

---

### ۲۸. چگونه دسترسی به مستندات Swagger UI و OpenAPI را در پروداکشن مسدود یا ایمن کنیم؟

**پاسخ:**

نمایش مستندات در محیط عملیاتی عمومی می‌تواند ریسک مهندسی معکوس و کشف آسیب‌پذیری توسط مهاجمان را بالا ببرد.

برای غیرفعال‌سازی در پروداکشن:

<div dir="ltr">

```python
import os

is_production = os.getenv("ENVIRONMENT") == "production"

app = FastAPI(
    docs_url=None if is_production else "/docs",
    redoc_url=None if is_production else "/redoc",
    openapi_url=None if is_production else "/openapi.json"
)
```

</div>

اگر دسترسی به مستندات برای توسعه‌دهندگان سازمان در پروداکشن نیاز است، نباید آن را کاملاً حذف کرد؛ بلکه باید با یک Dependency اختصاصی مبتنی بر `HTTPBasic` آن را رمزگذاری کرد.

---

### ۲۹. چگونه سیستم لاگ‌گیری ساختاریافته (Structured JSON Logging) با `request_id` را پیاده‌سازی کنیم؟

**پاسخ:**

در معماری میکروسرویس و پروداکشن، لاگ‌ها نباید متن ساده باشند؛ بلکه باید در قالب JSON و با یک شناسه یکتا به ازای هر درخواست (`trace_id` یا `request_id`) ثبت شوند تا در سیستم‌هایی مثل ELK یا Loki قابل ره‌گیری باشند:

<div dir="ltr">

```python
import uuid
import time
from fastapi import Request

@app.middleware("http")
async def logging_and_tracing_middleware(request: Request, call_next):
    request_id = request.headers.get("X-Request-ID", str(uuid.uuid4()))
    request.state.request_id = request_id
    
    start_time = time.perf_counter()
    response = await call_next(request)
    duration = time.perf_counter() - start_time
    
    response.headers["X-Request-ID"] = request_id
    
    # ثبت در سیستم لاگ به صورت Structured
    logger.info({
        "request_id": request_id,
        "method": request.method,
        "path": request.url.path,
        "status_code": response.status_code,
        "duration_ms": round(duration * 1000, 2)
    })
    
    return response
```

</div>

---

### ۳۰. شایع‌ترین نشانه‌های منفی (Red Flags) که یک داوطلب FastAPI را در مصاحبه رد می‌کند چیست؟

**پاسخ:**

از نگاه یک تیم فنی و مدیر استخدام، موارد زیر نشان‌دهنده عدم تسلط عمیق و کارایی نامناسب برای پروژه‌های پروداکشن است:

۱. **استفاده از کتابخانه‌های Blocking داخل `async def`:** مانند فراخوانی `requests.get` به جای `httpx` یا اجرای کوئری‌های ORM همگام جنگو در بدنه توابع ناهمگام.
۲. **تعریف متغیرهای سراسری (Global State) برای داده‌های درخواست:** عدم درک این مسئله که متغیرهای گلوبال در فریم‌ورک‌های مالتی‌ثرِد و Async بین درخواست‌های کاربران مختلف به اشتراک گذاشته شده و نشت داده ایجاد می‌کنند.
۳. **عدم استفاده از `response_model`:** ارسال مستقیم مدل‌های دیتابیس به کلاینت بدون فیلتر کردن فیلدهای حساس.
۴. **تنظیم `allow_origins=["*"]` در محیط پروداکشن.**
۵. **عدم مدیریت Connection Pool:** باز کردن اتصال مستقیم به دیتابیس در هر درخواست بدون استفاده از استخر مشترک یا فراموشی در بستن سشن‌ها در بلاک `finally`.
۶. **تصور اینکه FastAPI همیشه و در هر شرایطی بهتر از Django است:** فرد حرفه‌ای می‌داند که هر ابزار برای سناریوی خاصی است و FastAPI برای میکروسرویس‌ها، سرویس‌های Real-time و APIهای عملکرد بالا بی‌نظیر است، در حالی که جنگو برای برنامه‌های محتوامحور با پنل مدیریت داخلی آماده بسیار مناسب‌تر است.

---

## جمع‌بندی نکات کلیدی

| حوزه | تکنولوژی / ابزار | توصیه پروداکشن |
|---|---|---|
| **نوع توابع** | `async def` در برابر `def` | در I/O ناهمگام از `async def` و در عملیات مسدودکننده حتماً از `def` استفاده شود. |
| **اعتبارسنجی** | `Pydantic V2` | استفاده از `@field_validator` و `@model_validator` با موتور بهینه‌شده Rust. |
| **داده‌های خروجی** | `response_model` | تعریف اجباری مدل پاسخ برای جلوگیری قطعی از نشت داده‌های حساس دیتابیس. |
| **مدیریت منابع** | `yield` در `Depends` | تضمین بستن کانکشن‌ها و سشن‌های پایگاه داده در پایان چرخه درخواست. |
| **اتصال دیتابیس** | `SQLAlchemy 2.0 Async` | استفاده از `AsyncSession` همراه با `selectinload` برای رفع خطای N+1. |
| **پردازش پس‌زمینه** | `BackgroundTasks` vs `Celery` | کارهای سنگین و مأموریت‌های حساس فقط با Message Broker و Worker مجزا. |
| **امنیت** | `CORS` و `OpenAPI` | غیرفعال کردن مستندات عمومی و مشخص کردن دقیق Originهای مجاز در Production. |

</div>
