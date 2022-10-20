# DB-Design

SQL query one to many relationship

## Details of Tables

I have a table for Employees and another table with Training. The training table contains various training classes that the employees have completed. 
We have mandatory security awareness training, so every employee must complete this training class. 
I’m having trouble running a query that will return ALL employees' either listed completing the training or not. <br><br?

## Tables

Example Employee table <br>

╔════╦══════╗ <br>
║ ID ║ NAME ║ <br>
╠════╬══════╣ <br>
║  1 ║ Bob  ║ <br>
║  2 ║ Tom  ║ <br>
║  3 ║ John ║ <br>
╚════╩══════╩ <br>

Example Training table <br>

╔════╦══════════════╦════════════════════╗
║ ID ║ DEPARTMENT_ID║       CLASS        ║
╠════╬══════════════╬════════════════════╣
║  1 ║           1  ║ Security Awareness ║
║  2 ║           1  ║ Workplace Safety   ║
║  3 ║           2  ║ Security Awareness ║
╚════╩══════════════╩════════════════════╝ <br>

Target result <br>

╔════╦══════╦════════════════════╗
║ ID ║ NAME ║       CLASS        ║
╠════╬══════╬════════════════════╣
║  1 ║ Bob  ║ Security Awareness ║
║  2 ║ Tom  ║ Security Awareness ║
║  3 ║ John ║ (null)             ║
╚════╩══════╩════════════════════╝ <br>

Use LEFT JOIN and move the filtering condition during the joining of the table (specifically in the ON clause)

### SQL Query

SELECT  employee.id, employee.name, training.class FROM  employee    <br>
        LEFT JOIN training ON employee.id = training.department_id AND training.class LIKE '%SECURITY%' <br>
        ORDER  BY employee.id <br>
