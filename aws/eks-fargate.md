## Optimizing EKS Cluster Cost using AWS Fargate effectively

In today's world, where many organizations are moving towards cloud and containerization, **Kubernetes is becoming the go-to platform for doing Container Orchestration**. In this article we'll look at Managed Kubernetes Service provided by AWS called **EKS (Elastic Kubernetes Service)**. EKS is a fully managed service that makes it easy to deploy, manage, and scale containerized applications using Kubernetes on AWS. AWS will take of the control plane and we need to take care of the worker nodes. We can create worker nodes using self managed or managed node groups. In either case, we need to pay for the EC2 instances that are running in our account. Even if we don't have fully utilized them, we need to pay for EC2 instances. Let's find out how to optimize the cost of running an EKS cluster using AWS Fargate effectively.

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

Let's Consider two different scenarios to understand how Fargate can save the cost effectively.C And also, we'll see when we can't use Fargate so that we can choose the right option based on our requirements.

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

// Steps will be added


2. **For EKS with Worker Nodes:**

// Steps will be added

### Conclusion:

Just as I mentioned earlier, we can't tell one is best over the other. But for sure if we know when to use which one, we can save the cost effectively. We can use Fargate effectively in some scenarios like Batch Processing, Dev/Test Environments, Microservices, Scheduled Jobs, On-Demand Workloads, Stateless Applications. We can't use Fargate effectively in some scenarios like Stateful Applications, High Performance Workloads, Custom Networking, Custom AMIs, Custom IAM Roles, Custom Security Groups. We need to choose the right option based on our requirements to save the cost effectively.

I hope this article helps you to understand how to optimize the cost of running an EKS cluster using AWS Fargate effectively. If you have any questions, feel free to reach out to me on [LinkedIn](https://www.linkedin.com/in/mathesh-me/). I'm happy to help you.