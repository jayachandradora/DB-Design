# Bookmyshow DB-Design

### Database Design:

```
Here are a few observations about the data we are going to store:

- Each City can have multiple Cinemas.
- Each Cinema will have multiple halls.
- Each Movie will have many Shows, and each Show will have multiple Bookings.
- A user can have multiple bookings.
```
![image](https://user-images.githubusercontent.com/115500959/199643618-17b4c14a-50e0-4372-8ea7-88107dc02081.png)

### Ticket Booking Workflow

![image](https://user-images.githubusercontent.com/115500959/199647656-2fc850f9-564d-4f46-9e7d-31f4cf71eb9c.png)

### Concurrency

- How to handle concurrency; such that no two users are able to book same seat ?
- We can use transactions in SQL databases to avoid any clashes.
- For example:- if we are using SQL server we can utilize Transaction Isolation Levels to lock the rows before we can update them.

Below is the sample code:

![image](https://user-images.githubusercontent.com/115500959/199647736-ec152af3-7565-4735-8628-0ccfeab4cf03.png)

- ‘Serializable’ is the highest isolation level and guarantees safety from Dirty, Nonrepeatable and Phantoms reads.
- One thing to note here, within a transaction if we read rows we get a write lock on them so that they can’t be updated by anyone else.
- Once the above database transaction is successful, we can start tracking the reservation in ActiveReservationService.
