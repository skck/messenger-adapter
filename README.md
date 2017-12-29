# Enqueue bridge for Symfony Message component

This bridge will allow you to use [Php-Enqueue](https://github.com/php-enqueue/enqueue-dev)'s great list of brokers for your messages that are going through Symfony's Message component.

## Usage

1. Install the enqueue bridge and the enqueue AMQP extension. (Note that you can use any of the multiple php-enqueue extensions)

```
composer req sroze/enqueue-bridge:dev-master enqueue/amqp-ext
```

2. Enqueue's recipes should have created configuration files and added a `ENQUEUE_DSN` to your `.env` and `.env.dist` files. 
   Change the DSN to match your queue broker, for example:
```yaml
# .env
# ...

###> enqueue/enqueue-bundle ###
ENQUEUE_DSN=amqp://guest:guest@localhost:5672/%2f
###< enqueue/enqueue-bundle ###
```

3. Register your consumer & producer
```yaml
# config/services.yaml
services:
    app.message_receiver:
        class: Sam\Symfony\Bridge\EnqueueMessage\EnqueueReceiver
        arguments:
        - "@message.transport.default_decoder"
        - "@enqueue.transport.default.context"
        - "messages" # Name of the queue
        public: true

    app.message_sender:
        class: Sam\Symfony\Bridge\EnqueueMessage\EnqueueSender
        arguments: 
        - "@message.transport.default_encoder"
        - "@enqueue.transport.default.context"
        - "messages" # Name of the queue
```

4. Route your messages to the sender
```yaml
# config/packages/framework.yaml
framework:
    message:
        routing:
            'App\Message\MyMessage': app.message_sender
```

You are done. The `MyMessage` messages will be sent to your local RabbitMq instance. In order to process
them asynchronously, you need to consume the messages pushed in the queue. You can start a worker with the `message:consume`
command:

```bash
bin/console message:consume app.message_receiver
```

## Misc

### Local RabbitMq with Docker

If you don't have a RabbitMq instance working, you can use Docker to very easily have one running:
```
docker run -d --hostname rabbit --name rabbit -p 8080:15672 -p 5672:5672 rabbitmq:3-management
```