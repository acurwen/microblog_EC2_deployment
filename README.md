# Kura Labs Cohort 5- Deployment Workload 3
# Purpose
The purpose of this workload was to deploy a microblog application using Jenkins for CICD. In this project, we configured our infrastrucutre instead of relying  AWS managed services like Elastic Beanstalk to provision the environment for our application.

# Steps
## 1. Cloned Kura Workload 3 repository 
 Cloned Kura Workload 3 [repository](https://github.com/kura-labs-org/C5-Deployment-Workload-3/tree/main) and named it "microblog_EC2_deployment".

## 2. Created "Jenkins" EC2 Instance:
- AMI (Amazon Machine Image): Ubuntu
- Instance type: t3.medium
- Key pair used was my default one
- Used the default VPC (Virtual Private Cloud)
- Security group rules included HTTP (Port: 80), SSH (Port: 22), HTTPS (Port 443), Gunicorn (Port 5000), Jenkins (Port: 8080)
- Storage was set to 1x8 GiB and gp3 (General Purpose SSD) Root Volume

Instance:
![image](https://github.com/user-attachments/assets/612c8f28-885d-4302-9180-db3cf35214a9)

Security groups:
![image](https://github.com/user-attachments/assets/28b64be2-ccb3-4c91-9737-daf39d9bb2b4)


## 3. Updated Authorized Keys
Next, per the Canvas instructions, I appended the contents of the [public_key.txt](https://canvas.instructure.com/courses/9516210/files/269802266?wrap=1) to my authorized_keys file within my Jenkins instance. To do so, I cd'd into root (`cd /`), cd'd into the .ssh directory (`cd .ssh`) and nano'd into the authorized_keys file.


## 4. Installing Jenkins, Python3.9, Python3.9-venv, Python3-pip and Nginx:

To install Jenkins, Python3.9 and Python3.9-venv, I followed the code block we used in Workload 1, starting with the first line (below). However, I changed the versions of python and the python virtual environment to 3.9 since that's the version the microblog app uses:

`sudo apt update && sudo apt install fontconfig openjdk-17-jre software-properties-common && sudo add-apt-repository ppa:deadsnakes/ppa && sudo apt install python3.9 python3.9-venv`

Next, I proceeded with the rest of the code block to install Jenkins:
```
    $sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
    $echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
    $sudo apt-get update
    $sudo apt-get install jenkins
    $sudo systemctl start jenkins
    $sudo systemctl status jenkins
```
Confirmed Jenkins was up and running:

![image](https://github.com/user-attachments/assets/b6fbba7b-c7cf-46c2-bf1b-c2c2beb8152d)


This step installs Jenkins and all its dependencies to ensure I can access the Jenkins web server GUI and run a build later on. I also installed python3.9 and python3.9-venv alongside their dependencies so I am able to create and activate a virtual enviornment in my "Jenkins" EC2 to host the microblog application. 

Although I manually installed Jenkins, next time I can include these commands in a script that I can run instead for quicker installation. 

To install 'python3-pip' and 'nginx', I ran the following commands:
```
sudo apt install python3-pip
sudo apt-get install nginx
 ```
I found these commands from GFG articles [1](https://www.geeksforgeeks.org/how-to-install-pip-in-linux/) & [2](https://www.geeksforgeeks.org/install-nginx-in-aws-linux/).

Then I used `sudo systemctl status nginx` to chek if nginx was successfully on:

![image](https://github.com/user-attachments/assets/09c894cc-3b5e-49e5-aeba-3856243e6e6d)


## 5. Created Python Virtual Environment in my application directory
Used `git clone` to clone my GitHub repository onto my Jenkins EC2.

![image](https://github.com/user-attachments/assets/283e6034-34e8-4c83-b425-545271211bc1)

Then I cd'd into the microblog_EC2_deployment directory, created the virtual environment and activated it.

```
python3.9 -m venv venv
source venv/bin/activate
```

## 6. Installing Application Dependencies
Now in the virtual environment, I ran `pip install -r requirements.txt`.

This command installs all the python packages in requirements.txt that the microblog application depends on and their specified versions.

![image](https://github.com/user-attachments/assets/a263f666-891c-4cec-a0f7-6eb198b0849e)

Next, I ran `pip install gunicorn pymysql cryptography`.

This command installed the gunicorn server, "pymysql" which allows python programs to connect to MySQL databases and the cryptography python package that handles security measures like data encryption and overall data protection.

![image](https://github.com/user-attachments/assets/756525c5-6aba-4da6-a7cc-7a9ab78dda38)

After each pip command, I got a message that a new version of pip was available. I decided to leave that alone for now.

![image](https://github.com/user-attachments/assets/0448a889-a008-40d6-a916-aa99c8d00bd0)

## 7. Setting the Environmental Variable
Next, I ran `FLASK_APP=microblog.py`. 

With this command, I created a variable called "flask_app" and set it equal to microblog.py because that file represents the application to be deployed. This is an important command because this variable is called every time I run flask commands (like in the next step) and I need to make sure the actual application is what is being called. 

(Online I found another command called “flask run” to run the application so I wondered if I needed to also put that in the deploy stage later on.)


## 8. Updating Database and Translation files
Next, I ran the below commands: 
```
$flask translate compile
$flask db upgrade
```
Using the output of each command and google search, I surmised these commands were to 
1. compile information neeeded to translate languages for the application and 
2. update the SQL database used for the application.

![image](https://github.com/user-attachments/assets/f148100e-2c58-41aa-af39-f8aa70bee275)

## 9. Configuring Nginx
Next, I navigated to /etc/nginx/sites-enabled and (sudo) nano'd into "default" to paste the below under "location". I made sure to indent all three lines in case syntax would be an issue. 

This command makes Nginx redirect incoming traffic to gunicorn running on port 5000. The application's IP is 127.0.0.1.

![image](https://github.com/user-attachments/assets/16a1e52f-818a-4ea3-ba03-4b1372c1daae)
![image](https://github.com/user-attachments/assets/5d369d20-1e9a-4285-a0e3-00600928adf3)

## 10. Starting & Running Microblog Application
Next, I navigated back into my repository directory (within the Jenkins instance), activated my virtual environment and ran 
`gunicorn -b :5000 -w 4 microblog:app`

This command sets gunicorn to listen on port 5000 while serving up the microblog application. This command also starts up 4 addtional worker processes to manage multiple requests at the same time which improves fault tolerance in the case one worker stops working, for example. 

![image](https://github.com/user-attachments/assets/66badac1-564a-4d95-9bfc-2d6736b0b43e)

Then I put my Jenkins server's public IP in the address bar with ":5000" (http://44.223.106.13:5000/) and saw the login page for the microblog application!

![image](https://github.com/user-attachments/assets/2f7b35bb-622c-4f91-b964-6fb4fbd15d5b)

Then I stopped the application with Ctrl+C and moved on to the Jenkins build.

**Were steps 5-10 absolutely necessary for the CICD pipeline? Why or why not?** 

Steps 5-10 were not absolutely necessary for our CICD pipeline because we used them to test out our python virtual environment and if our application would work properly once started within the venv, while in our Jenkins instance. We do all these same "checks" or setup configurations within the Jenkinsfile when we are starting our build, testing out the application code and deploying the app. It was a good check however, to test out the individual components of our Jenkinsfile, mainly the Build stage, before running the entie build.

# Jenkinfile and Pipeline:

## Build Stage:
For the Build stage, I needed to create a virtual environment, install all dependencies, set variables and upgrade other features such as the database(s) and translation files, so I included all the commands from steps 5 through 10.

```
# BUILD Stage:
# create virtual environment
python3.9 -m venv venv

# activate virtual environment
source venv/bin/activate

# install all dependencies
pip install -r requirements.txt
pip install gunicorn pymysql cryptography

# setting environmental variables
FLASK_APP=microblog.py

# setting up databases and compiling translation files
flask translate compile
flask db upgrade
```

## Test Stage:
Next, I created my test_app.py script to run a unit test of the application source code. I decided to test the "explore" page from the routes.py module file.

In my actual script, I utilized fixtures (a new concept to me which I learned about [here](https://flask.palletsprojects.com/en/3.0.x/testing/#fixtures)) which acted like mini blocks of code functions to run within a script for various purposes. In my test_app.py, I used two fixtures - one to create the app and another to create my test client to test my app with. 

![image](https://github.com/user-attachments/assets/44ab986f-4015-4f23-8093-395ab6ef1d90)

Then with my test client I tested the "explore page" function from routes.py. I kept it simple and saw that the response code was 302, a redirection code, when I tested it in my terminal. So I wrote a line that asserted that the page on the endpoint ('/explore') is equal to 302. 
![image](https://github.com/user-attachments/assets/3fec317f-fb45-44a6-b1fa-34d908868fa6)

Routes.py:

![image](https://github.com/user-attachments/assets/e8481c16-3ec6-4566-8555-775b627d4e6d)


To avoid the ModuleNotFoundError, I also created a pytest.ini file in my application directory that clarifies the python path to be root and the test path to be my "tests" folder, so pytest knows where to find my test and also where the files being tested are located.
![image](https://github.com/user-attachments/assets/47f1e143-f79e-4172-bb77-af696ef2b0c6)

This pytest succeeded when I manually tested it in my instance as well as during the test stage of the Jenkinsbuild.  


In my GitHub repository, I updated my Jenkins file. To add my pytest, I clicked "Add file" > "Create new file", and then in the space to put the name of each file, I wrote "tests/unit.test_app.py" and pasted the contents of my test into it. In GH, using the "/" helps create a new directory and subdirectories.

![image](https://github.com/user-attachments/assets/6ed84a96-f14b-4e81-8d09-7b1d8ebc61d3)


## OWASP FS Scan Stage:
The OWASP stage scans the project to detect if there any parts of the app that are categorized as vulnerable or “CVE” (Common Vulnerability and Exposure) and reports them back; this plug in is run after the testing stage.

Before running this stage of the Jenkins build, I installed the "OWASP Dependency-Check" plug-in and installed it as a Dependency-Check tool called "DP-Check".

## Clean Stage:
The Clean stage runs an if conditional that if the gunicorn process ID is not equal to zero (so if it's running) then to put the process ID in a file called pid.txt and kill the catenation of the file or kill the process; then to exit the script. Since this stage "kills" the gunicorn process I wanted to make sure the Deploy stage would start it back up again. 

## Deploy Stage:
For the Deploy stage, I needed to run the commands required to deploy the application so that it is available to the internet.
So at first, I included activating the virtual environment and starting up gunicorn to host the application.
```
stage ('Deploy') {
            steps {
                sh '''#!/bin/bash
                source venv/bin/activate
                gunicorn -b :5000 -w 4 microblog:app
                '''
            }
        }
```

However, I ran into an issue where the Deploy stage would go on forever never stopping.

I realized I had to run gunicorn as a background process so that it wouldn't be at all affected by the Clean stage of killing the process and the Deploy stage of starting it back up.

To do so, I created a systemd service file called microblog.service. A systemd file can start, stop and monitor services and ensure they run in the background.
`sudo nano /etc/systemd/system/microblog.service`

and put the below contents into the file:
```
[Unit]
Description=gunicorn service to deploy microblog
After=network.target

[Service]
User=jenkins
Group=jenkins
WorkingDirectory=/var/lib/jenkins/workspace/workload_3_main/
Environment="PATH=/var/lib/jenkins/workspace/workload_3_main/venv/bin"
ExecStart=/var/lib/jenkins/workspace/workload_3_main/venv/bin/gunicorn -w 4 -b :5000 microblog:app

[Install]
WantedBy=multi-user.target
```
The user running the service is jenkins and the directory set is where our application files are. 
The environment set is our python virtual environment and the required bins or dependencies within it. Then the command used to start the app is specified as well.

Next, I ran the following commands to have the systemd amanager recognize and start the new microblog service created:
- reload the systemd manager to recognize the new service:
`sudo systemctl daemon-reload`
​
- Enable the service to start automatically on boot:
`sudo systemctl enable microblog`
​
- Start the service:
`sudo systemctl start microblog`
​
- Check the status of the service:
`sudo systemctl status microblog`

Then I changed my Jenkinsfile deploy stage to only include a line that restarted the microblog service so that the microblog application can launch regardless of the previous steps in the Jenkins Build:
```
stage ('Deploy') {
            steps {
                sh '''#!/bin/bash
                sudo systemctl restart microblog
                '''
            }
        }
```


Finally the build was a success!!

![image](https://github.com/user-attachments/assets/115d7dfd-3b77-4b88-b0b4-cae0f63a3292)


# Monitoring:
Created "Monitoring" EC2 Instance::
- AMI (Amazon Machine Image): Ubuntu
- Instance type: t3.micro
- Key pair used was my default one
- Used the default VPC (Virtual Private Cloud)
- Security group rules included SSH (Port: 22), Grafana (Port: 3000), Prometheus (Port: 9090) and Node Exporter (Port: 9100) for Prometheus to access node exporter on Jenkins server!
- Storage was set to 1x8 GiB and gp3 (General Purpose SSD) Root Volume

Instance:
![image](https://github.com/user-attachments/assets/3c108f7a-e34a-45fa-bf16-0d2dc161cd0e)

Security groups:
![image](https://github.com/user-attachments/assets/f8bae989-a935-47ff-b267-37b5dc58c0c1)


To install Prometheus, Grafana and Node Exporter, I followed the steps in Mike's [repository](https://github.com/mmajor124/monitorpractice_promgraf/tree/main):

- Created and ran [promgraf.sh](https://github.com/mmajor124/monitorpractice_promgraf/blob/main/promgraf.sh) within my Monitoring instance

![image](https://github.com/user-attachments/assets/6e1b45ee-c2f6-4167-98f2-d6ce31f446b7)

- Created and ran [nodex.sh](https://github.com/mmajor124/monitorpractice_promgraf/blob/main/nodex.sh) within my Jenkins instance

(Node Exporter lives on the Jenkins server because it will export the metrics of whatever else lives on the Jenkins server, a.k.a the microblog application.)


Before running nodex.sh, I commented out this part of nodex.sh since Prometheus and Grafana are instead installed on my Monitoring EC2. 
```
# Add Node Exporter job to Prometheus config
cat << EOF | sudo tee -a /opt/prometheus/prometheus.yml

  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']
EOF

# Restart Prometheus to apply the new configuration
sudo systemctl restart prometheus
```
![image](https://github.com/user-attachments/assets/9bc8039c-cb21-4e8c-9f0e-5f41623e57b0)


Then back in my Monitoring EC2, I ran those code sections:

I ran the below directly in the instance terminal.
```
cat << EOF | sudo tee -a /opt/prometheus/prometheus.yml

  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']
EOF
```
Then I ran `sudo nano /opt/prometheus/prometheus.yml` to doublecheck that the lines were added to my prometheus.yml file.

[Later on I changed 'localhost' to the private IP of my Jenkins instance so that it could remain static. If I were to use the localhost or public IP instead, I'd have to continuously update this and my data source within Grafana everytime my instance changed public IPs.] 

I also added 9100 in the security group for Node Exporter in the inbound rules of my Monitoring  server security group.

Then I ran `sudo systemctl restart prometheus` to update prometheus with my updated endpoints. 

We need all three of these systems because Grafana needs to understand where to find the data to make the graphs. Prometheus is the one grabbing all the data and lastly Node Exporter is the one storing all the metrics and handing Prometheus all the metrics. 

So Grafana speaks to Prometheus within the Monitoring instance which speaks to Node Exporter that's scraping metrics from the Jenkins instance.

**Creating a Dashboard** 

I logged into Grafana on port 3000 with 'admin, admin' credentials.

Created a new dashboard, added a data source (prometeheus and URL http://172.31.18.112:9090 - using the private IP of Jenkins server)

I can use the private IP of my Jenkins instance as an endpoint target because the default route table associated with the default public subnet my instances are in allows local connection between resources within the subnet.

![image](https://github.com/user-attachments/assets/8e28e01a-9ea5-4953-b7a5-b3dbb9862549)

Then I created a dashboard, selected the data source I made above and clicked on metrics.

The few metrics I chose were:
- http_request_duration_seconds: the time it takes for the application to respond to requests (e.g., API or web requests).
- node_cpu_seconds_total: Total CPU usage across all servers or pods.
- node_memory_dirty_bytes : The amount of memory that is marked as "dirty", or memory that has not yet been written back to disk
- up: If the service is up and running.

The screenshot below shows the visualizations for a 30 minute period.

![image](https://github.com/user-attachments/assets/f78b1e25-6181-4cca-ac03-63908f0dbc95)


Other metrics that would be useful for a microblog app once it's actually in use could be:
- api_requests_total: Number of API calls made to thr service.
- login_attempts_total: Tracking  login activity.


# System Design Diagram:
![image](https://github.com/user-attachments/assets/cc81ef9c-d7cc-485b-adf4-2b871a9a2d31)

# Issues and Troubleshooting:
**Syntax**

I tried to put psuedo code comments into my Jenkinsfile using either '//' or '/* */', but I found that my build wouldn't get past the first Build stage. so I removed them in case it was syntax error. 

**Pytest**
I had a lot of trouble creating and testing out my pytests. This is because I modeled my tests after the one example we did in class when testing out Workload 2's application.py module.

I noticed the application files in this workload were not the same format as workload 1's and seemed to be spread out into different files. As explained above, I used python fixtures to create an instance of the microblog application in my test and to create a test client. Then I was able to test a function from routes.py with that test client.

However, every time I imported a module to test, I received the ModuleNotFound error. To fix this, I learned I needed to set the Python path and test path somewhere in my application code so that pytest could find both my test_app.py file alongside understand where the rest of the code (modules) were that I was testing. 
![image](https://github.com/user-attachments/assets/f18d7d8d-90e9-4ada-af90-ba39d5a1d06a)

To achieve this, I created a pytest.ini file that set both the Python path and test path. This helped solve my ModuleNot Found error. 


**OWASP Stage**
My build kept failing at the OWASP stage and I didn't understand why. Finally, I realized I had installed the "OWASP Dependency-Check" plug-in, however, I didn't yet configure and install it as a tool to be used for Jenkinsbuild. After I completed that last step, I was able to get through the OWASP stage. 

**Deploy Stage**

During my systemd service file setup, when I ran `sudo systemctl status microblog` to check the status of the service, I got a failed message that indicated something was already running on port 5000. 

I ran `sudo lsof -i tcp:5000` to see what was running on port 5000 and it was my 5 workers (1 main, 4 helpers) that start up after the `gunicorn -b :5000 -w 4 microblog:app` command (the -w 4). Killing the process ID also didn't work. 
![image](https://github.com/user-attachments/assets/0c43ee54-074b-4cef-9a8d-a72227c7f8c7)


![image](https://github.com/user-attachments/assets/f09f13e3-a5e2-4fb4-b26e-4997b4f57dad)

I realized these workers were still running because I was still had that ongoing Deploy stage going in my Jenkins build:

![image](https://github.com/user-attachments/assets/02235ef8-7d37-4173-84e6-8e8bebcdfa2a)
![image](https://github.com/user-attachments/assets/76e28c24-c927-4384-bbd1-5f0655bdd21c)

Once I finally stopped the Jenkins build, I ran `gunicorn -b :5000 -w 4 microblog:app` to check if the micorblog application could llaunch and it did.
![image](https://github.com/user-attachments/assets/7e529326-c40e-4943-b64c-5352c7be08ba)

Additionally, the `sudo systemctl status microblog` command worked:
![image](https://github.com/user-attachments/assets/ac65afdf-d22f-4c82-b72d-73e2bf32daa5)

After this, however, the deploy stage failed with an error that it needed a password to proceed. To fix this, I ran 
`sudo visudo` to open up file and appended the below to it to grant jenkins sudo permssions to start and restart the microblog:
```
jenkins ALL=(ALL) NOPASSWD: /bin/systemctl restart microblog, /bin/systemctl status microblog, /bin/systemctl is-active microblog
```
(screenshot)

**Grafana**
At first, when I created a dashboad in Grafana, nothing showed up for me metric wise.

![image](https://github.com/user-attachments/assets/890ba260-94e6-4964-a886-f4afd3281886)
Later on, I realized this was due to putting 'local host' as the targets in the prometheus.yml file instead of the prviate IP of my Jenkins instance
![image](https://github.com/user-attachments/assets/b1639574-754a-4693-b7fb-65d6926d436a)

After editing the prometheus.yml file, I navigated to the Prometheus GUI on port 9090. Navigated to Status > Targets and saw my endpoints were configured correctly.

![image](https://github.com/user-attachments/assets/623843b0-0a17-4324-8414-5f4c85d7ac33)

**Configured custom VPC**
I was under the impression we had to create a custom VPC alongside with NACLs, public and private subnets, and route tables before creating our Jenkins instance in step 2. This unfortunately cost me ample time but fortunately gave me good practice on how to do so. When I was working in my custom VPC at first I ran into trouble with my NACL's permissions as SSH was blocked. I also saw a firewall issue in my VPC and went through multiple IAM permissions (and assigning them to my EC2) thinking that was a huge issue in terms of my EC2's ability to connect to the internet.

Other errors: Forgot to correctly name my pipeline "workload_3".

# Optimization:
**What are the advantages of provisioning ones own resources over using a managed service like Elastic Beanstalk?**
Provisioning your own resource allows you more customizing capabilities that best fit whatever you are doing in your infrastrucure (ie. deploying an application). Also, your infrastrucure's contents are not at risk in the case that a managed service like Elastic Beanstalk fails. 

**Could the infrastructure created in this workload be considered that of a "good system"? Why or why not? How would you optimize this infrastructure to address these issues?**
Using a t3.medium at first would have avoided issues related to insufficent memory resources used to carry out the Jenkinsbuild. 

# Conclusion:
This was a good experience provisioning my own infrastructure for the microblgo application. 
