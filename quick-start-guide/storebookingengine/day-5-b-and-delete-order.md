# Day 5: Leases and B

## Step 1: Understand the StoreBookingEngine booking flow

The StoreBookingEngine handles booking of each item, as well as creating the overall Lease or Order within a transaction.

The diagram below illustrates the abstract methods the StoreBookingEngine calls. Note that the OpportunityStore used for each set of OrderItems of each opportunity type is based on the  OpportunityStore routing configured in Day 4.

![Methods called by the StoreBookingEngine](../../.gitbook/assets/openactive-tooling-flows.png)

## Step **2**: Understand the StoreBookingEngine booking flow

Check that there are enough spaces in total to allow each



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


