---
group: php-developer-guide
title: Asynchronous Message Queue configuration files
contributor_name: comwrap GmbH
contributor_link: https://www.comwrap.com
functional_areas:
  - Services
---

When using the Magento message queue, four configuration files in your module must be updated:

*  communication.xml
*  queue_consumer.xml
*  queue_topology.xml
*  queue_publisher.xml

More information can be found in [Configure message queues]({{ page.baseurl }}/extension-dev-guide/message-queues/config-mq.html).

Asynchronous and Bulk APIs are built on top of the usual REST API and use the Magento Message Queue Framework for processing messages. To ease development efforts, the Asynchronous API pre-generates the following configuration files:

*  communication.xml
*  queue_publisher.xml

and provides the `queue_topology.xml` and `queue_consumer.xml` files within the `Magento/WebapiAsync` module.

### communication.xml

Information about the auto-generation of `communication.xml` can be found in [Topics in Asynchronous API]({{ page.baseurl }}/extension-dev-guide/message-queues/async-topics.html)

### queue_publisher.xml

The `queue_publisher.xml` file is generated by the `\Magento\WebapiAsync\Code\Generator\Config\RemoteServiceReader\Publisher` class, which implements `\Magento\Framework\Config\ReaderInterface` and is injected into the `\Magento\Framework\MessageQueue\Publisher\Config\CompositeReader` constructor argument in the main `di.xml` file.

```xml
<type name="Magento\Framework\MessageQueue\Publisher\Config\CompositeReader">
    <arguments>
        <argument name="readers" xsi:type="array">
            <item name="asyncServiceReader" xsi:type="object" sortOrder="0">Magento\WebapiAsync\Code\Generator\Config\RemoteServiceReader\Publisher</item>
            ...
        </argument>
    </arguments>
</type>
```

The sort order is set to 0 and allows developers to change some aspects of the generated configuration in configuration readers such as `queue_publisher.xml` and `env.php`.

`\Magento\WebapiAsync\Code\Generator\Config\RemoteServiceReader\Publisher::read()` calls `\Magento\AsynchronousOperations\Model\ConfigInterface::getServices()` to get an array of all REST API routes which can be executed asynchronously. Then it defines the connection name as `amqp` and the exchange as `magento` for each generated topic name.

### queue_consumer.xml

The asynchronous/bulk API has one defined consumer which processes all asynchronous/bulk APIs messages.

```xml
<consumer name="async.operations.all" queue="async.operations.all" connection="amqp"
              consumerInstance="Magento\AsynchronousOperations\Model\MassConsumer"/>
```

### queue_topology.xml

The message queue topology configuration links all auto-generated topic names with prefix `async.` to the `magento` exchange and defines the queue named `async.operations.all` as the destination for all messages.

```xml
<exchange name="magento" type="topic" connection="amqp">
    <binding id="async.operations.all" topic="async.#" destinationType="queue" destination="async.operations.all"/>
</exchange>
```