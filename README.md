
# 42_authentication_tutorial

Learn how to make your website more secure in a simple way using 42 auth ! This tutorial will show you step by step, how to implement it using NestJS, Prisma, and Passport.

In this tutorial, we'll walk through the process of adding 42 authentication to your NestJS application. 42 authentication allows you to log in using your 42 credentials.

# Prerequisites
#### Before we begin, make sure you have the following:

- Node.js installed on your machine.
- NestJS project set up and running.
# Install Dependencies
#### cd into your project and type this command to install Passport and Prisma [Prisma Quickstart Guide](https://www.prisma.io/docs/getting-started/quickstart) ).
```
npm install passport-42 && npm install @nestjs/passport passport
```
# Of course you need where to store user infos! follow this .
### Next, initialize a TypeScript project using npm:

```
npm init -y 
npm install typescript ts-node @types/node --save-dev
```
This creates a package.json with an initial setup for your TypeScript app.

### Now, initialize TypeScript:

```
npx tsc --init
```

Then, install the Prisma CLI as a development dependency in the project:

```
npm install prisma --save-dev
```
Finally, set up Prisma with the init command of the Prisma CLI:

```
npx prisma init --datasource-provider postgresql
```
This creates a new prisma directory with your Prisma schema file and configures SQLite as your database. You're now ready to model your data and create your database with some tables.

### Model your data in the Prisma schema
The Prisma schema provides an intuitive way to model data. Add the following models to your schema.prisma file:

in my case that's what i needed, i'm a little bit lazy haha.

```prisma
prisma/schema.prisma
model User {
  id                  String  @id @default(uuid())
  username            String  @unique
  email               String  @unique
  password_hashed     String  
  display_name        String  
  avatar_url          String
  two_factor_auth     String
  two_factor_secret_key String
  rank                String
}
```

- Models in the Prisma schema have two main purposes:

Represent the tables in the underlying database
Serve as foundation for the generated Prisma Client API
In the next section, you will map these models to database tables using Prisma Migrate.

### Run a migration to create your database tables with Prisma Migrate
At this point, you have a Prisma schema but no database yet. Run the following command in your terminal to create the SQLite database and the User and Post tables represented by your models:

```
npx prisma migrate dev --name init
```
This command did two things:

It creates a new SQL migration file for this migration in the prisma/migrations directory.
It runs the SQL migration file against the database.
Because the SQLite database file didn't exist before, the command also created it inside the prisma directory with the name dev.db as defined via the environment variable in the .env file.

Congratulations, you now have your database and tables ready. Let's go and learn how you can send some queries to read and write data!

### Next !
- Create a docker-compose.yml .
if you're not familiar with docker just copy this and put it in your docker compose.
```docker-compose
version: '3'

services:

  db:
    image: postgres
    restart: always
    environment:
      POSTGRES_PASSWORD: your_Password123 //it's a best practice to put your password in .env file
    ports:
      - 5432:5432
    volumes:
      - DB:/var/lib/postgresql
    
volumes:
  DB:
    driver: local
    driver_opts:
      device: //create a directory when to put your data if the container suddenlt stopped or failed ...and paste it's path here
      type: none
      o: bind
```
In summary, this Docker Compose configuration sets up a PostgreSQL database server in a Docker container, specifies the database password, maps the container's port to the host machine, and ensures that the database data is persisted on your host machine in the specified directory. This is a common setup for running a PostgreSQL database for development or testing purposes in a controlled and isolated environment.

- in .env put this:
```
DATABASE_URL="postgresql://postgres:your_Password123@localhost:5432/test"
```

# Now let's Begin !

- First, you need to be familiar with Nestjs. if you are not ! you better check this beautiful [DOCUMENTATION](https://docs.nestjs.com/)

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
