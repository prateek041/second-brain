## Request handlers
These are the functions that receive all incoming HTTP traffic connection. They have the following signature

```go
func(w http.ResponseWriter, r *http.Request)
```
- http.ResponseWriter is where we write all text/html response.
- http.Request contains all the information about the HTTP request.

### My SQL Database
Go has a package named `database/sql`to query all sorts of SQL databases. This unifies all common use-cases of an SQL database into one API. Go does not have database drivers, which are implemented separated by each database itself.

Drivers are low level implementation of details of a specific database.