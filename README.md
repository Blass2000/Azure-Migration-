# Azure-Migration-
General overview of what steps are required for an Azure Migration Project 

Key link https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/?WT.mc_id=DT-MVP-5001664

![image](https://github.com/Blass2000/Azure-Migration-/assets/89789502/ee071c50-b109-4c5d-969c-d2ecf7bd3ffa)

**Intorduction**

By following these steps and leveraging Azure’s comprehensive tools and frameworks, organizations can ensure a successful and secure migration to the cloud.

🔷 Understand the Cloud Adoption Framework
Begin by exploring the Cloud Adoption Framework. This framework provides comprehensive documentation, best practices, and guidance to prepare for cloud migration. It’s crucial for reviewing and optimizing existing Azure environments to enhance management and security.
https://lnkd.in/gbe_7FGa

🔷 Leverage the Well-Architected Framework
Incorporate the Well-Architected Framework, which offers technical guidance to harden, optimize, and secure your Azure environment. Integrated directly into Azure, this framework aids in improving cost posture, security, reliability, operational excellence, and performance.
https://lnkd.in/gcD7gYTi

Wait a sec.... just for the project leaders, so we will be on the same page 

🔷 When talking about the topic of cloud computing services, you must have heard of these concepts, Infrastructure as a Service (IaaS), Platform as a Service (PaaS), and Software as a Service (SaaS). They are the three major service categories provided by cloud providers. It is important to understand what they are and what is the difference between them because they will appear on various occasions where cloud computing services are discussed.

![image](https://github.com/Blass2000/Azure-Migration-/assets/89789502/4702e370-ed44-4cb2-a5af-584342757f70)


# Creating IAM role for Master Node
resource "aws_iam_role" "master" {
  name = "EKS-Master"

  assume_role_policy = jsonencode({
    "Version" : "2012-10-17",
    "Statement" : [
      {
        "Effect" : "Allow",
        "Principal" : {
          "Service" : "eks.amazonaws.com"
        },
        "Action" : "sts:AssumeRole"
      }
    ]
  })
}

# Attaching Policy to IAM role
resource "aws_iam_role_policy_attachment" "AmazonEKSClusterPolicy" {
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKSClusterPolicy"
  role       = aws_iam_role.master.name
}

# Attaching Policy to IAM role
resource "aws_iam_role_policy_attachment" "AmazonEKSServicePolicy" {
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKSServicePolicy"
  role       = aws_iam_role.master.name
}

# Attaching Policy to IAM role
resource "aws_iam_role_policy_attachment" "AmazonEKSVPCResourceController" {
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKSVPCResourceController"
  role       = aws_iam_role.master.name
}

# Creating IAM role for Worker Node
resource "aws_iam_role" "worker" {
  name = "ed-eks-worker"

  assume_role_policy = jsonencode({
    "Version" : "2012-10-17",
    "Statement" : [
      {
        "Effect" : "Allow",
        "Principal" : {
          "Service" : "ec2.amazonaws.com"
        },
        "Action" : "sts:AssumeRole"
      }
    ]
  })
}

# Creating IAM Policy for auto-scaler
resource "aws_iam_policy" "autoscaler" {
  name = "ed-eks-autoscaler-policy"
  policy = jsonencode({
    "Version" : "2012-10-17",
    "Statement" : [
      {
        "Action" : [
          "autoscaling:DescribeAutoScalingGroups",
          "autoscaling:DescribeAutoScalingInstances",
          "autoscaling:DescribeTags",
          "autoscaling:DescribeLaunchConfigurations",
          "autoscaling:SetDesiredCapacity",
          "autoscaling:TerminateInstanceInAutoScalingGroup",
          "ec2:DescribeLaunchTemplateVersions"
        ],
        "Effect" : "Allow",
        "Resource" : "*"
      }
    ]
  })
}

# Attaching Policy to IAM role
resource "aws_iam_role_policy_attachment" "AmazonEKSWorkerNodePolicy" {
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy"
  role       = aws_iam_role.worker.name
}

# Attaching Policy to IAM role
resource "aws_iam_role_policy_attachment" "AmazonEKS_CNI_Policy" {
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy"
  role       = aws_iam_role.worker.name
}

# Attaching Policy to IAM role
resource "aws_iam_role_policy_attachment" "AmazonSSMManagedInstanceCore" {
  policy_arn = "arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore"
  role       = aws_iam_role.worker.name
}

# Attaching Policy to IAM role
resource "aws_iam_role_policy_attachment" "AmazonEC2ContainerRegistryReadOnly" {
  policy_arn = "arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly"
  role       = aws_iam_role.worker.name
}

# Attaching Policy to IAM role
resource "aws_iam_role_policy_attachment" "x-ray" {
  policy_arn = "arn:aws:iam::aws:policy/AWSXRayDaemonWriteAccess"
  role       = aws_iam_role.worker.name
}

# Attaching Policy to IAM role
resource "aws_iam_role_policy_attachment" "s3" {
  policy_arn = "arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess"
  role       = aws_iam_role.worker.name
}

# Attaching Policy to IAM role
resource "aws_iam_role_policy_attachment" "autoscaler" {
  policy_arn = aws_iam_policy.autoscaler.arn
  role       = aws_iam_role.worker.name
}

resource "aws_iam_instance_profile" "worker" {
  depends_on = [aws_iam_role.worker]
  name       = "EKS-worker-nodes-profile"
  role       = aws_iam_role.worker.name
}
The above code will create the IAM role for the master and worker nodes and attach the necessary policy to it.
Create outputs.tf file and add the below code to it.
# IAM Wokrer Node Instance Profile 
output "instance_profile" {
  value = aws_iam_instance_profile.worker.name
}

# IAM Role Master's ARN
output "master_arn" {
  value = aws_iam_role.master.arn
}

# IAM Role Worker's ARN
output "worker_arn" {
  value = aws_iam_role.worker.arn
}
