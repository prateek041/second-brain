# Go has support for Generics similar to other languages.

## Basic Types of Parameters in Go

- **Type Parameters in Functions**:
- **Function Parameters**:
- **Type parameters in Structs**:
- **Type Constraints**:

### Generic Function

```go
package main

func print [T any](){
  fmt.Println(T)
}

func main(){
  Print(123)
  Print("hello")
  Print(3.14)
}
```

### Generic Structs

```go
package main

type Box[T any]struct {
    value T
}

func main(){
  number := Box[int]{value: 123}
  word := Box[string]{value: "hello"}
}
```

### Type Constraints

We can provide constraints to a functions, defining on what types that function can run, this is usually done using interfaces.

```go
package main

type CheckThis interface{
  Read() []string
  Write() [] string
}

// one struct to implement the CheckThis interface
type CheckType struct {
  name string
}

func (c *CheckType)Read()[]string{
  // method implementation
}

func (c *CheckType)Write()[]string{
  // method implementation
}

func checkFunc [C CheckThis](hello string) {
  // function implementation
}

// second struct to implement the CheckThis interface
type SecondCheckType struct {
  name string
}

func (c *SecondCheckType)Read()[]string{
  // method implementation
}

func (c *SecondCheckType)Write()[]string{
  // method implementation
}

func checkFunc [C CheckThis](hello string) {
  // function implementation
}

func main (){
  testVar := CheckType{name: "test name"}
  checkFunc[testVar]("test name again")
}

```

### Function Parameters

They define what types can

### Function Parameters vs Type Parameters

In the checkFunc above, there are two types of parameters, CheckThis of type C, which is called Type Parameter, then it takes one argument, which is function parameter.

> [!NOTE]
> Go supports generics, so we can define functions that can work on multiple types as long as they fall under the constraints.

We can implement constraints using interfaces, like in the above example, FilteredLabels interface is used, that has two methods keys and values, even though in our example we only have ProcessLabels struct, which implement the FilteredLabels interface, but we can add more in the future and we just would have to make sure that new struct implements the filteredLabels interface and the code would still work.

Now, the MustNewGranularCounter is a function that works on any type that implements the FilteredLabels interface, so when this function is called, we have to specify a type, for example, metrics.MustNewGranularCounter[metrics.ProcessLabels](opts, extraLabels), now since the ProcessLabels implements the Filtered Labels interface, this will work.

Now, MustNewGranularCounter is calling NewGranularCounter, which returns a GranularCounter of the type which I passed while calling the MustNewGranularCounter (i.e. ProcessLabels.
