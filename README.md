# SOLID, YAGNI, KISS, DRY Prinsipləri və Kod Nümunələri

Bu sənəd SOLID prinsiplərini, eləcə də YAGNI, KISS və DRY prinsiplərini izah edir. Hər bir prinsip üçün **səhv** və **düzgün** real dünya kod nümunələri Java dilində verilmişdir.

---

## Single Responsibility Principle (SRP)
Bir sinifin yalnız bir dəyişmə səbəbi olmalıdır, yəni yalnız bir vəzifəsi olmalıdır.

### Səhv Nümunə
```java
public class UserManager {
    public void createUser(String name, String email) {
        // İstifadəçi yaradılması
        User user = new User(name, email);
        saveToDatabase(user);
        sendWelcomeEmail(user);
    }

    private void saveToDatabase(User user) {
        // Verilənlər bazasına yazma
        System.out.println("Saving " + user + " to database");
    }

    private void sendWelcomeEmail(User user) {
        // E-poçt göndərmə
        System.out.println("Sending welcome email to " + user.getEmail());
    }
}

class User {
    private String name;
    private String email;

    public User(String name, String email) {
        this.name = name;
        this.email = email;
    }

    public String getEmail() {
        return email;
    }
}
```
**Problem**: `UserManager` istifadəçi yaradılması, verilənlər bazası əməliyyatları və e-poçt göndərmə kimi bir neçə vəzifəni yerinə yetirir, bu SRP-ni pozur.

### Düzgün Nümunə
```java
public class User {
    private String name;
    private String email;

    public User(String name, String email) {
        this.name = name;
        this.email = email;
    }

    public String getName() {
        return name;
    }

    public String getEmail() {
        return email;
    }
}

interface UserRepository {
    void save(User user);
}

class DatabaseUserRepository implements UserRepository {
    @Override
    public void save(User user) {
        System.out.println("Saving " + user.getName() + " to database");
    }
}

interface EmailService {
    void sendWelcomeEmail(User user);
}

class SmtpEmailService implements EmailService {
    @Override
    public void sendWelcomeEmail(User user) {
        System.out.println("Sending welcome email to " + user.getEmail());
    }
}

public class UserManager {
    private UserRepository repository;
    private EmailService emailService;

    public UserManager(UserRepository repository, EmailService emailService) {
        this.repository = repository;
        this.emailService = emailService;
    }

    public void createUser(String name, String email) {
        User user = new User(name, email);
        repository.save(user);
        emailService.sendWelcomeEmail(user);
    }
}
```
**Həll**: Hər sinif yalnız bir vəzifəyə malikdir: `User` məlumatları, `UserRepository` saxlama, `EmailService` e-poçt göndərmə, `UserManager` isə əməliyyatları idarə edir.

---

## Open-Closed Principle (OCP)
Siniflər genişləndirməyə açıq, lakin dəyişikliyə qapalı olmalıdır.

### Səhv Nümunə
```java
public class PaymentProcessor {
    public void processPayment(String paymentType, double amount) {
        if (paymentType.equals("credit_card")) {
            System.out.println("Processing credit card payment of " + amount);
        } else if (paymentType.equals("paypal")) {
            System.out.println("Processing PayPal payment of " + amount);
        }
    }
}
```
**Problem**: Yeni ödəniş növü əlavə etmək üçün `PaymentProcessor` sinfini dəyişmək lazımdır, bu OCP-ni pozur.

### Düzgün Nümunə
```java
interface PaymentMethod {
    void process(double amount);
}

class CreditCardPayment implements PaymentMethod {
    @Override
    public void process(double amount) {
        System.out.println("Processing credit card payment of " + amount);
    }
}

class PayPalPayment implements PaymentMethod {
    @Override
    public void process(double amount) {
        System.out.println("Processing PayPal payment of " + amount);
    }
}

public class PaymentProcessor {
    public void processPayment(PaymentMethod paymentMethod, double amount) {
        paymentMethod.process(amount);
    }
}
```
**Həll**: Yeni ödəniş növləri `PaymentMethod` interfeysini implementasiya etməklə əlavə edilə bilər, `PaymentProcessor` dəyişmədən.

