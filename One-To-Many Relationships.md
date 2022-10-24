# DB-Design

SQL query one to many relationship

## Details of Tables

I have a table for Employees and another table with Training. The training table contains various training classes that the employees have completed. 
We have mandatory security awareness training, so every employee must complete this training class. 
I’m having trouble running a query that will return ALL employees' either listed completing the training or not. <br><br?

## Tables

Example Employee table <br>

![image](https://user-images.githubusercontent.com/115500959/196854126-5549b43a-e66d-4449-8200-a5f2b5463fa3.png)


Example Training table <br>
![image](https://user-images.githubusercontent.com/115500959/196855229-5857bf55-bfb1-4343-b550-d2f696e42ef5.png)
 <br>

Target result <br>

![image](https://user-images.githubusercontent.com/115500959/196855309-a8f4202b-7207-4ebb-b2e5-57040ce69132.png)<br>

Use LEFT JOIN and move the filtering condition during the joining of the table (specifically in the ON clause)

### SQL Query

SELECT  employee.id, employee.name, training.class FROM  employee    <br>
        LEFT JOIN training ON employee.id = training.department_id AND training.class LIKE '%SECURITY%' <br>
        ORDER  BY employee.id <br>

### One to Many - One Question can have many Answers
![image](https://user-images.githubusercontent.com/115500959/196873943-1442d031-2bf8-4650-a120-311b0794e005.png)

### For More Understanding
![image](https://user-images.githubusercontent.com/115500959/197506878-7090ac99-94d9-405a-b16a-62091a8856f5.png)

DB Design Online: <br>
https://dbdiagram.io/d/6355fbcc4709410195c5a6d5 <br>

Data Base Practice online: <br>
https://www.programiz.com/sql/online-compiler/ <br>

INSERT INTO jd_product VALUES (1, 'shirt', "Nike Shirt");<br>
INSERT INTO jd_product VALUES (2, 'pant', "Addidas pant");<br>
INSERT INTO jd_product VALUES (3, 'inner', "Reebok inner");<br>
INSERT INTO jd_product VALUES (4, 'inner pant', "jokey inner pant");<br>


CREATE TABLE color ( <br>
  cid numeric(10) NOT NULL,   <br>
  color varchar2(50), <br>
  CONSTRAINT size_pk PRIMARY KEY (cid)<br>
); <br>

CREATE TABLE jd_product ( <br>
  id numeric(10) NOT NULL,  <br>
  product_name varchar2(50) NOT NULL,  <br>
  description varchar2(50), <br>
  CONSTRAINT product_pk PRIMARY KEY(id) <br>
); <br>
 

CREATE TABLE size ( <br>
  sid numeric(10) NOT NULL,   <br>
  size_value varchar2(50), <br>
  CONSTRAINT size_pk PRIMARY KEY (sid) <br>
); <br>

CREATE TABLE product_entry ( <br>
  id numeric(10) not null, <br>
  pid numeric(10) not null, <br>
  sid numeric(10) , <br>
  cid numeric(10) , <br>
  CONSTRAINT pid_jd_product FOREIGN KEY(pid) REFERENCES jd_product(id),   <br>
  CONSTRAINT size_id_size FOREIGN KEY(sid) REFERENCES size(sid), <br>
  CONSTRAINT color_id_color FOREIGN KEY(cid) REFERENCES color(cid), <br>
  CONSTRAINT product_entry_pk PRIMARY KEY (id) <br>
);  <br>

INSERT INTO Product_entry VALUES (4, 4, 3, 3); <br>



select c.customer_id, c.first_name, c.age, s.status from Customers c <br>
	inner join Orders o on c.customer_id = o.order_id <br>
    inner join Shippings s on c.customer_id = s.shipping_id <br>
	
