# Metrics Monitoring Application using Graphite and Grafana

Sample Springboot Application demonstrating metrics collection and monitoring using Graphite and Grafana

License : [![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)



# Prerequsites:
	JAVA Version = 8 or higher
	Compute : CPU 4
	Memory : 8GB or higher
	Docker Enviornment available
    Docker enviornment with cache clear - network , volumes , images , containers.
    
    
# Configurations:

## Springboot Application Configuration
The Springboot application contains both the "Actuator" and "Micrometer" dependencies.Micrometer is a dimensional-first metrics collection facade whose aim is to allow you to time, count, and gauge your code with a vendor neutral API. Through classpath and configuration, you may select one or several monitoring systems to export your metrics data to.

Micrometer is the metrics collection facility included in Spring Boot 2’s Actuator. It has also been backported to Spring Boot 1.5, 1.4, and 1.3 with the addition of another dependency.

Micrometer provides a vendor-neutral metrics collection API (rooted in io.micrometer.core.instrument.MeterRegistry) and implementations for a variety of monitoring systems:

	Netflix Atlas
	CloudWatch
	Datadog
	Ganglia
	Graphite
	InfluxDB
	JMX
	New Relic
	Prometheus
	SignalFx
	StatsD (Etsy, dogstatsd, Telegraf, and proprietary formats)
	Wavefront

### Micrometer and Graphite dependencies


  		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
		</dependency>

		<dependency>
		    <groupId>io.micrometer</groupId>
		    <artifactId>micrometer-core</artifactId>
		</dependency>
		<dependency>
			<groupId>io.micrometer</groupId>
			<artifactId>micrometer-registry-graphite</artifactId>
		</dependency>	


Application is accessible at:
	
	http://localhost:8088/hello


## Graphite:
Graphite is a graphing library responsible for storing and rendering visual representations of data. This means that Graphite requires other applications to collect and transfer the data points.

### The Graphite Web App:

    URL: http://localhost:80
    
    
### Carbon
Carbon is the storage backend for a Graphite configuration. A single Graphite configuration will have one or more Carbon daemons that are responsible for handling data that is sent over by other processes that collect and transmit statistics (the collectors are not part of Graphite).

### StatsD
StatsD is a very simple daemon that can be used to send other data to Graphite. The benefit of this approach is that it becomes trivial to build in stat tracking to applications and systems that you are creating.

### Collectd
Collectd can gather statistics about many different components of a server environment. It allows you to easily track common metrics like memory usage, CPU load, network traffic etc. This allows you to easily correlate events with the state of your systems.




Graphite is being provisned with Docker-Compose using the latest Graphite-Statsd image.


    graphite:
        image: graphiteapp/docker-graphite-statsd
        container_name: graphite
        ports:
          - 80:80
          - 2003:2003
          - 2004:2004
          - 2023:2023
          - 2024:2024
          - 8125:8125/udp
          - 8126:8126
        restart: unless-stopped    
        networks:
          - metrics-monitor
          
          
### Mapped Ports	

    Host	Container	Service
    80	80	nginx => used for springboot app to send data to graphite and grafana to query data
    2003	2003	carbon receiver - plaintext
    2004	2004	carbon receiver - pickle
    2023	2023	carbon aggregator - plaintext
    2024	2024	carbon aggregator - pickle
    8080	8080	Graphite internal gunicorn port (without Nginx proxying).
    8125	8125	statsd
    8126	8126	statsd admin
	
 Graphite will be running at following location on browser:
 
 	http://localhost:80
    
    
   ![alt text](https://github.com/dipsscor/MetricsMonitoring-Graphite-Grafana/blob/master/screenshots/graphite.png) 
    
    

  
  
  
## Grafana Configuration
Grafana is being provisned with Docker-Compose using the latest Grafana image.

	  grafana:
	    image: grafana/grafana
	    container_name: grafana
	    volumes:
	      - ./env/grafana_conf/provisioning:/etc/grafana/provisioning
	      - ./env/grafana_conf/custom_dashboards:/var/lib/grafana/dashboards
	    ports:
	      - 3000:3000
	    networks:
	      - metrics-monitor  
	    env_file:
	      - ./env/grafana_conf/grafana.env
          
          
 Grafana configurations are overwritten through grafana.env file located at local in ./env/grafana_conf/grafana.env
 
 In Grafana dashbaords and Datasources are pre-provisned through custom datasources and dashboards kept in env folder. These get copied to the appropriate location in Grafana's default location in Docker container.
 
 In Grafana - datasource.yml under ./env/provisioning/datasources the prometheus container URL is injected as datasource:
 
 	http://graphite:80
	
	
 In Grafana - dashboard.yml under ./env/provisioning/dashboards following config is used:
 
	folder: 'Spring-boot-apps'
	  type: 'file'
	  options:
	    folder: '/var/lib/grafana/dashboards'
	      
 #### Folder : Holds the dashboards . By default it is 'General'
 #### Options/Folder : holds the location where custom dashboards are injected.
 
 A custom dashboard is being used and modified from grafana market place for Springboot apps.
 
 
   ![alt text](https://github.com/dipsscor/MetricsMonitoring-Graphite-Grafana/blob/master/screenshots/grafana_1.png)
   
   ![alt text](https://github.com/dipsscor/MetricsMonitoring-Graphite-Grafana/blob/master/screenshots/grafana_2.png)
 
 
 # References
 
    https://hub.docker.com/r/graphiteapp/docker-graphite-statsd/
    https://www.digitalocean.com/community/tutorials/an-introduction-to-tracking-statistics-with-graphite-statsd-and-collectd
    https://docs.spring.io/spring-boot/docs/2.1.8.RELEASE/reference/html/production-ready-metrics.html
    https://medium.com/@eranda/monitoring-springboot-with-graphite-and-grafana-part-i-71c8dc90b0ab
    
