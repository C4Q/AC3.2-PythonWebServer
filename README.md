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

Some differences in our environment using AWS's default server. For one thing,
symbolically linking into the ec2-user directory required setting permissions
on that user's directory that prevented subsequent ssh sessions. Therefore, do not
run ```sudo ln -sT ~/flaskapp /var/www/html/flaskapp``` or change the permissions
on the home directory.

```sh
# this is the root directory of the server
$ cd /var/www/html

# create our directory as root
$ sudo mkdir flaskapp

# root can make us the owner again
$ sudo chown ec2-user flaskapp/


$ echo "Hello World" > index.html

# we can sanity test at this point

```
http://ec2-54-236-16-110.compute-1.amazonaws.com/flaskapp/


Edit /etc/httpd/conf.modules.d/10-wsgi.conf (or /etc/httpd/conf/httpd.conf 
if the former doesn't exist):

```
$ sudo nano /etc/httpd/conf.modules.d/10-wsgi.conf
```

Add these lines:

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

```sh
$ sudo service httpd restart
```

## Database / Python
   
### Installation

```sh
$ sudo yum install python27-pip
$ sudo yum install python27-devel
$ sudo yum install mysql27-devel
$ sudo yum install MySQL-python27
```

## Flask Code with Database

```python
from flask import Flask
from collections import Counter
from flask import json
from flask import jsonify
import MySQLdb

db = MySQLdb.connect(host="atlas.cf626xxbuyrf.us-east-1.rds.amazonaws.com",    # your host, usually localhost
                     user="gotuser",         # your username
                     passwd="winteriscoming",  # your password
                     db="got")        # name of the database

app = Flask(__name__)

@app.route('/')
def hello_world():
  return 'Hello from Flask!'

@app.route('/countme/<input_str>')
def count_me(input_str):
  input_counter = Counter(input_str)
  response = []
  for letter, count in input_counter.most_common():
      response.append('"{}": {}'.format(letter, count))
  return '<br>'.join(response)

@app.route('/api/characters')
def characters():
  ret = ''
  # you must create a Cursor object. It will let
  #  you execute all the queries you need
  cur = db.cursor()

  # Use all the SQL you like
  cur.execute("""select c.name, h.name as house
  from houses as h, characters as c, characters_houses as ch
  where ch.house_id = h.id
  and ch.character_id = c.id""")

  # print all the first cell of all the rows
  characters = []
  for row in cur.fetchall():
      name = row[0]
      house = row[1]
      characters.append({"name":name, "house":house})

  db.close()

  return jsonify(characters)
  # return ret

if __name__ == '__main__':
  app.run()

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
