# Case Study

I use this repo to do my tasks. 

Java: https://github.com/spring-projects/spring-petclinic

```markdown
# Spring PetClinic Sample Application [![Build Status](https://github.com/spring-projects/spring-petclinic/actions/workflows/maven-build.yml/badge.svg)](https://github.com/spring-projects/spring-petclinic/actions/workflows/maven-build.yml)

[![Open in Gitpod](https://gitpod.io/button/open-in-gitpod.svg)](https://gitpod.io/#https://github.com/spring-projects/spring-petclinic) [![Open in GitHub Codespaces](https://github.com/codespaces/badge.svg)](https://github.com/codespaces/new?hide_repo_select=true&ref=main&repo=7517918)

## Understanding the Spring Petclinic application with a few diagrams

[See the presentation here](https://speakerdeck.com/michaelisvy/spring-petclinic-sample-application)

## Run Petclinic locally

Spring Petclinic is a [Spring Boot](https://spring.io/guides/gs/spring-boot) application built using [Maven](https://spring.io/guides/gs/maven/) or [Gradle](https://spring.io/guides/gs/gradle/). You can build a jar file and run it from the command line (it should work just as well with Java 17 or newer):

```bash
git clone https://github.com/spring-projects/spring-petclinic.git
cd spring-petclinic
./mvnw package
java -jar target/*.jar
```

You can then access the Petclinic at <http://localhost:8080/>.

```

first i build the project and create a docker-compose file to test it on my local docker enverioment 

```jsx
mvn clean package -DskipTests -Dcheckstyle.skip=true

```

```
## Database configuration

In its default configuration, Petclinic uses an in-memory database (H2) which
gets populated at startup with data. The h2 console is exposed at `http://localhost:8080/h2-console`,
and it is possible to inspect the content of the database using the `jdbc:h2:mem:<uuid>` URL. The UUID is printed at startup to the console.

A similar setup is provided for MySQL and PostgreSQL if a persistent database configuration is needed. Note that whenever the database type changes, the app needs to run with a different profile: `spring.profiles.active=mysql` for MySQL or `spring.profiles.active=postgres` for PostgreSQL.

```

I changed the application.properties file 

```
# Database Configuration
spring.datasource.url=${SPRING_DATASOURCE_URL:jdbc:postgresql://localhost:5432/petclinic}
spring.datasource.username=${SPRING_DATASOURCE_USERNAME:petclinic}
spring.datasource.password=${SPRING_DATASOURCE_PASSWORD:petclinic}

# JPA for PostgreSQL
spring.jpa.database-platform=${SPRING_JPA_DATABASE_PLATFORM:org.hibernate.dialect.PostgreSQLDialect}

# Database initialization
spring.sql.init.mode=always
spring.sql.init.platform=postgres
spring.sql.init.schema-locations=classpath*:db/postgres/schema.sql
spring.sql.init.data-locations=classpath*:db/postgres/data.sql

```

Dockerfile for app

```
FROM openjdk:17
ARG JAR_FILE=target/*.jar
COPY ${JAR_FILE} app.jar
ENTRYPOINT ["java","-jar","/app.jar"]

```

 

docker-compose.yaml

```
version: "3.8"
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8080:8080"
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/petclinic
      SPRING_DATASOURCE_USERNAME: petclinic
      SPRING_DATASOURCE_PASSWORD: petclinic
      SPRING_JPA_DATABASE_PLATFORM: org.hibernate.dialect.PostgreSQLDialect
      SPRING_PROFILES_ACTIVE: postgres
    depends_on:
      - postgres

  postgres:
    image: postgres:latest
    environment:
      POSTGRES_DB: petclinic
      POSTGRES_USER: petclinic
      POSTGRES_PASSWORD: petclinic
    ports:
      - "5432:5432"
    volumes:
      - postgres-data:/var/lib/postgresql/data

volumes:
  postgres-data:

```
<img width="1662" alt="Screenshot 2024-03-03 at 15 19 46" src="https://github.com/tunahanyilmazz/spring-petclinic/assets/10364043/f887c4cc-8015-48d0-9bac-46582bc3b19e">


**to run app we need to write 
docker-compose up** 

Here I added some Owners to test data is coming from expected DB. 

```jsx
tuna@Tuna-2 spring-petclinic % docker ps
CONTAINER ID   IMAGE                           COMMAND                  CREATED          STATUS          PORTS                    NAMES
db720fe20ada   spring-petclinic-app            "java -jar /app.jar"     31 minutes ago   Up 31 minutes   0.0.0.0:8080->8080/tcp   spring-petclinic-app-1
bdbfbfe22002   postgres:latest                 "docker-entrypoint.s…"   31 minutes ago   Up 31 minutes   0.0.0.0:5432->5432/tcp   spring-petclinic-postgres-1
```

```jsx
tuna@Tuna-2 spring-petclinic % docker exec -it bdbfbfe22002 psql -U petclinic -d petclinic -c "SELECT * FROM owners;"

 id | first_name | last_name |        address        |    city     | telephone  
----+------------+-----------+-----------------------+-------------+------------
  1 | George     | Franklin  | 110 W. Liberty St.    | Madison     | 6085551023
  2 | Betty      | Davis     | 638 Cardinal Ave.     | Sun Prairie | 6085551749
  3 | Eduardo    | Rodriquez | 2693 Commerce St.     | McFarland   | 6085558763
  4 | Harold     | Davis     | 563 Friendly St.      | Windsor     | 6085553198
  5 | Peter      | McTavish  | 2387 S. Fair Way      | Madison     | 6085552765
  6 | Jean       | Coleman   | 105 N. Lake St.       | Monona      | 6085552654
  7 | Jeff       | Black     | 1450 Oak Blvd.        | Monona      | 6085555387
  8 | Maria      | Escobito  | 345 Maple St.         | Madison     | 6085557683
  9 | David      | Schroeder | 2749 Blackhawk Trail  | Madison     | 6085559435
 10 | Carlos     | Estaban   | 2335 Independence La. | Waunakee    | 6085555487
 11 | 11         | 1111      | 11                    | 111         | 111
 12 | 12         | 34        | 56                    | 12          | 123
 13 | 1          | 1         | 1                     | 1           | 1
 14 | 2          | 2         | 2                     | 2           | 2
 15 | 3          | 4         | 5                     | 3           | 3
(15 rows)
 
```

# Minikube Env

We can follow link below to install docker and minikube 
https://minikube.sigs.k8s.io/docs/drivers/docker/

```jsx
minikube start
minikube addons enable metrics-server
minikube dashboard 
kubectl apply -f petclinic-deployment.yaml
kubectl apply -f postgres-deployment.yaml
kubectl apply -f petclinic-service.yaml
```

<img width="1205" alt="Screenshot 2024-03-03 at 15 31 25" src="https://github.com/tunahanyilmazz/spring-petclinic/assets/10364043/5eef0340-877d-4779-9b57-83b4a8d7ec40">


If everything okay we should see minikube on docker app if not try to follow docs above. 

Then i created petclinic-deployment.yaml 

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: petclinic
spec:
  replicas: 1
  selector:
    matchLabels:
      app: petclinic
  template:
    metadata:
      labels:
        app: petclinic
    spec:
      containers:
        - name: petclinic
          image: tunahanyilmaz/petclinic:latest
          imagePullPolicy: Always
          ports:
            - containerPort: 8080
          env:
            - name: POSTGRES_URL
              value: "jdbc:postgresql://postgres/petclinic"
            - name: POSTGRES_USER
              value: "petclinic"
            - name: POSTGRES_PASS
              value: "petclinic"
            - name: JPA_PLATFORM
              value: "org.hibernate.dialect.PostgreSQLDialect"
            - name: DATABASE_PLATFORM
              value: "postgres"
            - name: SPRING_PROFILES_ACTIVE # Add this line
              value: "postgres" # And this line, adjusting if you have a specific profile name

```

postgres-deployment.yaml

```yaml
# postgres-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
        - name: postgres
          image: postgres:latest
          env:
            - name: POSTGRES_DB
              value: petclinic
            - name: POSTGRES_USER
              value: petclinic
            - name: POSTGRES_PASSWORD
              value: petclinic
          ports:
            - containerPort: 5432
---
apiVersion: v1
kind: Service
metadata:
  name: postgres
spec:
  ports:
    - port: 5432
  selector:
    app: postgres

```

we use image from dockerhub so create a Dockerfile and push it to docker hub 

Dockerfile 

```docker
FROM openjdk:17
ARG JAR_FILE=target/*.jar
COPY ${JAR_FILE} app.jar
ENTRYPOINT ["java","-jar","/app.jar"]

```

To check image is pushed to dockerhub 

```jsx
tuna@Tuna-2 spring-petclinic % docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: tunahanyilmaz
Password: 
Login Succeeded
tuna@Tuna-2 spring-petclinic % docker images
REPOSITORY                                                TAG                                                                          IMAGE ID       CREATED             SIZE
spring-petclinic-app                                      latest                                                                       d3c2018e5e4f   59 minutes ago      561MB
        
```

<img width="1290" alt="Screenshot 2024-03-03 at 15 40 17" src="https://github.com/tunahanyilmazz/spring-petclinic/assets/10364043/e75cda87-5c6b-43fa-b79f-ed89c3c4c488">


```
kubectl apply -f petclinic-deployment.yaml
kubectl apply -f postgres-deployment.yaml
kubectl apply -f petclinic-service.yaml
```

![Screenshot 2024-03-03 at 22 02 47](https://github.com/tunahanyilmazz/spring-petclinic/assets/10364043/e4f1e0a4-b3d3-4610-b667-2f11c0040251)


```
tuna@Tuna-2 spring-petclinic % minikube service petclinic-service --url

http://127.0.0.1:62379
❗  Because you are using a Docker driver on darwin, the terminal needs to be open to run it.
```

![Screenshot 2024-03-03 at 22 02 47](https://github.com/tunahanyilmazz/spring-petclinic/assets/10364043/b1eed18f-d4cd-4343-9724-fc547ec93a30)


We can doploy our BE to an instance on AWS. 

CICD yaml 

```
name: dev-deploy-cicd

on:
  push:
    branches: ["dev-deploy"]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Login to Docker Hub
        uses: docker/login-action@v1
        with:
          username: ${{ secrets.DOCKER_HUB_USERNAME }}
          password: ${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v1

      - name: Build and push
        uses: docker/build-push-action@v2
        with:
          context: .
          file: ./Dockerfile
          push: true
          tags: ${{ secrets.DOCKER_HUB_USERNAME }}/dev-deploy:latest

      - name: Download latest image on EC2
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.EC2_PUBLIC_IP }}
          username: ubuntu
          key: ${{ secrets.EC2_PRIVATE_KEY }}
          script: |
            docker pull ${{ secrets.DOCKER_HUB_USERNAME }}/dev-deploy:latest
            docker stop dev-backend-container || true
            docker rm dev-backend-container || true
            docker run -d --name dev-backend-container -p 8080:80 ${{ secrets.DOCKER_HUB_USERNAME }}/dev-deploy:latest

```

we can crete our EC2 with terraform 

We need to export them to don’t store these values on terrafrom file. 

export AWS_ACCESS_KEY_ID="your_access_key"
export AWS_SECRET_ACCESS_KEY="your_secret_key"

```
provider "aws" {
  region = "eu-central-1"
  access_key = "AWS_ACCESS_KEY_ID"
  secret_key = "AWS_SECRET_ACCESS_KEY"
}
resource "aws_instance" "terraform" {
  ami           = "ami-04dfd853d88e818e8"
  instance_type = "t2.micro"
  key_name      = "aws-eu-central-1-key"
  tags = {
    Name = "terraform-ec2-4"
  }

  security_groups = [aws_security_group.my_instance_security_group1.name]
  user_data = <<-EOF
              #!/bin/bash
              sudo apt-get update
              sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common
              curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
              sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
              sudo apt-get update
              sudo apt-get install -y docker-ce
              sudo usermod -aG docker ubuntu
              sudo systemctl start docker
              sudo systemctl enable docker
              EOF

  connection {
    type        = "ssh"
    host        = self.public_ip
    user        = "ubuntu"
    private_key = file("/Users/tuna/Desktop/pem/aws-eu-central-1-key.pem")
  }
}

resource "aws_security_group" "my_instance_security_group1" {
  name        = "my-instance-security-group1"
  description = "Security group for my EC2 instance"

  # Ingress rule to allow PostgreSQL connections from the EC2 instance
  ingress {
    from_port   = 5432
    to_port     = 5432
    protocol    = "tcp"
    security_groups = [aws_security_group.my_instance_security_group1.id]
  }
  
  # Ingress rules for SSH, HTTP, and HTTPS
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  # Egress rule to allow all traffic
  egress {
    from_port       = 0
    to_port         = 0
    protocol        = "-1"
    cidr_blocks     = ["0.0.0.0/0"]
    ipv6_cidr_blocks = ["::/0"]
  }
}

```
