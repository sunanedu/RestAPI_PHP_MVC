# คู่มือสร้าง REST API ด้วย PHP ฉบับสมบูรณ์: จากเริ่มต้นสู่มือโปร

สวัสดีครับ! ยินดีต้อนรับสู่คู่มือฉบับสมบูรณ์ที่สุดในการสร้าง **REST API** ด้วย PHP ที่ผมตั้งใจรวบรวมและเรียบเรียงขึ้นมาใหม่ทั้งหมด เพื่อให้คุณมั่นใจได้ว่าทุกไฟล์ ทุกบรรทัดของโค้ด จะทำงานร่วมกันได้อย่างสมบูรณ์ และสามารถทดสอบได้จริงทุก Endpoint ครับ 🚀

ในคู่มือนี้ ผมจะพาคุณไปทีละขั้นตอนเพื่อสร้าง REST API สำหรับระบบ E-commerce ที่รองรับการทำงานพื้นฐานครบถ้วนแบบ **CRUD** (Create, Read, Update, Delete) โดยเราจะใช้เครื่องมือที่เป็นมาตรฐานอุตสาหกรรมอย่าง **PHP PDO** ในการเชื่อมต่อฐานข้อมูล, ใช้ **JWT (JSON Web Token)** เพื่อสร้างระบบยืนยันตัวตนที่ปลอดภัย, และวางโครงสร้างโปรเจกต์แบบ **MVC (Model-View-Controller)** ที่จะช่วยให้โค้ดของเราสะอาด เป็นระเบียบ และง่ายต่อการดูแลรักษาในระยะยาว

เราจะลงลึกไปถึงการทำ **Rate Limiting** เพื่อป้องกันการโจมตี, การจัดการ **Foreign Key** เพื่อรักษาความถูกต้องของข้อมูล, และการสร้าง **Router** ที่ชาญฉลาดเพื่อจัดการ Middleware ที่ซับซ้อน
ถ้าพร้อมแล้ว... ไปสร้าง API ที่ยอดเยี่ยมด้วยกันเลยครับ!

## ขั้นตอนที่ 1: วางแผนและออกแบบ (The Blueprint)

ก่อนจะลงมือเขียนโค้ด เรามาทำความเข้าใจภาพรวมของระบบที่เรากำลังจะสร้างกันก่อนครับ API ของเราจะใช้สำหรับจัดการข้อมูลในฐานข้อมูล ซึ่งประกอบไปด้วยตารางต่างๆ ดังนี้:

* **users**: ข้อมูลผู้ใช้ (username, email, password, role)
* **categories**: หมวดหมู่สินค้า
* **products**: สินค้า
* **orders**: คำสั่งซื้อ
* **order\_items**: รายการสินค้าในคำสั่งซื้อ
* **addresses**: ที่อยู่สำหรับจัดส่ง
* **payments**: ข้อมูลการชำระเงิน
* **reviews**: รีวิวสินค้าจากผู้ใช้
* **rate\_limits**: ตารางสำหรับจำกัดจำนวนการเรียก API

## ขั้นตอนที่ 2: เตรียมเครื่องมือและสภาพแวดล้อม (Setup Your Workshop)

เพื่อให้ API ของเราทำงานได้ เราต้องจำลองเครื่องของเราให้เป็น Server ก่อน โดยใช้โปรแกรม **XAMPP** ครับ

### 2.1 ติดตั้ง XAMPP

