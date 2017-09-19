---
title:  "Beautiful Spark configuration"
date:   2017-09-08 08:31:12 +0200
---

Configuring Apache Spark jobs can be hard especially if you need to re-run the job with different parameters in order to achieve the best possible performance or even solve some of the performance issues. When you put into the mix productionizing Spark job and automating the build, deploy and scheduling process in multiple environments with multiple different configurations for the same Spark job code, it becomes obvious that most of the parameters need to be provided from the outside.

Apache Spark jobs are highly configurable and everything can be set from the command line using the parameters or through the code. Going with the hardcoded configuration is most of the times efficient enough in the early stages of development. Once the code gets more complex and we start deploying it to Spark cluster whether using manual, scripted or scheduled submit (I highly recommend [rundeck][rundeck-link] for automation) it quickly becomes obvious that externalizing the configuration is required. This is how I have approached the problem, and I'm really happy with the result.

First, I decided to rely on [Typesafe's config][typesafe-config-link] library because I really like HOCON (Human-Optimized Config Object Notation) and parsing the configuration file is super easy. This is how my example configuration file looks like

{% highlight hocon %}
example {

  appName = "example-app-name"

  spark {
    settings {
      "spark.executor.memory" = "8g"
      "spark.sql.shuffle.partitions" = "600"
      "spark.serializer" = "org.apache.spark.serializer.KryoSerializer"
    }
  }

}
{% endhighlight %}

The corresponding `Configuration` object looks like this

{% highlight scala %}
import com.typesafe.config.ConfigFactory

object Configuration {

  private lazy val defaultConfig = ConfigFactory.load("application.conf")

  private val config = ConfigFactory.load().withFallback(defaultConfig)

  // Validate config file
  config.checkValid(ConfigFactory.defaultReference(), "example")

  private lazy val root = config.getConfig("example")

  lazy val appName = root.getString("appName")

  object spark {
    private val spark = root.getConfig("spark")

    import scala.collection.JavaConversions._

    private val settingsObj = spark.getObject("settings")
    lazy val settings = settingsObj.map({ case (k, v) => (k, v.unwrapped().toString()) })
  }

}
{% endhighlight %}

If you look at the `settings` value in the configuration you can clearly see that I'm reading the `settings` object in the configuration as a key-value map of strings. This enables me to easily add any of the Spark configuration properties in the configuration file itself without changing anything in the code or rebuilding the jar. Loading these settings into the Spark configuration is also easy

{% highlight scala %}
trait JobConfig {

  private val config = new SparkConf(true)
    .setAppName(Configuration.appName)
    .setAll(Configuration.spark.settings)

  implicit val spark = SparkSession
    .builder()
    .config(config)
    .getOrCreate()

}
{% endhighlight %}

As you can see from the example, the settings are dynamically loaded and the Spark session is properly configured. Now the only thing left is to actually pass the configuration to the `spark-submit` command so that the library can load it with `ConfigFactory.load()`.

{% highlight bash %}
spark-submit --conf "spark.driver.extraJavaOptions=job.config" --class example.TestJob example-assembly-0.1-SNAPSHOT.jar
{% endhighlight %}

The `extraJavaOptions` parameter is what make the Typesafe's config library automatically load the config file. A good practice is to have a default configuration in case the external configuration is not specified but also to provide a reference configuration file structure. This is usually called `application.conf` and is located in the main resources directory of the project. Using this approach, the `application.conf` file also serves to set defaults with the ability to override them with an external configuration. But, be careful, everything specified in the `application.conf` is applied if not overridden with the external configuration file.

I like this approach because the external configuration file provides an extremely easy way to configure Spark job and enables it to run in different environments or even with different data input sources and output targets.

[rundeck-link]: https://www.rundeck.com/
[typesafe-config-link]: https://github.com/typesafehub/config
