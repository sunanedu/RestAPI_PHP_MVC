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
### สรุปหัวใจสำคัญของการออกแบบฐานข้อมูล

* **Normalization:** เราพยายามแยกข้อมูลแต่ละเรื่องออกจากกัน (เช่น User, Product) เพื่อลดความซ้ำซ้อนและทำให้จัดการง่าย
* **Primary Key:** ทุกตารางต้องมี `id` ที่เป็น Primary Key เพื่อให้ข้อมูลแต่ละแถวมีเอกลักษณ์ ไม่ซ้ำกัน
* **Foreign Key:** เราใช้ `user_id`, `category_id`, `order_id` เป็น Foreign Key เพื่อสร้าง "ความสัมพันธ์" ระหว่างตาราง ทำให้ข้อมูลเชื่อมโยงกันอย่างมีความหมาย
* **Data Integrity (ความถูกต้องของข้อมูล):** การใช้ `CONSTRAINT` และกฎ `ON DELETE` ช่วยให้มั่นใจว่าฐานข้อมูลของเราจะไม่มี "ข้อมูลกำพร้า" (เช่น ออเดอร์ที่มี `user_id` ที่ไม่มีอยู่จริง) และรักษาความถูกต้องของข้อมูลไว้เสมอ
**เกร็ดความรู้**: สังเกตว่าเราใช้ `ON DELETE CASCADE` และ `ON DELETE SET NULL` เพื่อจัดการความสัมพันธ์ของข้อมูล

* `CASCADE`: ถ้าลบข้อมูลหลัก (เช่น ลบ Order) ข้อมูลย่อยที่ผูกกันอยู่ (เช่น Order Items) จะถูกลบตามไปด้วย
* `SET NULL`: ถ้าลบข้อมูลหลัก (เช่น ลบ User) ข้อมูลที่เคยผูกอยู่ (เช่น Review) จะถูกเปลี่ยนค่า Foreign Key ให้เป็น `NULL` เพื่อรักษาข้อมูลรีวิวไว้

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

### 📄`src/Middlewares/AuthMiddleware.php`

```
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

### 📄`src/Middlewares/RateLimiter.php`

```
<?php
namespace App\Middlewares;
use App\Core\Database;
use Firebase\JWT\JWT;
use Firebase\JWT\Key;
use DateTime;
use PDO;

class RateLimiter {
    private $db;
    private $limit = 100;
    private $window = 3600;

    public function __construct() { $this->db = (new Database())->getConnection(); }

    public function handle() {
        $ip = $_SERVER['REMOTE_ADDR'];
        $endpoint = parse_url($_SERVER['REQUEST_URI'], PHP_URL_PATH);
        $userId = $this->getUserIdFromToken();
        $query = "SELECT id, request_count, window_start FROM rate_limits WHERE ip_address = :ip AND endpoint = :endpoint AND (user_id = :user_id OR (:user_id IS NULL AND user_id IS NULL))";
        $stmt = $this->db->prepare($query);
        $stmt->bindValue(':ip', $ip);
        $stmt->bindValue(':endpoint', $endpoint);
        $stmt->bindValue(':user_id', $userId, $userId ? PDO::PARAM_INT : PDO::PARAM_NULL);
        $stmt->execute();
        $rateLimit = $stmt->fetch(PDO::FETCH_ASSOC);
        $currentTime = new DateTime();

        if ($rateLimit) {
            $windowStart = new DateTime($rateLimit['window_start']);
            if ($currentTime->getTimestamp() - $windowStart->getTimestamp() > $this->window) {
                $stmt = $this->db->prepare("UPDATE rate_limits SET request_count = 1, window_start = :now WHERE id = :id");
                $stmt->execute([':now' => $currentTime->format('Y-m-d H:i:s'), ':id' => $rateLimit['id']]);
            } else {
                if ($rateLimit['request_count'] >= $this->limit) {
                    return ['status' => 429, 'message' => 'Too many requests. Please try again later.'];
                }
                $stmt = $this->db->prepare("UPDATE rate_limits SET request_count = request_count + 1 WHERE id = :id");
                $stmt->execute([':id' => $rateLimit['id']]);
            }
        } else {
            $stmt = $this->db->prepare("INSERT INTO rate_limits (user_id, ip_address, endpoint, request_count, window_start) VALUES (:user_id, :ip, :endpoint, 1, :now)");
            $stmt->bindValue(':user_id', $userId, $userId ? PDO::PARAM_INT : PDO::PARAM_NULL);
            $stmt->bindValue(':ip', $ip);
            $stmt->bindValue(':endpoint', $endpoint);
            $stmt->bindValue(':now', $currentTime->format('Y-m-d H:i:s'));
            $stmt->execute();
        }
        return true;
    }

