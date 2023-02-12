# Local Installations

## Install AWS CLI
* Follow the instructions from this page: [Installing or updating the latest version of the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
* Test 
```
    PS C:\Users\aniru\.aws> aws --version
    aws-cli/2.7.4 Python/3.9.11 Windows/10 exe/AMD64 prompt/off
```

## Install eksctl
* Follow the installation instructions from here : [eksctl installation](https://eksctl.io/introduction/#installation)
* Test
```
    PS C:\Users\aniru\.aws> eksctl version  
    0.99.0
```

## Install kubectl
* Follow instructions from here: [kubectl installation](https://kubernetes.io/docs/tasks/tools/)
* Test 
```
    PS C:\Users\aniru\.aws> kubectl version --client
    Client Version: version.Info{Major:"1", Minor:"23", GitVersion:"v1.23.6", GitCommit:"ad3338546da947756e8a88aa6822e9c11e7eac22", GitTreeState:"clean", BuildDate:"2022-04-14T08:49:13Z", GoVersion:"go1.17.9", Compiler:"gc", Platform:"windows/amd64"}
```