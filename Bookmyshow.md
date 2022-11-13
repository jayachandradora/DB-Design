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

### High Level Design
![image](https://user-images.githubusercontent.com/115500959/201508381-f1edbfdc-e448-4480-bec9-3286a6dca521.png)

### Extension to these design
1. Payment gate way
	- Walet payment
	- Gift voucher payment
	- Net banking
	- redme points 	
2. Cache service
3. Meta data Service
4. Search service with large extent (In-memory store for suggestion and search result for ELK stack and high search item in the cache service.
5. User contact service - this required while booking a ticket for sending notification.

### Concurrency

- How to handle concurrency; such that no two users are able to book same seat ?
- We can use transactions in SQL databases to avoid any clashes.
- For example:- if we are using SQL server we can utilize Transaction Isolation Levels to lock the rows before we can update them.

Below is the sample code:

![image](https://user-images.githubusercontent.com/115500959/199647736-ec152af3-7565-4735-8628-0ccfeab4cf03.png)

- ‘Serializable’ is the highest isolation level and guarantees safety from Dirty, Nonrepeatable and Phantoms reads.
- One thing to note here, within a transaction if we read rows we get a write lock on them so that they can’t be updated by anyone else.
- Once the above database transaction is successful, we can start tracking the reservation in ActiveReservationService.

## API Design
```
https://in.bookmyshow.com/api/explore/
https://in.bookmyshow.com/api/explore/{cities}/
{
	{
		"movies" : [
			"RRR",
			"Kashmir files",
			"Border"
		]
	}
}
https://in.bookmyshow.com/api/explore/cities/{RRR}/
{
	"Movie Details": {
		"ratting" : "3",
		"Name" : RRR,
		"Release Dt": "12/12/22"
	}
}
https://in.bookmyshow.com/api/explore/cities/{RRR}/bookticket
{
	"Show Details": {
		[ Cinema: "Inox", "timing" : [2:00, 6:00, 10:00], Prices: [100, 200, 300]],
		[ Cinema: "Inox", "timing" : [2:00, 6:00, 10:00], Prices: [100, 200, 300]],
		[ Cinema: "Inox", "timing" : [2:00, 6:00, 10:00], Prices: [100, 200, 300]]
}

https://in.bookmyshow.com/api/explore/cities/{RRR}/bookticket/{5}/reserveseats/
{
	"seats": [1,2,3,4],
	"booking Date" : "11/11/22",
	"Show time" : "10:00AM",
	"Show Date" : "11/11/22",
	"total price": "2000"
	"Convenience fees": 200,
	"Food & Beverage": 300
}

List<City> getCities()
List<Theater> getTheaters(city)
List<Show> getShows(theater)
List<List<Seats>> getSeats(show)
Bill researveSeat(seats)
Invoice makePayment(bill)
```