    private function getUserIdFromToken() {
        $authHeader = getallheaders()['Authorization'] ?? '';
        if (preg_match('/Bearer\s(\S+)/', $authHeader, $matches)) {
            try { return JWT::decode($matches[1], new Key(JWT_SECRET, 'HS256'))->sub; } catch (\Exception $e) { return null; }
        }
        return null;
    }
}
?>
```

### 5.4 โค้ดทั้งหมดสำหรับ Controllers และ Models (ฉบับอธิบาย)

นี่คือโค้ดทั้งหมดสำหรับส่วนควบคุมและจัดการข้อมูล พร้อมคำอธิบายการทำงานในแต่ละส่วนครับ

#### **Authentication**

##### `src/Controllers/AuthController.php`

```
<?php
namespace App\Controllers;

use Firebase\JWT\JWT;
use App\Core\Database;
use PDO;

/**
 * AuthController
 * จัดการการยืนยันตัวตน เช่น การล็อกอิน
 */
class AuthController {
    private $db;

    public function __construct() {
        // สร้างการเชื่อมต่อฐานข้อมูลเมื่อคลาสนี้ถูกสร้างขึ้น
        $this->db = (new Database())->getConnection();
    }

    /**
     * เมธอด login
     * รับ email และ password, ตรวจสอบกับฐานข้อมูล, และสร้าง JWT token ถ้าถูกต้อง
     */
    public function login() {
        // อ่านข้อมูล JSON ที่ส่งมาจากผู้ใช้
        $data = json_decode(file_get_contents("php://input"));

        // ตรวจสอบว่ามีข้อมูลที่จำเป็นครบหรือไม่
        if (empty($data->email) || empty($data->password)) {
            http_response_code(400); // Bad Request
            echo json_encode(['message' => 'Email and password are required.']);
            return;
        }

        // ค้นหาผู้ใช้ด้วย email
        $stmt = $this->db->prepare("SELECT id, password, role FROM users WHERE email = :email");
        $stmt->execute([':email' => $data->email]);
        $user = $stmt->fetch(PDO::FETCH_ASSOC);

        // ตรวจสอบว่าเจอผู้ใช้ และรหัสผ่านตรงกันหรือไม่
        if ($user && password_verify($data->password, $user['password'])) {
            // สร้างข้อมูลสำหรับใส่ใน Token (Payload)
            $payload = [
                'iss' => JWT_ISSUER, // ผู้ออก token
                'aud' => JWT_AUDIENCE, // ผู้รับ token
                'iat' => time(), // เวลาที่สร้าง token
                'exp' => time() + (60*60*24), // Token หมดอายุใน 24 ชั่วโมง
                'sub' => $user['id'], // Subject (ID ของผู้ใช้)
                'role' => $user['role'] // Role ของผู้ใช้
            ];

            // เข้ารหัส payload ให้เป็น JWT token
            $jwt = JWT::encode($payload, JWT_SECRET, 'HS256');
            http_response_code(200); // OK
            echo json_encode(['token' => $jwt, 'message' => 'Login successful.']);
        } else {
            // ถ้าข้อมูลไม่ถูกต้อง
            http_response_code(401); // Unauthorized
            echo json_encode(['message' => 'Invalid credentials.']);
        }
    }
}
?>
```

#### **Users**

##### `src/Models/User.php`

```
<?php
namespace App\Models;
use App\Core\Database;
use PDO;

/**
 * User Model
 * ทำหน้าที่จัดการข้อมูลในตาราง 'users' โดยตรง
 */
class User {
    private $db;
    public function __construct() { $this->db = (new Database())->getConnection(); }

