How to Setup AWS Elastic File System (EFS) on Ubuntu with persistent mount.
===========================================================================



1.  Go to **Amazon Elastic File System** on AWS console

![captionless image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*1o3Cu8nSlm-c7ChLQAZfZA.png)

2. Click **Create file system** and Name it accordingly with **VPC** of your choice, I am choosing default for this demonstration**.**

![captionless image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*UWho5-ZhSNhIHV8__N8Z5w.png)

3. Click on **customize** to modify the EFS settings as per our use case.

*   I am disabling **automated backups** as it comes with additional costs but it is highly recommended in production environments.

![General settings](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*ZIqNlqA16zAAlRr72sgzRg.png)![Lifecycle management](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*N5SnCRikzuuFEf8izuooIQ.png)

*   I have used **bursting mode** for basic performance requirements as we are going to create only basic files.

![Performance settings](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*zkfPx3K8AIFpIfBO16YOGg.png)

**Note**: As I am creating EFS using the minimum cost options for this demonstration you can create according to your use case.

4. Select the following networking setting as the **default VPC** we are using has default **Security group.**

![Networking Access](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*cU5rMgb5_Ys0QYop9j-F4g.png)

5. Leave the **File system policy** as is.

![File system policy](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*kuQT5T0vju47nvOigvswyg.png)

6. Review and Create the File system

![Review and Create](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*-0nHsvBLylsrZQmjGRzVhA.png)

7. Once Created wait until it’s state shows “**Available**”.

![captionless image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*VE2GJVOU6k-WMlHoyU53ig.png)

5. Now Let’s create two **instances**

*   While creating instances, carefully configure the **VPC subnet** to the zone where the EFS is created.
*   As I am using **One-Zone** for creation of File system.

![captionless image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*yFXP8WM7EoU4OujeaRRWxQ.png)

*   **Note**: You can skip above step if you are using **Regional option** for File system.

![captionless image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*yFXP8WM7EoU4OujeaRRWxQ.png)

*   Create two instances with free-tier eligible settings.

![captionless image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*fCNmYQDnvbjqjxc_s--vfA.png)

6. Use **EC2 Instance connect** to connect to the instances.

![captionless image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*LI_OJ7W3vNUCN3AW4GXM3Q.png)

7. Come back to **EFS** file system, select our file system we created and click on **view details**

![Elastic file system](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*LSkPGuXUD-FJzTi7Z6kYfQ.png)

8. Click on **Attach** to view options for mounting EFS to our instances. In this demo we will **mount via DNS**, if you want to mount it with IP address refer to this article: [https://docs.aws.amazon.com/efs/latest/ug/efs-mount-helper.html](https://docs.aws.amazon.com/efs/latest/ug/efs-mount-helper.html)

*   As we are using **ubuntu,** we need to install the **NFS client** to mount EFS. For that run the following command.

```
sudo apt-get update -y && sudo apt-get -y install nfs-common
```

9. Ensure the **security group** rules of instances is **allowed** in the EFS

![Security group for instances.](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*T3bidBSlYFnmVUHdGuRZUw.png)

*   Notice the **security group** of instance ends with **fcc2,** we have to add this to our **EFS network access**.
*   For that click on **Network** tab on the EFS page and click on **manage**.

![Network Settings for EFS](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*jM9O_P44xH7v9c4iDSWUXw.png)

10. Make a directory as “**efs**” and copy the command (2nd one as we are using NFS client) from EFS view details section

```
mkdir efs
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport <DNS name >:/ efs
``````

You can see it is mounted on the first instance EFS-Demo-2

![captionless image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*9-U_J29UZDCvl3e-9e8SgA.png)

11. Do the similar steps from **8 to 10** on EFS-Demo-Instance-1, you’ll see the EFS in mounted on both instances.

![captionless image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*QhmXzoi1u3OE1DIHzGgfQw.png)

12. Test **File system** by creating a file on one instance.

![Permission denied for user.](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*H0xTw89I5msJjZQWz57jIA.png)

*   You’ll face the similar **error** as **permission denied** for as EFS by default only allows **root user** access. You’ll have to change the **efs directory** ownership for **ubuntu** user. For that follow the bellow command

```
sudo chown ubuntu efs 
````
![captionless image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*5EKM8mRCAFHfIQ0bkVEfzg.png)

*   Now file1 should be visible on **EFS-Demo-Instance-2**

![captionless image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*ol7MjgEMKp3bKu8qpSvUHw.png)

*   Voila!! you have a working Elastic file system for both instances, you attach it to much instances as you want, **BUT** the **mount** is **temporary**.
*   What do you mean by temporary mount?
*   Once we **shutdown** the instance or the instances get shut down or **rebooted** by any reason, Whoosh! you efs mount is gone.

13. Lets see this in action

*   Reboot any one instance

![captionless image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*QVDTUUsL4Ly5U09HzpCthA.png)

*   I have **rebooted** the **EFS-Demo-Instance-1**

![Rebooting the instance 1](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*h1VbpqDjLs_lbvz5XIvX2g.png)

*   Now Let’s connect to EC2-Demo-Instance-1 and check the **efs** mount.

![Checking EFS mount](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*R0XgNmzIXO3G5ITw_PcklA.png)

*   As you can see **file1** is not visible even though it is present on File system. (check from EFS-Demo-Instance-2)
*   So, What do we do to make the changes persist even after reboot? Let’s see.

13. Mount the file system once again.

*   First mount the file system again on instance 1 using the above mount command in **step 10.**

![Mounting file system again](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*DQKbWfu0brsdGO_MBmxULg.png)

14. Register entry of file system in **/etc/fstab.**

* Edit the /etc/fstab using vim editor or editor of your choice.

```
sudo vim /etc/fstab
`````
![captionless image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*CNn3kjumbwHuu85okBTfGQ.png)

Append the following to file fstab file.

```
fs-0c5a2099e4dfd9fcd.efs.ap-south-1.amazonaws.com:/ /home/ubuntu/efs nfs4 defaults,_netdev 0 0
`````
This will **persist** the mount even if the instances are shut down or rebooted.

15. Follow this on **both** the instances.

Happy Learning!!!!!😁😁
