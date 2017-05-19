# Protocol Buffers Format

```protobuf
syntax='proto3';
package com.tritandb;

option java_package = "com.tritandb.engine.tsc.data";
option java_outer_classname = "EventProtos";

message Row {
    int64 timestamp = 1;
    repeated int64 value = 2;
}

message Rows {
    repeated Row row = 1;
}

message Col {
    string name = 1;
    enum ColType {
        LONG = 0;
        DOUBLE = 1;
        BOOLEAN = 2;
    }
    ColType type = 2;
}

message Cols {
    repeated Col col = 1;
}

message TritanEvent {
    enum EventType {
        INSERT = 0;
        INSERT_BATCH = 1;
        CLOSE = 2;
        QUERY = 3;
        INSERT_META = 4;
    }
    EventType type = 1;
    string name = 2;
    Rows rows = 4;
    Cols cols = 5;
}
```



