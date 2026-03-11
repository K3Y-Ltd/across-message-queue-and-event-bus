# ACROSS: Message Queue and Event Bus (MQEB)

This Repository holds code for configuring and setting up a Message Queue and Event Bus (MQEB) that utilizes Apache Kafka.
MQEB can be configured with SSL and is integrated with a REST API for facilitating CRUD operations on topics. MQEB has 
been developed in the context of the **ACROSS EU project** with Grant Agreement **No 101097122** (see [acknowledgements](#acknowledgements)).

![Architecture](/docs/MQEB.jpg)

MQEB is deployed as a container and all the necessary steps to create the associated certificates are performed during the
docker build step.


## Prerequisite
In order to run and deploy Apache Kafka the following must be included:
- docker
- docker-compose


## MQEB Instalation

To deploy and create the associated certificates you must follow the steps:
1) Create a `.env` file following the `.env.local` descriptions.
2) Change the `PASSWORD` in Dockerfile in the same as you specified in the `.env` file.
3) Change the `DOMAIN` in the Dockerfile in the same as you specified in the `.env` file.
4) Run `docker-compose up -d` to build the MQEB container and the associated certificates in the `/ssl` folder
5) Copy the certificates that have been created from inside the container to the hostmachine in order to provide them to potential clients to connect
   1) ***NOTE***: Before you copy the certificates you must change the ca-cert into CARoot that follow PEM format which is compatible with python
      1) Attach into the running container `docker exec -it {kafkassl} bash`
      2) Run the command to convert the ca-cert, `openssl x509 -in ca-cert -out CARoot.pem -outform PEM` inside the docker's container directory `/kafka_2.11-2.2.0/ssl`
      3) Disconnect from the running container
   2) Copy the certificates out with the command: `docker cp <container_id>:/kafka_2.11-2.2.0/ssl /path/on/your/host/vm`
6) Change the permission of the certificates (CARoot.pem, ca-cert, ca-key) that have been extracted from the Docker container via:
   `sudo chmod 777 ca-cert`
   `sudo chmod 777 ca-key`
   `sudo chmod 777 CARoot.pem` 
7) Change the paths for the certificates inside the admin.py


## REST API

In order to run the REST API the following packages must be installed:
- [`fastapi`](https://fastapi.tiangolo.com/)
- [`uvicorn`](https://www.uvicorn.org/)
- [`kafka-python`](https://kafka-python.readthedocs.io/en/master/)


### Execute

Run the Admin REST API in a long running process in the background which will log every operation in a 
uvicorn.log file for tracing:

`nohup uvicorn admin:app --host 0.0.0.0 --port 5001 > uvicorn.log 2>&1 &`

A swagger page can be founder under the `{VM_IP_ADDRESS}:5001/docs` or `{VM_IP_ADDRESS}:5001/redocs`. 
This repo also holds a `swagger.json` file for the reusability of the endpoints.


## Validation

In order to validate the connectivity with the running SSL Apache Kafka Broker in the repository two python
clients are provided under the clients directory:
- A publisher for publishing data
- A subscriber for receiving data

In order to run these clients follow the steps below:
1) Change the file paths for reading the certificates:
```python
CARoot = "{path_to_the_created_certificate}/CARoot.pem"
cert_file = "{path_to_the_created_certificate}/ca-cert"
key_file = "{path_to_the_created_certificate}/ca-key"
Topic = "{topic_name}"
```
2) Change the IP Address for the running broker:
```python
# E.g., bootstrap_servers=["10.10.10.10:9093"],
bootstrap_servers=["{ip_address}:{ssl_port_number}"]
```
3) Change the `ssl_password` with the one you specified in the env:
```python
# E.g., sl_password="your_must_add_password_here",
sl_password="{ssl_password}"
```
1) The clients can be executed as python scripts with the respective commands:
   1) `python subscriber.py`
   2) `python publisher.py`


## Execute as container

The docker image is built via `docker-compose`. The steps below must be followed: 

To build it for the github registry change the following entries in the associated [dockerfile](./Dockerfile):
1) Change the values to be the same as the `.env.local`:
```Dockerfile
ENV PASSWORD="your_must_add_password_here"
ENV DOMAIN="10.10.10.10"
```
2) Build the docker image for linux/AMD64 platform

```bash
docker build . -t ghcr.io/k3y-ltd/apache-kafka-ssl:0.0.1 --platform linux/amd64
```

3) Push image to your github registry:
```shell
docker push ghcr.io/k3y-ltd/apache-kafka-ssl:0.0.1
```

4) Once you have uploaded the docker image to the private registry you can use it in the corresponding `docker-compose.yml` by changing the following lines:
```diff
- build:
-   context: .
-   dockerfile: Dockerfile
```
to the lines:
```diff
# Change the above lines to the bottom
+ build:
+     iamge: ghcr.io/k3y-ltd/apache-kafka-ssl:0.0.1
```
5) Run or build the container via:
```bash
docker compose -f docker-compose.yml up -d
```


## Acknowledgements

🇪🇺 ACROSS project has received funding from the European Union's Horizon Europe research and innovation programme under **Grant Agreement No 101097122**, 
as well as from the Smart Networks and Services Joint Undertaking (SNS JU). 

*Disclaimer: Funded by the European Union. Views and opinions expressed are however those of the author(s) only and do not necessarily*
*reflect those of the European Union or European Commission. Neither the European Union nor the European Commission can be held responsible for them.*

<p align="middle">
  <img src="./docs/logo_across.jpg" height="100">
  <img src="./docs/logo_k3y.png" height="90">
</p>
