# Local Enviornment 
## Configurations
Navigate to your AWS credentials file and update the default profile with access-keys of the newly created IAM user
```
PS C:\Users\aniru\.aws> cd ~\.aws
PS C:\Users\aniru\.aws> ls  
    Directory: C:\Users\aniru\.aws
Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         11/1/2022  10:50 AM             99 config
-a----         1/24/2023   5:19 PM            461 credentials
```

Edit `credentials` file by updating the `[default] ` entry as follows:
```
[default]
aws_access_key_id=###
aws_secret_access_key=###
region=us-east-1
output=json
```




