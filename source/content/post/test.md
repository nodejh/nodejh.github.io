+++
title = "Test"
description = "Test"
date = 2017-08-18T00:45:05+08:00
tags = [""]
categories = [""]
draft = true
+++

test

```sh
cd /tmp

wget https://github.com/someone/something.tar.gz

tar -zxvf something.tar.gz

cd something

./setup.py install
```


```bash
#!/bin/bash

###### CONFIG
ACCEPTED_HOSTS="/root/.hag_accepted.conf"
BE_VERBOSE=false

if [ "$UID" -ne 0 ]
then
 echo "Superuser rights required"
 exit 2
fi

genApacheConf(){
 echo -e "# Host ${HOME_DIR}$1/$2 :"
}
```
