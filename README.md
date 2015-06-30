Flurry
======

Unique id generation service in scala

Flurry is a "unique id" generation service written in scala, but accesible in any language using thrift. Flurry took its inspiration (and very little code) from Twitter's snowflake service, but by starting from scratch, we have what I think is a cleaner implementation, less dependencies, and it's easily configurable and runnable.


## Running

Flurry comes with a service wrapper, which allows it to run on linux or OSX.  In order to run Flurry, follow the instructions below.

1. Download a [release archive](https://github.com/BazuSports/flurry/releases)
2. Unzip the release archive and cd into the application directory
3. Use the service wrapper to run Flurry (i.e. _service/flurry_)


## Configuration

Flurry allows you to provide you own configuration file to override the default properties.  The default configuration is below.

    flurry {
        server-port = 9090

        generator-class = com.chronotrack.flurry.ConfigurableGenerator

        generator-config {
            worker-id-generator-class = com.chronotrack.flurry.worker.StaticWorkerIdGenerator
            workerId                  = 2
            epochStart                = 1376937222000
            workerIdBits              = 14
            sequenceBits              = 10
        }
    }


In order to override the values, you can put a file named application.conf in the flurry application directory and restart the flurry service.

Here is an example of a custom configuration file.  You can omit any values you're not overriding as they will be infered from the default configuration.

    flurry {
        server-port = 9091

        generator-config {
            worker-id-generator-class = com.chronotrack.flurry.worker.HostWorkerIdGenerator
        }
    }


### Configuration properties

* **server-port** - the port on which the thrift server will listen on

* **generator-class** - the implementation class of the generator (for now, flurry only supports the ConfigurableGenerator class)

* **generator-config** - configurations used by the generator

    * **worker-id-generator-class** - the class the is responsible for generating the worker id.  There are currently two
    supported classes (HostWorkerIdGenerator and StaticWorkerIdGenerator)
    * **workerId** - If you are using StaticWorkerIdGenerator, you can specify a static worker id using this option
    * **epochStart** - The start of the epoch time for your software (once you create this, changing it can and will create collisions)
    * **workerIdBits** - The number of bits to use for the worker part of the 64 bit id
    * **sequenceBits** - The number of bits to use fot the sequence bit part of the id

_64 - (workerIdBits + sequenceBits) is the number of bits used for the time part of the id, so make sure you leave enough to support however long of a period you think you need.
(i.e. 40 bits = 2^40 = 1099511627776. Divide by 3.15569e10 [milliseconds in a year] and you get 34.8 years._

_Same goes for worker id and sequence._

* _14 bits for worker id = 2^14 = 16384 unique machines you can support._
* _10 bits for sequence = 2^10 = 1024 ids per millisecond per worker you can support._
    
_You can tune your time, worker, sequence support as needed for your application_

## Running in Docker

Flurry allows you to run the service inside a docker container. All the files needed to build the Docker container are inside the ./docker/ directory. A default application.conf is placed here to be used to configure your flurry containers. If you wish to change the template modify docker/application.conf.

1. docker build -t username/flurry .
2. docker run -d -P -e WORKER_ID=1 --name flurry-1 username/flurry
3. docker port flurry-1 # Find the port on which Docker exposed flurry-1 
4. Connect clients to port found above

## Running in fleetctl

Flurry has been set up to run inside a CoreOS system using fleetctl.

1. cd systemd/
2. fleetctl start flurry@{1..10} # Starts 10 isntances of flurry
3. fleetctl start flurry-discovery@{1..10} # Starts dicovery services that register the flurry service with etcd
4. fleetctl start flurry-example@1 # Starts a consumer that will query etcd for flurry endpoints and then use them to print IDs until killed


