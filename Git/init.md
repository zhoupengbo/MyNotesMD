##### Git global setup

```shell
git config --global user.name "zhoupb"
git config --global user.email "zhoupb@test.com"
```

##### Create a new repository

```shell
git clone https://gitlab.test.com/data/spark-hbase-util.git
cd spark-hbase-util
touch README.md
git add README.md
git commit -m "add README"
git push -u origin master
```

##### Push an existing folder

```shell
cd existing_folder
git init
git remote add origin https://gitlab.test.com/data/spark-hbase-util.git
git add .
git commit -m "Initial commit"
```

##### Push an existing Git repository

```shell
cd existing_repo
git remote rename origin old-origin
git remote add origin https://gitlab.test.com/data/spark-hbase-util.git
```



