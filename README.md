
# 42_authentication_tutorial

Learn how to make your website more secure in a simple way using 42 auth ! This tutorial will show you step by step, how to implement it using NestJS, Prisma, and Passport.

In this tutorial, we'll walk through the process of adding 42 authentication to your NestJS application. 42 authentication allows you to log in using your 42 credentials.

# Prerequisites
#### Before we begin, make sure you have the following:
- Go to intra setting in the API section  
   ![API](https://github.com/hamza-bouqssim/42_authentication_tutorial/blob/main/uploads/api%20(1).png)  
- Create new app and follow the steps bcs you will need the UID && secretID and type a callbackURL for example type this: `http://localhost:3000/auth/42/redirect`  
  ![APP](https://github.com/hamza-bouqssim/42_authentication_tutorial/blob/main/uploads/uidcsecret%20(1).png)
- You need Node.js installed on your machine.
- And a NestJS project set up and running.
# Install Dependencies
#### cd into your project and type this command to install Passport and Prisma [Prisma Quickstart Guide](https://www.prisma.io/docs/getting-started/quickstart) ).
```
npm install passport-42 && npm install @nestjs/passport passport
```

# Now Let's Begin !

- First, you need to be familiar with Nestjs. if you are not ! you better check this beautiful [DOC](https://docs.nestjs.com/)

generate these needed files by typing this command:
```
npx nest g module auth && npx nest g service auth && npx nest g controller auth
```
This command generates three files for a NestJS module named "auth": a module file (auth.module.ts), a service file (auth.service.ts), and a controller file (auth.controller.ts).
# Controller:
in the auth.controller.ts i have made two endpoints
```ruby
import { Get, Controller, UseGuards } from '@nestjs/common';
import { AuthService } from './auth.service';
import { AuthDto } from './dto/auth.dto';
import { FortyTwoAuthGuard } from './strategy/42.guards';

@Controller('auth')
export class AuthController {
    constructor (private authService: AuthService ){}

    @Get('42/login')
    @UseGuards(FortyTwoAuthGuard)
    ftLogin(){ return{msg:'42 Auth Login'}}
    
    @Get('42/redirect')
    @UseGuards(FortyTwoAuthGuard)
    ftRedirect() { return {msg: '42 OK'}}
}
```
This code set ups routes for authentication using the 42 in a NestJS app.

`/auth/42/login`: When a user visits this route, it checks if they are already authenticated with 42. If they are, it returns a message "42 Auth Login."

`/auth/42/redirect`: This route is used after a user loged in with 42. If the login is successful, it returns a message "42 OK."

The `@UseGuards` decorator its ensures that only authenticated users can access these routes.

# Create 42 Guards
Next, create guards for 42 authentication. Add the following code:

`42 Guards (42.guards.ts)`:
```typescript
import { Injectable, ExecutionContext } from '@nestjs/common';
import { AuthGuard } from '@nestjs/passport';

@Injectable()
export class FortyTwoAuthGuard extends AuthGuard('42') {
  canActivate(context: ExecutionContext) {
    return super.canActivate(context);
  }
}
```
This code is creating a guard for authentication in a NestJS application. i called "FortyTwoAuthGuard," and it's for protecting certain routes from being accessed by users who haven't logged in with a method called "42." The canActivate method checks if a user is authenticated and allows access to a route only if they are. [read more about Guards in Nejst Doc](https://docs.nestjs.com/) .

# Create 42 Strategy
In your NestJS application, create a strategy for 42 authentication. You can do this by adding the following code to your existing codebase:

`42 Strategy (42.strategy.ts)`:
```typescript
import { Injectable } from "@nestjs/common";
import { PassportStrategy } from "@nestjs/passport";
import { AuthService } from "../auth.service";
import { Strategy } from "passport-42";
import { AuthDto } from "../dto/auth.dto";

@Injectable()
export class FortyTwoStrategy extends PassportStrategy(Strategy, '42') {
    constructor(private readonly authService: AuthService){
        super({
            clientID: 'remember when you created an app in 42 intra ! put the UID here', // it's a best practice to put it your .env !!!!!
            clientSecret: 'remember when you created an app in 42 intra ! put the secretID here', // it's a best practice to put it your .env !!!!!
            callbackURL: 'http://localhost:3000/auth/42/redirect',
        });
    }

    async validate(accessToken: string, refreshToken: string, profile: any){
        // Your validation logic here
        // deep into access token and refresh token
        // print the three argument to understand what happens, especially the profile arg, it return the JSON from the 42 api containing the user infos that you will store in your Database or whatever you need .
    }
}
```

This code sets up a strategy for authenticating users with the "42" method in a NestJS application. It uses the PassportStrategy  for handling authentication.

The validate method is where you can add your  logic to validate and create user accounts based on their information.

Before using this code, you should replace 'your-client-id-here' and 'your-client-secret-here' with your actual 42 OAuth credentials.

# Finally
browse this `http://localhost:3000/auth/42/redirect`.
If everything is set up correctly, users should be able to log in using their 42 credentials.
