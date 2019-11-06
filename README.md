
# Learn to build Cloud Native Java Applications with Quarkus - Instructions

## Pull Docker images
```shell
docker pull fabric8/java-alpine-openjdk8-jre
docker pull registry.access.redhat.com/ubi8/ubi-minimal
docker pull quay.io/quarkus/ubi-quarkus-native-image:19.1.1
docker pull mariadb:10.4.4
docker pull adminer:4.7.3-standalone
```

## Clone repo
```shell
git clone https://github.com/redhat-developer-demos/quarkus-tutorial
```

## Prepare shell environment
```shell
cd quarkus-tutorial/work
export TUTORIAL_HOME=`pwd`

# community
export GRAALVM_HOME='$HOME/tools/graalvm-ce-19.2.1/Contents/Home'
# enterprise
export GRAALVM_HOME='$HOME/tools/graalvm-ee-19.2.1/Contents/Home'

cd $GRAALVM_HOME
# community
gu install native-image
# enterprise
curl -L https://download.oracle.com/otn/utilities_drivers/oracle-labs/native-image-installable-svm-svmee-darwin-amd64-19.2.1.jar
gu -L install native-image-installable-svm-svmee-darwin-amd64-19.2.1.jar
```

## Create a sample project
```shell
export QUARKUS_VERSION=0.27.0

mvn io.quarkus:quarkus-maven-plugin:"$QUARKUS_VERSION":create \
	-DprojectGroupId="com.example" \
	-DprojectArtifactId="fruits-app" \
	-DprojectVersion="1.0-SNAPSHOT" \
	-DclassName="FruitResource" \
	-Dpath="fruit"

cd fruits-app
```

## Build & run
### JVM mode
```shell
./mvnw -DskipTests clean package
java -jar target/fruits-app-1.0-SNAPSHOT-runner.jar
```
### Native mode
```shell
./mvnw -DskipTests clean package -Pnative
./target/fruits-app-1.0-SNAPSHOT-runner
```
### Docker-Native mode
`WARN`: Using the -Dnative-image.docker-build=true is very important as need a linux native binary what will be containerized.
```shell
./mvnw clean package -DskipTests -Pnative -Dquarkus.native.container-build=true
docker build -f src/main/docker/Dockerfile.native -t example/fruits-app:1.0-SNAPSHOT .
docker run -it --rm -p 8080:8080 example/fruits-app:1.0-SNAPSHOT
```
### Development mode
```shell
./mvnw compile quarkus:dev
```

## Check API
```shell
curl localhost:8080/fruit
```

## Run test
```shell
./mvnw clean compile test
```

## Build Docker image
### JVM mode
```shell
./mvnw -DskipTests clean package
docker build -f src/main/docker/Dockerfile.jvm -t example/fruits-app:1.0-SNAPSHOT .
```
### Native mode (not working with minikube!)
`WARN`: Using the -Dquarkus.native.container-build=true is very important as need a linux native binary what will be containerized.
```shell
./mvnw clean package -DskipTests -Pnative -Dquarkus.native.container-build=true
docker build -f src/main/docker/Dockerfile.native -t example/fruits-app:1.0-SNAPSHOT .
```

## Check Docker container
```shell
docker run -it --rm -p 8080:8080 example/fruits-app:1.0-SNAPSHOT
curl $(minikube ip):8080/fruit
```

## Deploy on Kubernetes
### Start minikube
```shell
minikube -p quarkus-tutorial start --memory=12288 --cpus=6 --disk-size=50g
eval $(minikube docker-env)
```
### Add Quarkus Kubernetes
```shell
./mvnw quarkus:add-extension -Dextensions="quarkus-kubernetes"
./mvnw clean package -DskipTests
```
### Inspect generated Kubernetes manifests
```shell
cat target/kubernetes/kubernetes.yml
```
### Build Docker image
```shell
./mvnw clean package -DskipTests
docker build -f src/main/docker/Dockerfile.jvm -t example/fruits-app:1.0-SNAPSHOT .
```
### Deploy on Kubernetes
```shell
kubectl apply -f $TUTORIAL_HOME/target/kubernetes/kubernetes.yml
```
### Expose Kubernetes service
```shell
kubectl patch svc fruits-app --type='json' -p '[{"op":"replace","path":"/spec/type","value":"NodePort"}]'
SVC_URL=$(minikube service fruits-app --url)
```
### Check API from Kubernetes
```
curl $SVC_URL/fruit
```
