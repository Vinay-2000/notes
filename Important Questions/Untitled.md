Hi, I'm Vinay, and I have over 3 years of experience as a Java Full Stack Developer at Accenture. 
Over the years, I've worked on enterprise applications for one of the largest U.S. retirement service providers, where I've been involved in building backend services, automating business processes, and developing full-stack features.
Currently, I'm part of the Vesting domain, where I design and develop Spring Boot REST APIs, build React user interfaces, and develop Spring Batch jobs that run on AWS Batch to automate daily business processes. 
Along with backend development, I've also worked with Oracle SQL, AWS cloud services for deploying and scheduling applications. Working on AWS also motivated me to strengthen my cloud knowledge, and I earned the AWS Cloud Practitioner certification. I'm someone who enjoys learning new technologies, which is why I've also earned Azure Fundamentals and GitHub Copilot certifications. Now I'm looking for an opportunity where I can work on more challenging backend systems, take on greater ownership, and continue growing as a Java backend engineer.

Currently, I'm working on a retirement services platform for one of the largest U.S. retirement providers. My team works in the **Vesting** domain, which is responsible for managing how retirement funds are distributed based on employer-defined vesting rules. Earlier, many of these requests were handled manually through Service Requests, which was time-consuming and prone to errors. To solve this, we built a self-service Vesting Sweep application that automates the entire process. In this project, I work across both the backend and frontend. I develop Spring Boot REST APIs, build React user interfaces, and develop Spring Batch jobs that process daily vesting transactions. These batch jobs run on AWS Batch and are scheduled using AutoSys. I also work with Oracle SQL for data storage, Redis for improving API performance, write unit and integration tests, and support deployments and production issues. Overall, my role involves developing new features, maintaining existing applications, fixing production issues, and working closely with business analysts, testers, and other developers to deliver new functionality.



> One of the biggest technical challenges was handling large file uploads for our Vesting application. Initially, the frontend uploaded the file through a REST API, and the API was responsible for reading every row and inserting the data into the database. This approach worked for smaller files, but as the file size increased, we started facing timeout and performance issues because the API was doing both the upload and the processing.
> 
> We redesigned the solution by using **S3 pre-signed URLs**. Instead of sending the file through our backend, the frontend uploads it directly to Amazon S3 using the pre-signed URL. Once the upload is complete, an event triggers our batch processing workflow, which reads the file from S3 and processes it asynchronously. This removed the heavy processing from the API, improved response time for users, and allowed us to handle much larger files efficiently. It was a great learning experience because it taught me how to design a more scalable, event-driven solution rather than trying to do everything in a single request.

---

### If they ask, "Did you face any other challenges?"

Then tell the S3 trigger story.

> Another interesting challenge involved handling files that contained sensitive information like Social Security Numbers. When a file was uploaded, an S3 event triggered our batch job. As part of processing, we masked the sensitive data and uploaded the masked file back to S3. However, uploading the processed file triggered the same S3 event again, causing the batch to start repeatedly.
> 
> To solve this, we created separate folders in the S3 bucket. Files uploaded from the UI went into an **unmasked** folder, and EventBridge was configured to trigger the batch only for that specific prefix. The processed files were written to a different folder, so they no longer retriggered the batch. This prevented the processing loop while keeping the solution simple and reliable.


> **Our application is deployed on AWS using ECS. The deployment starts with a CI/CD pipeline that builds a Docker image of the Spring Boot application and pushes it to Amazon ECR (Elastic Container Registry).**
> 
> **Once the image is available in ECR, AWS CloudFormation deploys or updates a stack. The CloudFormation template defines the infrastructure required for the application, including the ECS service and task definition. The ECS task pulls the latest Docker image from ECR and starts the application container.**
> 
> **The same CloudFormation stack also provisions and configures API Gateway, which acts as the entry point for client requests and routes them to the ECS service.**
> 
> **For high availability, the ECS service is integrated with an Application Load Balancer (ALB). The load balancer distributes incoming requests across running ECS tasks, performs health checks, and automatically stops routing traffic to unhealthy tasks. ECS then replaces failed tasks to maintain the desired number of running instances, providing self-healing.**
> 
> **We also configure Auto Scaling policies so that ECS can automatically increase or decrease the number of running tasks based on metrics such as CPU or memory utilization.**
> 
> **Finally, Route 53 provides a user-friendly domain name and routes traffic to the API Gateway, so clients don't need to know the underlying AWS endpoints.**

---

### Simple architecture you can draw in an interview

```
                CI/CD Pipeline
                      │
               Build Docker Image
                      │
                      ▼
                 Amazon ECR
                      │
                      ▼
          CloudFormation Stack
      ┌──────────────────────────┐
      │ ECS Service & Tasks       │
      │ API Gateway              │
      │ Load Balancer            │
      │ Auto Scaling             │
      └──────────────────────────┘
                      │
                      ▼
              ECS pulls image
                      │
                      ▼
        Spring Boot Container(s)
                      ▲
                      │
          Application Load Balancer
                      ▲
                      │
                API Gateway
                      ▲
                      │
             Route 53 (DNS)
                      ▲
                      │
             React / Client App
```