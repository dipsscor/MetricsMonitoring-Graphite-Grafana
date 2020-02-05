# Metrics Monitoring Application using Prometheus and Grafana

Sample Springboot Application demonstrating metrics collection and monitoring using Prometheus and Grafana

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

### Micrometer and Prometheus dependencies
The metrics are exposed to http://localhost:8088/actuator/prometheus 

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
			<artifactId>micrometer-registry-prometheus</artifactId>
		</dependency>	


Application is accessible at:
	
	http://localhost:8088/hello


## Prometheus Configuration
Prometheus is being provisned with Docker-Compose using the latest Prometheus image.


      prometheus:
        image: prom/prometheus
        container_name: prometheus
        volumes:
          - ./env/prometheus_conf/prometheus.yml:/etc/prometheus/prometheus.yml:ro
          - ./env/prometheus_conf/alert.rules:/etc/prometheus/alert.rules
        ports:
          - 9090:9090
        restart: unless-stopped    
        networks:
          - metrics-monitor
        depends_on:
          - cadvisor
          - node-exporter
	      
Prometheus confugurations are overwritten with prometheus.yml file located at : ./env/prometheus_conf/prometheus.yml. This goes into the docker container location - /etc/prometheus/prometheus.yml


In "prometheus.yml" file:
	
    -global scrape settings - consists of global scrapes
    
    - scrape configs conists of:
        - Prometheus config
        - node-exporter config
        - cadvisor config
        - alerts configs
	
 Prometheus will be running at following location on browser:
 
 	http://localhost:9090 
    
    
   ![alt text](https://github.com/dipsscor/MetricsMonitoring-Prometheus-Grafana/blob/master/screenshots/prom_targets.png) 
    
    
## Node exporter config
The Node Exporter exposes the prometheus metrics of the host machine in which it is running and shows the machine’s file system, networking devices, processor, memory usages and others features as well. Node exporter can be run as a docker container while reporting stats for the host system. We will append configuration setting to the existing docker-compose.yml and prometheus.yml to bring up life to node-exporter.

In docker-compose.yml:


    prometheus:
            ...............
            ...............
        depends_on:
          - ..........
          - node-exporter
          
     node-exporter:
        image: prom/node-exporter
        container_name: node-exporter
        ports:
        - '9100:9100'
        restart: unless-stopped  
        networks:
          - metrics-monitor  

 In prometheus.yml :

       - job_name: 'node-exporter'
         static_configs:
          - targets: ['node-exporter:9100']
          
          

          
          
## cadvisor config
cAdvisor (short for container Advisor) analyzes and exposes resource usage and performance data from running containers. cAdvisor exposes Prometheus metrics out of the box. 


In docker-compose.yml:


    prometheus:
            ...............
            ...............
        depends_on:
          - cadvisor
          - node-exporter
          
      cadvisor:
        image: google/cadvisor:latest
        container_name: cadvisor
        ports:
          - 8080:8080
        restart: unless-stopped  
        networks:
          - metrics-monitor
        volumes:
          - /:/rootfs:ro
          - /var/run:/var/run:rw
          - /sys:/sys:ro
          - /var/lib/docker/:/var/lib/docker:ro
        depends_on:
          - app


In prometheus.yml:

    ..............
    ..............
      - job_name: cadvisor
        scrape_interval: 5s
        static_configs:
          - targets:
            - cadvisor:8080
            
            
  cadvisor is available on :
  
        http://localhost:8080/
        
        
   ![alt text](https://github.com/dipsscor/MetricsMonitoring-Prometheus-Grafana/blob/master/screenshots/cadvisor.png)      
        
            
## Alert configuration
To setup notification, we need to configure three files-
-    Alert.rules to define rules on which alert will be fired
-    Map this file with the container in docker-compose.yml
-    Edit Prometheus.yml to add alertmanager as a service.

In ./env/prometheus_conf/alert.rules

    groups:
    - name: custom-alerts
      rules:

      # Alert for any instance that is unreachable for >2 minutes.
      - alert: service_down
        expr: up == 0
        for: 2m
        labels:
          severity: page
        annotations:
          summary: "Instance {{ $labels.instance }} down"
          description: "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 2 minutes."

      - alert: high_load
        expr: node_load1 > 0.5
        for: 2m
        labels:
          severity: page
        annotations:
          summary: "Instance {{ $labels.instance }} under high load"
          description: "{{ $labels.instance }} of job {{ $labels.job }} is under high load."
          
          
In prometheus.yml:          
          
    rule_files:
      - 'alert.rules'
  
  
  
  ![alt text](https://github.com/dipsscor/MetricsMonitoring-Prometheus-Grafana/blob/master/screenshots/prom_alerts.png)
  
  
  
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
 
 	http://prometheus:9090
	
	
 In Grafana - dashboard.yml under ./env/provisioning/dashboards following config is used:
 
	folder: 'Spring-boot-apps'
	  type: 'file'
	  options:
	    folder: '/var/lib/grafana/dashboards'
	      
 #### Folder : Holds the dashboards . By default it is 'General'
 #### Options/Folder : holds the location where custom dashboards are injected.
 
 A custom dashboard is being used and modified from grafana market place for Springboot apps.
 
 
   ![alt text](https://github.com/dipsscor/MetricsMonitoring-Prometheus-Grafana/blob/master/screenshots/grafana1.png)
   
   ![alt text](https://github.com/dipsscor/MetricsMonitoring-Prometheus-Grafana/blob/master/screenshots/grafana2.png)
 
 
 # References
 
    https://linoxide.com/containers/setup-monitoring-docker-containers-prometheus/
    https://github.com/vegasbrianc/prometheus/blob/master/prometheus/prometheus.yml
    https://prometheus.io/docs/guides/cadvisor/
    
