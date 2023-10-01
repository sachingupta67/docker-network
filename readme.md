# Docker Image 
- A file where instruction are written how application will run into a container

# Docker Network ( Mongo | NODE API | React APP)	

C1: Mongo we need to run in 1 container

    ```	
    docker container run --name mongoc1 -d mongo:6.0.5 				
    ```
 Here, if mongo image not found locally then it will pull from docker hub

C2 : Now we have to build image for api 

    ```
    Command :  docker image build -t <IMAGE_NAME> <PATH TO LOCAL SOURCE>
    ```
    
    ```
    RUN : docker image build -t node-api ./api-jokes
    ```

    ```
    RUN Container : docker container run --name nodec1 -d  node-api 

    Note : 
    Container Name : nodec1
    Image Name which we want to run in above container : node-api
    ```


# Problem Her in the code of api , we have written code to connect for mongo db , but its not connecting , how we have to resolve this

- We have to make a way to communicate mongodb container to node container
- Approach 1 - Port Mapping  (But here we can use this for local development, we need local env , which is   
              not convinient for production)
     - first we need to access mongodb on locally 

     ```
     docker container run --name mongoc1 -p 8000:27017 -d mongo:6.0.5

     Note
     Container Name : mongoc1
     local port : 8000
     mongo db default port : 27017
     Image : mongo (pull from docker hub)
     tag:6.0.5 (version)

     ```

     - Verify this on localhost:8000 

      output : It looks like you are trying to access MongoDB over HTTP on the native driver port. after getting this update into node-api

    - Note : but now if we have to access local env from container then we need to use dns

     ```
     General : mongodb://localhost:27017/mukeshdb
     DNS : mongodb://host.docker.internal:8000/mukeshdb

     Need to re build node image - docker image build -t node-api ./api-jokes
     Now Container run for Node - docker container run --name nodec1 -d node-api
     ```


     Here , whats going on 

     Node Container ---> LOCAL ENV(DNS)  <--- Mongo (8000:27017)

- Approach 2 : Hit Directly Mongo (if we get IP address then we can do this)

   Arch : Node APP ---IP of MOngo(Mongo)

    - Get Mongo container IP Address
      ```
        docker container inspect mongoc1

        Note - 
        container name : mongoc1
        for IP address : Network Settings > IP Address , which is 172.17.0.2
        Update Mongo COnnect URL : mongodb://172.17.0.2:27017/mukeshdb
        Re Image Build 
        Re Run container
      ```

    - This is also not a convinient way , because suppose container run down , then if you spinup then mongo IP  
      may be change, then again need to update node api code for updating IP

- Approach 3 : Create a network

  Arch [Mern Network] : In this network all container will be

  ```
  Create Network : docker network create <NAME> // NAME == mern-network
  View List of Docker network : docker network ls
  Now SpinUp Container into this network : docker container run --name mongodb --network mern-network -d mongo:6.0.5
  Update nodecode : mongodb://<MONGO_CONT_NAME>:27017/mukeshdb // MONGO_CONT_NAME = mongobd | Automatically detect If IP Change

  Rebuild node build

  ```

- With yaml file -
  - refer .yaml file
  - once done run : docker-compose up -d
  - we can set env variable too using env files 