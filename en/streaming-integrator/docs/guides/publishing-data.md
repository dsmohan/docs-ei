# Publishing Messages

## Introduction
Once information is processed by the Streaming Integrator, the output is presented as events in a streaming manner. 
This output can be published to databases, files, cloud-based applications or other streaming applications.

For the Streaming Integrator to publish events, the following is required.

* A message schema: The messages selected to be published by a streaming integration flow are identified by their 
schemas. This schema is defined via an output stream definition. 

* A sink: The output can be published to different interfaces including streaming applications, cloud-based applications,
 databases, and files. There are different sink types to support the different interfaces. The output can also be published in a range of formats. In order to select the required interface and format for a specific streaming integration flow, you need to configure a sink in the relevant Siddhi application via the `@sink` annotation.
 
   ![Publishing events](../images/publishing-messages/Publish-Messages-Flow.png)
 
 As shown in the image above, a sink configuration consists of three parts.
 
   + **`@sink`**: This annotation defines the sink type via which the messages are published, and allows you to configure the sink parameters (which change depending on the sink type). For the complete list of supported sink types, see [Siddhi Query Guide - Sink](https://siddhi.io/en/v4.x/docs/query-guide/#sink)
   + **`@map`**: This annotation specifies the format in which messages are published, and allows you to configure the mapping parameters (which change based of the mapping type/format selected). For the complete list of supported mapping types, see [Siddhi Query Guide - Sink Mapper](https://siddhi.io/en/v4.x/docs/query-guide/#sink-mapper)
   + **`@attributes`**: This annotation specifies a custom mapping based on which events in the streaming integration flow that need to be published are identified. This is useful when the attributes of the output messages you want the Streaming Integrator to publish are different to the corresponding attribute name in the stream definition. e.g., In a scenario where the Streaming Integrator is publishing the average temperature per second, the temperature can be referred to as  `avgTemp` in the output stream definition in your Siddhi application. However, you want to publish it with the `Temperature` to the streaming application to which you are publishing. In this instance, you need a custom mapping to indicate that `Temperature` is the same as `avgTemp`.

## Publishing messages using an event sink
This section explains how to configure a basic sink without mapping. A Siddhi application can contain a sink configuration inline, or refer to a sink configuration that is defined externally in a configuration file.

#### Defining event sink inline in the Siddhi application
To create a Siddhi application with the sink configuration defined inline, follow the steps below.

1. Open the Streaming Integrator Studio and start creating a new Siddhi application. For more information, see [Creating a Siddhi Application](../develop/creating-a-Siddhi-Application.md).
2. Enter a name for the Siddhi application as shown below.<br/>
   `@App:name("<Siddhi_Application_Name>)`<br/>e.g., `@App:name("SalesTotalsApp")`<br/>
3. Define the output stream based on the schema in which you want to publish messages. The format is as follows. <br/>
   `define stream <Stream_Name>(attribute1_name attribute1_type, attribute2_name attribute2_type, ...);`
   e.g., 
   `define stream PublishSalesTotalsStream (transNo int, product string, price int, quantity int, salesValue long);`
4. Connect a sink to the stream definition you added as follows.
    ```jql
    @sink(type='<SINK_TYPE>')
    define stream <Stream_Name>(attribute1_name attribute1_type, attribute2_name attribute2_type, ...);
    ```
    Here, the sink type needs to be selected based on the interface to which you want to publish the output. For more information, see [Supported sink types](#supported-event-sink-types).
    
    e.g., If you want to publish the output as logs in the console, you can add a sink with `log` as the type.
    ```jql
    @sink(type='log')
    define stream PublishSalesTotalsStream (transNo int, product string, price int, quantity int, salesValue long);
    ```
    
5. Add and configure parameters related to the sink type you selected as shown below.
    ```jql
    @sink(type='<SINK_TYPE>', <PARAMETER1_NAME>='<PARAMETER1_VALUE>', ...)
    define stream <Stream_Name>(attribute1_name attribute1_type, attribute2_name attribute2_type, ...);
    ```
    e.g., By adding a parameter named `prefix` to the log sink used as an example in the previous step, you can specify a prefix with which you want to see the output logs printed.
    ```jql
    @sink(type='log', prefix='Sales Totals:')
    define stream PublishSalesTotalsStream (transNo int, product string, price int, quantity int, salesValue long);
    ```
    
6. Now let's complete adding the required Siddhi constructs to receive and process the input messages.

    1. Add an input stream with a connected source configuration as shown below. For more information, see the [Consuming Messages guide](consuming-messages.md).
        ```
        @source(type='<SOURCE_TYPE>')
        define stream <Stream_Name>(attribute1_name attribute1_type, attribute2_name attribute2_type, ...);
        ```
        
        e.g., Assuming that the schema of the input messages are same as that of the output messages, and that they are received via HTTP, you can add the input stream definition
       ```
       @source(type='http', receiver.url='http://localhost:5005/SweetProductionEP')
       define stream ConsumeSalesTotalsStream (transNo int, product string, price int, quantity int, salesValue long);
       ```
    2. Add a query to get the received events from the input stream and direct them to the output stream as follows.
        ```jql
        from <INPUT_STREAM_NAME>
        select <ATTRIBUTE1_Name>, <ATTRIBUTE2_NAME>, ... 
        group by <ATTRIBUTE_NAME>
        insert into <OUTPUT_STREAM_NAME>;
        ```
        e.g., Assuming that you are publishing the events with the existing values as logs in the output console without any further processing, you can define the query as follows.
        
        ```jql
        from ConsumeSalesTotalsStream
        select transNo, product, price, quantity, salesValue
        group by product
        insert into PublishSalesTotalsStream;
        ``` 
   
7. Save the Siddhi Application.

#### Defining event sink externally in the configuration file

### Supported event sink types
<Sink categories table here> 

### Supported message formats

#### Publishing a message in default format

#### Publishing a message in custom format
