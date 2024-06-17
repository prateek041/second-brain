# Type Assertion in Go

There are many instances while building your application when you will know what the type of a variable is, but Go will not, in that cas you can use Type Assertion.

In case of a web server, it can be seen in

```go
type Product struct {
  ID int
  name string
}

func (p *Product)UpdateProduct(rw http.ResponseWriter, r *http.Request){
  prod := r.Context().Value(ProductKey).(Product)
}

// empty unique identifier for Product, so it can be picked from the context
type ProductKey struct {}

func (p *Product)Middleware(next http.Handler) http.Handler{
  return http.HandlerFunc(func (rw http.ResponseWriter, *r http.Request){
  prod = &Product{}
  // unmarshal the data into the prod variable

  // update the context in Request
  ctx := context.WithValue(r.Context(), ProductKey, prod)
  r := r.Context(ctx)
  next.ServeHTTP(rw, r)
  })

}
```

In the above example, we have two functions, Middleware is unmarshalling the Product from the request Body. And then putting it inside the request context so that main handlers can access it.

Then the Update handler is picking it up from the request Context, but `r.Context().Value(ProductKey)` returns a value of type `any/interface{}` but other functions like the ones interacting with the database expect something of type Product. So, we `assert` that the value returned by `r.Context().Value(ProductKey)` is Product, which changes `prod` from `any/interface{}` to `Product`
