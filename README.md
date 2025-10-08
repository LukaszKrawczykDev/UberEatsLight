# 🍔 Mini Uber Eats – Spring Boot + Vaadin

Aplikacja edukacyjna inspirowana Uber Eats, stworzona w 100% w Javie z użyciem **Spring Boot (backend + REST)** oraz **Vaadin (frontend webowy)**.  
Projekt wspiera wiele ról użytkowników: **Klient, Restauracja (Admin), Kurier**.  

---

## ✨ Funkcjonalności

### 👤 Klient
- Rejestracja i logowanie
- Dodawanie i edycja adresów dostawy
- Przeglądanie restauracji i menu
- Składanie zamówień (koszyk, cena dostawy)
- Śledzenie statusu zamówienia
- Wystawianie ocen restauracji i kurierom (po dostawie)

### 🏪 Restauracja (Admin)
- Zarządzanie danymi restauracji (nazwa, adres, cena dostawy)
- Dodawanie i edycja dań w menu
- Obsługa zamówień (statusy: `ACCEPTED → PREPARING → READY`)
- Przypisywanie kurierów do zamówień
- Zarządzanie kontami kurierów
- Podgląd ocen klientów

### 🚴 Kurier
- Przegląd przypisanych zamówień
- Aktualizacja statusu dostawy (`IN_DELIVERY → DELIVERED`)
- Podgląd własnych ocen

---

## 📡 REST API – Endpoints

> Wszystkie endpointy zabezpieczone Spring Security (JWT/Session), z wyjątkiem rejestracji/logowania.

### 🔓 Publiczne
| Metoda | Endpoint             | Opis |
|--------|----------------------|------|
| `POST` | `/api/auth/register` | Rejestracja klienta |
| `POST` | `/api/auth/login`    | Logowanie, zwraca token |

### 👤 Klient (`CLIENT`)
| Metoda | Endpoint                           | Opis |
|--------|------------------------------------|------|
| `GET`  | `/api/restaurants`                 | Lista restauracji |
| `GET`  | `/api/restaurants/{id}/menu`       | Menu restauracji |
| `POST` | `/api/addresses`                   | Dodanie adresu |
| `GET`  | `/api/addresses`                   | Lista adresów użytkownika |
| `POST` | `/api/orders`                      | Złożenie zamówienia (koszyk + adres) |
| `GET`  | `/api/orders`                      | Lista zamówień klienta |
| `GET`  | `/api/orders/{id}`                 | Szczegóły zamówienia |
| `POST` | `/api/reviews`                     | Dodanie recenzji (po `DELIVERED`) |

### 🏪 Restauracja (Admin) (`RESTAURANT_ADMIN`)
| Metoda | Endpoint                           | Opis |
|--------|------------------------------------|------|
| `GET`  | `/api/admin/orders`                | Lista zamówień restauracji |
| `PUT`  | `/api/admin/orders/{id}/status`    | Zmiana statusu zamówienia |
| `PUT`  | `/api/admin/orders/{id}/courier`   | Przypisanie kuriera |
| `POST` | `/api/admin/dishes`                | Dodanie dania |
| `PUT`  | `/api/admin/dishes/{id}`           | Edycja dania |
| `DELETE` | `/api/admin/dishes/{id}`         | Usunięcie dania |
| `GET`  | `/api/admin/couriers`              | Lista kurierów restauracji |
| `POST` | `/api/admin/couriers`              | Dodanie kuriera |
| `DELETE` | `/api/admin/couriers/{id}`       | Usunięcie kuriera |
| `PUT`  | `/api/admin/restaurant`            | Edycja danych restauracji (w tym cena dostawy) |

### 🚴 Kurier (`COURIER`)
| Metoda | Endpoint                           | Opis |
|--------|------------------------------------|------|
| `GET`  | `/api/courier/orders`              | Lista przypisanych zamówień |
| `PUT`  | `/api/courier/orders/{id}/status`  | Aktualizacja statusu (`IN_DELIVERY → DELIVERED`) |

