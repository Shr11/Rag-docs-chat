Filter Operator

The Filter Operator is a powerful tool that allows you to filter messages based on a condition. It is a simple operator that takes a single input and returns a boolean value. If the input meets the condition, the operator will return true. Otherwise, it will return false.
Prerequisites

This guide uses local Fluvio cluster. If you need to install it, please follow the instructions at [here][installation].
Syntax

Here is a simple filter operator. It takes a string as input and returns a boolean value. If the input string contains a question mark, the operator will return true. Otherwise, it will return false.

    transforms:
      - operator: filter
        run: |
          fn filter_questions(input: String) -> Result<bool> {
            Ok(input.contains("?"))
          }

Running the Example

Copy and paste following config and save it as dataflow.yaml.

# dataflow.yaml

apiVersion: 0.5.0
meta:
name: filter-example
version: 0.1.0
namespace: examples

config:
converter: raw

topics:
sentences:
schema:
value:
type: string

questions:
schema:
value:
type: string

services:
filter-service:
sources: - type: topic
id: sentences

    transforms:
      - operator: filter
        run: |
          fn filter_questions(input: String) -> Result<bool> {
            Ok(input.contains("?"))
          }

    sinks:
      - type: topic
        id: questions

To run example:

$ sdf run --ephemeral

Produce sentences to in sentence topic:

$ echo "Hello world" | fluvio produce sentences
$ echo "Are you there?" | fluvio produce sentences

Consume topic questions to retrieve the result in another terminal:

$ fluvio consume questions -Bd
Are you there?

Only questions are returned.
Cleanup

Exit sdf terminal and clean-up. The --force flag removes the topics:

sdf clean --force

Conclusion
We just covered one of the most basic operators in SDF, the Filter Operator. Combining with other basic operators, you can build more complex dataflows to process your data.

Map Operator

The Map Operator is a versatile tool that allows you to transform an existing message and re-encode the input into a new message format. The operator takes the input and maps it into any type. Every message in the original topic will be mapped to a new topic. Please see filter-mapping for filter and mapping.
Prequisites

This guide uses local Fluvio cluster. If you need to install it, please follow the instructions at [here][installation].
Syntax

Below is an example of a tranform function. The example demostrates how to map a sentence into a new string that outputs the size of the original string.

    transforms:
      - operator: map
        run: |
          fn get_sentence_length(input: String) -> Result<String> {
            Ok(format!("Inserted string size {}",input.len()))
          }

Running the Example

Copy and paste following config and save it as dataflow.yaml.

# dataflow.yaml

apiVersion: 0.5.0
meta:
name: filter-example
version: 0.1.0
namespace: examples

config:
converter: raw

topics:
sentences:
schema:
value:
type: string
len:
schema:
value:
type: string

services:
filter-service:
sources: - type: topic
id: sentences

    transforms:
      - operator: map
        run: |
          fn get_sentence_length(input: String) -> Result<String> {
            Ok(format!("Inserted string size {}",input.len()))
          }

    sinks:
      - type: topic
        id: len

To run example:

$ sdf run --ephemeral

Produce sentences to in sentence topic:

$ echo "This string is size 22" | fluvio produce sentences
$ echo "This string is size 23." | fluvio produce sentences

Consume topic questions to retrieve the result in another terminal:

$ fluvio consume len -Bd
Inserted string size 22
Inserted string size 23

Each string will be mapped via the function provided.
Cleanup

Exit sdf terminal and clean-up. The --force flag removes the topics:

sdf clean --force

Conclusion
We just covered another basic operator in SDF, the Map Operator.

Deployment/Worker Management

There are two modes to run dataflow in SDF: ephemeral and worker. When CLI is run in ephemeral mode, it will run the dataflow in the same process as the CLI. The dataflow will be terminated upon exiting CLI. This is useful for development dataflow as well as testing the package.

Worker is a process that runs the dataflows continuously. Unlike ephemeral CLI, worker is long running process. That means, when you exit CLI, dataflow does not terminate. It is primary used for running dataflow in production environment. It can also be used in development environment to run dataflow when there is no need to test package.

Worker can run anywhere as long as it can connect to the Fluvio cluster. Worker can be run in data center, cloud, or edge device. There is no limit on number of workers that can be run. Each worker can run multiple dataflows. It is recommended to a run a single worker per machine.

SDF can target different worker by selecting the worker profile. Worker profile is associated with the Fluvio profile. When you switch the Fluvio profile, SDF will automatically switch the worker profile. Once you have selected the worker, same worker will be used for all dataflow operation until you selecte different worker.

By default, SDF will run the dataflow in the worker. If you are using InfinyOn Cloud, the worker will be provisioned and automatically registered in the profile.
Using Host Worker for Local Cluster

