
### Exploring RabbitMQ with a sample application

RabbitMQ is a queueing service, while acts as a conduit for messages between a source and a destination. It is an effective service, ensuring a deterministic state between the sender and the receiver. Additionally, it also abstracts the source from the destination and the destination from th source as well.

Let us explore a sample application which simulates mails sent in succession to demonstrate the utility of RabbitMQ. The application involves two parallel processes which communicates between each other using a RabbitMQ messaging channel
1. Message payload generator
2. Email sender

The `message payload generator`, generates mail payload according to the inputs provided and pushes them into the RabbitMQ channel queue while the `email sender` process consumes these message payloads and generates an email, sending it to the mock mail server. 

### Setting up the application

Clone the application files from the [git repositorygit repository](https://github.com/operator-playground-io/RabbitMQ-Sample-Project.git)

```execute
git clone https://github.com/operator-playground-io/RabbitMQ-Sample-Project.git projects/sample-app
```

Change directoy tp the project directory usin this command

```execute 
cd projects/sample-app
```

### Running the application

Skaffold handles the workflow for building, pushing and deploying this application. The image for the application shall be built locally. The deployment.yaml and service.yaml manifests, shall be deployed as well.

Skaffold requires the local image registry which must be set, with the command given below

```execute
skaffold config set default-repo localhost:5000
```

Skaffold is now ready for use. Executing the command given below, builds the image and pushes it to the local image registry. Consecutively, the deployment and service are created from the manifest files present in k8s folder.

```execute
skaffold dev
```

The application can now be accessed at http://##DNS.ip##:30007 and the mail server can be accessd at http://##DNS.ip##:30080

### Usage of application

Create the email payload in the application, and specify number of mails and time intervals beween the mails have to be sent. The application will then begin to send mails to the mail server, observing the input provided.

The inputs can be provided in the application as shown

![](_images/app.JPG)

The mails can be viewed in the mail server as shown

![](_images/mail.JPG)