    // ดึงข้อมูลผู้ใช้ทั้งหมด
    public function getAll() {
        return $this->db->query("SELECT id, username, email, first_name, last_name, role, created_at FROM users")->fetchAll(PDO::FETCH_ASSOC);
    }
    // ดึงข้อมูลผู้ใช้คนเดียวด้วย ID
    public function getById($id) {
        $stmt = $this->db->prepare("SELECT id, username, email, first_name, last_name, role, created_at FROM users WHERE id = :id");
        $stmt->execute([':id' => $id]);
        return $stmt->fetch(PDO::FETCH_ASSOC);
    }
    // สร้างผู้ใช้ใหม่
    public function create($data) {
        if (empty($data->username) || empty($data->email) || empty($data->password)) {
            return ['status' => 400, 'message' => 'Username, email, and password are required.'];
        }
        $sql = "INSERT INTO users (username, email, password, first_name, last_name, role) VALUES (:username, :email, :password, :first_name, :last_name, :role)";
        $stmt = $this->db->prepare($sql);
        try {
            $stmt->execute([
                ':username' => htmlspecialchars(strip_tags($data->username)),
                ':email' => htmlspecialchars(strip_tags($data->email)),
                ':password' => password_hash($data->password, PASSWORD_BCRYPT), // เข้ารหัสผ่านก่อนเก็บ
                ':first_name' => $data->first_name ?? null,
                ':last_name' => $data->last_name ?? null,
                ':role' => $data->role ?? 'customer'
            ]);
            return ['status' => 201, 'message' => 'User created successfully.'];
        } catch (\PDOException $e) {
            // ดักจับ error กรณี username หรือ email ซ้ำ
            return ['status' => 409, 'message' => 'Username or email already exists.'];
        }
    }
    // อัปเดตข้อมูลผู้ใช้
    public function update($id, $data) {
        $sql = "UPDATE users SET first_name = :first_name, last_name = :last_name, role = :role WHERE id = :id";
        $stmt = $this->db->prepare($sql);
        $stmt->execute([':id' => $id, ':first_name' => $data->first_name, ':last_name' => $data->last_name, ':role' => $data->role]);
        return ['status' => 200, 'message' => 'User updated successfully.'];
    }
    // ลบผู้ใช้
    public function delete($id) {
        $stmt = $this->db->prepare("DELETE FROM users WHERE id = :id");
        $stmt->execute([':id' => $id]);
        return $stmt->rowCount() > 0 ? ['status' => 200, 'message' => 'User deleted successfully.'] : ['status' => 404, 'message' => 'User not found.'];
    }
    // ตรวจสอบว่ามี username/email นี้ในระบบหรือยัง
    public function checkExistence($data) {
        $field = isset($data->username) ? 'username' : 'email';
        $value = $data->username ?? $data->email;
        if (empty($value)) return ['status' => 400, 'message' => 'Username or email is required.'];

        $stmt = $this->db->prepare("SELECT id FROM users WHERE $field = :value");
        $stmt->execute([':value' => $value]);
        return $stmt->fetch() ? ['status' => 409, 'message' => ucfirst($field) . ' already exists.'] : ['status' => 200, 'message' => ucfirst($field) . ' is available.'];
    }
    // เปลี่ยนรหัสผ่าน
    public function changePassword($id, $data) {
        if (empty($data->old_password) || empty($data->new_password)) {
            return ['status' => 400, 'message' => 'Old and new passwords are required.'];
        }
        $stmt = $this->db->prepare("SELECT password FROM users WHERE id = :id");
        $stmt->execute([':id' => $id]);
        $user = $stmt->fetch(PDO::FETCH_ASSOC);
        if (!$user || !password_verify($data->old_password, $user['password'])) {
            return ['status' => 401, 'message' => 'Invalid old password.'];
        }
        $stmt = $this->db->prepare("UPDATE users SET password = :password WHERE id = :id");
        $stmt->execute([':password' => password_hash($data->new_password, PASSWORD_BCRYPT), ':id' => $id]);
        return ['status' => 200, 'message' => 'Password changed successfully.'];
    }
}
?>
```

##### `src/Controllers/UserController.php`

```
<?php
namespace App\Controllers;
use App\Models\User;

/**
 * UserController
 * เป็นตัวกลางรับ Request เกี่ยวกับ User แล้วส่งต่อไปให้ Model
 */
class UserController {
    private $model;
    public function __construct() { $this->model = new User(); }

    // เรียก Model เพื่อดึงผู้ใช้ทั้งหมด
    public function index() { echo json_encode($this->model->getAll()); }

    // เรียก Model เพื่อดึงผู้ใช้คนเดียว
    public function show($params) {
        $item = $this->model->getById($params['id']);
        if ($item) echo json_encode($item); else { http_response_code(404); echo json_encode(['message' => 'User not found.']); }
    }

    // เรียก Model เพื่อสร้างผู้ใช้
    public function create() {
        $data = json_decode(file_get_contents("php://input"));
        $result = $this->model->create($data);
        http_response_code($result['status']); echo json_encode(['message' => $result['message']]);
    }
    // เรียก Model เพื่ออัปเดตผู้ใช้
    public function update($params) {
        $data = json_decode(file_get_contents("php://input"));
        $result = $this->model->update($params['id'], $data);
        http_response_code($result['status']); echo json_encode(['message' => $result['message']]);
    }
    // เรียก Model เพื่อลบผู้ใช้
    public function delete($params) {
        $result = $this->model->delete($params['id']);
        http_response_code($result['status']); echo json_encode(['message' => $result['message']]);
    }
    // เรียก Model เพื่อเช็คข้อมูลซ้ำ
    public function checkExistence() {
        $data = json_decode(file_get_contents("php://input"));
        $result = $this->model->checkExistence($data);
        http_response_code($result['status']); echo json_encode(['message' => $result['message']]);
    }
    // เรียก Model เพื่อเปลี่ยนรหัสผ่าน
    public function changePassword($params) {
        $data = json_decode(file_get_contents("php://input"));
        $result = $this->model->changePassword($params['id'], $data);
        http_response_code($result['status']); echo json_encode(['message' => $result['message']]);
    }
}
?>
```

#### **Categories**

##### `src/Models/Category.php`

```
<?php
namespace App\Models;
use App\Core\Database;
use PDO;

