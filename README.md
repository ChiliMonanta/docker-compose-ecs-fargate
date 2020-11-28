# Deploy to ECS with Docker Compose

An example how to deploy a docker on AWS ECS with docker compose

What it does: Docker compose creates an ecs cluster that executes on fargate (or ec2).<br>
Docker compose uses cloudformation to deploy, you can inspect the template both before and after a deployment.

Docker compose needs some permissions to run. You can create a role with the
needed permissions with the included cloudformation script. You then have to assume role.

Use awsume to assume role, it's a handy tool to assume roles.

## How to deploy
1. Create role for deployment
    ```
    aws cloudformation deploy --template-file role.yaml --stack-name DockerComposeRole --capabilities CAPABILITY_NAMED_IAM
    ```
2. Assume role

    Assume role, example use awsu.mem see prerequisites how to setup.
    ```
    awsume composerole
    ```

3. Create docker context
    Create a docker context, select existing aws profile and select composerole (created in prerequisites)
    ```
    docker context create ecs myecs
    ```

4. Deploy on ecs
    ```
    docker compose up --context myecs
    ```

5. Test 

    It takes a while fot the ecs task to get status "running", you can monitoring the task in the ECS cluster.

    To get the uri (DNS name) you have to look in the aws load balancer page, EC2 -> Load Balancing<br>
    or from the docker command:
    ```
    docker compose ps --context myecs
    ```

## How to tear down

1. Remove the compose stack
    ```
    docker compose down --context myecs
    ```

2. Remove docker compose role
    ```
    aws cloudformation delete-stack --stack-name DockerComposeRole
    ```

3. Cleanup storage

    You have to cleanup the elastic file system manually.

    - Access points
    - File system

## Prerequisites

- Add assume role on your user.
    ```
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Sid": "VisualEditor0",
                "Effect": "Allow",
                "Action": "sts:AssumeRole",
                "Resource": "*"
            }
        ]
    }    
    ```

- AWS cli (to be able to deploy cloudformation)

    https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html

- Configurate awsu.me, add role arn in your ~/-aws/config

    ```
    [profile composerole]
    source_profile = my-aws
    role_arn = arn:aws:iam::<aws account id>:role/DockerComposeRole
    ``` 

- Docker (you might need the edge release)

    https://github.com/docker/compose-cli

## Limitations

If you tear down and setup a second time wordpress doesn't really work.<br>
It looks like it's keeping the old url in links.

## Links

https://docs.docker.com/engine/context/ecs-integration/

https://github.com/docker/compose-cli

https://awsu.me/

https://www.docker.com/blog/deploying-wordpress-to-the-cloud/

https://www.stevejgordon.co.uk/dotnet-on-aws-introducing-docker-ecs-integration

## TODO

- https ?
- [Autoscale](https://docs.docker.com/engine/context/ecs-integration/#auto-scaling)
- Green/Blue deployment?