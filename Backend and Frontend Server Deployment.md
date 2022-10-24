# Backend and Frontend Server Deployment



## Backend Deployment

### Deployment in the Form of Executable File

#### Java

If  you take SpringBoot as your backend implementation framework, you can directly use maven to package your project into an executable jar file or a war file running in the tomcat.

After you complete your backend implementation, you cloud use `mvn package` command to package your project into a executable jar file. If you take IntelliJ IDEA as your IDE, you could directly  complete the package process through the maven GUI interface. 

<img src="C:\Users\AgarthaSF\AppData\Roaming\Typora\typora-user-images\image-20221023163938512.png" alt="image-20221023163938512" style="zoom:67%;" />



If you meet errors like could not pass the maven testing or , please check your pom file that whether your project exists redundant dependencies or some invalid plugin, exclude options. 

After that you could send your packaged jar file to your server. In Windows you could use FileZilla or WinSCP to send your file in a GUI interface. These two software is similar in both function and appearance. FileZilla supports three protocols including SFTP, FTP and Storj, while WinSCP supports SFTP, FTP, SCP, WebDAV and Amazon S3. You can select any one you look to use, or use command line directly lol. 

The more safe and convenient way to transport your file is using the SFTP protocol. Firstly you should open port 22 of your server, and enable the login with password option. Then you can login with user name and password in the FileZilla and use the SFTP protocol to transport your file from host to server. In both FileZilla and WinSCP you could directly drag the file on Windows to your server.

<img src="C:\Users\AgarthaSF\AppData\Roaming\Typora\typora-user-images\image-20221023165310378.png" alt="image-20221023165310378" style="zoom:67%;" />



After that, login into your server and add executable permission to the jar file. Then 

```
java -jar ./xxx.jar
```

If you want to keep this service alive even if you , you could use nohup and & command to keep the service running in the background.

```
nohup java -jar ./xxx.jar &
```

If the port where the backend project  running at has been used by other thread, you could use the netstat command to get the detailed information of the occupied thread. You can get the process ID and then execute the kill command to free the occupied port.

```
netstat -tunlp | grep ${port}
```

![image-20221023170709445](C:\Users\AgarthaSF\AppData\Roaming\Typora\typora-user-images\image-20221023170709445.png)

What should be emphasized is  that you should open the port .If you are using the service provided by cloud platforms, please check your security group policy outside your inner server. Such servers usually have two layers of firewalls, you should open both of them and then you can access  from outside successfully.

#### Golang

If you use GoLand to develop your project, you could directly change your through editing your go build configuration in a GUI interface. Add the destination OS like linux or MacOS and the corresponding architecture like arm64 or x86. What's more, you should also uncheck the `Run after build` option. Then press then build button and you can get the linux executable file.

<img src="C:\Users\AgarthaSF\AppData\Roaming\Typora\typora-user-images\image-20221023171719949.png" alt="image-20221023171719949" style="zoom:67%;" />

You could also use command line to package your project into a linux executable file.

```
set GOARCH=amd64
set GOOS=linux
go build main.go
```

After that, repeat the procedure has been introduced above: transport the executable file to your server, open the corresponding port, and keep your service running in the background.

### Deployment in the Form of Microservices

This part is to be finished. I am expected to use docker and k8s to implement the micro service.

## Frontend Deployment

### Install and Launch Nginx Service

Firstly, install nginx if your server does not has one before.

```
apt-get update
apt-get install nginx
nginx -v
server nginx restart
```

open the port 80 of your server, and check your ip address in browser, if the website showing "welcome to nginx" means that you have successfully launched the nginx service.

### Edit Nginx Configuration

Firstly, go to the nginx directory and edit the configuration file `nginx.conf`

```
cd /etc/nginx
vim nginx.conf
```

Edit your nginx user as a user in your system at the first line, and then add include option at the end of http range

```
include /etc/nginx/hosts/*.host;
```

![image-20221023214910028](C:\Users\AgarthaSF\AppData\Roaming\Typora\typora-user-images\image-20221023214910028.png)

Create the hosts folder in the nginx directory

```
mkdir hosts
cd hosts
```

create the configuration file for you frontend service

```
vim vue.host
```

The vue project is a single-page application, so the web could not get the right router path through nginx service directly. So we need to configure the nginx server to let all the path or folder redirected to the frontpage before real jump, and then they can find the true router. This is the function of the `location @router`. The root path in location is the position where you your packaged frontend project at. The index in the location is the entrance of the project.

file content:

```
server{
        listen 8080;
        server_name vue;
        #access_log logs/host.access.log main;
        location / {
                root /home/lighthouse/dist;
                index index.html;
                try_files $uri $uri/ @router;
        }
        location @router{
                rewrite ^.*$ /index.html last;
        }

        error_page 500 502 503 503 /50x.html;
}
```



### Package your Frontend Project

In Vue CLI project, you could directly run the command 

```
npm run build
```

at your project folder, and then the npm will generate a folder named dist. 

### Deployment

Transfer the folder dist to where we wrote at the nginx configuration file and then change the permission of the folder:

```
chmod 777 dist
```

Restart the nginx service and open the port where frontend project running at

```
nginx -s reload
```

Now you can access your frontend web through corresponding ip and port.