---

## Liskov Substitution Principle (LSP)
Alt siniflər öz əsas sinifləri ilə əvəzlənə bilməlidir və proqramın düzgünlüyünə xələl gətirməməlidir.

### Səhv Nümunə
```java
public class Bird {
    public void fly() {
        System.out.println("Flying");
    }
}

class Ostrich extends Bird {
    @Override
    public void fly() {
        throw new UnsupportedOperationException("Ostriches can't fly");
    }
}
```
**Problem**: `Ostrich` `Bird` ilə əvəzlənə bilməz, çünki `fly` metodunun davranışını pozur.

### Düzgün Nümunə
```java
interface Bird {
}

interface FlyingBird extends Bird {
    void fly();
}

class Sparrow implements FlyingBird {
    @Override
    public void fly() {
        System.out.println("Sparrow flying");
    }
}

class Ostrich implements Bird {
    public void run() {
        System.out.println("Ostrich running");
    }
}
```
**Həll**: `Ostrich` `Bird` interfeysini implementasiya edir, lakin uçma qabiliyyəti yalnız `FlyingBird` üçün tələb olunur.

---

## Interface Segregation Principle (ISP)
Müştərilər istifadə etmədikləri interfeyslərdən asılı olmamalıdır.

### Səhv Nümunə
```java
interface Worker {
    void work();
    void eat();
}

class HumanWorker implements Worker {
    @Override
    public void work() {
        System.out.println("Human working");
    }

    @Override
    public void eat() {
        System.out.println("Human eating");
    }
}

class RobotWorker implements Worker {
    @Override
    public void work() {
        System.out.println("Robot working");
    }

    @Override
    public void eat() {
        throw new UnsupportedOperationException("Robots don't eat");
    }
}
```
**Problem**: `RobotWorker` istifadə etmədiyi `eat` metodunu implementasiya etməyə məcburdur.

### Düzgün Nümunə
```java
interface Workable {
    void work();
}

interface Eatable {
    void eat();
}

class HumanWorker implements Workable, Eatable {
    @Override
    public void work() {
        System.out.println("Human working");
    }

    @Override
    public void eat() {
        System.out.println("Human eating");
    }
}

class RobotWorker implements Workable {
    @Override
    public void work() {
        System.out.println("Robot working");
    }
}
```
**Həll**: Ayrı interfeyslər (`Workable`, `Eatable`) siniflərin yalnız lazım olan metodları implementasiya etməsini təmin edir.

---

## Dependency Inversion Principle (DIP)
Yüksək səviyyəli modullar aşağı səviyyəli modullardan asılı olmamalıdır. Hər ikisi abstraksiyalardan asılı olmalıdır.

### Səhv Nümunə
```java
class SQLDatabase {
    public void save(String data) {
        System.out.println("Saving " + data + " to SQL database");
    }
}

public class UserManager {
    private SQLDatabase database;

    public UserManager() {
        this.database = new SQLDatabase();
    }

    public void saveUser(String user) {
        database.save(user);
    }
}
```
**Problem**: `UserManager` birbaşa `SQLDatabase` sinfinə bağlıdır, bu başqa verilənlər bazasına keçidi çətinləşdirir.

### Düzgün Nümunə
```java
interface Database {
    void save(String data);
}

class SQLDatabase implements Database {
    @Override
    public void save(String data) {
        System.out.println("Saving " + data + " to SQL database");
    }
}

class MongoDatabase implements Database {
    @Override
    public void save(String data) {
        System.out.println("Saving " + data + " to MongoDB");
    }
}

public class UserManager {
    private Database database;

    public UserManager(Database database) {
        this.database = database;
    }

    public void saveUser(String user) {
        database.save(user);
    }
}
```
**Həll**: `UserManager` `Database` abstraksiyasından asılıdır, bu da verilənlər bazalarının asanlıqla dəyişdirilməsinə imkan verir.

