# Prefect 3 with docker compose

This Docker Compose sets up Prefect version 3 with Nginx and very basic authentication for the UI and API. It also includes a prefect worker for deploying workflows to google cloud run, you'd have to add google cloud credentials to use it.
I used it to test the serverless workflow integration of Prefect with Google Cloud Run or AWS ECS.
As the prefect workflows running in google cloud run (or aws ecs...) need to be able to communicate with the prefect server I created this docker compose and put it on a server in the cloud. You could also use something like https://ngrok.com to make your local computer available but as I also wanted to test out how cron jobs work out in prefect this wasn't an option.
The authentication implemented here is basic and not very secure. It is intended for testing purposes only! The Prefect Cloud offers better authentication options.

# Get started
```docker compose up -d```

Open up http://localhost:8080 to see the prefect ui!

Login with:  
User: ```admin```
Password: ```admin123```  

Set your Prefect Config either in ~/.prefect/profiles like this
```
active = "local"
[profiles.local]
PREFECT_API_URL = "http://127.0.0.1:4200/api"
PREFECT_API_KEY = "thytErvIShaBoDItYRupIerSMATIonA"
```
or set the environement variables
```
export PREFECT_API_URL=http://127.0.0.1:4200/api
export PREFECT_API_KEY=thytErvIShaBoDItYRupIerSMATIonA
```
to run the flow in /flows with   
```python weather_flow.py```



### other similar repos (without auth)
https://github.com/PrefectHQ/prefect-background-task-examples/blob/main/chaos-duck/docker-compose.yaml
https://github.com/rpeden/prefect-docker-compose  
https://github.com/flavienbwk/prefect-docker-compose

## How to setup gcp worker
https://docs.prefect.io/integrations/prefect-gcp/gcp-worker-guide

# Diagram  
```mermaid
graph TD
    A[Client] -->|8080, 4200| B(nginx)
    B -->|4200| C(prefect)
    C -->|DB Connection| D(postgres)
    E(prefect-worker) -->|API URL| C
    
    subgraph Docker Compose
        B
        C
        D
        E
    end
    
    C -.->|depends on| D
    E -.->|depends on| C
    
    G[Volumes]
    D -.->|postgres-data| G
    C -.->|prefect-data| G
    
    subgraph Nginx Configuration
        H[Port 4200]
        I[Port 8080]
        J{Authorization Check}
        K[Basic Auth]
    end
    
    B --> H
    B --> I
    H --> J
    J -->|Bearer Token| C
    I --> K
    K --> C
    
    style A fill:#f9f,stroke:#333,stroke-width:2px
    style B fill:#bbf,stroke:#333,stroke-width:2px
    style C fill:#bfb,stroke:#333,stroke-width:2px
    style D fill:#fbb,stroke:#333,stroke-width:2px
    style E fill:#bfb,stroke:#333,stroke-width:2px
    style G fill:#e6e6fa,stroke:#333,stroke-width:2px
    style H fill:#fffacd,stroke:#333,stroke-width:2px
    style I fill:#fffacd,stroke:#333,stroke-width:2px
    style J fill:#ff6347,stroke:#333,stroke-width:2px
    style K fill:#ff6347,stroke:#333,stroke-width:2px```
