# Command Line Interface

**TritanDb CLI **\([https://github.com/eugenesiow/tritandb-cli](https://www.gitbook.com/book/eugenesiow/tritandb/edit#)\)** **is the command line client to interact with a TritanDB server.

## Usage

```
usage: tritandb-cli [-h] --ingest [-H HOST] [-P PORT]

required arguments:
  --ingest,     Command/Mode of Operation
  --query,
  --list


optional arguments:
  -h, --help    show this help message and exit

  -H HOST,      Server Host e.g. localhost or 192.168.0.100
  --host HOST

  -P PORT,      Server Port e.g. 5700
  --port PORT

```

## Installation

You can build TritanDb's Command Line Client using Gradle

```
git clone https://github.com/eugenesiow/tritandb-cli.git
cd tritandb-cli
./gradlew build
```