---

## YAGNI (You Aren't Gonna Need It)
Lazım olmayan funksionallığı əlavə etməyin.

### Səhv Nümunə
```java
public class ShoppingCart {
    public void addItem(String item) {
        System.out.println("Adding " + item);
    }

    public void removeItem(String item) {
        System.out.println("Removing " + item);
    }

    public void applyDiscount(String discountCode) {
        System.out.println("Discount not implemented yet");
    }

    public void exportToPdf() {
        System.out.println("PDF export not implemented yet");
    }
}
```
**Problem**: `applyDiscount` və `exportToPdf` hazırda lazım olmayan spekulyativ funksiyalardır.

### Düzgün Nümunə
```java
public class ShoppingCart {
    public void addItem(String item) {
        System.out.println("Adding " + item);
    }

    public void removeItem(String item) {
        System.out.println("Removing " + item);
    }
}
```
**Həll**: Yalnız `addItem` və `removeItem` metodları implementasiya olunur, digər funksiyalar lazım olduqda əlavə edilir.

---

## KISS (Keep It Simple, Stupid)
Sistemlər mümkün qədər sadə olmalıdır.

### Səhv Nümunə
```java
public class TemperatureConverter {
    public double convert(double value, String fromUnit, String toUnit) {
        Map<String, Function<Double, Double>> conversions = new HashMap<>();
        conversions.put("C_F", x -> (x * 9/5) + 32);
        conversions.put("F_C", x -> (x - 32) * 5/9);
        conversions.put("C_K", x -> x + 273.15);
        conversions.put("K_C", x -> x - 273.15);
        String key = fromUnit + "_" + toUnit;
        return conversions.getOrDefault(key, x -> x).apply(value);
    }
}
```
**Problem**: Mürəkkəb xəritə və lambda funksiyaları kodu oxumaq və saxlamaq üçün çətinləşdirir.

### Düzgün Nümunə
```java
public class TemperatureConverter {
    public double celsiusToFahrenheit(double celsius) {
        return (celsius * 9/5) + 32;
    }

    public double fahrenheitToCelsius(double fahrenheit) {
        return (fahrenheit - 32) * 5/9;
    }

    public double celsiusToKelvin(double celsius) {
        return celsius + 273.15;
    }

    public double kelvinToCelsius(double kelvin) {
        return kelvin - 273.15;
    }
}
```
**Həll**: Sadə və açıq metodlar kodu oxunaqlı və saxlanıla bilən edir.

---

## DRY (Don't Repeat Yourself)
Eyni kodu təkrarlamamaq üçün ümumi funksionallığı abstraktlaşdırın.

### Səhv Nümunə
```java
public class ReportGenerator {
    public void generatePdfReport(String data) {
        System.out.println("Generating PDF header");
        System.out.println("PDF content: " + data);
        System.out.println("Generating PDF footer");
    }

    public void generateHtmlReport(String data) {
        System.out.println("Generating HTML header");
        System.out.println("HTML content: " + data);
        System.out.println("Generating HTML footer");
    }
}
```
**Problem**: Başlıq və altlıq yaradılması loqikası təkrarlanır.

### Düzgün Nümunə
```java
public class ReportGenerator {
    private void generateHeader(String format) {
        System.out.println("Generating " + format + " header");
    }

    private void generateFooter(String format) {
        System.out.println("Generating " + format + " footer");
    }

    public void generatePdfReport(String data) {
        generateHeader("PDF");
        System.out.println("PDF content: " + data);
        generateFooter("PDF");
    }

    public void generateHtmlReport(String data) {
        generateHeader("HTML");
        System.out.println("HTML content: " + data);
        generateFooter("HTML");
    }
}
```
**Həll**: Ümumi başlıq və altlıq loqikası təkrar istifadə edilə bilən metodlara çıxarılıb.
