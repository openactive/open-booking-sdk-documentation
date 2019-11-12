# Known Limitations

## StoreBookingEngine

The StoreBookingEngine has the following limitations, which can be overcome if implementers would find this helpful:

### Application-layer Transactions 

The booking is orchestrated in the library separately invoking various `Stores` that  each make separate calls to the database. There is not currently any facility that allows an application to create a database-level transaction \(for example by initialising and passing a [DbContextTransaction](https://docs.microsoft.com/en-us/dotnet/api/system.data.entity.dbcontexttransaction?view=entity-framework-6.2.0) through\). This can be added if implementers would consider it helpful.



