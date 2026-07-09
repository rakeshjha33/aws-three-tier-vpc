# AWS Highly Available & Secure Web Architecture

A production-grade AWS infrastructure project featuring a custom Virtual Private Cloud (VPC), isolated private subnets for web servers, an Auto Scaling Group (ASG) for dynamic scaling, and an Application Load Balancer (ALB) to smoothly distribute incoming internet traffic.

_________________________________________________________________________________________________________________

VPC-Three-Tier-Architecture
(https://github.com/rakeshjha33/aws-three-tier-vpc/blob/main/vpc-example-private-subnets.png)

_________________________________________________________________________________________________________________

---

## 🛠️ The Architecture at a Glance

Instead of just launching a single virtual machine connected to the public internet, this project mimics how actual tech companies deploy secure applications. 

* **The Safe Zone (VPC):** A private network isolated from the rest of the AWS ecosystem.
* **The Front Gate (Public Subnet):** Holds the Load Balancer and a "Jump Box" (Bastion Host) to accept public traffic and manage system entry.
* **The Vault (Private Subnet):** Where the actual web servers live. They have no public IP addresses, making them completely hidden from outside attackers.
* **The Bodyguards (Load Balancer & Auto Scaling):** The Load Balancer directs traffic smoothly between two servers. If traffic spikes, the Auto Scaling Group automatically creates copies of the servers to handle the load.

---

## 🚀 Step-by-Step Setup Guide (For Beginners)

Follow these clear, conversational steps to build this entire network from scratch.

### Step 1: Build Your Network Blueprint (VPC)
1. log in to your **AWS Console** and search for **VPC**.
2. Click **Create VPC** and select **VPC and more** (this saves a lot of manual configuration time!).
3. Name your project (e.g., `my-prod-network`).
4. Set **Number of Availability Zones (AZ)** to **2** (This puts your servers in two different physical data centers so if one goes down, the other keeps working).
5. Set **Number of Public Subnets** to **2** and **Private Subnets** to **2**.
6. Set **NAT Gateways** to **1 per AZ** (This acts as a one-way mirror, letting your hidden servers update from the internet without allowing the internet to look inside).
7. Scroll down and hit **Create VPC**. Wait a minute for the green checks to appear.

### Step 2: Create the Web Server Template (Launch Template)
Before launching servers automatically, AWS needs a template showing *what kind* of server to make.
1. Search for **EC2** in the console. On the left menu, click **Launch Templates** under *Instances*.
2. Click **Create launch template** and name it `web-server-template`.
3. Under **Application and OS Images**, pick **Ubuntu** (the free-tier eligible option).
4. For **Instance type**, pick **t2.micro** (keeps it free).
5. Choose your existing **Key pair** (login key) or create a new one.
6. Under **Network settings**, create a new **Security Group**. Select your new VPC and add two rules:
   * **Rule 1:** Allow **SSH (Port 22)** from anywhere (so you can log in).
   * **Rule 2:** Allow **Custom TCP (Port 8000)** from anywhere (this is the port our Python web page will run on).
7. Click **Create launch template**.

### Step 3: Set Up Automatic Server Scaling (Auto Scaling Group)
1. On the left EC2 menu, scroll all the way down to the bottom and click **Auto Scaling Groups**.
2. Click **Create Auto Scaling group**, name it, and select the `web-server-template` you just built. Hit Next.
3. Select your custom VPC. Under **Availability Zones and subnets**, select **ONLY your two Private subnets**. (We want our backend servers hidden!). Hit Next.
4. Skip the Load Balancer step for a brief moment and hit Next.
5. Under **Group Size**, set **Desired capacity**, **Minimum**, and **Maximum** all to **2**. This tells AWS: *"Always keep 2 servers running."*
6. Click Next through the remaining pages and hit **Create Auto Scaling group**. If you check your EC2 instances list, you will see 2 hidden servers starting up automatically!

### Step 4: Build the Secure Entry Point (Bastion/Jump Host)
Because your web servers are in a private vault, you can't log into them directly from your computer. We need a middleman server in the public subnet.
1. Go to **EC2 Instances** and click **Launch Instances**. Name it `Bastion-Host`.
2. Select **Ubuntu** and **t2.micro**. Select the same login Key Pair as before.
3. Under **Network Settings**, click Edit:
   * Select your custom VPC.
   * Pick one of your **Public Subnets**.
   * Toggle **Auto-assign public IP** to **Enable**.
   * Create a new Security Group allowing **SSH (Port 22)** from your IP address or anywhere.
4. Launch the instance.

### Step 5: Log in and Host Your Web Pages
Now, let's play the connection telephone game: Your Computer ➡️ Bastion Host ➡️ Private Web Server.
1. On your personal computer, open your terminal and copy your login key (`.pem` file) over to the Bastion host using the secure copy command:
   ```bash
   scp -i "your-key.pem" your-key.pem ubuntu@<BASTION-PUBLIC-IP>:/home/ubuntu/

  1. Now, SSH into your Bastion Host:
  </> Bash
  ssh -i "your-key.pem" ubuntu@<BASTION-PUBLIC-IP>

  2. Once inside the Bastion host, look up the Private IP address of your first Auto Scaling web server from the AWS console. SSH into it from the Bastion:
  </> Bash 
  ssh -i "your-key.pem" ubuntu@<PRIVATE-IP-OF-SERVER-1>
  
  3. Create a quick, simple web page inside this server:
  </> Bash
vim index.html

4. Spin up a quick network server on port 8000:
   </> Bash 
   python3 -m http.server 8000

 5. Type exit to go back to the Bastion host, then repeat steps 3, 4, and 5 for your second private web server (write a simple HTML page version 2).

Step 6: Create the Traffic Traffic Director (Application Load Balancer)

1. On the left EC2 menu, click Target Groups under Load Balancing.

2. Click Create target group, select Instances, name it web-target-group, and set the port to 8000 (where our python pages are running). Select your custom VPC and hit Next.

3. Select both of your auto-scaled private web servers, click Include as pending below, and hit Create target group.

4. Now, click Load Balancers on the left menu.

5. Click Create load balancer ➡️ Application Load Balancer.

6. Name it, keep it Internet-facing, and select your custom VPC.

7. Under Mappings, select both Availability Zones and match them explicitly to your Public Subnets.

8. Under Listeners and Routing, map HTTP Port 80 to forward to your newly created web-target-group.

9. Create the load balancer.

10. Crucial Security Check: Click on the Load Balancer's security group and make sure it has an Inbound Rule that allows HTTP (Port 80) traffic from anywhere (0.0.0.0/0), otherwise the public internet won't be able to reach it.



🎉 How to Test It
1. Wait a few minutes for the Load Balancer state to turn Active.

2. Copy the long DNS Name (URL link) found in your Load Balancer details page.

3. Paste it into your web browser.

4. Hit refresh a few times! You will notice the load balancer seamlessly balancing your clicks, taking you to Server 1 on one refresh and Server 2 on the next.

5. You've officially successfully run a highly secure, scaled multi-AZ cloud application infrastructure layout!


## 📸 Project Live Demonstration

Here is a look at the live architecture running smoothly on AWS:

### 1. The Active AWS Resources (Sydney Region)
*Shows 3 running instances (2 backend web servers + 1 Bastion Host) managed by 1 Auto Scaling group and 1 Load Balancer.*
![AWS EC2 Resources](https://github.com/rakeshjha33/aws-three-tier-vpc/blob/main/project-images/ec2-dashboard.png)

Load Balancer
(https://github.com/rakeshjha33/aws-three-tier-vpc/blob/main/project-images/health-load-balancer.png)

### 2. Successful Traffic Handling (HTTP 200 Logs)
*A look inside the isolated server logs showing successful `HTTP 200 OK` connection streams flowing from the Application Load Balancer.*
![Python Server Traffic Logs](https://github.com/rakeshjha33/aws-three-tier-vpc/blob/main/project-images/server%20launch.png)

### 3. The Final Static Web Page View
*The final landing page delivered securely across the custom VPC network infrastructure layout.*
![Live Web Page Landing:Version-1](https://github.com/rakeshjha33/aws-three-tier-vpc/blob/main/project-images/home1.png)

![Live Web Page Landing:Version-1] (https://github.com/rakeshjha33/aws-three-tier-vpc/blob/main/project-images/home2.png).

   
