# A Web Server in Python

* Hosting: AWS EC2
* Webserver: Apache
* Language: Python
* Framework: Flask

## EC2 - Elastic Cloud Computing

### Creating

1. Create an EC2 instance 
1. New security group
1. New keypair

### Connecting

> Locally (on your laptop)

After you download the key (pem file) from AWS, save it to the .ssh directory in your 
home directory.

```sh
  mv got.pem ~/.ssh
```

Change the permissions on that file in order for ssh to work.

```sh
  chmod 400 got.pem
```

Access the remote computer. ssh = Secure shell.

```sh
ssh -i ~/.ssh/got.pem ec2-user@ec2-54-164-88-224.compute-1.amazonaws.com
```

> This video is pretty good:
> https://www.youtube.com/watch?v=M2Wc8JIS-p8
> However:
> I’d recommend keeping all your keys in ~/.ssh/


## Installations

### Environment
```sh
$ sudo yum update
```

### Apache (Web Server)
```sh
$ sudo yum install -y httpd24 
```
### Python/Apache interoperability
```sh
$ sudo yum install mod24_wsgi-python27.x86_64
```

### Flask framework
```sh
$ sudo pip install flask
```

## Operation

Run the web server

```sh
$ sudo service httpd restart
```

## Flask

Reference 
* http://flask.pocoo.org/docs/0.12/quickstart/

Flask in AWS (one way):
* http://www.datasciencebytes.com/bytes/2015/02/24/running-a-flask-app-on-aws-ec2/

```sh
chmod 711 . on home directory
mkdir ~/flaskapp
sudo ln -sT ~/flaskapp /var/www/html/flaskapp

$ cd ~/flaskapp
$ echo "Hello World" > index.html

# we can sanity test at this point
sudo vi /etc/httpd/conf/httpd.conf 
```

```
WSGIDaemonProcess flaskapp threads=5
WSGIScriptAlias / /var/www/html/flaskapp/flaskapp.wsgi

<Directory flaskapp>
    WSGIProcessGroup flaskapp
    WSGIApplicationGroup %{GLOBAL}
    Order deny,allow
    Allow from all
</Directory>
```


$ sudo service httpd restart

## Database / Python
   
### Installation

```sh
$ sudo yum install python27-pip
$ sudo yum install python27-devel
$ sudo yum install mysql27-devel
$ sudo yum install MySQL-python27
```

## Monitoring

```sh
# hits
$ sudo tail -f /var/log/httpd/access_log

# errors
$ sudo tail -f /var/log/httpd/error_log
```

## Other References

1. Helpful but in PHP
    http://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_Tutorials.WebServerDB.CreateWebServer.html