If you are using local cluster, you need either create Host or register the Remote worker. Easiest is to create Host worker. Host worker run in the same machine as the CLI.

To create host worker, you can use the following command:

$> sdf worker create <name>

This will spawn the worker process in the background. It will run until you shutdown the worker or machine is rebooted. The name can be anything as long as it is unique for your machine since profile are not shared across different machines.

Once you have created the worker, You can list them:

$> sdf worker create main
Worker `main` created for cluster: `local`
$> sdf worker list
NAME TYPE CLUSTER WORKER ID

- main Host local 7fd7eda3-2738-41ef-8edc-9f04e500b919

The \* indicates the current selected worker. The worker id is internal unique identifier for current fluvio cluster. Unless specified, it will be generated by the system.

SDF only support running a single HOST worker for each machine since a single worker can support many dataflow. If you try to create another worker, you will get an error message.

$ sdf worker create main2
$ Starting worker: main2
There is already a host worker with pid 20686 running. Please terminate it first

Shutting down worker will terminate all running dataflow and worker process.

$> sdf worker shutdown main
sdf worker shutdown main
Shutting down pid: 20688
Shutting down pid: 20686
Host worker: main has been shutdown

Even though host worker is shutdown and removed from the profile, the dataflow files and state are still persisted. You can restart the worker and the dataflow will resume.

For example, if you have dataflow fraud-detector and car-processor running in the worker and you shut down the worker, the dataflow process will be terminated. But you can resume by recreating the HOST worker.

$> sdf worker create main

The local worker store the dataflow state in the local file system. The dataflow state is stored in the ~/.sdf/<cluster>/worker/<dataflow>. Sor for local cluster, files will be stored in ~/.sdf/local/worker/dataflows.

if you have delete the fluvio cluster, worker need to be manually shutdown and created again. This limitation will be removed in the future release
Remote Worker

Remote worker is a worker runs in different machine from the CLI. It is typicall set up by devops team for production environment.

Typical lifecycle for using remote worker is:

    Start remote worker in the server.
    Register the worker with the name in your machine.
    Run the dataflow in the remote worker.
    Unregister the worker when it is no longer needed. This doesn't shutdown the worker but remove it from the profile.

Note that there are many ways to manage the remote worker. You can use Kubernetes, Docker, Systemd, Terraform, Ansible, or any other tool that can manage the server process and ensure it can restart when server is rebooted.

InfinyOn cloud is good example of remote worker. When you create a cluster in InfinyOn cloud, it will automatically provision the worker for you. It will also sync the profile when cluster is created.

Once know there are remote worker, you can discover them using sdf worker list -all command.

$> sdf worker list -all
NAME TYPE CLUSTER WORKER ID

- main Host local 7fd7eda3-2738-41ef-8edc-9f04e500b919
  N/A Remote local edg2-worker-id

These shows a host worker in your local machine and remote worker with id edg2-worker-id running somewhere. To register the remote worker, you can use register command.

$> sdf worker register <name> <worker-id>

Example, registering the remote worker with name edge2 and worker id edg2-worker-id.

$> sdf worker register edge2 edg2-worker-id
Worker `edge2` is registered for cluster: `local`

You can switch among worker using switch command.

$> sdf worker switch <worker_profile>

To unregister the worker after you are done with and no longer need, you can use unregister command:

$> sdf worker unregister <name>

Listing and switching the worker

To list all known worker, you can use list command:

$> sdf worker list
NAME TYPE CLUSTER WORKER ID

- main Host local 7fd7eda3-2738-41ef-8edc-9f04e500b919
  edge2 Remote local edg2-worker-id

To switch the worker, you can use switch command:

$> sdf worker switch <worker-name>

This assumes worker-name is already created or registered.
Managing dataflow in worker

When you are running dataflow in the worker, it will indicate name of the worker in the prompt:

$> sdf run
[main] >> show state

Listing and selecting dataflow

To list all dataflow running in the worker, you can use dataflow list command:

$> sdf dataflow list
[jolly-pond]>> show dataflow
Dataflow Status  
 wordcount-window-simple running

- user-job-map running
  [jolly-pond]>>

Other commands like show state requires active dataflow. If there is no active dataflow, it will show error message.

[jolly-pond]>> show state
No dataflow selected. Run `select dataflow`
[jolly-pond]>>

To select the dataflow, you can use dataflow select command:

[jolly-pond]>> select dataflow wordcount-window-simple
dataflow switched to: wordcount-window-simple

Deleting dataflow

To delete the dataflow, you can use dataflow delete command:

$> sdf dataflow delete user-job-map
[jolly-pond]>> show dataflow
Dataflow Status  
 wordcount-window-simple running

