---
layout: post
title:  "Running Vapor and Consul in Docker"
date:   2017-08-11 10:00:00 +0200
---

I really love using [Vapor](https://vapor.codes) and [Consul](https://www.consul.io). So i created a docker container where you can run both easily.

- [DockerHub](https://hub.docker.com/r/cpageler93/vapor_consul/)
- [GitHub](https://github.com/cpageler93/vapor_consul)

# Dockerfile

{% highlight Docker %}
FROM cpageler93/vapor_consul
 
# add your vapor application
ADD . $APP_PATH
 
# build in release configuration
RUN swift build --configuration release
{% endhighlight %}


# Build

{% highlight shell %}
docker build -t my_vapor_application . 
{% endhighlight %}

# Run

{% highlight shell %}
docker run -p 80:8080 -p 81:8500 -t my_vapor_application
{% endhighlight %}

Ports
- **8080**: Your Vapor Application is available on Port **80**
- **8500**: Consul is available on Port **81**

The default command will run both applications. By overriding the command you can pass parameters to your application.

For example 

{% highlight shell %}
docker run -p 80:8080 -p 81:8500 -t my_vapor_application --help
{% endhighlight %}

will print:
{% highlight shell %}

  NAME:

    Vapor Application

  DESCRIPTION:

    Helper Application to start Vapor+Consul Application

  COMMANDS:
        
    help  Display global or [command] help documentation                
    start Starts the application        

  GLOBAL OPTIONS:
        
    -h, --help 
        Display help documentation
        
    -v, --version 
        Display version information
        
    -t, --trace 
        Display backtrace when an error occurs
{% endhighlight %}