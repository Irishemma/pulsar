## Apache Flink Connectors for Pulsar

This page describes how to use the connectors to read and write Pulsar topics with [Apache Flink](https://flink.apache.org/) stream processing applications.

Build end-to-end stream processing pipelines that use Pulsar as the stream storage and message bus, and Apache Flink for computation over the streams.
See the [Pulsar Concepts](https://pulsar.apache.org/docs/en/concepts-overview/) page for more information.

## Example

### PulsarConsumerSourceWordCount

This Flink streaming job is consuming from a Pulsar topic and couting the wordcount in a streaming fashion. The job can write the word count results
to stdout or another Pulsar topic.

The steps to run the example:

1. Start Pulsar Standalone.

    You can follow the [instructions](https://pulsar.apache.org/docs/en/standalone/) to start a Pulsar standalone locally.

    ```shell
    $ bin/pulsar standalone
    ```

2. Start Flink locally.

    You can follow the [instructions](https://ci.apache.org/projects/flink/flink-docs-release-1.6/quickstart/setup_quickstart.html) to download and start Flink.

    ```shell
    $ ./bin/start-cluster.sh
    ```

3. Build the examples.

    ```shell
    $ cd ${PULSAR_HOME}
    $ mvn clean install -DskipTests
    ```

4. Run the word count example to print results to stdout.

    ```shell
    $ ./bin/flink run  ${PULSAR_HOME}/examples/flink-consumer-source/target/pulsar-flink-streaming-wordcount.jar --service-url pulsar://localhost:6650 --input-topic test_src --subscription test_sub
    ```

5. Produce messages to topic `test_src`.

    ```shell
    $ bin/pulsar-client produce -m "hello world test again" -n 100 test_src
    ```

6. You can check the flink taskexecutor `.out` file. The `.out` file will print the counts at the end of each time window as long as words are floating in, e.g.:

    ```shell
PulsarConsumerSourceWordCount.WordWithCount(word=hello, count=200)
PulsarConsumerSourceWordCount.WordWithCount(word=again, count=200)
PulsarConsumerSourceWordCount.WordWithCount(word=test, count=200)
PulsarConsumerSourceWordCount.WordWithCount(word=world, count=200)
PulsarConsumerSourceWordCount.WordWithCount(word=hello, count=100)
PulsarConsumerSourceWordCount.WordWithCount(word=again, count=100)
PulsarConsumerSourceWordCount.WordWithCount(word=test, count=100)
    ```

Alternatively, when you run the flink word count example at step 4, you can choose dump the result to another pulsar topic.

```shell
$ ./bin/flink run  ${PULSAR_HOME}/examples/flink-consumer-source/target/pulsar-flink-streaming-wordcount.jar --service-url pulsar://localhost:6650 --input-topic test_src --subscription test_sub --output-topic test_dest
```

Once the flink word count example is running, you can use `bin/pulsar-client` to tail the results produced into topic `test_dest`.

```shell
$ bin/pulsar-client consume -n 0 -s test test_dest
```

You will see similar results as what you see at step 6 when running the word count example to print results to stdout.
