# SOLID principles explanation (With examples in PHP)

1. Single responsibility principles (SRP)
2. Open-closed priciple (OCP)
3. Liskov substitution principle (LSP)
4. Interface segregation principle (ISP)
5. Dependency injection principle (DIP)

## Single responsibility principle
We keep separate classes by their responsibility.
### Wrong Example:
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
   
  public function getProductFromDB($productId) {/* Get from db logic */}
  
  public function saveToDatabase($product) {/* Save to db logic */}
   
  public function deleteFromDatabase($orderId) {/* Delete from db logic */}
  
  public function applyDiscount($product) {/* Discount logic */}
  
  public function sendCreateOrderNotification($order) {/* Notification logic */}

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
  public function create($order) {/* Insert order to db logic */}
  public function delete($orderId) {/* Delete order from db logic */}
}

class ProductRepository {
  public function getById($productId) { /* Get product from db logic */ }
}

class DiscountService {
  public function apply($product) {/* Discount logic */}
}

class OrderNotificationService {
  public function sendCreateNotification($order) {/* Create notification logic */}
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
