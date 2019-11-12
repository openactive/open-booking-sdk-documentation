# Day 1: Fake Implementation

### **Day 1 - Objective: test database works locally**

Summary: Install the library, hook up to stub in-memory db library \(using ref implementation as an example\), and pass the validator with test fake database \(uses a test prefix or some identifier pattern in response by default, so as to ensure the validator can recognise \_test mode\_\)  


Copy and paste from reference implementation, and update the details to fit in your environment, creating a working stub by instantiating StubImplementation  


Create the following endpoints, 

* Dataset Site
* Open Data RPDE feeds
* OrderQuote Creation \(C1 & C2\)
* OrderQuote Creation \(C2\)
* OrderQuote Deletion
* Order Creation \(B\)
* Order Deletion
* Order Cancellation
* Orders RPDE Feed
* Order Status

Create the internal application endpoints / scheduled CRON for:

* Cleanup Expired Leases \(if required\)

Create test-harness endpoints for:

* Reset test data \(wired to fake database\)
* Trigger provider cancellation \(wired to fake database\)
* Change logistics \(wired to fake database\)

All tests should now pass  