class Category {
    private $db;
    public function __construct() { $this->db = (new Database())->getConnection(); }
    public function getAll() { return $this->db->query("SELECT * FROM categories ORDER BY name ASC")->fetchAll(PDO::FETCH_ASSOC); }
    public function getById($id) {
        $stmt = $this->db->prepare("SELECT * FROM categories WHERE id = :id");
        $stmt->execute([':id' => $id]);
        return $stmt->fetch(PDO::FETCH_ASSOC);
    }
    public function create($data) {
        $sql = "INSERT INTO categories (name, description) VALUES (:name, :description)";
        $stmt = $this->db->prepare($sql);
        $stmt->execute([':name' => $data->name, ':description' => $data->description ?? null]);
        return ['status' => 201, 'message' => 'Category created.'];
    }
    public function update($id, $data) {
        $sql = "UPDATE categories SET name = :name, description = :description WHERE id = :id";
        $stmt = $this->db->prepare($sql);
        $stmt->execute([':id' => $id, ':name' => $data->name, ':description' => $data->description ?? null]);
        return ['status' => 200, 'message' => 'Category updated.'];
    }
    public function delete($id) {
        $stmt = $this->db->prepare("DELETE FROM categories WHERE id = :id");
        $stmt->execute([':id' => $id]);
        return $stmt->rowCount() > 0 ? ['status' => 200, 'message' => 'Category deleted.'] : ['status' => 404, 'message' => 'Category not found.'];
    }
}
?>
```

##### `src/Controllers/CategoryController.php`

```
<?php
namespace App\Controllers;
use App\Models\Category;

