# Your Express TypeScript API with Swagger Integration

## Overview

This project is a RESTful API built with **Express** and **TypeScript** that integrates **Swagger** for API documentation. The application employs **async handlers** for better error management and uses a modular structure for routers, allowing for easy expansion and organization of the application.

## Features

- **Express Framework**: A lightweight web application framework for building web applications in Node.js.
- **TypeScript**: Provides strong typing for better development experience and less runtime errors.
- **Swagger Integration**: Automatically generates interactive API documentation, making it easy for developers and consumers to understand the available endpoints.
- **Async Handlers**: Supports async/await syntax for writing cleaner and more maintainable asynchronous code without deeply nested callbacks.
- **Modular Router Structure**: Each router is automatically imported and mounted, providing clean separation of endpoints and logic.

## Technologies Used

- Node.js
- Express
- TypeScript
- Zod (for input validation)
- ts-zod-decorators (for validation using Zod with decorators)
- utils-decorators (for middleware utilities like throttling and error handling)
- Swagger (for API documentation)

## Installation

To get started with the project, follow these steps:

1. **Clone the repository**:

   ```bash
   git clone https://github.com/your-repo/your-project.git
   cd your-project
   ```

2. **Install dependencies**:

   ```bash
   npm install
   ```

3. **Build the TypeScript files** (if necessary):

   ```bash
   npm run build
   ```

4. **Run the application**:

   ```bash
   npm start
   ```

5. **Visit the Swagger UI**: Open your browser and go to `http://localhost:3000/api-docs` to view the automatically generated API documentation.

## Sample Project Structure

/src  
├── /routers # Directory containing all router files  
│ ├── /user # Routers for user-related endpoints  
│ │ ├── index.ts  
│ │ ├── user_router1.ts  
│ │ ├── user_router2.ts  
│ │ ├── user.controller.ts  
│ │ ├── user.service.ts  
│ │ ├── user.dto.ts  
│ ├── /product # Router for product-related endpoints  
│ │ ├── index.ts  
│ │ ├── product.spec.ts  
│ │ ├── product.controller.ts  
│ │ ├── product.service.ts  
│ │ ├── dto  
│ │ │ ├── product.dto.ts  
│ │ ├── /order # Router for order-related endpoints  
│ │ │ ├── index.ts  
│ │ │ ├── order_test.spec.ts  
│ │ │ ├── order.controller.ts  
│ │ │ ├── order.service.ts  
│ └── index.ts # Home page  
├── app.ts # Main application file  
├── config.ts # Application configuration file  
└── ... # Other files (middleware, models, etc.)

### Router Directory

Each router file in the `/routers` directory is organized to handle related endpoints. The `index.ts` file in the router directory automatically imports all routers and mounts them to the main Express application, making it simple to add new routes without modifying central files.

Files that end with `.controller`, `.service`, `.spec`, `.dto`, `.middleware`, `.error`, or `.decorator`, as well as those that start with `_` or `@`, are excluded from the router list and can be utilized for various other purposes within the application.

Example of a basic router:

```typescript
import { Router, Request, Response } from "express";
import asyncHandler from "express-async-handler";

const router = Router();

/**
 * @swagger
 * /:
 *   get:
 *     summary: Get the home page
 *     description: Returns the home page.
 *     responses:
 *       200:
 *         description: home page
 */
router.get(
  "/",
  asyncHandler(async (req: Request, res: Response) => {
    res.send("Home");
  })
);

export = router;
```

### Controllers

The **Controllers** in this Express TypeScript API act as the intermediary between the incoming HTTP requests and the application logic. Each controller is responsible for handling specific routes and defining the behavior associated with those routes. This organization promotes a clean architecture by separating business logic, validation, and routing concerns.

Controllers can be organized within the router folders, allowing them to stay closely related to their respective routes. However, they are not limited to this structure and can be placed anywhere within the `src` folder as needed, providing flexibility in organizing the codebase.

Controllers leverage decorators from the `ts-zod-decorators` package to implement input validation and error handling gracefully. With validators like `@Validate`, controllers can ensure that incoming data adheres to defined schemas before any processing occurs, preventing invalid data from reaching the service layer. This capability enhances data integrity and application reliability.

For example, in the provided `UserController`, the `createUser` method demonstrates how to apply input validation and error handling through decorators. It employs `@rateLimit` to restrict the number of allowed requests within a specified timeframe, effectively guarding against too many rapid submissions. When an error arises, the @onError decorator provides a systematic way to handle exceptions, allowing for logging or other error management processes to be performed centrally.

Here’s a brief breakdown of key components used in the `UserController`:

- **Validation**: The `CreateUserDto` is validated against incoming data using the `@ZodInput` decorator, ensuring that only well-formed data is passed to the business logic, which is crucial for maintaining application stability.

```typescript
import { z } from "zod";

export const CreateUserDto = z.object({
  username: z.string().min(3),
  email: z.string().email(),
  password: z.string().min(6),
});

export type CreateUser = z.infer<typeof CreateUserDto>;
```

- **Throttling and Rate Limiting**: The `@rateLimit` decorator is applied to safeguard the application's endpoints from abuse by limiting how frequently a particular method can be invoked.

- **Error Handling**: The `@onError` decorator captures any exceptions that occur during the execution of the createUser method, allowing for centralized error management, which can greatly simplify debugging and improve maintainability.

By using a well-organized controller structure, this project makes it easier to add, modify, and manage endpoints as the application grows. Developers can focus on implementing business logic while the controllers handle the intricacies of request parsing, validation, and response formatting. Additionally, this separation of concerns improves unit testing, as controllers can be tested independently from the rest of the application logic, ensuring robust and reliable API behavior.

Here is a quick reference to the UserController in practice:

```typescript
import { Validate, ZodInput } from "ts-zod-decorators";
import { CreateUserDto, CreateUser } from "./dto/user.dto";
import { onError, rateLimit, throttle } from "utils-decorators";

function exceedHandler() {
  throw new Error("Too much call in allowed window");
}

function errorHandler(e: Error): void {
  console.error(e);
}

export default class UserController {
  //constructor(private readonly userService: UserService) { }

  // Throttle the createUser method to 1 request per 200 milliseconds
  @rateLimit({
    timeSpanMs: 60000,
    allowedCalls: 300,
    exceedHandler,
  })
  @onError({
    func: errorHandler,
  })
  @Validate
  public async createUser(
    @ZodInput(CreateUserDto) data: CreateUser
  ): Promise<string> {
    // Here you can include logic to save user to database
    console.log("Creating user:", data);
    return "User created successfully";
  }
}
```

This structure not only supports effective code organization but also ensures that each part of the application is working towards the same goal: a scalable, maintainable, and robust API.

### Async Handlers

To maintain cleaner code and improve error handling, the app suggests the `express-async-handler` package that automatically catches exceptions inside of async express routes and passing them to the express error handlers. This allows you to focus on writing the business logic without worrying about try/catch blocks.

### API Documentation

Once the application is running, visit the Swagger UI at http://localhost:3000/api-docs. This automatically generated documentation will provide you with all available routes along with details on request parameters, response structures, and possible error codes.

### Decorators

To prevent abuse of your API, you can utilize throttling, and validation decorators from the `utils-decorators` and `ts-zod-decorators` packages respectively. This packages provides decorators that can be applied directly to your service and controller classes.

### Contributing

Contributions are welcome! If you have suggestions for improvements or new features, feel free to create an issue or submit a pull request.
