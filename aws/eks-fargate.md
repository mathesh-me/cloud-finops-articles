## Optimizing EKS Cluster Cost using AWS Fargate effectively

In today's world, where many organizations are moving towards cloud and containerization, **Kubernetes is becoming the go-to platform for doing Container Orchestration**. In this article we'll look at Managed Kubernetes Service provided by AWS called **EKS (Elastic Kubernetes Service)**. EKS is a fully managed service that makes it easy to deploy, manage, and scale containerized applications using Kubernetes on AWS. AWS will take of the control plane and we need to take care of the worker nodes. We can create worker nodes using self managed or managed node groups. In either case, we need to pay for the EC2 instances that are running in our account. Even if we don't have fully utilized them, we need to pay for EC2 instances. Let's find out how to optimize the cost of running an EKS cluster using AWS Fargate effectively.

![Faragate vs Worker Nodes](https://github.com/user-attachments/assets/d532ed5f-2261-47cb-9d1c-d9e3cf356fa8)

Before get into the cost saving techniques, I like to explain some basic concepts of EKS and Fargate.
 
### What is EKS?

***Amazon Elastic Kubernetes Service (Amazon EKS) is a managed Kubernetes service by Amazon Web Services to run Kubernetes on AWS.*** Amazon EKS automatically manages the availability and scalability of the Kubernetes control plane nodes responsible for scheduling containers, managing application availability, storing cluster data, and other key tasks. 

**Cost:**
AWS will take care of the control plane and we need to take care of the worker nodes. We can create worker nodes using self managed or managed node groups. There will be a fixed cost for the control plane(As of now, it's $0.10 per hour --> 73 USD per month). For worker nodes, we need to pay for the EC2 instances that are running in our account. If we are using fargate we need to pay for the vCPU and memory that is used by the pods.

So, ***Total cost = Control Plane Cost + Worker Nodes Cost***

You can read more about EKS [here](https://aws.amazon.com/eks/)

### Worker Nodes(Self Managed, Managed Node Groups) 

We need to choose either self managed or managed node groups to run our applications. In self managed node groups, we need to create EC2 instances to run our applications. In managed node groups, AWS will create EC2 instances in our account to run our applications. AWS will take care of the scaling, patching, and monitoring of the EC2 instances. We only need to pay for the EC2 instances that are running in our account. In either case, we need to pay for the EC2 instances that are running in our account. Even if we don't have fully utilized them, we need to pay for EC2 instances.

### What is AWS Fargate?

AWS [Fargate](https://aws.amazon.com/fargate/) is a serverless option for running containers in AWS. With Fargate, we no longer have to provision, configure, or scale the worker nodes in our EKS cluster to run the containers. In simple words, We are going to create one [faragate profile](https://docs.aws.amazon.com/eks/latest/userguide/fargate-profile.html) and associate it with the EKS cluster. Fargate will take care of running the pods in the managed infrastructure. We don't need to worry about the underlying infrastructure. We only pay for the vCPU and memory that is used by the pods. It's just like running our pods with required resources without worrying about the underlying infrastructure. We are not going to waste the resources that are not utilized by the pods. So, In this case we are going to save the cost of not utilized resources in Worker nodes option.

### How it actually saves the cost?

Both worker nodes and Fargate have their own advantages and disadvantages. We can't tell one is best over the other. But for sure if we know when to use which one, we can save the cost effectively. 

**Some Scenarios where we can consider using Fargate:**

1. **Batch Processing**: If you have a batch processing job that runs at a specific time and you don't want to keep the worker nodes running all the time, you can use Fargate. You can run the job at the scheduled time and Fargate will take care of running the job in the managed infrastructure. You only pay for the resources that are used by the job.

2. **Dev/Test Environments**: If you have dev/test environments that are not used all the time, you can use Fargate. You can run the pods only when you need them and Fargate will take care of running the pods in the managed infrastructure. You only pay for the resources that are used by the pods.

3. **Microservices**: If you have microservices that are not used all the time, you can use Fargate. You can run the pods only when you need them and Fargate will take care of running the pods in the managed infrastructure. You only pay for the resources that are used by the pods.

4. **Stateless Applications**: If you have stateless applications that are not used all the time, you can use Fargate. You can run the pods only when you need them and Fargate will take care of running the pods in the managed infrastructure. You only pay for the resources that are used by the pods.

5. **Scheduled Jobs**: If you have scheduled jobs that run at a specific time and you don't want to keep the worker nodes running all the time, you can use Fargate. You can run the job at the scheduled time and Fargate will take care of running the job in the managed infrastructure. You only pay for the resources that are used by the job.

6. **On-Demand Workloads**: If you have on-demand workloads that are not used all the time, you can use Fargate. You can run the pods only when you need them and Fargate will take care of running the pods in the managed infrastructure. You only pay for the resources that are used by the pods.

**Some Scenarios where we can't use Fargate effectively:**

1. **Stateful Applications**: If you have stateful applications that require persistent storage, you can't use Fargate. Fargate doesn't support persistent storage. You need to use worker nodes in this case.

2. **High Performance Workloads**: If you have high performance workloads that require high CPU and memory, you can't use Fargate. Fargate has some limitations on the resources that can be used by the pods. You need to use worker nodes in this case.

3. **Custom Networking**: If you have custom networking requirements, you can't use Fargate. Fargate doesn't support custom networking. You need to use worker nodes in this case.

and so on...

### Some real time examples:

Let's Consider two different scenarios to understand how Fargate can save the cost effectively. We will also see where we can't use Fargate, So that we can choose the right option based on our requirements.<br>

I will show you how to calculate the Cost for both Fargate and Worker Nodes in each scenario using [AWS Pricing Calculator](https://calculator.aws.amazon.com/) at the end of this article. But for now just look at the amount that we need to pay for each option. If you don't understand the math behind the cost calculation, don't worry. You can easily get it using the AWS Pricing Calculator.


**Scenario 1:**

We need to run 20 pods with 2 vCPU and 8GB memory for 1 hour daily. We don't want to keep the worker nodes running all the time. We only want to run the pods for 1 hour daily. We can use Fargate in this case. We only pay for the resources that are used by the pods.<br>

This is how the cost will be calculated:

1. **For Fargate:**

```
Number of tasks or pods: 20 per day * (730 hours in a month / 24 hours in a day) = 608.33 per month

Pricing calculations:
608.33 tasks x 2 vCPU x 1 hours x 0.04048 USD per hour = 49.25 USD for vCPU hours
608.33 tasks x 8.00 GB x 1 hours x 0.004445 USD per GB per hour = 21.63 USD for GB hours
20 GB - 20 GB (no additional charge) = 0.00 GB billable ephemeral storage per task
49.25 USD for vCPU hours + 21.63 USD for GB hours = 70.88 USD total
Fargate cost (monthly): 70.88 USD
```

***Total cost = Control Plane Cost + Fargate Cost***

```
Control Plane Cost: 73 USD
Fargate Cost: 70.88 USD
Total Cost: 143.88 USD
```

2. **For Worker Nodes:**

```
2 instances x 0.096 USD On Demand hourly cost x 730 hours in a month = 140.160000 USD
On-Demand instances (monthly): 140.160000 USD
```

***Total cost = Control Plane Cost + Worker Nodes Cost***

```
Control Plane Cost: 73 USD
Worker Nodes Cost: 140.160000 USD
Total Cost: 213.160000 USD
```

Here, we can see that we can save 69.28 USD by using Fargate effectively. We only need to pay for the resources that are used by the pods. We don't need to pay for the EC2 instances that are running in our account.

**Scenario 2:**

We need to run 20 pods with 2 vCPU and 8GB memory for 10 hours daily. 

This is how the cost will be calculated:

1. **For Fargate:**

```
Unit conversions management events
Number of tasks or pods: 20 per day * (730 hours in a month / 24 hours in a day) = 608.33 per month
Pricing calculations
608.33 tasks x 2 vCPU x 10 hours x 0.04048 USD per hour = 492.50 USD for vCPU hours
608.33 tasks x 8.00 GB x 10 hours x 0.004445 USD per GB per hour = 216.32 USD for GB hours
20 GB - 20 GB (no additional charge) = 0.00 GB billable ephemeral storage per task
492.50 USD for vCPU hours + 216.32 USD for GB hours = 708.82 USD total
Fargate cost (monthly): 708.82 USD
```

***Total cost = Control Plane Cost + Fargate Cost***

```
Control Plane Cost: 73 USD
Fargate Cost: 708.82 USD
Total Cost: 781.82 USD
```

2. **For Worker Nodes:**

```
2 instances x 0.096 USD On Demand hourly cost x 730 hours in a month = 140.160000 USD
On-Demand instances (monthly): 140.160000 USD
```

***Total cost = Control Plane Cost + Worker Nodes Cost***

```
Control Plane Cost: 73 USD
Worker Nodes Cost: 140.160000 USD
Total Cost: 213.160000 USD
```

Here, You can see that Fargate is not cost effective in this case. We need to pay more if we use Fargate. 

### Cost Calculation using AWS Pricing Calculator:

Navigate to [AWS Pricing Calculator](https://calculator.aws.amazon.com/)


1. **For EKS with Fargate:**

![Screenshot 2024-10-20 153859](https://github.com/user-attachments/assets/ba022177-acf4-482d-9b5a-139fa9b431c2)

![Screenshot 2024-10-20 153919](https://github.com/user-attachments/assets/55e22a2e-33df-45c3-831d-10db3ed15495)

![Screenshot 2024-10-20 153942](https://github.com/user-attachments/assets/8452e759-1f8e-4e75-909f-460ae4d35155)

![Screenshot 2024-10-20 153957](https://github.com/user-attachments/assets/d529274a-5b9b-47fd-ae2e-f20770b473b9)

**You can see Per month Cost for EKS Cluster Control Plane at the Left bottom**<br>

**Let's add Fargate to our Estimate now, I am going to configure my Pod resources(CPU, Memory) based on the Scenarios we considered.**

![Screenshot 2024-10-20 154022](https://github.com/user-attachments/assets/1c8287a7-9c41-433d-8439-3c7b2fcf8cf0)

![Screenshot 2024-10-20 154048](https://github.com/user-attachments/assets/c4932cfa-45bd-4507-affd-ae43537c1968)

![Screenshot 2024-10-20 155017](https://github.com/user-attachments/assets/79cedcfd-5c32-4d11-85c6-ff7ef31f0405)

![Screenshot 2024-10-20 155059](https://github.com/user-attachments/assets/df8c383d-7af5-43ee-bb5c-ca764582ff67)

![Screenshot 2024-10-20 160816](https://github.com/user-attachments/assets/65b0c6a5-5149-448f-b49f-1a687c24e785)

2. **For EKS with Worker Nodes:**

**For worker nodes, we will consider the same pod requirements as Fargate: vCPUs: 2, Memory: 8 GB, and Number of Pods: 20. To meet the requirement of 2 vCPUs and 8 GB of memory, we need to choose the `m5.large` instance type. There is some restriction on Number of Pods can be placed on per EKS node. For 20 Pods one `m5.large` instance will not be enough to accommodate 20 pods, we will need two `m5.large` instances in this case.**<br>

Refer How many Pods can be Placed on Each node: [Click here](https://docs.aws.amazon.com/eks/latest/userguide/choosing-instance-type.html#determine-max-pods)<br>

**Remove Fargate from the total estimate so that we can add our worker nodes estimate to the EKS cluster.**

![Screenshot 2024-10-20 155145](https://github.com/user-attachments/assets/ff77c3a0-82b9-4d3d-8c27-3fe7d7f19be2)

![Screenshot 2024-10-20 191717](https://github.com/user-attachments/assets/3ebd26b5-e5c6-4927-8e8a-c77abda51ae9)

![Screenshot 2024-10-20 155205](https://github.com/user-attachments/assets/bb2ba0d7-a917-4da9-b74c-d03b59293f27)

![Screenshot 2024-10-20 155217](https://github.com/user-attachments/assets/5a2c93c5-212e-46be-b093-aa49745866dc)

![Screenshot 2024-10-20 161829](https://github.com/user-attachments/assets/5a9cc40b-6a6c-45a6-bd40-850dbaf13f22)

![Screenshot 2024-10-20 155234](https://github.com/user-attachments/assets/5aacdc8d-4ada-44f8-b723-077a049bb526)

![Screenshot 2024-10-20 155248](https://github.com/user-attachments/assets/62ef698a-c231-4b6f-8505-5b0a6f8dec56)

![Screenshot 2024-10-20 155400](https://github.com/user-attachments/assets/a4e0c718-d9c2-49ed-b30b-866afb7fdd8c)


### Conclusion:

Just as I mentioned earlier, we can't tell one is best over the other. But for sure if we know when to use which one, we can save the cost effectively. We need to choose the right option based on our requirements to save the cost effectively. I recommend Everyone to try out the AWS Pricing Calculator to get the exact cost for your requirements. Get more Hands-on experience with EKS and Fargate to understand the concepts better.

I hope this article helps you to understand how to optimize the cost of running an EKS cluster using AWS Fargate effectively. If you have any questions, feel free to reach out to me on [LinkedIn](https://www.linkedin.com/in/mathesh-me/). I'm happy to help you.
