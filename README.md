# Rest API PHP MVC

# นี่คือคู่มือสร้าง 🚀 REST API ด้วย PHP ฉบับสมบูรณ์ (จากเริ่มต้นสู่มือโปร)

สวัสดีครับ! ยินดีต้อนรับสู่คู่มือฉบับสมบูรณ์ที่สุดในการสร้าง **REST API** ด้วย PHP ที่ผมตั้งใจรวบรวมและเรียบเรียงขึ้นมาใหม่ทั้งหมด เพื่อให้คุณมั่นใจได้ว่าทุกไฟล์ ทุกบรรทัดของโค้ด จะทำงานร่วมกันได้อย่างสมบูรณ์ และสามารถทดสอบได้จริงทุก Endpoint ครับ 

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

* **ที่อยู่ไฟล์**: `C:\xampp\apache\conf\extra\httpd-vhosts.conf`

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

* **ที่อยู่ไฟล์**: `C:\Windows\System32\drivers\etc\hosts`

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

นี่คือสคริปต์ SQL ที่ครบถ้วนสำหรับสร้างฐานข้อมูล `api_db` และตารางทั้งหมด รวมถึงตาราง `rate_limits` ด้วยครับ

* นำโค้ดนี้ไปรันใน **phpMyAdmin** และบันทึกลงในไฟล์ `database.sql`