---

## 📋 Plan implementacji (Checklist)

### 1) Przygotowanie projektu
1. Repozytorium: Git + README + `.gitignore` (Java, Maven/Gradle, IDE).
2. Archetyp (Spring Initializr):  
   - spring-boot-starter-web  
   - spring-boot-starter-data-jpa  
   - spring-boot-starter-security  
   - vaadin-spring-boot-starter  
   - spring-boot-starter-validation  
   - DB driver: PostgreSQL (prod) + H2 (dev)  
   - Migracje: Liquibase albo Flyway  
   - (opc.) Lombok, DevTools
3. Profile: `application.yml` dla `dev` (H2, testowe loginy) i `prod` (Postgres).
4. Build: Maven/Gradle + plugin Vaadin (production build), JDK 17/21.

---

### 2) Migracje i model danych (DDL)
- Tabele: **users, restaurants, dishes, addresses, order_statuses, orders, order_items, reviews**.  
- Klucze, indeksy, relacje FK, ograniczenia.  
- Snapshoty cen w `orders` i `order_items` dla spójności.

---

### 3) Encje JPA
- User, Restaurant, Dish, Address, OrderStatus, Order, OrderItem, Review  
- Bean Validation: `@NotBlank`, `@Positive`, `@Email`, `@Min(1)`.

---

### 4) Repozytoria
- UserRepository, RestaurantRepository, DishRepository, AddressRepository,  
  OrderStatusRepository, OrderRepository, OrderItemRepository, ReviewRepository.

---

### 5) Warstwa serwisów
- **AuthService** – rejestracja/logowanie klientów (BCrypt)  
- **RestaurantService** – zarządzanie restauracją  
- **DishService** – CRUD menu  
- **AddressService** – CRUD adresów  
- **OrderService** – logika zamówień + zmiany statusów + przydział kurierów  
- **CourierService** – zarządzanie kurierami  
- **ReviewService** – recenzje dostępne tylko po `DELIVERED`  
- **NotificationService** (opc.) – Vaadin Push do real-time update.

---

### 6) Bezpieczeństwo
- Spring Security + BCrypt  
- Role: `CLIENT`, `RESTAURANT_ADMIN`, `COURIER`  
- Ochrona tras w Vaadin, ograniczenia biznesowe, CSRF.  

---

### 7) UI (Vaadin)
- **Layout**: MainLayout z menu i AppBar  
- **Klient**: RestaurantsView, MenuView, CartView, MyOrdersView, ReviewDialog  
- **Admin restauracji**: DashboardView, MenuManagementView, OrdersManagementView, CouriersManagementView, RestaurantSettingsView  
- **Kurier**: AssignedOrdersView, HistoryView  
- Real-time update: Vaadin Push na zdarzenia statusów.

---

### 8) Walidacje i reguły biznesowe
- delivery_price ≥ 0, dish.price ≥ 0, quantity ≥ 1  
- wszystkie pozycje w zamówieniu muszą pochodzić z jednej restauracji  
- adres należy do zalogowanego klienta  
- recenzje tylko po `DELIVERED`.  

---

### 9) Obsługa błędów
- `@ControllerAdvice` + ExceptionHandlers  
- Optimistic Locking (@Version w Order)  
- Spójne kody błędów domenowych (np. `ORDER_INVALID_STATUS_TRANSITION`).  

---

### 10) Testy
- Unit (serwisy)  
- Integration (SpringBootTest + Testcontainers)  
- Security (autoryzacja, dostęp)  
- Data (migracje + seed).  

---

### 11) Dane startowe (seed)
- Role i statusy  
- 1–2 restauracje + admini  
- Kurierzy (po 2 na restaurację)  
- Klienci (2–3)  
- Menu i przykładowe zamówienia.  

---

### 12) Uruchomienie i packaging
```bash
# dev mode (H2, seed data)
mvn spring-boot:run

# build produkcyjny
mvn clean package -Pproduction
