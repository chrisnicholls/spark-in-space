### spark-in-space

requires: a private space with dns-discovery enabled

```
git clone https://github.com/heroku/spark-in-space
cd spark-in-space
heroku create your-spark-app --space your-dns-enabled-space
# have a herokai with sudo add this to your app until run-inside is released
# heroku sudo labs:enable dyno-run-inside -a your-spark-app
heroku buildpacks:add http://github.com/kr/heroku-buildpack-inline.git -a your-spark-app
heroku buildpacks:add https://github.com/heroku/heroku-buildpack-jvm-common.git -a your-spark-app
git push heroku master
heroku scale master=1 worker=2:private-l -a your-spark-app
heroku logs -a your-spark-app -t
```

once you see the master online and the workers registered, you can verify the workers do work by

```
heroku run:inside master.1 bash -a your-spark-app
./bin/spark-shell
sc.parallelize(1 to 1000000).reduce(_ + _)
```

### viewing spark ui

there is an nginx server that can proxy to any dyno in the space.

To set a cookie so that you proxy to the spark master, hit the following url

`http://<your-spark-app>.herokuapp.com/set-backend/1.master.<your-spark-app>.app.localspace:7077`

then go to

`http://<your-spark-app>.herokuapp.com`

You will see proxied version of the spark master UI. You can hit the `/set-backend` path with other in-space hostname:port combos
to be able to see workers and driver program ui.

### HA Spark Masters

High availability spark masters can be accomplished by adding a heroku kafka addon, and utilizing the zookeeper server available in the addon.

If there is a `HEROKU_ZOOKEEPER_URL` set, then the spark processes will be configured to use zookeeper for recovery.

If you set the `SPARK_MASTERS` config var to a number greater than 1, then workers and spark-shell will use spark master urls that point at
the number of masters you specify.

For example if `SPARK_MASTERS` is set to 3 on the app `your-app`, the master url will be

`spark://1.master.your-app.app.localspace:7077,2.master.your-app.app.localspace:7077,3.master.your-app.app.localspace`






### TODO:

* example s3 hdfs config via bucketeer
* factor into a buildpack that can be added to any jvm app, which gets the master,worker,web procs made available.