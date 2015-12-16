## Mini Continuous Delivery with Docker for a basic 2 tier Java EE application

Showcase for a continuous delivery based on following principles:
 - short lived environments (environments built and destroyed for each iteration  / run = mvn verify)
 - immutable environments (environemnts built in automated fashion, no manual jobs / touches, hands off :) )
 - infrastructure as code 

### Continuous Delivery life-cycle
 - Build war artifact
 - Upload war to maven repository
 - Start database (mysql/postgresql) container and apply database scripts with flyway
 - Build web application docker image (with latest war artifact) and start container
 - Execute integration test
 - Stop and remove web and database containers
 - INVESTIGATE: Create image from db container

### Techno stack
 1. Java JEE (Web Application)
 2. Wildfly 9.0.2
 3. MySQL 5.7 or PostgreSQL 9.4
 4. Docker 1.9
 5. Docker Compose 1.5
 6. [Jolokia docker maven plugin](https://github.com/rhuss/docker-maven-plugin)
 7. [Flywaydb](http://flywaydb.org/)

### Prerequisites
#### What is required to run this project:
 - Linux with:
  - Java 8
  - Maven 3.3
  - Docker 1.9
  - Docker Compose 1.5

#### Project preparation:
 - This project depends on some docker images that can be built using following script:
  - `#  ./pre_requisites.sh`
 - Follow [instructions](https://github.com/tecris/docker/blob/v3.5/nexus/README.md) to add jboss repository (as proxy repository) to nexus

#### And a Hack :(
 - Re: [Jolokia docker maven plugin](https://github.com/rhuss/docker-maven-plugin), either I am misusing it or some functionality is missing. Until this is sorted out this hack is required to get project deployed:

 ```
 # git clone https://github.com/tecris/docker-maven-plugin
 # cd docker-maven-plugin
 # mvn install
 ```

### How to run / deploy
#### With Wildfly & MySQL
1. Start web(wildfly) and database(mysql) containers
 1. `# docker-compose --x-networking up -d`
1. Deploy database scripts
 1. `# mvn clean compile flyway:migrate -Pdb-mysql`
1. Deploy application
 1. `# mvn clean wildfly:deploy`
1. Run integration tetss
 1. `# mvn clean integration-test -Prun-it`
 
#### With Wildfly & PostgreSQL
1. Start web(wildfly) and database(postgresql) containers
 1. `# docker-compose --x-networking -f wildfly-postgres.yml up -d`
1. Deploy database scripts
 1. `# mvn clean compile flyway:migrate -Pdb-postgres`
1. Deploy application
 1. `# mvn clean wildfly:deploy -Ddatasource.name=PostgresDS`
1. Run integration tetss
 1. `# mvn clean integration-test -Prun-it`

Go to http://localhost:8080/planets

### Continuous delivery
1. With Wildfly and MySQL
 * `# mvn clean verify -Pcd-mysql`
1. With Wildfly and PostgreSQL
 * `# mvn clean verify -Pcd-postgres`
 
 Use `-Dmaven.buildNumber.doCheck=false` if project contains local changes
