# multi-docker

Multi-container deployment using Docker, Travis-CI, AWS Elastic Beanstalk, AWS Elasticache (Redis) & AWS RDS (Postgres).

Travis-CI will build a dev environment (./client/Dockerfile.dev) from all merges to the master branch, it will then run test functions on the container. 
After a successful test, Travis-CI will then build the remaining containers, push to Docker Hub and deploy to AWS Elastic Beanstalk (using the docker-compose.yml file in the root directory).

Note: you must set-up an AWS Elastic Beanstalk, AWS Elasticache & AWS RDS Instance to deploy this project to production. You can deploy a local dev environment with docker-compose-dev.yml from the root directory.

## High-level Architecture Overview.

### Client: 

nginx hosted React site.

### Nginx:

Path based routing to the React server (eg: /index.html or /main.js) and to the Express server (eg: /api/values/current/).

### Server:

Express server, when given a number from the Client, will store it in Postgres DB (historical data) and into the Redis DB (calculations). 
If a number is not in the Redis DB, the Worker service is called.

### Worker:

Watches the Redis DB for new indicies, pulls each new indicies, calculates the new value then places into Redis.

### Redis:

Stores the "Calculated Values" data in Redis.
In development, this is a local redis container. In production, it leverages AWS Elasticache.

### Postgres:

Stores previously generated results. "Values I have seen" on the frontend
In development, this is a local postgres container. In production, it leverages AWS RDS.
