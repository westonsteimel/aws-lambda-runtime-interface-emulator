## AWS Lambda Runtime Interface Emulator

![Apache-2.0](https://img.shields.io/npm/l/aws-sam-local.svg)

The Lambda Runtime Interface Emulator is a proxy for Lambda’s Runtime and Extensions APIs, which allows customers to
locally test their Lambda function packaged as a container image. It is a lightweight web-server that converts
HTTP requests to JSON events and maintains functional parity with the Lambda Runtime API in the cloud. It
allows you to locally test your functions using familiar tools such as cURL and the Docker CLI (when testing
functions packaged as container images). It also simplifies running your application on additional computes.
You can include the Lambda Runtime Interface Emulator in your container image to have it accept HTTP
requests instead of the JSON events required for deployment to Lambda. This component does not emulate
Lambda’s orchestrator, or security and authentication configurations. You can get started by downloading and installing it on your local machine. When the Lambda Runtime API emulator is executed, a `/2015-03-31/functions/function/invocations` endpoint will be stood up within the container that you post data to it in order to invoke your function for testing.


## Installing

Instructions for installing AWS Lambda Runtime Interface Emulator for your platform

| Platform | Command to install |
|---------|---------
| macOS | `mkdir -p ~/.aws-lambda-rie && curl -Lo ~/.aws-lambda-rie/aws-lambda-rie https://github.com/aws/aws-lambda-runtime-interface-emulator/releases/latest/download/aws-lambda-rie && chmod +x ~/.aws-lambda-rie/aws-lambda-rie` |
| Linux | `mkdir -p ~/.aws-lambda-rie && curl -Lo ~/.aws-lambda-rie/aws-lambda-rie https://github.com/aws/aws-lambda-runtime-interface-emulator/releases/latest/download/aws-lambda-rie && chmod +x ~/.aws-lambda-rie/aws-lambda-rie` |
| Windows | `Invoke-WebRequest -OutFile 'C:\Program Files\aws lambda\aws-lambda-rie' https://github.com/aws/aws-lambda-runtime-interface-emulator/releases/latest/download/aws-lambda-rie` |


## Getting started

There are a few ways you use the Runtime Interface Emulator (RIE) to locally test your function depending on the base image used. 


### Test an image with RIE included in the image

The AWS base images for Lambda include the runtime interface emulator. You can also follow these steps if you built the RIE into your alternative base image. 

#### To test your Lambda function with the emulator

1. Build your image locally using the docker build command. 

    `docker build -t myfunction:latest .`

2. Run your container image locally using the docker run command. 

    `docker run -p 9000:8080  myfunction:latest`

    This command runs the image as a container and starts up an endpoint locally at `localhost:9000/2015-03-31/functions/function/invocations`. 

3. Post an event to the following endpoint using a curl command: 

    `curl -XPOST "http://localhost:9000/2015-03-31/functions/function/invocations" -d '{}'`

    This command invokes the function running in the container image and returns a response.

### Build RIE into your base image

You can build RIE into a base image. Download the RIE from GitHub to your local machine and update your Dockerfile to install RIE.
 
#### To build the emulator into your image

1. Create a script and save it in your project directory. The following example shows a typical script for a Node.js function. The presence of the AWS_LAMBDA_RUNTIME_API environment variable indicates the presence of the runtime API. If the runtime API is present, the script runs the runtime interface client (https://docs.aws.amazon.com/lambda/latest/dg/runtimes-images.html#runtimes-api-client). Otherwise, the script runs the runtime interface emulator. 
    ```
    #!/bin/sh
    if [ -z "${AWS_LAMBDA_RUNTIME_API}" ]; then
      exec /usr/local/bin/aws-lambda-rie /usr/bin/npx aws-lambda-ric
    else
      exec /usr/bin/npx aws-lambda-ric
    fi
    ```
   
2. Download the runtime interface emulator (https://github.com/aws/aws-lambda-runtime-interface-emulator/releases/latest/download/aws-lambda-rie) from GitHub into your project directory. 

3. Install the emulator package and change ENTRYPOINT to run the new script by adding the following lines to your Dockerfile:
    ```
    ADD aws-lambda-rie /usr/local/bin/aws-lambda-rie 
    ENTRYPOINT [ "/entry_script.sh" ]
    ```

4. Build your image locally using the docker build command. 
    ```
    docker build -t myfunction:latest .
    ```

5. Run your image locally using the docker run command.     
    ```
    docker run -p 9000:8080 myfunction:latest
    ```

### Test an image without adding RIE to the image

You install the runtime interface emulator to your local machine. When you run the image function, you set the entry point to be the emulator. 
*To test an image without adding RIE to the image *

1. From your project directory, run the following command to download the RIE from GitHub and install it on your local machine. 

    ```
   mkdir -p ~/.aws-lambda-rie && curl -Lo ~/.aws-lambda-rie/aws-lambda-rie \
   https://github.com/aws/aws-lambda-runtime-interface-emulator/releases/latest/download/aws-lambda-rie \
   && chmod +x ~/.aws-lambda-rie/aws-lambda-rie
   ```      

2. Run your Lambda image function using the docker run command. 

    `docker run -d -v ~/.aws-lambda-rie:/aws-lambda -p 9000:8080 myfunction:latest 
        --entrypoint /aws-lambda/aws-lambda-rie  <image entrypoint> <(optional) image command>`

    This runs the image as a container and starts up an endpoint locally at `localhost:9000/2015-03-31/functions/function/invocations`. 

3. Post an event to the following endpoint using a curl command: 

    `curl -XPOST "http://localhost:9000/2015-03-31/functions/function/invocations" -d '{}'`

    This command invokes the function running in the container image and returns a response.

## How to configure 

`aws-lambda-rie` can be configured through Environment Variables within the local running Image. 
You can configure your credentials by setting:
* `AWS_ACCESS_KEY_ID`
* `AWS_SECRET_ACCESS_KEY`
* `AWS_SESSION_TOKEN`
* `AWS_REGION`

You can configure timeout by setting AWS_LAMBDA_FUNCTION_TIMEOUT to the number of seconds you want your function to timeout in.

The rest of these Environment Variables can be set to match AWS Lambda's environment but are not required.
* `AWS_LAMBDA_FUNCTION_VERSION`
* `AWS_LAMBDA_FUNCTION_NAME`
* `AWS_LAMBDA_FUNCTION_MEMORY_SIZE`

## Level of support

You can use the emulator to test if your function code is compatible with the Lambda environment, executes successfully 
and provides the expected output. For example, you can mock test events from different event sources. You can also use 
it to test extensions and agents built into the container image against the Lambda Extensions API. This component 
does *not *emulate* *the orchestration behavior of AWS Lambda. For example, Lambda has a network and security 
configurations that will not be emulated by this component. 


* You can use the emulator to test if your function code is compatible with the Lambda environment, runs successfully and provides the expected output.
* You can also use it to test extensions and agents built into the container image against the Lambda Extensions API.
* This component does _not_ emulate Lambda’s orchestration, or security and authentication configurations. 
* The component does _not_ support X-ray and other Lambda integrations locally. 
* The component supports only Linux x86-64 architectures.

## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This project is licensed under the Apache-2.0 License.
