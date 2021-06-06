#### 搭建命令
1. 搭建mysql
`docker run --name some-mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=my-secret-pw -e MYSQL_DATABASE=test -e MYSQL_USER=test -e MYSQL_PASSWORD=test  -d mysql`

2. 搭建wordpress

`docker run --name some-wordpress -p 8080:80 -d wordpress`

#### 数据库信息输入

这里的host 注意一定要填写自己pc的实际ip地址，而不是`127.0.0.1`


#### 效果
![image](https://user-images.githubusercontent.com/30467613/120913113-31217580-c6c7-11eb-9fd5-d5e1eabb6829.png)
