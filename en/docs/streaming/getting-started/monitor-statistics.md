# Step 7: Monitor Statistics

This step shows how you can monitor the CDC and file statistics of the WSO2 Streaming Integrator deployment you started and the `SweetFactoryApp` Siddhi application you created and deployed in the previous steps. For this purpose, you are using the some of the pre-configured dashboards provided by WSO2 Streaming Integrator. You can host these dashboards in Grafana and view statistices related to ETL activities carried out by the Streaming Integrator. For more information about these dashboards, see [Monitoring ETL Statistics with Grafana]({{base_path}}/observe/si-observe/viewing-etl-flow-dashboards)

## Configuring WSO2 SI to visualize statistics

To be able to see visualizations of statistics generated by WSO2 Streaming Integrator, you are required to download and install Prometheus and Grafana. You need to download the required pre-configured dashboards and import them to Grafana.

### Downloading the required dashboards

WSO2 Streaming Integrator provides you with pre-configured dashboards in JSON format. You can import these dashboards bto Grafana to view statistics of your Streaming Integrator deployment.

For this scenario, download the following dashboards:

- [WSO2 Streaming Integrator - Overall Statistics.json](https://github.com/wso2/streaming-integrator/blob/master/modules/distribution/carbon-home/resources/dashboards/overview-statistics/WSO2%20Streaming%20Integrator%20-%20Overall%20Statistics.json)

- [WSO2 Streaming Integrator - App Statistics.json](https://github.com/wso2/streaming-integrator/blob/master/modules/distribution/carbon-home/resources/dashboards/overview-statistics/WSO2%20Streaming%20Integrator%20-%20App%20Statistics.json)

- [WSO2 Streaming Integrator - CDC Statistics.json](https://github.com/wso2/streaming-integrator/blob/master/modules/distribution/carbon-home/resources/dashboards/cdc-statistics/WSO2%20Streaming%20Integrator%20-%20CDC%20Statistics.json)

- [WSO2 Streaming Integrator - CDC Streaming Statistics.json](https://github.com/wso2/streaming-integrator/blob/master/modules/distribution/carbon-home/resources/dashboards/cdc-statistics/WSO2%20Streaming%20Integrator%20-%20CDC%20Streaming%20Statistics.json)

- [WSO2 Streaming Integrator - File Sink Statistics.json](https://github.com/wso2/streaming-integrator/blob/master/modules/distribution/carbon-home/resources/dashboards/file-statistics/WSO2%20Streaming%20Integrator%20-%20File%20Sink%20Statistics.json)

- [WSO2 Streaming Integrator - File Source Statistics.json](https://github.com/wso2/streaming-integrator/blob/master/modules/distribution/carbon-home/resources/dashboards/file-statistics/WSO2%20Streaming%20Integrator%20-%20File%20Source%20Statistics.json)

- [WSO2 Streaming Integrator - File Statistics.json](https://github.com/wso2/streaming-integrator/blob/master/modules/distribution/carbon-home/resources/dashboards/file-statistics/WSO2%20Streaming%20Integrator%20-%20File%20Statistics.json)


### Downloading and setting up Prometheus

WSO2 Streaming Integrator uses Perometheus to expose its statistics to Grafana. Therefore, to download and configure Prometheus, follow the steps below:

1. Download Prometheus from the [Prometheus site](https://prometheus.io/download/). For instructions, see [the Prometheus Getting Started Guide](https://prometheus.io/docs/prometheus/latest/getting_started/).

2. Extract the downloaded file. The directory that opens as a result is referred to as the `<PROMETHEUS_HOME>` from here on.

3. To enable statistics for the Prometheus reporter, open the `<SI_HOME>/conf/server/deployment.yaml` file and set the `enabled` parameter in the `wso2.metrics` section to `true`, and update the other parameters in the section as shown below. You also need to add the `metrics.prometheus:` as shown.

    ```
     wso2.metrics:
       # Enable Metrics
       enabled: true
       reporting:
         console:
           - # The name for the Console Reporter
             name: Console
    
             # Enable Console Reporter
             enabled: false
    
             # Polling Period in seconds.
             # This is the period for polling metrics from the metric registry and printing in the console
             pollingPeriod: 2
    
     metrics.prometheus:
      reporting:
        prometheus:
          - name: prometheus
            enabled: true
            serverURL: "http://localhost:9005"
    ```
   
4. Open the `<PROMETHEUS_HOME>/prometheus.yml` file and add the following configuration in the `scrape_configs:` section.

    ```
     scrape_configs:
       - job_name: 'prometheus'
         static_configs:
         - targets: ['localhost:9005']
    ```
   
5. In the terminal, navigate to the `<PROMETHEUS_HOME` and issue the following command to start the Prometheus server.

    `./prometheus`
    
!!! info
    The above steps to configure and start Prometheus need to be performed before you start the Grafana server.
    
### Downloading and setting up Grafana

The pre-configured dashboards provided by WSO2 Streaming Integrator which you previously downloaded are rendered in Grafana to visualize statistics. To download and set up Grafana, follow the steps below:

!!! tip "Before you begin:"
    Start the Prometheus server as instructed under [Downloading and setting up Prometheus](#downloading-and-setting-up-prometheus).

1. Download Grafana from the [Grafana Labs - Download Grafana](https://grafana.com/grafana/download).

2. Start Grafana.

    !!! info
        The procedure to start Grafana depends on your operating system and the installation process. e.g., If your operating system is Mac OS and you have installed Grafana via Homebrew, you start Grafana by issuing the `brew services start grafana` command.
        
3. In the **Data Sources** section, click **Add your first data source**. In the **Add data source** page that appears, click **Select** for **Prometheus**.
       
4. In the **Add data source** page -> **Settings** tab, update the configurations for Prometheus as follows.<br/>   
   ![prometheus configuration]({{base_path}}/assets/img/streaming/cdc-monitoring/prometheus-configurations.png)<br/>    
   1. Click **Default** to make Prometheus the default data source.
   
   2. Under **HTTP**, enter `http://localhost:9090` as the URL.
   
   3. Click **Save & Test**. If the data source is successfully configured, it is indicated via a message.
       ![Save and Test]({{base_path}}/assets/img/streaming/cdc-monitoring/save-and-test.png)
       
   4. To import the dashboards that you previously downloaded as JSON files, follow the procedure below:
   
    1. Start Grafana and access it via http://localhost:3000/.
        
    2. To load a new dashboard, click the plus icon **(+)** in the side panel. Then click **Import**.
        
    3. In the **Import** page, click **Upload .json** file. Then browse and select the .json file of the pre-configured dashboard that you downloaded (i.e., in step 5, substep 1).
        
    4. If required, change the unique identifier displayed in the **Unique Identifier (uid)**.
        
    5. Click **Import**.
    
## Enable the Siddhi application to publish statistics
    
To enable the `SweetFactoryApp` Siddhi application to publish statistics to Prometheus, add the `@App:statistics(reporter = 'prometheus')` annotation to it below the `@App:name` annotation as shown below:

    ```text
        @App:name('SweetFactoryApp')
        @App:statistics(reporter = 'prometheus')
    ```
!!! tip 
    You can update the Siddhi application in Streaming Integrator Tooling and deploy it again in the Streaming Integrator server as you did in [Step 5: Update the Siddhi Application](update-the-siddhi-application.md).
    
## Viewing statistics

To generate some statistics and view them, follow the procedure below.

1. Start WSO2 Streaming Integrator.

2. To generate statistics, insert as many events as you want into the `SweetProductionTable` MySQL table that you created for this scenario in [Step 1: Download Streaming Integrator and Dependencies](download-install-and-start-si.md). Also, manually add as many rows as you want in the `/Users/foo/productioninserts.csv` file.

3. Access Grafana via the `localhost:3000` URL.

4. In the side panel, click the **Dashboards** icon and click **Dashboards**.

    ![Access Dashboards]({{base_path}}/assets/img/streaming/quick-start-guide-101/access-grafana-dashboards.png)
    
    Then click on the **WSO2 Streaming Integrator - Overall Statistics** dashboard. It opens as follows.
    
    !!! info
        The statistics displayed will be different based on the number of records you inserted to the `SweetProductionTable` MySQL table and the number of rows you added in the `/Users/foo/productioninserts.csv` file during the last 30 minutes. You can also change the time interval for which statistics are displayed via the field for selecting the time interval in the top panel.
    
    ![overall-statistics]({{base_path}}/assets/img/streaming/quick-start-guide-101/overall-staistics.png)
    
5. Under **Overview Statistics**, click **SweetFactoryApp**. The **overview-statistics / WSO2 Streaming Integrator App Statistics** dashboard opens.

    ![app-statistics]({{base_path}}/assets/img/streaming/quick-start-guide-101/app-staistics.png)
    
6. Scroll down to the **Sources** section. The following is displayed.

    ![source-statistics]({{base_path}}/assets/img/streaming/quick-start-guide-101/sources.png)
    
    The two entries displayed above represent the `file` source and the `cdc` source used in the `SweetFactoryApp` Siddhi application.
    
7. Scroll down further to the **Destinations** section. The `file` sink in the `SweetFactoryApp` Siddhi application is displayed as shown below.

    ![destination-statistics]({{base_path}}/assets/img/streaming/quick-start-guide-101/destination.png)
    
8. Under **Sources**, click on the link to the `productioninserts.csv` file. The **WSO2 Streaming Integrator - File Statistics** dashboard opens. The contents of the `productioninserts.csv` file is the output of one query and the input of another. Therefore, it is a source as well as a destination, statistics are displayed for it under **Source** and **Sink** as shown below.

    **Source Statistics**
    
    ![File Statistics - Source]({{base_path}}/assets/img/streaming/quick-start-guide-101/file-statistics-source.png)
    
    **Sink Statistics**
    
    ![File Statistics - Sink]({{base_path}}/assets/img/streaming/quick-start-guide-101/file-statistics-sink.png)
    
9. Under **WSO2 Streaming Integrator - File Statistics** dashboard -> **Sources**, click on the file link. The **file-statistics / WSO2 Streaming Integrator / File Source Statistics**  dashboard opens displaying detailed statistics for the file when it is functioning as a source.

    ![File Source Statistics]({{base_path}}/assets/img/streaming/quick-start-guide-101/file-statistics-sink.png)

10. Under **WSO2 Streaming Integrator - File Statistics** dashboard -> **Sources**, click on the file link. The **file-statistics / WSO2 Streaming Integrator / File Sink Statistics**  dashboard opens displaying detailed statistics for the file when it is functioning as a source.

    ![File Sink Statistics]({{base_path}}/assets/img/streaming/quick-start-guide-101/file-statistics-sink.png)
                                                                                   
11. In the **overview-statistics / WSO2 Streaming Integrator App Statistics** dashboard -> **CDC** section, click on the **SweetProductionTable** link. The **cdc-statistics / WSO2 Streaming Integrator / CDC Statistics**  dashboard opens with statistics generated for the `cdc` source in the `SweetFactoryApp` Siddhi application.

    ![CDC Statistics]({{base_path}}/assets/img/streaming/quick-start-guide-101/cdc-statistics.png)
    
    Under **Streaming**, click on the **SweetProductionTable** link. The **cdc-statistics / WSO2 Streaming Integrator / CDC Streaming Statistics** dashboard opens as follows.
    
    ![CDC Streaming Statistics]({{base_path}}/assets/img/streaming/quick-start-guide-101/cdc-streaming-statistics.png)
    
!!! tip "What's Next?"
    - To learn more about the key concepts of the Streaming Integrator component, see [Streaming Integrator Key Concepts]({{base_path}}/streaming/streaming-key-concepts.md).<br/>    
    - For more hands-on experience with Streaming Integrator component, try the [Streaming Integrator Tutorials]({{base_path}}/use-cases/streaming-tutorials/tutorials-overview.md).<br/>    
    - To learn about use cases specific to the Streaming Integrator component, see [Streaming Integrator Use Cases]({{base_path}}/use-cases/streaming-usecase/use-cases).<br/>
    -  Learn how to run WSO2 Streaming Integrator in containerized environments, try [Running SI with Docker and Kubernetes](use-cases/streaming-tutorials/running-si-with-docker-and-kubernetes)

    

    