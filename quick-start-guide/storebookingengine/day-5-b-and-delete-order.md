# Day 5: Leases and B

**Create tables \(if no equivalent tables exist in your system\):**

* **Orders table \(with lease flag\)**
* **OrderItems table**
* **Auth table**
* **Seller table \(booking systems only\)**

**Implement one more complex response:**

* **createOrder**
* **getOrder**
* **bookOrderItem**
* **destroyOrder**

**Tests should pass for B, using test data, and checking getOrder**  


**Implement one more complex response:**

* **createOrder\(lease=true, expiry\)**
* **getOrder \(will work exactly as previously\)**
* **bookOrderItem\(lease=true\)**
* **destroyOrder\(lease=true\)**
* **cleanExpiredOrders\(\)**

**Implement a cron job to call cleanExpiredOrders\(\) with wget**  


**Lease tests should pass for C1 and C2, using test data**

**Note the lease duration should be set to 2 seconds for the lease expiry test**  