class CategoryController {
    private $model;
    public function __construct() { $this->model = new Category(); }
    public function index() { echo json_encode($this->model->getAll()); }
    public function show($params) {
        $item = $this->model->getById($params['id']);
        if ($item) echo json_encode($item);
        else { http_response_code(404); echo json_encode(['message' => 'Category not found.']); }
    }
    public function create() {
        $data = json_decode(file_get_contents("php://input"));
        $result = $this->model->create($data);
        http_response_code($result['status']);
        echo json_encode(['message' => $result['message']]);
    }
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

#### **Products**

##### `src/Models/Product.php`

```
<?php
namespace App\Models;
use App\Core\Database;
use PDO;

class Product {
    private $db;
    public function __construct() { $this->db = (new Database())->getConnection(); }
    public function getAll() { return $this->db->query("SELECT p.*, c.name as category_name FROM products p LEFT JOIN categories c ON p.category_id = c.id ORDER BY p.name ASC")->fetchAll(PDO::FETCH_ASSOC); }
    public function getById($id) {
        $stmt = $this->db->prepare("SELECT p.*, c.name as category_name FROM products p LEFT JOIN categories c ON p.category_id = c.id WHERE p.id = :id");
        $stmt->execute([':id' => $id]);
        return $stmt->fetch(PDO::FETCH_ASSOC);
    }
    public function create($data) {
        $sql = "INSERT INTO products (name, price, description, category_id, stock) VALUES (:name, :price, :description, :category_id, :stock)";
        $stmt = $this->db->prepare($sql);
        $stmt->execute([':name' => $data->name, ':price' => $data->price, ':description' => $data->description ?? null, ':category_id' => $data->category_id, ':stock' => $data->stock ?? 0]);
        return ['status' => 201, 'message' => 'Product created.'];
    }
    public function update($id, $data) {
        $sql = "UPDATE products SET name = :name, price = :price, description = :description, category_id = :category_id, stock = :stock WHERE id = :id";
        $stmt = $this->db->prepare($sql);
        $stmt->execute([':id' => $id, ':name' => $data->name, ':price' => $data->price, ':description' => $data->description ?? null, ':category_id' => $data->category_id, ':stock' => $data->stock ?? 0]);
        return ['status' => 200, 'message' => 'Product updated.'];
    }
    public function delete($id) {
        $stmt = $this->db->prepare("DELETE FROM products WHERE id = :id");
        $stmt->execute([':id' => $id]);
        return $stmt->rowCount() > 0 ? ['status' => 200, 'message' => 'Product deleted.'] : ['status' => 404, 'message' => 'Product not found.'];
    }
}
?>
```

##### `src/Controllers/ProductController.php`

```
<?php
namespace App\Controllers;
use App\Models\Product;

class ProductController {
    private $model;
    public function __construct() { $this->model = new Product(); }
    public function index() { echo json_encode($this->model->getAll()); }
    public function show($params) {
        $item = $this->model->getById($params['id']);
        if ($item) echo json_encode($item);
        else { http_response_code(404); echo json_encode(['message' => 'Product not found.']); }
    }
    public function create() {
        $data = json_decode(file_get_contents("php://input"));
        $result = $this->model->create($data);
        http_response_code($result['status']);
        echo json_encode(['message' => $result['message']]);
    }
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

#### **Orders**

##### `src/Models/Order.php`

```
<?php
namespace App\Models;
use App\Core\Database;
use PDO;

class Order {
    private $db;
    public function __construct() { $this->db = (new Database())->getConnection(); }
    public function getAll() {
        $sql = "SELECT o.*, u.username FROM orders o JOIN users u ON o.user_id = u.id ORDER BY o.created_at DESC";
        return $this->db->query($sql)->fetchAll(PDO::FETCH_ASSOC);
    }
    public function getById($id) {
        $stmt = $this->db->prepare("SELECT o.*, u.username FROM orders o JOIN users u ON o.user_id = u.id WHERE o.id = :id");
        $stmt->execute([':id' => $id]);
        return $stmt->fetch(PDO::FETCH_ASSOC);
    }
    public function create($data) {
        if (empty($data->user_id) || !isset($data->total_amount)) {
            return ['status' => 400, 'message' => 'User ID and total amount are required.'];
        }
        $sql = "INSERT INTO orders (user_id, total_amount, status) VALUES (:user_id, :total_amount, :status)";
        $stmt = $this->db->prepare($sql);
        $stmt->execute([':user_id' => $data->user_id, ':total_amount' => $data->total_amount, ':status' => $data->status ?? 'pending']);
        return ['status' => 201, 'message' => 'Order created successfully.'];
    }
    public function update($id, $data) {
        if (!isset($data->total_amount) || empty($data->status)) {
            return ['status' => 400, 'message' => 'Total amount and status are required for update.'];
        }
        $sql = "UPDATE orders SET total_amount = :total_amount, status = :status WHERE id = :id";
        $stmt = $this->db->prepare($sql);
        $stmt->execute([':id' => $id, ':total_amount' => $data->total_amount, ':status' => $data->status]);
        return ['status' => 200, 'message' => 'Order updated successfully.'];
    }
    public function delete($id) {
        $stmt = $this->db->prepare("DELETE FROM orders WHERE id = :id");
        $stmt->execute([':id' => $id]);
        return $stmt->rowCount() > 0 ? ['status' => 200, 'message' => 'Order deleted successfully.'] : ['status' => 404, 'message' => 'Order not found.'];
    }
}
?>
```

##### `src/Controllers/OrderController.php`

```
<?php
namespace App\Controllers;
use App\Models\Order;

class OrderController {
    private $model;
    public function __construct() { $this->model = new Order(); }
    public function index() { echo json_encode($this->model->getAll()); }
    public function show($params) {
        $item = $this->model->getById($params['id']);
        if ($item) echo json_encode($item);
        else { http_response_code(404); echo json_encode(['message' => 'Order not found.']); }
    }
    public function create() {
        $data = json_decode(file_get_contents("php://input"));
        $result = $this->model->create($data);
        http_response_code($result['status']);
        echo json_encode(['message' => $result['message']]);
    }
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

#### **Order Items**

##### `src/Models/OrderItem.php`

```
<?php
namespace App\Models;
use App\Core\Database;
use PDO;

class OrderItem {
    private $db;
    public function __construct() { $this->db = (new Database())->getConnection(); }
    public function getAll() {
        $sql = "SELECT oi.*, p.name as product_name FROM order_items oi JOIN products p ON oi.product_id = p.id ORDER BY oi.id ASC";
        return $this->db->query($sql)->fetchAll(PDO::FETCH_ASSOC);
    }
    public function getById($id) {
        $stmt = $this->db->prepare("SELECT oi.*, p.name as product_name FROM order_items oi JOIN products p ON oi.product_id = p.id WHERE oi.id = :id");
        $stmt->execute([':id' => $id]);
        return $stmt->fetch(PDO::FETCH_ASSOC);
    }
    public function create($data) {
        if (empty($data->order_id) || empty($data->product_id) || empty($data->quantity) || !isset($data->unit_price)) {
            return ['status' => 400, 'message' => 'Order ID, Product ID, quantity, and unit price are required.'];
        }
        $sql = "INSERT INTO order_items (order_id, product_id, quantity, unit_price) VALUES (:order_id, :product_id, :quantity, :unit_price)";
        $stmt = $this->db->prepare($sql);
        $stmt->execute([':order_id' => $data->order_id, ':product_id' => $data->product_id, ':quantity' => $data->quantity, ':unit_price' => $data->unit_price]);
        return ['status' => 201, 'message' => 'Order item created successfully.'];
    }
    public function update($id, $data) {
        if (empty($data->quantity) || !isset($data->unit_price)) {
            return ['status' => 400, 'message' => 'Quantity and unit price are required.'];
        }
        $sql = "UPDATE order_items SET quantity = :quantity, unit_price = :unit_price WHERE id = :id";
        $stmt = $this->db->prepare($sql);
        $stmt->execute([':id' => $id, ':quantity' => $data->quantity, ':unit_price' => $data->unit_price]);
        return ['status' => 200, 'message' => 'Order item updated successfully.'];
    }
    public function delete($id) {
        $stmt = $this->db->prepare("DELETE FROM order_items WHERE id = :id");
        $stmt->execute([':id' => $id]);
        return $stmt->rowCount() > 0 ? ['status' => 200, 'message' => 'Order item deleted successfully.'] : ['status' => 404, 'message' => 'Order item not found.'];
    }
}
?>
```

##### `src/Controllers/OrderItemController.php`

```
<?php
namespace App\Controllers;
use App\Models\OrderItem;

class OrderItemController {
    private $model;
    public function __construct() { $this->model = new OrderItem(); }
    public function index() { echo json_encode($this->model->getAll()); }
    public function show($params) {
        $item = $this->model->getById($params['id']);
        if ($item) echo json_encode($item);
        else { http_response_code(404); echo json_encode(['message' => 'Order Item not found.']); }
    }
    public function create() {
        $data = json_decode(file_get_contents("php://input"));
        $result = $this->model->create($data);
        http_response_code($result['status']);
        echo json_encode(['message' => $result['message']]);
    }
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

#### **Addresses**

##### `src/Models/Address.php`

```
<?php
namespace App\Models;
use App\Core\Database;
use PDO;

class Address {
    private $db;
    public function __construct() { $this->db = (new Database())->getConnection(); }
    public function getAll() { return $this->db->query("SELECT * FROM addresses")->fetchAll(PDO::FETCH_ASSOC); }
    public function getById($id) {
        $stmt = $this->db->prepare("SELECT * FROM addresses WHERE id = :id");
        $stmt->execute([':id' => $id]);
        return $stmt->fetch(PDO::FETCH_ASSOC);
    }
    public function create($data) {
        if (empty($data->user_id) || empty($data->address_line) || empty($data->city) || empty($data->postal_code) || empty($data->country)) {
            return ['status' => 400, 'message' => 'All address fields are required.'];
        }
        $sql = "INSERT INTO addresses (user_id, address_line, city, postal_code, country) VALUES (:user_id, :address_line, :city, :postal_code, :country)";
        $stmt = $this->db->prepare($sql);
        $stmt->execute(['user_id' => $data->user_id, 'address_line' => $data->address_line, 'city' => $data->city, 'postal_code' => $data->postal_code, 'country' => $data->country]);
        return ['status' => 201, 'message' => 'Address created successfully.'];
    }
    public function update($id, $data) {
        if (empty($data->address_line) || empty($data->city) || empty($data->postal_code) || empty($data->country)) {
            return ['status' => 400, 'message' => 'All address fields are required for update.'];
        }
        $sql = "UPDATE addresses SET address_line = :address_line, city = :city, postal_code = :postal_code, country = :country WHERE id = :id";
        $stmt = $this->db->prepare($sql);
        $stmt->execute(['id' => $id, 'address_line' => $data->address_line, 'city' => $data->city, 'postal_code' => $data->postal_code, 'country' => $data->country]);
        return ['status' => 200, 'message' => 'Address updated successfully.'];
    }
    public function delete($id) {
        $stmt = $this->db->prepare("DELETE FROM addresses WHERE id = :id");
        $stmt->execute([':id' => $id]);
        return $stmt->rowCount() > 0 ? ['status' => 200, 'message' => 'Address deleted successfully.'] : ['status' => 404, 'message' => 'Address not found.'];
    }
}
?>
```

##### `src/Controllers/AddressController.php`

```
<?php
namespace App\Controllers;
use App\Models\Address;

class AddressController {
    private $model;
    public function __construct() { $this->model = new Address(); }
    public function index() { echo json_encode($this->model->getAll()); }
    public function show($params) {
        $item = $this->model->getById($params['id']);
        if ($item) echo json_encode($item);
        else { http_response_code(404); echo json_encode(['message' => 'Address not found.']); }
    }
    public function create() {
        $data = json_decode(file_get_contents("php://input"));
        $result = $this->model->create($data);
        http_response_code($result['status']);
        echo json_encode(['message' => $result['message']]);
    }
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

#### **Payments**

##### `src/Models/Payment.php`

```
<?php
namespace App\Models;
use App\Core\Database;
use PDO;

class Payment {
    private $db;
    public function __construct() { $this->db = (new Database())->getConnection(); }
    public function getAll() { return $this->db->query("SELECT * FROM payments ORDER BY created_at DESC")->fetchAll(PDO::FETCH_ASSOC); }
    public function getById($id) {
        $stmt = $this->db->prepare("SELECT * FROM payments WHERE id = :id");
        $stmt->execute([':id' => $id]);
        return $stmt->fetch(PDO::FETCH_ASSOC);
    }
    public function create($data) {
        if (empty($data->order_id) || !isset($data->amount) || empty($data->method)) {
            return ['status' => 400, 'message' => 'Order ID, amount, and method are required.'];
        }
        $sql = "INSERT INTO payments (order_id, amount, method, status) VALUES (:order_id, :amount, :method, :status)";
        $stmt = $this->db->prepare($sql);
        $stmt->execute(['order_id' => $data->order_id, 'amount' => $data->amount, 'method' => $data->method, 'status' => $data->status ?? 'pending']);
        return ['status' => 201, 'message' => 'Payment created successfully.'];
    }
    public function update($id, $data) {
         if (!isset($data->amount) || empty($data->method) || empty($data->status)) {
            return ['status' => 400, 'message' => 'Amount, method, and status are required for update.'];
        }
        $sql = "UPDATE payments SET amount = :amount, method = :method, status = :status WHERE id = :id";
        $stmt = $this->db->prepare($sql);
        $stmt->execute(['id' => $id, 'amount' => $data->amount, 'method' => $data->method, 'status' => $data->status]);
        return ['status' => 200, 'message' => 'Payment updated successfully.'];
    }
    public function delete($id) {
        $stmt = $this->db->prepare("DELETE FROM payments WHERE id = :id");
        $stmt->execute([':id' => $id]);
        return $stmt->rowCount() > 0 ? ['status' => 200, 'message' => 'Payment deleted successfully.'] : ['status' => 404, 'message' => 'Payment not found.'];
    }
}
?>
```

##### `src/Controllers/PaymentController.php`

```
<?php
namespace App\Controllers;
use App\Models\Payment;

class PaymentController {
    private $model;
    public function __construct() { $this->model = new Payment(); }
    public function index() { echo json_encode($this->model->getAll()); }
    public function show($params) {
        $item = $this->model->getById($params['id']);
        if ($item) echo json_encode($item);
        else { http_response_code(404); echo json_encode(['message' => 'Payment not found.']); }
    }
    public function create() {
        $data = json_decode(file_get_contents("php://input"));
        $result = $this->model->create($data);
        http_response_code($result['status']);
        echo json_encode(['message' => $result['message']]);
    }
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

#### **Reviews**

##### `src/Models/Review.php`

```
<?php
namespace App\Models;
use App\Core\Database;
use PDO;

class Review {
    private $db;
    public function __construct() { $this->db = (new Database())->getConnection(); }
    public function getAll() {
        return $this->db->query("SELECT r.*, u.username, p.name as product_name FROM reviews r JOIN users u ON r.user_id = u.id JOIN products p ON r.product_id = p.id ORDER BY r.created_at DESC")->fetchAll(PDO::FETCH_ASSOC);
    }
    public function getById($id) {
        $stmt = $this->db->prepare("SELECT r.*, u.username, p.name as product_name FROM reviews r JOIN users u ON r.user_id = u.id JOIN products p ON r.product_id = p.id WHERE r.id = :id");
        $stmt->execute([':id' => $id]);
        return $stmt->fetch(PDO::FETCH_ASSOC);
    }
    public function create($data) {
        if (empty($data->product_id) || empty($data->user_id) || empty($data->rating)) {
            return ['status' => 400, 'message' => 'Product ID, User ID, and rating are required.'];
        }
        if ($data->rating < 1 || $data->rating > 5) {
             return ['status' => 400, 'message' => 'Rating must be between 1 and 5.'];
        }
        $sql = "INSERT INTO reviews (product_id, user_id, rating, comment) VALUES (:product_id, :user_id, :rating, :comment)";
        $stmt = $this->db->prepare($sql);
        $stmt->execute(['product_id' => $data->product_id, 'user_id' => $data->user_id, 'rating' => $data->rating, 'comment' => $data->comment ?? '']);
        return ['status' => 201, 'message' => 'Review created successfully.'];
    }
    public function update($id, $data) {
        if (empty($data->rating)) {
            return ['status' => 400, 'message' => 'Rating is required for update.'];
        }
        if ($data->rating < 1 || $data->rating > 5) {
             return ['status' => 400, 'message' => 'Rating must be between 1 and 5.'];
        }
        $sql = "UPDATE reviews SET rating = :rating, comment = :comment WHERE id = :id";
        $stmt = $this->db->prepare($sql);
        $stmt->execute(['id' => $id, 'rating' => $data->rating, 'comment' => $data->comment ?? '']);
        return ['status' => 200, 'message' => 'Review updated successfully.'];
    }
    public function delete($id) {
        $stmt = $this->db->prepare("DELETE FROM reviews WHERE id = :id");
        $stmt->execute([':id' => $id]);
        return $stmt->rowCount() > 0 ? ['status' => 200, 'message' => 'Review deleted successfully.'] : ['status' => 404, 'message' => 'Review not found.'];
    }
}
?>
```

##### `src/Controllers/ReviewController.php`

```
<?php
namespace App\Controllers;
use App\Models\Review;

class ReviewController {
    private $model;
    public function __construct() { $this->model = new Review(); }
    public function index() { echo json_encode($this->model->getAll()); }
    public function show($params) {
        $item = $this->model->getById($params['id']);
        if ($item) echo json_encode($item);
        else { http_response_code(404); echo json_encode(['message' => 'Review not found.']); }
    }
    public function create() {
        $data = json_decode(file_get_contents("php://input"));
        $result = $this->model->create($data);
        http_response_code($result['status']);
        echo json_encode(['message' => $result['message']]);
    }
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

### 5.5 ไฟล์กำหนดเส้นทาง (Routes) ฉบับสมบูรณ์

#### `src/routes.php`

ไฟล์นี้จะเชื่อมโยงทุก Endpoint เข้ากับ Controller ที่ถูกต้อง

```
<?php
// PUBLIC ROUTES
$router->post('/auth/login', 'AuthController@login', ['App\Middlewares\RateLimiter']);
$router->post('/users', 'UserController@create', ['App\Middlewares\RateLimiter']);
$router->post('/users/check-existence', 'UserController@checkExistence', ['App\Middlewares\RateLimiter']);
$router->get('/categories', 'CategoryController@index', ['App\Middlewares\RateLimiter']);
$router->get('/categories/{id}', 'CategoryController@show', ['App\Middlewares\RateLimiter']);
$router->get('/products', 'ProductController@index', ['App\Middlewares\RateLimiter']);
$router->get('/products/{id}', 'ProductController@show', ['App\Middlewares\RateLimiter']);
$router->get('/reviews', 'ReviewController@index', ['App\Middlewares\RateLimiter']);
$router->get('/reviews/{id}', 'ReviewController@show', ['App\Middlewares\RateLimiter']);

// AUTHENTICATED ROUTES
$router->post('/orders', 'OrderController@create', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isAuthenticated']);
$router->post('/order-items', 'OrderItemController@create', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isAuthenticated']);
$router->post('/addresses', 'AddressController@create', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isAuthenticated']);
$router->post('/payments', 'PaymentController@create', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isAuthenticated']);
$router->post('/reviews', 'ReviewController@create', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isAuthenticated']);

// PERMISSION-BASED ROUTES
$router->get('/users', 'UserController@index', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isEmployee']);
$router->get('/users/{id}', 'UserController@show', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isSelfOrManager']);
$router->put('/users/{id}', 'UserController@update', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isSelfOrManager']);
$router->post('/users/{id}/change-password', 'UserController@changePassword', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isSelfOrManager']);
$router->delete('/users/{id}', 'UserController@delete', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isAdmin']);
$router->post('/categories', 'CategoryController@create', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isManager']);
$router->put('/categories/{id}', 'CategoryController@update', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isManager']);
$router->delete('/categories/{id}', 'CategoryController@delete', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isManager']);
$router->post('/products', 'ProductController@create', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isManager']);
$router->put('/products/{id}', 'ProductController@update', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isManager']);
$router->delete('/products/{id}', 'ProductController@delete', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isManager']);
$router->get('/orders', 'OrderController@index', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isEmployee']);
$router->get('/orders/{id}', 'OrderController@show', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isOrderOwnerOrManager']);
$router->put('/orders/{id}', 'OrderController@update', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isOrderOwnerOrManager']);
$router->delete('/orders/{id}', 'OrderController@delete', ['App\Middlewares\RateLimiter', 'App\Middlewares\AuthMiddleware@isAdmin']);
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

## ขั้นตอนที่ 6: ทดสอบด้วย Postman (The Final Check)

ตอนนี้ API ของคุณพร้อมเต็ม 100% แล้วครับ คุณสามารถใช้ **Postman Collection** ที่ผมแก้ไขให้ก่อนหน้านี้เพื่อทดสอบทุก Endpoint ได้เลย

* **อย่าลืม**: เปลี่ยนรหัสผ่านใน Request Body ของ `Login` ให้ตรงกับที่คุณตั้งไว้ในฐานข้อมูล

* **การทดสอบ**: เริ่มจากการ Login เพื่อรับ Token จากนั้นนำ Token ไปใส่ใน Header ของ Request อื่นๆ ที่ต้องการการยืนยันตัวตน

## ขั้นตอนที่ 7: การแก้ไขปัญหาและก้าวต่อไป (Troubleshooting & Next Steps)

* **ปัญหาที่พบบ่อย**: `404 Not Found` (เช็ค URL), `500 Internal Server Error` (เช็ค log), `401/403 Unauthorized/Forbidden` (เช็ค Token และ Role)

* **ก้าวต่อไป**: ลองศึกษาเรื่อง **Validation** (การตรวจสอบข้อมูล), **Pagination** (การแบ่งหน้า), การทำ **API Documentation** ด้วย Swagger, และการเขียน **Unit Test** เพื่อให้ API ของคุณสมบูรณ์แบบยิ่งขึ้น

ผมหวังว่าคู่มือฉบับสมบูรณ์นี้จะช่วยให้คุณสร้างและทำความเข้าใจโปรเจกต์ได้ทั้งหมดนะครับ ขอให้สนุกกับการเขียนโค้ดครับ!