Note that since user-job-map is deleted, it is no longer listed in the dataflow list.
Using worker in InfinyOn Cloud

With InfinyOn Cloud, there is no need to manage the worker. It provisions the worker for you. It also sync profile when cluster is created.

For example, creating cloud cluster will automatically provision and create SDF worker profile.

$> fluvio cloud login --use-oauth2
$> fluvio cloud cluster create
Creating cluster...
Done!
Downloading cluster config
Registered sdf worker: jellyfish
Switched to new profile: jellyfish

You can unregister the cloud worker like any other remote worker.
Advanced: Starting remote worker

To start worker as remote worker, you can use launch command:

$> sdf worker launch --base-dir <dir> --worker-id <worker-id>

where base-dir and worker-id are optional parameters. If you don't specify base-dir, it will use the default directory: /sdf. If you don't specify worker-id, it will generate unique id for you.

This script is typically used by devops team to start the worker in the server.

Fluvio is a lightweight high-performance distributed data streaming system written in Rust and Web Assembly.
Quick Start - Get started with Fluvio in 2 minutes or less!
Step 1. Download Fluvio Version Manager:

On your terminal run

curl -fsS https://hub.infinyon.cloud/install/install.sh | bash

Follow the instructions and copy/paste the path to the bin directory to your startup script file.

Fluvio version manager will give you the ability to download different versions of Fluvio:

    Including our read-only edge cluster with built-in compression, caching, and mirroring to never lose data even with extended downtimes.
    Or our Developer Preview of Stateful Streaming which we are building using the web assembly component model to support all web assembly compatible languages.

Step 2. Start local cluster:

The following command will start a local cluster by default:

fluvio cluster start

Step 3. Create Topic:

The following command will create a topic called hello-fluvio:

fluvio topic create hello-fluvio

Step 4. Produce to Topic, Consume From Topic:

Produce data to your topic. Run the command first and then type some messages:

fluvio produce hello-fluvio

> hello fluvio
> Ok!
> test message
> Ok!

Consume data from the topic, Run the following command in a different terminal:

fluvio consume hello-fluvio -B -d

Just like that! You have a local cluster running.
Using Pre-Build Fluvio Versions

You may want to prefer other Fluvio versions than the latest stable release. You can do so by specifying the version in the VERSION environment variable.

Install Latest Release (as of master branch)

$ curl -fsS https://hub.infinyon.cloud/install/install.sh | VERSION=latest bash

Install Specific Version

$ curl -fsS https://hub.infinyon.cloud/install/install.sh | VERSION=x.y.z bash

Next Steps

Now that you have a cluster running you can try building data flows in different paradigms.
Check Fluvio Core Documentation

Fluvio documentation will provide additional context on how to use the Fluvio clusters, CLI, clients, a development kits.

    Fluvio docs home
    Fluvio CLI docs home
    Fluvio Architecture

Learn how to build custom connectors

Fluvio can connect to practically any system that you can think of.

    For first party systems, fluvio clients can integrate with the edge system or application to source data.
    For third party systems fluvio connectors connect at the protocol level and collects data into fluvio topics.

Out of the box Fluvio has native http, webhook, mqtt, kafka inbound connectors. In terms of outbound connectors out of the box Fluvio supports SQL, DuckDB, Graphite, experimental builds of Redis, S3 etc.

Using Connector Development Kit, we built our existing connectors in a matter of few days. Check out the docs and let us know if you need help building any connector.

    Connector docs
    Connector Development Kit (cdk) docs

Learn how to build custom smart modules

Fluvio applies wasm based stream processing and data transformations. We call these reusable transformation functions smart modules. Reusable Smart modules are built using Smart Module Development Kit and can be distributed using InfinyOn Cloud hub.

    Smart Modules docs
    Smart Modules Development Kit (smdk) docs

There are some limitations on the amount of polyglot development interface support. While bindings can be generated for wasm compatible languages, there are quirks in that approach. We have a better solution with Stateful Service Development Kit, which we are implementing using the web assembly component model. In the upcoming releases we will be able to natively support all wasm compatible programming languages.

    Stateful Service Development Kit docs- Coming Soon Request Developer Preview Invite

Try workflows on InfinyOn Cloud

InfinyOn Cloud is Fluvio on the cloud as a managed service. All new users get $3000 worth of credits to build data flows on InfinyOn Cloud.

    Check InfinyOn Cloud Guides
    Check out experimental data flows on InfinyOn Labs Repo

Clients

    Fluvio Client API docs home

Language Specifc API docs:

    Rust API docs
    Python API docs
    Javascript API docs
    Java API docs

Community Maintained:

    Go API docs
    Java API docs
    Elixir API docs
