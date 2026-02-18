# Syslog-ng Integration with Apache IoTDB

This documentation describes the concept and configuration for streaming log data into Apache IoTDB using syslog-ng, specifically focusing on the "Auto-Creation" (First Push) capability.


## Concept: Auto-Creation on 1st Push

The "First Push" concept ensures that the database schema (time series paths or tables) is generated automatically as soon as the first log message arrives.

- **Dynamic Path Mapping**: By using syslog-ng macros like ```${HOST}``` and ```${PROGRAM}```, data is routed into hierarchical paths (e.g., ```root.logs.device1.ssh```).
- **Schema-less Ingestion**: IoTDB allows the creation of time series on-the-fly during an ```INSERT``` statement. Syslog-ng leverages this to avoid manual database pre-configuration.
- **Scalability**: This approach allows for thousands of unique log sources to be registered in the IoTDB tree without administrative overhead.


## Configuration Example

The most efficient way to connect syslog-ng to IoTDB is via the **HTTP/REST API** or the **SQL (JDBC)** module. Below is a configuration using the HTTP module to leverage IoTDB's REST interface.

**syslog-ng**:
```conf
@version: 4.0

source s_network {
    network(port(514) transport("udp"));
};

destination d_iotdb_rest {
    http(
        url("http://127.0.0.1")
        method("POST")
        user("root")
        password("root")
        header("Content-Type: application/json")
        # Constructing the IoTDB-native JSON format
        body('{"timestamps": [${UNIXTIME}000], "measurements": ["message", "level"], "values": [["${MSG}", "${LEVEL}"]], "prefixPath": "root.syslog.${HOST}.${PROGRAM}"}')
    );
};

log {
    source(s_network);
    destination(d_iotdb_rest);
};
```


### Key Parameters for Setup

- **prefixPath**: Defines the location in the IoTDB storage group. Using ```${HOST}``` ensures that a new "device" is created in IoTDB as soon as a new host sends its first log.
- **Timestamps**: IoTDB requires precise timestamps (usually milliseconds). Using ```${UNIXTIME}000``` scales the syslog second-based timestamp to the required format.
- **Storage Group**: Ensure that your storage group (e.g., ```root.syslog```) is defined in IoTDB or that "Auto-Create Schema" is enabled in the ```iotdb-common.properties```.


### Implementation Steps

1. **Download JDBC/REST Drivers**: Ensure the Apache IoTDB REST Service is enabled in your IoTDB instance.
2. **Verify Permissions**: The user (e.g., ```root```) must have ```INSERT_TIMESERIES``` and ```CREATE_TIMESERIES``` privileges.
3. **Test Connectivity**: Use a simple ```logger``` command to send a test message and check the IoTDB CLI using ```SHOW TIMESERIES```.

For detailed module options, refer to the [syslog-ng](https://syslog-ng.github.io/admin-guide/010_Introduction_to_syslog-ng/README "Introduction to syslog-ng") HTTP Destination Documentation.
