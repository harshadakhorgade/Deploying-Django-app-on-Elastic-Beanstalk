

# Deploying a Django Project on AWS Elastic Beanstalk with RDS

This guide provides a step-by-step approach to setting up and deploying a **Django** project using **AWS Elastic Beanstalk**, **PostgreSQL RDS**, and **S3 for static files**.

---

## **Prerequisites**

Ensure you have the following installed:
- Python (3.7+ recommended)
- Pip and Virtualenv
- AWS CLI
- AWS Elastic Beanstalk CLI (EB CLI)
- Git
- VS Code or any text editor

---

## **Step 1: Install Virtualenv**
```sh
pip install virtualenv
```

## **Step 2: Clone the AWS Elastic Beanstalk CLI Setup Repository**
```sh
git clone https://github.com/aws/aws-elastic-beanstalk-cli-setup
```

Run the setup file inside the **scripts** folder to install EB CLI.

```sh
cd aws-elastic-beanstalk-cli-setup
```

## **Step 3: Create a Virtual Environment and Activate It**
```sh
cd ..
virtualenv eb-virt
eb-virt\Scripts\activate  # On Windows
```

## **Step 4: Install Django and Dependencies**
```sh
pip install django==2.2
pip install setuptools
pip freeze > requirements.txt
```

## **Step 5: Create a New Django Project**
```sh
django-admin startproject ebdjango
cd ebdjango
python manage.py runserver
```

## **Step 6: Create Elastic Beanstalk Configuration Files**
```sh
mkdir .ebextensions
```
Create a new file inside `.ebextensions` named **django.config** and paste the following content:

```yaml
option_settings:
  aws:elasticbeanstalk:container:python:
    WSGIPath: ebdjango.wsgi:application
```

## **Step 7: Initialize Elastic Beanstalk Application**
```sh
eb init -p python-3.7 django-tutorial
```

You need to create an AWS **Access Key** and **Secret Key** before running the command.

## **Step 8: Assign AWS Permissions**
Go to **AWS IAM Console**, create a new user, and attach the following policies:
- `AWSElasticBeanstalkFullAccess`
- `AmazonS3FullAccess`
- (Optional) `IAMFullAccess`

If necessary, create a **custom policy**:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "elasticbeanstalk:*",
                "ec2:*",
                "s3:*",
                "cloudwatch:*",
                "autoscaling:*",
                "elasticloadbalancing:*",
                "rds:*",
                "sns:*",
                "cloudformation:*",
                "iam:PassRole"
            ],
            "Resource": "*"
        }
    ]
}
```

## **Step 9: Configure AWS Credentials Locally**
Edit the AWS credentials file:
```sh
notepad %USERPROFILE%\.aws\credentials
```
Set the region:
```sh
eb init -r ap-south-1
```

## **Step 10: Create Elastic Beanstalk Environment**
```sh
eb create django-env
```

🚀 This process takes some time. If you encounter a **subnet error**, ensure that you have **subnets configured** for your VPC.

Check the environment status:
```sh
eb status
```

## **Step 11: Update Django Settings**
Edit **settings.py**:

```python
ALLOWED_HOSTS = ['your-app-name.elasticbeanstalk.com']

STATIC_ROOT = os.path.join(BASE_DIR, 'static')
```

## **Step 12: Migrate Database & Collect Static Files**
```sh
cd ..
eb-virt\Scripts\activate  # Activate virtual environment
cd ebdjango
python manage.py migrate
python manage.py createsuperuser
python manage.py collectstatic
```

Update `django.config` inside `.ebextensions` to include static file settings:

```yaml
aws:elasticbeanstalk:environment:proxy:staticfiles:
  /static: static
```

## **Step 13: Deploy the Application**
```sh
eb deploy
```

🎉 Now check your application in **AWS Elastic Beanstalk Console**.

## **Step 14: Clean Up (Optional)**
To **terminate the environment**:
```sh
eb terminate django-env
```
To remove the virtual environment:
```sh
rmdir /s /q eb-virt
```
To remove the virtual project:
```sh
rmdir /s /q ebdjango
```

---

## **Troubleshooting**

1. **Access Denied Errors**: Ensure your IAM user has the correct policies attached.
2. **Database Connection Issues**: Make sure your RDS instance is publicly accessible and security groups allow access.
3. **Deployment Failures**: Check logs using:
   ```sh
   eb logs
   ```

---



🚀 **Your Django app is now live on AWS!** 🎉

