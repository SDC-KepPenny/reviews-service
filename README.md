# reviews-service

**About**

A containerized REST API service connected to a MongoDB database, scaled with Docker Swarm.

**Technology Stack**

- Node.js
- Express
- MongoDB
- AWS - EC2
- Docker
- Docker Swarm
- Loader.io

**Description**

The Pipe36/reviews REST API service consists of an Express server connected to a MongoDB database, composed together into a single application stack via the Docker Engine. The API exposes endpoints for handling CRUD operations from a React-Redux front-end client. The major motivation behind this project was to gain hands-on experience in system design, specifically one involving the Extract-Transform-Load (ETL) process, deployment, data migration, and optimization via horizontal-scaling.

**Install**

Run ‘docker-compose up’ to initialize the application stack.

The server listens on port 8000 and connects to MongoDB on port 27017

API Routes

- GET /reviews
    - Returns a list of reviews for a particular product. This list does not include any reported reviews.
- GET /reviews/meta
    - Returns review metadata for a given product.
- PUT /reviews/:review_id/helpful
    - Increments a review’s helpfulness score by one point.
- PUT /reviews/:review_id/report
    - Soft-deletes the review by marking it as ‘reported’. The review will not be returned in future GET requests.
- POST /reviews
    - Adds a review for the given product and updates its metadata.

**Extract-Load-Transform (ELT) Process**

The initial data is completely normalized and exists as CSVs.

1. E: This data is fed into an extraction and cleanup pipeline using Node.js Stream API. In this initial step, the data undergoes minor transformation (e.g. standardizing all date values' format) but remains normalized. before being saved as JSON files, ready for database loading. 
2. L: Each JSON document is passed through another pipeline which parses JSON, and formats the data according to its corresponding Mongoose data model schema, before saving it to the database. The Mongoose schema helps to further filter out entries that do not conform to appropriate value types.
3. T: Once loaded, data is further transformed via MongoDB's Aggregation Pipeline into data shapes accepted by the front-end client. 

**Data Migration**

For the purpose of stress testing, the containerized service was first deployed to an AWS EC2 t2.micro instance. A mongodump / mongorestore pipeline was used to handle data migration from the local instance of the service to the EC2 instance. In total, 5.7 million documents of user reviews were successfully migrated in BSON format.  

**Optimization with Docker Swarm**

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f75583d3-cf3d-4519-9ccd-7de969b7e583/Screen_Shot_2021-05-17_at_3.36.14_AM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f75583d3-cf3d-4519-9ccd-7de969b7e583/Screen_Shot_2021-05-17_at_3.36.14_AM.png)

To scale up the service's ability to handle increased traffic, the application's architecture was reworked to take advantage of the Docker Swarm orchestration technology. Firstly, multiple EC2 instances of the server were spun up and networked into a single swarm consisting of a leader node and 3 worker nodes. All 4 nodes are linked via a network mesh that implements a round-robin load-balancing of incoming client traffic. Secondly, the database was separated out from the server logic and all server was reconfigured to send all requests towards the single database instance. This is to ensure data consistency after create operations. This approach to horizontal scaling allowed for drastically improved response times and throughput rate from the API service. After optimization, throughput of the most-encountered GET route increased from 700 RPS to 2000 RPS, and average response time dropped from 2100ms down to 21ms. Error rate held steady at 0.1%.