* หากยังไม่มี ให้ดาวน์โหลดและติดตั้ง **XAMPP** จาก [เว็บไซต์ทางการ](https://www.apachefriends.org/ "null")

* เปิด **XAMPP Control Panel** ขึ้นมา แล้วกด **Start** ที่โมดูล **Apache** และ **MySQL**

### 2.2 สร้าง Virtual Host (เพื่อให้ใช้ URL สวยๆ)

เราจะตั้งค่าให้สามารถเข้าผ่าน `http://one.com` แทน `http://localhost/v.3_api1/public` เพื่อความสวยงามและเป็นมืออาชีพ

#### 1. แก้ไขไฟล์ `httpd-vhosts.conf`

* **ที่อยู่ไฟล์**: 📄 `C:\xampp\apache\conf\extra\httpd-vhosts.conf`

* เปิดไฟล์ขึ้นมาแล้วเพิ่มโค้ดนี้เข้าไปที่ด้านล่างสุด:

```bash
#คงไว้ให้สามารถใช้ localhost ได้ปกติเพื่อใช้งาน phpmyadmin
<VirtualHost *:80>
    DocumentRoot "C:/xampp/htdocs"
    ServerName localhost
</VirtualHost>

#สร้างใหม่ โดยกำหนดชื่อโดเมนและพาธ ของโปรเจ็ค ปกติจะอยู่ที่ /xampp/htdocs/
<VirtualHost *:80>
    DocumentRoot "C:/xampp/htdocs/v.3_api1/public"
    ServerName one.com
    <Directory "C:/xampp/htdocs/v.3_api1/public">
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

---

#### 2. แก้ไขไฟล์ `hosts` ของ Windows

* **ที่อยู่ไฟล์**: 📄 `C:\Windows\System32\drivers\etc\hosts`
* **สำคัญ**: คุณต้องเปิดโปรแกรมแก้ไขข้อความ (เช่น Notepad) แบบ **Run as Administrator** ก่อน ถึงจะแก้ไขไฟล์นี้ได้

* เพิ่มบรรทัดนี้เข้าไปที่ด้านล่างสุด:

```bash
127.0.0.1   localhost
127.0.0.1   one.com
```

#### 3. รีสตาร์ท Apache

* กลับไปที่ **XAMPP Control Panel** แล้วกด **Stop** และ **Start** ที่โมดูล **Apache** อีกครั้งเพื่อให้การตั้งค่ามีผล

### 2.3 สร้างโครงสร้างไฟล์และโฟลเดอร์

โครงสร้างที่ดีคือหัวใจของโปรเจกต์ที่ดูแลรักษาง่าย เราจะใช้ Command Prompt (cmd) เพื่อสร้างทุกอย่างในครั้งเดียว

* เปิด **Command Prompt** แล้วพิมพ์คำสั่งตามลำดับนี้:

```bash
cd C:\xampp\htdocs
mkdir v.3_api1
cd v.3_api1
mkdir public src src\Controllers src\Models src\Middlewares src\Core config
type nul > composer.json
type nul > database.sql
cd public
type nul > index.php
type nul > .htaccess
cd ..\config
type nul > config.php
cd ..\src
type nul > routes.php
cd Core
type nul > Database.php
type nul > Router.php
cd ..\Middlewares
type nul > AuthMiddleware.php
type nul > RateLimiter.php
cd ..\Controllers
type nul > AuthController.php
type nul > UserController.php
type nul > CategoryController.php
type nul > ProductController.php
type nul > OrderController.php
type nul > OrderItemController.php
type nul > AddressController.php
type nul > PaymentController.php
type nul > ReviewController.php
cd ..\Models
type nul > User.php
type nul > Category.php
type nul > Product.php
type nul > Order.php
type nul > OrderItem.php
type nul > Address.php
type nul > Payment.php
type nul > Review.php
```

---

## ขั้นตอนที่ 3: สร้างฐานข้อมูล (The Foundation)
ฐานข้อมูลก็เปรียบเสมือน **"โกดังเก็บของ"** ขนาดใหญ่ของ API ของเรา ถ้าเราออกแบบชั้นวางของ (ตาราง) และติดป้ายกำกับ (คีย์) ได้ดี การค้นหา หยิบ หรือเก็บของ (ข้อมูล) ก็จะทำได้อย่างรวดเร็วและไม่ผิดพลาด

เราจะมาดูกันทีละส่วนว่าสคริปต์ SQL ที่เราใช้สร้างฐานข้อมูลนั้น แต่ละบรรทัดมีความหมายว่าอะไร และทำไมเราถึงออกแบบมันมาแบบนี้ครับ

### ภาพรวมแนวคิดการออกแบบ

เราใช้แนวคิดที่เรียกว่า **"Relational Database"** หรือฐานข้อมูลเชิงสัมพันธ์ ซึ่งหมายความว่าตารางต่างๆ จะมีความเชื่อมโยงกัน เหมือนแผนผังครอบครัว

* **ตาราง (Table):** คือ "แฟ้มเอกสาร" สำหรับเก็บข้อมูลเรื่องเดียวกัน เช่น แฟ้ม `users` ก็จะเก็บข้อมูลของลูกค้าทุกคน

* **คอลัมน์ (Column/Field):** คือ "ช่อง" ในแบบฟอร์มเอกสาร เช่น `first_name`, `email`

* **แถว (Row/Record):** คือ "เอกสาร 1 ใบ" ที่กรอกข้อมูลครบแล้วสำหรับลูกค้า 1 คน

* **คีย์ (Key):** คือ "เลขที่อ้างอิง" ที่ทำให้ข้อมูลแต่ละใบไม่ซ้ำกัน และใช้เชื่อมโยงแฟ้มต่างๆ เข้าด้วยกัน

นี่คือสคริปต์ SQL ที่ครบถ้วนสำหรับสร้างฐานข้อมูล `api_db` และตารางทั้งหมด รวมถึงตาราง `rate_limits` ด้วยครับ

* นำโค้ดนี้ไปรันใน **phpMyAdmin** และบันทึกลงในไฟล์ `database.sql`

```sql
-- บรรทัดที่ขึ้นต้นด้วย -- คือ Comment หรือคำอธิบาย จะไม่มีผลกับการทำงานของโค้ด

-- =======================================================================
-- ส่วนที่ 1: สร้างโกดังเก็บของ (Database)
-- =======================================================================

-- CREATE DATABASE IF NOT EXISTS `api_db` ...
-- คำสั่งนี้บอกว่า: "ถ้ายังไม่มีโกดังที่ชื่อ api_db ให้สร้างขึ้นมาใหม่นะ"
-- การใส่ IF NOT EXISTS ช่วยป้องกัน Error เวลาเรารันสคริปต์นี้ซ้ำ
-- DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
-- นี่คือการตั้งค่า "ภาษา" ที่ใช้ในโกดังของเราครับ
-- utf8mb4 เป็นมาตรฐานที่รองรับอักขระได้หลากหลาย รวมถึงภาษาไทยและ Emoji ด้วย 🤠
CREATE DATABASE IF NOT EXISTS `api_db` DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;

-- USE `api_db`;
-- บอกว่า "เอาล่ะ ต่อจากนี้ไป คำสั่งทั้งหมดให้ทำกับโกดังที่ชื่อ api_db นะ"
USE `api_db`;


-- =======================================================================
-- ส่วนที่ 2: สร้างแฟ้มเก็บเอกสาร (Tables)
-- =======================================================================

-- --- แฟ้มที่ 1: ข้อมูลผู้ใช้ (users) ---
CREATE TABLE `users` (
  -- `id` int(11) NOT NULL AUTO_INCREMENT:
  -- id คือ "เลขประจำตัวประชาชน" ของผู้ใช้แต่ละคน
  -- int(11): เป็นตัวเลข จำนวน 11 หลัก
  -- NOT NULL: ช่องนี้ห้ามเว้นว่างเด็ดขาด
  -- AUTO_INCREMENT: ระบบจะสร้างเลขให้เองอัตโนมัติ ไม่ซ้ำกัน (1, 2, 3, ...)
  `id` int(11) NOT NULL AUTO_INCREMENT,

  -- `username` varchar(50) NOT NULL:
  -- varchar(50): เป็นข้อความ ความยาวไม่เกิน 50 ตัวอักษร
  `username` varchar(50) NOT NULL,
  `email` varchar(255) NOT NULL,
  `password` varchar(255) NOT NULL,
  `first_name` varchar(100) DEFAULT NULL, -- DEFAULT NULL: ถ้าไม่กรอกข้อมูล จะเป็นค่าว่าง (NULL) ได้
  `last_name` varchar(100) DEFAULT NULL,

  -- `role` enum('customer','employee','manager','director','admin') DEFAULT 'customer':
  -- enum(...): กำหนดว่าช่องนี้ต้องเลือกค่าจากในวงเล็บเท่านั้น
  -- DEFAULT 'customer': ถ้าไม่ระบุ จะให้เป็น 'customer' โดยอัตโนมัติ
  `role` enum('customer','employee','manager','director','admin') DEFAULT 'customer',

  -- `created_at` timestamp ... DEFAULT current_timestamp():
  -- timestamp: เก็บข้อมูล วัน-เวลา
  -- DEFAULT current_timestamp(): บันทึกเวลาที่สร้างข้อมูลนี้โดยอัตโนมัติ
  `created_at` timestamp NOT NULL DEFAULT current_timestamp(),

  -- `updated_at` ... ON UPDATE current_timestamp():
  -- พิเศษกว่า created_at คือ "ทุกครั้งที่มีการแก้ไขข้อมูลแถวนี้ ให้บันทึกเวลาใหม่โดยอัตโนมัติ"
  `updated_at` timestamp NOT NULL DEFAULT current_timestamp() ON UPDATE current_timestamp(),

  -- --- การกำหนดคีย์ (Keys) ---
  -- PRIMARY KEY (`id`):
  -- ประกาศให้ `id` เป็น "คีย์หลัก" หรือตัวชี้บ่งที่สำคัญที่สุดของตารางนี้
  PRIMARY KEY (`id`),

  -- UNIQUE KEY `username` (`username`), UNIQUE KEY `email` (`email`):
  -- ประกาศว่าข้อมูลในคอลัมน์ `username` และ `email` "ห้ามซ้ำกันเด็ดขาด" ทั้งตาราง
  UNIQUE KEY `username` (`username`),
  UNIQUE KEY `email` (`email`)

) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4; -- ENGINE=InnoDB: เลือกใช้ "เครื่องยนต์" จัดการฐานข้อมูลที่รองรับความสัมพันธ์ (Foreign Keys)


-- --- แฟ้มที่ 2: หมวดหมู่สินค้า (categories) ---
CREATE TABLE `categories` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(100) NOT NULL,
  `description` text DEFAULT NULL, -- text: สำหรับเก็บข้อความยาวๆ
  `created_at` timestamp NOT NULL DEFAULT current_timestamp(),
  PRIMARY KEY (`id`),
  UNIQUE KEY `name` (`name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;


-- --- แฟ้มที่ 3: สินค้า (products) ---
CREATE TABLE `products` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(255) NOT NULL,
  `price` decimal(10,2) NOT NULL, -- decimal(10,2): เลขทศนิยม, 10 คือจำนวนหลักทั้งหมด, 2 คือจำนวนหลักทศนิยม (เช่น 12345678.99)
  `description` text DEFAULT NULL,
  `category_id` int(11) DEFAULT NULL, -- นี่คือ "คีย์นอก" (Foreign Key) ที่จะใช้เชื่อมกับตาราง categories
  `stock` int(11) NOT NULL DEFAULT 0,
  `created_at` timestamp NOT NULL DEFAULT current_timestamp(),
  `updated_at` timestamp NOT NULL DEFAULT current_timestamp() ON UPDATE current_timestamp(),
  PRIMARY KEY (`id`),

  -- --- การสร้างความสัมพันธ์ (Relationships) ---
  -- KEY `category_id` (`category_id`):
  -- สร้าง "ดัชนี" (Index) ให้กับคอลัมน์นี้ เพื่อให้ค้นหาได้เร็วขึ้น
  KEY `category_id` (`category_id`),

  -- CONSTRAINT `products_ibfk_1` FOREIGN KEY (`category_id`) REFERENCES `categories` (`id`) ON DELETE SET NULL:
  -- นี่คือการ "ลากเส้นเชื่อม" แฟ้มเอกสารเข้าด้วยกัน
  -- CONSTRAINT: สร้างกฎความสัมพันธ์
  -- FOREIGN KEY (`category_id`): บอกว่าคอลัมน์ `category_id` ในตารางนี้...
  -- REFERENCES `categories` (`id`): ...จะอ้างอิงไปที่คอลัมน์ `id` ในตาราง `categories`
  -- ON DELETE SET NULL: คือกฎสำคัญที่บอกว่า "ถ้าหมวดหมู่ (Category) ที่สินค้านี้สังกัดอยู่ถูกลบไป,
  -- ให้ตั้งค่า category_id ของสินค้านี้ให้เป็นค่าว่าง (NULL) นะ" (เพื่อไม่ให้สินค้าหายไปด้วย)
  CONSTRAINT `products_ibfk_1` FOREIGN KEY (`category_id`) REFERENCES `categories` (`id`) ON DELETE SET NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;


-- --- แฟ้มที่ 4: คำสั่งซื้อ (orders) ---
CREATE TABLE `orders` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `user_id` int(11) NOT NULL, -- เชื่อมกับตาราง users
  `total_amount` decimal(10,2) NOT NULL,
  `status` enum('pending','completed','cancelled') DEFAULT 'pending',
  `created_at` timestamp NOT NULL DEFAULT current_timestamp(),
  `updated_at` timestamp NOT NULL DEFAULT current_timestamp() ON UPDATE current_timestamp(),
  PRIMARY KEY (`id`),
  KEY `user_id` (`user_id`),

  -- CONSTRAINT ... ON DELETE CASCADE:
  -- กฎนี้ต่างจาก SET NULL, มันบอกว่า "ถ้าผู้ใช้ (User) ที่เป็นเจ้าของออเดอร์นี้ถูกลบออกจากระบบ,
  -- ให้ลบออเดอร์นี้ทิ้งตามไปด้วยเลย (CASCADE)" เพราะออเดอร์จะไม่มีความหมายถ้าไม่มีเจ้าของ
  CONSTRAINT `orders_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `users` (`id`) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;


-- (ตารางที่เหลือจะใช้หลักการเดียวกันกับที่อธิบายไปข้างต้นครับ)

-- --- แฟ้มที่ 5: รายการสินค้าในคำสั่งซื้อ (order_items) ---
-- ตารางนี้คือ "ตารางเชื่อม" (Junction Table) ระหว่าง orders และ products
CREATE TABLE `order_items` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `order_id` int(11) NOT NULL, -- เชื่อมกับ orders
  `product_id` int(11) DEFAULT NULL, -- เชื่อมกับ products
  `quantity` int(11) NOT NULL,
  `unit_price` decimal(10,2) NOT NULL,
  `created_at` timestamp NOT NULL DEFAULT current_timestamp(),
  PRIMARY KEY (`id`),
  KEY `order_id` (`order_id`),
  KEY `product_id` (`product_id`),
  CONSTRAINT `order_items_ibfk_1` FOREIGN KEY (`order_id`) REFERENCES `orders` (`id`) ON DELETE CASCADE,
  CONSTRAINT `order_items_ibfk_2` FOREIGN KEY (`product_id`) REFERENCES `products` (`id`) ON DELETE SET NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- (ตาราง Addresses, Payments, Reviews, Rate_limits ก็จะใช้หลักการเดียวกัน)
```

#### สรุปหัวใจสำคัญของการออกแบบฐานข้อมูล

* **Normalization:** เราพยายามแยกข้อมูลแต่ละเรื่องออกจากกัน (เช่น User, Product) เพื่อลดความซ้ำซ้อนและทำให้จัดการง่าย
* **Primary Key:** ทุกตารางต้องมี `id` ที่เป็น Primary Key เพื่อให้ข้อมูลแต่ละแถวมีเอกลักษณ์ ไม่ซ้ำกัน
* **Foreign Key:** เราใช้ `user_id`, `category_id`, `order_id` เป็น Foreign Key เพื่อสร้าง "ความสัมพันธ์" ระหว่างตาราง ทำให้ข้อมูลเชื่อมโยงกันอย่างมีความหมาย
* **Data Integrity (ความถูกต้องของข้อมูล):** การใช้ `CONSTRAINT` และกฎ `ON DELETE` ช่วยให้มั่นใจว่าฐานข้อมูลของเราจะไม่มี "ข้อมูลกำพร้า" (เช่น ออเดอร์ที่มี `user_id` ที่ไม่มีอยู่จริง) และรักษาความถูกต้องของข้อมูลไว้เสมอ
**เกร็ดความรู้**: สังเกตว่าเราใช้ `ON DELETE CASCADE` และ `ON DELETE SET NULL` เพื่อจัดการความสัมพันธ์ของข้อมูล

* `CASCADE`: ถ้าลบข้อมูลหลัก (เช่น ลบ Order) ข้อมูลย่อยที่ผูกกันอยู่ (เช่น Order Items) จะถูกลบตามไปด้วย
* `SET NULL`: ถ้าลบข้อมูลหลัก (เช่น ลบ User) ข้อมูลที่เคยผูกอยู่ (เช่น Review) จะถูกเปลี่ยนค่า Foreign Key ให้เป็น `NULL` เพื่อรักษาข้อมูลรีวิวไว้

---

## ขั้นตอนที่ 4: ติดตั้ง Dependencies ด้วย Composer

**Composer** คือเครื่องมือจัดการ "ส่วนเสริม" หรือ Library ต่างๆ ในโปรเจกต์ PHP ของเรา เราจะใช้มันเพื่อติดตั้ง `php-jwt` สำหรับจัดการ Token

### 4.1 สร้างไฟล์ `composer.json`

* เปิดไฟล์ 📄 `C:\xampp\htdocs\v.3_api1\composer.json` แล้วใส่โค้ดนี้เข้าไป:

  ```json
  {
      "require": {
          "firebase/php-jwt": "^6.10"
      },
      "autoload": {
          "psr-4": {
              "App\\": "src/"
          }
      }
  }
  ```

* `autoload` ส่วนนี้สำคัญมาก มันบอก Composer ว่าไฟล์คลาสทั้งหมดของเราอยู่ในโฟลเดอร์ `src` ทำให้เราสามารถเรียกใช้งานคลาสต่างๆ ได้ง่ายๆ ด้วย `use App\Controllers\UserController;` เป็นต้น

### 4.2 ติดตั้ง Dependencies

* เปิด Command Prompt ที่พาธของโปรเจกต์ (`C:\xampp\htdocs\v.3_api1`)

* รันคำสั่ง:

  ```bash
  composer install
  ```

* รอจนติดตั้งเสร็จ คุณจะเห็นโฟลเดอร์ `vendor` ปรากฏขึ้นในโปรเจกต์ของคุณ

## ขั้นตอนที่ 5: ลงมือเขียนโค้ด (The Magic Happens Here)

ถึงเวลาสนุกแล้วครับ! เราจะเริ่มสร้างไฟล์แต่ละไฟล์ตามโครงสร้าง MVC ที่วางไว้

### 5.1 ภาพรวมการทำงานและการไหลของข้อมูล (Big Picture & Data Flow)

ก่อนจะดูโค้ดแต่ละไฟล์ เรามาทำความเข้าใจ "การเดินทาง" ของ Request หนึ่งๆ กันก่อนดีกว่าครับ เมื่อคุณยิง API จาก Postman มาที่ `GET http://one.com/api/users/1` มันเกิดอะไรขึ้นบ้าง?

1. **Request เริ่มต้นที่ `.htaccess`**: ไฟล์นี้จะรับ Request ทั้งหมดแล้วส่งต่อไปยังประตูหลักของเรา
2. **ประตูหลัก `public/index.php`**: ไฟล์นี้จะโหลดไฟล์ที่จำเป็นทั้งหมดขึ้นมาเตรียมไว้ และสร้าง `Router` ขึ้นมา
3. **แผนที่ `src/routes.php`**: `Router` จะเปิดแผนที่นี้เพื่อหาว่า URL `/users/1` ตรงกับเส้นทางไหน
4. **ด่านตรวจ `src/Middlewares`**: เมื่อเจอเส้นทาง `Router` จะเรียก `Middleware` ที่กำหนดไว้ (เช่น `AuthMiddleware@isSelfOrManager`) มาทำงานก่อน `Middleware` จะตรวจ Token และสิทธิ์การเข้าถึง ถ้าไม่ผ่าน ก็จะส่ง Error กลับไปทันที
5. **ผู้ควบคุม `src/Controllers/UserController.php`**: ถ้าผ่านด่านตรวจมาได้ `Router` จะเรียกเมธอด `show()` ใน `UserController` พร้อมส่งค่า `id=1` ไปให้
6. **คนทำงาน `src/Models/User.php`**: `UserController` ไม่ทำงานกับฐานข้อมูลโดยตรง แต่จะสั่งให้ `User` Model ไปหาข้อมูลผู้ใช้ที่มี `id=1`
7. **ฐานข้อมูล (Database)**: `User` Model จะเขียนคำสั่ง SQL ไปดึงข้อมูลจากตาราง `users`
8. **ส่งข้อมูลกลับ**: `Model` ส่งข้อมูลที่ได้กลับไปให้ `Controller`
9. **ส่ง Response**: `Controller` นำข้อมูลมาแปลงเป็น JSON แล้วส่งกลับไปให้ผู้ใช้

การแบ่งหน้าที่แบบนี้ (MVC) ทำให้โค้ดของเราสะอาด เป็นระเบียบ และง่ายต่อการแก้ไขในอนาคตครับ

### 5.2 ไฟล์พื้นฐานและไฟล์ Core System

### 📄`public/.htaccess`
#### อธิบายไฟล์ public/.htaccess: ยามหน้าประตูและผู้ควบคุมการจราจร

ถ้า `index.php` คือพนักงานต้อนรับที่คอยจัดการแขกอยู่ _ข้างใน_ บริษัท ไฟล์ `.htaccess` ก็เปรียบเสมือน **"เจ้าหน้าที่รักษาความปลอดภัยและควบคุมการจราจร"** ที่ยืนอยู่ _หน้าประตู_ บริษัทเลยครับ เขามีหน้าที่หลักๆ คือ:

1. **กำหนดกฎการจราจร (Rewrite Rules):** คอยโบกรถ (Requests) ทุกคันที่วิ่งเข้ามา ให้เลี้ยวเข้าไปที่ "ประตูทางเข้าหลัก" (`index.php`) เพียงประตูเดียว ไม่ว่าจะพยายามจะไปที่ไหนก็ตาม
2. **จัดการเอกสารข้ามแดน (CORS):** คอยตรวจสอบและอนุญาตให้ "แขกจากต่างบริษัท" (เช่น เว็บไซต์อื่น หรือ Mobile App) สามารถเข้ามาติดต่อสื่อสารกับบริษัทของเราได้
3. **เปิด-ปิดระบบ (Engine On/Off):** เป็นคนตัดสินใจว่าจะเริ่มใช้กฎจราจรเหล่านี้หรือไม่

ไฟล์นี้มีความพิเศษตรงที่มันไม่ใช่ไฟล์ PHP แต่เป็นไฟล์ตั้งค่าของ **Apache Web Server** โดยตรง มันจะทำงานก่อนที่โค้ด PHP ของเราจะเริ่มทำงานเสียอีกครับ

```bash
# บรรทัดที่ขึ้นต้นด้วย # คือ Comment หรือคำอธิบาย จะไม่มีผลกับการทำงานของโค้ด

# =======================================================================
# ส่วนที่ 1: เปิดระบบควบคุมการจราจร
# =======================================================================

# RewriteEngine On
# นี่คือสวิตช์เปิด-ปิดระบบครับ คำสั่งนี้บอก Apache ว่า "เอาล่ะ! เริ่มใช้กฎการจราจร (Rewrite Rules) ที่ฉันจะเขียนต่อจากนี้ได้เลย"
# ถ้าบรรทัดนี้เป็น Off หรือไม่มีอยู่ กฎทั้งหมดด้านล่างจะถูกเมินเฉย
RewriteEngine On

# =======================================================================
# ส่วนที่ 2: ตั้งกฎสำหรับแขกจากต่างแดน (CORS Headers)
# =======================================================================
# ส่วนนี้สำคัญมากสำหรับการทำ API เพราะ Client (เช่น React App, Vue App)
# ที่รันอยู่คนละโดเมนกับ API ของเรา จะถูกเบราว์เซอร์บล็อกไว้ด้วยเหตุผลด้านความปลอดภัย
# เราจึงต้อง "ประกาศ" บอกเบราว์เซอร์ว่า "เรายินดีต้อนรับแขกจากทุกที่นะ"

# Header set Access-Control-Allow-Origin "*"
# ประกาศว่า "เว็บไซต์จากโดเมนไหนก็ได้ (* คือทั้งหมด) สามารถส่ง Request มาหาเราได้"
Header set Access-Control-Allow-Origin "*"

# Header set Access-Control-Allow-Methods "GET, POST, PUT, DELETE, OPTIONS"
# บอกว่า "เราอนุญาตให้ใช้ท่า (Method) ในการสื่อสารได้แก่ GET, POST, PUT, DELETE, และ OPTIONS นะ"
Header set Access-Control-Allow-Methods "GET, POST, PUT, DELETE, OPTIONS"

# Header set Access-Control-Allow-Headers "Content-Type, Authorization"
# บอกว่า "ในการส่ง Request มาหาเรา คุณสามารถแนบหัวข้อ (Headers) เรื่อง Content-Type (ชนิดของข้อมูล)
# และ Authorization (ข้อมูลยืนยันตัวตน เช่น Token) มาได้นะ"
Header set Access-Control-Allow-Headers "Content-Type, Authorization"

# =======================================================================
# ส่วนที่ 3: กฎการจราจรหลัก (Main Rewrite Rules)
# =======================================================================
# นี่คือหัวใจของการทำ "Friendly URLs" หรือ URL สวยๆ ครับ
# เป้าหมายคือการบังคับให้ทุก Request ที่เข้ามา วิ่งไปที่ไฟล์ index.php เสมอ

# RewriteCond %{REQUEST_FILENAME} !-f
# RewriteCond %{REQUEST_FILENAME} !-d
# สองบรรทัดนี้คือ "เงื่อนไข" ที่ทำงานคู่กัน มันบอกว่า:
# "ถ้า... URL ที่ร้องขอมานั้น...
#   1. ไม่ใช่ไฟล์ที่มีอยู่จริงในเซิร์ฟเวอร์ (!-f)
#   2. และ ไม่ใช่โฟลเดอร์ที่มีอยู่จริงในเซิร์ฟเวอร์ (!-d)
# ...แล้วค่อยทำตามกฎในบรรทัดถัดไป"
#
# พูดง่ายๆ คือ ถ้ามีคนขอไฟล์รูปภาพ (เช่น /images/logo.png) ที่มีอยู่จริง
# Apache ก็จะส่งไฟล์รูปนั้นกลับไปเลย โดยไม่ยุ่งกับกฎข้อถัดไป
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d

# RewriteRule ^ index.php [QSA,L]
# นี่คือ "กฎ" หลักของเรา มันบอกว่า:
# "ถ้าเงื่อนไขข้างบนเป็นจริง ให้เปลี่ยนเส้นทาง (Rewrite) ของ Request ทั้งหมด (^)
# ไปที่ไฟล์ index.php"
#
# [QSA] ย่อมาจาก Query String Append หมายความว่าถ้ามีอะไรต่อท้าย URL (เช่น ?sort=price)
# ให้ส่งต่อไปให้ index.php ด้วยนะ อย่าทำตกหล่น
#
# [L] ย่อมาจาก Last หมายความว่าถ้ากฎนี้ทำงานแล้ว ให้หยุดทันที ไม่ต้องไปดูกฎข้ออื่นต่อแล้ว
RewriteRule ^ index.php [QSA,L]
```

---

### 📄`public/index.php`
#### อธิบายไฟล์ public/index.php: ประตูหน้าด่านของ API ของเรา

ลองจินตนาการว่า API ของเราเป็นเหมือนบริษัทขนาดใหญ่ ไฟล์ `index.php` ก็เปรียบเสมือนกับ **"พนักงานต้อนรับ"** ที่นั่งอยู่หน้าประตู ทุกๆ คน (หรือทุกๆ Request) ที่จะเข้ามาในบริษัทนี้ จะต้องผ่านพนักงานต้อนรับคนนี้ก่อนเสมอ พนักงานคนนี้มีหน้าที่หลักๆ 3 อย่างคือ:

1. **จัดเตรียมสถานที่ให้พร้อม (Setup):** เปิดไฟ, เปิดแอร์, ตั้งค่ากฎระเบียบต่างๆ ให้พร้อมรับแขก
2. **เรียกทีมงานที่จำเป็น (Loading):** เรียกผู้จัดการ (Router), เลขา (Database), และอ่านแผนผังบริษัท (routes.php) มาเตรียมไว้
3. **ส่งแขกไปให้ถูกแผนก (Dispatching):** เมื่อมีแขกเข้ามา ก็จะดูว่าแขกต้องการติดต่อเรื่องอะไร แล้วส่งต่อไปให้ผู้จัดการ (Router) เพื่อพาไปหาแผนก (Controller) ที่ถูกต้อง

ตอนนี้เรามาดูโค้ดจริงๆ กันดีกว่าครับว่า "พนักงานต้อนรับ" คนเก่งของเราทำงานอย่างไรบ้าง


```php
<?php
/**
 * =======================================================================
 * ส่วนที่ 1: การตั้งค่าพื้นฐาน (เตรียมสถานที่ให้พร้อม)
 * =======================================================================
 */

// ini_set('display_errors', 1);
// error_reporting(E_ALL);
// สองบรรทัดนี้เปรียบเสมือนการ "เปิดไฟทุกดวงในออฟฟิศให้สว่างที่สุด"
// เพื่อที่เราจะเห็นข้อผิดพลาด (Error) ทั้งหมดในระหว่างการพัฒนา
// เมื่อนำโปรเจกต์ไปใช้งานจริง (Production) เรามักจะปิดส่วนนี้ไปเพื่อความปลอดภัย
ini_set('display_errors', 1);
error_reporting(E_ALL);


/**
 * =======================================================================
 * ส่วนที่ 2: การเรียกเครื่องมือและทีมงาน (เรียกทีมงานที่จำเป็น)
 * =======================================================================
 */

// require_once __DIR__ . '/../vendor/autoload.php';
// นี่คือคำสั่งที่สำคัญที่สุด! มันคือการเรียก "หัวหน้าแม่บ้าน" (Composer Autoloader)
// ที่รู้ว่าเครื่องมือ (Class) ต่างๆ ที่เราติดตั้งผ่าน Composer เก็บอยู่ที่ไหน
// หลังจากนี้ เราจะสามารถเรียกใช้ Class จาก Library ภายนอก (เช่น JWT) หรือ Class ที่เราสร้างเองได้เลย
require_once __DIR__ . '/../vendor/autoload.php';

// require_once __DIR__ . '/../config/config.php';
// เรียก "สมุดบันทึก" ที่เก็บข้อมูลสำคัญ เช่น รหัสผ่านฐานข้อมูล, Secret Key
// การแยกข้อมูลเหล่านี้ออกมาทำให้โค้ดปลอดภัยและแก้ไขง่าย
require_once __DIR__ . '/../config/config.php';

// require_once __DIR__ . '/../src/Core/Database.php';
// require_once __DIR__ . '/../src/Core/Router.php';
// เรียก "ทีมงานหลัก" ของเราเข้ามา นั่นคือ "เลขา" (Database) ที่จะคุยกับฐานข้อมูล
// และ "ผู้จัดการ" (Router) ที่จะคอยจัดการ Request ทั้งหมด
require_once __DIR__ . '/../src/Core/Database.php';
require_once __DIR__ . '/../src/Core/Router.php';

// $router = new App\Core\Router();
// "จ้างผู้จัดการ" (Router) ให้เริ่มทำงาน โดยการสร้าง instance ของคลาส Router ขึ้นมา
$router = new App\Core\Router();

// require_once __DIR__ . '/../src/routes.php';
// "ยื่นแผนผังบริษัท" (routes.php) ให้กับผู้จัดการ
// เพื่อให้ผู้จัดการรู้ว่าถ้ามีคนมาติดต่อเรื่อง A ต้องไปที่แผนกไหน (Controller ไหน)
require_once __DIR__ . '/../src/routes.php';


/**
 * =======================================================================
 * ส่วนที่ 3: การตั้งกฎระเบียบและจัดการ Request (ส่งแขกไปให้ถูกแผนก)
 * =======================================================================
 */

// date_default_timezone_set('Asia/Bangkok');
// ตั้งนาฬิกาของ Server ให้เป็นเวลาประเทศไทย
date_default_timezone_set('Asia/Bangkok');

// header("Content-Type: application/json; charset=UTF-8");
// "ติดป้ายประกาศหน้าบริษัท" บอกทุกคนว่า "บริษัทนี้สื่อสารด้วยภาษา JSON เท่านั้นนะ"
// นี่เป็นการกำหนดว่า Response ที่ส่งกลับไปจะเป็นรูปแบบ JSON
header("Content-Type: application/json; charset=UTF-8");

// if ($_SERVER['REQUEST_METHOD'] === 'OPTIONS') { ... }
// ส่วนนี้เป็นการจัดการ "CORS Preflight Request"
// เปรียบเสมือนการที่เบราว์เซอร์โทรมาถามก่อนว่า "สวัสดีครับ ผมมาจากเว็บ xxx, ผมสามารถส่งคำขอแบบ POST ไปหาคุณได้ไหม?"
// เราก็ตอบกลับไปว่า "ได้เลย!" (ด้วย status 200) เพื่อให้เบราว์เซอร์ส่งคำขอจริงๆ ตามมา
if ($_SERVER['REQUEST_METHOD'] === 'OPTIONS') {
    http_response_code(200);
    exit();
}

// $router->dispatch();
// คำสั่งสุดท้ายและสำคัญที่สุด! คือการบอก "ผู้จัดการ" (Router) ว่า
// "เริ่มทำงานได้เลย! ดู URL ที่เข้ามา แล้วส่งต่อไปให้แผนกที่ถูกต้องตามแผนผังซะ!"
// เมธอด dispatch() จะเป็นตัวเริ่มกระบวนการค้นหาเส้นทางและเรียก Controller ที่เหมาะสม
$router->dispatch();
?>

```
#### สรุปหัวใจสำคัญของ `index.php`

ไฟล์ `index.php` เป็นหัวใจของ **"Single Entry Point Pattern"** ซึ่งเป็นแนวคิดการออกแบบสถาปัตยกรรมซอฟต์แวร์ที่ทันสมัยและปลอดภัย

* **ความปลอดภัย:** การบังคับให้ทุก Request ต้องผ่านจุดนี้จุดเดียว ทำให้เราสามารถดักจับ, ตรวจสอบ, และจัดการความปลอดภัยทั้งหมดได้ในที่เดียว

* **ความเป็นระเบียบ:** แทนที่จะมีไฟล์ PHP กระจัดกระจายเต็มไปหมด เรามี "ประตูทางเข้า" เพียงประตูเดียว ทำให้ง่ายต่อการจัดการและทำความเข้าใจภาพรวมของโปรเจกต์

* **ความยืดหยุ่น:** เราสามารถเพิ่มหรือแก้ไขกฎระเบียบต่างๆ (เช่น การตั้งค่า Header) ได้จากไฟล์นี้ไฟล์เดียว โดยมีผลกับทุกๆ Endpoint ใน API ของเรา

ตอนนี้คุณน่าจะเข้าใจแล้วว่าทำไม "พนักงานต้อนรับ" คนนี้ถึงมีความสำคัญกับ "บริษัท API" ของเรามากขนาดนี้ครับ!

---

### 📄`config/config.php`
ลองนึกภาพว่า API ของเราคือ "สายลับ" ที่ต้องไปปฏิบัติภารกิจ ไฟล์ `config.php` ก็คือ **"สมุดจดรหัสลับ"** ที่สายลับคนนี้พกติดตัวไว้ ในสมุดเล่มนี้จะเก็บข้อมูลที่สำคัญและเป็นความลับสุดยอด ซึ่งจำเป็นต่อการทำภารกิจให้สำเร็จ

**ทำไมเราต้องแยกไฟล์นี้ออกมา?** เหตุผลง่ายๆ เลยคือ **ความปลอดภัย** และ **ความสะดวก** ครับ
* **ความปลอดภัย:** เราไม่ต้องการเขียนรหัสผ่านฐานข้อมูล หรือ "กุญแจลับ" (Secret Key) ปะปนไปกับโค้ดส่วนอื่นๆ โดยตรง การแยกออกมาทำให้เราจัดการเรื่องความปลอดภัยได้ง่ายขึ้น
* **ความสะดวก:** ถ้าเราต้องย้ายบ้าน (เช่น ย้ายจากเครื่องเราไป Server จริง) เราก็แค่หยิบสมุดเล่มนี้มาแก้ข้อมูลที่อยู่ใหม่ (เช่น ชื่อฐานข้อมูล, รหัสผ่าน) โดยไม่ต้องไปยุ่งกับโค้ดหลักของโปรเจกต์เลยแม้แต่น้อย

```php
<?php
/**
 * =======================================================================
 * ส่วนที่ 1: ข้อมูลสำหรับเชื่อมต่อ "โกดัง" (Database Credentials)
 * =======================================================================
 * ส่วนนี้บอกให้ API ของเรารู้ว่าจะไปหาโกดังข้อมูล (Database) ได้ที่ไหน
 * และต้องใช้บัตรพนักงาน (Username/Password) อะไรในการเข้าไป
 */

// define('DB_HOST', '127.0.0.1');
// ที่อยู่ของโกดังข้อมูล (ปกติในเครื่องเราคือ 127.0.0.1 หรือ localhost)
define('DB_HOST', '127.0.0.1');

// define('DB_USER', 'root');
// ชื่อผู้ใช้สำหรับเข้าไปในโกดัง (ค่าเริ่มต้นของ XAMPP คือ root)
define('DB_USER', 'root');

// define('DB_PASS', '');
// รหัสผ่านสำหรับเข้าไปในโกดัง (ค่าเริ่มต้นของ XAMPP คือ เว้นว่างไว้)
define('DB_PASS', '');

// define('DB_NAME', 'api_db');
// ชื่อของโกดังที่เราต้องการจะเข้าไปทำงานด้วย
define('DB_NAME', 'api_db');


/**
 * =======================================================================
 * ส่วนที่ 2: ข้อมูลสำหรับสร้าง "บัตรพนักงาน" (JWT Settings)
 * =======================================================================
 * ส่วนนี้คือข้อมูลลับที่ใช้ในการสร้างและตรวจสอบ "บัตรพนักงานดิจิทัล" (JWT Token)
 * เพื่อให้เรารู้ว่าคนที่มาใช้งาน API ของเราคือใครและมีสิทธิ์ทำอะไรได้บ้าง
 */

// define('JWT_SECRET', 'YOUR_SUPER_SECRET_KEY_CHANGE_ME_NOW_!@#$%^');
// นี่คือ "ลายเซ็นลับ" หรือ "กุญแจลับ" ที่สุดในโปรเจกต์!
// ใช้ในการเข้ารหัสและถอดรหัส Token ห้ามให้ใครรู้เด็ดขาด และควรตั้งให้ซับซ้อนคาดเดายาก
define('JWT_SECRET', 'YOUR_SUPER_SECRET_KEY_CHANGE_ME_NOW_!@#$%^');

// define('JWT_ISSUER', 'one.com');
// ชื่อ "บริษัท" หรือผู้ออกบัตรพนักงานนี้ (เราตั้งให้ตรงกับชื่อเว็บของเรา)
define('JWT_ISSUER', 'one.com');

// define('JWT_AUDIENCE', 'one.com');
// ระบุว่าบัตรนี้มีไว้สำหรับใช้งานกับ "บริษัท" ไหน (ปกติจะตั้งเป็นชื่อเดียวกับ Issuer)
define('JWT_AUDIENCE', 'one.com');
?>
```

---

### 📄`src/Core/Database.php`
ถ้า `config.php` คือ "สมุดจดรหัสลับ" ไฟล์ `Database.php` ก็เปรียบเสมือน **"ช่างผู้ชำนาญการ"** ที่ถูกจ้างมาเพื่อทำหน้าที่เชื่อมต่อกับ "โกดังข้อมูล" (Database) ของเราโดยเฉพาะ ช่างคนนี้มีความสามารถพิเศษคือ:

1. **อ่านรหัสลับได้:** เขารู้วิธีเปิดสมุด `config.php` เพื่อเอาข้อมูลที่อยู่, ชื่อผู้ใช้, และรหัสผ่านของโกดังมาใช้งาน
2. **เชี่ยวชาญการเชื่อมต่อ:** เขารู้วิธีการเชื่อมต่อที่ปลอดภัยและเป็นมาตรฐานที่สุด โดยใช้เครื่องมือที่ชื่อว่า **PDO (PHP Data Objects)**
3. **ทำงานเมื่อถูกเรียกใช้เท่านั้น:** เขาจะไม่ออกไปเชื่อมต่อพร่ำเพรื่อ แต่จะลงมือทำก็ต่อเมื่อมีคน (เช่น Model) มาเรียกใช้เท่านั้น
4. **รอบคอบและปลอดภัย:** เขามีแผนสำรองเสมอ ถ้าหากการเชื่อมต่อล้มเหลว (เช่น ใส่รหัสผิด) เขาจะแจ้งข้อผิดพลาดอย่างสุภาพและหยุดการทำงานทันที เพื่อไม่ให้ระบบทั้งหมดพังลงมา

การมี "ช่าง" แยกออกมาโดยเฉพาะแบบนี้ ทำให้โค้ดส่วนอื่นๆ ของเรา (เช่น `User.php`, `Product.php`) ไม่ต้องกังวลเรื่องวิธีการเชื่อมต่อที่ซับซ้อนเลย แค่เรียกใช้ช่างคนนี้คนเดียว ทุกอย่างก็เรียบร้อย!

```php
<?php
// ประกาศว่าไฟล์นี้อยู่ใน "แผนก" Core
namespace App\Core;

// นำเครื่องมือที่ชื่อว่า PDO (PHP Data Objects) เข้ามาเตรียมใช้งาน
// PDO เป็นเครื่องมือมาตรฐานของ PHP สำหรับคุยกับฐานข้อมูลได้หลากหลายชนิดอย่างปลอดภัย
use PDO;
use PDOException; // นำ "กล่องเครื่องมือจัดการข้อผิดพลาด" ของ PDO เข้ามาด้วย

/**
 * คลาส Database
 * ช่างผู้ชำนาญการด้านการเชื่อมต่อฐานข้อมูล
 */
class Database {
    // ตัวแปรสำหรับเก็บ "ท่อเชื่อมต่อ" ที่สร้างขึ้น
    private $conn;

    /**
     * เมธอด __construct (ทำงานอัตโนมัติเมื่อถูก "จ้าง")
     * ทันทีที่มีคนสร้าง instance ของคลาสนี้ (new Database()) โค้ดในนี้จะทำงานทันที
     */
    public function __construct() {
        $this->conn = null; // เริ่มต้นด้วยการบอกว่ายังไม่มีท่อเชื่อมต่อ

        // "ลอง" ทำการเชื่อมต่อ (นี่คือความรอบคอบของช่าง)
        try {
            // สร้าง "ท่อเชื่อมต่อ" (Connection) ใหม่ด้วยข้อมูลจาก config.php
            // นี่คือบรรทัดที่การเชื่อมต่อเกิดขึ้นจริงๆ
            $this->conn = new PDO("mysql:host=" . DB_HOST . ";dbname=" . DB_NAME . ";charset=utf8mb4", DB_USER, DB_PASS);

            // ตั้งค่า "กฎ" ของท่อเชื่อมต่อ: "ถ้ามีอะไรผิดพลาด ให้โยน Error ออกมาเลยนะ อย่าเงียบ"
            $this->conn->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

        // "ถ้า" การลองล้มเหลว (catch)
        } catch (PDOException $e) {
            // ให้หยุดทุกอย่าง แล้วแจ้งข้อผิดพลาดอย่างสุภาพ
            http_response_code(500); // Internal Server Error
            echo json_encode(['message' => 'Database Connection Error: ' . $e->getMessage()]);
            exit(); // หยุดการทำงานของสคริปต์ทันที
        }
    }

    /**
     * เมธอด getConnection (สำหรับส่งมอบท่อเชื่อมต่อ)
     * เมื่อโค้ดส่วนอื่นต้องการคุยกับฐานข้อมูล เขาจะมาเรียกเมธอดนี้
     * เพื่อขอ "ท่อเชื่อมต่อ" ที่ช่างคนนี้สร้างไว้
     */
    public function getConnection() {
        return $this->conn;
    }
}
?>
```
#### สรุปหัวใจสำคัญของ `Database.php`

* **แยกความรับผิดชอบ (Separation of Concerns):** ไฟล์นี้ทำหน้าที่เดียวคือ "การเชื่อมต่อฐานข้อมูล" โค้ดส่วนอื่นไม่ต้องมายุ่ง ทำให้โค้ดสะอาดและจัดการง่าย
* **ใช้ PDO เพื่อความปลอดภัย:** PDO ช่วยป้องกันการโจมตีประเภทหนึ่งที่เรียกว่า "SQL Injection" ได้ดีกว่าวิธีเชื่อมต่อแบบเก่า
* **จัดการข้อผิดพลาด (Error Handling):** การใช้ `try...catch` ทำให้ API ของเราไม่ "ตาย" แบบไม่ทราบสาเหตุเมื่อเชื่อมต่อฐานข้อมูลไม่ได้ แต่จะส่ง Response ที่เป็น JSON พร้อมบอกสาเหตุกลับไปอย่างเป็นมิตร

---

#### 📄`src/Core/Router.php` 
ถ้า `index.php` คือพนักงานต้อนรับ และ `.htaccess` คือยามหน้าประตู `Router.php` ก็คือ **"ผู้จัดการแผนกจ่ายงาน"** หรือ **"โอเปอเรเตอร์"** ที่นั่งอยู่กลางออฟฟิศเลยครับ เขาคือคนที่รู้ทุกซอกทุกมุมของบริษัท และมีหน้าที่รับเรื่องต่อจากพนักงานต้อนรับเพื่อส่งต่อไปยังแผนกที่ถูกต้อง

ความสามารถพิเศษของผู้จัดการคนนี้คือ:

1. **อ่านแผนผังบริษัทได้:** เขารับ "แผนผัง" (`routes.php`) มาแล้วจำได้ทั้งหมดว่าเส้นทาง (URL) ไหน ต้องไปติดต่อแผนก (Controller) ไหน
2. **เข้าใจคำขอที่ซับซ้อน:** เขาสามารถแยกแยะได้ว่า URL อย่าง `/users/1` กับ `/users/2` เป็นเรื่องเดียวกัน (คือการดูข้อมูลผู้ใช้) แต่แค่ต้องการดู "เอกสารคนละใบ" (ID ต่างกัน)
3. **รู้จักด่านตรวจ:** เขารู้ว่าก่อนจะส่งแขกไปบางแผนก ต้องส่งไปผ่าน "ด่านตรวจ" (`Middleware`) ก่อน เพื่อเช็คบัตรและสิทธิ์การเข้าถึง
4. **จ่ายงานแม่นยำ:** เมื่อทุกอย่างเรียบร้อย เขาจะ "จ่ายงาน" ไปให้หัวหน้าแผนก (`Controller`) ที่ถูกต้อง พร้อมแนบข้อมูลที่จำเป็น (เช่น ID จาก URL) ไปให้ด้วย

การมีผู้จัดการที่ฉลาดแบบนี้ ทำให้ `index.php` ไม่ต้องทำงานหนัก แค่รับแขกแล้วส่งต่อให้ผู้จัดการคนนี้คนเดียว ทุกอย่างก็เป็นไปตามกระบวนการอย่างราบรื่นครับ

```php
<?php
// ประกาศว่าไฟล์นี้อยู่ใน "แผนก" Core
namespace App\Core;

/**
 * คลาส Router
 * ผู้จัดการแผนกจ่ายงานอัจฉริยะ
 */
class Router {
    // "สมุดจด" สำหรับเก็บเส้นทางทั้งหมดที่อ่านมาจาก routes.php
    protected $routes = [];

    /**
     * เมธอด addRoute (สำหรับจดเส้นทางลงสมุด)
     * เมธอดนี้จะถูกเรียกใช้โดยไฟล์ routes.php
     */
    private function addRoute($method, $uri, $action, $middlewares = []) {
        // แปลง URL สวยๆ เช่น /users/{id} ให้กลายเป็น "ภาษานักสืบ" (Regular Expression)
        // เพื่อให้คอมพิวเตอร์เข้าใจและสามารถดึงค่า {id} ออกมาได้
        $uri = preg_replace('/\{([a-z_]+)\}/', '(?P<$1>[^/]+)', $uri);
        $uri = '#^/api' . $uri . '$#'; // กำหนดกรอบให้ชัดเจนว่าต้องขึ้นต้นด้วย /api

        // บันทึกข้อมูลเส้นทางลงในสมุดจด
        $this->routes[] = [
            'method' => $method, // ท่าที่ใช้ (GET, POST)
            'uri' => $uri,       // เส้นทาง (ในภาษานักสืบ)
            'action' => $action, // แผนกที่จะให้ไป (Controller@method)
            'middleware' => is_array($middlewares) ? $middlewares : ($middlewares ? explode(',', $middlewares) : []) // ด่านตรวจที่ต้องผ่าน
        ];
    }

    // เมธอดทางลัดสำหรับเพิ่มเส้นทางแต่ละแบบให้ง่ายขึ้น
    public function get($uri, $action, $middleware = []) { $this->addRoute('GET', $uri, $action, $middleware); }
    public function post($uri, $action, $middleware = []) { $this->addRoute('POST', $uri, $action, $middleware); }
    public function put($uri, $action, $middleware = []) { $this->addRoute('PUT', $uri, $action, $middleware); }
    public function delete($uri, $action, $middleware = []) { $this->addRoute('DELETE', $uri, $action, $middleware); }

    /**
     * เมธอด dispatch (หัวใจของการจ่ายงาน)
     * เมธอดนี้จะถูกเรียกจาก index.php เพื่อเริ่มกระบวนการทั้งหมด
     */
    public function dispatch() {
        // ดูว่าแขกที่เข้ามาใช้ URL อะไร และมาด้วยท่าไหน (GET, POST, etc.)
        $requestUri = parse_url($_SERVER['REQUEST_URI'], PHP_URL_PATH);
        $requestMethod = $_SERVER['REQUEST_METHOD'];

        // เปิดสมุดจด แล้วไล่ดูทีละเส้นทางว่าตรงกับที่แขกขอมาหรือไม่
        foreach ($this->routes as $route) {
            // ถ้าท่าไม่ตรง (เช่น ขอมาเป็น POST แต่ในสมุดจดเป็น GET) ก็ข้ามไปดูข้อถัดไป
            if ($route['method'] !== $requestMethod) continue;

            // "ลองเทียบ" URL ของแขกกับเส้นทางในสมุด (ที่แปลงเป็นภาษานักสืบแล้ว)
            if (preg_match($route['uri'], $requestUri, $matches)) {
                // ถ้าเจอเส้นทางที่ตรงกัน!
                // ดึงข้อมูลจาก URL ออกมา (เช่น id=1)
                $params = array_filter($matches, 'is_string', ARRAY_FILTER_USE_KEY);

                // ส่งแขกไปผ่าน "ด่านตรวจ" ทั้งหมดที่กำหนดไว้
                foreach ($route['middleware'] as $mw) {
                    $middlewareResult = $this->executeMiddleware($mw, $params);
                    // ถ้าด่านตรวจไม่อนุญาตให้ผ่าน...
                    if ($middlewareResult !== true) {
                        // ...ก็ส่งแขกกลับบ้านพร้อมแจ้งเหตุผลทันที
                        http_response_code($middlewareResult['status']);
                        echo json_encode(['message' => $middlewareResult['message']]);
                        return; // จบการทำงาน
                    }
                }

                // ถ้าผ่านทุกด่านแล้ว ก็ถึงเวลาส่งไปให้แผนกที่ถูกต้อง
                list($controller, $method) = explode('@', $route['action']);
                $controllerInstance = new ("App\\Controllers\\" . $controller)();
                
                // เรียกหัวหน้าแผนก (Controller) ให้เริ่มทำงาน พร้อมส่งข้อมูลที่จำเป็น (params) ไปให้
                $controllerInstance->$method($params);
                return; // ทำงานสำเร็จแล้ว จบการทำงาน
            }
        }

        // ถ้าวนดูจนหมดสมุดแล้วยังไม่เจอเส้นทางที่ตรงกัน
        http_response_code(404); // Not Found
        echo json_encode(['message' => 'Endpoint not found.']);
    }

    /**
     * เมธอด executeMiddleware (ผู้ช่วยที่คอยเรียก รปภ.)
     * เมธอดนี้ฉลาดพอที่จะรู้ว่าต้องส่งข้อมูลอะไรไปให้ด่านตรวจบ้าง
     */
    private function executeMiddleware($middlewareClass, $urlParams) {
        $method = 'handle';
        // เช็คว่ามีการระบุเมธอดเฉพาะหรือไม่ (เช่น AuthMiddleware@isSelfOrManager)
        if (strpos($middlewareClass, '@') !== false) {
            list($middlewareClass, $method) = explode('@', $middlewareClass);
        }
        $instance = new $middlewareClass();
        $reflectionMethod = new \ReflectionMethod($instance, $method);
        $requiredParams = $reflectionMethod->getParameters();
        $args = [];
        // เตรียมข้อมูล (เช่น id) เพื่อส่งไปให้เมธอดของ Middleware
        foreach ($requiredParams as $param) {
            if (isset($urlParams[$param->getName()])) {
                $args[] = $urlParams[$param->getName()];
            }
        }
        // เรียกด่านตรวจให้ทำงานพร้อมส่งข้อมูลที่จำเป็น
        return $instance->{$method}(...$args);
    }
}
?>
```
* **Centralized Routing:** เป็นศูนย์กลางในการจัดการเส้นทางทั้งหมดของ API ทำให้เราเห็นภาพรวมและแก้ไขได้ง่ายในที่เดียว
* **Clean URLs:** เป็นกลไกหลักที่ทำให้เราสามารถสร้าง URL ที่สวยงามและเข้าใจง่าย (เช่น `/products/10`)
* **Middleware Integration:** ทำหน้าที่เป็นผู้ควบคุมการเรียกใช้ Middleware ทำให้เราสามารถเพิ่ม "ด่านตรวจ" เพื่อรักษาความปลอดภัยให้กับเส้นทางต่างๆ ได้อย่างยืดหยุ่น

### 5.3 Middlewares (ด่านตรวจ)
ถ้า `Router` คือผู้จัดการที่คอยจ่ายงาน `AuthMiddleware` ก็เปรียบเสมือน **"ด่านตรวจรักษาความปลอดภัย"** ที่ตั้งอยู่หน้าแผนกสำคัญๆ ของบริษัทเราเลยครับ "รปภ." ที่ด่านนี้มีหน้าที่หลักๆ คือ:

1. **ขอดูบัตรพนักงาน (Check for Token):** เมื่อมีใครเดินมา รปภ. จะขอดู "บัตรพนักงานดิจิทัล" (JWT Token) ที่แนบมากับคำขอ (Request Header) ก่อนเป็นอันดับแรก ถ้าไม่มีบัตรมา ก็จะถูกเชิญกลับทันที
2. **ตรวจสอบบัตร (Validate Token):** รปภ. จะนำบัตรไปตรวจสอบกับ "ลายเซ็นลับ" (`JWT_SECRET`) ของบริษัท เพื่อให้แน่ใจว่าเป็นบัตรของจริง ไม่ใช่บัตรปลอม และยังไม่หมดอายุ
3. **เช็คข้อมูลในบัตร (Decode Payload):** เมื่อแน่ใจว่าเป็นบัตรจริง รปภ. จะอ่านข้อมูลในบัตรเพื่อดูว่าพนักงานคนนี้คือใคร (User ID) และมีตำแหน่งอะไร (Role)
4. **ตรวจสอบสิทธิ์การเข้าถึง (Authorize):** สุดท้าย รปภ. จะดูว่าตำแหน่งของพนักงานคนนี้ได้รับอนุญาตให้เข้าแผนกนี้หรือไม่ (เช่น บางแผนกเข้าได้เฉพาะ Manager ขึ้นไป)

ถ้าทุกอย่างถูกต้อง รปภ. ก็จะอนุญาตให้ผ่านเข้าไปได้ แต่ถ้ามีขั้นตอนไหนผิดพลาดแม้แต่นิดเดียว รปภ. ก็จะปฏิเสธการเข้าถึงและแจ้งเหตุผลกลับไปทันที

---

### 📄`src/Middlewares/AuthMiddleware.php`

```php
<?php
// ประกาศว่าไฟล์นี้อยู่ใน "แผนก" Middlewares
namespace App\Middlewares;

// นำเครื่องมือที่จำเป็นเข้ามาเตรียมไว้
use Firebase\JWT\JWT;      // เครื่องมือหลักสำหรับสร้างและถอดรหัส Token
use Firebase\JWT\Key;       // เครื่องมือสำหรับจัดการ Secret Key
use App\Core\Database;    // ช่างเชื่อมต่อฐานข้อมูล
use PDO;

/**
 * คลาส AuthMiddleware
 * ด่านตรวจรักษาความปลอดภัย ตรวจสอบ Token และสิทธิ์การเข้าถึง
 */
class AuthMiddleware {
    private $db;
    // ตัวแปรสำหรับเก็บข้อมูลที่ถอดรหัสจาก Token แล้ว เพื่อไม่ต้องถอดรหัสซ้ำซ้อน
    private $decodedToken = null;

    public function __construct() {
        // "เบิกเครื่องมือ" โดยการสร้างการเชื่อมต่อฐานข้อมูลเตรียมไว้
        $this->db = (new Database())->getConnection();
    }

    /**
     * เมธอด authenticate (หัวใจของการตรวจบัตร)
     * เป็นเมธอดภายในที่ใช้ตรวจสอบและถอดรหัส Token
     */
    private function authenticate() {
        // ถ้าเคยตรวจบัตรใบนี้ไปแล้ว ก็ให้ผ่านเลย ไม่ต้องตรวจซ้ำ
        if ($this->decodedToken !== null) return true;

        // 1. ขอดูบัตร: ค้นหา Token จาก Request Header
        $headers = getallheaders();
        $authHeader = $headers['Authorization'] ?? '';

        // ถ้าไม่มี Header 'Authorization' หรือรูปแบบไม่ถูกต้อง (ไม่ใช่ "Bearer [token]")
        if (!$authHeader || !preg_match('/Bearer\s(\S+)/', $authHeader, $matches)) {
            return ['status' => 401, 'message' => 'Authorization token not found.'];
        }

        // 2. ตรวจสอบบัตร: ลองถอดรหัส Token
        try {
            $token = $matches[1];
            // ใช้ Secret Key ของเราในการถอดรหัส ถ้า Key ไม่ตรง หรือ Token หมดอายุ ส่วนนี้จะโยน Error (Exception) ออกมา
            $this->decodedToken = JWT::decode($token, new Key(JWT_SECRET, 'HS256'));

            // 3. เช็คข้อมูลในบัตร: ตรวจสอบว่า User ID ที่อยู่ใน Token ยังมีตัวตนอยู่จริงในฐานข้อมูลหรือไม่
            $stmt = $this->db->prepare("SELECT id FROM users WHERE id = :id");
            $stmt->execute([':id' => $this->decodedToken->sub]);
            if (!$stmt->fetch()) {
                // ถ้าไม่เจอ (เช่น user ถูกลบไปแล้ว) ก็ถือว่าบัตรนี้ใช้ไม่ได้
                return ['status' => 401, 'message' => 'User associated with this token no longer exists.'];
            }

            // ถ้าทุกอย่างถูกต้อง ก็คืนค่า true (อนุญาตให้ผ่าน)
            return true;

        // ถ้าการถอดรหัสล้มเหลว (catch)
        } catch (\Exception $e) {
            return ['status' => 401, 'message' => 'Invalid or expired token.'];
        }
    }

    /**
     * =======================================================================
     * เมธอดสาธารณะ (Public Methods)
     * ที่ Router จะเรียกใช้เพื่อกำหนด "กฎ" ของแต่ละด่านตรวจ
     * =======================================================================
     */

    // กฎข้อที่ 1: "แค่มีบัตรที่ถูกต้องก็พอ" (แค่ล็อกอินมาก็พอ)
    public function isAuthenticated() {
        return $this->authenticate();
    }

    // กฎข้อที่ 2: "ต้องมีตำแหน่งอย่างน้อยตามที่กำหนด"
    public function hasRole($requiredRole) {
        // ตรวจบัตรก่อน
        $authResult = $this->authenticate();
        if ($authResult !== true) return $authResult;

        // สร้างลำดับชั้นของตำแหน่ง (Role Hierarchy)
        $rolesHierarchy = ['customer' => 1, 'employee' => 2, 'manager' => 3, 'director' => 4, 'admin' => 5];
        $userRole = $this->decodedToken->role; // ดูตำแหน่งจากบัตร

        // เปรียบเทียบตำแหน่ง: ถ้าตำแหน่งในบัตรต่ำกว่าที่กำหนด ก็ไม่ให้ผ่าน
        if (!isset($rolesHierarchy[$userRole]) || $rolesHierarchy[$userRole] < $rolesHierarchy[$requiredRole]) {
            return ['status' => 403, 'message' => 'Forbidden. Insufficient permissions.'];
        }
        return true;
    }

    // สร้างเมธอดทางลัดเพื่อง่ายต่อการเรียกใช้ใน routes.php
    public function isAdmin() { return $this->hasRole('admin'); }
    public function isManager() { return $this->hasRole('manager'); }
    public function isEmployee() { return $this->hasRole('employee'); }

    // กฎข้อที่ 3: "ต้องเป็นเจ้าของข้อมูล หรือเป็น Manager ขึ้นไป"
    public function isSelfOrManager($id) {
        $authResult = $this->authenticate();
        if ($authResult !== true) return $authResult;

        // เช็คว่า ID จากบัตร ตรงกับ ID จาก URL หรือไม่
        if ($this->decodedToken->sub == $id) {
            return true; // เป็นเจ้าของเอง ให้ผ่าน
        }

        // ถ้าไม่ใช่เจ้าของ ก็เช็คว่าเป็น Manager หรือไม่
        return $this->isManager();
    }

    // กฎข้อที่ 4: "ต้องเป็นเจ้าของออเดอร์ หรือเป็น Manager ขึ้นไป" (ซับซ้อนขึ้น)
    public function isOrderOwnerOrManager($id) {
        $authResult = $this->authenticate();
        if ($authResult !== true) return $authResult;

        // ต้องไปถามฐานข้อมูลก่อนว่า ออเดอร์ ID นี้ เป็นของใคร
        $stmt = $this->db->prepare("SELECT user_id FROM orders WHERE id = :id");
        $stmt->execute([':id' => $id]);
        $order = $stmt->fetch(PDO::FETCH_ASSOC);

        if (!$order) {
            return ['status' => 404, 'message' => 'Resource not found.'];
        }

        // เช็คว่าเป็นเจ้าของออเดอร์หรือไม่
        if ($order['user_id'] == $this->decodedToken->sub) {
            return true;
        }

        // ถ้าไม่ใช่ ก็เช็คว่าเป็น Manager หรือไม่
        return $this->isManager();
    }
}
?>
```
* **Gatekeeper:** ทำหน้าที่เป็น "ผู้เฝ้าประตู" ให้กับเส้นทาง (Routes) ที่ต้องการความปลอดภัย
* **Decoupling (การแยกส่วน):** แยกตรรกะการตรวจสอบสิทธิ์ที่ซับซ้อนออกมาจาก Controller ทำให้ Controller ของเรามีโค้ดที่สะอาดและสนใจแค่การจัดการข้อมูลเท่านั้น
* **Reusability (การนำกลับมาใช้ใหม่):** เราสามารถสร้างกฎการเข้าถึงที่ซับซ้อน (เช่น `isSelfOrManager`) ไว้ในนี้ที่เดียว แล้วนำไปใช้กับหลายๆ เส้นทางใน `routes.php` ได้อย่างง่ายดาย

---

### 📄`src/Middlewares/RateLimiter.php`
ลองจินตนาการว่า API ของเราคือ "ร้านอาหารบุฟเฟ่ต์" ที่มีโปรโมชันน่าสนใจ ถ้าเราไม่จำกัดจำนวนคนเข้าร้านในแต่ละชั่วโมง อาจจะมีคนแห่กันเข้ามาพร้อมกันเยอะๆ จนครัวทำอาหารไม่ทัน, พนักงานบริการไม่พอ, และสุดท้ายร้านก็ล่ม ลูกค้าคนอื่นๆ ก็จะได้รับประสบการณ์ที่แย่ไปด้วย

`RateLimiter` ก็คือ **"ผู้จัดการร้าน"** ที่คอยควบคุมคิวอยู่หน้าร้านนั่นเองครับ เขามีหน้าที่:

1. **จดบันทึกผู้ใช้บริการ:** เมื่อมีใคร (Request) เดินมาที่หน้าร้าน เขาจะจดไว้ว่าคนนี้ (ระบุด้วย IP Address หรือ User ID) มาใช้บริการที่ "โซน" ไหน (Endpoint ไหน) และมาเมื่อไหร่

2. **นับจำนวนครั้ง:** เขาจะดูในสมุดบันทึก (ตาราง `rate_limits` ในฐานข้อมูล) ว่าคนๆ นี้มาใช้บริการโซนนี้ไปกี่ครั้งแล้วในชั่วโมงที่ผ่านมา

3. **ตัดสินใจ:**

   * **ถ้าน้อยกว่าโควต้า (เช่น 100 ครั้ง/ชั่วโมง):** เขาก็จะอนุญาตให้เข้าร้านได้ พร้อมกับขีดนับจำนวนครั้งเพิ่มในสมุด

   * **ถ้าเต็มโควต้าแล้ว:** เขาก็จะปฏิเสธอย่างสุภาพว่า "ขออภัยครับ ตอนนี้คิวเต็มแล้ว กรุณาลองใหม่อีกครั้งในชั่วโมงถัดไป" (ส่ง Response `429 Too Many Requests`)

4. **รีเซ็ตโควต้า:** เมื่อขึ้นชั่วโมงใหม่ เขาก็จะเริ่มนับจำนวนครั้งของลูกค้าคนนั้นใหม่

การทำแบบนี้ช่วยป้องกันไม่ให้คนคนเดียว (หรือบอท) มายิง Request ถล่ม API ของเราจนล่ม ซึ่งเป็นการรักษาเสถียรภาพของระบบและทำให้ผู้ใช้ทุกคนได้รับบริการที่ดีอย่างเท่าเทียมกันครับ

```php
<?php
// ประกาศว่าไฟล์นี้อยู่ใน "แผนก" Middlewares
namespace App\Middlewares;

// นำเครื่องมือที่จำเป็นเข้ามาเตรียมไว้
use App\Core\Database;    // ช่างเชื่อมต่อฐานข้อมูล
use Firebase\JWT\JWT;      // เครื่องมือสำหรับถอดรหัส Token (เพื่อหา User ID)
use Firebase\JWT\Key;
use DateTime;             // เครื่องมือสำหรับจัดการ วัน-เวลา
use PDO;

/**
 * คลาส RateLimiter
 * ด่านตรวจควบคุมปริมาณการใช้งาน API
 */
class RateLimiter {
    private $db;
    // กำหนด "โควต้า" การใช้งาน
    private $limit = 100; // อนุญาตให้เรียกได้ 100 ครั้ง
    private $window = 3600; // ภายในช่วงเวลา 1 ชั่วโมง (3600 วินาที)

    public function __construct() {
        // "เบิกเครื่องมือ" โดยการสร้างการเชื่อมต่อฐานข้อมูลเตรียมไว้
        $this->db = (new Database())->getConnection();
    }

    /**
     * เมธอด handle (หัวใจของการควบคุมคิว)
     * เมธอดหลักที่จะถูกเรียกโดย Router
     */
    public function handle() {
        // 1. จดบันทึกผู้ใช้บริการ: ระบุตัวตนของผู้ใช้ด้วย IP Address และ Endpoint ที่เขาเรียก
        $ip = $_SERVER['REMOTE_ADDR'];
        $endpoint = parse_url($_SERVER['REQUEST_URI'], PHP_URL_PATH);
        // ลองหา User ID จาก Token ด้วย (ถ้ามี) เพื่อการระบุตัวตนที่แม่นยำขึ้น
        $userId = $this->getUserIdFromToken();

        // 2. ค้นหาสมุดบันทึก: ค้นหาข้อมูลในตาราง `rate_limits`
        $query = "SELECT id, request_count, window_start FROM rate_limits 
                  WHERE ip_address = :ip AND endpoint = :endpoint AND (user_id = :user_id OR (:user_id IS NULL AND user_id IS NULL))";
        
        $stmt = $this->db->prepare($query);
        $stmt->bindValue(':ip', $ip);
        $stmt->bindValue(':endpoint', $endpoint);
        // ถ้ามี userId ก็ใช้ userId ในการค้นหา, ถ้าไม่มี (เป็น Guest) ก็จะค้นหาเฉพาะ IP
        $stmt->bindValue(':user_id', $userId, $userId ? PDO::PARAM_INT : PDO::PARAM_NULL);
        $stmt->execute();
        $rateLimit = $stmt->fetch(PDO::FETCH_ASSOC);
        
        $currentTime = new DateTime(); // เวลาปัจจุบัน

        // 3. ตัดสินใจ:
        // --- กรณีเคยมีบันทึกอยู่แล้ว ---
        if ($rateLimit) {
            $windowStart = new DateTime($rateLimit['window_start']); // เวลาที่เริ่มนับครั้งแรก
            $interval = $currentTime->getTimestamp() - $windowStart->getTimestamp(); // คำนวณว่าผ่านไปกี่วินาทีแล้ว

            // 4. รีเซ็ตโควต้า: ถ้าเวลาผ่านไปเกิน 1 ชั่วโมงแล้ว
            if ($interval > $this->window) {
                // ให้เริ่มนับใหม่ที่ 1 และอัปเดตเวลาเริ่มต้นเป็นเวลาปัจจุบัน
                $stmt = $this->db->prepare("UPDATE rate_limits SET request_count = 1, window_start = :now WHERE id = :id");
                $stmt->execute([':now' => $currentTime->format('Y-m-d H:i:s'), ':id' => $rateLimit['id']]);
            } else {
                // --- กรณีที่ยังไม่ครบชั่วโมง ---
                // ถ้าจำนวนครั้งเกินโควต้าแล้ว
                if ($rateLimit['request_count'] >= $this->limit) {
                    // ปฏิเสธการเข้าใช้บริการ
                    return ['status' => 429, 'message' => 'Too many requests. Please try again later.'];
                }
                // ถ้ายังไม่เกินโควต้า ก็ให้นับเพิ่มไปอีก 1
                $stmt = $this->db->prepare("UPDATE rate_limits SET request_count = request_count + 1 WHERE id = :id");
                $stmt->execute([':id' => $rateLimit['id']]);
            }
        // --- กรณีที่ยังไม่เคยมีบันทึกมาก่อน (มาใช้บริการครั้งแรกในชั่วโมงนี้) ---
        } else {
            // สร้างบันทึกใหม่ โดยเริ่มนับที่ 1
            $stmt = $this->db->prepare("INSERT INTO rate_limits (user_id, ip_address, endpoint, request_count, window_start) 
                                        VALUES (:user_id, :ip, :endpoint, 1, :now)");
            $stmt->bindValue(':user_id', $userId, $userId ? PDO::PARAM_INT : PDO::PARAM_NULL);
            $stmt->bindValue(':ip', $ip);
            $stmt->bindValue(':endpoint', $endpoint);
            $stmt->bindValue(':now', $currentTime->format('Y-m-d H:i:s'));
            $stmt->execute();
        }

        // ถ้าผ่านมาได้ทั้งหมด ก็อนุญาตให้ผ่าน
        return true;
    }

    /**
     * เมธอดภายในสำหรับดึง User ID จาก Token
     * เป็นเมธอดเสริมที่ช่วยให้การระบุตัวตนแม่นยำขึ้น
     */
    private function getUserIdFromToken() {
        $authHeader = getallheaders()['Authorization'] ?? '';
        if (preg_match('/Bearer\s(\S+)/', $authHeader, $matches)) {
            try {
                // ลองถอดรหัส Token ถ้าสำเร็จก็คืนค่า User ID กลับไป
                return JWT::decode($matches[1], new Key(JWT_SECRET, 'HS256'))->sub;
            } catch (\Exception $e) {
                // ถ้า Token ไม่ถูกต้อง ก็คืนค่า null
                return null;
            }
        }
        return null;
    }
}
?>
```
### สรุปหัวใจสำคัญของ `RateLimiter.php`

* **Stability (ความเสถียร):** เป็นเครื่องมือสำคัญที่ช่วยป้องกันไม่ให้เซิร์ฟเวอร์ทำงานหนักเกินไปจาก Request ที่เข้ามาพร้อมกันจำนวนมาก ทำให้ API ของเรามีความเสถียรและพร้อมใช้งานอยู่เสมอ
* **Security (ความปลอดภัย):** ช่วยลดความเสี่ยงจากการถูกโจมตีแบบ **Denial-of-Service (DoS)** หรือการพยายามสุ่มรหัสผ่าน (Brute Force Attack)
* **Fair Usage (การใช้งานที่เท่าเทียม):** ทำให้มั่นใจได้ว่าผู้ใช้ทุกคนจะได้รับโอกาสในการเข้าถึงทรัพยากรของ API อย่างเท่าเทียมกัน ไม่ใช่ใครคนใดคนหนึ่งมาใช้งานจนหมด

---

### 5.4 โค้ดทั้งหมดสำหรับ Controllers และ Models 

นี่คือโค้ดทั้งหมดสำหรับส่วนควบคุมและจัดการข้อมูล พร้อมคำอธิบายการทำงานในแต่ละส่วนครับ

#### **Authentication**

### 📄`src/Controllers/AuthController.php`
ถ้า `Router` คือผู้จัดการที่คอยจ่ายงาน และ `Middleware` คือด่านตรวจ `Controller` ก็เปรียบเสมือน **"หัวหน้าแผนก"** ที่รับเรื่องต่อมาเพื่อดำเนินการให้สำเร็จลุล่วง สำหรับ `AuthController` นั้น หน้าที่ของเขาก็คือ **"หัวหน้าแผนกออกบัตรพนักงาน"** นั่นเองครับ

กระบวนการทำงานของแผนกนี้ตรงไปตรงมา แต่สำคัญมาก:

1. **รับคำร้อง (Receive Request):** เมื่อมีคนเดินมาที่แผนกพร้อมยื่น "ใบคำร้องขอทำบัตร" (ซึ่งก็คือ `email` และ `password` ที่ผู้ใช้กรอกมาในรูปแบบ JSON) หัวหน้าแผนกก็จะรับเรื่องไว้

2. **ตรวจสอบข้อมูล (Validate & Verify):**

   * ขั้นแรก, เขาจะดูก่อนว่ากรอกข้อมูลในใบคำร้องมาครบถ้วนหรือไม่
   * จากนั้น, เขาจะนำ `email` ไปค้นใน "แฟ้มประวัติพนักงาน" (ตาราง `users` ในฐานข้อมูล)
   * ถ้าเจอ, เขาจะนำ `password` ที่ได้มา ไปเทียบกับรหัสผ่านที่ถูกเข้ารหัสเก็บไว้ในแฟ้ม โดยใช้เครื่องมือพิเศษ (`password_verify`) เพื่อดูว่าตรงกันหรือไม่

3. **ออกบัตร (Issue Token):**

   * ถ้าข้อมูลทุกอย่างถูกต้อง, เขาจะสร้าง "บัตรพนักงานดิจิทัล" (JWT Token) ใบใหม่ขึ้นมา
   * ในบัตรนั้นจะระบุข้อมูลสำคัญ เช่น "บัตรนี้ออกโดยบริษัท one.com", "หมดอายุใน 24 ชั่วโมง", "เจ้าของบัตรคือพนักงาน ID ที่ 1", และ "ตำแหน่ง Customer"
   * สุดท้าย, เขาจะ "ประทับลายเซ็นลับ" (`JWT_SECRET`) ลงบนบัตร เพื่อป้องกันการปลอมแปลง

4. **ส่งมอบบัตร (Send Response):** เขายื่นบัตร (Token) ที่สร้างเสร็จแล้วกลับไปให้ผู้ใช้ พร้อมกับบอกว่า "ล็อกอินสำเร็จ!"

ถ้าขั้นตอนไหนผิดพลาด เช่น หา `email` ไม่เจอ หรือรหัสผ่านไม่ตรงกัน เขาก็จะปฏิเสธคำร้องและแจ้งกลับไปว่า "ข้อมูลไม่ถูกต้อง"

```php
<?php
// ประกาศว่าไฟล์นี้อยู่ใน "แผนก" Controllers
namespace App\Controllers;

// นำเครื่องมือที่จำเป็นเข้ามาเตรียมไว้
use Firebase\JWT\JWT;      // เครื่องมือสำหรับสร้าง Token
use App\Core\Database;    // ช่างเชื่อมต่อฐานข้อมูล
use PDO;

/**
 * คลาส AuthController
 * หัวหน้าแผนกออกบัตรพนักงาน (จัดการการยืนยันตัวตน)
 */
class AuthController {
    // ตัวแปรสำหรับเก็บการเชื่อมต่อฐานข้อมูล
    private $db;

    public function __construct() {
        // เมื่อแผนกนี้เริ่มทำงาน ก็ให้ไป "เบิกเครื่องมือ" คือการเชื่อมต่อฐานข้อมูลมาเตรียมไว้เลย
        $this->db = (new Database())->getConnection();
    }

    /**
     * เมธอด login
     * คือกระบวนการทำงานหลักของแผนกนี้
     */
    public function login() {
        // 1. รับคำร้อง: อ่านข้อมูล JSON ที่ผู้ใช้ส่งมา (email, password)
        $data = json_decode(file_get_contents("php://input"));

        // 2. ตรวจสอบข้อมูล (เบื้องต้น): เช็คว่ากรอกข้อมูลมาครบหรือไม่
        if (empty($data->email) || empty($data->password)) {
            http_response_code(400); // Bad Request
            echo json_encode(['message' => 'Email and password are required.']);
            return; // หยุดการทำงาน
        }

        // 2. ตรวจสอบข้อมูล (เชิงลึก): ค้นหาผู้ใช้ในฐานข้อมูลด้วย email
        $stmt = $this->db->prepare("SELECT id, password, role FROM users WHERE email = :email");
        $stmt->execute([':email' => $data->email]);
        $user = $stmt->fetch(PDO::FETCH_ASSOC);

        // ใช้ password_verify() เพื่อเปรียบเทียบรหัสผ่านที่ผู้ใช้กรอก (plain text)
        // กับรหัสผ่านที่ถูกเข้ารหัสไว้ในฐานข้อมูล (hashed password)
        // ฟังก์ชันนี้ปลอดภัยและเป็นวิธีที่ถูกต้องในการตรวจสอบรหัสผ่าน
        if ($user && password_verify($data->password, $user['password'])) {
            // --- ถ้าข้อมูลถูกต้อง ---

            // 3. ออกบัตร: เตรียมข้อมูลที่จะใส่ลงในบัตร (Payload)
            $payload = [
                'iss' => JWT_ISSUER,          // Issuer: ใครเป็นคนออกบัตร
                'aud' => JWT_AUDIENCE,        // Audience: บัตรนี้ใช้กับใคร
                'iat' => time(),              // Issued At: เวลาที่ออกบัตร
                'exp' => time() + (60*60*24), // Expiration Time: เวลาหมดอายุ (24 ชั่วโมง)
                'sub' => $user['id'],         // Subject: บัตรนี้เป็นของใคร (User ID)
                'role' => $user['role']       // ข้อมูลเพิ่มเติม: ตำแหน่งอะไร
            ];

            // "ประทับลายเซ็นลับ" ลงบนบัตร โดยใช้ JWT::encode
            $jwt = JWT::encode($payload, JWT_SECRET, 'HS256');

            // 4. ส่งมอบบัตร: ส่ง Token กลับไปให้ผู้ใช้
            http_response_code(200); // OK
            echo json_encode(['token' => $jwt, 'message' => 'Login successful.']);
        } else {
            // --- ถ้าข้อมูลไม่ถูกต้อง ---
            // ปฏิเสธคำร้อง
            http_response_code(401); // Unauthorized
            echo json_encode(['message' => 'Invalid credentials.']);
        }
    }
}
?>
```
#### สรุปหัวใจสำคัญของ `AuthController.php`

* **Single Responsibility:** Controller นี้ทำหน้าที่เดียวคือ "การยืนยันตัวตน" ไม่ไปยุ่งกับเรื่องการสร้างหรือแก้ไขข้อมูลผู้ใช้ ซึ่งเป็นหน้าที่ของ `UserController` ทำให้โค้ดเป็นระเบียบ
* **Stateless Authentication:** การใช้ JWT Token ทำให้การยืนยันตัวตนของเราเป็นแบบ "Stateless" คือเซิร์ฟเวอร์ไม่จำเป็นต้องจำว่าใครล็อกอินอยู่บ้าง แค่ตรวจสอบความถูกต้องของ Token ที่ส่งมาในแต่ละครั้งก็พอ ซึ่งดีต่อการขยายระบบในอนาคต
* **Clear Flow:** คุณจะเห็นการทำงานที่เป็นเส้นตรงชัดเจน: **รับข้อมูล -> ตรวจสอบ -> ประมวลผล (สร้าง Token) -> ส่งผลลัพธ์** ซึ่งเป็นรูปแบบพื้นฐานของ Controller ทั่วไป

---

#### **Users** แผนกบุคคลและหัวหน้าแผนก

### 📄`src/Models/User.php` (พนักงานฝ่ายบุคคล - คนทำงาน)

มาเจาะลึกการทำงานของคู่หูที่ทำงานร่วมกันอย่างใกล้ชิด นั่นคือ `UserController` (หัวหน้าแผนก) และ `User` Model (พนักงานในแผนก) ครับ สองไฟล์นี้รับผิดชอบทุกอย่างที่เกี่ยวกับข้อมูลผู้ใช้ ตั้งแต่การสร้าง, อ่าน, แก้ไข, ไปจนถึงการลบข้อมูล

#### แนวคิดการทำงาน: หัวหน้า vs. ลูกน้อง

เพื่อให้เห็นภาพง่ายขึ้น ให้ลองนึกภาพตามนี้ครับ:

* **`UserController.php` (หัวหน้าแผนกบุคคล):**

  * เป็นคนรับ "คำสั่ง" มาจากผู้จัดการใหญ่ (`Router`)
  * **ไม่ทำงานกับเอกสาร (ฐานข้อมูล) โดยตรง** แต่จะสั่งงานลูกน้องในแผนก
  * มีหน้าที่รับข้อมูลที่ลูกน้องหามาได้ แล้วนำไปจัดรูปแบบเพื่อ "รายงาน" กลับไปให้ผู้ใช้
  * เป็นคนตัดสินใจว่าจะตอบกลับผู้ใช้อย่างไร (เช่น ตอบ `200 OK` หรือ `404 Not Found`)

* **`src/Models/User.php` (พนักงานฝ่ายบุคคล):**

  * เป็น "คนทำงาน" ที่เชี่ยวชาญเรื่องแฟ้มเอกสาร `users` ที่สุด
  * เป็นคนเดียวที่ได้รับอนุญาตให้เดินเข้าไปใน "ห้องเก็บเอกสาร" (ฐานข้อมูล) เพื่อเขียน, อ่าน, แก้ไข, หรือทำลายเอกสาร (`users` table)
  * มีหน้าที่รับคำสั่งจากหัวหน้า แล้วไปจัดการกับเอกสารให้เรียบร้อย จากนั้นนำผลลัพธ์กลับไปรายงานหัวหน้า

การแบ่งหน้าที่ชัดเจนแบบนี้ ทำให้หัวหน้าไม่ต้องกังวลกับรายละเอียดหยุมหยิมของการจัดการเอกสาร และพนักงานก็ไม่ต้องไปยุ่งกับเรื่องการสื่อสารกับคนนอกแผนก ทำให้ทั้งสองฝ่ายทำงานของตัวเองได้อย่างเต็มประสิทธิภาพครับ


```php
<?php
// ประกาศว่าไฟล์นี้อยู่ใน "แผนก" Models
namespace App\Models;

// นำเครื่องมือที่จำเป็นเข้ามา
use App\Core\Database; // ช่างเชื่อมต่อฐานข้อมูล
use PDO;             // เครื่องมือมาตรฐานสำหรับคุยกับฐานข้อมูล

/**
 * คลาส User (Model)
 * พนักงานฝ่ายบุคคลที่รับผิดชอบตาราง 'users' โดยเฉพาะ
 */
class User {
    // ตัวแปรสำหรับเก็บการเชื่อมต่อฐานข้อมูล
    private $db;

    public function __construct() {
        // "เบิกเครื่องมือ" คือการเชื่อมต่อฐานข้อมูลมาเตรียมไว้
        $this->db = (new Database())->getConnection();
    }

    /**
     * เมธอด getAll
     * หน้าที่: ไปเอา "เอกสารประวัติพนักงาน" มาทั้งหมดทุกใบ
     */
    public function getAll() {
        // เขียนคำสั่ง SQL เพื่อดึงข้อมูลที่ต้องการ (ไม่เอา password มาเพื่อความปลอดภัย)
        $query = "SELECT id, username, email, first_name, last_name, role, created_at FROM users";
        // ส่งคำสั่งไปทำงานและนำผลลัพธ์ทั้งหมดกลับมา
        return $this->db->query($query)->fetchAll(PDO::FETCH_ASSOC);
    }

    /**
     * เมธอด getById
     * หน้าที่: ไปหา "เอกสารประวัติพนักงาน" มา 1 ใบตามเลข ID ที่หัวหน้าสั่ง
     */
    public function getById($id) {
        $stmt = $this->db->prepare("SELECT id, username, email, first_name, last_name, role, created_at FROM users WHERE id = :id");
        // ใช้ prepare + execute เพื่อป้องกัน SQL Injection
        $stmt->execute([':id' => $id]);
        return $stmt->fetch(PDO::FETCH_ASSOC);
    }

    /**
     * เมธอด create
     * หน้าที่: สร้าง "เอกสารพนักงานใหม่" 1 ใบจากข้อมูลที่หัวหน้าให้มา
     */
    public function create($data) {
        // ตรวจสอบข้อมูลเบื้องต้น
        if (empty($data->username) || empty($data->email) || empty($data->password)) {
            return ['status' => 400, 'message' => 'Username, email, and password are required.'];
        }
        $sql = "INSERT INTO users (username, email, password, first_name, last_name, role) VALUES (:username, :email, :password, :first_name, :last_name, :role)";
        $stmt = $this->db->prepare($sql);
        try {
            $stmt->execute([
                // ใช้ htmlspecialchars และ strip_tags เพื่อป้องกันการโจมตีแบบ XSS
                ':username' => htmlspecialchars(strip_tags($data->username)),
                ':email' => htmlspecialchars(strip_tags($data->email)),
                // เข้ารหัสผ่านด้วยวิธีที่ปลอดภัยที่สุดก่อนเก็บลงฐานข้อมูล
                ':password' => password_hash($data->password, PASSWORD_BCRYPT),
                ':first_name' => $data->first_name ?? null,
                ':last_name' => $data->last_name ?? null,
                ':role' => $data->role ?? 'customer'
            ]);
            // รายงานผลกลับไปให้หัวหน้า
            return ['status' => 201, 'message' => 'User created successfully.'];
        } catch (\PDOException $e) {
            // ถ้าเกิด Error (เช่น username ซ้ำ) ก็รายงานกลับไป
            return ['status' => 409, 'message' => 'Username or email already exists.'];
        }
    }

    /**
     * เมธอด update
     * หน้าที่: แก้ไขข้อมูลใน "เอกสารพนักงาน" ที่มีอยู่
     */
    public function update($id, $data) {
        $sql = "UPDATE users SET first_name = :first_name, last_name = :last_name, role = :role WHERE id = :id";
        $stmt = $this->db->prepare($sql);
        $stmt->execute([':id' => $id, ':first_name' => $data->first_name, ':last_name' => $data->last_name, ':role' => $data->role]);
        return ['status' => 200, 'message' => 'User updated successfully.'];
    }

    /**
     * เมธอด delete
     * หน้าที่: ทำลาย "เอกสารพนักงาน" ทิ้ง
     */
    public function delete($id) {
        $stmt = $this->db->prepare("DELETE FROM users WHERE id = :id");
        $stmt->execute([':id' => $id]);
        // ตรวจสอบว่ามีเอกสารถูกทำลายจริงหรือไม่ แล้วรายงานกลับ
        return $stmt->rowCount() > 0 ? ['status' => 200, 'message' => 'User deleted successfully.'] : ['status' => 404, 'message' => 'User not found.'];
    }

    /**
     * เมธอด checkExistence
     * หน้าที่: ตรวจสอบว่ามี "เอกสาร" ที่มี username หรือ email นี้อยู่แล้วหรือไม่
     */
    public function checkExistence($data) {
        $field = isset($data->username) ? 'username' : 'email';
        $value = $data->username ?? $data->email;
        if (empty($value)) return ['status' => 400, 'message' => 'Username or email is required.'];

        $stmt = $this->db->prepare("SELECT id FROM users WHERE $field = :value");
        $stmt->execute([':value' => $value]);
        return $stmt->fetch() ? ['status' => 409, 'message' => ucfirst($field) . ' already exists.'] : ['status' => 200, 'message' => ucfirst($field) . ' is available.'];
    }

    /**
     * เมธอด changePassword
     * หน้าที่: เปลี่ยนรหัสผ่านในเอกสาร
     */
    public function changePassword($id, $data) {
        if (empty($data->old_password) || empty($data->new_password)) {
            return ['status' => 400, 'message' => 'Old and new passwords are required.'];
        }
        // ต้องไปเอารหัสผ่านเก่ามาดูก่อน
        $stmt = $this->db->prepare("SELECT password FROM users WHERE id = :id");
        $stmt->execute([':id' => $id]);
        $user = $stmt->fetch(PDO::FETCH_ASSOC);
        // ตรวจสอบว่ารหัสผ่านเก่าที่ให้มาถูกต้องหรือไม่
        if (!$user || !password_verify($data->old_password, $user['password'])) {
            return ['status' => 401, 'message' => 'Invalid old password.'];
        }
        // ถ้ารหัสเก่าถูก ก็อัปเดตเป็นรหัสใหม่ที่เข้ารหัสแล้ว
        $stmt = $this->db->prepare("UPDATE users SET password = :password WHERE id = :id");
        $stmt->execute([':password' => password_hash($data->new_password, PASSWORD_BCRYPT), ':id' => $id]);
        return ['status' => 200, 'message' => 'Password changed successfully.'];
    }
}
?>
```

### 📄`src/Controllers/UserController.php` (หัวหน้าแผนกบุคคล)

```php
<?php
namespace App\Controllers;
use App\Models\User;

/**
 * คลาส UserController
 * หัวหน้าแผนกบุคคลที่รับคำสั่งจาก Router และสั่งงาน Model
 */
class UserController {
    // "ลูกน้อง" ประจำตัวของหัวหน้าคนนี้
    private $model;

    public function __construct() {
        // "จ้าง" พนักงานฝ่ายบุคคล (User Model) มาเตรียมไว้
        $this->model = new User();
    }

    /**
     * เมธอด index
     * รับคำสั่ง: "ไปเอาข้อมูลพนักงานมาทั้งหมด"
     */
    public function index() {
        // 1. สั่งงานลูกน้อง
        $users = $this->model->getAll();
        // 2. นำผลลัพธ์มารายงานกลับ
        echo json_encode($users);
    }

    /**
     * เมธอด show
     * รับคำสั่ง: "ไปเอาข้อมูลพนักงาน ID ที่ [id] มา"
     */
    public function show($params) {
        // 1. สั่งงานลูกน้อง พร้อมบอก ID ที่ต้องการ
        $user = $this->model->getById($params['id']);
        // 2. ตัดสินใจว่าจะรายงานกลับอย่างไร
        if ($user) {
            // ถ้ารายงานสำเร็จ
            echo json_encode($user);
        } else {
            // ถ้าลูกน้อง báo cáo ว่าหาไม่เจอ
            http_response_code(404); // Not Found
            echo json_encode(['message' => 'User not found.']);
        }
    }

    /**
     * เมธอด create
     * รับคำสั่ง: "สร้างพนักงานใหม่ตามข้อมูลนี้"
     */
    public function create() {
        // 1. รับข้อมูลคำร้องมาจากผู้ใช้
        $data = json_decode(file_get_contents("php://input"));
        // 2. ส่งข้อมูลให้ลูกน้องไปจัดการ
        $result = $this->model->create($data);
        // 3. นำผลลัพธ์ (สำเร็จหรือล้มเหลว) มารายงานกลับ
        http_response_code($result['status']);
        echo json_encode(['message' => $result['message']]);
    }

    // (เมธอด update, delete, checkExistence, changePassword จะใช้หลักการเดียวกัน:
    // รับคำสั่ง -> ส่งให้ Model -> นำผลลัพธ์มารายงานกลับ)

    public function update($params) {
        $data = json_decode(file_get_contents("php://input"));
        $result = $this->model->update($params['id'], $data);
        http_response_code($result['status']);
        echo json_encode(['message' => $result['message']]);
    }

    public function delete($params) {
        $result = $this->model->delete($params['id']);
        http_response_code($result['status']);
        echo json_encode(['message' => $result['message']]);
    }

    public function checkExistence() {
        $data = json_decode(file_get_contents("php://input"));
        $result = $this->model->checkExistence($data);
        http_response_code($result['status']);
        echo json_encode(['message' => $result['message']]);
    }

    public function changePassword($params) {
        $data = json_decode(file_get_contents("php://input"));
        $result = $this->model->changePassword($params['id'], $data);
        http_response_code($result['status']);
        echo json_encode(['message' => $result['message']]);
    }
}
?>
```
#### สรุปหัวใจสำคัญของ Model และ Controller

* **Model คือ "คนทำงาน" (The Worker):**

  * สนใจแค่การคุยกับฐานข้อมูล
  * มีหน้าที่จัดการข้อมูล (CRUD)
  * ต้องมีความปลอดภัยสูง (ป้องกัน SQL Injection, XSS, เข้ารหัสผ่าน)

* **Controller คือ "หัวหน้างาน" (The Manager):**

  * สนใจแค่การสื่อสารระหว่าง "โลกภายนอก" (Router) กับ "ลูกน้อง" (Model)
  * มีหน้าที่รับ Request, สั่งงาน Model, และส่ง Response กลับ
  * ทำให้โค้ดส่วนตรรกะทางธุรกิจ (Business Logic) แยกออกจากส่วนจัดการข้อมูลอย่างชัดเจน

การเข้าใจความสัมพันธ์แบบ "หัวหน้า-ลูกน้อง" นี้ คือกุญแจสำคัญในการทำความเข้าใจสถาปัตยกรรม MVC และเป็นรากฐานในการสร้างแอปพลิเคชันที่มีขนาดใหญ่และซับซ้อนขึ้นในอนาคตครับ

---

#### **Categories**
Category.php & CategoryController.php: แผนกจัดเรียงสินค้า
เราจะมาดูการทำงานของอีกหนึ่งคู่หูสำคัญ นั่นคือ `CategoryController` (หัวหน้าแผนก) และ `Category` Model (พนักงาน) ครับ สองไฟล์นี้มีหน้าที่รับผิดชอบทุกอย่างที่เกี่ยวกับ "หมวดหมู่สินค้า" (Categories) เช่น "เครื่องใช้ไฟฟ้า", "เสื้อผ้า", "หนังสือ" เป็นต้น

##### แนวคิดการทำงาน: ยังคงเป็น "หัวหน้า vs. ลูกน้อง"

หลักการทำงานของแผนกนี้เหมือนกับ "แผนกบุคคล" ที่เราดูกันไปก่อนหน้าเป๊ะๆ เลยครับ แค่เปลี่ยนเรื่องที่รับผิดชอบเท่านั้นเอง:

* **`CategoryController.php` (หัวหน้าแผนกจัดเรียงสินค้า):**

  * รับคำสั่งมาจากผู้จัดการใหญ่ (`Router`) เช่น "ขอรายชื่อหมวดหมู่ทั้งหมด" หรือ "เพิ่มหมวดหมู่ใหม่ชื่อ 'ของเล่น'"
  * **ไม่ยุ่งกับแฟ้มเอกสารโดยตรง** แต่จะสั่งงานลูกน้องในแผนก
  * รับผลลัพธ์จากลูกน้อง แล้วนำมารายงานกลับไปให้ผู้ใช้ในรูปแบบ JSON

* **`src/Models/Category.php` (พนักงานแผนกจัดเรียงสินค้า):**

  * เป็น "คนทำงาน" ที่เชี่ยวชาญเรื่อง "แฟ้มหมวดหมู่สินค้า" (ตาราง `categories`) ที่สุด
  * มีหน้าที่รับคำสั่งจากหัวหน้า แล้วไปจัดการกับแฟ้มเอกสาร (ฐานข้อมูล) ไม่ว่าจะเป็นการสร้าง, อ่าน, แก้ไข, หรือลบหมวดหมู่
  * เมื่อทำงานเสร็จ ก็จะนำผลลัพธ์กลับไปรายงานหัวหน้า

การใช้หลักการเดียวกันซ้ำๆ แบบนี้ คือหัวใจของการสร้างโค้ดที่ดีครับ เพราะมันทำให้โปรแกรมเมอร์คนอื่น (หรือแม้แต่ตัวเราในอนาคต) สามารถเข้ามาอ่านและทำความเข้าใจโค้ดส่วนอื่นๆ ได้อย่างรวดเร็ว

### 📄 `src/Models/Category.php` (พนักงานจัดเรียงสินค้า - คนทำงาน)

```php
<?php
// ประกาศว่าไฟล์นี้อยู่ใน "แผนก" Models
namespace App\Models;

// นำเครื่องมือที่จำเป็นเข้ามา
use App\Core\Database; // ช่างเชื่อมต่อฐานข้อมูล
use PDO;             // เครื่องมือมาตรฐานสำหรับคุยกับฐานข้อมูล

/**
 * คลาส Category (Model)
 * พนักงานที่รับผิดชอบตาราง 'categories' โดยเฉพาะ
 */
class Category {
    // ตัวแปรสำหรับเก็บการเชื่อมต่อฐานข้อมูล
    private $db;

    public function __construct() {
        // "เบิกเครื่องมือ" คือการเชื่อมต่อฐานข้อมูลมาเตรียมไว้
        $this->db = (new Database())->getConnection();
    }

    /**
     * เมธอด getAll
     * หน้าที่: ไปเอา "แฟ้มหมวดหมู่" มาทั้งหมดทุกใบ
     */
    public function getAll() {
        // เขียนคำสั่ง SQL เพื่อดึงข้อมูลทั้งหมดจากตาราง categories และเรียงตามชื่อ A-Z
        return $this->db->query("SELECT * FROM categories ORDER BY name ASC")->fetchAll(PDO::FETCH_ASSOC);
    }

    /**
     * เมธอด getById
     * หน้าที่: ไปหา "แฟ้มหมวดหมู่" มา 1 ใบตามเลข ID ที่หัวหน้าสั่ง
     */
    public function getById($id) {
        $stmt = $this->db->prepare("SELECT * FROM categories WHERE id = :id");
        $stmt->execute([':id' => $id]);
        return $stmt->fetch(PDO::FETCH_ASSOC);
    }

    /**
     * เมธอด create
     * หน้าที่: สร้าง "แฟ้มหมวดหมู่ใหม่" 1 ใบจากข้อมูลที่หัวหน้าให้มา
     */
    public function create($data) {
        $sql = "INSERT INTO categories (name, description) VALUES (:name, :description)";
        $stmt = $this->db->prepare($sql);
        // ใช้ htmlspecialchars เพื่อแปลงอักขระพิเศษเป็นโค้ด HTML ป้องกันการโจมตีแบบ XSS
        $stmt->execute([
            ':name' => htmlspecialchars(strip_tags($data->name)),
            ':description' => htmlspecialchars(strip_tags($data->description ?? ''))
        ]);
        // รายงานผลกลับไปให้หัวหน้า
        return ['status' => 201, 'message' => 'Category created successfully.'];
    }

    /**
     * เมธอด update
     * หน้าที่: แก้ไขข้อมูลใน "แฟ้มหมวดหมู่" ที่มีอยู่
     */
    public function update($id, $data) {
        $sql = "UPDATE categories SET name = :name, description = :description WHERE id = :id";
        $stmt = $this->db->prepare($sql);
        $stmt->execute([
            ':id' => $id,
            ':name' => htmlspecialchars(strip_tags($data->name)),
            ':description' => htmlspecialchars(strip_tags($data->description ?? ''))
        ]);
        return ['status' => 200, 'message' => 'Category updated successfully.'];
    }

    /**
     * เมธอด delete
     * หน้าที่: ทำลาย "แฟ้มหมวดหมู่" ทิ้ง
     */
    public function delete($id) {
        $stmt = $this->db->prepare("DELETE FROM categories WHERE id = :id");
        $stmt->execute([':id' => $id]);
        // ตรวจสอบว่ามีแฟ้มถูกทำลายจริงหรือไม่ แล้วรายงานกลับ
        return $stmt->rowCount() > 0 ? ['status' => 200, 'message' => 'Category deleted successfully.'] : ['status' => 404, 'message' => 'Category not found.'];
    }
}
?>

```

### 📄`src/Controllers/CategoryController.php` (หัวหน้าแผนกจัดเรียงสินค้า)

```php
<?php
namespace App\Controllers;
use App\Models\Category;

/**
 * คลาส CategoryController
 * หัวหน้าแผนกที่รับคำสั่งเกี่ยวกับหมวดหมู่ และสั่งงาน Category Model
 */
class CategoryController {
    // "ลูกน้อง" ประจำตัวของหัวหน้าคนนี้
    private $model;

    public function __construct() {
        // "จ้าง" พนักงาน (Category Model) มาเตรียมไว้
        $this->model = new Category();
    }

    /**
     * เมธอด index
     * รับคำสั่ง: "ขอรายชื่อหมวดหมู่ทั้งหมด"
     */
    public function index() {
        // 1. สั่งงานลูกน้อง
        $categories = $this->model->getAll();
        // 2. นำผลลัพธ์มารายงานกลับ
        echo json_encode($categories);
    }

    /**
     * เมธอด show
     * รับคำสั่ง: "ขอข้อมูลหมวดหมู่ ID ที่ [id]"
     */
    public function show($params) {
        // 1. สั่งงานลูกน้อง พร้อมบอก ID ที่ต้องการ
        $category = $this->model->getById($params['id']);
        // 2. ตัดสินใจว่าจะรายงานกลับอย่างไร
        if ($category) {
            echo json_encode($category);
        } else {
            http_response_code(404); // Not Found
            echo json_encode(['message' => 'Category not found.']);
        }
    }

    /**
     * เมธอด create
     * รับคำสั่ง: "สร้างหมวดหมู่ใหม่ตามข้อมูลนี้"
     */
    public function create() {
        // 1. รับข้อมูลคำร้องมาจากผู้ใช้
        $data = json_decode(file_get_contents("php://input"));
        // 2. ส่งข้อมูลให้ลูกน้องไปจัดการ
        $result = $this->model->create($data);
        // 3. นำผลลัพธ์มารายงานกลับ
        http_response_code($result['status']);
        echo json_encode(['message' => $result['message']]);
    }

    // (เมธอด update และ delete ก็ใช้หลักการเดียวกัน)

    public function update($params) {
        $data = json_decode(file_get_contents("php://input"));
        $result = $this->model->update($params['id'], $data);
        http_response_code($result['status']);
        echo json_encode(['message' => $result['message']]);
    }

    public function delete($params) {
        $result = $this->model->delete($params['id']);
        http_response_code($result['status']);
        echo json_encode(['message' => $result['message']]);
    }
}
?>
?>
```
### สรุปหัวใจสำคัญ

คุณจะเห็นว่าโครงสร้างและหลักการทำงานของ `CategoryController` และ `Category` Model นั้นแทบจะเหมือนกับของ `User` ทุกประการ สิ่งนี้คือข้อดีของการออกแบบสถาปัตยกรรมที่ดีครับ:

* **Consistency (ความสม่ำเสมอ):** เมื่อเราเข้าใจการทำงานของส่วนหนึ่งแล้ว เราก็จะสามารถเข้าใจการทำงานของส่วนอื่นๆ ได้อย่างรวดเร็ว
* **Scalability (การขยายระบบ):** ถ้าในอนาคตเราต้องการเพิ่ม "แผนก" ใหม่ เช่น "แผนกจัดการโปรโมชัน" เราก็แค่สร้าง `PromotionController.php` และ `Promotion.php` ขึ้นมาตามรูปแบบเดิมได้เลย ทำให้การต่อเติมระบบทำได้ง่ายและรวดเร็ว

---

#### **Products**
Product.php & ProductController.php: แผนกจัดการสต็อกสินค้า
อีกหนึ่งแผนกที่คึกคักที่สุดในบริษัทของเรา นั่นก็คือ **"แผนกจัดการสต็อกสินค้า"** ซึ่งรับผิดชอบโดยคู่หู `ProductController` (หัวหน้าแผนก) และ `Product` Model (พนักงาน) ครับ สองไฟล์นี้มีหน้าที่ดูแลทุกอย่างที่เกี่ยวกับ "สินค้า" ของเราโดยตรง ตั้งแต่การเพิ่มสินค้าใหม่, เช็คสต็อก, แก้ไขราคา, ไปจนถึงการนำสินค้าออกจากชั้นวาง

#### แนวคิดการทำงาน: หลักการเดิม เพิ่มเติมคือความสัมพันธ์

แน่นอนว่าแผนกนี้ยังคงใช้หลักการ "หัวหน้า vs. ลูกน้อง" ที่เราคุ้นเคยกันดี:

* **`ProductController.php` (หัวหน้าแผนกสต็อก):**

  * รับคำสั่งจาก `Router` เช่น "ขอดูรายการสินค้าทั้งหมด" หรือ "แก้ไขราคาสินค้า ID ที่ 1"
  * สั่งงานต่อให้ลูกน้องในแผนก (`Product` Model)
  * รับรายงานจากลูกน้อง แล้วจัดทำเป็นเอกสาร JSON ส่งกลับให้ผู้ใช้

* **`src/Models/Product.php` (พนักงานแผนกสต็อก):**

  * เป็นผู้เชี่ยวชาญ "แฟ้มข้อมูลสินค้า" (ตาราง `products`)
  * เป็นคนเดียวที่ลงมือทำงานกับฐานข้อมูลโดยตรง
  * **ความสามารถพิเศษ:** พนักงานคนนี้ไม่ได้ดูแค่แฟ้มของตัวเอง แต่ยังสามารถเดินไปขอดูข้อมูลจาก "แผนกจัดเรียงสินค้า" (`categories` table) ได้ด้วย เพื่อนำชื่อหมวดหมู่มาประกอบรายงานให้สมบูรณ์ (นี่คือการใช้ `JOIN` ใน SQL ครับ)

การที่พนักงานแผนกสต็อกสามารถทำงานร่วมกับแผนกอื่นได้ ทำให้รายงานที่หัวหน้าได้รับนั้นสมบูรณ์และมีประโยชน์มากขึ้น โดยที่หัวหน้าไม่ต้องเสียเวลาไปประสานงานเองเลย

### 📄`src/Models/Product.php` (พนักงานแผนกสต็อก - คนทำงาน)

```php
<?php
// ประกาศว่าไฟล์นี้อยู่ใน "แผนก" Models
namespace App\Models;

// นำเครื่องมือที่จำเป็นเข้ามา
use App\Core\Database; // ช่างเชื่อมต่อฐานข้อมูล
use PDO;             // เครื่องมือมาตรฐานสำหรับคุยกับฐานข้อมูล

/**
 * คลาส Product (Model)
 * พนักงานที่รับผิดชอบตาราง 'products'
 */
class Product {
    // ตัวแปรสำหรับเก็บการเชื่อมต่อฐานข้อมูล
    private $db;

    public function __construct() {
        // "เบิกเครื่องมือ" คือการเชื่อมต่อฐานข้อมูลมาเตรียมไว้
        $this->db = (new Database())->getConnection();
    }

    /**
     * เมธอด getAll
     * หน้าที่: ไปเอา "แฟ้มสินค้า" มาทั้งหมด พร้อมทั้งเดินไปถาม "ชื่อหมวดหมู่" จากแผนกข้างๆ มาด้วย
     */
    public function getAll() {
        // นี่คือ SQL ที่มีความสามารถพิเศษ:
        // SELECT p.*, c.name as category_name: เลือกทุกคอลัมน์จากตาราง products (p) และเอาคอลัมน์ name จากตาราง categories (c) มาด้วย โดยจะเปลี่ยนชื่อเป็น category_name
        // FROM products p LEFT JOIN categories c ON p.category_id = c.id:
        // คือการนำแฟ้ม products (p) มา "เชื่อม" กับแฟ้ม categories (c) โดยใช้ "เลขที่อ้างอิง" (category_id) เป็นตัวเชื่อม
        $sql = "SELECT p.*, c.name as category_name FROM products p LEFT JOIN categories c ON p.category_id = c.id ORDER BY p.name ASC";
        return $this->db->query($sql)->fetchAll(PDO::FETCH_ASSOC);
    }

    /**
     * เมธอด getById
     * หน้าที่: ไปหา "แฟ้มสินค้า" 1 ใบ พร้อมข้อมูลหมวดหมู่
     */
    public function getById($id) {
        $stmt = $this->db->prepare("SELECT p.*, c.name as category_name FROM products p LEFT JOIN categories c ON p.category_id = c.id WHERE p.id = :id");
        $stmt->execute([':id' => $id]);
        return $stmt->fetch(PDO::FETCH_ASSOC);
    }

    /**
     * เมธอด create
     * หน้าที่: สร้าง "แฟ้มสินค้าใหม่" 1 ใบ
     */
    public function create($data) {
        $sql = "INSERT INTO products (name, price, description, category_id, stock) VALUES (:name, :price, :description, :category_id, :stock)";
        $stmt = $this->db->prepare($sql);
        $stmt->execute([
            ':name' => htmlspecialchars(strip_tags($data->name)),
            ':price' => $data->price,
            ':description' => htmlspecialchars(strip_tags($data->description ?? '')),
            ':category_id' => $data->category_id,
            ':stock' => $data->stock ?? 0
        ]);
        return ['status' => 201, 'message' => 'Product created successfully.'];
    }

    /**
     * เมธอด update
     * หน้าที่: แก้ไขข้อมูลใน "แฟ้มสินค้า"
     */
    public function update($id, $data) {
        $sql = "UPDATE products SET name = :name, price = :price, description = :description, category_id = :category_id, stock = :stock WHERE id = :id";
        $stmt = $this->db->prepare($sql);
        $stmt->execute([
            ':id' => $id,
            ':name' => htmlspecialchars(strip_tags($data->name)),
            ':price' => $data->price,
            ':description' => htmlspecialchars(strip_tags($data->description ?? '')),
            ':category_id' => $data->category_id,
            ':stock' => $data->stock ?? 0
        ]);
        return ['status' => 200, 'message' => 'Product updated successfully.'];
    }

    /**
     * เมธอด delete
     * หน้าที่: ทำลาย "แฟ้มสินค้า" ทิ้ง
     */
    public function delete($id) {
        $stmt = $this->db->prepare("DELETE FROM products WHERE id = :id");
        $stmt->execute([':id' => $id]);
        return $stmt->rowCount() > 0 ? ['status' => 200, 'message' => 'Product deleted successfully.'] : ['status' => 404, 'message' => 'Product not found.'];
    }
}
?>
```

### 📄`src/Controllers/ProductController.php` (หัวหน้าแผนกสต็อก)

```php
<?php
namespace App\Controllers;
use App\Models\Product;

/**
 * คลาส ProductController
 * หัวหน้าแผนกที่รับคำสั่งเกี่ยวกับสินค้า และสั่งงาน Product Model
 */
class ProductController {
    // "ลูกน้อง" ประจำตัวของหัวหน้าคนนี้
    private $model;

    public function __construct() {
        // "จ้าง" พนักงาน (Product Model) มาเตรียมไว้
        $this->model = new Product();
    }

    /**
     * เมธอด index
     * รับคำสั่ง: "ขอรายการสินค้าทั้งหมด"
     */
    public function index() {
        // 1. สั่งงานลูกน้อง: "ไปเอาข้อมูลสินค้ามาทั้งหมดเลยนะ"
        $products = $this->model->getAll();
        // 2. นำผลลัพธ์มารายงานกลับ
        echo json_encode($products);
    }

    /**
     * เมธอด show
     * รับคำสั่ง: "ขอข้อมูลสินค้า ID ที่ [id]"
     */
    public function show($params) {
        // 1. สั่งงานลูกน้อง: "ไปเอาข้อมูลสินค้า ID นี้มาหน่อย"
        $product = $this->model->getById($params['id']);
        // 2. ตัดสินใจว่าจะรายงานกลับอย่างไร
        if ($product) {
            echo json_encode($product);
        } else {
            http_response_code(404); // Not Found
            echo json_encode(['message' => 'Product not found.']);
        }
    }

    /**
     * เมธอด create
     * รับคำสั่ง: "เพิ่มสินค้าใหม่ตามข้อมูลนี้"
     */
    public function create() {
        // 1. รับข้อมูลคำร้องมาจากผู้ใช้
        $data = json_decode(file_get_contents("php://input"));
        // 2. ส่งข้อมูลให้ลูกน้องไปจัดการ
        $result = $this->model->create($data);
        // 3. นำผลลัพธ์มารายงานกลับ
        http_response_code($result['status']);
        echo json_encode(['message' => $result['message']]);
    }

    // (เมธอด update และ delete ก็ใช้หลักการเดียวกัน)

    public function update($params) {
        $data = json_decode(file_get_contents("php://input"));
        $result = $this->model->update($params['id'], $data);
        http_response_code($result['status']);
        echo json_encode(['message' => $result['message']]);
    }

    public function delete($params) {
        $result = $this->model->delete($params['id']);
        http_response_code($result['status']);
        echo json_encode(['message' => $result['message']]);
    }
}
?>
```
#### สรุป

* **Reusability (การนำกลับมาใช้ใหม่):** คุณจะเห็นว่าโครงสร้างของ Model และ Controller สำหรับ Product นั้นเหมือนกับของ Category และ User มาก นี่คือพลังของการออกแบบที่ดี เราสามารถสร้าง "พิมพ์เขียว" การทำงานขึ้นมาครั้งเดียว แล้วนำไปใช้กับข้อมูลส่วนอื่นๆ ได้อย่างรวดเร็ว

* **Data Relationships (ความสัมพันธ์ของข้อมูล):** `Product` Model แสดงให้เห็นถึงการดึงข้อมูลจากตารางที่เกี่ยวข้องกัน (`categories`) ด้วยคำสั่ง `JOIN` ซึ่งเป็นเทคนิคที่สำคัญมากในการทำงานกับฐานข้อมูลเชิงสัมพันธ์ ทำให้เราสามารถแสดงข้อมูลที่สมบูรณ์และมีความหมายมากขึ้นใน Response เดียว

เมื่อเราเข้าใจหลักการของแผนกนี้แล้ว การจะทำความเข้าใจแผนกอื่นๆ ที่เหลือ เช่น `Order` หรือ `Review` ก็จะกลายเป็นเรื่องง่ายทันทีครับ!

---

#### **Orders**
Order.php & OrderController.php: แผนกรับออเดอร์และประสานงาน
เดินทางมาถึงแผนกที่เป็นศูนย์กลางของการดำเนินงานเลยก็ว่าได้ นั่นคือ **"แผนกรับออเดอร์และประสานงาน"** ซึ่งดูแลโดยคู่หู `OrderController` และ `Order` Model ครับ แผนกนี้รับผิดชอบการสร้างและจัดการ "ใบสั่งซื้อ" (Orders) ทั้งหมดของลูกค้า

### แนวคิดการทำงาน: ความซับซ้อนที่เพิ่มขึ้นและการประสานงาน

แผนกนี้ยังคงใช้หลักการ "หัวหน้า vs. ลูกน้อง" เหมือนเดิม แต่มีความท้าทายเพิ่มขึ้นมา คือ **ต้องทำงานร่วมกับแผนกอื่นอยู่เสมอ**

* **`OrderController.php` (หัวหน้าแผนกรับออเดอร์):**

  * รับคำสั่งจาก `Router` เช่น "มีลูกค้าสร้างออเดอร์ใหม่" หรือ "ขอดูรายละเอียดออเดอร์หมายเลข 5"
  * สั่งงานลูกน้องในแผนก (`Order` Model)
  * เป็นคนประสานงานหลัก รับรายงานจากลูกน้องแล้วส่งกลับให้ผู้ใช้

* **`src/Models/Order.php` (พนักงานแผนกรับออเดอร์):**

  * เป็นผู้เชี่ยวชาญ "แฟ้มใบสั่งซื้อ" (ตาราง `orders`)
  * **ความสามารถพิเศษ:** พนักงานคนนี้ต้องวิ่งไปประสานงานกับแผนกอื่นเสมอ เช่น:
    * ต้องไปที่ **"แผนกบุคคล"** (`users` table) เพื่อดูว่า "ใครคือเจ้าของใบสั่งซื้อนี้"
  * การทำงานที่ต้องเชื่อมโยงกับข้อมูลจากหลายๆ แฟ้ม ทำให้พนักงานแผนกนี้ต้องมีความสามารถในการใช้ `JOIN` ใน SQL อย่างคล่องแคล่ว เพื่อรวบรวมข้อมูลทั้งหมดมาไว้ในรายงานฉบับเดียว

การจัดการความสัมพันธ์ที่ซับซ้อนนี้ คือจุดเด่นที่แสดงให้เห็นว่าทำไมสถาปัตยกรรม MVC ถึงมีประสิทธิภาพในการจัดการระบบใหญ่ๆ ครับ

### 📄 `src/Models/Order.php`(พนักงานแผนกรับออเดอร์ - คนทำงาน)

```php
<?php
// ประกาศว่าไฟล์นี้อยู่ใน "แผนก" Models
namespace App\Models;

// นำเครื่องมือที่จำเป็นเข้ามา
use App\Core\Database; // ช่างเชื่อมต่อฐานข้อมูล
use PDO;             // เครื่องมือมาตรฐานสำหรับคุยกับฐานข้อมูล

/**
 * คลาส Order (Model)
 * พนักงานที่รับผิดชอบตาราง 'orders' และประสานงานกับตารางอื่น
 */
class Order {
    // ตัวแปรสำหรับเก็บการเชื่อมต่อฐานข้อมูล
    private $db;

    public function __construct() {
        // "เบิกเครื่องมือ" คือการเชื่อมต่อฐานข้อมูลมาเตรียมไว้
        $this->db = (new Database())->getConnection();
    }

    /**
     * เมธอด getAll
     * หน้าที่: ไปเอา "แฟ้มใบสั่งซื้อ" มาทั้งหมด พร้อมชื่อลูกค้าจากแผนกบุคคล
     */
    public function getAll() {
        // ใช้ JOIN เพื่อเชื่อมตาราง orders (o) กับ users (u)
        // เพื่อดึง username ของเจ้าของออเดอร์มาแสดงด้วย
        $sql = "SELECT o.*, u.username FROM orders o JOIN users u ON o.user_id = u.id ORDER BY o.created_at DESC";
        return $this->db->query($sql)->fetchAll(PDO::FETCH_ASSOC);
    }

    /**
     * เมธอด getById
     * หน้าที่: ไปหา "แฟ้มใบสั่งซื้อ" 1 ใบ พร้อมชื่อลูกค้า
     */
    public function getById($id) {
        $stmt = $this->db->prepare("SELECT o.*, u.username FROM orders o JOIN users u ON o.user_id = u.id WHERE o.id = :id");
        $stmt->execute([':id' => $id]);
        return $stmt->fetch(PDO::FETCH_ASSOC);
    }

    /**
     * เมธอด create
     * หน้าที่: สร้าง "แฟ้มใบสั่งซื้อใหม่" 1 ใบ
     */
    public function create($data) {
        // ตรวจสอบข้อมูลที่จำเป็นสำหรับการสร้างออเดอร์
        if (empty($data->user_id) || !isset($data->total_amount)) {
            return ['status' => 400, 'message' => 'User ID and total amount are required.'];
        }
        $sql = "INSERT INTO orders (user_id, total_amount, status) VALUES (:user_id, :total_amount, :status)";
        $stmt = $this->db->prepare($sql);
        $stmt->execute([
            ':user_id' => $data->user_id,
            ':total_amount' => $data->total_amount,
            ':status' => $data->status ?? 'pending' // ถ้าไม่ระบุสถานะมา ให้เป็น 'pending'
        ]);
        return ['status' => 201, 'message' => 'Order created successfully.'];
    }

    /**
     * เมธอด update
     * หน้าที่: แก้ไขข้อมูลใน "แฟ้มใบสั่งซื้อ" (ส่วนใหญ่คือการเปลี่ยนสถานะ)
     */
    public function update($id, $data) {
        if (!isset($data->total_amount) || empty($data->status)) {
            return ['status' => 400, 'message' => 'Total amount and status are required for update.'];
        }
        $sql = "UPDATE orders SET total_amount = :total_amount, status = :status WHERE id = :id";
        $stmt = $this->db->prepare($sql);
        $stmt->execute([
            ':id' => $id,
            ':total_amount' => $data->total_amount,
            ':status' => $data->status
        ]);
        return ['status' => 200, 'message' => 'Order updated successfully.'];
    }

    /**
     * เมธอด delete
     * หน้าที่: ทำลาย "แฟ้มใบสั่งซื้อ" ทิ้ง (และข้อมูลที่เกี่ยวข้องจะถูกลบตามไปด้วย เพราะกฎ ON DELETE CASCADE)
     */
    public function delete($id) {
        $stmt = $this->db->prepare("DELETE FROM orders WHERE id = :id");
        $stmt->execute([':id' => $id]);
        return $stmt->rowCount() > 0 ? ['status' => 200, 'message' => 'Order deleted successfully.'] : ['status' => 404, 'message' => 'Order not found.'];
    }
}
?>
```

### 📄 `src/Controllers/OrderController.php` (หัวหน้าแผนกรับออเดอร์)

```php
<?php
namespace App\Controllers;
use App\Models\Order;

/**
 * คลาส OrderController
 * หัวหน้าแผนกที่รับคำสั่งเกี่ยวกับออเดอร์ และสั่งงาน Order Model
 */
class OrderController {
    // "ลูกน้อง" ประจำตัวของหัวหน้าคนนี้
    private $model;

    public function __construct() {
        // "จ้าง" พนักงาน (Order Model) มาเตรียมไว้
        $this->model = new Order();
    }

    /**
     * เมธอด index
     * รับคำสั่ง: "ขอรายการออเดอร์ทั้งหมด"
     */
    public function index() {
        // 1. สั่งงานลูกน้อง: "ไปเอาข้อมูลออเดอร์มาทั้งหมด"
        $orders = $this->model->getAll();
        // 2. นำผลลัพธ์มารายงานกลับ
        echo json_encode($orders);
    }

    /**
     * เมธอด show
     * รับคำสั่ง: "ขอข้อมูลออเดอร์ ID ที่ [id]"
     */
    public function show($params) {
        // 1. สั่งงานลูกน้อง: "ไปเอาข้อมูลออเดอร์ ID นี้มา"
        $order = $this->model->getById($params['id']);
        // 2. ตัดสินใจว่าจะรายงานกลับอย่างไร
        if ($order) {
            echo json_encode($order);
        } else {
            http_response_code(404); // Not Found
            echo json_encode(['message' => 'Order not found.']);
        }
    }

    /**
     * เมธอด create
     * รับคำสั่ง: "สร้างออเดอร์ใหม่ตามข้อมูลนี้"
     */
    public function create() {
        // 1. รับข้อมูลคำร้องมาจากผู้ใช้
        $data = json_decode(file_get_contents("php://input"));
        // 2. ส่งข้อมูลให้ลูกน้องไปจัดการ
        $result = $this->model->create($data);
        // 3. นำผลลัพธ์มารายงานกลับ
        http_response_code($result['status']);
        echo json_encode(['message' => $result['message']]);
    }

    // (เมธอด update และ delete ก็ใช้หลักการเดียวกัน)

    public function update($params) {
        $data = json_decode(file_get_contents("php://input"));
        $result = $this->model->update($params['id'], $data);
        http_response_code($result['status']);
        echo json_encode(['message' => $result['message']]);
    }

    public function delete($params) {
        $result = $this->model->delete($params['id']);
        http_response_code($result['status']);
        echo json_encode(['message' => $result['message']]);
    }
}
?>
```
* **Handling Complexity:** `Order` Model เป็นตัวอย่างที่ดีของการจัดการความสัมพันธ์ระหว่างข้อมูลหลายส่วน (Orders และ Users) การใช้ `JOIN` ช่วยให้เราสามารถดึงข้อมูลที่สมบูรณ์มาได้ภายในการ query ครั้งเดียว ซึ่งมีประสิทธิภาพมากกว่าการ query แยกกันหลายๆ ครั้ง
* **Business Logic in Models:** การตรวจสอบข้อมูลเบื้องต้น (เช่น `user_id` ต้องไม่ว่าง) เกิดขึ้นใน Model ซึ่งเป็นที่ที่เหมาะสมที่สุด เพราะ Model คือผู้ที่รู้ดีที่สุดว่าข้อมูลของตัวเองต้องการอะไรบ้าง
* **Consistency is Key:** โครงสร้างของ `OrderController` ยังคงเหมือนกับ Controller อื่นๆ ทุกประการ ซึ่งตอกย้ำถึงความดีงามของการมี "พิมพ์เขียว" ที่ชัดเจนในการทำงาน
การทำความเข้าใจแผนกนี้จะช่วยให้คุณเห็นภาพรวมของระบบ E-commerce ที่สมบูรณ์มากยิ่งขึ้น และเป็นพื้นฐานในการสร้างแผนกอื่นๆ ที่ต้องมีการประสานงานกันต่อไปครับ!

---

#### **Order Items**
OrderItem.php & OrderItemController.php: แผนกจัดของลงกล่อง
มาถึงแผนกสุดท้ายในกระบวนการสั่งซื้อ นั่นคือ **"แผนกจัดของลงกล่อง"** หรือ `OrderItem` ครับ แผนกนี้เปรียบเสมือน "พนักงาน QC" ที่คอยบันทึกรายละเอียดของสินค้าแต่ละชิ้นที่ถูกเพิ่มเข้ามาในใบสั่งซื้อ (Order)

#### แนวคิดการทำงาน: ตารางเชื่อม (Junction Table) และการประสานงานสามฝ่าย

แผนกนี้มีความพิเศษตรงที่เป็น **"แผนกประสานงานกลาง"** ที่ต้องคุยกับแผนกอื่นถึง 2 แผนกพร้อมกัน:

* **`OrderItemController.php` (หัวหน้าแผนกจัดของ):**
  * รับคำสั่งจาก `Router` เช่น "ลูกค้าเพิ่ม 'Laptop' 1 เครื่องลงในออเดอร์หมายเลข 5"
  * สั่งงานลูกน้องในแผนก (`OrderItem` Model)
* **`src/Models/OrderItem.php` (พนักงานจัดของ):**
  * เป็นผู้เชี่ยวชาญ "แฟ้มรายการสินค้าในออเดอร์" (ตาราง `order_items`)
  * **ความสามารถพิเศษ:** พนักงานคนนี้ต้องวิ่งไปประสานงานกับ 2 แผนกพร้อมกัน:
    1. ไปที่ **"แผนกรับออเดอร์"** (`orders` table) เพื่อดูว่ารายการสินค้านี้อยู่ใน "ใบสั่งซื้อ" ใบไหน
    2. ไปที่ **"แผนกสต็อกสินค้า"** (`products` table) เพื่อดูว่าสินค้านี้คืออะไร และดึง "ชื่อสินค้า" มาประกอบรายงาน
  * ตาราง `order_items` นี้ทำหน้าที่เป็น **"ตารางเชื่อม" (Junction Table)** ที่เชื่อมความสัมพันธ์แบบ Many-to-Many ระหว่าง `orders` และ `products` (คือ 1 ออเดอร์ มีได้หลายสินค้า และ 1 สินค้า ก็สามารถอยู่ในหลายออเดอร์ได้)

การมีแผนกนี้ทำให้เราสามารถเก็บ "ประวัติ" การซื้อขายได้อย่างสมบูรณ์ เช่น แม้ว่าในอนาคตสินค้าชิ้นนั้นจะเปลี่ยนราคาไปแล้ว แต่ใน `order_items` จะยังคงบันทึกราคา ณ วันที่ลูกค้ากดสั่งซื้อไว้เสมอ

### 📄 `src/Models/OrderItem.php` (พนักงานจัดของ - คนทำงาน)

```php
<?php
// ประกาศว่าไฟล์นี้อยู่ใน "แผนก" Models
namespace App\Models;

// นำเครื่องมือที่จำเป็นเข้ามา
use App\Core\Database; // ช่างเชื่อมต่อฐานข้อมูล
use PDO;             // เครื่องมือมาตรฐานสำหรับคุยกับฐานข้อมูล

/**
 * คลาส OrderItem (Model)
 * พนักงานที่รับผิดชอบตาราง 'order_items' และประสานงานกับตารางอื่น
 */
class OrderItem {
    // ตัวแปรสำหรับเก็บการเชื่อมต่อฐานข้อมูล
    private $db;

    public function __construct() {
        // "เบิกเครื่องมือ" คือการเชื่อมต่อฐานข้อมูลมาเตรียมไว้
        $this->db = (new Database())->getConnection();
    }

    /**
     * เมธอด getAll
     * หน้าที่: ไปเอา "แฟ้มรายการสินค้า" มาทั้งหมด พร้อมชื่อสินค้าจากแผนกสต็อก
     */
    public function getAll() {
        // ใช้ JOIN เพื่อเชื่อมตาราง order_items (oi) กับ products (p)
        // เพื่อดึงชื่อสินค้า (product_name) มาแสดงด้วย
        $sql = "SELECT oi.*, p.name as product_name FROM order_items oi LEFT JOIN products p ON oi.product_id = p.id ORDER BY oi.id ASC";
        return $this->db->query($sql)->fetchAll(PDO::FETCH_ASSOC);
    }

    /**
     * เมธอด getById
     * หน้าที่: ไปหา "แฟ้มรายการสินค้า" 1 ใบ พร้อมชื่อสินค้า
     */
    public function getById($id) {
        $stmt = $this->db->prepare("SELECT oi.*, p.name as product_name FROM order_items oi LEFT JOIN products p ON oi.product_id = p.id WHERE oi.id = :id");
        $stmt->execute([':id' => $id]);
        return $stmt->fetch(PDO::FETCH_ASSOC);
    }

    /**
     * เมธอด create
     * หน้าที่: สร้าง "แฟ้มรายการสินค้าใหม่" 1 ใบ (เพิ่มของลงกล่อง)
     */
    public function create($data) {
        // ตรวจสอบข้อมูลที่จำเป็น
        if (empty($data->order_id) || empty($data->product_id) || empty($data->quantity) || !isset($data->unit_price)) {
            return ['status' => 400, 'message' => 'Order ID, Product ID, quantity, and unit price are required.'];
        }
        $sql = "INSERT INTO order_items (order_id, product_id, quantity, unit_price) VALUES (:order_id, :product_id, :quantity, :unit_price)";
        $stmt = $this->db->prepare($sql);
        $stmt->execute([
            ':order_id' => $data->order_id,
            ':product_id' => $data->product_id,
            ':quantity' => $data->quantity,
            ':unit_price' => $data->unit_price // บันทึกราคา ณ ตอนที่ซื้อ
        ]);
        return ['status' => 201, 'message' => 'Order item created successfully.'];
    }

    /**
     * เมธอด update
     * หน้าที่: แก้ไขข้อมูลใน "แฟ้มรายการสินค้า" (เช่น เปลี่ยนจำนวนสินค้า)
     */
    public function update($id, $data) {
        if (empty($data->quantity) || !isset($data->unit_price)) {
            return ['status' => 400, 'message' => 'Quantity and unit price are required.'];
        }
        $sql = "UPDATE order_items SET quantity = :quantity, unit_price = :unit_price WHERE id = :id";
        $stmt = $this->db->prepare($sql);
        $stmt->execute([
            ':id' => $id,
            ':quantity' => $data->quantity,
            ':unit_price' => $data->unit_price
        ]);
        return ['status' => 200, 'message' => 'Order item updated successfully.'];
    }

    /**
     * เมธอด delete
     * หน้าที่: ทำลาย "แฟ้มรายการสินค้า" ทิ้ง (เอาของออกจากกล่อง)
     */
    public function delete($id) {
        $stmt = $this->db->prepare("DELETE FROM order_items WHERE id = :id");
        $stmt->execute([':id' => $id]);
        return $stmt->rowCount() > 0 ? ['status' => 200, 'message' => 'Order item deleted successfully.'] : ['status' => 404, 'message' => 'Order item not found.'];
    }
}
?>
```

### 📄 `src/Controllers/OrderItemController.php` (หัวหน้าแผนกจัดของ)

```php
<?php
namespace App\Controllers;
use App\Models\OrderItem;

/**
 * คลาส OrderItemController
 * หัวหน้าแผนกที่รับคำสั่งเกี่ยวกับรายการสินค้าในออเดอร์ และสั่งงาน OrderItem Model
 */
class OrderItemController {
    // "ลูกน้อง" ประจำตัวของหัวหน้าคนนี้
    private $model;

    public function __construct() {
        // "จ้าง" พนักงาน (OrderItem Model) มาเตรียมไว้
        $this->model = new OrderItem();
    }

    /**
     * เมธอด index
     * รับคำสั่ง: "ขอรายการสินค้าในออเดอร์ทั้งหมด"
     */
    public function index() {
        // 1. สั่งงานลูกน้อง
        $items = $this->model->getAll();
        // 2. นำผลลัพธ์มารายงานกลับ
        echo json_encode($items);
    }

    /**
     * เมธอด show
     * รับคำสั่ง: "ขอข้อมูลรายการสินค้า ID ที่ [id]"
     */
    public function show($params) {
        // 1. สั่งงานลูกน้อง
        $item = $this->model->getById($params['id']);
        // 2. ตัดสินใจว่าจะรายงานกลับอย่างไร
        if ($item) {
            echo json_encode($item);
        } else {
            http_response_code(404); // Not Found
            echo json_encode(['message' => 'Order Item not found.']);
        }
    }

    /**
     * เมธอด create
     * รับคำสั่ง: "เพิ่มของลงกล่องตามข้อมูลนี้"
     */
    public function create() {
        // 1. รับข้อมูลคำร้องมาจากผู้ใช้
        $data = json_decode(file_get_contents("php://input"));
        // 2. ส่งข้อมูลให้ลูกน้องไปจัดการ
        $result = $this->model->create($data);
        // 3. นำผลลัพธ์มารายงานกลับ
        http_response_code($result['status']);
        echo json_encode(['message' => $result['message']]);
    }

    // (เมธอด update และ delete ก็ใช้หลักการเดียวกัน)

    public function update($params) {
        $data = json_decode(file_get_contents("php://input"));
        $result = $this->model->update($params['id'], $data);
        http_response_code($result['status']);
        echo json_encode(['message' => $result['message']]);
    }

    public function delete($params) {
        $result = $this->model->delete($params['id']);
        http_response_code($result['status']);
        echo json_encode(['message' => $result['message']]);
    }
}
?>
```
* **Junction Table:** `order_items` คือตัวอย่างที่สมบูรณ์แบบของ "ตารางเชื่อม" ที่ใช้แก้ปัญหาความสัมพันธ์แบบ "กลุ่มต่อกลุ่ม" (Many-to-Many) ซึ่งเป็นแนวคิดพื้นฐานที่สำคัญมากในการออกแบบฐานข้อมูล
* **Data Snapshot:** การที่เราเก็บ `unit_price` ไว้ในตาราง `order_items` โดยตรง แทนที่จะไปดึงราคาล่าสุดจากตาราง `products` ทุกครั้ง เป็นเทคนิคที่เรียกว่าการทำ "Snapshot" ข้อมูล ณ เวลาที่เกิด ซึ่งสำคัญมากสำหรับระบบที่ต้องการความถูกต้องของข้อมูลย้อนหลัง เช่น ระบบบัญชี หรือ E-commerce
* **Building Blocks:** คุณจะเห็นว่าเมื่อเรามี "บล็อก" พื้นฐาน (เช่น Product Model, Order Model) ที่แข็งแรงแล้ว การจะสร้าง "บล็อก" ใหม่ที่ต้องทำงานร่วมกับบล็อกเดิม (อย่าง OrderItem Model) ก็จะทำได้ง่ายและเป็นระบบ

---

#### **Addresses**
Address.php & AddressController.php: แผนกจัดส่งและดูแลข้อมูลที่อยู่
เราจะมาทำความรู้จักกับอีกหนึ่งแผนกสนับสนุนที่สำคัญ นั่นคือ **"แผนกจัดส่งและดูแลข้อมูลที่อยู่"** ซึ่งรับผิดชอบโดยคู่หู `AddressController` และ `Address` Model ครับ แผนกนี้มีหน้าที่จัดการ "สมุดที่อยู่" ของลูกค้าทั้งหมด

##### แนวคิดการทำงาน: แผนกสนับสนุนที่เชื่อมโยงกับบุคคล

แผนกนี้ทำงานเหมือนแผนกอื่นๆ ที่เราเคยเห็นมา แต่มีความสัมพันธ์ที่ชัดเจนและเรียบง่ายกับ "แผนกบุคคล" (`users` table):

* **`AddressController.php` (หัวหน้าแผนกจัดส่ง):**
  * รับคำสั่งจาก `Router` เช่น "ลูกค้า ID ที่ 1 ต้องการเพิ่มที่อยู่ใหม่" หรือ "ขอดูที่อยู่ทั้งหมดของลูกค้า ID ที่ 1"
  * สั่งงานลูกน้องในแผนก (`Address` Model)

* **`src/Models/Address.php` (พนักงานแผนกจัดส่ง):**
  * เป็นผู้เชี่ยวชาญ "แฟ้มข้อมูลที่อยู่" (ตาราง `addresses`)
  * **ความสามารถพิเศษ:** พนักงานคนนี้จะรู้เสมอว่าที่อยู่แต่ละแห่งเป็นของ "พนักงาน" คนไหนใน "แผนกบุคคล" (เชื่อมโยงกับ `users` table ผ่าน `user_id`)

ความสัมพันธ์ของแผนกนี้เป็นแบบ **One-to-Many** คือ "พนักงาน 1 คน (`User`) สามารถมีได้หลายที่อยู่ (`Address`)" ซึ่งเป็นรูปแบบความสัมพันธ์ที่พบบ่อยที่สุดในการออกแบบฐานข้อมูลครับ

### 📄 `src/Models/Address.php` (พนักงานแผนกจัดส่ง - คนทำงาน)

```php
<?php
// ประกาศว่าไฟล์นี้อยู่ใน "แผนก" Models
namespace App\Models;

// นำเครื่องมือที่จำเป็นเข้ามา
use App\Core\Database; // ช่างเชื่อมต่อฐานข้อมูล
use PDO;             // เครื่องมือมาตรฐานสำหรับคุยกับฐานข้อมูล

/**
 * คลาส Address (Model)
 * พนักงานที่รับผิดชอบตาราง 'addresses'
 */
class Address {
    // ตัวแปรสำหรับเก็บการเชื่อมต่อฐานข้อมูล
    private $db;

    public function __construct() {
        // "เบิกเครื่องมือ" คือการเชื่อมต่อฐานข้อมูลมาเตรียมไว้
        $this->db = (new Database())->getConnection();
    }

    /**
     * เมธอด getAll
     * หน้าที่: ไปเอา "แฟ้มที่อยู่" มาทั้งหมด
     */
    public function getAll() {
        return $this->db->query("SELECT * FROM addresses")->fetchAll(PDO::FETCH_ASSOC);
    }

    /**
     * เมธอด getById
     * หน้าที่: ไปหา "แฟ้มที่อยู่" 1 ใบตามเลข ID ที่หัวหน้าสั่ง
     */
    public function getById($id) {
        $stmt = $this->db->prepare("SELECT * FROM addresses WHERE id = :id");
        $stmt->execute([':id' => $id]);
        return $stmt->fetch(PDO::FETCH_ASSOC);
    }

    /**
     * เมธอด create
     * หน้าที่: สร้าง "แฟ้มที่อยู่ใหม่" 1 ใบ
     */
    public function create($data) {
        // ตรวจสอบข้อมูลที่จำเป็นสำหรับการสร้างที่อยู่
        if (empty($data->user_id) || empty($data->address_line) || empty($data->city) || empty($data->postal_code) || empty($data->country)) {
            return ['status' => 400, 'message' => 'All address fields are required.'];
        }
        $sql = "INSERT INTO addresses (user_id, address_line, city, postal_code, country) VALUES (:user_id, :address_line, :city, :postal_code, :country)";
        $stmt = $this->db->prepare($sql);
        $stmt->execute([
            ':user_id' => $data->user_id,
            ':address_line' => htmlspecialchars(strip_tags($data->address_line)),
            ':city' => htmlspecialchars(strip_tags($data->city)),
            ':postal_code' => htmlspecialchars(strip_tags($data->postal_code)),
            ':country' => htmlspecialchars(strip_tags($data->country))
        ]);
        return ['status' => 201, 'message' => 'Address created successfully.'];
    }

    /**
     * เมธอด update
     * หน้าที่: แก้ไขข้อมูลใน "แฟ้มที่อยู่"
     */
    public function update($id, $data) {
        if (empty($data->address_line) || empty($data->city) || empty($data->postal_code) || empty($data->country)) {
            return ['status' => 400, 'message' => 'All address fields are required for update.'];
        }
        $sql = "UPDATE addresses SET address_line = :address_line, city = :city, postal_code = :postal_code, country = :country WHERE id = :id";
        $stmt = $this->db->prepare($sql);
        $stmt->execute([
            ':id' => $id,
            ':address_line' => htmlspecialchars(strip_tags($data->address_line)),
            ':city' => htmlspecialchars(strip_tags($data->city)),
            ':postal_code' => htmlspecialchars(strip_tags($data->postal_code)),
            ':country' => htmlspecialchars(strip_tags($data->country))
        ]);
        return ['status' => 200, 'message' => 'Address updated successfully.'];
    }

    /**
     * เมธอด delete
     * หน้าที่: ทำลาย "แฟ้มที่อยู่" ทิ้ง
     */
    public function delete($id) {
        $stmt = $this->db->prepare("DELETE FROM addresses WHERE id = :id");
        $stmt->execute([':id' => $id]);
        return $stmt->rowCount() > 0 ? ['status' => 200, 'message' => 'Address deleted successfully.'] : ['status' => 404, 'message' => 'Address not found.'];
    }
}
?>
```

### 📄 `src/Controllers/AddressController.php` (หัวหน้าแผนกจัดส่ง)

```php
<?php
namespace App\Controllers;
use App\Models\Address;

/**
 * คลาส AddressController
 * หัวหน้าแผนกที่รับคำสั่งเกี่ยวกับที่อยู่ และสั่งงาน Address Model
 */
class AddressController {
    // "ลูกน้อง" ประจำตัวของหัวหน้าคนนี้
    private $model;

    public function __construct() {
        // "จ้าง" พนักงาน (Address Model) มาเตรียมไว้
        $this->model = new Address();
    }

    /**
     * เมธอด index
     * รับคำสั่ง: "ขอรายชื่อที่อยู่ทั้งหมด"
     */
    public function index() {
        // 1. สั่งงานลูกน้อง
        $addresses = $this->model->getAll();
        // 2. นำผลลัพธ์มารายงานกลับ
        echo json_encode($addresses);
    }

    /**
     * เมธอด show
     * รับคำสั่ง: "ขอข้อมูลที่อยู่ ID ที่ [id]"
     */
    public function show($params) {
        // 1. สั่งงานลูกน้อง
        $address = $this->model->getById($params['id']);
        // 2. ตัดสินใจว่าจะรายงานกลับอย่างไร
        if ($address) {
            echo json_encode($address);
        } else {
            http_response_code(404); // Not Found
            echo json_encode(['message' => 'Address not found.']);
        }
    }

    /**
     * เมธอด create
     * รับคำสั่ง: "สร้างที่อยู่ใหม่ตามข้อมูลนี้"
     */
    public function create() {
        // 1. รับข้อมูลคำร้องมาจากผู้ใช้
        $data = json_decode(file_get_contents("php://input"));
        // 2. ส่งข้อมูลให้ลูกน้องไปจัดการ
        $result = $this->model->create($data);
        // 3. นำผลลัพธ์มารายงานกลับ
        http_response_code($result['status']);
        echo json_encode(['message' => $result['message']);
    }

    // (เมธอด update และ delete ก็ใช้หลักการเดียวกัน)

    public function update($params) {
        $data = json_decode(file_get_contents("php://input"));
        $result = $this->model->update($params['id'], $data);
        http_response_code($result['status']);
        echo json_encode(['message' => $result['message']]);
    }

    public function delete($params) {
        $result = $this->model->delete($params['id']);
        http_response_code($result['status']);
        echo json_encode(['message' => $result['message']]);
    }
}
?>
```
* **Data Integrity:** ตาราง `addresses` มีความสัมพันธ์กับตาราง `users` ผ่าน `user_id` ในฐานข้อมูล เราได้ตั้งกฎ `ON DELETE SET NULL` ไว้ ซึ่งหมายความว่า "ถ้าผู้ใช้ถูกลบออกจากระบบ ที่อยู่ที่เคยเป็นของเขาจะไม่ถูกลบตามไปด้วย แต่จะถูกตั้งค่า `user_id` ให้เป็นค่าว่าง" ซึ่งเป็นประโยชน์ในการเก็บรักษาประวัติการสั่งซื้อที่เคยส่งไปยังที่อยู่นั้นๆ
* **Simplicity and Focus:** คุณจะเห็นว่า Model และ Controller ของ Address นั้นตรงไปตรงมาและไม่ซับซ้อน เพราะมันมีหน้าที่ที่ชัดเจนเพียงอย่างเดียวคือการจัดการข้อมูลที่อยู่ ซึ่งเป็นข้อดีของการแบ่งโค้ดออกเป็นส่วนเล็กๆ (Modularization)
* **Pattern Repetition:** การที่เราใช้รูปแบบโค้ดที่เหมือนกันในทุกๆ "แผนก" (Controller & Model) ทำให้ API ทั้งหมดของเรามีความสม่ำเสมอ (Consistent) และง่ายต่อการเรียนรู้และบำรุงรักษาในระยะยาว

---

#### **Payments**
Payment.php & PaymentController.php: แผนกการเงินและบัญชี
เดินทางมาถึงแผนกสุดท้ายแต่สำคัญไม่แพ้ใคร นั่นคือ **"แผนกการเงินและบัญชี"** ซึ่งรับผิดชอบโดยคู่หู `PaymentController` และ `Payment` Model ครับ แผนกนี้มีหน้าที่บันทึกและจัดการข้อมูล "การชำระเงิน" ทั้งหมดที่เกิดขึ้นในระบบ

### แนวคิดการทำงาน: แผนกสนับสนุนที่ผูกติดกับใบสั่งซื้อ

แผนกนี้ทำงานอย่างใกล้ชิดกับ "แผนกรับออเดอร์" และมีหลักการทำงานที่ชัดเจน:

* **`PaymentController.php` (หัวหน้าแผนกการเงิน):**
  * รับคำสั่งจาก `Router` เช่น "ลูกค้าชำระเงินสำหรับออเดอร์หมายเลข 5 แล้ว" หรือ "ขอดูสถานะการชำระเงินของออเดอร์หมายเลข 5"
  * สั่งงานลูกน้องในแผนก (`Payment` Model)
* **`src/Models/Payment.php` (พนักงานบัญชี):**
  * เป็นผู้เชี่ยวชาญ "แฟ้มบันทึกการชำระเงิน" (ตาราง `payments`)
  * **ความสามารถพิเศษ:** พนักงานคนนี้จะรู้เสมอว่าการชำระเงินแต่ละครั้ง เป็นของ "ใบสั่งซื้อ" (`Order`) ใบไหน (เชื่อมโยงกับ `orders` table ผ่าน `order_id`)

ความสัมพันธ์ของแผนกนี้เป็นแบบ **One-to-One** (ในทางปฏิบัติ) คือ "ใบสั่งซื้อ 1 ใบ (`Order`) ควรจะมีการชำระเงิน 1 ครั้ง (`Payment`)" การเชื่อมโยงนี้สำคัญมากในการตรวจสอบว่าออเดอร์ไหนจ่ายเงินแล้วหรือยังไม่ได้จ่ายครับ

### 📄 `src/Models/Payment.php` (พนักงานบัญชี - คนทำงาน)

```php
<?php
// ประกาศว่าไฟล์นี้อยู่ใน "แผนก" Models
namespace App\Models;

// นำเครื่องมือที่จำเป็นเข้ามา
use App\Core\Database; // ช่างเชื่อมต่อฐานข้อมูล
use PDO;             // เครื่องมือมาตรฐานสำหรับคุยกับฐานข้อมูล

/**
 * คลาส Payment (Model)
 * พนักงานที่รับผิดชอบตาราง 'payments'
 */
class Payment {
    // ตัวแปรสำหรับเก็บการเชื่อมต่อฐานข้อมูล
    private $db;

    public function __construct() {
        // "เบิกเครื่องมือ" คือการเชื่อมต่อฐานข้อมูลมาเตรียมไว้
        $this->db = (new Database())->getConnection();
    }

    /**
     * เมธอด getAll
     * หน้าที่: ไปเอา "แฟ้มบันทึกการชำระเงิน" มาทั้งหมด
     */
    public function getAll() {
        return $this->db->query("SELECT * FROM payments ORDER BY created_at DESC")->fetchAll(PDO::FETCH_ASSOC);
    }

    /**
     * เมธอด getById
     * หน้าที่: ไปหา "แฟ้มบันทึกการชำระเงิน" 1 ใบตามเลข ID ที่หัวหน้าสั่ง
     */
    public function getById($id) {
        $stmt = $this->db->prepare("SELECT * FROM payments WHERE id = :id");
        $stmt->execute([':id' => $id]);
        return $stmt->fetch(PDO::FETCH_ASSOC);
    }

    /**
     * เมธอด create
     * หน้าที่: สร้าง "แฟ้มบันทึกการชำระเงินใหม่" 1 ใบ
     */
    public function create($data) {
        // ตรวจสอบข้อมูลที่จำเป็น
        if (empty($data->order_id) || !isset($data->amount) || empty($data->method)) {
            return ['status' => 400, 'message' => 'Order ID, amount, and method are required.'];
        }
        $sql = "INSERT INTO payments (order_id, amount, method, status) VALUES (:order_id, :amount, :method, :status)";
        $stmt = $this->db->prepare($sql);
        $stmt->execute([
            ':order_id' => $data->order_id,
            ':amount' => $data->amount,
            ':method' => $data->method,
            ':status' => $data->status ?? 'pending' // ถ้าไม่ระบุ ให้เป็น 'pending' ไว้ก่อน
        ]);
        return ['status' => 201, 'message' => 'Payment created successfully.'];
    }

    /**
     * เมธอด update
     * หน้าที่: แก้ไขข้อมูลใน "แฟ้มบันทึกการชำระเงิน" (เช่น อัปเดตสถานะเป็น 'completed')
     */
    public function update($id, $data) {
         if (!isset($data->amount) || empty($data->method) || empty($data->status)) {
            return ['status' => 400, 'message' => 'Amount, method, and status are required for update.'];
        }
        $sql = "UPDATE payments SET amount = :amount, method = :method, status = :status WHERE id = :id";
        $stmt = $this->db->prepare($sql);
        $stmt->execute([
            ':id' => $id,
            ':amount' => $data->amount,
            ':method' => $data->method,
            ':status' => $data->status
        ]);
        return ['status' => 200, 'message' => 'Payment updated successfully.'];
    }

    /**
     * เมธอด delete
     * หน้าที่: ทำลาย "แฟ้มบันทึกการชำระเงิน" ทิ้ง
     */
    public function delete($id) {
        $stmt = $this->db->prepare("DELETE FROM payments WHERE id = :id");
        $stmt->execute([':id' => $id]);
        return $stmt->rowCount() > 0 ? ['status' => 200, 'message' => 'Payment deleted successfully.'] : ['status' => 404, 'message' => 'Payment not found.'];
    }
}
?>
```

### 📄 `src/Controllers/PaymentController.php` (หัวหน้าแผนกการเงิน)

```php
<?php
namespace App\Controllers;
use App\Models\Payment;

/**
 * คลาส PaymentController
 * หัวหน้าแผนกที่รับคำสั่งเกี่ยวกับการชำระเงิน และสั่งงาน Payment Model
 */
class PaymentController {
    // "ลูกน้อง" ประจำตัวของหัวหน้าคนนี้
    private $model;

    public function __construct() {
        // "จ้าง" พนักงาน (Payment Model) มาเตรียมไว้
        $this->model = new Payment();
    }

    /**
     * เมธอด index
     * รับคำสั่ง: "ขอรายการชำระเงินทั้งหมด"
     */
    public function index() {
        // 1. สั่งงานลูกน้อง
        $payments = $this->model->getAll();
        // 2. นำผลลัพธ์มารายงานกลับ
        echo json_encode($payments);
    }

    /**
     * เมธอด show
     * รับคำสั่ง: "ขอข้อมูลการชำระเงิน ID ที่ [id]"
     */
    public function show($params) {
        // 1. สั่งงานลูกน้อง
        $payment = $this->model->getById($params['id']);
        // 2. ตัดสินใจว่าจะรายงานกลับอย่างไร
        if ($payment) {
            echo json_encode($payment);
        } else {
            http_response_code(404); // Not Found
            echo json_encode(['message' => 'Payment not found.']);
        }
    }

    /**
     * เมธอด create
     * รับคำสั่ง: "สร้างรายการชำระเงินใหม่ตามข้อมูลนี้"
     */
    public function create() {
        // 1. รับข้อมูลคำร้องมาจากผู้ใช้
        $data = json_decode(file_get_contents("php://input"));
        // 2. ส่งข้อมูลให้ลูกน้องไปจัดการ
        $result = $this->model->create($data);
        // 3. นำผลลัพธ์มารายงานกลับ
        http_response_code($result['status']);
        echo json_encode(['message' => $result['message']]);
    }

    // (เมธอด update และ delete ก็ใช้หลักการเดียวกัน)

    public function update($params) {
        $data = json_decode(file_get_contents("php://input"));
        $result = $this->model->update($params['id'], $data);
        http_response_code($result['status']);
        echo json_encode(['message' => $result['message']]);
    }

    public function delete($params) {
        $result = $this->model->delete($params['id']);
        http_response_code($result['status']);
        echo json_encode(['message' => $result['message']]);
    }
}
?>
```
* **Critical Data Integrity:** ตาราง `payments` มีความสัมพันธ์ที่สำคัญอย่างยิ่งกับตาราง `orders` ผ่าน `order_id` ในฐานข้อมูล เราได้ตั้งกฎ `ON DELETE CASCADE` ไว้ ซึ่งหมายความว่า **"ถ้าใบสั่งซื้อถูกลบ รายการชำระเงินที่เกี่ยวข้องกับใบสั่งซื้อนั้นจะถูกลบตามไปด้วยโดยอัตโนมัติ"** ซึ่งเป็นกฎที่สมเหตุสมผล เพราะข้อมูลการชำระเงินจะไม่มีความหมายหากไม่มีใบสั่งซื้อให้อ้างอิง
* **Clear Transaction Flow:** การมี "แผนกการเงิน" แยกออกมา ทำให้กระบวนการจัดการ E-commerce ของเราครบวงจร ตั้งแต่การสมัครสมาชิก, เลือกสินค้า, สร้างออเดอร์, และสุดท้ายคือการชำระเงิน
* **Final Piece of the Puzzle:** การทำความเข้าใจแผนกนี้จะทำให้คุณเห็นภาพรวมของระบบทั้งหมด และสามารถนำหลักการไปประยุกต์ใช้สร้างระบบอื่นๆ ที่มีความซับซ้อนได้ในอนาคต

---

#### **Reviews**
Review.php & ReviewController.php: แผนกรับฟังความคิดเห็นลูกค้า
มาถึงแผนกสุดท้ายที่จะช่วยสร้างความน่าเชื่อถือและชุมชนให้กับร้านค้าของเรา นั่นคือ **"แผนกรับฟังความคิดเห็นลูกค้า"** ซึ่งรับผิดชอบโดยคู่หู `ReviewController` และ `Review` Model ครับ แผนกนี้มีหน้าที่จัดการ "รีวิว" หรือความคิดเห็นที่ลูกค้ามีต่อสินค้าแต่ละชิ้น

#### แนวคิดการทำงาน: ศูนย์รวมความคิดเห็นที่เชื่อมโยงทุกฝ่าย

แผนกนี้เป็นอีกหนึ่ง "แผนกประสานงานกลาง" ที่มีความสำคัญมาก เพราะต้องทำงานเชื่อมโยงกับแผนกอื่นถึง 2 แผนก:

* **`ReviewController.php` (หัวหน้าแผนกรับฟังความคิดเห็น):**
  * รับคำสั่งจาก `Router` เช่น "ลูกค้า ID ที่ 1 ต้องการรีวิวสินค้า 'Laptop' 5 ดาว" หรือ "ขอดูรีวิวทั้งหมดของสินค้า 'Laptop'"
  * สั่งงานลูกน้องในแผนก (`Review` Model)
* **`src/Models/Review.php` (พนักงานรวบรวมความคิดเห็น):**
  * เป็นผู้เชี่ยวชาญ "แฟ้มรีวิวสินค้า" (ตาราง `reviews`)
  * **ความสามารถพิเศษ:** พนักงานคนนี้ต้องวิ่งไปประสานงานกับ 2 แผนกพร้อมกันเสมอ:
    1. ไปที่ **"แผนกบุคคล"** (`users` table) เพื่อดูว่า "ใครเป็นคนเขียนรีวิวนี้"
    2. ไปที่ **"แผนกสต็อกสินค้า"** (`products` table) เพื่อดูว่า "รีวิวนี้เป็นของสินค้าชิ้นไหน"
  * การดึงข้อมูลชื่อผู้ใช้และชื่อสินค้ามาแสดงพร้อมกับรีวิว ทำให้ข้อมูลที่ส่งกลับไปมีความสมบูรณ์และพร้อมใช้งานทันที

การมีแผนกนี้ทำให้เราสามารถสร้างฟีเจอร์ที่สำคัญสำหรับ E-commerce ได้ เช่น การแสดงคะแนนดาวเฉลี่ยของสินค้า หรือการแสดงความคิดเห็นจากผู้ใช้งานจริง ซึ่งช่วยในการตัดสินใจซื้อของลูกค้คนอื่นๆ ครับ

### 📄 `src/Models/Review.php` (พนักงานรวบรวมความคิดเห็น - คนทำงาน)

```php
<?php
// ประกาศว่าไฟล์นี้อยู่ใน "แผนก" Models
namespace App\Models;

// นำเครื่องมือที่จำเป็นเข้ามา
use App\Core\Database; // ช่างเชื่อมต่อฐานข้อมูล
use PDO;             // เครื่องมือมาตรฐานสำหรับคุยกับฐานข้อมูล

/**
 * คลาส Review (Model)
 * พนักงานที่รับผิดชอบตาราง 'reviews' และประสานงานกับตาราง users และ products
 */
class Review {
    // ตัวแปรสำหรับเก็บการเชื่อมต่อฐานข้อมูล
    private $db;

    public function __construct() {
        // "เบิกเครื่องมือ" คือการเชื่อมต่อฐานข้อมูลมาเตรียมไว้
        $this->db = (new Database())->getConnection();
    }

    /**
     * เมธอด getAll
     * หน้าที่: ไปเอา "แฟ้มรีวิว" มาทั้งหมด พร้อมชื่อผู้เขียนและชื่อสินค้า
     */
    public function getAll() {
        // นี่คือสุดยอดการประสานงานสามฝ่าย!
        // ใช้ JOIN ถึงสองครั้งเพื่อเชื่อมตาราง reviews (r) เข้ากับ users (u) และ products (p)
        $sql = "SELECT r.*, u.username, p.name as product_name 
                FROM reviews r 
                JOIN users u ON r.user_id = u.id 
                JOIN products p ON r.product_id = p.id 
                ORDER BY r.created_at DESC";
        return $this->db->query($sql)->fetchAll(PDO::FETCH_ASSOC);
    }

    /**
     * เมธอด getById
     * หน้าที่: ไปหา "แฟ้มรีวิว" 1 ใบ พร้อมข้อมูลประกอบครบถ้วน
     */
    public function getById($id) {
        $stmt = $this->db->prepare("SELECT r.*, u.username, p.name as product_name 
                                    FROM reviews r 
                                    JOIN users u ON r.user_id = u.id 
                                    JOIN products p ON r.product_id = p.id 
                                    WHERE r.id = :id");
        $stmt->execute([':id' => $id]);
        return $stmt->fetch(PDO::FETCH_ASSOC);
    }

    /**
     * เมธอด create
     * หน้าที่: สร้าง "แฟ้มรีวิวใหม่" 1 ใบ
     */
    public function create($data) {
        // ตรวจสอบข้อมูลที่จำเป็น
        if (empty($data->product_id) || empty($data->user_id) || empty($data->rating)) {
            return ['status' => 400, 'message' => 'Product ID, User ID, and rating are required.'];
        }
        // ตรวจสอบกฎของข้อมูล (Business Logic) ว่าคะแนนต้องอยู่ระหว่าง 1-5 เท่านั้น
        if ($data->rating < 1 || $data->rating > 5) {
             return ['status' => 400, 'message' => 'Rating must be between 1 and 5.'];
        }
        $sql = "INSERT INTO reviews (product_id, user_id, rating, comment) VALUES (:product_id, :user_id, :rating, :comment)";
        $stmt = $this->db->prepare($sql);
        $stmt->execute([
            ':product_id' => $data->product_id,
            ':user_id' => $data->user_id,
            ':rating' => $data->rating,
            ':comment' => htmlspecialchars(strip_tags($data->comment ?? ''))
        ]);
        return ['status' => 201, 'message' => 'Review created successfully.'];
    }

    /**
     * เมธอด update
     * หน้าที่: แก้ไขข้อมูลใน "แฟ้มรีวิว" (ปกติลูกค้าจะแก้ได้แค่คะแนนหรือคอมเมนต์)
     */
    public function update($id, $data) {
        if (empty($data->rating)) {
            return ['status' => 400, 'message' => 'Rating is required for update.'];
        }
        if ($data->rating < 1 || $data->rating > 5) {
             return ['status' => 400, 'message' => 'Rating must be between 1 and 5.'];
        }
        $sql = "UPDATE reviews SET rating = :rating, comment = :comment WHERE id = :id";
        $stmt = $this->db->prepare($sql);
        $stmt->execute([
            ':id' => $id,
            ':rating' => $data->rating,
            ':comment' => htmlspecialchars(strip_tags($data->comment ?? ''))
        ]);
        return ['status' => 200, 'message' => 'Review updated successfully.'];
    }

    /**
     * เมธอด delete
     * หน้าที่: ทำลาย "แฟ้มรีวิว" ทิ้ง
     */
    public function delete($id) {
        $stmt = $this->db->prepare("DELETE FROM reviews WHERE id = :id");
        $stmt->execute([':id' => $id]);
        return $stmt->rowCount() > 0 ? ['status' => 200, 'message' => 'Review deleted successfully.'] : ['status' => 404, 'message' => 'Review not found.'];
    }
}
?>
```

### 📄 `src/Controllers/ReviewController.php` (หัวหน้าแผนกรับฟังความคิดเห็น)

```php
<?php
namespace App\Controllers;
use App\Models\Review;

/**
 * คลาส ReviewController
 * หัวหน้าแผนกที่รับคำสั่งเกี่ยวกับรีวิว และสั่งงาน Review Model
 */
class ReviewController {
    // "ลูกน้อง" ประจำตัวของหัวหน้าคนนี้
    private $model;

    public function __construct() {
        // "จ้าง" พนักงาน (Review Model) มาเตรียมไว้
        $this->model = new Review();
    }

    /**
     * เมธอด index
     * รับคำสั่ง: "ขอรีวิวทั้งหมด"
     */
    public function index() {
        // 1. สั่งงานลูกน้อง
        $reviews = $this->model->getAll();
        // 2. นำผลลัพธ์มารายงานกลับ
        echo json_encode($reviews);
    }

    /**
     * เมธอด show
     * รับคำสั่ง: "ขอข้อมูลรีวิว ID ที่ [id]"
     */
    public function show($params) {
        // 1. สั่งงานลูกน้อง
        $review = $this->model->getById($params['id']);
        // 2. ตัดสินใจว่าจะรายงานกลับอย่างไร
        if ($review) {
            echo json_encode($review);
        } else {
            http_response_code(404); // Not Found
            echo json_encode(['message' => 'Review not found.']);
        }
    }

    /**
     * เมธอด create
     * รับคำสั่ง: "สร้างรีวิวใหม่ตามข้อมูลนี้"
     */
    public function create() {
        // 1. รับข้อมูลคำร้องมาจากผู้ใช้
        $data = json_decode(file_get_contents("php://input"));
        // 2. ส่งข้อมูลให้ลูกน้องไปจัดการ
        $result = $this->model->create($data);
        // 3. นำผลลัพธ์มารายงานกลับ
        http_response_code($result['status']);
        echo json_encode(['message' => $result['message']]);
    }

    // (เมธอด update และ delete ก็ใช้หลักการเดียวกัน)

    public function update($params) {
        $data = json_decode(file_get_contents("php://input"));
        $result = $this->model->update($params['id'], $data);
        http_response_code($result['status']);
        echo json_encode(['message' => $result['message']]);
    }

    public function delete($params) {
        $result = $this->model->delete($params['id']);
        http_response_code($result['status']);
        echo json_encode(['message' => $result['message']]);
    }
}
?>
```
* **Connecting Multiple Tables:** `Review` Model เป็นตัวอย่างที่ยอดเยี่ยมของการใช้ `JOIN` เพื่อเชื่อมข้อมูลจาก 3 ตารางเข้าด้วยกัน ทำให้ Response ที่ได้มีความหมายและสมบูรณ์ในตัวเอง
* **Business Logic Validation:** การตรวจสอบว่า `rating` ต้องอยู่ระหว่าง 1-5 เป็นตัวอย่างของ "Business Logic" หรือกฎของธุรกิจที่ควรจะถูกตรวจสอบในระดับ Model เพราะ Model คือผู้ที่รู้ข้อมูลของตัวเองดีที่สุด
* **Completing the Ecosystem:** การมีระบบรีวิวทำให้ "บริษัท API" ของเรามีความเป็น E-commerce ที่สมบูรณ์แบบมากยิ่งขึ้น และแสดงให้เห็นว่าสถาปัตยกรรม MVC ที่เราสร้างขึ้นมานั้นยืดหยุ่นพอที่จะรองรับ "แผนก" ใหม่ๆ ที่เพิ่มเข้ามาได้เสมอ

---

### 5.5 ไฟล์กำหนดเส้นทาง (Routes) 
แผนที่และสารบัญของ API
เราจะมาเจาะลึกไฟล์ที่เป็นเหมือน "แผนที่" หรือ "สารบัญ" ของ "บริษัท API" ของเราทั้งหมด นั่นคือ `src/routes.php` ครับ

ไฟล์นี้ไม่มีโค้ดที่ซับซ้อน แต่มันคือศูนย์กลางที่เชื่อมโยงทุกอย่างเข้าด้วยกัน ลองจินตนาการว่าไฟล์นี้คือ **"สมุดคู่มือของโอเปอเรเตอร์"** ที่ `Router` (ผู้จัดการ) ของเราเปิดดูอยู่ตลอดเวลา ในสมุดเล่มนี้จะบอกไว้หมดเลยว่า:

* **"ถ้ามีคนติดต่อมาที่เบอร์ต่อ (URL) นี้..."** (เช่น `/api/products/1`)

* **"...ด้วยวิธีการ (Method) แบบนี้..."** (เช่น `GET` คือการขอดูข้อมูล)

* **"...จะต้องโอนสายไปให้ 'หัวหน้าแผนก' (Controller) คนไหน..."** (เช่น `ProductController`)

* **"...ที่โต๊ะทำงาน (Method) เบอร์อะไร..."** (เช่น `show()`)

* **"...และก่อนจะโอนสาย ต้องให้แขกผ่าน 'ด่านตรวจ' (Middleware) ด่านไหนบ้าง"** (เช่น `RateLimiter` และ `AuthMiddleware`)

ดังนั้น ไฟล์นี้จึงเป็นที่ที่เรากำหนด "กฎ" ทั้งหมดของ API ของเราว่าใครสามารถทำอะไร ที่ไหน ได้บ้าง การมีแผนที่รวมศูนย์แบบนี้ทำให้เราเห็นภาพรวมของ API ทั้งหมดได้ในที่เดียว และง่ายต่อการเพิ่ม, ลบ, หรือแก้ไขเส้นทางในอนาคต

### 📄 `src/routes.php`

ไฟล์นี้จะเชื่อมโยงทุก Endpoint เข้ากับ Controller ที่ถูกต้อง

```php
<?php
/**
 * API Routes Configuration (แผนที่และสารบัญของ API)
 *
 * ไฟล์นี้ทำหน้าที่ "วาดแผนที่" โดยบอก Router ว่า URL ไหน จะต้องไปเรียกใช้ Controller ตัวไหน
 * และต้องผ่านด่านตรวจ (Middleware) อะไรบ้าง
 *
 * ตัวแปร $router ถูกสร้างมาจากไฟล์ public/index.php แล้ว
 */


// =======================================================================
// กลุ่มที่ 1: PUBLIC ROUTES (เส้นทางสาธารณะ)
// ใครๆ ก็สามารถเข้ามาใช้บริการได้ แต่ยังต้องผ่านด่านตรวจ "ควบคุมจำนวนคน" (RateLimiter)
// =======================================================================

// --- แผนกยืนยันตัวตน ---
// ถ้ามีคนส่งคำขอแบบ POST มาที่ /api/auth/login
// ให้โอนสายไปที่ AuthController ที่โต๊ะทำงานชื่อ login()
// โดยต้องผ่านด่าน RateLimiter ก่อน
$router->post('/auth/login', 'AuthController@login', ['App\Middlewares\RateLimiter']);

// --- แผนกบุคคล (สำหรับการสมัคร) ---
// อนุญาตให้คนทั่วไปสร้าง user ใหม่ (สมัครสมาชิก) ได้
$router->post('/users', 'UserController@create', ['App\Middlewares\RateLimiter']);
$router->post('/users/check-existence', 'UserController@checkExistence', ['App\Middlewares\RateLimiter']);

// --- แผนกดูข้อมูลสินค้า (สาธารณะ) ---
$router->get('/categories', 'CategoryController@index', ['App\Middlewares\RateLimiter']);
// {id} คือ "ตัวแปร" ใน URL Router จะดึงค่าตรงนี้ (เช่น เลข 1, 2, 3) ไปใช้งานต่อได้
$router->get('/categories/{id}', 'CategoryController@show', ['App\Middlewares\RateLimiter']);
$router->get('/products', 'ProductController@index', ['App\Middlewares\RateLimiter']);
$router->get('/products/{id}', 'ProductController@show', ['App\Middlewares\RateLimiter']);
$router->get('/reviews', 'ReviewController@index', ['App\Middlewares\RateLimiter']);
$router->get('/reviews/{id}', 'ReviewController@show', ['App\Middlewares\RateLimiter']);


// =======================================================================
// กลุ่มที่ 2: AUTHENTICATED ROUTES (เส้นทางสำหรับผู้ที่ล็อกอินแล้ว)
// ต้องมี "บัตรพนักงาน" (Token) ที่ถูกต้อง ถึงจะเข้ามาได้ (ผ่านด่าน isAuthenticated)
// =======================================================================

$router->post('/orders', 'OrderController@create', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isAuthenticated']);
$router->post('/order-items', 'OrderItemController@create', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isAuthenticated']);
$router->post('/addresses', 'AddressController@create', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isAuthenticated']);
$router->post('/payments', 'PaymentController@create', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isAuthenticated']);
$router->post('/reviews', 'ReviewController@create', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isAuthenticated']);


// =======================================================================
// กลุ่มที่ 3: PERMISSION-BASED ROUTES (เส้นทางที่ต้องการสิทธิ์เฉพาะ)
// นอกจากจะต้องมีบัตรแล้ว "ตำแหน่ง" ในบัตรก็ต้องสูงพอด้วย
// =======================================================================

// --- Users ---
// การดู user ทั้งหมด ต้องเป็น Employee ขึ้นไป
$router->get('/users', 'UserController@index', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isEmployee']);
// การดู user คนอื่น ต้องเป็นเจ้าของ หรือ Manager ขึ้นไป (isSelfOrManager)
$router->get('/users/{id}', 'UserController@show', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isSelfOrManager']);
$router->put('/users/{id}', 'UserController@update', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isSelfOrManager']);
$router->post('/users/{id}/change-password', 'UserController@changePassword', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isSelfOrManager']);
// การลบ user ต้องเป็น Admin เท่านั้น
$router->delete('/users/{id}', 'UserController@delete', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isAdmin']);

// --- Categories & Products (จัดการโดย Manager ขึ้นไป) ---
$router->post('/categories', 'CategoryController@create', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isManager']);
$router->put('/categories/{id}', 'CategoryController@update', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isManager']);
$router->delete('/categories/{id}', 'CategoryController@delete', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isManager']);
$router->post('/products', 'ProductController@create', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isManager']);
$router->put('/products/{id}', 'ProductController@update', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isManager']);
$router->delete('/products/{id}', 'ProductController@delete', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isManager']);

// --- Orders ---
$router->get('/orders', 'OrderController@index', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isEmployee']);
// การดู Order ต้องเป็นเจ้าของ หรือ Manager ขึ้นไป (isOrderOwnerOrManager)
$router->get('/orders/{id}', 'OrderController@show', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isOrderOwnerOrManager']);
$router->put('/orders/{id}', 'OrderController@update', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isOrderOwnerOrManager']);
$router->delete('/orders/{id}', 'OrderController@delete', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isAdmin']);

// (เส้นทางที่เหลือทั้งหมดจะใช้หลักการเดียวกัน)
$router->get('/order-items', 'OrderItemController@index', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isEmployee']);
$router->get('/order-items/{id}', 'OrderItemController@show', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isOrderOwnerOrManager']);
$router->put('/order-items/{id}', 'OrderItemController@update', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isManager']);
$router->delete('/order-items/{id}', 'OrderItemController@delete', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isManager']);
$router->get('/addresses', 'AddressController@index', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isEmployee']);
$router->get('/addresses/{id}', 'AddressController@show', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isSelfOrManager']);
$router->put('/addresses/{id}', 'AddressController@update', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isSelfOrManager']);
$router->delete('/addresses/{id}', 'AddressController@delete', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isSelfOrManager']);
$router->get('/payments', 'PaymentController@index', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isEmployee']);
$router->get('/payments/{id}', 'PaymentController@show', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isOrderOwnerOrManager']);
$router->put('/payments/{id}', 'PaymentController@update', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isManager']);
$router->delete('/payments/{id}', 'PaymentController@delete', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isAdmin']);
$router->put('/reviews/{id}', 'ReviewController@update', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isSelfOrManager']);
$router->delete('/reviews/{id}', 'ReviewController@delete', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isSelfOrManager']);

?>

```
#### สรุปหัวใจสำคัญของ `routes.php`

* **API's Public Face:** ไฟล์นี้คือตัวกำหนด "หน้าตา" ทั้งหมดของ API ของเรา มันบอกให้โลกรู้ว่า API ของเรามีบริการอะไรบ้าง และจะเรียกใช้บริการเหล่านั้นได้อย่างไร
* **Centralized Security:** เป็นจุดที่เราใช้บังคับกฎความปลอดภัยทั้งหมด เราสามารถเห็นภาพรวมได้ทันทีว่าเส้นทางไหนเปิดสาธารณะ เส้นทางไหนต้องล็อกอิน และเส้นทางไหนต้องการสิทธิ์ระดับสูง
* **Readability (ความอ่านง่าย):** การจัดกลุ่มเส้นทางตามทรัพยากร (Users, Products, Orders) และตามระดับการเข้าถึง (Public, Authenticated) ทำให้ไฟล์นี้อ่านและทำความเข้าใจได้ง่ายมาก
ไฟล์ `routes.php` คือบทสรุปสุดท้ายที่นำทุกส่วนที่เราสร้างมา (Controllers, Middlewares) มาประกอบร่างเข้าด้วยกัน และทำให้ "บริษัท API" ของเราพร้อมเปิดให้บริการอย่างเป็นทางการครับ!

## ขั้นตอนที่ 6: ทดสอบด้วย Postman (The Final Check)

ตอนนี้ API ของคุณพร้อมเต็ม 100% แล้วครับ คุณสามารถใช้ **Postman Collection** ที่ผมแก้ไขให้ก่อนหน้านี้เพื่อทดสอบทุก Endpoint ได้เลย

* **อย่าลืม**: เปลี่ยนรหัสผ่านใน Request Body ของ `Login` ให้ตรงกับที่คุณตั้งไว้ในฐานข้อมูล

* **การทดสอบ**: เริ่มจากการ Login เพื่อรับ Token จากนั้นนำ Token ไปใส่ใน Header ของ Request อื่นๆ ที่ต้องการการยืนยันตัวตน

## ขั้นตอนที่ 7: การแก้ไขปัญหาและก้าวต่อไป (Troubleshooting & Next Steps)

* **ปัญหาที่พบบ่อย**: `404 Not Found` (เช็ค URL), `500 Internal Server Error` (เช็ค log), `401/403 Unauthorized/Forbidden` (เช็ค Token และ Role)

* **ก้าวต่อไป**: ลองศึกษาเรื่อง **Validation** (การตรวจสอบข้อมูล), **Pagination** (การแบ่งหน้า), การทำ **API Documentation** ด้วย Swagger, และการเขียน **Unit Test** เพื่อให้ API ของคุณสมบูรณ์แบบยิ่งขึ้น

ผมหวังว่าคู่มือฉบับสมบูรณ์นี้จะช่วยให้คุณสร้างและทำความเข้าใจโปรเจกต์ได้ทั้งหมดนะครับ ขอให้สนุกกับการเขียนโค้ดครับ!
