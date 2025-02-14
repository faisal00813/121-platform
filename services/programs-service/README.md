Programs-Service
================

The Programs Service is the backend for the  *Humanitarian Organization* web portal, where 121-programs can be created and monitored, etc.

----------

## Getting started / Installation

Clone the repository

    git clone https://github.com/global-121/121.global.git

Switch to the repository folder

    cd service/programs-service/

Install dependencies

    npm install

Copy config file and set JsonWebToken secret key

    cp src/secrets.ts.example src/secrets.ts

----------

## Database

The example codebase uses [TypeORM](http://typeorm.io/) with a Dockerized Postgres database.

Install Docker & create a Docker network (needed because we will create separate Docker containers for DB and for app, which need to be linked) through:

    docker network create 121network

Create a new local Postgres-database through Docker with:

    docker run --name 121db -p 5438:5432 -e POSTGRES_USER=global121 -e POSTGRES_PASSWORD=global121 -e POSTGRES_DB=global121 -t --restart always --network 121network -v ${PWD}/postgresql.conf:/etc/postgresql.conf -d postgres:9.6 -c 'config_file=/etc/postgresql.conf'

Copy TypeORM config example file for database settings

    cp ormconfig.json.example ormconfig.json

& fill in the password.

```json
{
  "type": "postgres",
  "host": "121db",
  "port": 5432,
  "username": "global121",
  "password": "<secret>",
  "database": "global121",
  "entities": ["src/**/**.entity{.ts,.js}"],
  "dropSchema": true,
  "synchronize": true
}
```

On application start, tables for all entities will be created automatically.

When running this setup locally, you can access the dockerized database for example through pgAdmin (or any other GUI) on port 5438 (as that's where the docker container's port forwarding goes).

----------

## Start application locally

Create the Docker image from the Dockerfile in this folder through:

    docker build -t 121-node .

Start the app Docker container through (NOTE: the ${PWD} code for current directory may not translate to all OS's):

    docker run --name=121-programs-service -v ${PWD}:/home/121 -p 3000:3000 -it --network 121network 121-node

If you've already created the container before and just want to start again:

    docker start -i 121-programs-service

The Docker container automatically runs `npm start` (defined in Dockerfile)
Possibly rebuild/rerun by changing this to:
- `npm run start:dev` (uses `tswatch` instead of `nodemon`)
- `npm run start:watch` (to use with `nodemon` for restart upon change)

## Start application on VM

Same as bove. But replace '-it' tag in 'docker run' or 'docker start' commands by '-d' to run in detached mode.
Also, the CMD line of Dockerfile should be changed from CMD ["npm", "run", "start:dev"] to CMD ["npm", "start"].

## How to use Swagger (with authorization features)
Access Swagger API via `http://localhost:3000/docs`

#### Signup/Signin
- If you have no users in your database yet, start with 'USER /POST user'. Leave the default input as is, and execute.
- If you already have created the above user earlier, start with 'USER /POST user/login'. Leave the default input as is, and execute.
- In either case, copy the value of the Token-attribute from the output.
- Click 'Authorize' (top-right) and fill in `Bearer <copied token>`
- This will now give you access to all hitherto forbidden API-calls.
- NOTE: for ease of development, if not logged in, it will take the default-user. So you do need to create this default user with email test@test.nl, but the Authorize part is not necessary any more. Otherwise you would need to repeat the Authorize-setup after each refresh of Swagger, i.e. after each code change.
#### Admin vs Fieldworker
- Different authorizations for admin or fieldworker are added.
- In USER /POST you can set role='admin' or role='aidworker'.
- With 'admin' you have access to all API-calls
- With 'aidworker' you have access only to (most) GET requests
- Only the USER /POST call is completely open at the moment, as otherwise you cannot create a first admin-user. To improve in the future.
- NOTE: this admin/aidworker distinction is only working if you're using the Bearer authentication described above. If not, then the default-user will be used, which will have admin-rights automatically (even if you haven't specified role='admin' for that user initially).
#### Using other endpoints
A typical flow to use other endpoints (after being signed up/in):
- Create programs with POST /programs
- Get a list of all programs with GET /programs 
- Create 2 countries Malawi and Ethiopia with POST /countrys (NOTE: countries will be relevant e.g. to preload a list of standard criteria based on country, which you can subsequently choose to use in your program or not. You can additionally add custom criteria as well.)
- Create criterium 'age' with POST /criterium and add it to country Ethiopia with PUT /countrys/{countryId}
- Create criterium 'gender' with POST /criterium and add it to country Malawi with PUT /countrys/{countryId}
- Create criterium-options 'male' and 'female' for criterium 'gender' with POST /criterium-options/{criteriumId}



----------

## Other relevant NPM scripts

- `npm start` - Start application
- `npm run start:watch` - Start application in watch mode
- `npm run test` - run Jest test runner
- `npm run start:prod` - Build application

----------

## Authentication

This applications uses [JSON Web Token](https://jwt.io/) (JWT) to handle authentication. The token is passed with each request using the `Authorization` header with `Token` scheme. The JWT authentication middleware handles the validation and authentication of the token.

----------

## Swagger API docs

We use the NestJS swagger module for API documentation. [NestJS Swagger](https://github.com/nestjs/swagger) - [swagger.io](https://swagger.io/)
