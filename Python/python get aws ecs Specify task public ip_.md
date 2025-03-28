---
title: 'python get aws ecs Specify task public ip '
tags: [aws, python, devops]

---

If you only have the container name and need to find the public IP address of the corresponding ECS task, you'll need to follow these steps:

1. List all tasks in the cluster.
2. For each task, describe the task to get details about the containers.
3. Match the container name to the task.
4. Retrieve the public IP address from the corresponding ENI.

Here's how you can do it using Boto3 in Python:

```python
import boto3

def get_public_ip_from_container(cluster_name, container_name):
    ecs_client = boto3.client('ecs')
    ec2_client = boto3.client('ec2')

    # List all tasks in the cluster
    tasks = []
    next_token = None

    while True:
        response = ecs_client.list_tasks(
            cluster=cluster_name,
            nextToken=next_token
        )
        tasks.extend(response['taskArns'])
        next_token = response.get('nextToken')
        if not next_token:
            break

    for task_arn in tasks:
        response = ecs_client.describe_tasks(
            cluster=cluster_name,
            tasks=[task_arn]
        )

        for task in response['tasks']:
            for container in task['containers']:
                if container['name'] == container_name:
                    # Get the ENI attached to the ECS task
                    attachment = task['attachments'][0]
                    eni_id = None
                    for detail in attachment['details']:
                        if detail['name'] == 'networkInterfaceId':
                            eni_id = detail['value']
                            break

                    if eni_id:
                        # Describe the ENI to get the public IP
                        eni_response = ec2_client.describe_network_interfaces(
                            NetworkInterfaceIds=[eni_id]
                        )

                        public_ip = eni_response['NetworkInterfaces'][0].get('Association', {}).get('PublicIp', None)
                        return public_ip

    return None

# Example usage
cluster_name = 'your-cluster-name'
container_name = 'your-container-name'

public_ip = get_public_ip_from_container(cluster_name, container_name)
if public_ip:
    print(f"The public IP address of the container '{container_name}' is: {public_ip}")
else:
    print(f"No public IP address found for the container '{container_name}'.")
```

**Explanation:**

1. **List all tasks in the cluster:** The script lists all task ARNs in the specified ECS cluster.
2. **Describe each task:** It describes each task to get details about the containers.
3. **Match the container name:** For each task, it checks if the specified container name exists.
4. **Retrieve the public IP address:** If the container is found, it retrieves the ENI ID, describes the ENI, and gets the public IP address.

**AWS Credentials:** Ensure your AWS credentials are configured using the AWS CLI (`aws configure`). Replace `your-cluster-name` and `your-container-name` with your actual ECS cluster name and container name.