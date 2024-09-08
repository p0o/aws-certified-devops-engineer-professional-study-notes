
> Clarification needed for how auto scaling works with Beanstalk

Version quota: 1000
# .ebextensions

Resources defined within your environment, meaning that if you include your databases here, they are going to be recreated upon blue-green deployments.

### **Files**
You can use the `files` key to create files on the EC2 instance. The content can be either inline in the configuration file, or the content can be pulled from a URL. The files are written to disk in lexicographic order.

### **Sources**
You can use the `sources` key to download an archive file from a public URL and unpack it in a target directory on the EC2 instance.

### **Packages**
You can use the `packages` key to download and install prepackaged applications and components.

### **Commands**
You can use the `commands` key to execute commands on the EC2 instance. The commands run before the application and web server are set up and the application version file is extracted.

### **Container commands**
You can use the `container_commands` key to execute commands that affect your application source code. Container commands run after the application and web server have been set up and the application version archive has been extracted, but before the application version is deployed

You can use `leader_only` to only run the command on a single instance, or configure a `test` to only run the command when a test command evaluates to `true`

Further reading:
https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/customize-containers-ec2.html
