# SOLID principles explanation (With examples in PHP)

1. [Single responsibility principles (SRP)](#single-responsibility-principle)
2. [Open-closed priciple (OCP)](#open-closed-principle)
3. Liskov substitution principle (LSP)
4. Interface segregation principle (ISP)
5. Dependency injection principle (DIP)

## Single responsibility principle
We keep separate classes by their responsibility.

### Wrong example:
We have class Orders:
```
class Order {
  public function createRequest(Request $request) {
    $product = $this->getProductFromDB($request->productId);
    
    $product = $this->applyDiscount($product);
    
    $order = $this->saveToDatabase($product);
    
    $this->sendCreateOrderNotification($order);
  }
  
  public function deleteRequest(Request $request) {
    $this->deleteOrderFromDb($request->orderId);
    
    $this->sendDeleteOrderNotification();
  }
   
  public function getProductFromDB(int $productId) : Product {/* Get from db logic */}
  
  public function saveToDatabase(Product $product) {/* Save to db logic */}
   
  public function deleteFromDatabase(int $orderId) {/* Delete from db logic */}
  
  public function applyDiscount(Product $product) {/* Discount logic */}
  
  public function sendCreateOrderNotification(Order $order) {/* Notification logic */}

  public function sendDeleteOrderNotification() {/* Notification logic */}
}
```
We may think this class is responsible for Orders, but in fact we have here 5 responsibilities:
1. Processing requests
2. Getting product from database
3. Applying discount
4. Saving and deleting orders
5. Sending notifications to users

### Better example
```
class OrderController {
  public function __constructor(OrderRepository $orderRepository, DiscountService $discountService, OrderNotificationService $orderNotificationService, ProductRepository $productRepository) {
    $this->orderRepository = $orderRepository;
    $this->discountService = $discountService;
    $this->orderNotificationService = $orderNotificator;
    $this->productRepository = $productRepository;
  }

  public function create(Request $request) {
    $product = $this->productRepository->getById($request->productId);
    
    $product = $this->discountService->applyDiscount($product);
    
    $order = $this->orderRepository->create($product);
    
    $this->orderNotificator->sendCreateNotification($order);
  }
}

class OrderRepository {
  public function create(Order $order) : Order {/* Insert order to db logic */}
  public function delete(int $orderId) {/* Delete order from db logic */}
}

class ProductRepository {
  public function getById(int $productId) : Product { /* Get product from db logic */ }
}

class DiscountService {
  public function apply(Product $product) : int {/* Discount logic */}
}

class OrderNotificationService {
  public function sendCreateNotification(Order $order) {/* Create notification logic */}
  public function sendDeleteNotification() {/* Delete notification logic */}
}
```
After refatoring our code we have 5 classes with single responsibility in them:
1. OrderController - processing order requests
2. OrderRepository - saving and deleting orders from db
3. ProductRepository - saving and deleting products from db
4. DiscountService - applying discount
5. OrderNotificationService - send notifications to users

Of couse we can split their responsibilities even futher, for example if we have complicated notification logic, we may split `OrderNotification` class to `OrderPushNotificationService` and `OrderEmailNotificationService`. Just listen to common sense and keep balance.  

## Open-Closed principle
If we are extending functionality of an application, we should not be forced to modify current functionality.

### Wrong example:
```
class UserController {
  public function delete(User $user) {
    if($user instanceof User) {
      /* Delete from db */
      
      return 'User is deleted from database';
    }
    else if($user instanceof AdminUser) {
      /* Delete from db */
      
      return 'Admin is deleted from database';
    }
  }
}

class User {}

class AdminUser extends User {}
```

Imagine we decided to add new type of users, `GuestUser` which is stored in session.

```
class UserController {
  public function delete(User $user) {
    if($user instanceof User) {
      /* Delete from db */
      
      return 'User is deleted from database';
    }
    else if($user instanceof AdminUser) {
      /* Delete from db */
      
      return 'Admin is deleted from database';
    }
    else if($user instanceof GuestUser) {
      /* Delete from session */
      
      return 'Guest data is deleted from session storage';
    }
  }
}

class User {}

class AdminUser extends User {}

class GuestUser extends User {}
```

That's breaking open-closed principle. Because we are forced to make changes to `UserRepository@delete` method, in order to extend functionality with `GuestUser`.

### Better example

```
class UserController {
  public function delete(User $user) {
    return $user->delete();
  }
}

class User {
  public function delete() {
    /* Delete from db */
    
    return 'User is deleted from database';
  }
}

class AdminUser extends User {
  public function delete() {
    /* Delete from db */
    
    return 'Admin is deleted from database';
  }
}

class GuestUser extends User {
  public function delete() {
    /* Delete from db */
    
    return 'Guest data is deleted from session storage';
  }
}
```

We have delegated `delete` responsibility from `UserController` to `User`, `AdminUser` and `GuestUser`, that way satisfying open-closed principle. If we would like to add new `SupportUser`, we would be able to do it without modifying `UserController@delete` method functionality.

We can futher improve our code by delegating `/* Delete from db */` to DatabaseUserRepository and `/* Delete from session */` to SessionUserRepository, that way we will satisfy single responsibility principle as well. 

## Liskov substitution principle

When we have `Parent` class extended by `Child`, we should be able to replace `Parent` with `Child` without breaking functionality of an application.

### Wrong example
```
class DeliveryController {
  public function getCost(Product $product, int $distance, int $amount) : int {
    return $product->calculateDelivery($distance) * $amount;
  }
}

class Product {
  private $deliveryCostPerKm = 20;

  public function calculateDelivery(int $distance) : int {
    return $this->deliveryCostPerKm * $distance;
  }
}

class Sand extends Product {
  public function calculateDelivery(int $distance, int $weight) : int {
    return $this->deliveryCostPerKm * $distance * $weight;
  }
}

class Cartoon extends Product {
  public function calculateDelivery($distance) {
    throw new Exception('Can't deliver online product');
  }
}
```

What if we would pass to `DeliveryController@getCost` -> `Product`, `Sand` or `Cartoon` instances:

```
const controller = new DeliveryController();

$controller->getCost(new Product(), 10, 10); // successfully returns price of product (10 * 10 * 20 = 2000)
$controller->getCost(new Sand(), 10, 10); // fails because we have not passed `$weight` parameter to `Sand@calculateDelivery` 
$controller->getCost(new Cartoon(), 10, 10); // fails because `Cartoon@calculateDelivery` is throwing an Exception that it's parent doesn't have
```

So we have failed Liksov substitution principle, because parameters and Exceptions of classes derived from `Product` not the same.

### Better example
We would need to add an Interface to define a contract for all classes that implements it.

```
/**
 * @property int $deliveryCostPerKm;
 */
interface ProductInterface {
    /**
     * @throws NotDeliverableException if the product is not possible to deliver
     */
    public function calculateDelivery(int $distance, int $weight): int;
}
```

According to this contract we would need to make few changes to the code:
1. Add exception handling to `DeliveryController`
2. Add `$weight` parameter to the `ProductController` and `Product` classes

**Complete code:**
```
/**
 * @property int $deliveryCostPerKm;
 */
interface ProductInterface {
    /**
     * @throws NotDeliverableException if the product is not possible to deliver
     */
    public function calculateDelivery(int $distance, int $weight): int;
}

class DeliveryController {
  public function getCost(Product $product, int $distance, int $amount, int $weight = 0) : int {
    try {
      return $product->calculateDelivery($distance, $weight) * $amount;
    }
    catch(NotDeliverableException $exception) {
      /* Exception handling logic */
      
      return 0;
    }
  }
}

class Product implements ProductInterface {
  private $deliveryCostPerKm = 20;

  public function calculateDelivery(int $distance, $weight = 0) : int {
    return $this->deliveryCostPerKm * $distance;
  }
}

class Sand extends Product {
  public function calculateDelivery(int $distance, int $weight) : int {
    return $this->deliveryCostPerKm * $distance * $weight;
  }
}

class Cartoon extends Product {
  public function calculateDelivery($distance) {
    throw new Exception('Can't deliver online product');
  }
}
```

If we run the code now, we would see following result.

```
const controller = new DeliveryController();

$controller->getCost(new Product(), 10, 10); // returns price of product (10 * 10 * 20 = 2000)
$controller->getCost(new Sand(), 10, 10, 15); //  returns price of product (10 * 10 * 20 * 2 = 4000)
$controller->getCost(new Cartoon(), 10, 10); // handles errors, and returns zero delivery cost
```
