# Project Access

The microservices governance platform is designed to help you better observe the health status of your services, covering different dimensions such as distributed full-link tracking, error analysis, alarm management and custom dashboard, and simplify the operation and maintenance process to make it easier and more efficient.

The platform manages associated application data from a project perspective, so you need to focus on how to create or access microservice governance projects first to use the monitoring and analysis functions provided by the platform. It is necessary as the data generated by the application, such as metrics, traces, and logs, must be explicitly authorized to be properly captured and stored by the platform. The process is as follows:

1. Create a project, and data belonging to the same project will be managed together.
2. Generate an authorization token to identify the attribution of the reported data.
3. The application integrates the collection SDK to report monitoring data.

Go to **Org Center > Projects > Add Project**. There are several project types available, such as DevOps project, monitoring project, code hosting project, etc. DevOps project and monitoring project are supported for integration by the microservice governance platform, with different access methods, which will be introduced in detail in the following.

![](http://terminus-paas.oss-cn-hangzhou.aliyuncs.com/paas-doc/2022/03/17/b4cfec75-fc59-4e26-a6b5-ca5eb46cef4b.png)


## How to Select a Project

### DevOps Project

DevOps projects provide functions such as project management, code hosting, and CI/CD. For DevOps projects, the platform accesses the microservice governance platform by providing a monitor addon, which will automatically complete the injection of authorization token and collection SDK.

You can choose the DevOps project for the following situations:

- In addition to the needs of the microservice governance platform, you also need the features such as project management, code hosting, and CI/CD provided by the DevOps project.
- Your application language is Java or Nodejs (currently the monitor addon only supports SDK injections for programs in these two languages).

### Monitoring Project

The monitoring project focuses on features such as service monitoring, tracing analysis, custom dashboard, and alarm. Compared to DevOps projects, monitoring projects offer a more common, flexible, and configurable way for access, while serving as the server for protocols such as OpenTelemetry and OpenTracing, and integrating with numerous collection SDKs that support the protocol.

You can choose the monitoring project for the following situations:

- You only need the monitoring features provided by the microservices governance platform.
- Your project is not hosted by Erda and cannot be accessed as the DevOps project described above.
- Your application language is not supported by the monitor addon.
- You already have a collection SDK that is compatible with the OpenTelemetry or OpenTracing protocol.

## DevOps Project Access

Before you start, please make sure you have created a DevOps project, erda.yaml, and pipeline for deployment. For details, see [Deploy via Pipeline](../../../dop/guides/deploy/deploy-by-cicd-pipeline).

### Add Monitor Addon

::: tip Tips
This step is optional. Applications deployed through the pipeline will have the monitor addon added automatically.
:::

Edit erda.yaml to add the monitor addon.

You can add by the graphic editing mode:

![](http://terminus-paas.oss-cn-hangzhou.aliyuncs.com/paas-doc/2022/03/17/3cda3da3-f0f3-4773-961a-9882da381978.png)

Or edit erda.yaml in text mode:

```yaml
  addons:
    monitor:
      plan: monitor:basic
      options:
        version: '3.6'
```

### Run Pipeline to Deploy Application

1. Go to **App > Pipeline**, add a new pipeline and run it.

   During the pipeline deployment, it will perform specific logic specified by the monitor addon:

   - Register a project with the microservice governance platform.

   - Apply for an authorization token from the microservices governance platform.

   - Add the collection SDK to the packaged image.

   - Inject the authorization token into the environment variable of the deployed application.


2. After the pipeline is successfully executed, you can go to the homepage of the microservice governance platform to view the project just deployed.


## Monitoring Project Access

### Add Monitoring Project

Please follow the steps below to create a project:

1. Go to **Org Center > Projects**.


![](http://terminus-paas.oss-cn-hangzhou.aliyuncs.com/paas-doc/2022/03/17/6708136b-6071-4c62-aa83-aaac2e1eae43.png)

2. Click **Add Project**.

3. Select the **Monitoring Project**, and enter the project name, identifier and other information according to the prompts.

![](http://terminus-paas.oss-cn-hangzhou.aliyuncs.com/paas-doc/2022/03/17/b4cfec75-fc59-4e26-a6b5-ca5eb46cef4b.png)

4. After completing the project creation, go to the homepage of the microservice governance platform to view the project just created, and click to enter the project space.

### Create Access Token

1. Go to **Microservice Governance Platform > Environment Settings > Access Configuration**.

![](http://terminus-paas.oss-cn-hangzhou.aliyuncs.com/paas-doc/2022/03/17/47bf7148-9fc8-4ea4-b309-aac648f5fb4b.png)

2. Click **Create Token** in the upper right corner, and the page will show the information of the created token.

![](http://terminus-paas.oss-cn-hangzhou.aliyuncs.com/paas-doc/2022/03/17/5fb3e667-2ff0-4f36-995a-a45546ffd147.png)

### Integrate SDK

For external project access, considering the multi-language and openness issues, currently the platform integrates with the protocols of the open-source community and supports all collection SDKs compatible with OpenTelemetry and OpenTracing to report data.

On the access configuration page, the platform provides instructions for Jaeger client access in multiple languages by default. Here takes the Java language as an example.

#### Get Endpoint and Authentication Information

![](http://terminus-paas.oss-cn-hangzhou.aliyuncs.com/paas-doc/2022/03/17/67ecfada-c0b6-44c7-9110-342dcbb0f9a3.png)

- **Endpoint**: The API address for data receiving.
- **Environment ID**: Different projects and environments (development, testing, staging, production) have different environment IDs to distinguish data attribution.
- **Token**: Used for authentication.

#### Reference SDK and Configure Authentication Information

You can configure authentication information in the header of the HTTP/HTTPS protocol or the tag field of the Jaeger protocol. The fields and descriptions are as follows.

- **erda.env.id**: Environment ID of microservice project.
- **erda.env.token**: Authentication token of microservice environment, created and obtained on this page.

The following is an example of data reported by Jaeger SpringCloud Starter:

1. Import opentracing-spring-jaeger-cloud-starter dependency.

   ```xml
   <dependency>
       <groupId>io.opentracing.contrib</groupId>
       <artifactId>opentracing-spring-jaeger-cloud-starter</artifactId>
       <version>3.3.1</version>
   </dependency>
   ```

2. Configure Jaeger in application.yml.

   ```yaml
   opentracing:
     jaeger:
       service-name: <your_service_name>
       http-sender:
         url: https://collector.daily.terminus.io/api/jaeger/traces
       log-spans: true
       tags:
         erda.env.id: <your_env_id>
         erda.env.token: <your_token>
   ```