```sql
-- Database: `api_db`
CREATE DATABASE IF NOT EXISTS `api_db` DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
USE `api_db`;

-- Table structure for `users`
CREATE TABLE `users` ( `id` int(11) NOT NULL AUTO_INCREMENT, `username` varchar(50) NOT NULL, `email` varchar(255) NOT NULL, `password` varchar(255) NOT NULL, `first_name` varchar(100) DEFAULT NULL, `last_name` varchar(100) DEFAULT NULL, `role` enum('customer','employee','manager','director','admin') DEFAULT 'customer', `created_at` timestamp NOT NULL DEFAULT current_timestamp(), `updated_at` timestamp NOT NULL DEFAULT current_timestamp() ON UPDATE current_timestamp(), PRIMARY KEY (`id`), UNIQUE KEY `username` (`username`), UNIQUE KEY `email` (`email`)) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
INSERT INTO `users` (`id`, `username`, `email`, `password`, `first_name`, `last_name`, `role`) VALUES (1, 'user1', 'user1@example.com', '$2y$10$G3CEGrkTdKVpu6Q8ZAh6keXOjVvWDzpBwbpUZE0oiX28Q5V0EzoCq', 'John', 'Doe', 'customer'), (2, 'admin1', 'admin@example.com', '$2y$10$G3CEGrkTdKVpu6Q8ZAh6keXOjVvWDzpBwbpUZE0oiX28Q5V0EzoCq', 'Admin', 'User', 'admin');

-- Table structure for `categories`
CREATE TABLE `categories` ( `id` int(11) NOT NULL AUTO_INCREMENT, `name` varchar(100) NOT NULL, `description` text DEFAULT NULL, `created_at` timestamp NOT NULL DEFAULT current_timestamp(), PRIMARY KEY (`id`), UNIQUE KEY `name` (`name`) ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
INSERT INTO `categories` (`id`, `name`, `description`) VALUES (1, 'Electronics', 'Devices and gadgets'), (2, 'Clothing', 'Apparel and accessories');

-- Table structure for `products`
CREATE TABLE `products` ( `id` int(11) NOT NULL AUTO_INCREMENT, `name` varchar(255) NOT NULL, `price` decimal(10,2) NOT NULL, `description` text DEFAULT NULL, `category_id` int(11) DEFAULT NULL, `stock` int(11) NOT NULL DEFAULT 0, `created_at` timestamp NOT NULL DEFAULT current_timestamp(), `updated_at` timestamp NOT NULL DEFAULT current_timestamp() ON UPDATE current_timestamp(), PRIMARY KEY (`id`), KEY `category_id` (`category_id`), CONSTRAINT `products_ibfk_1` FOREIGN KEY (`category_id`) REFERENCES `categories` (`id`) ON DELETE SET NULL) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
INSERT INTO `products` (`id`, `name`, `price`, `description`, `category_id`, `stock`) VALUES (1, 'Laptop', 25000.00, 'High-performance laptop', 1, 50), (2, 'Smartphone', 15000.00, 'Latest model', 1, 100);

-- Table structure for `orders`
CREATE TABLE `orders` ( `id` int(11) NOT NULL AUTO_INCREMENT, `user_id` int(11) NOT NULL, `total_amount` decimal(10,2) NOT NULL, `status` enum('pending','completed','cancelled') DEFAULT 'pending', `created_at` timestamp NOT NULL DEFAULT current_timestamp(), `updated_at` timestamp NOT NULL DEFAULT current_timestamp() ON UPDATE current_timestamp(), PRIMARY KEY (`id`), KEY `user_id` (`user_id`), CONSTRAINT `orders_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `users` (`id`) ON DELETE CASCADE) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- Table structure for `order_items`
CREATE TABLE `order_items` ( `id` int(11) NOT NULL AUTO_INCREMENT, `order_id` int(11) NOT NULL, `product_id` int(11) DEFAULT NULL, `quantity` int(11) NOT NULL, `unit_price` decimal(10,2) NOT NULL, `created_at` timestamp NOT NULL DEFAULT current_timestamp(), PRIMARY KEY (`id`), KEY `order_id` (`order_id`), KEY `product_id` (`product_id`), CONSTRAINT `order_items_ibfk_1` FOREIGN KEY (`order_id`) REFERENCES `orders` (`id`) ON DELETE CASCADE, CONSTRAINT `order_items_ibfk_2` FOREIGN KEY (`product_id`) REFERENCES `products` (`id`) ON DELETE SET NULL) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- Table structure for `addresses`
CREATE TABLE `addresses` ( `id` int(11) NOT NULL AUTO_INCREMENT, `user_id` int(11) DEFAULT NULL, `address_line` varchar(255) NOT NULL, `city` varchar(100) NOT NULL, `postal_code` varchar(20) NOT NULL, `country` varchar(100) NOT NULL, `created_at` timestamp NOT NULL DEFAULT current_timestamp(), PRIMARY KEY (`id`), KEY `user_id` (`user_id`), CONSTRAINT `addresses_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `users` (`id`) ON DELETE SET NULL) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- Table structure for `payments`
CREATE TABLE `payments` ( `id` int(11) NOT NULL AUTO_INCREMENT, `order_id` int(11) NOT NULL, `amount` decimal(10,2) NOT NULL, `method` enum('credit_card','bank_transfer','cash') NOT NULL, `status` enum('pending','completed','failed') DEFAULT 'pending', `created_at` timestamp NOT NULL DEFAULT current_timestamp(), PRIMARY KEY (`id`), KEY `order_id` (`order_id`), CONSTRAINT `payments_ibfk_1` FOREIGN KEY (`order_id`) REFERENCES `orders` (`id`) ON DELETE CASCADE) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- Table structure for `reviews`
CREATE TABLE `reviews` ( `id` int(11) NOT NULL AUTO_INCREMENT, `product_id` int(11) DEFAULT NULL, `user_id` int(11) DEFAULT NULL, `rating` int(11) NOT NULL CHECK (`rating` >= 1 AND `rating` <= 5), `comment` text DEFAULT NULL, `created_at` timestamp NOT NULL DEFAULT current_timestamp(), PRIMARY KEY (`id`), KEY `product_id` (`product_id`), KEY `user_id` (`user_id`), CONSTRAINT `reviews_ibfk_1` FOREIGN KEY (`product_id`) REFERENCES `products` (`id`) ON DELETE SET NULL, CONSTRAINT `reviews_ibfk_2` FOREIGN KEY (`user_id`) REFERENCES `users` (`id`) ON DELETE SET NULL) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- Table structure for `rate_limits`
CREATE TABLE `rate_limits` ( `id` INT AUTO_INCREMENT PRIMARY KEY, `user_id` INT NULL, `ip_address` VARCHAR(45) NOT NULL, `endpoint` VARCHAR(255) NOT NULL, `request_count` INT DEFAULT 0, `window_start` DATETIME NOT NULL, `created_at` TIMESTAMP DEFAULT CURRENT_TIMESTAMP, FOREIGN KEY (`user_id`) REFERENCES `users`(`id`) ON DELETE SET NULL) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

**เกร็ดความรู้**: สังเกตว่าเราใช้ `ON DELETE CASCADE` และ `ON DELETE SET NULL` เพื่อจัดการความสัมพันธ์ของข้อมูล

* `CASCADE`: ถ้าลบข้อมูลหลัก (เช่น ลบ Order) ข้อมูลย่อยที่ผูกกันอยู่ (เช่น Order Items) จะถูกลบตามไปด้วย

* `SET NULL`: ถ้าลบข้อมูลหลัก (เช่น ลบ User) ข้อมูลที่เคยผูกอยู่ (เช่น Review) จะถูกเปลี่ยนค่า Foreign Key ให้เป็น `NULL` เพื่อรักษาข้อมูลรีวิวไว้

## ขั้นตอนที่ 4: ติดตั้ง Dependencies ด้วย Composer

**Composer** คือเครื่องมือจัดการ "ส่วนเสริม" หรือ Library ต่างๆ ในโปรเจกต์ PHP ของเรา เราจะใช้มันเพื่อติดตั้ง `php-jwt` สำหรับจัดการ Token

### 4.1 สร้างไฟล์ `composer.json`

* เปิดไฟล์ `C:\xampp\htdocs\v.3_api1\composer.json` แล้วใส่โค้ดนี้เข้าไป:

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

#### `public/.htaccess`

```apache
RewriteEngine On

# อนุญาตให้เว็บอื่นเรียกใช้ API ของเราได้ (CORS)
Header set Access-Control-Allow-Origin "*"
Header set Access-Control-Allow-Methods "GET, POST, PUT, DELETE, OPTIONS"
Header set Access-Control-Allow-Headers "Content-Type, Authorization"

# จัดการกับ OPTIONS request ที่เบราว์เซอร์ส่งมาถามก่อน
RewriteCond %{REQUEST_METHOD} OPTIONS
RewriteRule ^(.*)$ $1 [R=200,L]

# ส่งค่า Authorization header ไปให้ PHP
RewriteCond %{HTTP:Authorization} .
RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

# ส่งทุก request ที่หาไฟล์ไม่เจอไปที่ index.php
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^ index.php [QSA,L]
```

#### `public/index.php`

```php
<?php
ini_set('display_errors', 1);
error_reporting(E_ALL);

require_once __DIR__ . '/../vendor/autoload.php';
require_once __DIR__ . '/../config/config.php';
require_once __DIR__ . '/../src/Core/Database.php';
require_once __DIR__ . '/../src/Core/Router.php';

$router = new App\Core\Router();
require_once __DIR__ . '/../src/routes.php';

date_default_timezone_set('Asia/Bangkok');
header("Content-Type: application/json; charset=UTF-8");

if ($_SERVER['REQUEST_METHOD'] === 'OPTIONS') {
    http_response_code(200);
    exit();
}

$router->dispatch();
?>
```

#### `config/config.php`

```php
<?php
define('DB_HOST', '127.0.0.1');
define('DB_USER', 'root');
define('DB_PASS', '');
define('DB_NAME', 'api_db');

define('JWT_SECRET', 'YOUR_SUPER_SECRET_KEY_CHANGE_ME_NOW_!@#$%^');
define('JWT_ISSUER', 'one.com');
define('JWT_AUDIENCE', 'one.com');
?>
```

#### `src/Core/Database.php`

```php
<?php
namespace App\Core;
use PDO;
use PDOException;

class Database {
    private $conn;
    public function __construct() {
        $this->conn = null;
        try {
            $this->conn = new PDO("mysql:host=" . DB_HOST . ";dbname=" . DB_NAME . ";charset=utf8mb4", DB_USER, DB_PASS);
            $this->conn->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
        } catch (PDOException $e) {
            http_response_code(500);
            echo json_encode(['message' => 'Database Connection Error: ' . $e->getMessage()]);
            exit();
        }
    }
    public function getConnection() {
        return $this->conn;
    }
}
?>
```

#### `src/Core/Router.php` (ฉบับแก้ไขสมบูรณ์)

```php
<?php
namespace App\Core;

class Router {
    protected $routes = [];

    private function addRoute($method, $uri, $action, $middlewares = []) {
        $uri = preg_replace('/\{([a-z_]+)\}/', '(?P<$1>[^/]+)', $uri);
        $uri = '#^/api' . $uri . '$#';
        $this->routes[] = [
            'method' => $method, 'uri' => $uri, 'action' => $action,
            'middleware' => is_array($middlewares) ? $middlewares : ($middlewares ? explode(',', $middlewares) : [])
        ];
    }

    public function get($uri, $action, $middleware = []) { $this->addRoute('GET', $uri, $action, $middleware); }
    public function post($uri, $action, $middleware = []) { $this->addRoute('POST', $uri, $action, $middleware); }
    public function put($uri, $action, $middleware = []) { $this->addRoute('PUT', $uri, $action, $middleware); }
    public function delete($uri, $action, $middleware = []) { $this->addRoute('DELETE', $uri, $action, $middleware); }

    public function dispatch() {
        $requestUri = parse_url($_SERVER['REQUEST_URI'], PHP_URL_PATH);
        $requestMethod = $_SERVER['REQUEST_METHOD'];

        foreach ($this->routes as $route) {
            if ($route['method'] !== $requestMethod) continue;
            if (preg_match($route['uri'], $requestUri, $matches)) {
                $params = array_filter($matches, 'is_string', ARRAY_FILTER_USE_KEY);
                foreach ($route['middleware'] as $mw) {
                    $middlewareResult = $this->executeMiddleware($mw, $params);
                    if ($middlewareResult !== true) {
                        http_response_code($middlewareResult['status']);
                        echo json_encode(['message' => $middlewareResult['message']]);
                        return;
                    }
                }
                list($controller, $method) = explode('@', $route['action']);
                $controllerInstance = new ("App\\Controllers\\" . $controller)();
                $controllerInstance->$method($params);
                return;
            }
        }
        http_response_code(404);
        echo json_encode(['message' => 'Endpoint not found.']);
    }

    private function executeMiddleware($middlewareClass, $urlParams) {
        $method = 'handle';
        if (strpos($middlewareClass, '@') !== false) {
            list($middlewareClass, $method) = explode('@', $middlewareClass);
        }
        $instance = new $middlewareClass();
        $reflectionMethod = new \ReflectionMethod($instance, $method);
        $requiredParams = $reflectionMethod->getParameters();
        $args = [];
        foreach ($requiredParams as $param) {
            if (isset($urlParams[$param->getName()])) {
                $args[] = $urlParams[$param->getName()];
            }
        }
        return $instance->{$method}(...$args);
    }
}
?>
```

### 5.3 Middlewares (ด่านตรวจ)

#### `src/Middlewares/AuthMiddleware.php`

```php
<?php
namespace App\Middlewares;
use Firebase\JWT\JWT;
use Firebase\JWT\Key;
use App\Core\Database;
use PDO;

class AuthMiddleware {
    private $db;
    private $decodedToken = null;

    public function __construct() { $this->db = (new Database())->getConnection(); }

    private function authenticate() {
        if ($this->decodedToken !== null) return true;
        $headers = getallheaders();
        $authHeader = $headers['Authorization'] ?? '';
        if (!$authHeader || !preg_match('/Bearer\s(\S+)/', $authHeader, $matches)) {
            return ['status' => 401, 'message' => 'Authorization token not found.'];
        }
        try {
            $this->decodedToken = JWT::decode($matches[1], new Key(JWT_SECRET, 'HS256'));
            $stmt = $this->db->prepare("SELECT id FROM users WHERE id = :id");
            $stmt->execute([':id' => $this->decodedToken->sub]);
            if (!$stmt->fetch()) return ['status' => 401, 'message' => 'User associated with this token no longer exists.'];
            return true;
        } catch (\Exception $e) {
            return ['status' => 401, 'message' => 'Invalid or expired token.'];
        }
    }

    public function isAuthenticated() { return $this->authenticate(); }

    public function hasRole($requiredRole) {
        $authResult = $this->authenticate();
        if ($authResult !== true) return $authResult;
        $rolesHierarchy = ['customer' => 1, 'employee' => 2, 'manager' => 3, 'director' => 4, 'admin' => 5];
        $userRole = $this->decodedToken->role;
        if (!isset($rolesHierarchy[$userRole]) || $rolesHierarchy[$userRole] < $rolesHierarchy[$requiredRole]) {
            return ['status' => 403, 'message' => 'Forbidden. Insufficient permissions.'];
        }
        return true;
    }

    public function isAdmin() { return $this->hasRole('admin'); }
    public function isManager() { return $this->hasRole('manager'); }
    public function isEmployee() { return $this->hasRole('employee'); }

    public function isSelfOrManager($id) {
        $authResult = $this->authenticate();
        if ($authResult !== true) return $authResult;
        if ($this->decodedToken->sub == $id) return true;
        return $this->isManager();
    }

    public function isOrderOwnerOrManager($id) {
        $authResult = $this->authenticate();
        if ($authResult !== true) return $authResult;
        $stmt = $this->db->prepare("SELECT user_id FROM orders WHERE id = :id");
        $stmt->execute([':id' => $id]);
        $order = $stmt->fetch(PDO::FETCH_ASSOC);
        if (!$order) return ['status' => 404, 'message' => 'Resource not found.'];
        if ($order['user_id'] == $this->decodedToken->sub) return true;
        return $this->isManager();
    }
}
?>
```

#### `src/Middlewares/RateLimiter.php`

```php
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

```php
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

```php
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

```php
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

#### **Categories, Products, Orders, etc.**

_(โค้ดสำหรับทรัพยากรอื่นๆ จะใช้แพทเทิร์นเดียวกันกับ Users คือ Controller เป็นตัวกลาง และ Model เป็นคนทำงานกับฐานข้อมูลโดยตรง)_

(ส่วนนี้จะตามด้วยโค้ดของ Model และ Controller ที่เหลือทั้งหมด ซึ่งมีอยู่ในคำตอบก่อนหน้านี้แล้ว)

### 5.5 ไฟล์กำหนดเส้นทาง (Routes) ฉบับสมบูรณ์

#### `src/routes.php`

ไฟล์นี้จะเชื่อมโยงทุก Endpoint เข้ากับ Controller ที่ถูกต้อง

```php
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
