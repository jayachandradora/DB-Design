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
