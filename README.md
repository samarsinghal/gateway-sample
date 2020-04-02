## Walkthrough: Cloud Foundry Spring cloud gateway Sample

## Introduction


This walkthrough aims to show a basic configuration of the spring cloud gateway using Cloud Foundry with a simple Spring Boot application that prints request headers. In the walkthrough, we are using Cloud Foundry spring cloud gateway service and test some filters and predicates on the request headers.

### Prerequisites

- git installed
- Maven installed and on your path
- Cloud Foundry command line client installed

### Step 1 - Download Example Source Code 

    $ cd [GITHUB HOME]
    $ git clone https://github.com/samarsinghal/gateway-sample.git

### Step 2 - Create Gateway Service Instance

    cf create-service p.gateway standard <GATEWAY-NAME> -c '{ "count": 2 }' 
    $ cf create-service p.gateway standard my-gateway -c '{ "count": 2 }' 

### Step 3 - Deploy Application to CF

  #### Step 3A - New Application deploy without route

    To package and deploy the sample app :

      $ cd [GITHUB HOME]/gateway-sample
      $ mvn clean package
      $ cf push --no-route

  #### Step 3B - Existing application delete route

    If application is already deployed then delete routes for existing apps

      cf delete-route <APPLICATION-HOST>
      $ cf delete-route gateway-sample

### Step 4 - Map Internal route
 
    app.internal is not a public route. its not exposed outside CF  

      cf map-route <APPLICATION-NAME> apps.internal –hostname <APPLICATION-HOST>
      $ cf map-route gateway-sample apps.internal --hostname gateway-sample 

### Step 5 - Bind service

      cf bind-service <APPLICATION-NAME>  <GATEWAY-NAME>  -c '{"routes": [{"path": "/**"}]}'
      $ cf bind-service gateway-sample  my-gateway  -c '{"routes": [{"path": "/**"}]}'

### Step 6 - Test application access via gateway

The sample app is a simple Spring Boot application that exposes a simple ***/header*** endpoint that when called (***GET*** request) will return all available headers. Check that the application is correctly configured. so far no predicates/filters added to gateway. The /headers application route should be accessible via gateway.

---------------------------------------------------------------------

Curl https://GATEWAY-ROUTE/APPLICATION-NAME/PATH" and you should see all headers.
    
    curl --location --request GET 'https://gateway-hostname.cfapps.haas-228.pez.pivotal.io/gateway-sample/headers' --header  'uniqueid: ABC123'      
    
    https://gateway-hostname.cfapps.haas-228.pez.pivotal.io/gateway-sample/headers 
    (assuming your cf deployment system domain is cfapps.haas-228.pez.pivotal.io)  
---------------------------------------------------------------------

### Step 7 - Apply Predicates and filters

  #### 1. Unbind Service

    $ cf unbind-service <APPLICATION-NAME>  <GATEWAY-NAME> 

  #### 2. Bind service with predicates and filters

    $ cf bind-service <APPLICATION-NAME>  <GATEWAY-NAME> -c 'router.json'

   #### Modify your gateway configuration router.json file as follows.  

   ##### To validate client IP
   
    { "routes": [ { "path": "/**", "predicates": ["RemoteAddr=73.153.239.27"] } ] }
        
   ##### To Validate client certificate

    { "routes": [ { "path": "/**", "filters": ["ClientCertificateHeader=*.example.com"] } ] }
    
**Reference Doc**

    https://docs.pivotal.io/spring-cloud-gateway/1-0/route-filters.html
---

### Step 8 - Test application without matching predicates and filters

  Failure : If we call ***/headers*** end point without matching predicates and filters then we will get 404 resource not found.

### Step 9 - Test application access with matching predicates and filters
            
  Success : If request headers matching configured predicates and filters then the gateway will route traffic to application container.

