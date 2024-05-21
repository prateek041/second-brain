# To define a controller in Nest

Controllers are responsible for handling incoming requests and returning a response. Routing mechanism is responsible for deciding which controller recieves the request. Each controller may have more than one routes with each route performing a different action.

```TypeScript
@Controller()
class CatsController{
  @Get()
  findAll(): string{
    return "This action returns all cats"
  }
}
```

## @Controller decorator

- This decorator is required to defina basic controller.
- We can pass a prefix to a specific controller


### Routing

Nest employs two different options for manipulating responses.

- *Standard*: Built in method, when a request handler returns a JS object or array, it will be serialised to JSON, but leaves primitive types like (string, number etc.) as it is.

Also, the Nest response code defaults to `200` and `201` which can be changed, check out [this](https://docs.nestjs.com/controllers#routing)
- Library Specific: We can inject the library specific response and request objects, using the `@Res` decorator.
> TODO: More about it later.

### [Request Object](https://docs.nestjs.com/controllers#request-object)

We access the Request object by telling Nest to inject it by adding the `@Req()` decorator to the handler's signature

```TypeScript
import { Request } from 'express'
@Controller()
export class CatsController{
  @Get()
  findAll(@Req() request: Request):string {
  return "This action returns all cats"
  }
}
```

> Note: There are more decorators like @Body, @Query etc.


> TODO: Learn about the Solid Principle
> TODO: Learn about [creating your own decorators in Nest](https://docs.nestjs.com/custom-decorators).
